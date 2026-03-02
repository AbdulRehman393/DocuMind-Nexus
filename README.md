# DocuMind Nexus

**DocuMind Nexus** is a docs‑first **RAG (Retrieval‑Augmented Generation)** assistant that lets you upload **PDF, DOCX, and HTML** files, index them into a **Chroma vector database**, and chat with an LLM to get grounded answers. It also includes an optional **live web search fallback** using **SerpAPI** for “today/current/latest” questions.

> Built with **FastAPI + Streamlit + LangChain + Chroma**.

---

## Highlights

- **Upload & Index**: PDF / DOCX / HTML
- **Docs‑first RAG**: searches your indexed documents before anything else
- **Chroma Vector Store**: persisted locally in `./chroma_db`
- **Session Memory**: chat history stored in SQLite (`rag_app.db`)
- **Live Web Search Fallback**: SerpAPI used for current info (weather/news/date)
- **Offline Embeddings Demo Mode**: `SimpleHashEmbeddings` (no downloads, no API calls)

---

## Demo




![UI](images/image_1.png)

![UI](images/image_2.png)


---

## Architecture

**Frontend (Streamlit)**
- `streamlit_app.py` — UI + chat view + session handling
- `sidebar.py` — upload/index UI, document list, model selector
- `api_utils.py` — HTTP client to backend

**Backend (FastAPI)**
- `main.py` — API endpoints (`/chat`, `/upload-doc`, `/list-docs`, `/delete-doc`)
- `langchain_utils.py` — docs-first chain + web fallback (SerpAPI)
- `chroma_utils.py` — loaders, chunking, indexing, Chroma persistence
- `db_utils.py` — SQLite logs + document store

**Storage**
- `rag_app.db` — SQLite database for chat logs & document records
- `./chroma_db` — Chroma persistence directory

---

## Repo Structure

```text
.
├── api_utils.py
├── chat_interface.py
├── chroma_utils.py
├── db_utils.py
├── langchain_utils.py
├── main.py
├── pydantic_models.py
├── requirements.txt
├── sidebar.py
└── streamlit_app.py
```

---

## Requirements

- Python **3.10+** recommended
- Works on Windows/macOS/Linux

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Environment Variables

Your code loads `.env` from **one directory above** the backend files:

```py
load_dotenv(dotenv_path=os.path.join(os.path.dirname(__file__), '..', '.env'))
```

So place `.env` in the parent folder (one level above your backend code folder), or adjust the path.

### Required
- `OPENROUTER_API_KEY` — used by LangChain `ChatOpenAI` via OpenRouter

### Optional (for web search)
- `SERPAPI_API_KEY` — used by `SerpAPIWrapper`

Example `.env`:

```env
OPENROUTER_API_KEY=your_openrouter_key_here
SERPAPI_API_KEY=your_serpapi_key_here
```

---

## Run Locally

### 1) Start the backend (FastAPI)

```bash
python main.py
```

Backend runs on:

- `http://127.0.0.1:8000`

### 2) Start the frontend (Streamlit)

```bash
streamlit run streamlit_app.py
```

Frontend runs on:

- `http://localhost:8501`

---

## How It Works (RAG Flow)

1. Upload a document in the sidebar → `/upload-doc`
2. Backend:
   - Loads document (PDF/DOCX/HTML)
   - Splits into chunks using `RecursiveCharacterTextSplitter`
   - Embeds chunks using **SimpleHashEmbeddings** (offline demo)
   - Stores vectors in **Chroma**
3. When you ask a question:
   - `document_search()` retrieves top chunks (`k=2`)
   - LLM is prompted to answer strictly from retrieved extract
   - If not found and question looks “live/current”, SerpAPI web search is used
   - Otherwise it falls back to a general LLM response

---

## API Endpoints

### `POST /chat`
Ask a question.

Request:
```json
{
  "question": "What is this document about?",
  "model": "nvidia/nemotron-nano-9b-v2:free",
  "session_id": "optional-session-id"
}
```

Response:
```json
{
  "answer": "...",
  "session_id": "...",
  "model": "nvidia/nemotron-nano-9b-v2:free",
  "source": "document | web | llm | greeting"
}
```

> Note: Your backend currently returns `source` (see `pydantic_models.py`). If you remove that field, update the response schema here.

### `POST /upload-doc`
Upload and index a file (`multipart/form-data`).

### `GET /list-docs`
List indexed documents.

### `POST /delete-doc`
Delete a document by `file_id`.

---

## Notes & Troubleshooting

### 1) OpenRouter “429 Rate limit exceeded”
If you are using free models, you can hit daily request limits. In that case the backend logs will show something like:

- `Error code: 429 - Rate limit exceeded: free-models-per-day`

Fixes:
- Add credits to OpenRouter
- Switch models / use a paid tier
- Reduce calls (increase chunk quality, cache, etc.)

### 2) SerpAPI not working
If web search returns errors:
- confirm `SERPAPI_API_KEY` exists in `.env`
- verify your SerpAPI plan allows requests

### 3) “Backend offline” in the sidebar
- Ensure FastAPI is running (`python main.py`)
- Ensure `API_URL` in `api_utils.py` matches backend host/port (`127.0.0.1:8000`)

---

## Why `SimpleHashEmbeddings`?

This project intentionally includes an **offline embeddings** option to avoid:
- network/SSL issues
- model downloads
- API costs

It’s great for demos and learning. For best retrieval quality, swap it with a real embeddings model later.

---

## Roadmap (Ideas)

- Add **citations** (show chunk sources + file names)
- Add **streaming** responses in Streamlit
- Add **per-document filters** in retrieval
- Add **Docker** support
- Replace hash embeddings with **SentenceTransformers** or OpenAI embeddings

---

## Contact

- Email: **khawajaabdulrehman393@gmail.com**
- GitHub: **AbdulRehman3939** (https://github.com/AbdulRehman3939)

---

## License

Choose a license (MIT is common). Add a `LICENSE` file when you’re ready.

---

## Author

Built by **Abdul Rehman**
