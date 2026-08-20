# WEEK 3 — Mon 07/09 → Sun 13/09 · Milestone M2 (Advanced features)

> **Goal:** the 2.5-point group. Per-note passwords, sharing with enforced permissions, live
> collaboration, and both AI features.
>
> **Exit condition (M2):** a note can be locked and only opened with its own password; a note can be
> shared read-only or editable with enforcement proven by test; two browsers editing a shared note
> see each other live; summary and grounded Q&A both work with references.

> [!IMPORTANT]
> **This is the highest-risk week.** Criteria 23–26 fail whenever enforcement turns out to be
> client-side. Every task here that says "enforced server-side" means there is a test proving a
> direct API call is refused — not that a button is hidden.

---

## Task status

| Track | Total | ☐ | 🛠️ | 🔍 | ⛔ | ✅ | ✂️ |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Shared | 3 | 3 | 0 | 0 | 0 | 0 | 0 |
| T1 | 2 | 2 | 0 | 0 | 0 | 0 | 0 |
| T2 | 3 | 3 | 0 | 0 | 0 | 0 | 0 |
| T3 | 10 | 10 | 0 | 0 | 0 | 0 | 0 |
| **Total** | **18** | **18** | **0** | **0** | **0** | **0** | **0** |

---

## Critical path

1. **`W3-T3-03` — the read-only-write refusal test.** The single most important test in the
   project. Write it the same day the share endpoints land, not at the end of the week.
2. **`W3-T3-06` — grant revocation on password change.** Easy to forget, and it is the difference
   between criterion 24 passing and failing.
3. **`W3-SH-03` — record the AI demo footage this week**, while quota is known-good. Risk R4 is
   about the video depending on live quota during Week 4.

---

## Shared

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W3-SH-01` | ☐ | Team | Close Q-07 | Decide whether the share notification email (`FR-SHR-08`, SHOULD) is in or out. If Week 3 is tight, cut it — it is optional in the FRS and costs nothing to drop. | Recorded in `../TRACKING_Nevo.md` §3 |
| `W3-SH-02` | ☐ | — | Rate limiting | `@nestjs/throttler` on login (10/15 min per IP), note unlock (5/15 min per user+note), `/ai/ask` (20/h per user). | e2e: the 11th login attempt returns 429 |
| `W3-SH-03` | ☐ | Team | Draft + record the demo script | Script derived from the rubric checklist, one line per criterion. **Record the AI segments this week** while quota is healthy. | Script exists; AI footage captured |

## T1 — Identity & Shell

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W3-T1-01` | ☐ | — | Socket client + reconnection | `socket_io_client`, JWT in the handshake, re-join rooms after reconnect, refetch the open note once to close the gap. | Live updates survive a dropped connection |
| `W3-T1-02` | ☐ | — | Session hardening | `UserStatus` re-checked on refresh; logout drops unlock grants; account switch closes the Drift database. | e2e: an inactive user cannot refresh |

## T2 — Notes & Organisation

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W3-T2-01` | ☐ | — | Shared-with-me screen (FR-SHR-06) | `/shared` listing permission level, sharer, and share timestamp; edit-permission notes open into the collaborative editor. | AC covering shared-with-me passes |
| `W3-T2-02` | ☐ | — | Editor: remote-change handling (FR-SHR-10) | Clean buffer → apply the patch; dirty buffer → banner offering reload, buffer preserved. | AC-14 passes: B's unsaved work is never destroyed |
| `W3-T2-03` | ☐ | — | Note-password dialog (FR-PROT client side) | Set / change / remove / unlock states; grant held in memory only; locked note content never written to Drift. | Locked note has no cached body — verified by test |

## T3 — Protection, Sharing & AI

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W3-T3-01` | ☐ | — | Note password: set / change / remove (FR-PROT-01, 02, 06) | bcrypt cost 10; change and remove both require the current note password first. | Wrong current password changes nothing |
| `W3-T3-02` | ☐ | — | **[CRITICAL]** Unlock grants (FR-PROT-03, 04, 05) | `POST /notes/:id/unlock`, Redis grant TTL 15 min, guard refuses every operation without it — view, edit, delete, share, summarise, **and AI retrieval**. Fails closed when Redis is down. | AC-11 passes for every listed operation |
| `W3-T3-03` | ☐ | — | **[CRITICAL]** Share endpoints + enforcement (FR-SHR-01…05, 07) | Share by email at READ/EDIT, owner-only control, revocation immediate, re-share updates. **Write the read-only-write refusal test in the same PR.** | AC-13 passes: `READ` collaborator's direct `PATCH` returns 403 |
| `W3-T3-04` | ☐ | — | Share dialog (FR-SHR-01…03) | Add recipients by email, assign and change permission, revoke; unregistered email and self-share both refused with the right message. | AC-15 passes |
| `W3-T3-05` | ☐ | — | Realtime gateway (FR-SHR-09) | `NevoGateway` on `/realtime`; handshake verifies JWT and disconnects invalid sockets **before** any room join; `join_note` uses the shared resolver; broadcast after persistence, never before. | AC-14 passes; unauthenticated socket disconnected |
| `W3-T3-06` | ☐ | — | **[CRITICAL]** Grant revocation (FR-PROT-07) | Changing or removing a note password deletes `note_unlock:*:{noteId}` for **every** user, and emits `note:locked` so viewers drop cached content. | AC-12 passes: an older grant stops working immediately |
| `W3-T3-07` | ☐ | — | AI summary (FR-AI-01…03) | `POST/GET /notes/:id/ai/summary`, cache by content hash, regenerate bypassing cache, staleness flag. **Never writes `Note.content`.** | AC-16 passes: content byte-identical before and after |
| `W3-T3-08` | ☐ | — | **[CRITICAL]** AI Q&A retrieval scoping (FR-AI-04, 05) | Candidate set = owned ∪ shared, minus locked notes, computed **in SQL before the prompt is built**. Never a post-filter on the answer. | AC-17 passes: another user's note is never cited |
| `W3-T3-09` | ☐ | — | AI Q&A answers + references (FR-AI-06, 07) | Grounded prompt from retrieved excerpts; references that open the note on both platforms; explicit `INSUFFICIENT_CONTEXT` below the relevance threshold. | AC-18 passes on Web and Android |
| `W3-T3-10` | ☐ | — | AI panel + Q&A screen | Summary panel beside the note body (never inside it); `/ask` screen with all four states and tappable reference chips. | Panels render all four states |

---

## End-of-week check (Sun 13/09)

```
□ M2 exit conditions met
□ Locked note refuses view, edit, delete, share, summarise, AND AI retrieval
□ Changing a note password kills every outstanding grant immediately
□ READ collaborator's direct PATCH returns 403 — test passing in CI
□ Revoking a share takes effect on the next request and evicts the socket
□ Two browsers on one shared note see each other's changes without refresh
□ Unauthenticated socket disconnected before joining any room
□ AI answer never cites a note outside the caller's visible set
□ AI references open the note on both platforms
□ AI demo footage recorded while quota is healthy
□ Every member has ≥2 meaningful commits (../TRACKING_Nevo.md §5)
□ KB security register updated: SEC-004…SEC-010 re-checked against real tests
```
