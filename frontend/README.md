# Zenith Frontend

The Zenith frontend is a React + TypeScript single-page application built with Vite.

## Main areas

- `components/layout/PageShell.tsx` — active application shell and view transitions
- `pages/CommandCenter.tsx` — dashboard, triage, announcements, chat and study view
- `components/graph/KnowledgeGraph.tsx` — 3D knowledge graph
- `pages/Calendar.tsx` — seeded assignment calendar
- `pages/Profile.tsx` — sample student profile
- `components/study/StudyLabPane.tsx` — flashcard, quiz and study-guide generation
- `store/useAppStore.ts` — primary Zustand store
- `lib/api.ts` — backend API client

## Development

```bash
npm install
npm run dev
```

## Scripts

- `npm run dev` — Vite development server
- `npm run build` — TypeScript build and Vite production build
- `npm run lint` — ESLint
- `npm run preview` — preview the production build

## Backend connection

The current API client uses `http://localhost:8000`. The voice hook has its own hard-coded localhost origin as well. These should be moved to a shared environment-based configuration before deploying the frontend.

## State and data

The frontend loads seeded data from the FastAPI backend and stores it in Zustand. Chat history, generated artifacts, and mastery changes are currently in-memory. There is no authenticated account or durable browser/backend persistence.

See the repository-level [README](../README.md), [Architecture](../docs/ARCHITECTURE.md), [API reference](../docs/API_REFERENCE.md), [Development guide](../docs/DEVELOPMENT.md), and [Audit](../docs/AUDIT.md).
