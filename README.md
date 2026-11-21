# Zenith

**Zenith** is an AI-assisted academic workspace prototype. It brings course information, assignments, announcements, a visual knowledge graph, AI study materials, and conversational study support into one interface.

> **Project status:** Prototype / active engineering work. The current repository uses seeded University of Maryland (UMD) sample data. It is not yet a production-ready, multi-user academic platform.

## What it contains

- **Command Center** — an academic dashboard with assignment triage, announcements, a morning brief, and an AI chat interface.
- **Neural Lattice** — a 3D knowledge graph of concepts and prerequisite relationships, with course, mastery, and view filters.
- **Study Lab** — AI-generated flashcards, multiple-choice quizzes, and study guides.
- **Concept Inspector** — concept context, connected concepts, generated study artifacts, and knowledge-context generation.
- **Calendar** — a calendar view over the seeded assignment data.
- **Student Profile** — a profile and academic-history demonstration using static sample identity fields.
- **AI services** — TerpAI chat integration, Gemini-based context generation/fallbacks, and ElevenLabs voice services.
- **Document parsing** — PDF/text extraction for concept processing.
- **UMD catalog integration** — course, department, term, and section lookups through the public UMD.io API.

Some visible controls and service indicators are currently demonstrations or disconnected paths. See [the audit](docs/AUDIT.md) before treating any feature as production-ready.

## Technology

| Layer | Technologies |
|---|---|
| Frontend | React, TypeScript, Vite |
| UI and styling | Tailwind CSS, Radix UI, shadcn-style components, Lucide, Framer Motion |
| State | Zustand |
| Visualization | react-force-graph-3d, Three.js |
| Backend | Python, FastAPI, Pydantic |
| HTTP / integrations | httpx, UMD.io |
| AI | TerpAI bridge, Google Gemini |
| Voice | ElevenLabs, browser speech APIs |
| Document extraction | pdfplumber |

## Architecture at a glance

```text
Browser
  └─ React + TypeScript SPA
      ├─ Zustand application state
      ├─ Command Center / Calendar / Profile
      ├─ 3D Knowledge Graph
      └─ Study Lab / Chat / Voice UI
             │ HTTP + Server-Sent Events
             ▼
      FastAPI application (/api)
        ├─ Static sample-data routers
        ├─ Chat and generation routers
        ├─ Graph and knowledge routers
        ├─ Document parsing router
        ├─ Voice router
        ├─ UMD catalog router
        └─ In-memory mastery router
             │
             ├─ TerpAI HTTP bridge
             ├─ Gemini service
             ├─ ElevenLabs service
             ├─ UMD.io API
             └─ pdfplumber
```

For the detailed request flows and boundaries, see [Architecture](docs/ARCHITECTURE.md).

## Repository layout

```text
.
├── backend/
│   ├── data/                  # Seeded course, assignment, note, graph and syllabus data
│   ├── models/                # Pydantic request/response and data models
│   ├── prompts/               # Prompts for chat, graph analysis and study artifacts
│   ├── routers/               # FastAPI endpoints
│   ├── services/              # TerpAI, Gemini, UMD, parsing and voice integrations
│   ├── .env.example           # Environment-variable template
│   ├── main.py                # FastAPI app and router registration
│   └── run.py                 # Local development server entry point
├── frontend/
│   ├── public/
│   └── src/
│       ├── components/        # Dashboard, graph, chat, study, voice and UI components
│       ├── hooks/             # API loading, graph filtering, chat and voice hooks
│       ├── lib/               # API client, graph layout and voice controller
│       ├── pages/             # Command Center, Calendar, Profile and Chat
│       ├── store/             # Zustand stores
│       └── types/             # Shared frontend types
└── docs/
    ├── API_REFERENCE.md
    ├── ARCHITECTURE.md
    ├── AUDIT.md
    └── DEVELOPMENT.md
```

## Local development

### Prerequisites

- Python 3.10 or newer
- Node.js and npm compatible with the Vite version in `frontend/package.json`
- API credentials for whichever external services you intend to exercise

### 1. Backend

From the repository root:

```bash
cd backend
python -m venv .venv
```

Activate the environment:

```bash
# Windows PowerShell
.venv\\Scripts\\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install dependencies and configure environment variables:

```bash
pip install -r requirements.txt
# Copy .env.example to .env, then add credentials locally.
```

Start the API:

```bash
python run.py
```

The development server is configured for `127.0.0.1:8000`. The health endpoint is `http://127.0.0.1:8000/api/health`.

### 2. Frontend

In a second terminal:

```bash
cd frontend
npm install
npm run dev
```

Vite's default development URL is normally `http://localhost:5173`.

### 3. Available frontend scripts

Run these from `frontend/`:

| Command | Purpose |
|---|---|
| `npm run dev` | Start Vite development server |
| `npm run build` | Run TypeScript project build, then create the Vite production bundle |
| `npm run lint` | Run ESLint |
| `npm run preview` | Preview a built frontend |

The backend currently has a data-validation script:

```bash
cd backend
python -m data.validate
```

The current validation script is expected to report issues because the seeded concept records do not satisfy the declared Pydantic model. See [Audit findings](docs/AUDIT.md). These commands are documented test entry points; they have **not** been executed in this repository review environment.

## Environment configuration

The backend template currently defines:

| Variable | Purpose |
|---|---|
| `GEMINI_API_KEY` | Gemini context generation and fallback responses |
| `ELEVENLABS_API_KEY` | ElevenLabs speech services |
| `TERPAI_AGENT_NAME` | TerpAI agent display name |
| `TERPAI_AGENT_URL` | TerpAI agent/chat URL |
| `TERPAI_TOKEN` | TerpAI bearer credential |

The code also references `TERPAI_CONFIG_STATE_ID` for knowledge-source updates, but this variable is absent from the checked-in `.env.example`. Add it to your local environment only if that integration path is being used. Never commit real API keys, bearer tokens, session cookies, or local `.env` files.

**Deployment warning:** the frontend API client currently targets `http://localhost:8000`, the backend CORS configuration is limited to the local Vite origin, and the TerpAI HTTP client currently disables TLS certificate verification. Production deployment requires configuration and security work first.

## Data and persistence model

The current application is a single-user demonstration built around Python modules in `backend/data/`. These modules supply seeded courses, assignments, notes, announcements, concepts, connections, and syllabi.

There is no database or authentication layer in the current implementation. Frontend Zustand state and backend mastery state are in memory; generated artifacts, chat history, and mastery changes are not durably persisted. The profile and several dashboard values are static sample content. The UMD.io integration reads course-catalog information; it is not a Canvas LMS integration.

## Documentation

- [Architecture and data flow](docs/ARCHITECTURE.md)
- [API reference](docs/API_REFERENCE.md)
- [Local development and configuration](docs/DEVELOPMENT.md)
- [Engineering audit and known issues](docs/AUDIT.md)

## Scope of this documentation

This documentation describes the repository as it exists on the documentation branch. It separates implemented code paths from mock data, disconnected UI paths, and items that still require runtime verification. No application code has been changed as part of this documentation pass.
