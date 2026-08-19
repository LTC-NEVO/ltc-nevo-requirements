---
document_type: Product Requirement Document
project: Nevo — Cross-Platform Note Management Application
team: LTC
version: "1.0"
status: Draft
last_updated: "2026-08-20"
upstream_brd: "BRD_Nevo_v1.0.md"
upstream_specs: ["../documents/FRS.md", "../documents/SDD.md", "../documents/Screen-Inventory.md"]
---

# PRD — Nevo v1.0

> **Purpose:** describe every functional behaviour of Nevo at build granularity — enough that a
> developer can implement without asking, and a reviewer can verify without interpreting.
>
> **Conventions:**
> - `MUST` / `SHOULD` carry their RFC-2119 sense. `MUST` failures fail the rubric criterion.
> - `[TO CONFIRM]` = waiting on an open question (see BRD §11 and `TRACKING_Nevo.md` §3).
> - Every requirement carries its **Rubric** criterion. A requirement with no rubric mapping is out
>   of scope until the BRD says otherwise.
> - Diagrams live in the knowledge base, not here — this document links rather than duplicates.
>
> **Sources:** `../documents/FRS.md` (32 requirements, rubric-mapped), `../documents/SDD.md`
> (technical design), `../documents/Screen-Inventory.md` (21 screens with routes and access levels),
> and `../ltc-nevo-knowledge-base/` (architecture, flows, security model).
>
> **Out of this document's scope:** architecture, data model, API contracts, deployment. Those live
> in [`../ltc-nevo-knowledge-base/`](../ltc-nevo-knowledge-base/) and are not restated here.

---

## 0. Metadata

| Field | Value |
|---|---|
| Product | Nevo — cross-platform note management (v1.0) |
| Status | Draft — 8 open questions, 0 blocked requirements. Not yet reviewed or signed off. **Implementation not started: both repos are scaffolds.** |
| Owner (DRI) | Product: Long (Dranov) · Eng: `[TO CONFIRM]` (Q-01) |
| Version | 1.0 |
| Repos | `ltc-nevo-customer-api` (NestJS 11) · `ltc-nevo-web-app` (Flutter) |
| Tech stack | Locked — see [`../ltc-nevo-knowledge-base/02-tech-stack.md`](../ltc-nevo-knowledge-base/02-tech-stack.md) |
| Upstream BRD | [BRD_Nevo_v1.0.md](BRD_Nevo_v1.0.md) |
| Screens | 21, specified in [`../documents/Screen-Inventory.md`](../documents/Screen-Inventory.md) |
| Requirement count | **74 build-level requirements** across 8 groups, covering all 32 rubric criteria |
| Updated | 2026-08-20 |

---

# PART A — PRODUCT

## 1. Problem & goals

**Problem.** Personal notes are either device-local (unavailable elsewhere) or cloud-only
(unavailable offline, and uniformly exposed to whoever holds the session). Neither handles the case
of "most of my notes are ordinary, a few are not".

**Goals.**

| # | Goal | Measured by | Target |
|---|---|---|---|
| G1 | Every rubric criterion implemented and demonstrable | Criteria shown working in `demo.mp4` | 32/32 |
| G2 | Notes usable with no connectivity | Previously loaded notes readable + editable offline | 100% of loaded notes |
| G3 | Protection that survives an attacker with the session | Operations on a locked note refused server-side | 100%, all operations |
| G4 | Sharing that cannot be bypassed | Read-only collaborator's direct API write | Always 403 |
| G5 | AI answers that never leak | Notes cited from outside the caller's visible set | 0 |
| G6 | One codebase, two platforms | Mandatory features working on Web and Android | Full parity |

---

## 2. Scope

### In scope — by capability

