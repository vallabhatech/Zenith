# Development Guide

## Repository conventions

- Frontend source is under `frontend/src`.
- Backend source is under `backend/`.
- Backend routers are mounted under `/api` by `backend/main.py`.
- Pydantic API schemas live in `backend/models/schemas.py`.
- Pydantic seeded-data schemas live in `backend/models/data_models.py`.
- Seed data lives in `backend/data/`.
- Prompt builders live in `backend/prompts/`.
- External-service adapters live in `backend/services/`.

## Local setup

### Backend

```bash
cd backend
python -m venv .venv
# Activate the virtual environment for your operating system.
pip install -r requirements.txt
# Copy .env.example to .env and set credentials locally.
python run.py
```

The backend development entry point binds to `127.0.0.1:8000`.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend API client currently uses a hard-coded `http://localhost:8000` base URL. A configurable `VITE_API_BASE_URL` should be introduced before deployment.

## Environment variables

Current template variables:

- `GEMINI_API_KEY`
- `ELEVENLABS_API_KEY`
- `TERPAI_AGENT_NAME`
- `TERPAI_AGENT_URL`
- `TERPAI_TOKEN`

The TerpAI service also reads `TERPAI_CONFIG_STATE_ID`, which is not currently documented in `.env.example`.

Keep secrets in a local `.env` file or a deployment secret manager. Do not paste credentials into source files, issue comments, logs, or commits. Rotate any credential that has been committed or shared unintentionally.

## Useful commands

### Frontend

```bash
npm run dev
npm run build
npm run lint
npm run preview
```

### Backend data validation

```bash
cd backend
python -m data.validate
```

The current validator is expected to fail against the seeded concept data until the required description field and related graph data are reconciled.

## Verification checklist

Before merging a code change, verify:

1. Backend imports and application startup succeed.
2. `GET /api/health` returns a healthy response.
3. All seeded-data endpoints return the expected shape.
4. `python -m data.validate` passes.
5. Frontend `npm run build` succeeds.
6. Frontend `npm run lint` succeeds.
7. Chat streams content and displays provider errors.
8. Flashcards, quizzes, and study guides generate and render.
9. Graph initial-data and extraction endpoints work with real seeded records.
10. Voice generation and transcription work with configured credentials.
11. No UI control advertises an action that is not wired.
12. Production configuration uses HTTPS, verified TLS, restricted CORS, authentication, and request/upload limits.

## Test status in this review

This review was performed through repository source inspection. No local checkout or executable runtime was available, so the commands above were not run. The repository currently has no GitHub Actions workflow runs and no reported commit status checks. The legacy Playwright fallback contains a manual browser-inspection script, not a maintained automated test suite.

## Deployment blockers

Do not deploy the current configuration as a public multi-user service until the following are addressed:

- Replace hard-coded localhost API URLs with environment-based configuration.
- Configure CORS for explicit production origins.
- Restore TLS certificate verification in the TerpAI HTTP client.
- Add authentication and authorization.
- Add rate limits and request-size limits, especially for file and audio uploads.
- Add durable storage and per-user data isolation.
- Replace static sample identity and academic data with an explicit demo mode or real authenticated data source.
- Add automated tests and CI.
