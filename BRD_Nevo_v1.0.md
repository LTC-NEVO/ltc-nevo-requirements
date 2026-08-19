---
document_type: Business Requirement Document
project: Nevo — Cross-Platform Note Management Application
team: LTC
version: "1.0"
status: Draft
last_updated: "2026-08-20"
upstream: ["../documents/FRS.md", "../documents/SDD.md", "../documents/Screen-Inventory.md"]
---

# BRD — Business Requirement Document — `Nevo v1.0`

> **Purpose:** why this project exists, what success means, what is in and out of scope, and what
> could sink it. Product behaviour lives in [PRD_Nevo_v1.0.md](PRD_Nevo_v1.0.md); architecture lives
> in [`../ltc-nevo-knowledge-base/`](../ltc-nevo-knowledge-base/).
>
> **Conventions:**
> - `[TO CONFIRM]` = information still missing (tracked in §11).
> - Living document — change via PR and bump the version in §0.

---

## 0. Document history

| Version | Date | Author | Change |
|---|---|---|---|
| 1.0 | 2026-08-20 | Team LTC | Initial BRD derived from FRS v1.1, SDD v1.1, and the Screen Inventory v2.1 |

---

## 1. Executive Summary

Nevo is a cross-platform note-taking application: one Flutter codebase producing a publicly
deployed Web build and a submitted Android APK, backed by a custom NestJS API that owns
authentication, authorization, real-time collaboration, and all AI features.

It is the **final project for course 503107**. That framing matters more than it looks: the grading
rubric's 32 criteria are not a quality wish-list, they are the acceptance contract, and a criterion
that is not demonstrated in the submitted demo video scores zero regardless of how well it was
built. This document therefore treats "the rubric" and "the customer" as the same stakeholder.

The product itself is a real one, not a toy: notes with attachments and labels, per-note password
protection, sharing with read/edit permissions and live collaborative editing, LLM-powered
summarisation and grounded question answering, and full offline operation with conflict-handled
sync.

---

## 2. Business Context

**Situation.** Course 503107 requires a cross-platform application demonstrating a single Flutter
codebase across two platforms, a real backend with server-side authorization, offline capability,
and AI integration. The team (LTC) has prior production experience with NestJS clean architecture
(`swt-lpi-administration-api`), which is why a custom backend was chosen over a BaaS.

**Problem being solved by the product.** Personal notes tend to live either in a fully local app
(safe, but stranded on one device) or in a fully cloud app (available everywhere, but unusable
offline and indiscriminately visible to anyone with the account). Nevo takes the position that
those are not the only two options: notes are cached locally and remain fully readable and editable
offline, while individual notes can carry their own password so that account access does not
automatically mean access to everything.

**Problem being solved by the project.** A four-week window, three people, and a fixed submission
date with no extension. The risk is not "can we build a note app" — it is finishing 32 gradable
criteria, deployed and demonstrable, without discovering in the last week that authorization was
never actually enforced server-side.

**Current state (2026-08-20).** Both repositories are framework scaffolds. `ltc-nevo-customer-api`
is a stock `nest new`; `ltc-nevo-web-app` is the default Flutter counter demo. Zero of 32 criteria
are implemented. The knowledge base and specifications are complete; the code is not started.

---

## 3. Business Goals, Objectives & KPI

| # | Goal | Measured by | Target |
|---|---|---|---|
| G1 | Satisfy the grading rubric in full | Criteria implemented **and demonstrated in `demo.mp4`** | 32/32 |
| G2 | Prove authorization is real, not cosmetic | Integration test: a read-only collaborator's direct `PATCH` is refused | Passing, in CI |
| G3 | Stay deployed for the whole grading window | `GET /health` and the Web URL reachable | 100% of the grading period |
| G4 | Avoid the teamwork deduction | Consecutive calendar weeks with ≥2 meaningful commits per member | 4/4 weeks |
| G5 | Ship a reproducible build | A clean clone builds Web + APK using only the documented commands | Verified twice: mid-project and pre-submission |
| G6 | Keep the demo honest | Rubric self-assessment matching what the video actually shows | No criterion claimed but unshown |

G1 is the outcome; G2–G6 are the failure modes that most commonly cost points on projects that
otherwise "work on my machine".

---

## 4. High-level Business Capabilities