| Group | Capability | Requirements | Rubric | Status |
|---|---|---|---|---|
| `FR-ACC` | Account: register, activate, sign in, reset, profile, preferences | 14 | 1–8 | ✅ Specified |
| `FR-NOTE` | Note CRUD, auto-save, lifecycle, ordering, badges, search | 17 | 9–14, 17–19 | ✅ Specified |
| `FR-LBL` | Labels: manage, tag, filter | 7 | 20–22 | ✅ Specified |
| `FR-ATT` | Attachments: add, validate, view, remove | 7 | 15–16 | ✅ Specified |
| `FR-PROT` | Per-note password protection and unlock | 8 | 23–24 | ✅ Specified |
| `FR-SHR` | Sharing, permissions, real-time collaboration | 10 | 25–26 | ✅ Specified |
| `FR-AI` | AI summary and grounded Q&A | 7 | 27–28 | ✅ Specified |
| `FR-XC` | Cross-cutting: UI/UX, architecture + tests, offline sync, deployment | 4 | 29–32 | ✅ Specified |
| **Total** | | **74** | **32 criteria** | |

### Out of scope (v1.0)

Confirmed in BRD §5 — repeated here only so nobody has to open two documents to answer "are we
doing X":

- A second native target beyond Android; the other platform folders stay unused.
- Note version history, PDF/Markdown export, a dedicated desktop shortcut layer (FRS §5.9
  optional enhancements — explicitly not planned).
- Admin console, multi-tenancy, billing.
- Character-level collaborative merge (OT/CRDT). FR-26 requires live propagation; the documented
  strategy is last-write-wins with a preserved conflict copy.
- Vector search / embeddings.

> [!NOTE]
> **Share notification email (`FR-SHR-08`) is marked SHOULD, not MUST.** FRS §5.9 lists it as an
> optional enhancement, but the knowledge base already designs `MailService` to send it and the
> infrastructure is needed for activation and password reset anyway. It is specified so it can be
> built if Week 3 has room, and dropped without a scope change if it does not. Tracked as Q-07.

---

## 3. Users & user stories

| Persona | Description |
|---|---|
| **Visitor** | Not signed in. Can reach login, register, forgot/reset password, and account activation only. |
| **Owner** | A signed-in user acting on their own notes. Full control including delete, share, and note password. |
| **Collaborator (edit)** | Signed in, holds an `EDIT` share on someone else's note. May change content, labels, attachments. May not delete, re-share, or change the note password. |
| **Collaborator (read)** | Signed in, holds a `READ` share. May view only — including live updates — and cannot mutate through any path. |
| **Grader** | Uses supplied credentials to verify all 32 criteria on the deployed Web build and the submitted APK. |

| # | User story | Rubric |
|---|---|---|
| US-1 | As a visitor, I register with my email and start using the app immediately, without waiting for email verification. | 1, 2 |
| US-2 | As a user, I sign in and land on my own notes; if I am not signed in, protected screens send me to login. | 3 |
| US-3 | As a user who forgot my password, I reset it from an email and sign in again. | 4 |
| US-4 | As a user, I view and edit my display name and avatar, change my password, and set theme, font size, note colour, and default view — and those settings follow me to my other device. | 5–8 |
| US-5 | As a user, I browse my notes as a grid or a list, search them as I type, filter by label, and see at a glance which are pinned, shared, or locked. | 9, 10, 17–19, 22 |
| US-6 | As a user, I write a note that saves itself, and my last words survive backgrounding, closing, or killing the app. | 11, 12, 14 |
| US-7 | As a user, I attach images, videos, and files to a note from my gallery, camera, or file picker. | 15, 16 |
| US-8 | As a user, I organise notes with labels I can rename or delete without losing the notes. | 20–22 |
| US-9 | As a user, I lock an individual note with its own password so that holding my session is not enough to read it. | 23, 24 |
| US-10 | As an owner, I share a note read-only or editable, change or revoke that access at any time, and know a read-only recipient truly cannot change it. | 25 |
| US-11 | As a collaborator, I see notes shared with me and edit the editable ones alongside the owner, live. | 26 |
| US-12 | As a user, I summarise a long note with AI without losing the original. | 27 |
| US-13 | As a user, I ask questions about my notes and get answers grounded in them, with links back to the source. | 28 |
| US-14 | As a user, I keep reading and writing notes with no connectivity, and my changes sync when I am back. | 31 |
| US-15 | As a grader, I open the public URL and the installed APK and verify each criterion on both. | 29, 30, 32 |

---

## 4. Functional Requirements

