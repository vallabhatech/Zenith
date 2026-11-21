# Zenith API Reference

All backend endpoints are registered under the `/api` prefix. The local development server is configured for `127.0.0.1:8000`.

This reference lists routes found in the current FastAPI router declarations. It is not a guarantee that every route passes runtime tests.

## Health

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | Basic application/service health response |

The health response is a lightweight application check. It should not be interpreted as a comprehensive check of every external dependency.

## Seeded academic data

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/courses` | Return seeded courses |
| GET | `/api/assignments` | Return seeded assignments |
| GET | `/api/notes` | Return seeded notes |
| GET | `/api/announcements` | Return seeded announcements |
| GET | `/api/concepts` | Return seeded knowledge-graph concepts |
| GET | `/api/connections` | Return seeded concept connections |
| GET | `/api/syllabi` | Return seeded course syllabi |

These endpoints read Python data modules. They do not query a student account or Canvas.

## Chat

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/chat` | Stream an AI response using Server-Sent Events |

The request schema is `ChatRequest` in `backend/models/schemas.py`. The frontend currently sends:

- `messages`: conversation entries with `role` and `content`
- `course_context`: optional course ID
- `triage_statuses`: optional assignment-status map
- `system_status`: optional dashboard status object
- `active_remediation`: optional remediation context
- `upcoming_deadlines`: optional deadline summary
- `graph_context`: optional concept/mastery summary
- `graph_connections`: optional graph-relationship summary
- `agent_url`: optional specialized-agent URL

The response is an SSE stream. The frontend expects JSON payloads in `data:` lines and appends the `content` property when present.

## Study-material generation

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/generate/flashcards` | Generate structured flashcards |
| POST | `/api/generate/quiz` | Generate multiple-choice questions |
| POST | `/api/generate/study-guide` | Generate a Markdown study guide |

The shared request shape includes a `course_id` and optional `topic`. Response models are defined in `backend/models/schemas.py`.

## Document parsing

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/parse` | Accept a document upload and return extracted text/concepts |

The current parser implementation uses the backend parsing service. Upload-size enforcement and file-content validation require additional hardening; see [Audit findings](AUDIT.md).

## Knowledge graph

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/graph/initial` | Return the initial graph payload |
| POST | `/api/graph/extract` | Extract concept nodes and relationships from supplied text |

The graph extraction request/response models are defined in `backend/models/schemas.py`. The initial-graph route currently has a field-name mismatch with the seeded data and should be fixed before clients rely on it.

## Knowledge context and course recommendations

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/knowledge/build-context` | Build topic-focused context and prepare a TerpAI agent/context |
| POST | `/api/knowledge/refresh-from-graph` | Build and synchronize context from graph state |
| POST | `/api/knowledge/recommend-courses` | Recommend next courses using UMD catalog information and AI analysis |

These endpoints depend on configured AI services and TerpAI integration state. The graph-refresh endpoint currently needs stronger success/failure reporting.

## Mastery

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/mastery/update` | Update a concept's in-memory mastery value |
| GET | `/api/mastery/{concept_id}` | Read one concept's mastery value |
| GET | `/api/mastery` | Read the mastery map |

Backend mastery state is in memory and resets when the process restarts. The active quiz flow also updates frontend Zustand state locally; it is not currently a durable synchronized record.

## UMD.io catalog proxy

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/umd/courses` | Fetch courses for a department |
| GET | `/api/umd/courses/{course_id}` | Fetch one course |
| GET | `/api/umd/courses/{course_id}/sections` | Fetch sections for a course |
| GET | `/api/umd/departments` | Fetch departments |
| GET | `/api/umd/terms` | Fetch academic terms |

The underlying integration uses `https://api.umd.io/v1`.

## Voice

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/voice` | Generate speech from text |
| POST | `/api/transcribe` | Transcribe an uploaded audio file |

The voice router delegates to the configured speech provider. The active Command Center uses the frontend TTS hook; the microphone control in that view is not yet wired to the transcription endpoint.

## Error handling and operational notes

- Most routes rely on external API availability and local environment variables.
- Some integration services log errors and return empty collections rather than propagating upstream status codes.
- Chat error SSE events are not currently rendered as explicit errors by the frontend client.
- There is no authentication or per-user authorization layer.
- Do not expose this API publicly until authentication, request limits, upload limits, CORS configuration, TLS verification, and error handling have been reviewed.
