# raw/ — Upstream source documents

Unlike `swt-tb-requirements/raw/`, which holds original use-case documents and meeting minutes,
Nevo's source specifications live **outside this repository** and are deliberately not copied here.

| Document | Location | What it is |
|---|---|---|
| FRS | [`../../documents/FRS.md`](../../documents/FRS.md) | Functional Requirements Specification — 32 functional + 8 non-functional requirements, mapped 1:1 to rubric criteria, with the traceability matrix and acceptance summary |
| SDD | [`../../documents/SDD.md`](../../documents/SDD.md) | Software Design & Project Documentation — architecture, data model, security design, offline strategy, AI design, testing plan, build/deployment, submission package |
| Screen Inventory | [`../../documents/Screen-Inventory.md`](../../documents/Screen-Inventory.md) | 21 screens and overlays with routes, hierarchy, access levels, and rubric mapping |

## Why they are not copied

A copy is a second source of truth that starts drifting the moment either version is edited. The
BRD and PRD in this repository **derive** from these documents and cite them by section; the
specifications remain canonical.

If a specification changes, the cascade is:

```
../../documents/*.md  changed
        ↓
PRD_Nevo_v1.0.md      requirement text and acceptance criteria
TRACKING_Nevo.md      status rows, open questions
        ↓
../../ltc-nevo-knowledge-base/09-requirements/   (README, traceability matrix, business rules)
../../ltc-nevo-knowledge-base/_meta/manifest.yml requirements[]
```

That cascade is also registered as a watch rule in
[`../../ltc-nevo-knowledge-base/_meta/sync-config.yaml`](../../ltc-nevo-knowledge-base/_meta/sync-config.yaml).

## What belongs here instead

Genuine raw inputs that exist nowhere else, if they ever appear:

- Meeting notes where scope or a requirement interpretation was decided
- Instructor clarifications on rubric criteria
- Screenshots or exports used to settle an open question

Anything filed here should be dated and attributed, because its value is being the record of *when*
something was decided and *by whom*.
