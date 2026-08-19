# Software Design & Project Documentation (SDD)

## Nevo &ndash; Cross-Platform Note Management Application

**Course:** 503107 &ndash; Cross-Platform Mobile App Development, Semester I/2026-2027, Ton Duc Thang University
**Document type:** Full Project / Technical Design Documentation (companion to the FRS)
**Prepared by:** Long (Dranov)
**Status:** Draft v1.1 &ndash; Date: 2026-08-19

# 1. Introduction

This document is the technical companion to the Functional Requirements Specification (FRS). While the FRS defines *what* the system must do, this document defines *how* the team will build it: architecture, technology choices, data model, security design, offline strategy, AI feature design, testing plan, and the submission/deployment process required by the course. It is written to industry-standard practice (comparable to an SRS/SDD pair) so the instructor can trace every rubric item to both a requirement and an implementation decision.

# 2. System Architecture

## 2.1 Architectural Style

Single Flutter project, layered architecture, producing two build targets (Web + one native platform) from one Dart codebase:

- **Presentation layer** &ndash; widgets/screens, purely declarative, no direct data-access or business logic.
- **State-management layer** &ndash; e.g. Riverpod, Bloc/Cubit, or Provider (team selects one and applies it consistently; documented in Readme.txt). Owns UI state, loading/error/empty state, and orchestrates calls into the domain layer.
- **Domain / business-logic layer** &ndash; use-case/service classes (e.g. NoteService, AuthService, ShareService, AiService) independent of any specific backend SDK, so the backend can be swapped or mocked in tests.
- **Data-access layer** &ndash; repositories that talk to the backend (BaaS SDK or REST/GraphQL client) and to the local offline store, hiding persistence details from the domain layer.
- **Platform-specific layer** &ndash; a thin abstraction (e.g. conditional imports / platform channels) isolating genuinely platform-specific code (camera/file-picker differences, secure storage), never duplicating core logic.

Recommended dependency direction: Presentation &rarr; State &rarr; Domain &rarr; Data access &rarr; (Remote backend | Local cache). This satisfies Rubric ID 30 (separation of presentation, state, business logic, data access).

## 2.2 High-Level Component Diagram (textual)

Flutter Client (Web build / Native build, shared Dart core) &harr; HTTPS/WebSocket &harr; Backend (Auth service, Database, Realtime channel, File storage, Serverless functions) &harr; LLM Provider (invoked only from serverless functions, never directly from the client).

## 2.3 Technology Stack (reference selection)

| Concern | Recommended option | Notes |
|---|---|---|
| Client framework | Flutter (stable channel) + Dart | Single project; Web + Android (or Windows/macOS) targets |
| State management | Riverpod or Bloc | Pick one, apply consistently, document rationale |
| Backend | Firebase (Auth + Firestore + Storage + Cloud Functions + Realtime listeners) *or* Supabase (Auth + Postgres + Storage + Realtime + Edge Functions) | Either satisfies all mandatory requirements; Supabase gives SQL + row-level security, Firebase gives fully managed NoSQL + security rules |
| Local persistence (offline) | Drift/SQLite, Hive, or the BaaS SDK's built-in offline cache | Must support pending-operation queue and per-account isolation |
| Real-time collaboration | Firestore snapshot listeners or Supabase Realtime channels | Drives live updates on shared, editable notes |
| LLM integration | Any provider (e.g. OpenAI/Anthropic/Gemini API) called only from Cloud/Edge Functions | API key stored as a server-side secret, never in the client |
| CI/testing | flutter test, integration_test package | 3 unit + 3 widget + 1 integration test minimum |
| Hosting (Web) | Firebase Hosting, Vercel, Netlify, or Supabase-compatible static host | Must remain publicly reachable through grading |

# 3. Data Model

## 3.1 Entity-Relationship Overview

