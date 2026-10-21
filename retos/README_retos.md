# Retos — resumen (hasta 30 puntos)

Notebook: [`04_retos_rag.ipynb`](04_retos_rag.ipynb). Autosuficiente; requiere
`NVIDIA_API_KEY`. Resuelve cuatro retos (tope de 30 pts):

| Reto | Título | Idea | Resultado esperado |
|---|---|---|---|
| **C1** | FAISS vs HNSW | Comparar índice exacto (`IndexFlatIP`) contra aproximado (`IndexHNSWFlat`) variando `efSearch`. | Tabla `efSearch → recall@5 → ms/consulta`: al subir `ef`, recall↑ y latencia↑; `ef ≥ k`. |
| **C2** | Tercer paper | Añadir **BERT** (arXiv 1810.04805) al índice y preguntar algo que conecte los tres papers. | La respuesta cita RAG + BERT + Attention; el retriever trae chunks de los tres. |
| **C3** | Filtros por metadata | Buscar con y sin filtro `source = attention_paper.pdf` usando el payload de Qdrant. | El filtro limita los resultados a un solo paper sin reconstruir el índice. |
| **C4** | Evaluación automática | Métrica `keyword-recall` sobre un set con *ground truth*. | `keyword-recall` promedio reproducible; sirve para CI y para comparar configs. |

## Cómo se relaciona con la teoría (Bloque 0)
- **C1** materializa el compromiso **`ef` ↔ recall/velocidad** de la presentación
  "Búsqueda en HNSW" y la regla **`ef ≥ k`**.
- **C3** demuestra por qué **Qdrant guarda la metadata junto al vector** (resuelve
  el invariante frágil del `FAISSVectorStore` y habilita filtrado).

> 🔧 Tras ejecutar en Colab, pega aquí la tabla real de C1 y el `keyword-recall`
> de C4 para dejar evidencia numérica de tus corridas.
