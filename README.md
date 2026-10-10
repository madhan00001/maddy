# Maddy AI - Your Intelligent AI Assistant

A modern, production-grade AI chatbot application inspired by ChatGPT, powered by the official **Google Gemini API** (`@google/genai`) as the LLM backend with full conversation memory, streaming responses, code syntax highlighting, and **RAG (Retrieval-Augmented Generation) knowledge base vector search**.

---

## 🌟 Key Features

* **Google Gemini API Powered**: Native server-side integration using `@google/genai` TypeScript SDK with resilient automatic model cascade on rate limits (`gemini-3.8-flash` ➔ `gemini-3.1-flash-lite` ➔ `gemini-flash-latest`).
* **RAG Knowledge Base & Vector DB**:
  * Semantic embeddings with `gemini-embedding-001` (or `gemini-embedding-2-preview`).
  * Configurable chunking with `CHUNK_SIZE = 1000` and `CHUNK_OVERLAP = 200`.
  * Top-K semantic retrieval with `TOP_K = 5` and cosine similarity matching.
  * Vector storage persisted to disk at `./vector_db`.
  * Ingest text files, markdown, notes, and documents.
  * Grounded answers display an expandable citation badge with similarity match scores.
* **Real-Time Streaming**: Smooth Server-Sent Events (SSE) streaming with live cursor and typing indicators.
* **Chat Memory & Persistence**: Multi-turn conversation context preserved across sessions with an SQLite database file (`./chatbot.db`).
* **Instant Stop & Regenerate**: Stop ongoing responses at any moment, or regenerate any previous assistant response with one click.
* **Message Editing**: Edit previous user messages and regenerate from that conversation branch.
* **Automated Chat Titles**: Automatic 3–6 word conversation title generation after the first message.
* **Rich Markdown & Code Highlighting**: Full Markdown support (tables, lists, quotes, bold, links) plus syntax-highlighted code blocks with language indicators and one-click copy.
* **Customizable System Instructions**: Fine-tune Maddy AI's persona, goals, and technical explanation style from the settings modal.
* **Model & Parameter Tuning**: Switch between `gemini-3.8-flash` and `gemini-3.1-flash-lite`, and tune temperature from 0.0 to 1.0.
* **Responsive Design**: Collapsible sidebar on mobile, responsive layout on desktop, with both Dark and Light theme modes.
* **Production Security**: Strict server-side API key isolation (`process.env.GEMINI_API_KEY`), request payload sanitization, and no client-side key leakage.

---

## ⚙️ Configuration Parameters

```env
# Gemini API Key (Required)
GEMINI_API_KEY=your_gemini_api_key

# LLM Configuration
LLM_PROVIDER=gemini
LLM_MODEL=gemini-3.8-flash

# Embeddings & Vector Database
EMBEDDING_PROVIDER=gemini
EMBEDDING_MODEL=gemini-embedding-001

CHUNK_SIZE=1000
CHUNK_OVERLAP=200
TOP_K=5

VECTOR_DB_PATH=./vector_db
DATABASE_URL=sqlite:///./chatbot.db
JWT_SECRET=change-this-to-a-random-secret-key
```

---

## 🏗️ Project Architecture

```text
maddy-ai/
├── server.ts                  # Express server mounting APIs and Vite in dev / static in prod
├── server/
│   ├── config/
│   │   └── aiConfig.ts        # Centralized system prompt, model configs, RAG defaults
│   ├── database/
│   │   └── db.ts              # SQLite database adapter (sql.js) with disk persistence
│   ├── services/
│   │   ├── gemini.ts          # Server-side @google/genai client & streaming generator
│   │   └── vectorStore.ts     # Gemini embeddings, text chunking, and cosine similarity
│   └── routes/
│       ├── chat.ts            # POST /api/chat (SSE streaming with RAG)
│       ├── documents.ts       # Document ingestion & vector search endpoints
│       └── conversations.ts   # CRUD routes for conversations & messages
├── src/
│   ├── components/
│   │   ├── CodeBlock.tsx          # Syntax-highlighted code block with copy button
│   │   ├── Header.tsx             # Header with model badge, Knowledge Base button, settings
│   │   ├── KnowledgeBaseModal.tsx # Document upload, chunk viewer, semantic search tester
│   │   ├── MessageInput.tsx       # Auto-resizing textarea with RAG toggle, Enter to send & Stop
│   │   ├── MessageItem.tsx        # Message item with Markdown and RAG sources badge
│   │   ├── SettingsModal.tsx      # Modal for theme, temperature, model, custom prompt
│   │   ├── Sidebar.tsx            # Conversation history, search, Knowledge Base button
│   │   └── WelcomeScreen.tsx      # Starter suggestions (explain concept, write code, etc.)
│   ├── hooks/
│   │   └── useChat.ts         # Chat state machine, conversation manager, stream handler
│   ├── services/
│   │   └── api.ts             # Typed API client with SSE parser & document APIs
│   ├── types/
│   │   └── chat.ts            # Shared TypeScript interfaces
│   ├── App.tsx                # Main app layout
│   ├── index.css              # Tailwind CSS styling and animations
│   └── main.tsx               # React client entry point
├── Dockerfile                 # Multi-stage production container build
├── .dockerignore              # Excluded files for clean Docker builds
├── .env.example               # Environment variables template
├── metadata.json              # Applet metadata
├── package.json               # Node.js dependencies and scripts
└── tsconfig.json              # TypeScript configuration
```

