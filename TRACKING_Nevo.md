---
document_type: Tracking Checklist
tracks: ["BRD_Nevo_v1.0.md", "PRD_Nevo_v1.0.md", "../documents/FRS.md", "../ltc-nevo-knowledge-base/09-requirements/traceability-matrix.md"]
last_updated: "2026-08-20"
updated_by: "Team LTC — initial baseline before Week 1"
---

# Tracking Checklist — Nevo v1.0

> Status of all **74 build-level requirements** (8 groups) from the PRD, the **8 open questions**
> that gate them, the **4 project milestones** from the BRD, and the **weekly commit compliance**
> that carries a scored deduction. One file, updated by whoever changes the thing being tracked.
>
> **Baseline 2026-08-20:** implementation has not started. Both repositories are scaffolds. Every
> requirement is 📝 Specified; none is ✅ Confirmed, 🛠️ Building, or beyond.

## How to update this file

1. When a requirement changes stage, update its **Status** tag in §2 — not just a checkbox. The tag
   says which stage it is in (📝 → 🔍 → ✅ → 🛠️ → 🧪 → 🚀), and stages are not skipped.
2. When an open question in §3 is answered, record the **answer and the date in that row**, then
   unblock whatever it held up in §2 in the same edit. A resolved question with a still-blocked
   requirement means one of the two is lying.
3. After editing, update `last_updated` / `updated_by` in the frontmatter and the rollup in §1.
4. The rollup in §1 is derived, not independent. If it disagrees with §2, §2 is right and the
   rollup was not recomputed.
5. §5 (commit compliance) is checked **every Friday**, not reconstructed at the end. It cannot be
   fixed retroactively.

**Status legend**

| Tag | Meaning |
|---|---|
| 🚫 Blocked | Not enough information to specify — see the linked open question in §3 |
| 📝 Specified | Written in the PRD, not yet reviewed |
| 🔍 In review | Under requirement or technical review |
| ✅ Confirmed | Reviewed; ready to build |
| 🛠️ Building | Implementation in progress |
| 🧪 Testing | Built; under test or verification |
| 🚀 Done | Verified, deployed, and demonstrable |
| ❌ Out of scope | Confirmed not being built in v1.0 |

---

## 1. Rollup

**Current phase: Requirements complete, implementation not started.**

| Group | Requirements | 🚫 | 📝 | ✅+ | Rubric criteria | % specified |
|---|:-:|:-:|:-:|:-:|:-:|:-:|
| FR-ACC — Account | 14 | 0 | 14 | 0 | 1–8 | 100% |
| FR-NOTE — Notes | 17 | 0 | 17 | 0 | 9–14, 17–19 | 100% |
| FR-LBL — Labels | 7 | 0 | 7 | 0 | 20–22 | 100% |
| FR-ATT — Attachments | 7 | 0 | 7 | 0 | 15–16 | 100% |
| FR-PROT — Note protection | 8 | 0 | 8 | 0 | 23–24 | 100% |
| FR-SHR — Sharing & collaboration | 10 | 0 | 10 | 0 | 25–26 | 100% |
| FR-AI — AI features | 7 | 0 | 7 | 0 | 27–28 | 100% |
| FR-XC — Cross-cutting | 4 | 0 | 4 | 0 | 29–32 | 100% |
| **Total** | **74** | **0** | **74** | **0** | **32** | **100%** |

| Rubric coverage | Value |
|---|---|
| Criteria specified | 32 / 32 |
| Criteria implemented | **0 / 32** |
| Points at risk today | **10.0 / 10.0** |

> [!NOTE]
> 100% specified and 0% implemented is the honest state on 2026-08-20. The number that moves during
> the project is the implemented count, and it moves in §2 first.

---

## 2. Status by requirement

Requirement text is authoritative in [PRD_Nevo_v1.0.md](PRD_Nevo_v1.0.md) §4. This section tracks
**stage**, not wording — if a requirement's wording changes, change it in the PRD, not here.

### FR-ACC — Account (14) — rubric 1–8

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-ACC-01…05 | Registration, bcrypt hash, duplicate handling, auto-login, activation email | 1 | 📝 | — | Milestone M0 |
| FR-ACC-06…07 | Unverified accounts keep full access; persistent banner | 2 | 📝 | — | No `isEmailVerified` gate anywhere |
| FR-ACC-08…10 | Route protection, login errors, logout + refresh | 3 | 📝 | — | Critical path — everything depends on it |
| FR-ACC-11…12 | Password reset, no auto-login | 4 | 📝 | — | Link vs OTP pending **Q-04** |
| FR-ACC-13 | Profile view/edit, avatar validation | 5, 6 | 📝 | — | Needs object storage (**Q-02**) |
| FR-ACC-14 | Change password, server-side preferences | 7, 8 | 📝 | — | Preferences unblock FR-NOTE-01 |

