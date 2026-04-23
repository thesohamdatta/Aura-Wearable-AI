# AURA Backend

FastAPI backend that powers AURA's transcription, memory, and AI chat pipeline.

For the full module reference see [`INDEX.md`](INDEX.md).

---

## What it does

| Feature | How |
|:---|:---|
| Audio transcription | Deepgram real-time STT over WebSocket |
| Vision analysis | GPT-4o processes images from the pendant |
| Memory storage | Pinecone vector DB + Firestore |
| Chat | LangGraph agentic system with RAG |
| Auth | Firebase Authentication |
| Caching | Redis (Upstash recommended) |

---

## Prerequisites

| Tool | Install |
|:---|:---|
| Python 3.10+ | [python.org](https://www.python.org/downloads/) |
| Google Cloud SDK | [cloud.google.com/sdk](https://cloud.google.com/sdk/docs/install) |
| git + ffmpeg | Package manager or [ffmpeg.org](https://ffmpeg.org) |
| ngrok | [ngrok.com](https://ngrok.com) |

---

## Quick Setup

### 1 — Google Cloud / Firebase

You need a Google Cloud project with Firebase enabled.

```bash
gcloud auth login
gcloud config set project <your-project-id>
gcloud auth application-default login --project <your-project-id>
```

**Enable these APIs in [Google Cloud Console](https://console.cloud.google.com/apis/dashboard):**
- Cloud Resource Manager API
- Firebase Management API
- Cloud Firestore API

**Create Firestore composite indexes** ([Firebase Console](https://console.firebase.google.com/) → Firestore → Indexes):

| Collection | Fields |
|:---|:---|
| `dev_api_keys` | `user_id` (Ascending) + `created_at` (Descending) |
| `mcp_api_keys` | `user_id` (Ascending) + `created_at` (Descending) |

Without these, the Developer Settings page will fail.

---

### 2 — Environment File

```bash
cd Aura-Wearable-AI/backend

# Windows
copy .env.template .env

# macOS / Linux
cp .env.template .env
```

Fill in your keys. Minimum required:

```env
OPENAI_API_KEY=sk-...
DEEPGRAM_API_KEY=...
PINECONE_API_KEY=...
PINECONE_INDEX_NAME=aura-memories
REDIS_DB_HOST=...
REDIS_DB_PORT=...
REDIS_DB_PASSWORD=...
ADMIN_KEY=any-local-secret
GOOGLE_APPLICATION_CREDENTIALS=google-credentials.json
```

**Pinecone index:** create with dimension `1536` (OpenAI text-embedding-3-small).

---

### 3 — Install Dependencies

```bash
# Create virtual environment (recommended)
python -m venv venv

# Activate
# Windows:
venv\Scripts\activate
# macOS / Linux:
source venv/bin/activate

# Install
pip install -r requirements.txt
```

---

### 4 — Start Server

```bash
uvicorn main:app --reload --env-file .env --port 8000
```

Test it:

```bash
curl http://localhost:8000/health
```

---

### 5 — Expose via ngrok

The AURA pendant and app need a public HTTPS URL to reach the backend.

```bash
# One-time setup
ngrok config add-authtoken <your-token>

# Start tunnel (use your static domain if you have one)
ngrok http --domain=example.ngrok-free.app 8000

# Or simple:
ngrok http 8000
```

In the AURA app settings, set:

```
API_BASE_URL=https://your-ngrok-url.ngrok-free.app/
```

Include the trailing slash.

---

## API Keys Reference

| Service | Free tier? | Get key |
|:---|:---:|:---|
| Firebase / GCP | Yes | [console.firebase.google.com](https://console.firebase.google.com) |
| OpenAI | No | [platform.openai.com](https://platform.openai.com) |
| Deepgram | Yes ($200 credit) | [console.deepgram.com](https://console.deepgram.com) |
| Pinecone | Yes | [app.pinecone.io](https://app.pinecone.io) |
| Redis / Upstash | Yes | [console.upstash.com](https://console.upstash.com) |

---

## Module Structure

```
backend/
├── main.py               ← FastAPI app entry point
├── dependencies.py       ← shared dependencies & auth
├── requirements.txt
├── .env.template         ← copy to .env
│
├── routers/              ← API endpoints
│   ├── transcribe.py     ← audio WebSocket
│   ├── conversations.py  ← memory CRUD
│   ├── chat.py           ← AI chat
│   ├── memories.py       ← memory operations
│   └── developer.py      ← developer API
│
├── database/             ← data access layer
│   ├── conversations.py
│   ├── memories.py
│   ├── vector_db.py      ← Pinecone
│   └── redis_db.py
│
└── utils/
    ├── llm/              ← LLM clients + processing
    ├── stt/              ← speech-to-text (Deepgram)
    └── retrieval/        ← RAG + LangGraph agent
```

See [`INDEX.md`](INDEX.md) for full module reference.

---

## Troubleshooting

**`No internet connection` when loading models:**

Add to `utils/stt/vad.py` after imports:
```python
import ssl
ssl._create_default_https_context = ssl._create_unverified_context
```

**`Internal Server Error` on Developer Settings page:**

Firestore composite indexes are missing — see [Step 1](#1--google-cloud--firebase).

**`authentication errors` from gcloud:**

Re-run application-default login:
```bash
gcloud auth application-default login --project <your-project-id>
```

**Deactivate virtual environment when done:**
```bash
deactivate
```
