# Evidencias de ejecución

Capturas de pantalla que demuestran que los notebooks del Workshop RAG se
ejecutaron correctamente en Google Colab con NVIDIA NIM.

> **Cómo subir las capturas:** entra a esta carpeta en GitHub
> (`docs/evidencias/`), pulsa **Add file → Upload files**, arrastra tus imágenes
> y haz *Commit changes*. Usa los nombres sugeridos abajo para que coincidan con
> las referencias de este documento.

---

## Parte A — App RAG empresarial (notebook `02b`)

### `A_respuesta_enterprise.png`
La app Gradio responde a *"¿Cuánto cuesta el plan Enterprise de AuraCode?"* con
**"$69/mes por desarrollador con compromiso anual. (Fuente: AuraCode - Planes y
precios)"**.

Esto demuestra de un solo golpe:
- **A2** — corpus propio AuraCode.
- **A3** — prompt mejorado: la respuesta **cita la fuente** entre paréntesis.
- **A4** — la app muestra las fuentes recuperadas.
- Además confirma que el **modelo de generación auto-seleccionado**
  (`nvidia/nemotron-3.5-super-vl-120b-a12b`) funciona, tras el fix del modelo
  deprecado.

### `A_pregunta_trampa.png` *(opcional pero recomendada)*
La app responde a *"¿Cuál es el salario del CEO?"* con **"No tengo esa
información en los documentos"** — demuestra que el sistema **no alucina** cuando
el dato no está en el corpus.

---

## Parte B — RAG sobre papers (notebook `03`)

### `B_quiz.png`
Salida del quiz (preguntas 1–5) donde se ve, por ejemplo, el **BLEU 28.4** del
Transformer en la pregunta 1 y las respuestas citando el paper correcto.

---

## Retos (notebook `04`)

### `C1_faiss_vs_hnsw.png`
Tabla `efSearch | recall@5 | ms/consulta` del reto C1.

### `C4_evaluacion.png`
Línea final con el **keyword-recall promedio** del reto C4.

---

> Reemplaza o añade capturas según lo que ejecutes. Lo mínimo para evidenciar la
> Parte A ya está cubierto con `A_respuesta_enterprise.png`.
