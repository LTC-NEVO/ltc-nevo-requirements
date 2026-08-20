# 4-Week Delivery Plan — Nevo

| Field | Value |
|---|---|
| Window | **Mon 24/08/2026 → Sun 20/09/2026** — 4 consecutive weeks |
| Submission | After M3, per `../../documents/SDD.md` §9 |
| Team | 3 members: Kaka, Chou, Long (Dranov) — **pending Q-01** |
| Requirement source | `../PRD_Nevo_v1.0.md` (74 requirements) · `../BRD_Nevo_v1.0.md` · `../TRACKING_Nevo.md` |
| Architecture source | `../../ltc-nevo-knowledge-base/` — conventions, flows, security model, playbooks |
| Version | v1 — initial plan |
| Updated | 2026-08-20 |

> [!IMPORTANT]
> The four-week window is chosen so that satisfying the **teamwork rule** (≥2 meaningful commits
> per member per week, 4 consecutive weeks) is a by-product of working normally rather than a
> separate chore. Weeks are Mon–Sun to match how the rule is measured.

---

## 1. Where the project actually stands

Verified by reading the repositories on 2026-08-19, not assumed:

| Repo | Real state |
|---|---|
| `ltc-nevo-customer-api` | Stock `nest new`. `app.controller.ts` returns "Hello World!". No layers, no Prisma, no auth, no `.env`, no Dockerfile, no CI. |
| `ltc-nevo-web-app` | Default Flutter counter demo. Two dependencies (`cupertino_icons`, `flutter_lints`). No architecture, no routes, no tests beyond the generated smoke test. Android app id still `com.example.ltc_nevo_web_app`. |
| `ltc-nevo-knowledge-base` | Complete: architecture, conventions, flows, security model, 144 anchored sections, drift tooling passing 9/9. |
| `ltc-nevo-requirements` | This repo — BRD, PRD, tracking, and this plan. |

**This is a from-zero build.** Four weeks, three people, 74 requirements, 32 graded criteria. The
knowledge base means the *design* questions are already answered — the work is implementation, not
discovery.

---

## 2. Work split

Three members, three vertical tracks, each owning a feature end to end (API + client) so nobody
waits on a hand-off across the stack.

| Track | Owner | Scope | Requirements |
|---|---|---|---|
| **T1 — Identity & Shell** | `[TO CONFIRM]` | Auth, session, guards, profile, preferences, app shell, routing, theme | FR-ACC (14) + FR-XC-01 |
| **T2 — Notes & Organisation** | `[TO CONFIRM]` | Note CRUD, editor + auto-save, home views, search, ordering, badges, labels, attachments | FR-NOTE (17) + FR-LBL (7) + FR-ATT (7) |
| **T3 — Protection, Sharing & AI** | `[TO CONFIRM]` | Note password + unlock, sharing + permissions, real-time, AI summary, AI Q&A | FR-PROT (8) + FR-SHR (10) + FR-AI (7) |
| **Shared** | All three | Offline sync (FR-XC-03), tests (FR-XC-02), deployment (FR-XC-04), demo | FR-XC (4) |

> [!WARNING]
> **Assignment is unfilled until Q-01 closes.** Do not start Week 1 without naming an owner per
> track — an unowned track is how a rubric group reaches Week 4 untouched. If the team turns out to
> be two people, merge T3 into T1 and T2 and cut the optional `FR-SHR-08` immediately.

### Avoiding single points of failure

| Area | Primary | Backup |
|---|---|---|
| Auth and guards (everything depends on it) | T1 | T3 — shares the authorization model |
| `NoteAccessGuard` and the permission resolver | T3 | T1 |
| Offline sync | T2 | T1 |
| Deployment and CI | T1 | T2 |
| Demo recording | All | — |

The permission resolver deliberately has two people who understand it. It is the single most
heavily graded piece of logic in the project (criteria 23–26, 2.5 pt).

---

## 3. Critical path

Three things block everything downstream. If they slip, the schedule slips.

1. **Backend foundation** (Week 1) — Prisma schema, the six layer directories, response envelope,
   error factories, config. Nothing else can be built against a scaffold.
2. **Auth end to end** (Week 1) — every other endpoint sits behind `JwtAuthGuard`, and every client
   screen sits behind the router redirect.