### FR-NOTE — Notes (17) — rubric 9–14, 17–19

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-NOTE-01…02 | Grid default, list toggle, persisted, responsive | 9, 10 | 📝 | — | |
| FR-NOTE-03…07 | One reusable editor, auto-save, save states, lifecycle flush | 11, 12, 14 | 📝 | — | Content format pending **Q-05** |
| FR-NOTE-08 | Confirmed delete, soft delete with cascade | 13 | 📝 | — | |
| FR-NOTE-09…12 | Pin ordering, badges incl. sync states | 17, 18 | 📝 | — | Client must not re-sort |
| FR-NOTE-13…14 | Live search online and offline | 19 | 📝 | — | Needs the tsvector column |
| FR-NOTE-15…17 | Visible-set scoping, note colour, pagination | 9, 25 | 📝 | — | |

### FR-LBL — Labels (7) — rubric 20–22

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-LBL-01…03 | CRUD, per-owner uniqueness, rename propagation | 20 | 📝 | — | Rename works by reference, not rewrite |
| FR-LBL-04…05 | Delete leaves notes intact; zero-or-more tagging | 21 | 📝 | — | |
| FR-LBL-06…07 | Filter by labels; labels private to owner | 22 | 📝 | — | |

### FR-ATT — Attachments (7) — rubric 15–16

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-ATT-01 | Image/video/file from gallery, camera, picker | 15 | 📝 | — | |
| FR-ATT-02 | Server-side type and size validation | 15, 16 | 📝 | — | Caps pending **Q-06** |
| FR-ATT-03…05 | Cancel is a no-op; denied permission handled; camera hidden on Web | 15, 29 | 📝 | — | |
| FR-ATT-06…07 | Private objects, pre-signed download, authorization on parent | 16, 25 | 📝 | — | Needs bucket (**Q-02**) |

### FR-PROT — Note protection (8) — rubric 23–24

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-PROT-01…02 | Per-note password, bcrypt, verified before content returns | 23 | 📝 | — | **High risk (R1)** |
| FR-PROT-03 | Every operation refused while locked, no exceptions | 23 | 📝 | — | Includes AI retrieval |
| FR-PROT-04…05 | 15-minute Redis grant, rate-limited unlock | 23 | 📝 | — | Needs Redis (**Q-02**) |
| FR-PROT-06…07 | Current password required; change revokes all grants | 24 | 📝 | — | |
| FR-PROT-08 | Grant in memory only; locked content never cached | 23, 31 | 📝 | — | |

### FR-SHR — Sharing & collaboration (10) — rubric 25–26

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-SHR-01…03 | Share by email at READ/EDIT; owner-only control | 25 | 📝 | — | **High risk (R1)** |
| FR-SHR-04 | READ collaborator cannot mutate by any path | 25 | 📝 | — | **The single most important test in the project** |
| FR-SHR-05 | Revocation immediate on every surface | 25 | 📝 | — | |
| FR-SHR-06…07 | Shared-with-me listing; re-share updates | 26, 25 | 📝 | — | |
| FR-SHR-08 | Share notification email | — | 📝 | — | SHOULD — pending **Q-07** |
| FR-SHR-09…10 | Live propagation; local buffer never destroyed | 26 | 📝 | — | |

### FR-AI — AI features (7) — rubric 27–28

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-AI-01…03 | Summary generation, separation from content, staleness | 27 | 📝 | — | Needs LLM key (**Q-03**) |
| FR-AI-04…05 | Retrieval scoped before prompt; locked notes excluded | 28, 23 | 📝 | — | **High risk (R1)** |
| FR-AI-06…07 | Grounded answers with openable references; explicit non-answer | 28 | 📝 | — | |

### FR-XC — Cross-cutting (4) — rubric 29–32

| # | Summary | Rubric | Status | Owner | Note |
|---|---|:-:|:-:|---|---|
| FR-XC-01 | Four states per screen, adaptive, multi-input | 29 | 📝 | — | Verified per screen, not once |
| FR-XC-02 | Architecture, one state solution, 3+3+1 tests, clean analyzer | 30 | 📝 | — | |
| FR-XC-03 | Offline read/write, sync, per-account isolation, conflict copies | 31 | 📝 | — | **High risk (R2)** |
| FR-XC-04 | Public deployment + native artifact from one project | 32 | 📝 | — | **High risk (R3)** — gates verification of everything else |

---

## 3. Open questions

