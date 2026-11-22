# Local RAG over `.docx` files

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/lazizatemirova/local-rag-docx/blob/main/local_rag_docx_colab.ipynb)

A small retrieval-augmented question-answering app for Word documents that runs **entirely inside a free Google Colab T4 GPU** — no hosted APIs. Upload `.docx` files, build an index, and ask questions in English, Russian or Uzbek. Answers come only from your documents, with `[n]` citations and a sources panel.

## Pipeline

| Stage | Component |
|---|---|
| Loading | `python-docx` — paragraphs and tables in document order, heading paths kept as context |
| Chunking | ~500 LLM tokens, ~80-token overlap, paragraph/sentence boundaries |
| Embeddings | `BAAI/bge-m3` (multilingual), normalized vectors |
| Vector store | FAISS `IndexFlatIP` (cosine), optionally saved to Google Drive |
| Reranker (optional) | `BAAI/bge-reranker-v2-m3` cross-encoder over the top 20 candidates |
| LLM | `Qwen/Qwen2.5-7B-Instruct`, 4-bit NF4, greedy decoding (falls back to 3B if VRAM is short) |
| UI | Gradio (`share=True`) |

If the best retrieval score is below `MIN_SCORE`, the LLM is skipped and the app answers *"I couldn't find this in the documents."*

## Quick start

1. Click the **Open in Colab** badge above.
2. **Runtime → Change runtime type → T4 GPU**, then **Runtime → Run all**.
3. If Colab asks to restart the session after the install cell, accept and run all again.
4. Approve Google Drive access (optional — used to save/reuse the index and to read `.docx` files from `MyDrive/rag_docs/`).
5. Upload your `.docx` files when the widget appears. Skip it to use a saved index or a tiny built-in demo document.
6. Check the self-test output, then open the `gradio.live` link printed by the last cell.

The first run downloads roughly 17 GB of models (about 5–10 minutes).

## Tuning

All settings live in the `CONFIG` dictionary (Cell 2).

- **Chunk size / overlap** — 300–400 tokens for FAQs and precise facts, 700–900 for narrative text. Changing it requires re-indexing.
- **top-k** — 3–5 for precise questions, 8–10 for "summarize / list all" questions.
- **Reranker** — helps when many sections look alike; costs ~1.1 GB VRAM and ~0.5–1 s per question.
- **`MIN_SCORE`** — raise it if the app answers off-topic questions, lower it if it refuses too often.
- **`MAX_NEW_TOKENS`** — raise it for longer answers.
- **Embedder** — changing it requires re-indexing.

Details for each option are in the notebook's first cell.

## Notes

- Only `.docx` is supported. Convert `.doc` or `.pdf` files to `.docx` first.
- Image-only documents have no extractable text and are skipped.
- A 7B model quantized to 4-bit can be shallow on "explain and compare" questions; use a higher top-k and the reranker, and check cited passages against the sources panel.