Guard notation: `public` = no session; `JWT` = signed in; `note:read|edit|own` = `NoteAccessGuard`
with the stated permission. Endpoint and screen references are defined in
[`../ltc-nevo-knowledge-base/06-integrations.md`](../ltc-nevo-knowledge-base/06-integrations.md)
and [`../documents/Screen-Inventory.md`](../documents/Screen-Inventory.md).

### 4.1 Account (`FR-ACC`) — rubric 1–8

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-ACC-01 | System MUST register a user with email, display name, and password entered twice; both password fields MUST match, and the password MUST be ≥8 characters. | Must | 1 | US-1 |
| FR-ACC-02 | System MUST store the account password as a bcrypt hash (cost 10) and MUST NOT return it from any endpoint. | Must | 1 | US-1 |
| FR-ACC-03 | Registration with an email already in use MUST fail with `USER_ALREADY_EXISTS` and surface as an inline field error, not a generic toast. | Must | 1 | US-1 |
| FR-ACC-04 | On successful registration the user MUST be signed in automatically and land on Home — not returned to the login screen. | Must | 1 | US-1 |
| FR-ACC-05 | Registration MUST dispatch an activation email; SMTP failure MUST NOT fail the registration, and the client MUST offer "Resend activation". | Must | 1, 2 | US-1 |
| FR-ACC-06 | An unverified account MUST retain **full** functionality. No route, guard, or endpoint may check `isEmailVerified`. | Must | 2 | US-1 |
| FR-ACC-07 | A persistent, prominent banner MUST appear on Home and its children while the account is unverified, and MUST disappear after successful activation — identically on Web and Android. | Must | 2 | US-1 |
| FR-ACC-08 | Unauthenticated access to any protected screen MUST redirect to `/login`; `/register`, `/forgot-password`, and `/reset-password/:token` MUST remain reachable without a session; successful login MUST route to Home showing the user's own notes. | Must | 3 | US-2 |
| FR-ACC-09 | Login MUST return the same `INVALID_CREDENTIALS` error for an unknown email and a wrong password, and MUST refuse any account whose status is not `ACTIVE`. | Must | 3 | US-2 |
| FR-ACC-10 | Logout MUST revoke server-side (clear the refresh hash, delete the Redis access key) **before** the client clears local state; an expired access token MUST be refreshed transparently once, with concurrent 401s sharing a single refresh call. | Must | 3 | US-2 |
| FR-ACC-11 | Password reset MUST be requested by email and completed through an emailed link or OTP `[TO CONFIRM — Q-04]`; the token/OTP MUST expire in 15 minutes and be single-use; the forgot-password response MUST be identical whether or not the email exists. | Must | 4 | US-3 |
| FR-ACC-12 | A completed reset MUST invalidate existing sessions and MUST NOT sign the user in — they are routed to `/login` to sign in manually. | Must | 4 | US-3 |
| FR-ACC-13 | Profile MUST show display name and avatar with a sensible default when none is set, and MUST allow updating both; an uploaded avatar MUST be validated for type and size server-side (≤5 MB, image MIME allow-list). | Must | 5, 6 | US-4 |
| FR-ACC-14 | Change password MUST require the current password plus the new password twice, and on success MUST invalidate other sessions; preferences (theme, font size, default note colour, default note view) MUST persist **server-side** per user and re-apply on any submitted platform after reopening. | Must | 7, 8 | US-4 |

