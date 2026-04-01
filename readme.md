# RAG Chat — Italian Legislative Document Assistant

A Retrieval-Augmented Generation (RAG) chat application for querying Italian legislative and governmental documents. The system retrieves relevant context from a vector store and uses an LLM to generate accurate, grounded answers streamed in real time.

## Architecture

```
React Frontend (port 3000)
    │
    │  SSE (Server-Sent Events)
    ▼
FastAPI Backend (port 8000)
    │
    ├── LangChain Conversational Retrieval Chain
    │       │
    │       ├── Ollama LLM (Llama)
    │       └── Ollama Embeddings (nomic-embed-text)
    │
    └── ChromaDB Vector Store
```

**Frontend** — React 18 chat interface with Material UI, Tailwind CSS, markdown rendering, suggested questions, and voice input support.

**Backend** — FastAPI server that orchestrates the RAG pipeline: matches user-selected settings to a configuration, builds an LLM + retriever chain via LangChain, and streams tokens back as Server-Sent Events.

## Features

- Streaming chat responses via SSE
- Configurable LLM and vector store combinations (`backend/config.json`)
- Two retriever modes: `conversation` (direct retrieval) and `condense` (refines answers with source references)
- Voice input with Whisper transcription
- Dark mode toggle
- Markdown-formatted responses with GFM support

## Project Structure

```
RAG/
├── backend/
│   ├── main.py                          # FastAPI app and endpoints
│   ├── config.json                      # Available LLM/embedding/vectorstore presets
│   ├── Dockerfile
│   └── src/
│       ├── chat/chat.py                 # RAG orchestration and streaming
│       ├── config/settings.py           # Paths and service endpoints
│       ├── models/llm_factory.py        # LLM construction (Ollama)
│       ├── rag_models/
│       │   ├── embedding/               # Embedding factory (nomic-embed-text)
│       │   ├── vectorestore/            # Vector store factory (ChromaDB)
│       │   └── retriever/               # Retrieval chain factory
│       └── utils/
│           ├── prompts/prompt.py        # System prompt template
│           └── embedder_utils/          # Embedding normalization helper
├── frontend/
│   ├── package.json
│   ├── Dockerfile
│   ├── .env.example
│   └── src/
│       ├── App.jsx                      # Main application
│       ├── DarkModeContext.jsx           # Dark mode state
│       └── components/
│           ├── Layouts.jsx              # Chat layout and input
│           ├── Messages.jsx             # Message rendering
│           ├── FormattedText.jsx         # Markdown formatting
│           ├── SuggestedQuestions.jsx    # Predefined question chips
│           └── PermissionAlert.jsx      # Microphone permission dialog
├── docker-compose.yml
├── requirements.txt                     # Python dependencies
└── readme.md
```

## Prerequisites

- **Python 3.10+**
- **Node.js 18+** and npm
- **Ollama** running locally with the following models pulled:
  ```bash
  ollama pull llama3.2
  ollama pull nomic-embed-text
  ```
- **ChromaDB** vector store pre-populated with your document embeddings
- **FFmpeg** (required only for voice input / audio processing)

## Getting Started

### 1. Clone the Repository

```bash
git clone <your-repo-url>
cd RAG
```

### 2. Backend Setup

Create and activate a virtual environment:

```bash
python3 -m venv env
source env/bin/activate        # macOS / Linux
# .\env\Scripts\activate       # Windows
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Configure `backend/src/config/settings.py` with your paths:

```python
CHROMA_DIR = "/path/to/your/chroma_langchain_db"
OLLAMA_LLM_ENDPOINT = "http://localhost:11434/"
OLLAMA_EMBEDDINGS_ENDPOINT = "http://localhost:11434/"
```

Start the backend (from the `backend` directory):

```bash
cd backend
uvicorn main:app --reload
```

The API will be available at `http://localhost:8000`.

### 3. Frontend Setup

```bash
cd frontend
cp .env.example .env
# Edit .env and set REACT_APP_BASE_URL=http://localhost:8000
npm install
npm start
```

The UI will be available at `http://localhost:3000`.

### 4. Docker (Alternative)

Make sure to update the Chroma volume path in `docker-compose.yml` to match your local setup:

```yaml
volumes:
  - /path/to/your/chroma_langchain_db:/app/chroma_langchain_db
```

Create environment files:
- `backend/.env` — backend environment variables
- `frontend/.env` — set `REACT_APP_BASE_URL`

Then run:

```bash
docker compose up --build
```

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/chat/` | Send a message and receive a streamed response |
| `POST` | `/process_audio/` | Upload audio for transcription and streamed response |

### POST `/chat/`

**Request body:**

```json
{
  "content": "Quali sono i requisiti per il permesso di soggiorno?",
  "selectedLLM": "llama + nomic-embed-text",
  "vectordb": "Chroma",
  "retrieverType": "conversation"
}
```

**Response:** Server-Sent Events stream with `{ "response": "..." }` chunks.

## Configuration

The backend uses `backend/config.json` to define available LLM + embedding + vector store combinations. Each entry specifies:

```json
{
  "selected_approach": "llama + nomic-embed-text",
  "llm_name": "llama",
  "embedding_name": "nomic-embed-text",
  "vectore_store_type": "Chroma",
  "collection_name": "turismo_metadata"
}
```

The frontend lets users select from these presets in the UI.

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| Frontend | React 18, Material UI, Tailwind CSS, react-markdown |
| Backend | Python, FastAPI, Uvicorn |
| RAG Pipeline | LangChain, Ollama, ChromaDB |
| LLM | Llama 3.2 (via Ollama) |
| Embeddings | nomic-embed-text (via Ollama) |
| Speech-to-Text | OpenAI Whisper, FFmpeg |
| Infrastructure | Docker, Docker Compose |

## Testing Ollama Connection

Verify Ollama is running and models are available:

```bash
# Test LLM
curl -X POST http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{"model": "llama3.2", "prompt": "Hello", "stream": false}'

# Test Embeddings
curl -X POST http://localhost:11434/api/embeddings \
  -H "Content-Type: application/json" \
  -d '{"model": "nomic-embed-text", "input": "test"}'
```