| Entity | Key Fields | Notes |
|---|---|---|
| User | id, email, displayName, avatarUrl, isEmailVerified, passwordHash*, preferences (theme, fontSize, defaultNoteColor, defaultView), createdAt | *passwordHash only relevant for a custom backend; managed auth stores this internally |
| Note | id, ownerId, title, content, createdAt, updatedAt, isPinned, pinnedAt, isPasswordProtected, notePasswordHash, color, syncStatus (synced/pending/failed), isDeleted | syncStatus supports offline UI states |
| Label | id, ownerId, name, createdAt | Renaming updates display everywhere via reference by id, not by copied string |
| NoteLabel | noteId, labelId | Many-to-many join |
| Attachment | id, noteId, type (image/video/file), storagePath, fileName, sizeBytes, mimeType, uploadedAt | Access gated by note authorization |
| Share | id, noteId, ownerId, recipientEmail, recipientUserId, permission (read/edit), sharedAt | Enforced by backend security rules, not client-side filtering |
| AiSummary | id, noteId, summaryText, generatedAt, model | Regenerable; does not overwrite Note.content |
| AiQueryLog | id, userId, question, answer, sourceNoteIds[], createdAt | Optional audit trail for the Q&A feature |

## 3.2 Local (Offline) Store

Mirrors Note, Label, NoteLabel, and Attachment metadata, scoped by the currently authenticated userId (a distinct local database/namespace per account prevents cross-account leakage). Each locally modified row carries a syncStatus and a pendingOperation (create/update/delete) that a background sync worker replays against the backend when connectivity returns, following a documented conflict rule (e.g. last-write-wins by updatedAt, or a merge/flag-for-review strategy the team documents in Readme.txt).

# 4. Security Design

- **Account passwords:** delegated to managed auth, or hashed with bcrypt/argon2 if self-implemented; never logged or returned in API responses.
- **Note passwords:** stored as a salted hash, never plaintext; verified server-side before returning protected content, not just before rendering it client-side.
- **Authorization:** every read/write to a note, attachment, or share record is checked against backend security rules (Firestore/Storage security rules, Supabase row-level security policies, or custom backend middleware) so that a read-only collaborator cannot mutate a note even by calling the data API directly.
- **Secrets:** LLM API keys, DB credentials, and signing keys live only in server-side environment configuration (.env, function config, or secret manager); a safe .env.example template ships in the repo; nothing sensitive is committed to Git history.
- **Transport:** all traffic over HTTPS/TLS; the deployed Web build is served over HTTPS.

# 5. Offline Persistence & Synchronization Design

1. On each successful fetch, notes/labels/attachment metadata are cached locally, scoped to the active account.
2. While offline, reads are served from the local cache; the UI shows an "offline" indicator.
3. Writes made offline are applied optimistically to the local cache and queued as pending operations; the UI marks affected notes "pending sync."
4. On reconnect, a sync worker replays the queue in order, retries transient failures with backoff, and marks unresolved failures "sync failed" with a retry action surfaced to the user.
5. Conflicts (remote updatedAt newer than the queued local base) are resolved per the documented strategy (default: last-write-wins on updatedAt, with the losing version optionally preserved as a local "conflict copy" for manual review).
6. Switching accounts clears/segregates the local cache so no note data leaks between users on a shared device.

# 6. AI Feature Design

## 6.1 AI Summary (Rubric 27)

Client sends noteId to a secured backend function &rarr; function verifies the caller is authorized to read the note (and, if protected, that it has been unlocked in-session) &rarr; function calls the LLM with the note content &rarr; returns a generated summary &rarr; client displays it separately from the note body and offers a "Regenerate" action; the original note.content is never overwritten automatically.

## 6.2 AI Question Answering (Rubric 28)

Retrieval-augmented flow: backend retrieves candidate notes owned by or shared with the authenticated user (optionally narrowed by a vector or keyword index), filters out still-locked protected notes, constructs a grounded prompt from the retrieved content, and asks the LLM to answer using only that content. The response includes references to the specific source note(s); the client renders each reference as a link that opens the note on both Web and native. If retrieval finds nothing sufficiently relevant, the backend returns an explicit "not enough information" result instead of asking the LLM to guess.

# 7. Testing Strategy (Rubric 30)

| Level | Minimum | Example targets |
|---|---|---|
| Unit tests | 3 | Note repository CRUD logic, auto-save debounce logic, note-password hashing/verification helper |
| Widget tests | 3 | Note editor auto-save UI states, grid/list view toggle rendering, pinned-note ordering in the note list widget |
| Integration test | 1 (critical flow) | End-to-end: register &rarr; log in &rarr; create note &rarr; share with a second account &rarr; verify recipient sees it with correct permission |

