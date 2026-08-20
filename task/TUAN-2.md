# WEEK 2 — Mon 31/08 → Sun 06/09 · Milestone M1 (Core product)

> **Goal:** Nevo becomes usable. Notes can be created, edited with auto-save, organised with
> labels, searched, and given attachments — on both platforms.
>
> **Exit condition (M1):** notes CRUD with auto-save, home grid/list with search and label filter,
> attachments, profile and preferences, all working against the deployed API.

> [!NOTE]
> **Wed 02/09 is Vietnamese National Day.** This week is planned at reduced capacity — 4 working
> days, not 6. Do not backfill the holiday with weekend work; the commit rule needs sustainable
> weeks, not one heroic one.

---

## Task status

Symbols and DoD as in [TUAN-1.md](TUAN-1.md).

| Track | Total | ☐ | 🛠️ | 🔍 | ⛔ | ✅ | ✂️ |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Shared | 2 | 2 | 0 | 0 | 0 | 0 | 0 |
| T1 | 4 | 4 | 0 | 0 | 0 | 0 | 0 |
| T2 | 9 | 9 | 0 | 0 | 0 | 0 | 0 |
| T3 | 3 | 3 | 0 | 0 | 0 | 0 | 0 |
| **Total** | **18** | **18** | **0** | **0** | **0** | **0** | **0** |

---

## Critical path

**`W2-T2-02` (editor + auto-save)** is this week's spine — FR-11, FR-12, and FR-14 are three
separate rubric criteria riding on one screen, and the lifecycle flush in `W2-T2-03` is the part
teams usually discover is missing during the demo.

Close **Q-05** (note content format: plain text or Markdown) by **Tue 01/09** — the editor, the AI
summary panel, and the XSS posture all depend on the answer.

---

## Shared

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W2-SH-01` | ☐ | Team | Close Q-05 and Q-06 | Note content format (plain text vs Markdown) and attachment size caps, the latter constrained by whatever the hosting plan chosen in Week 1 actually allows. | Both recorded in `../TRACKING_Nevo.md` §3 |
| `W2-SH-02` | ☐ | — | CI for both repos | GitHub Actions: API (install → lint → test → build), client (`flutter analyze` → `flutter test` → `build web`). Closes GAP-105 and GAP-205. | Both workflows green on `main` |

## T1 — Identity & Shell

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W2-T1-01` | ☐ | — | Profile view + edit (FR-ACC-13) | `GET/PATCH /users/me`, avatar upload with server-side type and size validation, default avatar when none set. | AC covering profile passes on both platforms |
| `W2-T1-02` | ☐ | — | Change password (FR-ACC-14) | Current password required; success invalidates other sessions; client re-establishes its session safely. | Wrong current password changes nothing |
| `W2-T1-03` | ☐ | — | Preferences (FR-ACC-14) | `GET/PUT /users/me/preferences`; theme, font size, note colour, default view applied through `ThemeData`, loaded during bootstrap so the first frame is already correct. | AC-5 passes: set on Web, verified on Android |
| `W2-T1-04` | ☐ | — | Activation banner (FR-ACC-07) | Persistent banner in the shell on Home and children while unverified; disappears after activation on both platforms. | AC-2 passes |

## T2 — Notes & Organisation

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W2-T2-01` | ☐ | — | Note CRUD endpoints | `POST /notes`, `GET /notes/:id`, `PATCH /notes/:id`, `DELETE /notes/:id` (soft, cascading in one transaction), pin/unpin. Only title and content mandatory. | Endpoints match the PRD; soft delete cascades |
| `W2-T2-02` | ☐ | — | **[CRITICAL]** Editor with auto-save (FR-NOTE-03…05) | **One** screen serving `/notes/new` and `/notes/:noteId`. 800 ms debounce, no Save button anywhere, three visible save states. | AC-6 first half passes; no save button exists |
| `W2-T2-03` | ☐ | — | **[CRITICAL]** Lifecycle flush (FR-NOTE-06, 07) | `AppLifecycleListener` flush on pause/inactive/detached; `beforeunload`-equivalent on Web. | AC-6 second half: background mid-typing, reopen, text is there — on both platforms |
| `W2-T2-04` | ☐ | — | Home grid + list (FR-NOTE-01, 02) | Grid default, list toggle, choice written to preferences, responsive columns at all three breakpoints. | AC-7 first half passes |
| `W2-T2-05` | ☐ | — | Ordering + badges (FR-NOTE-09…12) | Pinned-first ordering rendered as the API returns it; pinned / locked / shared / pending / failed badges co-existing on one card, in both views. | AC-7 second half passes |
| `W2-T2-06` | ☐ | — | Live search (FR-NOTE-13, 14) | 300 ms debounce, title + content, no Search button; offline path querying Drift. | AC-8 passes online and offline |
| `W2-T2-07` | ☐ | — | Delete confirmation (FR-NOTE-08) | Explicit confirm dialog before any deletion; cancel does nothing. | Deleting without confirming is impossible |
| `W2-T2-08` | ☐ | — | Labels (FR-LBL-01…07) | CRUD, per-owner case-insensitive uniqueness, rename by reference, delete removes links only, filter by multiple labels, labels private to owner. | AC-9 passes |
| `W2-T2-09` | ☐ | — | Attachments (FR-ATT-01…07) | Upload with server-side validation, gallery/camera/file picker behind `lib/platform/`, camera hidden on Web, cancel is a no-op, denied permission explained, pre-signed download gated on the parent note. | AC-10 passes on Android and Web |

## T3 — Protection, Sharing & AI (build-up)

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W2-T3-01` | ☐ | — | `NoteAccessGuard` + `@NotePermission` | Wire the Week 1 resolver into a guard; apply it to every existing `:id` route. Ordering: existence → permission → protection. | Guard applied to every note route; e2e proves 404 before 403 |
| `W2-T3-02` | ☐ | — | Share domain + repository | `NoteShareEntity`, unique `(noteId, recipientUserId)`, repository with `isDeleted` filtering. Prepares Week 3. | Unit tests cover upsert-on-reshare |
| `W2-T3-03` | ☐ | — | `LlmService` behind `ILlmProvider` | Provider client, 30 s timeout, one retry on 429/5xx, `AI_PROVIDER_UNAVAILABLE` after that. **The only file that reads `LLM_API_KEY`.** Depends on Q-03. | A grep for `LLM_API_KEY` outside this file returns nothing |

---

## End-of-week check (Sun 06/09)

```
□ M1 exit conditions met
□ A note survives: type → background app → kill → reopen (both platforms)
□ Search returns a content-only match without pressing anything
□ Labels rename and delete without damaging notes
□ Attachments upload and download on both platforms; oversized file refused server-side
□ Preferences set on one platform apply on the other
□ NoteAccessGuard on every note-scoped route
□ CI green on both repos
□ Every member has ≥2 meaningful commits (../TRACKING_Nevo.md §5)
□ KB updated; PLANNED → CONFIRMED for everything now real
```
