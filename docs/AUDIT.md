# Zenith Engineering Audit

**Review type:** Static source review  
**Reviewed repository:** `vallabhatech/Zenith`  
**Reviewed branch:** `main` at `83e685c91a7758cd35855b3ca9913a75727efd63`  
**Documentation branch:** `docs/zenith-project-documentation`  
**Runtime status:** Not executed in this review environment

## Executive summary

Zenith has a substantial frontend and backend prototype: a React command center, a Three.js knowledge graph, study-material generation, a FastAPI API, AI-provider adapters, voice endpoints, document parsing, and UMD catalog lookups.

The repository is **not yet verified as working end to end**. The current implementation contains data-contract mismatches, stale sample dates, disconnected UI paths, deployment configuration assumptions, and security hardening gaps. The README and frontend README were also stale and did not describe the current Zenith project.

This document intentionally distinguishes static findings from runtime-confirmed failures. No application code has been modified in this documentation pass.

## Findings

### Critical — address before relying on the affected feature

#### 1. Initial graph endpoint does not match the seeded data contract

- `backend/data/concepts.py` provides concept records without the `description` field required by `ConceptNode`.
- `backend/data/connections.py` uses `source_id` and `target_id`.
- `backend/routers/graph.py` constructs the initial graph using `description`, `source`, and `target` lookups.
- Result: `GET /api/graph/initial` is expected to fail when it processes the current seeded records.

The frontend's active graph path fetches concepts and connections separately and normalizes missing descriptions to an empty string, so this defect may not appear in the primary graph UI.

#### 2. Seed-data validator conflicts with concept schema

`backend/data/validate.py` validates each concept against `ConceptModel`. The current concept records do not include the required description field. Failed concept validation also prevents those concept IDs from being added to the validator's valid-ID set, which can produce cascading connection and syllabus reference errors.

The validator has not been executed here; this is a static contract mismatch.

#### 3. Default semester graph can be empty with current sample dates

- The store defaults graph view mode to `semester`.
- `useGraphFilters` includes only courses whose start/end dates contain the current date.
- The seeded course date range ends in May 2026.
- The review date is November 21, 2026.

Consequently, the default semester view is expected to filter out the seeded courses and their concepts. The fixed semester start in `useWeekGraph.ts` is also stale for a new academic term.

#### 4. Production API configuration is not supported yet

- `frontend/src/lib/api.ts` hard-codes `http://localhost:8000`.
- `frontend/src/hooks/useVoice.ts` independently hard-codes the same backend origin.
- `backend/main.py` only allows the local Vite development origin through CORS.
- `backend/run.py` binds to loopback.

The app therefore needs configurable frontend API origins and environment-specific CORS before it can operate as a deployed web application.

#### 5. TLS certificate verification is disabled in the TerpAI bridge

The active HTTPX TerpAI service constructs its client with TLS verification disabled. This removes certificate validation and creates a man-in-the-middle exposure. Restore normal certificate verification before any deployment or use with real credentials.

### High — functionality or correctness is materially limited

#### 6. Notes endpoint is not loaded by the frontend

The backend exposes `GET /api/notes`, and the frontend store has a `notes` field and setter. However, `useMockData.ts` does not request notes or call `setNotes`. Notes therefore remain empty in frontend state.

#### 7. Specialized-agent routing is not isolated

The knowledge flow can create a topic-specific agent and save an agent URL in frontend state. The active chat bridge primarily uses the configured main agent and injects topic context into its prompt; it does not consistently route requests to the saved specialized agent URL as a separate agent session. The UI wording should be aligned with actual routing behavior or the routing implementation should be completed.

#### 8. Knowledge refresh can report success after a skipped update

The knowledge refresh path depends on `TERPAI_CONFIG_STATE_ID`. That variable is read by the service but is missing from `.env.example`. The route can return a success-shaped response even when the service skipped the actual knowledge update. Success should reflect confirmed synchronization.

#### 9. Voice input is not wired into the active Command Center

The active Command Center's microphone control has no recording/transcription handler. Its voice-mode toggle changes presentation state and placeholder text, but does not start speech recognition. The separate `useVoiceChat` flow exists in the Chat modal path, but that modal is not opened by the active shell's normal navigation.

The voice-command hook also contains a generic `cmsc` alias that can match before a more specific CMSC course code, causing a command for a course such as CMSC351 to resolve to CMSC330. Its hard-coded course aliases do not cover the current course catalog.

#### 10. Browser speech fallback state and stop handling are inconsistent

The TTS hook falls back to browser speech synthesis when the backend voice request fails. The fallback path does not synchronize the same playback state as the audio-element path, and `stop()` does not cancel browser speech synthesis. This can cause the UI to show the wrong playback state and can interfere with conversational listening behavior.

#### 11. Frontend TypeScript build likely has unused-symbol failures

The frontend TypeScript configuration enables `noUnusedLocals`. Static inspection found at least:
- An unused `courseName` local in `TriageAlertFeed.tsx`.
- An unused `AlertTriangle` import in `AssignmentInspector.tsx`.
- An unused local `status` declaration in the assignment-click handler in `TriageAlertFeed.tsx`.

The build command has not been run here, so this is a strong static indication, not a captured compiler log.

#### 12. Chat SSE errors are not surfaced by the frontend

The backend can emit error-only SSE payloads. The frontend stream parser only appends payloads containing `content`, so a provider error event can leave an empty assistant message rather than a useful error state.

