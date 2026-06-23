# Multimodal Document RAG with ColQwen2 + Llama 3.2 Vision / GPT-4o

A visual, OCR-free Retrieval-Augmented Generation (RAG) system that lets you upload a document (PDF, DOCX, PPTX, TXT, or an image) and ask natural-language questions about it. Instead of chunking text, the system retrieves the most relevant **page image** and hands it directly to a vision-language model to answer — no OCR, no text extraction pipeline, no lost layout/table/chart context.

This project demonstrates core skills for an AI/ML engineering role: **RAG, embedding-based applications, and prompt engineering**.

## Why this approach

Traditional RAG extracts text from documents before embedding it, which breaks down on tables, charts, scanned pages, and complex layouts. This project uses **ColPali/ColQwen2**, a multimodal retriever that embeds page *images* directly, so retrieval quality isn't bottlenecked by OCR errors. The retrieved page image is then passed to a vision-capable LLM that reasons over the page visually.

## How it works

1. **Ingest** — Upload a PDF, DOCX, PPTX, TXT file, or image through a Gradio UI.
2. **Normalize** — Non-PDF formats (DOCX/PPTX/TXT) are converted to PDF on the fly (via `python-docx`, `python-pptx`, and `reportlab`) so every document type flows through the same visual indexing pipeline.
3. **Index** — `ColQwen2` (via the `byaldi` library) embeds each page as an image and builds a searchable index — no text extraction step.
4. **Retrieve** — The user's question is embedded and matched against the page index to retrieve the most relevant page (image, not text).
5. **Reason** — The retrieved page image + the question are sent to a vision-language model, which reads the page directly and answers — including content in tables, charts, and diagrams that text-only RAG would miss.
6. **Respond** — The answer and the retrieved page image are displayed together in the UI, so the answer is traceable to its source.

Two interchangeable backends are implemented for step 5:

| Variant | Vision-Language Model | Served via |
|---|---|---|
| `multimodal_rag` (Llama variant) | Llama-3.2-90B-Vision-Instruct-Turbo | Together AI API |
| `multimodal_rag` (GPT-4o variant) | GPT-4o | OpenAI API |

## Tech Stack

- **Retrieval:** ColQwen2 (ColPali-family multimodal retriever) via `byaldi`
- **Reasoning:** Llama 3.2 90B Vision (Together AI) or GPT-4o (OpenAI)
- **Multi-format ingestion:** `python-docx`, `python-pptx`, `reportlab`, `pdf2image`, `Pillow`
- **Interface:** Gradio
- **Core:** Python

## Project Structure

```
.
├── MultiModal_RAG_with_ColQwen2_and_Llama_3_2_Vision_GPT_4o.ipynb
└── README.md
```

## Setup

```bash
pip install byaldi together openai pdf2image gradio filetype python-docx python-pptx reportlab
sudo apt-get install -y poppler-utils
```

> A T4 GPU (e.g., free-tier Google Colab) is recommended for faster ColQwen2 indexing. Also runs on a 16GB M1 MacBook Pro.

## Usage

1. Open the notebook and run all cells to launch the Gradio app.
2. Enter your **Together AI** or **OpenAI** API key (used at runtime only, never stored or logged).
3. Upload a document (PDF / DOCX / PPTX / TXT / image).
4. Ask a question about its content.
5. View the model's answer alongside the exact page image it was retrieved from.

## What I Built / Adapted

The core ColPali + Together AI retrieval pattern is based on [Together AI's cookbook example](https://github.com/togethercomputer/together-cookbook/blob/main/MultiModal_RAG_with_Nvidia_Investor_Slide_Deck.ipynb). On top of that foundation, I:

- Extended the pipeline to accept **DOCX, PPTX, and TXT** inputs (not just PDFs) by building a format-normalization layer that converts each to a paginated PDF before indexing.
- Implemented a **second, independent backend using GPT-4o** as a drop-in alternative reasoning engine to the Together-hosted Llama 3.2 Vision model.
- Wrapped both pipelines in an interactive **Gradio application** with file upload, API-key input, and side-by-side answer + retrieved-page display, so the system is usable end-to-end without touching code.

## Relevance to an AI/ML Engineer Role

| Skill Area | Demonstrated By |
|---|---|
| RAG (Retrieval Augmented Generation) | End-to-end visual RAG pipeline (retrieval + generation) |
| Embedding-based applications | ColQwen2 page-image embeddings for retrieval |
| Prompt engineering | Structured multimodal prompts to Llama 3.2 Vision / GPT-4o |
| Python (Pandas/NumPy-adjacent tooling) | Full pipeline implemented in Python |
| API integration | Together AI and OpenAI API integration |
| Front-end integration for AI apps | Gradio-based interactive UI |
| LLM-powered workflows for data extraction | Multi-format document parsing into a unified pipeline |

## Future Improvements

- Persist indexes across sessions instead of re-indexing per query
- Add multi-page retrieval (top-k > 1) with answer aggregation across pages
- Add a vector database (FAISS/ChromaDB) for hybrid text + visual retrieval at scale
- Containerize with Docker for reproducible deployment

## Acknowledgements

Built on top of the multimodal RAG pattern from [Together AI's cookbook](https://github.com/togethercomputer/together-cookbook), using [ColPali](https://arxiv.org/abs/2407.01449)/ColQwen2 via [byaldi](https://github.com/AnswerDotAI/byaldi).
