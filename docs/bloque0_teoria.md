# Bloque 0 — Teoría: cómo busca un índice vectorial (HNSW y FAISS)

> 5 ejercicios sobre HNSW y FAISS · 10 puntos
>
> Basado en las cuatro presentaciones del Workshop RAG (Small World y HNSW,
> Inserción de un vector, Búsqueda en HNSW, FAISSVectorStore).

Este documento resume los conceptos de la teoría y responde los cinco
ejercicios del bloque, además de las tres preguntas guía que el sitio del
taller propone llevarse a los notebooks.

---

## Marco conceptual (resumen de las 4 presentaciones)

### 1. Small World y HNSW
Comparar la pregunta contra **todos** los vectores (fuerza bruta) cuesta `O(N)`
por consulta: no escala cuando hay millones de vectores. En un grafo de tipo
**Small World** cualquier nodo llega a cualquier otro en pocos saltos porque
hay enlaces cortos (vecinos cercanos) y algunos enlaces largos (atajos). **NSW**
construye ese grafo insertando puntos y conectándolos a sus vecinos más
próximos. **HNSW** (Hierarchical NSW) apila varias capas: las capas de arriba
tienen pocos nodos con enlaces largos (para moverse rápido entre zonas lejanas)
y las capas de abajo son densas (para afinar). La búsqueda baja de capa en capa.

### 2. Inserción de un vector
Al insertar un vector nuevo: (a) se **sortea su capa máxima** con una
distribución exponencial (la mayoría caen en capas bajas, pocos llegan arriba);
(b) desde la capa más alta se hace un **descenso greedy** hasta acercarse a la
zona del vector; (c) en cada capa donde vive, se conecta a sus vecinos usando
una cola de candidatos de tamaño **`ef_construction`** (más grande = grafo de
mejor calidad pero inserción más lenta) y un límite de conexiones por nodo `M`.

### 3. Búsqueda en HNSW — tres fases
1. **Entrada por la cima:** empieza en el nodo de entrada de la capa superior.
2. **Descenso greedy por capas superiores:** en cada capa se mueve al vecino
   más cercano a la consulta hasta que no mejora, y baja una capa.
3. **Búsqueda fina en la capa 0:** explora el grafo denso manteniendo una cola
   de tamaño **`ef`** (ef_search). Devuelve los `k` mejores.
El compromiso clave: **`ef` grande → mayor recall (más preciso) pero más lento**;
`ef` pequeño → más rápido pero puede perder vecinos. Regla: **`ef ≥ k`**.

### 4. FAISSVectorStore — el invariante
FAISS indexa **solo vectores** y al buscar devuelve **enteros** (posiciones:
"vector #42"). Para saber qué documento es cada posición, el `FAISSVectorStore`
mantiene **una lista paralela** `int → Document`. El **invariante** es que el
índice FAISS y esa lista deben estar **perfectamente alineados y del mismo
tamaño**: si se desincronizan (p. ej. borras un vector y no actualizas la
lista), la posición `i` apunta al documento equivocado y las respuestas se
corrompen silenciosamente.

---

## Ejercicio 1 — ¿Por qué la fuerza bruta no escala y cómo lo resuelve HNSW?

**Respuesta.** La búsqueda por fuerza bruta (flat / exact) calcula la distancia
de la consulta contra los `N` vectores del índice, con costo `O(N·d)` por
consulta (`d` = dimensión). Con millones de documentos esto es inviable en
tiempo real. HNSW convierte el problema en una **búsqueda aproximada sobre un
grafo navegable**: gracias a la propiedad *small world* (enlaces cortos + unos
pocos atajos largos) y a la **jerarquía de capas**, llega a la vecindad correcta
en un número de saltos que crece de forma **logarítmica** (`~O(log N)`) en lugar
de lineal. Paga un pequeño precio en recall (es aproximado, no exacto) a cambio
de una aceleración enorme.

| | Fuerza bruta (flat) | HNSW |
|---|---|---|
| Costo por consulta | `O(N)` | `~O(log N)` |
| Resultado | Exacto | Aproximado (recall < 100%) |
| Memoria | Baja (solo vectores) | Mayor (vectores + grafo) |
| Escala a millones | No | Sí |

---

## Ejercicio 2 — Rol de `ef_construction` en la inserción