| # | Capability | Rubric criteria | Weight |
|---|---|---|---|
| C1 | Account management — register, activate, sign in, reset password, profile, preferences | 1–8 | 2.0 pt |
| C2 | Note management — CRUD with auto-save, attachments, labels, search, pinning, status badges | 9–22 | 3.5 pt |
| C3 | Advanced notes — per-note password protection, sharing with permissions, real-time collaboration, AI summary, AI Q&A | 23–28 | 2.5 pt |
| C4 | Cross-cutting quality — adaptive UI, documented architecture with tests, offline sync, public deployment | 29–32 | 2.0 pt |
| — | Teamwork evidence | deduction rule | −0.5 pt if unmet |

C2 carries the most weight and the least risk. C3 carries the most risk: note protection, share
enforcement, and AI scoping are the three places where a plausible-looking implementation can still
fail the criterion because enforcement was client-side.

---

## 5. Scope, Assumptions & Dependencies

### In scope (v1.0)

| Area | Included |
|---|---|
| Client | One Flutter project → deployed Web build + Android release APK |
| Backend | `ltc-nevo-customer-api` — NestJS 11, clean architecture, PostgreSQL, Redis, object storage, SMTP, LLM proxy |
| Auth | Email/password, activation email, password reset by link or OTP, JWT with rotating refresh |
| Notes | CRUD, auto-save, attachments (image/video/file), labels, live search, pinning, status badges |
| Protection | Per-note password with server-enforced unlock grants |
| Collaboration | Share by email at read/edit level, "Shared with me", live collaborative editing |
| AI | Note summarisation and retrieval-grounded Q&A, both proxied server-side |
| Offline | Local cache, optimistic writes, queued sync, documented conflict handling |
| Delivery | Public HTTPS deployment, submitted APK, demo video, submission package |

### Out of scope (v1.0)

| Excluded | Why |
|---|---|
| A second native target (iOS, Windows, macOS) | The brief requires exactly one native artifact besides Web. Android is chosen; the other platform folders stay unused. |
| An admin console | No operator role exists in the requirements. |
| Note version history, PDF/Markdown export, desktop keyboard-shortcut layer | Listed as optional enhancements in FRS §5.9. Explicitly not planned — they may not weaken any mandatory requirement, and there is no spare capacity. |
| Vector database / embeddings | Postgres full-text retrieval is sufficient at this corpus size (KB decision AD-4). |
| Multi-tenancy, org accounts, billing | Not a product requirement. |
| Real-time cursor presence / OT-CRDT merge | FR-26 requires live propagation, not character-level co-editing. Last-write-wins with a conflict copy is the documented strategy. |

### Assumptions

| # | Assumption | If wrong |
|---|---|---|
| A1 | Hosting for API + Postgres + Redis + bucket stays free/affordable for the grading window | Deployment must move; FR-32 at risk (R3) |
| A2 | An LLM API key with sufficient quota is available for the whole period | FR-27/28 degrade; summaries cached but Q&A blocked (R4) |
| A3 | All three members can commit every week | Teamwork deduction (R6) |
| A4 | The rubric is interpreted as written in `../documents/FRS.md` | Re-derivation of PRD requirements |
| A5 | The instructor grades against the deployed Web build plus the submitted APK | Deployment freeze must hold |

### Dependencies

| # | Dependency | Type | Owner |
|---|---|---|---|
| D1 | `../documents/` specifications | Internal, done | Long (Dranov) |
| D2 | `../ltc-nevo-knowledge-base/` architecture and conventions | Internal, done | Team |
| D3 | Managed Postgres, Redis, S3-compatible bucket, SMTP | External | `[TO CONFIRM]` — provider not yet chosen (Q-02) |
| D4 | LLM provider account and key | External | `[TO CONFIRM]` (Q-03) |
| D5 | GitHub repositories with full history for all members | Internal | Team |

---

## 6. Stakeholders & RACI

| Stakeholder | Interest |
|---|---|
| Course instructor / grader | Verifies all 32 criteria on the deployed Web build and the submitted artifact |
| Team LTC — Kaka, Chou, Long (Dranov) | Build, test, deploy, demo |
| End user (persona) | An individual keeping personal notes across phone and browser, some of them private, some shared |

| Activity | Kaka | Chou | Long (Dranov) |
|---|---|---|---|
| Requirements and specifications | C | C | **A/R** |
| Backend implementation | `[TO CONFIRM]` | `[TO CONFIRM]` | `[TO CONFIRM]` |
| Client implementation | `[TO CONFIRM]` | `[TO CONFIRM]` | `[TO CONFIRM]` |
| Deployment and submission package | C | C | **A** |
| Demo video | R | R | R |

> [!NOTE]
> **Assignment is deliberately unfilled.** `../documents/Screen-Inventory.md` leaves every
> Owner/Assignee as *Unassigned* and says the team should claim rows during sprint planning —
> nothing was assumed on anyone's behalf there, and nothing is assumed here either. Fill this in at
> the Week 1 planning session, then mirror it into `task/00-TONG-QUAN.md`.

