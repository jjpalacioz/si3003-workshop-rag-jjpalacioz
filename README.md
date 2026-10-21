# Workshop RAG — SI3003 Inteligencia Artificial (EAFIT)

**Autor:** Juan José Palacio
**Curso:** SI3003 — Inteligencia Artificial · Lecture 10 (RAG)
**Tema:** Retrieval-Augmented Generation con LangChain, NVIDIA NIM y Qdrant.

Este repositorio es mi entrega del Workshop RAG. Construye sistemas RAG sobre
dos dominios (documentos de empresa y papers académicos), responde la teoría de
índices vectoriales (HNSW/FAISS) y resuelve varios retos de la bonificación.

---

## ¿Qué es RAG? (en una frase)

RAG = **buscar** los fragmentos de mi propio contenido más parecidos a la
pregunta, y **dárselos a un LLM** para que responda con esa evidencia en vez de
inventar. Dos piezas: un **retriever** (índice vectorial) y un **generador** (LLM).

---

## Estructura del repositorio

```
.
├── README.md                     # este archivo
├── requirements.txt              # dependencias (se instalan en Colab)
├── notebooks/
│   ├── 02b_rag_app_nvidia_nim_colab.ipynb   # Parte A — App RAG empresarial (Gradio)
│   └── 03_rag_workshop_papers_nvidia_nim_colab.ipynb  # Parte B — RAG sobre papers
├── retos/
│   ├── 04_retos_rag.ipynb        # Retos C1–C4
│   └── README_retos.md
├── data/
│   ├── company_documents.json        # corpus propio (AuraCode) — Parte A
│   └── company_documents_demo.json   # corpus demo original (referencia)
├── papers/
│   ├── attention_paper.pdf
│   └── rag_paper.pdf
└── docs/
    ├── bloque0_teoria.md         # Bloque 0 — Teoría (HNSW/FAISS)
    └── bloque_R_reflexion.md     # Bloque R — Reflexión final
```

---

## Qué cubre esta entrega (mapa a la bonificación)

| Bloque | Puntos | Dónde está | Estado |
|---|---|---|---|
| **0 · Teoría** | 10 | [`docs/bloque0_teoria.md`](docs/bloque0_teoria.md) | ✅ Escrito |
| **A · App RAG** | 20 | [`notebooks/02b_...ipynb`](notebooks/02b_rag_app_nvidia_nim_colab.ipynb) | ✅ Código listo · ⏳ ejecutar en Colab |
| **B · Papers** | 30 | [`notebooks/03_...ipynb`](notebooks/03_rag_workshop_papers_nvidia_nim_colab.ipynb) | ✅ Código listo · ⏳ ejecutar en Colab |
| **R · Reflexión** | 10 | [`docs/bloque_R_reflexion.md`](docs/bloque_R_reflexion.md) | ✅ Escrito (ajustar 🔧 con tus corridas) |
| **Retos** | hasta 30 | [`retos/04_retos_rag.ipynb`](retos/04_retos_rag.ipynb) | ✅ Código listo · ⏳ ejecutar en Colab |

### Parte A — App RAG empresarial (notebook `02b`)
- **A1** ejecutar el pipeline end-to-end.
- **A2** **corpus propio**: plataforma *AuraCode* (8 documentos en `data/company_documents.json`).
- **A3** **prompt mejorado**: anti-alucinación, citado de fuente y formato fijo.
- **A4** **scores**: la interfaz Gradio muestra el score de similitud de cada fragmento recuperado.

### Parte B — RAG sobre papers (notebook `03`)
- **B1** quiz de 5 preguntas comparado con *ground truth*.
- **B2** experimento de **hiperparámetros** (`k`, `score_threshold`).
- **B3** **preguntas trampa** (fuera de los papers) para verificar que no alucina.
- **B4** **español**: preguntar en español sobre papers en inglés.
- **B5** **mi propio quiz**: 5 preguntas nuevas con su *ground truth*.

### Retos (notebook `04`)
- **C1** FAISS exacto vs HNSW aproximado (tabla `ef → recall → latencia`).
- **C2** tercer paper (**BERT**) y pregunta que conecta los tres.
- **C3** filtros por metadata en Qdrant.
- **C4** evaluación automática (`keyword-recall`).

---

## Cómo ejecutar (Google Colab)

1. Crea una **API key de NVIDIA** en <https://build.nvidia.com/> (gratis).
2. Abre el notebook en Colab. En **🔑 Secrets** crea un secreto
   `NVIDIA_API_KEY` y habilítalo para el notebook (o pégalo cuando lo pida).
3. Para la **Parte A**, sube la carpeta `data/` al entorno de Colab (o el
   archivo `company_documents.json`), para que use el corpus propio.
4. Ejecuta las celdas **en orden**. La app de la Parte A publica una URL pública
   de Gradio; las Partes B y los Retos imprimen resultados por celda.

**Modelos usados:**
- Embeddings: `nvidia/nemotron-3-embed-1b`
- Generación: **selección automática** del primer modelo de chat disponible en tu
  cuenta (prioriza `nvidia/nemotron-3.5-super-vl-120b-a12b` y Nemotron recientes).

> ⚠️ **Nota importante:** el modelo original del curso
> `nvidia/nemotron-3-super-120b-a12b` fue **deprecado por NVIDIA el 19/10/2026**.
> Por eso los notebooks ahora **eligen el modelo de generación automáticamente**
> (función `pick_generation_model`), de modo que siguen funcionando aunque NVIDIA
> renombre o retire un modelo. Al ejecutar verás impreso qué modelo se seleccionó.

---

## Pendiente antes de entregar (para Juan José)

- [ ] Ejecutar los tres notebooks en Colab con mi `NVIDIA_API_KEY`.
- [ ] Reemplazar las marcas 🔧 (ground truth verificado, tabla de C1,
      keyword-recall de C4) con los números reales de mis corridas.
- [ ] Revisar el Bloque R con las observaciones reales.