### 4.2 Notes (`FR-NOTE`) — rubric 9–14, 17–19

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-NOTE-01 | Home MUST display notes as a **grid by default**, switchable to a list; the choice MUST be written to `defaultNoteView` and re-applied on any platform. | Must | 9, 10 | US-5 |
| FR-NOTE-02 | Both views MUST render correctly from compact mobile through tablet, desktop, and browser widths, in both orientations. | Must | 9, 10, 29 | US-5, US-15 |
| FR-NOTE-03 | A single reusable editor screen MUST serve both creation (`/notes/new`) and editing (`/notes/:noteId`) — the same widget, not two implementations. Only title and content are mandatory; content format is `[TO CONFIRM — Q-05]`. | Must | 11 | US-6 |
| FR-NOTE-04 | The editor MUST auto-save content after ~800 ms of typing inactivity, with **no Save button anywhere** in the editor. | Must | 11, 12 | US-6 |
| FR-NOTE-05 | The editor MUST always show one of three save states: *Saving…*, *Saved HH:MM*, or *Couldn't save · Retry*. A silent save is a defect. | Must | 12 | US-6 |
| FR-NOTE-06 | On app pause, background, or tab close the editor MUST flush any pending debounce immediately rather than waiting it out. | Must | 14 | US-6 |
| FR-NOTE-07 | The last valid change MUST survive pause, background, close, kill, and reopen on each submitted platform. | Must | 14 | US-6 |
| FR-NOTE-08 | Deleting a note MUST require an explicit confirmation dialog client-side, and MUST be a soft delete server-side that cascades to labels, attachments, shares, and summaries in one transaction. | Must | 13 | US-5 |
| FR-NOTE-09 | Notes MUST order pinned-first, then by pin time descending, then by last modified descending. The client MUST render the order the API returns and MUST NOT re-sort. | Must | 17 | US-5 |
| FR-NOTE-10 | Pinning and unpinning MUST keep `isPinned` and `pinnedAt` consistent, and MUST be available to the owner only. | Must | 17 | US-5 |
| FR-NOTE-11 | Note cards MUST show recognisable indicators for pinned, password-protected, and shared states, in **both** grid and list, on **both** platforms; multiple indicators MUST be able to co-exist on one card. | Must | 18 | US-5 |
| FR-NOTE-12 | Note cards MUST additionally show pending-sync and sync-failed indicators when the local row is in those states. | Must | 18, 31 | US-14 |
| FR-NOTE-13 | Search MUST filter notes by keyword in **title and content** as the user types, debounced ~300 ms, with no Search button. | Must | 19 | US-5 |
| FR-NOTE-14 | While offline, search MUST run against the local cache and return the same shape of results. | Must | 19, 31 | US-14 |
| FR-NOTE-15 | Every note list query MUST be scoped in SQL to the caller's visible set (owned ∪ shared), never fetched broadly and filtered afterwards. | Must | 25 | US-10 |
| FR-NOTE-16 | Note colour MUST default from the user's `defaultNoteColor` preference and be changeable per note. | Should | 8 | US-4 |
| FR-NOTE-17 | List endpoints MUST paginate (`page`, `pageSize` default 20, max 100) and return `{ items, totalCount }`. | Must | 9 | US-5 |

### 4.3 Labels (`FR-LBL`) — rubric 20–22

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-LBL-01 | Users MUST be able to list, create, rename, and delete their own labels. | Must | 20 | US-8 |
| FR-LBL-02 | Label names MUST be unique per owner, compared case-insensitively; a duplicate MUST fail with `LABEL_NAME_DUPLICATE`. | Must | 20 | US-8 |
| FR-LBL-03 | Renaming a label MUST change its display everywhere it appears, with no rewrite pass — notes reference labels by id, never by copied string. | Must | 20 | US-8 |
| FR-LBL-04 | Deleting a label MUST remove only the note↔label links. Every previously tagged note MUST remain intact and reachable. | Must | 21 | US-8 |
| FR-LBL-05 | A note MUST support zero or more labels, attachable and detachable from the editor. | Must | 21 | US-8 |
| FR-LBL-06 | Home MUST filter notes by one or more selected labels; filtering by two labels MUST return notes carrying both. | Must | 22 | US-5 |
| FR-LBL-07 | Labels MUST be private to their owner — a collaborator MUST NOT see the owner's labels on a shared note. | Must | 22 | US-11 |