**Respuesta.** `ef_construction` es el tamaño de la cola de candidatos que HNSW
usa **al insertar** un vector para decidir a qué vecinos conectarlo. Un valor
alto explora más candidatos → el nuevo nodo se conecta a vecinos de mejor
calidad → el grafo resultante tiene **mejor recall en las búsquedas**, pero la
**construcción del índice es más lenta** y usa más memoria. Es un parámetro de
*build time* (no cambia en cada consulta). No debe confundirse con `ef`
(`ef_search`), que actúa en *query time*. También interviene `M`, el número
máximo de conexiones por nodo: más `M` = grafo más denso y preciso, pero más
memoria.

---

## Ejercicio 3 — Las tres fases de una búsqueda HNSW y el papel de `ef`

**Respuesta.** (1) **Entrada** por el nodo de la capa superior. (2) **Descenso
greedy** por las capas altas: en cada capa avanza al vecino más cercano a la
query hasta que ninguno mejora, luego baja. (3) **Búsqueda fina en la capa 0**
con una cola dinámica de tamaño **`ef`**: mantiene los mejores candidatos vistos
y expande sus vecinos hasta que no puede mejorar; finalmente recorta a `k`.

- `ef` **grande** → explora más grafo → **mayor recall** pero más latencia.
- `ef` **pequeño** → menos exploración → más rápido pero puede **omitir** vecinos
  verdaderos.
- Restricción práctica: **`ef ≥ k`** (no puedes devolver más resultados de los
  que la cola llegó a considerar).

---

## Ejercicio 4 — El invariante del `FAISSVectorStore` y cómo lo resuelve Qdrant

**Respuesta.** FAISS solo almacena vectores y devuelve **índices enteros**. El
`FAISSVectorStore` de LangChain guarda aparte un mapa **`int → Document`**
(`index_to_docstore_id` + un docstore). El **invariante** es que ese mapa y el
índice FAISS deben mantenerse **sincronizados** (mismo tamaño, mismas
posiciones): si se rompe, el entero `i` devuelto por FAISS apunta a un documento
incorrecto. **Qdrant** elimina ese riesgo porque **guarda la metadata (payload)
junto al vector dentro del propio índice**: cada punto ya conoce su `title`,
`source`, `chunk_index`, etc. No hay lista externa que mantener alineada y,
además, permite **filtrar por metadata** directamente durante la búsqueda.

---

## Ejercicio 5 — Diferencia entre `k` y `ef` (y por qué `ef ≥ k`)

**Respuesta.**
- **`k`** es cuántos fragmentos **devuelve** el retriever al usuario. Vive en
  `search_kwargs={"k": ...}`. Es una decisión de la **consulta**, no modifica el
  índice.
- **`ef`** (ef_search) es el tamaño de la **cola interna de exploración** que el
  índice HNSW usa mientras recorre el grafo en la capa 0. Es un parámetro del
  **algoritmo de búsqueda**.

Deben cumplir **`ef ≥ k`**: la cola de exploración tiene que contener al menos
tantos candidatos como resultados se quieren devolver; con `ef < k` el índice
no habría considerado suficientes nodos para entregar `k` vecinos de calidad.
Dos números distintos: `k` = cuántos entrego, `ef` = cuánto exploro para
elegirlos.

---

## Tres preguntas guía (para llevar a los notebooks)

**1. Cuando subes `score_threshold`, ¿cambias el índice o la consulta?**
La **consulta**. `score_threshold` es un filtro de *query time* que descarta
resultados cuya similitud es menor al umbral. El grafo/índice no se toca; solo
se vuelve más exigente el criterio de qué se acepta como "relevante". Efecto
práctico: subirlo mejora la precisión pero puede dejar preguntas sin contexto
(el retriever devuelve menos o ningún documento).

**2. Si cambias `chunk_size`, ¿hay que reconstruir el índice? ¿Y si cambias `k`?**
- `chunk_size` → **SÍ**. Cambiar el tamaño de los fragmentos produce textos
  distintos → embeddings distintos → el índice entero queda obsoleto y debe
  reconstruirse (`add_documents` sobre una colección nueva).
- `k` → **NO**. `k` solo cambia cuántos resultados pide la consulta; el índice
  permanece igual.

**3. ¿Por qué Qdrant guarda la metadata junto al vector y qué problema del
`FAISSVectorStore` resuelve?**
Porque así cada vector es autodescriptivo: transporta su `title`, `source` y
`chunk_index` dentro del índice. Resuelve el problema del **invariante frágil**
de FAISS (mantener sincronizada una lista externa `int → Document`) y habilita
**filtrado por metadata** en la propia búsqueda, algo que FAISS plano no ofrece.