3. **`NoteAccessGuard` + permission resolver** (Week 3, designed Week 1) — note protection, sharing,
   and AI scoping all depend on one resolver. Writing it three times is how the three criteria
   diverge and one of them fails.

Deployment (Week 1, verified again in Week 4) is not on the critical path for *building*, but it is
on the critical path for *being graded* — see R3.

---

## 4. Week-by-week

| Week | Dates | Theme | Milestone |
|---|---|---|---|
| [W1](TUAN-1.md) | 24/08 → 30/08 | Foundation: schema, layers, auth, client shell, deployment skeleton | M0 |
| [W2](TUAN-2.md) | 31/08 → 06/09 | Core product: notes, editor, search, labels, attachments, profile | M1 |
| [W3](TUAN-3.md) | 07/09 → 13/09 | Advanced: note password, sharing, real-time, AI | M2 |
| [W4](TUAN-4.md) | 14/09 → 20/09 | Offline sync, tests, hardening, deploy, demo, submission | M3 |

> [!NOTE]
> **Vietnamese National Day, Wed 02/09/2026, falls inside Week 2.** Week 2 is planned at reduced
> capacity. The plan does not assume seven working days in any week.

---

## 5. Definition of Done

A task is done when **all six** hold. Five out of six is not done.

```
1. Code implements the PRD requirement as written — not an approximation of it
2. Convention-compliant per ../../ltc-nevo-knowledge-base/03-conventions.md
3. Tested — at minimum the level the review checklist requires for that change type
4. Works on BOTH platforms where the requirement is platform-facing
5. Knowledge base updated if the change touches anything it documents
   (and any PLANNED marker that is now real flipped to CONFIRMED with a file:line)
6. TRACKING_Nevo.md status advanced
```

For anything touching guards, tokens, permissions, note passwords, or secrets, the blocking
security checklist in
[`../../ltc-nevo-knowledge-base/review-checklist.md`](../../ltc-nevo-knowledge-base/review-checklist.md) §3
applies as well.

---

## 6. Working agreements

| Topic | Agreement |
|---|---|
| Branching | `feat/…`, `fix/…`, `docs/…`, `chore/…` off `main`. No direct pushes to `main`. |
| Commits | `type(scope): imperative summary`. Meaningful commits only — the teamwork rule does not count filler. |
| Review | Every PR reviewed by one other member. Security-relevant changes reviewed against the blocking checklist. |
| Code review tooling | `ltc-nevo-code-reviewer` runs the multi-agent review; its rules mirror the KB conventions. |
| KB updates | Same session as the code change, not "later". |
| Commit check | Every Friday, against `../TRACKING_Nevo.md` §5. |
| Blocked? | Say so the same day. A blocker held quietly for three days costs a quarter of a week. |

---

## 7. Risks carried into execution

Full register in `../BRD_Nevo_v1.0.md` §9. The ones that shape this plan:

| # | Risk | How the plan responds |
|---|---|---|
| R1 | Authorization ends up client-side only | `NoteAccessGuard` designed in W1, built before the sharing UI in W3, and proven by the read-only-write refusal test in the same week |
| R2 | Offline sync retrofitted late | The repository layer is local-first from W2, so offline is the same code path rather than a rewrite in W4 |
| R3 | Hosting sleeps or expires during grading | Deployed in W1 (not W4), non-sleeping plan chosen up front, re-verified in W4, deployment frozen before submission |
| R4 | LLM quota exhausted | Summaries cached by content hash; AI demo recorded in W3, so the video does not depend on live quota in W4 |
| R5 | Demo misses a built criterion | Demo script derived from the rubric checklist in W3, rehearsed in W4 with the checklist open |
| R6 | A member misses a commit week | Friday check, every week, in `../TRACKING_Nevo.md` §5 |
| R8 | Flutter Web-specific breakage found late | Both targets exercised from W1; platform differences isolated in `lib/platform/` from the start |

---

## 8. What this plan deliberately does not do

- **No optional enhancements.** Version history, PDF export, and a desktop shortcut layer are out of
  scope (BRD §5). They may not weaken a mandatory requirement, and there is no spare capacity.
- **No second native target.** Android only, alongside Web.
- **No polish before correctness.** A criterion that works plainly scores the same as one that
  works beautifully; a criterion that is beautiful but unenforced scores zero.