---

## 🚀 Step-by-Step Deployment Guide

Deploying Maddy AI is straightforward because it is designed as a unified fullstack application (Express + React in a single container or service).

### Prerequisites
1. A **Google Gemini API Key**: Obtain one from [Google AI Studio](https://aistudio.google.com/).
2. A **GitHub Repository**: Push your Maddy AI code to a repository on GitHub.

---

### Option 1: Deploying on Render (Recommended & Beginner Friendly)

1. **Sign Up / Log In**: Go to [Render](https://render.com/) and connect your GitHub account.
2. **Create New Web Service**: Click **New +** ➔ **Web Service** ➔ Select your `maddy-ai` GitHub repository.
3. **Configure Build Settings**:
   * **Name**: `maddy-ai`
   * **Language / Environment**: `Node`
   * **Region**: Choose the closest region (e.g., Oregon, Frankfurt, Singapore)
   * **Branch**: `main`
   * **Build Command**: `npm install && npm run build`
   * **Start Command**: `npm start`
4. **Configure Environment Variables**:
   Click **Advanced** ➔ **Add Environment Variable**:
   * `GEMINI_API_KEY`: `<Your Gemini API Key>`
   * `NODE_ENV`: `production`
   * `PORT`: `10000` (or leave default, Render sets this automatically)
5. **Add Persistent Disk (Optional for SQLite persistence across restarts)**:
   * In Render service settings, go to **Disks** ➔ **Add Disk**.
   * Mount path: `/app/vector_db`
6. **Deploy**: Click **Create Web Service**. Render will automatically build the React assets, start the Express backend, and provide an `https://maddy-ai-xxxx.onrender.com` URL with free SSL!

---

### Option 2: Deploying on Railway

1. Go to [Railway.app](https://railway.app/) and click **Start a New Project**.
2. Select **Deploy from GitHub repo** and pick `maddy-ai`.
3. In the Railway project canvas, click on your service and go to the **Variables** tab.
4. Add:
   * `GEMINI_API_KEY`: `<Your Gemini API Key>`
   * `NODE_ENV`: `production`
5. Railway will automatically detect Node.js, run `npm install && npm run build`, and execute `npm start`.
6. Go to **Settings** ➔ **Networking** ➔ Click **Generate Domain** to get your public URL.

---

### Option 3: Deploying with Docker (Any VPS / DigitalOcean / AWS EC2)

1. **Build Docker image**:
   ```bash
   docker build -t maddy-ai .
   ```
2. **Run container**:
   ```bash
   docker run -d \
     -p 3000:3000 \
     -e GEMINI_API_KEY="your_gemini_api_key_here" \
     -v maddy_data:/app/vector_db \
     --name maddy-ai \
     maddy-ai
   ```
3. Visit `http://<your-server-ip>:3000`.

---

### Option 4: Deploying on Google Cloud Run

1. Make sure you have the Google Cloud CLI installed (`gcloud`).
2. Run the build and deploy command:
   ```bash
   gcloud run deploy maddy-ai \
     --source . \
     --platform managed \
     --region us-central1 \
     --allow-unauthenticated \
     --set-env-vars GEMINI_API_KEY="your_api_key_here"
   ```
3. Cloud Run will build the container with Google Cloud Build and deploy it to a serverless autoscaling URL with HTTPS.

---

## 🛠️ Troubleshooting Guide

| Issue | Cause | Solution |
| :--- | :--- | :--- |
| **"Rate limit reached (429)"** | Exceeded Gemini free-tier quota limits on one model | Maddy AI automatically falls back to `gemini-3.1-flash-lite`. Ensure your API key has active quota. |
| **"Gemini API key is not configured"** | Missing `GEMINI_API_KEY` in environment variables | Ensure `.env` contains `GEMINI_API_KEY="your_key"` and restart the server. |
| **Responses do not stream** | Reverse proxy buffering SSE | Ensure `X-Accel-Buffering: no` is allowed (configured automatically in `server/routes/chat.ts`). |
| **Vector search returns 0 results** | No documents uploaded or similarity too low | Upload documents via the **Knowledge Base** button in the header or sidebar. |

---

## 📄 License

Apache-2.0
