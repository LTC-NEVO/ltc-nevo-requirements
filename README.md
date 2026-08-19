# LTC-NEVO Requirements & Planning (`ltc-nevo-requirements`)

Business and product requirement documents (BRD, PRD, Tracking) and the week-by-week delivery plan
for **Nevo** — the cross-platform note-taking application built by team **LTC** for course 503107
(Cross-Platform Mobile App Development, Ton Duc Thang University, Semester I/2026-2027).

Structure modelled on `swt-tb-requirements`, adapted to a course project: one 4-week delivery
window, three team members, and a 32-criterion grading rubric that doubles as the acceptance
contract.

---

## 📁 Directory layout

```
ltc-nevo-requirements/
├── BRD_Nevo_v1.0.md          # Business Requirement Document — why, goals, scope, risks, milestones
├── PRD_Nevo_v1.0.md          # Product Requirement Document — 74 functional requirements + AC
├── TRACKING_Nevo.md          # Progress checklist per requirement + open questions
├── task/                     # Delivery plan
│   ├── 00-TONG-QUAN.md       # 4-week plan, team split, critical path, risks
│   ├── TUAN-1.md             # Week 1 tasks (24/08 → 30/08)
│   ├── TUAN-2.md             # Week 2 tasks (31/08 → 06/09)
│   ├── TUAN-3.md             # Week 3 tasks (07/09 → 13/09)
│   └── TUAN-4.md             # Week 4 tasks (14/09 → 20/09)
└── raw/
    └── README.md             # Pointers to the upstream specifications (not copies)
```

---

## 🔗 Where requirements actually come from

This repository does **not** own the specifications. It decomposes them into buildable work.

| Layer | Lives in | Owns |
|---|---|---|
| **Specifications** | `../documents/FRS.md`, `SDD.md`, `Screen-Inventory.md` | What the course requires. 32 rubric criteria, 21 screens, the technical design. |
| **Engineering knowledge base** | `../ltc-nevo-knowledge-base/` | How the system is built: architecture, conventions, flows, security model, traceability. |
| **This repo** | here | Why we are building it (BRD), what each requirement means at build granularity (PRD), where each one stands (Tracking), and who does what when (task/). |
| **Code** | `../ltc-nevo-customer-api/`, `../ltc-nevo-web-app/` | The implementation. |

**Nothing here overrides `../documents/`.** When a specification and a document in this repo
disagree, the specification wins and the document is wrong — same rule the knowledge base follows.

---

## 🔢 Two requirement numbering schemes, on purpose

| Scheme | Example | Granularity | Lives in |
|---|---|---|---|
| **Rubric level** | `FR-19` | One per rubric criterion (1–32). This is what gets graded. | `../documents/FRS.md`, `../ltc-nevo-knowledge-base/09-requirements/` |
| **Build level** | `FR-NOTE-07` | Decomposed into individually testable statements. This is what gets assigned. | `PRD_Nevo_v1.0.md` (this repo) |

Every build-level requirement carries its rubric criterion in a `Rubric` column. That mapping is
the only thing keeping the two schemes honest — if you add a PRD requirement that maps to no rubric
criterion, you have found either a gap in the rubric reading or scope creep. Usually scope creep.

---

## 🚀 Start here

1. **[PRD_Nevo_v1.0.md](PRD_Nevo_v1.0.md)** — the functional requirements and acceptance criteria.
   The single most useful document if you are about to write code.
2. **[TRACKING_Nevo.md](TRACKING_Nevo.md)** — current status of every requirement, plus the open
   questions that block work.
3. **[task/00-TONG-QUAN.md](task/00-TONG-QUAN.md)** — the 4-week plan, who owns what, and the
   critical path.
4. **[BRD_Nevo_v1.0.md](BRD_Nevo_v1.0.md)** — context, goals, scope boundaries, risks. Read once at
   the start; revisit when someone proposes new scope.

For architecture, conventions, and API/screen detail, go to
[`../ltc-nevo-knowledge-base/`](../ltc-nevo-knowledge-base/) — this repo deliberately does not
duplicate them.

---

## 🗣️ Language

English, matching the upstream specifications (`../documents/`) and the knowledge base. This
differs from `swt-tb-requirements`, which is Vietnamese-first because its source use-case documents
are Vietnamese. Team discussion happens in Vietnamese; written artifacts stay in English so they
line up with the specs they derive from and the code they describe.

---

## 🔄 Keeping this repo true

| When this changes | Update |
|---|---|
| A specification in `../documents/` | PRD requirement text, TRACKING rows, and any affected task |
| A requirement's status | TRACKING (status tag + rollup), and the task board if it unblocks work |
| An open question gets answered | TRACKING §3 (record the answer and date), then unblock the requirements it held up |
| Scope is added or dropped | BRD §5 first — a scope change that only appears in the PRD is how a project quietly grows |
| Architecture or conventions | `../ltc-nevo-knowledge-base/`, not here |