### 4.4 Attachments (`FR-ATT`) — rubric 15–16

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-ATT-01 | A note MUST support one or more attachments of type image, video, or general file, added from gallery, camera, browser picker, or native picker as the platform allows. | Must | 15 | US-7 |
| FR-ATT-02 | File type and size MUST be validated **server-side** regardless of any client check: image ≤10 MB, video ≤50 MB, file ≤20 MB `[TO CONFIRM — Q-06]`, against a MIME allow-list. | Must | 15, 16 | US-7 |
| FR-ATT-03 | Cancelling the picker MUST be a silent no-op — no error surfaced. | Must | 15 | US-7 |
| FR-ATT-04 | A denied Android permission MUST be explained with a path to app settings, and MUST NOT crash or dead-end the flow. | Must | 15 | US-7 |
| FR-ATT-05 | Camera capture MUST be offered on Android and hidden on Web, where it does not exist — a broken control is worse than an absent one. | Must | 15, 29 | US-7 |
| FR-ATT-06 | Attachment objects MUST be private. Download MUST require authorization on the parent note and MUST be served through a pre-signed URL valid ≤5 minutes. | Must | 16 | US-7 |
| FR-ATT-07 | A user with no access to a note MUST NOT be able to fetch its attachments by any URL. | Must | 16, 25 | US-10 |

### 4.5 Note protection (`FR-PROT`) — rubric 23–24

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-PROT-01 | Any note MUST be protectable with its own password, independent of the account password and of every other note. | Must | 23 | US-9 |
| FR-PROT-02 | The note password MUST be stored as a bcrypt hash (cost 10) and MUST be verified server-side **before** protected content is returned — never after rendering it. | Must | 23 | US-9 |
| FR-PROT-03 | While protected and not unlocked, **every** operation on the note MUST be refused with `NOTE_LOCKED` (423): view, edit, delete, share, summarise, and inclusion in AI retrieval. There is no exempt operation. | Must | 23 | US-9 |
| FR-PROT-04 | A correct password MUST issue a server-side unlock grant valid 15 minutes, held in Redis and keyed per user and note. | Must | 23 | US-9 |
| FR-PROT-05 | Unlock attempts MUST be rate-limited (5 per 15 minutes per user and note). | Must | 23 | US-9 |
| FR-PROT-06 | Changing or removing the note password MUST require the current note password first. | Must | 24 | US-9 |
| FR-PROT-07 | Changing or removing protection MUST immediately revoke every outstanding unlock grant for that note, for every user. | Must | 24 | US-9 |
| FR-PROT-08 | The client MUST hold an unlock grant in session memory only, MUST NOT persist it, and MUST NOT cache the content of a locked note to the offline store. | Must | 23, 31 | US-9, US-14 |

### 4.6 Sharing & collaboration (`FR-SHR`) — rubric 25–26

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-SHR-01 | An owner MUST be able to share a note with one or more **registered** accounts by email, at `READ` or `EDIT` permission per recipient. | Must | 25 | US-10 |
| FR-SHR-02 | Sharing with an unregistered email MUST fail with `SHARE_RECIPIENT_NOT_REGISTERED`; sharing with yourself MUST fail with `SHARE_SELF_NOT_ALLOWED`. | Must | 25 | US-10 |
| FR-SHR-03 | Only the owner MUST be able to share, change a permission, or revoke access. An `EDIT` collaborator MUST NOT be able to re-share, delete, or change the note password. | Must | 25 | US-10 |
| FR-SHR-04 | A `READ` collaborator MUST NOT be able to mutate the note through any path, **including a direct API call with no UI involved**. Enforcement is `NoteAccessGuard`; hiding the control is not enforcement. | Must | 25 | US-10 |
| FR-SHR-05 | Revoking access MUST take effect on the recipient's very next request, and MUST evict their socket from the note room. | Must | 25 | US-10 |
| FR-SHR-06 | "Shared with me" MUST list notes shared with the current user showing permission level, sharer identity, and share timestamp. | Must | 26 | US-11 |
| FR-SHR-07 | Re-sharing a note with the same recipient MUST update the existing share rather than creating a duplicate. | Must | 25 | US-10 |
| FR-SHR-08 | System SHOULD email the recipient when a note is shared with them. | Should | — (FRS §5.9) | US-10 |
| FR-SHR-09 | Edits to a shared, editable note MUST propagate live to every authorized viewer with no manual refresh; `READ` collaborators MUST receive the same updates while remaining unable to write. | Must | 26 | US-11 |
| FR-SHR-10 | A remote update MUST NOT silently destroy a local edit in progress: if the local buffer is clean, apply the patch; if it is dirty, inform the user and let them choose. | Must | 26 | US-11 |