| # | Question | Blocks | Owner | Status / answer |
|---|---|---|---|---|
| Q-01 | Two members or three? Screen-Inventory names Kaka, Chou, Long (Dranov); the submission folder format takes two IDs | Teamwork rule (§5), RACI, task assignment | Team | ☐ **Open — must close in Week 1** |
| Q-02 | Hosting provider for API, Postgres, Redis, and object storage | FR-XC-04, FR-ATT-06, FR-PROT-04, R3 | — | ☐ **Open — must close in Week 1** |
| Q-03 | LLM provider, key, and quota ceiling | FR-AI-01…07, R4 | — | ☐ Open — needed by Week 3 |
| Q-04 | Password reset by link, OTP, or both | FR-ACC-11 | — | ☐ Open — needed by Week 1 |
| Q-05 | Note content format: plain text or Markdown | FR-NOTE-03, FR-AI-02 | — | ☐ Open — needed by Week 2 |
| Q-06 | Attachment size caps (10/50/20 MB proposed) | FR-ATT-02 | — | ☐ Open — depends on Q-02 |
| Q-07 | Share notification email in scope? | FR-SHR-08 | Team | ☐ Open — decide in Week 3 |
| Q-08 | Who creates grading accounts and seed data, and when | Submission package | — | ☐ Open — needed by Week 4 |

None of these block **specification** — all 74 requirements are fully written. They block
**decisions**, and two of them (Q-01, Q-02) block work that starts in Week 1.

---

## 4. Milestones

| Milestone | Date | Exit condition | Status |
|---|---|---|---|
| M0 — Foundation | Sun 30/08/2026 | Schema migrated, layers compiling, envelope + errors, auth end to end, client shell reaching a real login | ☐ |
| M1 — Core product | Sun 06/09/2026 | Notes CRUD + auto-save, home views, search, labels, attachments, profile, preferences | ☐ |
| M2 — Advanced | Sun 13/09/2026 | Note password, sharing enforced, real-time, both AI features | ☐ |
| M3 — Hardening + submission | Sun 20/09/2026 | Offline sync, tests at minimum, deployed and verified, demo recorded, package assembled | ☐ |

---

## 5. Weekly commit compliance (scored deduction)

The rule: **≥2 meaningful commits per member per week, for 4 consecutive Mon–Sun weeks**, under
each member's own GitHub identity. Merge-only, empty, whitespace-only, and generated-file commits
do not count. Missing it costs the whole team 0.5 points and **cannot be fixed afterwards**.

Check every Friday. Anyone short pairs on a small task rather than waiting for the weekend.

| Week | Dates | Kaka | Chou | Long (Dranov) | OK? |
|---|---|:-:|:-:|:-:|:-:|
| W1 | 24/08 → 30/08 | ☐ | ☐ | ☐ | ☐ |
| W2 | 31/08 → 06/09 | ☐ | ☐ | ☐ | ☐ |
| W3 | 07/09 → 13/09 | ☐ | ☐ | ☐ | ☐ |
| W4 | 14/09 → 20/09 | ☐ | ☐ | ☐ | ☐ |

```bash
# Friday check, run from the project root
for r in ltc-nevo-customer-api ltc-nevo-web-app ltc-nevo-knowledge-base ltc-nevo-requirements; do
  echo "== $r"; git -C $r log --since="last monday" --format="%an" | sort | uniq -c
done
```

Evidence for submission: a GitHub Insights contributor-activity screenshot covering all four weeks,
saved as `Screenshot.png`.

---

## 6. Verification gates before submission

Beyond per-requirement status, these are the checks that decide whether the criteria actually
count. Full checklist: BRD §12 and
[`../ltc-nevo-knowledge-base/review-checklist.md`](../ltc-nevo-knowledge-base/review-checklist.md) §5.

| Gate | Status |
|---|---|
| Clean clone builds Web and APK from documented commands | ☐ |
| Deployed Web URL loads and login works from a fresh browser profile | ☐ |
| `GET /health` returns ok with db and redis up | ☐ |
| APK runs on a device that never had a debug build | ☐ |
| Read-only collaborator's direct write refused (integration test in CI) | ☐ |
| Every operation on a locked note refused without a grant | ☐ |
| No secret anywhere in Git history | ☐ |
| ≥3 unit, ≥3 widget, ≥1 integration test passing; `flutter analyze` clean | ☐ |
| Account switch on one device leaks no notes | ☐ |
| `demo.mp4` covers every claimed criterion, AI on both platforms, all members speaking | ☐ |
| `Rubric.xlsx` self-assessment matches the video | ☐ |
| Submission package assembled and named correctly | ☐ |
