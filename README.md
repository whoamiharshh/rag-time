# RAG Pipeline: Ask Questions About Your Own Documents

A beginner-friendly **Retrieval-Augmented Generation (RAG)** project built step by step in a Jupyter notebook. It reads your documents, stores them in a vector database, finds the passages most relevant to a question, and asks an LLM to answer using only those passages.

Every answer comes with its **source files** and a **confidence score**, and the full context sent to the LLM can be shown on request.

## How it works

```
Documents -> Load -> Split into chunks -> Embed -> Store in ChromaDB      (done once)

Question  -> Embed -> Find closest chunks -> Build prompt -> LLM -> Answer  (every question)
                                                                    + sources + confidence
```

1. **Load:** reads files from the `data/` folder and turns them into LangChain `Document` objects.
2. **Split:** cuts each document into chunks of about 800 characters (100 overlap) with `RecursiveCharacterTextSplitter`.
3. **Embed:** converts every chunk into a 384-number vector with `all-MiniLM-L6-v2` (Sentence Transformers).
4. **Store:** saves chunks, vectors and metadata in a persistent **ChromaDB** collection.
5. **Retrieve:** embeds the question and returns the closest chunks (cosine similarity).
6. **Generate:** sends the chunks and the question to a **Groq**-hosted LLM, which is told to answer only from that text and to say "I don't know" otherwise.

## Features

- Loaders for `.txt`, `.md`, `.pdf`, `.docx`, `.html`, `.pptx` and `.xlsx` files
- Persistent vector database, so you embed your documents once
- Answers with **sources** and a **confidence score** (High / Medium / Low)
- Optional display of the **full context** given to the LLM
- Returns "I don't know" when nothing relevant is found, instead of guessing

## Tech stack

| Part | Tool |
|---|---|
| Language | Python 3.14 |
| Orchestration | LangChain (`langchain-core`, `langchain-community`, `langchain-text-splitters`, `langchain-groq`) |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) |
| Vector database | ChromaDB |
| LLM | Groq (`openai/gpt-oss-20b`) |
| File readers | `pypdf`, `python-docx`, `python-pptx`, `openpyxl`, `beautifulsoup4` |

## Project structure

```
rag 1/
├── notebook/
│   ├── doument.ipynb        # the whole pipeline, step by step
│   └── data/
│       ├── txt_files/       # sample documents (sample.txt, ml.txt)
│       └── vector_store/    # ChromaDB files (created when you run the notebook)
├── src/rag_1/               # package skeleton
├── requirements.txt         # dependencies
├── .env                     # your API key (never committed)
└── README.md
```

## Getting started

1. **Clone the repo and create an environment** (Python 3.14):
   ```bash
   uv venv
   uv pip install -r requirements.txt
   ```
2. **Add your Groq API key.** Get a key at [console.groq.com](https://console.groq.com) and create a file named `.env` in the project root:
   ```
   GROQ_API_KEY=your-key-here
   ```
   `.env` is listed in `.gitignore`, so the key is not uploaded.
3. **Add your documents** to `notebook/data/` (any subfolder). The repo includes two sample text files in `notebook/data/txt_files/`.
4. **Open `notebook/doument.ipynb`**, select the project's `.venv` kernel and run the cells from top to bottom.
5. **Ask a question** in the last cell. You'll be prompted to type it.

Example result:

```
Question  : What is machine learning?
Answer    : Machine learning (ML) is a field of study in artificial intelligence concerned
            with the development and study of statistical algorithms that can learn from
            data and generalize to unseen data...
Sources   : ['data\\txt_files\\ml.txt']
Confidence: 67.0% (High)
```

## Status

- Tested end to end (load, split, embed, store, retrieve, answer) on **text files**.
- Loading and splitting also verified for **Markdown** and **Excel** files.
- Loaders for PDF, Word, HTML and PowerPoint are implemented, and testing them with real files is in progress.
- Your own documents are not included in the repository. Add them to `notebook/data/` before running the notebook.

## What I learned

- How embeddings turn text into numbers so similar meanings can be searched
- Why documents are split into overlapping chunks before embedding
- How a vector database stores and searches embeddings
- How to build a prompt that keeps an LLM grounded in the retrieved context

## Ideas for next steps

- Test and tune chunk size for PDFs and slides
- Add a simple chat interface (Streamlit or Gradio)
- Re-rank retrieved chunks for better accuracy
- Add automated tests for the loaders
