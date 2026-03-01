# RAG Builder v2 — Full MVP

**Stack**: Node.js/Express · React · React Flow · Monaco Editor · Claude AI

---

## What's inside

```
rag-builder-v2/
├── backend/
│   ├── server.js          # Express API — AI generator + code templates
│   ├── package.json
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── App.jsx        # Main UI: prompt, tabs, panels
│   │   └── RAGNode.jsx    # Custom React Flow node component
│   ├── index.html
│   ├── vite.config.js
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── .env.example
```

---

## Features

| Feature | Implementation |
|---------|---------------|
| **AI Workflow Generator** | Calls `claude-sonnet-4-6` to generate custom node graphs from a prompt |
| **React Flow Node Editor** | Drag nodes, connect them, edit params inline, delete nodes |
| **Monaco Code Viewer** | Full VS Code editor — `server.js`, `package.json`, `Dockerfile`, `.env` |
| **Embed Panel** | iframe snippet + deployment guide |
| **Fallback Template** | If Anthropic key missing, uses built-in PDF chatbot template |
| **Stream-ready Generated App** | Generated app supports SSE streaming responses |

---

## Quick Start

### 1. Clone & configure
```bash
cp .env.example .env
# Add your ANTHROPIC_API_KEY to .env
```

### 2. Run with Docker Compose
```bash
docker-compose up
```

- Frontend: http://localhost:5173
- Backend:  http://localhost:8000

### 3. Or run manually

**Backend:**
```bash
cd backend
cp ../.env.example .env   # add your ANTHROPIC_API_KEY
npm install
npm run dev
```

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

---

## API

### `POST /generate`
```json
{
  "prompt": "PDF Q&A chatbot for my documentation",
  "openai_api_key": "sk-...",      // optional — injected into generated code
  "pinecone_api_key": "pcsk-..."   // optional — injected into generated code
}
```

**Response:**
```json
{
  "appId": "abc12345",
  "aiUsed": true,
  "workflow": {
    "name": "PDF Documentation Assistant",
    "description": "...",
    "nodes": [...],
    "edges": [...]
  },
  "code": {
    "serverJs": "...",
    "packageJson": "...",
    "dockerfile": "...",
    "envExample": "..."
  },
  "embedUrl": "http://localhost:3001",
  "embedCode": "<iframe src=\"...\" ...></iframe>",
  "status": "code_ready"
}
```

---

## How the AI generator works

1. User prompt is sent to `claude-sonnet-4-6` with a system prompt that defines the JSON schema
2. Claude returns a workflow JSON with nodes, edges, positions, colors, icons, and params
3. If parsing fails (or no Anthropic key), falls back to the built-in PDF chatbot template
4. Code generator (`generateCode()`) templates a full Node.js app from the workflow metadata

---

## Generated App Architecture

The generated `server.js` is a production-ready Express app that:
- Accepts PDF uploads via `multer`
- Extracts text with `pdf-parse`
- Chunks with LangChain's `RecursiveCharacterTextSplitter`
- Embeds with OpenAI `text-embedding-3-small`
- Stores in Pinecone (auto-creates index)
- Retrieves top-K chunks + generates answers with GPT-4o
- Streams responses via SSE
- Maintains per-session conversation history

---

## Roadmap

- [ ] Multi-app hosting (per-user subdomains)
- [ ] Docker build + health-check pipeline in backend
- [ ] Node library palette (drag from sidebar)
- [ ] Workflow → code sync (edit code → update nodes)
- [ ] Export to LangChain / LlamaIndex format
