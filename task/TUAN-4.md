# WEEK 4 — Mon 14/09 → Sun 20/09 · Milestone M3 (Hardening & submission)

> **Goal:** offline sync, the required tests, a verified deployment, the demo video, and the
> submission package.
>
> **Exit condition (M3):** everything in the BRD §12 acceptance checklist ticked.

> [!IMPORTANT]
> **No new features this week.** Anything not built by Sunday 13/09 is cut, not squeezed in. A
> half-finished criterion scores the same as an absent one, but it also destabilises the ones that
> were working.

---

## Task status

| Track | Total | ☐ | 🛠️ | 🔍 | ⛔ | ✅ | ✂️ |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Shared | 8 | 8 | 0 | 0 | 0 | 0 | 0 |
| T1 | 3 | 3 | 0 | 0 | 0 | 0 | 0 |
| T2 | 4 | 4 | 0 | 0 | 0 | 0 | 0 |
| T3 | 2 | 2 | 0 | 0 | 0 | 0 | 0 |
| **Total** | **17** | **17** | **0** | **0** | **0** | **0** | **0** |

---

## Schedule inside the week

| Day | Focus |
|---|---|
| Mon 14/09 – Tue 15/09 | Offline sync completion (`W4-T2-01…03`) — the last real feature work |
| Wed 16/09 | Tests to rubric minimum (`W4-SH-01…02`); freeze features |
| Thu 17/09 | Deployment verification, seed data, grading accounts (`W4-SH-03…05`) |
| Fri 18/09 | **Deployment freeze.** Demo recording (`W4-SH-06`) |
| Sat 19/09 | `Readme.txt`, `Rubric.xlsx`, `Screenshot.png`, package assembly (`W4-SH-07`) |
| Sun 20/09 | Final verification from a clean machine (`W4-SH-08`), submit |

---

## Shared

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W4-SH-01` | ☐ | — | Flutter tests to rubric minimum (FR-XC-02) | ≥3 unit (offline queue, auto-save debounce, permission resolver mapping), ≥3 widget (save-state indicator, grid/list toggle, pinned ordering), ≥1 integration (register → login → create → share → recipient sees it with the right permission, **and a read-only write refused**). Each asserting real behaviour. | `flutter test` and `flutter test integration_test` green; `flutter analyze` clean |
| `W4-SH-02` | ☐ | — | Backend tests | Unit tests for auth, note password, permission resolver, AI retrieval scoping; e2e for the authorization refusals. | `pnpm test` and `pnpm test:e2e` green in CI |
| `W4-SH-03` | ☐ | — | Seed grading data + accounts | At least two accounts so sharing can be demonstrated; notes with labels and attachments; one password-protected note; one shared read-only and one shared editable. Idempotent seed. Closes Q-08. | A grader can see every feature without creating data |
| `W4-SH-04` | ☐ | — | Deployment verification | Both artifacts from a clean clone; public Web URL and `/health`; APK on a device that never had a debug build; CORS origin exact. | The full post-deploy checklist in `playbooks/deployment.md` passes |
| `W4-SH-05` | ☐ | — | Secret + history audit | `git log -p --all | grep -Ei 'key|secret|password'` across both repos; confirm `LLM_API_KEY` is absent from `build/web`; confirm no hash field in any response. | Clean scan, recorded |
| `W4-SH-06` | ☐ | Team | Record `demo.mp4` | All members speaking; architecture, state management, backend, persistence, platforms introduced; every claimed criterion shown; ≥1 AI feature on **both** Web and Android; login, CRUD, attachments, offline sync, sharing with live updates; ≥1080p, clear audio. | Video covers every criterion marked done in `Rubric.xlsx` |
| `W4-SH-07` | ☐ | — | Assemble the submission package | `Rubric.xlsx` (self-assessment, URL, artifact name, repo, grading credentials), `source/` with `.git` and no build output, `release/` with `web-url.txt` + APK, `Readme.txt`, `Screenshot.png`. Folder and ZIP named `id1_fullname1_id2_fullname2`. | Package matches SDD §9 item by item |
| `W4-SH-08` | ☐ | Team | Final verification from a clean machine | Walk BRD §12 end to end on a machine that never built this project. | Every line ticked |

## T1 — Identity & Shell

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W4-T1-01` | ☐ | — | Offline / sync banner (FR-XC-03) | Global banner for offline, pending-sync count, and sync-failure with retry. | AC-19 shows the right banner at each stage |
| `W4-T1-02` | ☐ | — | Four-state audit (FR-XC-01) | Walk all 21 screens: loading, empty, success, error present on each; keyboard focus order, Escape closes dialogs, 48dp touch targets. | Every screen passes; misses fixed, not noted |
| `W4-T1-03` | ☐ | — | Responsive audit (FR-XC-01) | Compact portrait and landscape, tablet, desktop browser, resized window — on both platforms. | No layout breaks at any breakpoint |

## T2 — Notes & Organisation

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W4-T2-01` | ☐ | — | Sync worker (FR-XC-03) | Replay in enqueue order, backoff 1s/4s/15s/60s/300s then failed, 4xx never auto-retried, **push before pull**, reconcile via `GET /sync/changes`. | AC-19 passes: queue drains, badges clear |
| `W4-T2-02` | ☐ | — | Conflict handling (FR-XC-03) | Last-write-wins by `updatedAt`; losing version preserved as a conflict copy the user can open; strategy documented in `Readme.txt`. | Unit test: both versions survive a simulated conflict |
| `W4-T2-03` | ☐ | — | Per-account isolation check (FR-XC-03) | Verify sign-out closes the database and sign-in opens a different file; no note data in any shared store. | AC-20 passes: account switch leaks nothing |
| `W4-T2-04` | ☐ | — | Sync endpoints | `GET /sync/changes?since=`, `POST /sync/push`. | Delta reconciliation works after an outage |

## T3 — Protection, Sharing & AI

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W4-T3-01` | ☐ | — | Security register close-out | Walk SEC-001…SEC-014 in `07-security-permissions.md`; each closes only when its named test passes. Anything without a passing test stays open and is reported honestly. | Register reflects reality, not intent |
| `W4-T3-02` | ☐ | — | AI resilience pass | Provider outage → `AI_PROVIDER_UNAVAILABLE` with retry, not a blank panel; cache serves unchanged notes; quota headroom confirmed before the demo. | AI degrades visibly and recovers |

---

## Final gate (Sun 20/09)

```
□ Every line of BRD §12 ticked
□ Deployment frozen since Fri 18/09 and still green
□ demo.mp4 shows every criterion claimed in Rubric.xlsx — no claim without footage
□ 4 consecutive weeks × ≥2 meaningful commits per member, evidenced in Screenshot.png
□ Package named id1_fullname1_id2_fullname2, contents per SDD §9
□ TRACKING_Nevo.md reflects the true final state — including anything cut
□ Knowledge base final sync: bash scripts/verify-kb-sync.sh passes
```

> [!NOTE]
> **Report what is actually true.** If a criterion was cut, mark it cut in `Rubric.xlsx` and say so.
> A criterion claimed but not demonstrated scores zero anyway, and an inaccurate self-assessment
> costs credibility on the ones that were genuinely done.