### 4.7 AI (`FR-AI`) — rubric 27–28

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-AI-01 | System MUST generate a concise LLM summary of a single note capturing its main ideas without altering their meaning, regenerable on demand. | Must | 27 | US-12 |
| FR-AI-02 | The summary MUST be displayed separately from the note body and MUST NEVER overwrite `Note.content`. Content MUST be byte-identical before and after generating a summary. | Must | 27 | US-12 |
| FR-AI-03 | A summary older than the note's last edit MUST be labelled stale rather than silently shown as current. | Should | 27 | US-12 |
| FR-AI-04 | AI Q&A MUST answer natural-language questions using **only** notes the authenticated caller may read; scoping MUST happen before the prompt is built, never as a filter on the answer. | Must | 28 | US-13 |
| FR-AI-05 | Protected notes without a live unlock grant MUST be excluded from AI retrieval. | Must | 28, 23 | US-13 |
| FR-AI-06 | Answers MUST be grounded in retrieved note content — not keyword matching alone, not model world-knowledge — and MUST carry references to the source notes that open on both Web and Android. | Must | 28 | US-13 |
| FR-AI-07 | When retrieval finds nothing sufficiently relevant, System MUST return an explicit "not enough information" result rather than an unfounded answer. | Must | 28 | US-13 |

### 4.8 Cross-cutting (`FR-XC`) — rubric 29–32

| # | Requirement | Priority | Rubric | Story |
|---|---|---|---|---|
| FR-XC-01 | Every screen MUST implement four states — loading, empty, success, error — and MUST be usable by touch, mouse, and keyboard, adapting across compact, medium, and expanded widths in both orientations. | Must | 29 | US-15 |
| FR-XC-02 | Both platforms MUST build from one Flutter project with a documented architecture separating presentation, state, business logic, and data access, using one state-management solution consistently; minimum **3 unit + 3 widget + 1 integration** tests, each asserting real behaviour, with a warning-free analyzer. | Must | 30 | US-15 |
| FR-XC-03 | Previously loaded notes MUST remain viewable offline; offline writes MUST apply optimistically, queue, and sync on reconnect; the UI MUST communicate offline, pending-sync, and sync-failed states; the local cache MUST be isolated per account; conflicts MUST resolve last-write-wins by `updatedAt` while preserving the losing version as a conflict copy. | Must | 31 | US-14 |
| FR-XC-04 | The Flutter Web build and all backend services MUST be deployed publicly over HTTPS and remain operational throughout grading; one working native artifact (Android APK) MUST be submitted; both releases MUST come from the same Flutter project with the same mandatory functionality. | Must | 32 | US-15 |

---

## 5. Acceptance criteria

Written as demo steps, because a criterion not demonstrable is a criterion not earned. Each maps to
requirements above; the full requirement↔test↔file matrix is in
[`../ltc-nevo-knowledge-base/09-requirements/traceability-matrix.md`](../ltc-nevo-knowledge-base/09-requirements/traceability-matrix.md).