> [!WARNING]
> **Team size is ambiguous and it affects a graded rule.** `Screen-Inventory.md` names three people
> (Kaka, Chou, Long/Dranov), but the submission package format in `SDD.md` §9 is
> `id1_fullname1_id2_fullname2` — two IDs. The teamwork rule requires ≥2 meaningful commits **per
> member** per week for 4 consecutive weeks, so the member list must be settled before Week 1 ends,
> not at submission. Tracked as **Q-01**.

---

## 7. Business Process (As-is → To-be)

**As-is.** The user keeps notes in a device-local app or a cloud notes app. Local notes do not
follow them to another device. Cloud notes are unavailable when connectivity drops, and every note
is visible to anyone holding the account session.

**To-be.**

```
Register → activate (non-blocking) → write notes on any device
                                        │
        ┌───────────────────────────────┼────────────────────────────────┐
        ▼                               ▼                                ▼
  organise                        protect                          collaborate
  labels, pins, search       per-note password                 share read/edit
  attachments                enforced server-side              live co-editing
        │                               │                                │
        └───────────────────────────────┴────────────────────────────────┘
                                        ▼
                              ask the corpus questions
                        (AI answers only from notes you may read)
                                        ▼
                        works offline; syncs on reconnect
```

The step that distinguishes Nevo from the as-is is the middle one: protection is per-note and
enforced by the server, so losing your unlocked session does not expose your locked notes, and a
collaborator you granted read access cannot write no matter what request they construct.

---

## 8. Constraints & Compliance

| # | Constraint | Source | Consequence if broken |
|---|---|---|---|
| K1 | Single Flutter project for both platforms — no per-platform replacement app | FRS §2.4 | Rubric failure |
| K2 | No secrets in client source, client bundle, or Git history | FRS §2.4, §5.8 | Rubric failure; requires key rotation |
| K3 | Authorization enforced server-side, never by hiding UI | FRS §2.4 | Criteria 23–26 fail |
| K4 | Passwords (account and per-note) never stored in plaintext | FRS §2.4 | Rubric failure |
| K5 | No low-code/no-code generation of the application | FRS §2.4 | Rubric failure |
| K6 | ≥4 consecutive weeks, ≥2 meaningful commits per member per week | FRS §5.8 | −0.5 pt, whole team, unrecoverable |
| K7 | Deployment reachable for the entire grading period | FRS FR-32 | Criterion 32 fails, and graders cannot verify others |
| K8 | Every claimed criterion demonstrated in `demo.mp4` | FRS §7 | Criterion scores zero even if implemented |

K6 and K8 are the two that are lost by omission rather than by error. Both are tracked weekly.

---

## 9. Business Risks & Mitigations

| # | Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R1 | Authorization implemented client-side only; discovered late | Medium | **Critical** — 2.5 pt | `NoteAccessGuard` before the happy path, not after; the read-only-write refusal test lands in Week 3 and runs in CI | `[TO CONFIRM]` |
| R2 | Offline sync started too late and destabilises everything it touches | Medium | High — 0.5 pt + regressions | Repository layer designed local-first from Week 1, so offline is the same code path rather than a retrofit | `[TO CONFIRM]` |
| R3 | Hosting sleeps, expires, or exceeds a free tier during grading | Medium | **Critical** — blocks verification of every criterion | Choose a non-sleeping plan in Week 1 (Q-02); uptime ping against `/health`; deployment freeze before submission | `[TO CONFIRM]` |
| R4 | LLM quota exhausted or provider outage | Medium | Medium — 1.0 pt | Cache summaries by content hash; record a demo of AI features early so the video does not depend on live quota | `[TO CONFIRM]` |
| R5 | Demo video misses a criterion that was actually built | Medium | High — silent point loss | Demo script derived from the rubric checklist, rehearsed in Week 4 with the checklist open | `[TO CONFIRM]` |
| R6 | A member misses a commit week | Medium | Medium — 0.5 pt, unrecoverable | Weekly Friday check in `TRACKING_Nevo.md`; anyone at risk pairs on a small task rather than waiting | Team |
| R7 | Scope creep into optional enhancements while mandatory criteria are open | Medium | High | FRS §5.9 items are explicitly out of scope in §5; adding one requires a BRD change, not a PRD edit | Team |
| R8 | Flutter Web-specific breakage found late (secure storage, file picking, WASM SQLite) | Medium | High — parity is graded | Both targets exercised from Week 1; platform differences isolated in `lib/platform/` and listed in the KB |  `[TO CONFIRM]` |

