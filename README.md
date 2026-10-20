# Book Q&A RAG

A local Retrieval-Augmented Generation (RAG) app for asking questions about a PDF book. The Streamlit interface lets a user upload a PDF, indexes its text with the Hugging Face `sentence-transformers/all-mpnet-base-v2` embedding model, retrieves relevant passages with Chroma, and generates answers with the Groq API.

## Features

- Upload a PDF book in the Streamlit app.
- Split the book text into chunks and create embeddings.
- Retrieve relevant passages with Chroma using MMR search.
- Ask questions in a chat interface and view the source page numbers.
- Save a separate persistent Chroma database for each uploaded PDF under `chroma_db/<file-hash>/`.

## Requirements

- Python 3.10 or newer
- A Groq API key
- Internet access the first time the embedding model is downloaded, and when calling Groq

The embedding model is downloaded from the Hugging Face Hub the first time it runs. A Hugging Face token is optional for this public model; without one, the Hub may apply lower rate limits.

## Setup

Open PowerShell in the project directory and run:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If PowerShell blocks activation for this session, run this once in that terminal and activate again:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\.venv\Scripts\Activate.ps1
```

For local use, create a `.env` file in the project root and add your Groq API key:

```text
GROQ_API_KEY=your_groq_api_key_here
```

Do not commit `.env` or share your API key.

For Streamlit Community Cloud, add the key in the app's **Settings → Secrets** (or in the deploy dialog's **Advanced settings**) in TOML format:

```toml
GROQ_API_KEY = "your_groq_api_key_here"
```

## Run the Streamlit app

```powershell
streamlit run app.py
```

Open the local URL printed in the terminal (usually `http://localhost:8501`), upload a PDF, wait for indexing to finish, and ask a question. The first run may take longer while the Hugging Face embedding model downloads. Indexing time also depends on the PDF size and your computer.

The current uploader accepts text-based PDFs. Scanned PDFs that contain page images instead of selectable text may need OCR before this app can search their contents.

## Project files

- `app.py` — Streamlit UI, PDF ingestion, retrieval, and question answering.
- `create_database.py` — standalone script for creating a persistent Chroma database from its configured PDF path.
- `main.py` — command-line question-answering script for the persistent `chroma_db` database.
- `chroma_db/` — local persistent Chroma databases. The Streamlit app creates one subfolder per uploaded PDF; `create_database.py` uses the configured standalone database path.

## Notes

- The Streamlit upload flow saves a persistent Chroma index in `chroma_db/<file-hash>/` for each uploaded PDF. Uploading the same PDF again reuses its saved index. A different PDF gets a separate folder, so it does not overwrite the earlier upload's index.
- `main.py` loads the persistent `chroma_db` folder. Run `create_database.py` first if you want to use the command-line flow.
- Keep API keys, virtual environments, uploaded books, and generated database files out of GitHub.