| # | Given | When | Then | Covers |
|---|---|---|---|---|
| AC-1 | A fresh browser, no session | Register with a new email, matching passwords | Signed in on Home, activation banner visible, all features usable, activation email received | FR-ACC-01…07 |
| AC-2 | A registered, unverified user | Follow the emailed activation link | Banner disappears; behaviour identical on Web and Android | FR-ACC-05, 07 |
| AC-3 | Signed out | Open `/home` directly | Redirected to `/login`; after signing in, Home shows only that user's notes | FR-ACC-08 |
| AC-4 | A registered user | Request a password reset, use the emailed link/OTP, set a new password | Routed to login (not auto-signed-in); old session rejected; new password works | FR-ACC-11, 12 |
| AC-5 | Signed in on Web, preferences set to dark + list view | Open the Android build and sign in | Same theme and list view applied without reconfiguring | FR-ACC-14, FR-NOTE-01 |
| AC-6 | The note editor open | Type, pause ~1 s, watch the indicator; then background the app mid-typing and reopen | *Saving…* → *Saved HH:MM*; after reopen the last typed text is present; no Save button exists | FR-NOTE-04…07 |
| AC-7 | Several notes, two pinned | Open Home in grid, then switch to list | Pinned notes appear first in both views, ordered by pin time; badges for pinned/locked/shared render in both | FR-NOTE-09…11 |
| AC-8 | ~20 notes, one containing "invoice" only in its body | Type "invo" in search | Matching note appears without pressing anything, within ~300 ms of the last keystroke | FR-NOTE-13 |
| AC-9 | A note tagged with label "Work" | Rename "Work" → "Job", then delete "Job" | The new name shows everywhere immediately; after deletion the note still exists with its content intact | FR-LBL-03, 04 |
| AC-10 | The editor open on Android | Attach a photo from gallery, then attempt a 60 MB video | Photo attaches and renders; the video is refused with a size message; cancelling the picker does nothing | FR-ATT-01…03 |
| AC-11 | A protected note, session not unlocked | Attempt view, edit, delete, share, and summarise — including a direct API call | Every one refused with `NOTE_LOCKED`; after unlocking, all succeed for 15 minutes | FR-PROT-03, 04 |
| AC-12 | A protected note unlocked in two sessions | The owner changes the note password | Both existing grants stop working immediately; the correct new password unlocks again | FR-PROT-06, 07 |
| AC-13 | Note shared `READ` with user B | B calls `PATCH /notes/:id` directly, bypassing the UI | API responds 403; the note is unchanged. **Covered by the mandatory integration test.** | FR-SHR-04 |
| AC-14 | Note shared `EDIT`, open in two browsers | User A edits and the save completes | User B sees the change without refreshing; B's own unsaved buffer is not destroyed | FR-SHR-09, 10 |
| AC-15 | Access revoked while B is viewing | Owner revokes the share | B's next request fails, B is routed away, and the note leaves B's "Shared with me" | FR-SHR-05 |
| AC-16 | A long note | Generate a summary, then regenerate | A summary appears beside the body; `Note.content` is byte-identical before and after | FR-AI-01, 02 |
| AC-17 | Two accounts each with notes; a question answerable only from account A's notes | Account B asks that question | The answer never cites A's notes; when B has nothing relevant, an explicit "not enough information" is returned | FR-AI-04, 07 |
| AC-18 | A question answerable from B's own notes | B asks it | The answer cites B's notes, and tapping a reference opens that note — on both Web and Android | FR-AI-06 |
| AC-19 | Notes loaded, then network disabled | Read a note, edit it, re-enable the network | The note renders offline; the edit is marked pending; the queue drains on reconnect and the badge clears | FR-XC-03 |
| AC-20 | Account A signed in on a device with notes cached | Sign out and sign in as account B | B sees none of A's notes | FR-XC-03 |
| AC-21 | A clean machine | Follow `Readme.txt` build commands from a fresh clone | Web build and APK both produced; the deployed URL and `/health` both respond | FR-XC-04 |

---

## 6. Non-functional requirements

| # | Requirement | Rubric / source |
|---|---|---|
| NFR-1 | Search responds within ~300 ms of the last keystroke; auto-save debounce is short enough to feel live without excessive writes; real-time updates need no manual refresh | FRS §5.2 |
| NFR-2 | Account and note passwords bcrypt-hashed; authorization enforced server-side for every protected resource; no secret in client source, bundle, or Git history | FRS §5.1, criteria 23–26 |
| NFR-3 | Deployment reachable for the entire grading window; sync retries safely on reconnect | FRS §5.3, criterion 32 |
| NFR-4 | Consistent navigation, meaningful empty/loading/error feedback, adaptive layouts, accessible input | FRS §5.4, criterion 29 |
| NFR-5 | Documented architecture and state management in `Readme.txt`; tests exercising real behaviour; analyzer-clean code | FRS §5.5, criterion 30 |
| NFR-6 | Core Dart shared across both targets; platform-specific code isolated behind `lib/platform/` | FRS §5.6 |
| NFR-7 | Per-account local cache isolation; documented conflict-handling strategy applied consistently | FRS §5.7, criterion 31 |
| NFR-8 | No low-code generation; no plaintext secrets in history; ≥4 consecutive weeks with ≥2 meaningful commits per member | FRS §5.8, deduction rule |

---