#### 13. Study Lab course selection can initialize empty

`StudyLabPane` initializes `courseId` from the courses currently in the store. If the pane mounts before asynchronous course loading finishes, the initial value is empty and is not subsequently synchronized when courses arrive.

#### 14. Dashboard/profile values are demonstrations, not live records

The profile contains hard-coded sample identity, university ID, GPA, academic year, and transcript metadata. The command-center brief and triage store contain fixed sample assignments/statuses. The system-status display is initialized with static statuses rather than live dependency checks. These should be clearly marked as demo data or replaced with real authenticated data.

#### 15. No durable persistence or user isolation

Chat messages, generated study artifacts, frontend mastery updates, and backend mastery state are in-memory. Refreshing or restarting loses these values. There is no database, authentication, or per-user authorization.

### Medium — maintainability and documentation

#### 16. Brand identity is mixed

The repository is named Zenith, but source copy, prompts, CSS class names, component names, and legacy plans still contain Aether branding. The main README and frontend README were stale. The documentation branch updates the two README files and introduces technical documentation; application branding itself remains unchanged pending a dedicated code-change task.

#### 17. Legacy Playwright fallback is not the active integration

The active TerpAI service uses HTTPX. `backend/services/playwright_fallback/` contains the previous browser-automation implementation and manual inspection scripts. Playwright is not included in the current backend requirements. Keep this directory explicitly archival or remove it in a separately reviewed cleanup.

#### 18. Dependency and test reproducibility need work

- No frontend lockfile is tracked.
- Backend dependency versions are not pinned.
- No GitHub Actions workflows are present.
- GitHub reports zero workflow runs and the reviewed commit has no status checks.
- There is no maintained automated backend test suite.
- The legacy Playwright test script is a manual browser inspector, not a CI test.

#### 19. Some visible controls are not connected

Examples found during static inspection include the Command Center's Chat History control, microphone controls, the Profile upload control, and other legacy modal/page components. Some components are not referenced by the active shell. Review controls against actual behavior before release.

#### 20. Static sample dates and terminology are stale

The sample dataset, voice brief, fixed semester start, and triage comments are centered on a prior academic period. Chat prompt wording calls the deadlines upcoming while including historical records. The seed data should be refreshed or explicitly labeled as a fixed demo scenario.

## Feature status matrix

| Area | Source implementation | Current assessment |
|---|---|---|
| Command Center layout | React components and Zustand | Implemented UI; much content is seeded/static |
| 3D knowledge graph | ForceGraph3D + Three.js | Implemented UI; default semester filter is stale |
| Calendar | Calendar page over seeded assignments | Implemented UI; data freshness/filtering needs work |
| Profile | Static profile and academic history | Demo-only |
| Chat | FastAPI SSE + TerpAI/Gemini | Implemented path; error handling and routing need work |
| Flashcards / quiz / study guide | Generate endpoints + UI components | Implemented path; provider/runtime testing pending |
| Concept context generation | Knowledge router + Gemini/TerpAI | Partially implemented; sync status/routing need work |
| Voice output | ElevenLabs TTS + browser fallback | Partially implemented; fallback state/stop handling needs work |
| Voice input | Transcription route + separate hook | Backend route exists; active Command Center control is disconnected |
| UMD catalog | UMD.io proxy | Implemented external catalog integration |
| Document parsing | Upload route + parser service | Implemented path; upload hardening/testing pending |
| Mastery tracking | Frontend and backend in-memory stores | Prototype only; not durable/synchronized |
| Canvas integration | Status labels and sample data only | Not implemented as a real Canvas integration |
| Authentication / database | None found | Not implemented |

## Verification evidence and limitations

The following repository facts were checked through source and GitHub metadata:

- Default branch: `main`
- Reviewed commit: `83e685c91a7758cd35855b3ca9913a75727efd63`
- Tracked tree: 136 files and 164 tree entries at the time of inspection
- GitHub Actions workflow runs: 0
- Commit status checks: none reported
- `.github/workflows`: not present in the repository tree

The following commands were **not** executed because this review used repository API access rather than a local checkout/runtime:

- `npm install`
- `npm run build`
- `npm run lint`
- `python -m data.validate`
- FastAPI startup and endpoint smoke tests
- Live TerpAI, Gemini, ElevenLabs, and UMD.io integration tests
- Browser end-to-end tests

Therefore, this is a source-level audit, not a claim that the application has passed runtime verification.

## Recommended remediation order

1. Fix the concept/connection schema mismatch and make `python -m data.validate` pass.
2. Refresh sample course/syllabus dates and make semester selection data-driven.
3. Fix TypeScript unused-symbol errors and establish a green frontend build/lint baseline.
4. Repair the initial graph endpoint and add API tests.
5. Add environment-based frontend API configuration and explicit CORS origins.
6. Restore TLS verification and improve secret/configuration handling.
7. Wire voice input and study commands into the active UI; correct course alias matching and speech cancellation.
8. Make specialized-agent routing and knowledge-refresh results truthful.
9. Add database persistence, authentication, authorization, and per-user data isolation before multi-user use.
10. Add CI, automated API/component tests, and browser smoke tests.

## Scope of this branch

This branch is documentation-only:

- Updated the root README to describe Zenith.
- Replaced the default frontend README with project-specific guidance.
- Added architecture, API, development, and audit documents.

No application source code, dependencies, credentials, deployment settings, or existing sample data have been changed.