Artificial tests that do not exercise real project behavior are not counted toward the rubric; each test must assert meaningful, feature-relevant outcomes.

# 8. Build, Deployment & Repository Structure

## 8.1 Suggested Repository Layout

```
/lib
  /presentation   (screens, widgets)
  /state          (providers / blocs / cubits)
  /domain         (services, models, use cases)
  /data           (repositories, remote + local data sources)
  /platform       (platform-specific abstractions)
/test             (unit + widget tests)
/integration_test (integration test)
/backend or /functions   (custom backend source or serverless functions, if applicable)
pubspec.yaml
pubspec.lock
Readme.txt
.env.example
```

## 8.2 Deployment Targets

- **Flutter Web:** built via `flutter build web`, deployed to a publicly reachable HTTPS host, kept online for the entire grading period.
- **Native release (choose one):** Android release APK (`flutter build apk --release`), a complete Windows release folder/ZIP with executable and dependencies (`flutter build windows`), or a macOS .app bundle ZIP (`flutter build macos`). A bare Windows .exe without its dependency files is not accepted.
- **Backend:** managed BaaS project configuration (security rules, indexes, functions) or, for a custom backend, complete source, DB schema/migrations, .env.example, and Docker Compose if multiple services are needed.

## 8.3 Git & Teamwork Compliance Plan

To satisfy the mandatory teamwork rule, the team plans at least 4 consecutive calendar weeks (Mon&ndash;Sun) in which every member pushes &ge;2 meaningful commits per week under their own GitHub identity (feature work, refactors, tests, config, fixes, or substantial documentation &ndash; not empty/merge-only/whitespace/generated-file commits). Before submission, capture a GitHub Insights contributor-activity screenshot covering the qualifying weeks and store it as Screenshot.png; keep the repository's .git history intact in the submitted source folder (a GitHub ZIP download is not sufficient).

# 9. Submission Package Checklist (per Section VII of the brief)

| Item | Contents |
|---|---|
| Rubric.xlsx | Team self-assessment of all 32 criteria; public Web URL; native platform + artifact name; GitHub repo URL; grading credentials |
| source/ | Full Flutter project cloned from GitHub (with .git folder), pubspec.yaml/.lock, tests, platform folders, backend config/security rules or custom backend source; no build outputs or dependency caches |
| release/ | web-url.txt (deployed Web URL) + one native artifact (APK / Windows ZIP / macOS .app ZIP) |
| demo.mp4 | All members participate; introduces architecture/state-mgmt/backend/persistence/platforms; demonstrates every implemented rubric criterion on &ge;1 submitted platform, plus login, note CRUD, attachments, offline sync, sharing/real-time updates, and &ge;1 AI feature on both Web and native; &ge;1080p, clear audio |
| Readme.txt | SDK versions, platforms, backend/hosting, architecture/state-mgmt, persistence/sync strategy, key packages, exact build/test/run commands, config instructions, public URLs, native-artifact info, grading accounts, known limitations |
| Screenshot.png | GitHub Insights contributor-activity chart covering the 4 qualifying weeks, clearly identifying repo, contributors, and period |

Final packaging: all of the above inside a folder named `id1_fullname1_id2_fullname2`, compressed to a ZIP of the same name, submitted only through the designated e-learning system.

# 10. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Missing a qualifying commit week for one member &rarr; 0.5 pt deduction for the whole team | Track weekly commit status per member in the To-dos tracker; remind members mid-week |
| Secrets accidentally committed | .gitignore for .env; pre-commit check; .env.example only in Git |
| Offline sync conflicts corrupt data | Adopt and document a single conflict strategy early; cover with a unit test |
| Native build fails to reproduce for the instructor | Document exact build commands in Readme.txt; verify a clean clone builds before submission |
| LLM key exposure | All LLM calls routed through serverless functions only; never call the LLM directly from Flutter |

# 11. Glossary

See FRS Section 1.3 for shared definitions and acronyms (FRS and this document share one glossary).
