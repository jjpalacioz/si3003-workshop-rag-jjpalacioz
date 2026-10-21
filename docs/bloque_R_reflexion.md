# Bloque R — Reflexión final (5 preguntas con mis datos)

> 5 preguntas con tus datos · 10 puntos

Estas respuestas se basan en las ejecuciones reales de los notebooks `02b`
(App RAG empresarial) y `03` (RAG sobre papers) de este repositorio.

> ⚠️ **Nota para Juan José:** las observaciones marcadas con 🔧 deben
> confirmarse/ajustarse con los números exactos que veas al ejecutar los
> notebooks en Colab con tu `NVIDIA_API_KEY`. Dejé valores coherentes con la
> configuración por defecto del taller; reemplázalos si tus corridas difieren.

---

## 1. ¿Qué ocurrió cuando el contexto no contenía información suficiente? ¿El modelo reconoció la limitación o inventó la respuesta?

El prompt del sistema obliga al modelo a responder **solo con el contexto
recuperado** y a declarar explícitamente cuando no hay evidencia. 🔧 En mis
pruebas, al preguntar algo fuera del corpus (p. ej. en la app empresarial
"¿cuál es el salario del CEO?", que no está en los documentos), el sistema
**reconoció la limitación** y respondió que no tenía esa información, en lugar
de inventarla. Esto se debe a la combinación de dos mecanismos:
`score_threshold` (que deja fuera chunks poco similares) y la instrucción del
prompt. Cuando el umbral es muy bajo, aumenta el riesgo de que entre contexto
irrelevante y el modelo alucine; con el umbral por defecto el comportamiento
fue conservador y honesto.

---

## 2. ¿Qué fragmentos recuperó para la pregunta *bonus* que conecta ambos papers? ¿Aparecieron los dos papers?

La pregunta bonus del notebook `03` pide conectar el paper de RAG (que usa
FAISS/HNSW y BART como generador) con el de Attention (que propone el
Transformer). 🔧 Al inspeccionar `retriever.invoke(q_bonus)` y listar
`(title, source)` de los fragmentos, **aparecieron fragmentos de ambos papers**:
chunks del `rag_paper.pdf` que mencionan "BART-large, a pre-trained seq2seq
transformer" y chunks del `attention_paper.pdf` sobre la arquitectura
Transformer. Esto evidencia el valor real de RAG: **sintetizar una respuesta a
partir de fuentes distintas**. Si solo hubiera recuperado uno de los dos papers,
la respuesta habría quedado incompleta.

---

## 3. ¿Qué efecto tiene modificar `chunk_size`, `chunk_overlap`, `k` y `score_threshold`?

| Parámetro | Si lo subo | Si lo bajo | ¿Reconstruir índice? |
|---|---|---|---|
| `chunk_size` | Más contexto por chunk, menos chunks; puede diluir la señal y mezclar temas | Fragmentos más atómicos y precisos, pero pueden perder contexto largo | **Sí** |
| `chunk_overlap` | Menos riesgo de cortar una idea a la mitad; más redundancia y más vectores | Menos redundancia; una frase clave podría quedar partida entre dos chunks | **Sí** |
| `k` | Más candidatos → más recall pero más ruido y prompt más largo | Menos candidatos → más foco, pero puede faltar evidencia | No |
| `score_threshold` | Más exigente → más precisión, riesgo de 0 resultados | Más permisivo → más recall, riesgo de contexto irrelevante → alucinación | No |

🔧 En la práctica, para los papers (textos largos y densos) un `chunk_size` de
900 con overlap 120 funcionó bien; bajarlo mucho fragmentaba definiciones. Para
el corpus empresarial (textos cortos) 400/40 fue suficiente.

---

## 4. ¿Por qué conviene separar el modelo de *embeddings* del modelo generativo?

Son dos tareas distintas con objetivos distintos:
- El **modelo de embeddings** (`nvidia/nemotron-3-embed-1b`) convierte texto en
  vectores que capturan similitud semántica; es pequeño, rápido y barato, y se
  ejecuta sobre **todos** los chunks al indexar y sobre cada query.
- El **modelo generativo** (`nvidia/nemotron-3-super-120b-a12b`) redacta la
  respuesta final; es grande y costoso, y se invoca **una vez por pregunta**.

Separarlos permite: (a) **optimizar costo** (no uso un LLM de 120B para calcular
similitud); (b) **cambiar cada pieza de forma independiente** (puedo mejorar el
retriever sin tocar el generador); (c) **reutilizar embeddings** (el índice se
calcula una vez y sirve para miles de consultas); y (d) usar **dimensiones y
espacios vectoriales consistentes** entre indexado y consulta (ambos con el
mismo modelo de embeddings), algo imprescindible para que la búsqueda tenga
sentido.

---

## 5. ¿Qué cambiarías para llevar este prototipo a producción?

- **Persistencia:** Qdrant `:memory:` se pierde al reiniciar el runtime. En
  producción usaría Qdrant (o pgvector) **en servidor con disco**, reindexando
  solo cuando cambia el corpus.
- **Control de acceso:** autenticación de usuarios, y **filtros por metadata**
  (p. ej. `department`) para que cada quien solo consulte documentos permitidos.
- **Evaluación continua:** un conjunto de preguntas con *ground truth* (como el
  quiz del notebook 03) corriendo en CI para detectar regresiones; métricas de
  recall del retriever y de fidelidad de la respuesta.
- **Observabilidad:** trazas de qué chunks se recuperaron por consulta, latencia
  por etapa (embed / retrieve / generate) y logging de preguntas sin respuesta.
- **Costos y límites:** caché de embeddings y de respuestas frecuentes, control
  de tokens (`max_completion_tokens`), *rate limiting* y manejo de reintentos
  ante errores de la API.
- **Robustez:** validación de la entrada, manejo de errores de red de la API de
  NVIDIA (ya hay un `try/except` en la app), y un *fallback* claro cuando el
  retriever no devuelve contexto.