# PART B — BEHAVIOUR DETAIL

## 7. Key concepts

Defined once in
[`../ltc-nevo-knowledge-base/00-glossary.md`](../ltc-nevo-knowledge-base/00-glossary.md) — note,
owner, collaborator, share, label, attachment, note password, unlock grant, pin, AI summary, AI
Q&A, preferences, pending sync, conflict copy. Not restated here.

## 8. Flows and state machines

End-to-end flows (registration, login/refresh, reset, auto-save, browse/search, attachments,
labels, note password, sharing, AI, offline sync, profile) are specified in
[`../ltc-nevo-knowledge-base/05-flows.md`](../ltc-nevo-knowledge-base/05-flows.md).

State machines (auto-save, sync, note protection, session, collaborative reconciliation, AI panel,
connectivity) are in
[`../ltc-nevo-knowledge-base/diagrams/state-machines.md`](../ltc-nevo-knowledge-base/diagrams/state-machines.md).

## 9. Business logic worth stating twice

These are the rules most likely to be implemented subtly wrong.

### 9.1 Effective permission on a note

```
resolve(userId, noteId):
    note missing or soft-deleted            → 404 NOTE_NOT_FOUND
    note.ownerId == userId                  → 'own'   (read + edit + delete + share + password + pin)
    active share, permission EDIT           → 'edit'  (read + edit content/labels/attachments)
    active share, permission READ           → 'read'  (read only)
    otherwise                               → 403 FORBIDDEN
```

Existence is decided before permission, and permission before protection — so a caller with no
relationship to a protected note receives `FORBIDDEN`, not `NOTE_LOCKED`. The lock state of a note
you cannot access is not disclosed.

### 9.2 Note protection gate

Applies to every note-scoped operation, with no exceptions:

```
if note.isPasswordProtected:
    grant = redis.get('note_unlock:{userId}:{noteId}')
    if missing, expired, or mismatched → 423 NOTE_LOCKED
```

Redis unavailable means grants cannot be verified, so protected notes stay locked. The system fails
closed; it never assumes unlocked.

### 9.3 Auto-save

```
keystroke → restart 800 ms debounce
debounce fires OR app pauses OR tab closes → flush now
  write local → PATCH
    200          → Saved HH:MM
    network gone → local write kept, operation queued, badge "pending sync"
    4xx          → Failed, retry offered (never auto-retried — it will fail identically)
```

### 9.4 AI Q&A retrieval

```
1. candidate set = notes owned by caller ∪ notes shared with caller (not deleted)
2. minus every protected note without a live unlock grant
3. rank by full-text relevance against the question, take top-K (K=8)
4. best score below threshold → INSUFFICIENT_CONTEXT, no LLM call
5. build a grounded prompt from those excerpts only
6. answer + references to the notes actually used
```

Steps 1–2 are the criterion. Filtering after the model has answered is not equivalent and does not
satisfy FR-AI-04.

### 9.5 Offline sync and conflicts

```
reconnect → replay queue in enqueue order (CREATE before its own UPDATE before DELETE)
  transient failure → backoff 1s, 4s, 15s, 60s, 300s, then mark failed
  permanent (4xx)   → mark failed, surface Retry
  conflict (server updatedAt > local base)
        → last-write-wins by updatedAt
        → the losing version is kept as a conflict copy the user can open
queue drained → pull changes since lastSyncAt and reconcile
```

Push before pull. Reversing the order lets the pull resurrect notes deleted offline, because the
tombstone has not been sent yet.

---

# PART C — GOVERNANCE

## 10. Dependencies, risks, open questions, change log

**Dependencies and risks:** BRD §5 and §9.

**Open questions:** BRD §11, tracked with answers and dates in
[TRACKING_Nevo.md](TRACKING_Nevo.md) §3. Eight open, none blocking specification — every
requirement above is fully specified. Q-01 (team size) and Q-02 (hosting) must close during Week 1
because they gate the teamwork rule and deployment respectively.

**Change log**

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-08-20 | Initial PRD: 74 build-level requirements across 8 groups covering all 32 rubric criteria, 21 acceptance criteria, business-logic detail for the five rules most likely to be built wrong |