---

## 10. Timeline & Milestones

Four consecutive calendar weeks, Monday to Sunday, chosen to satisfy the teamwork rule (K6) as a
side effect of the delivery plan rather than as a separate exercise.

| Milestone | Date | Exit condition |
|---|---|---|
| M0 — Foundation ready | Sun 30/08/2026 | Schema migrated, layer skeleton compiling, envelope + error factories, auth end to end, client shell routing to a real login |
| M1 — Core product | Sun 06/09/2026 | Notes CRUD with auto-save, home grid/list, search, labels, attachments, profile and preferences |
| M2 — Advanced features | Sun 13/09/2026 | Note password, sharing with enforced permissions, real-time updates, both AI features |
| M3 — Hardening + submission | Sun 20/09/2026 | Offline sync, tests at rubric minimum, deployed and verified, demo recorded, package assembled |

> [!NOTE]
> **Vietnamese National Day falls on Wednesday 02/09/2026**, inside Week 2. Week 2 is planned at
> reduced capacity for that reason; the plan does not assume seven working days.

---

## 11. Glossary & Open Questions

Domain terms are defined once, in
[`../ltc-nevo-knowledge-base/00-glossary.md`](../ltc-nevo-knowledge-base/00-glossary.md). They are
not repeated here — a second glossary is a second thing to keep true.

| # | Open question | Blocks | Owner | Status |
|---|---|---|---|---|
| Q-01 | Is the team two or three members? Screen-Inventory names three; the submission folder format takes two IDs | K6 teamwork tracking, RACI, task assignment | Team | ☐ Open — **must close in Week 1** |
| Q-02 | Which hosting provider for API, Postgres, Redis, and object storage? | R3, deployment tasks, FR-32 | `[TO CONFIRM]` | ☐ Open — **must close in Week 1** |
| Q-03 | Which LLM provider and key, and what quota ceiling? | FR-27, FR-28, R4 | `[TO CONFIRM]` | ☐ Open |
| Q-04 | Password reset by emailed link, emailed OTP, or both? FRS allows either; the Screen Inventory routes `/reset-password/:token` and mentions OTP entry | FR-ACC-09…11 | `[TO CONFIRM]` | ☐ Open |
| Q-05 | Note content format — plain text or Markdown? Affects the editor, the AI summary panel, and XSS posture | FR-NOTE-03, FR-AI-02 | `[TO CONFIRM]` | ☐ Open |
| Q-06 | Attachment size caps confirmed at 10 MB image / 50 MB video / 20 MB file? Hosting plan may force lower | FR-ATT-02 | `[TO CONFIRM]` | ☐ Open |
| Q-07 | Is a share notification email in scope? FRS §5.9 lists it as optional but the KB already designs `MailService` for it | FR-SHR-08 | Team | ☐ Open |
| Q-08 | Grading account credentials and seeded demo data — who creates them and when? | Submission package | `[TO CONFIRM]` | ☐ Open |

Answers are recorded in [TRACKING_Nevo.md](TRACKING_Nevo.md) §3 with the date, and the requirements
they block are unblocked in the same edit.

---

## 12. Acceptance Checklist & Sign-off

The project is accepted when all of the following hold. This is deliberately the same list the
instructor will effectively walk.

```
□ All 32 rubric criteria implemented, each traceable to a PRD requirement and a test or demo step
□ Deployed Web build reachable over HTTPS; login works from a clean browser profile
□ GET /health returns ok with db and redis up, from the public host
□ Android release APK installs and runs on a device that never had a debug build
□ A read-only collaborator's direct API write is refused — proven by an integration test
□ Every operation on a locked note is refused without an unlock grant
□ No secret in client source, client bundle, or Git history
□ ≥3 unit, ≥3 widget, ≥1 integration test passing; flutter analyze clean
□ Offline: previously loaded notes readable, edits queue, queue drains on reconnect
□ Switching accounts on one device shows none of the previous account's notes
□ demo.mp4 shows every claimed criterion, ≥1 AI feature on both platforms, all members speaking
□ Rubric.xlsx self-assessment matches what the video actually demonstrates
□ 4 consecutive weeks × ≥2 meaningful commits per member, evidenced in Screenshot.png
□ Submission package assembled per SDD §9 and named id1_fullname1_id2_fullname2
```

| Role | Name | Date | Signature |
|---|---|---|---|
| Team lead | `[TO CONFIRM]` | | |
| Member | `[TO CONFIRM]` | | |
| Member | `[TO CONFIRM]` | | |
