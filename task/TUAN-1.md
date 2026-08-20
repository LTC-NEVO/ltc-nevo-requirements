# WEEK 1 — Mon 24/08 → Sun 30/08 · Milestone M0 (Foundation)

> **Goal:** turn two scaffolds into a running system. By Sunday a user can register, sign in, and
> land on a real (empty) Home screen served by a real database — deployed, not just localhost.
>
> **Exit condition (M0):** schema migrated, six layer directories compiling, response envelope and
> error factories in place, auth end to end, client shell routing through a real login, and the API
> reachable at a public `/health`.

---

## Task status — how to read and update

| Symbol | Meaning | Who changes it |
|:-:|---|---|
| `☐` | Not started | — |
| `🛠️` | In progress | Assignee |
| `🔍` | In review (PR open) | Assignee |
| `⛔` | Blocked — state the reason inline | Assignee |
| `✅` | Done, all six DoD conditions met | Reviewer |
| `✂️` | Cut — state why | Team |

### Count

| Track | Total | ☐ | 🛠️ | 🔍 | ⛔ | ✅ | ✂️ |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Shared | 5 | 5 | 0 | 0 | 0 | 0 | 0 |
| T1 | 7 | 7 | 0 | 0 | 0 | 0 | 0 |
| T2 | 5 | 5 | 0 | 0 | 0 | 0 | 0 |
| T3 | 4 | 4 | 0 | 0 | 0 | 0 | 0 |
| **Total** | **21** | **21** | **0** | **0** | **0** | **0** | **0** |

---

## Critical path this week

Three things block everything after them. All three must be done by **Thu 27/08**:

1. **`W1-SH-02` — Prisma schema + first migration.** Nothing can be persisted until this exists.
2. **`W1-SH-03` — layer skeleton + envelope + error factories.** Every endpoint is built on it.
3. **`W1-T1-03` — auth end to end.** Every other endpoint sits behind its guard.

`W1-SH-05` (deploy) is not blocking development, but it is deliberately in Week 1 rather than
Week 4 — see risk R3.

---

## Shared — infrastructure and decisions

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W1-SH-01` | ☐ | Team | **[BLOCKING]** Close Q-01 and Q-02 | Settle team size (two or three members — the teamwork rule and every assignment depends on it) and choose the hosting provider for API + Postgres + Redis + bucket. Record both answers in `../TRACKING_Nevo.md` §3 with the date. | Both questions closed; `task/00-TONG-QUAN.md` §2 has real names |
| `W1-SH-02` | ☐ | — | **[CRITICAL]** Prisma schema + initial migration | Implement the schema from [`../../ltc-nevo-knowledge-base/diagrams/data-model.md`](../../ltc-nevo-knowledge-base/diagrams/data-model.md): 11 models, `isDeleted` on every table, the composite indexes, plus the raw migration for the `search_vector` tsvector column and its GIN index. | `pnpm prisma:migrate:dev` succeeds; all 11 tables exist with indexes |
| `W1-SH-03` | ☐ | — | **[CRITICAL]** Clean-architecture skeleton | Create the six `Nevo.CustomerService.*` layer directories, `ResponseInterceptor`, `HttpExceptionFilter`, `ErrorCode` enum + `BusinessException` factories, `ConfigModule`, `main.ts` bootstrap (global prefix `nevo/customer-service/api`, URI versioning with `defaultVersion: '1'`, CORS, validation pipe, Swagger). **`JWT_SECRET` must have no fallback — the app refuses to boot without it.** | `GET /health` returns the envelope shape; boot fails cleanly with no `JWT_SECRET` |
| `W1-SH-04` | ☐ | — | Local dependency stack | `docker-compose.yml` with Postgres 16, Redis 7, MinIO, Mailpit; `.env.example` with every variable from [`08-deployment-cicd.md`](../../ltc-nevo-knowledge-base/08-deployment-cicd.md); `.gitignore` covering `.env*` except the example. Closes GAP-104. | Any member runs the full stack from a clean clone |
| `W1-SH-05` | ☐ | — | Deploy the skeleton publicly | Dockerfile, deploy the API to the chosen host, provision managed Postgres + Redis + bucket, run `migrate deploy`, confirm public `/health`. **Deployed in Week 1 on purpose** — discovering hosting problems in Week 4 is risk R3. | Public HTTPS `/health` returns ok with db and redis up |

---

## T1 — Identity & Shell

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W1-T1-01` | ☐ | — | User + preferences domain | `UserEntity`, `UserPreferencesEntity`, `IUserRepository`, DI tokens, repository implementation with `mapToEntity` and `isDeleted` filtering. | Unit test creates and reads a user through the interface |
| `W1-T1-02` | ☐ | — | Registration (FR-ACC-01…05) | `POST /auth/register`: validation, bcrypt cost 10, duplicate → `USER_ALREADY_EXISTS`, auto-login response, default preferences row, activation token + email dispatch that cannot fail the registration. | AC-1 passes end to end against the deployed API |
| `W1-T1-03` | ☐ | — | **[CRITICAL]** Login, refresh, logout (FR-ACC-08…10) | `POST /auth/login` / `/refresh-login` / `/logout`, `JwtStrategy`, `JwtAuthGuard` registered as global `APP_GUARD`, `@Public()` limited to the documented allow-list, `@CurrentUser()`. Refresh rotates; status checked on **both** login and refresh; Redis access key for revocation. | Protected route returns 401 without a token; refresh rotates; logout revokes server-side |
| `W1-T1-04` | ☐ | — | Activation + password reset (FR-ACC-06, 07, 11, 12) | `verify-email`, `resend-verification`, `forgot-password`, `reset-password`. Identical response whether or not the email exists; token single-use, 15 min; reset clears refresh hashes and does **not** auto-login. Depends on Q-04. | AC-2 and AC-4 pass |
| `W1-T1-05` | ☐ | — | Flutter project skeleton | Replace the counter demo: `lib/core/{config,router,theme,network,di,error}`, `lib/features/`, `lib/platform/`, `lib/shared/`; add the dependency set from [`02-tech-stack.md`](../../ltc-nevo-knowledge-base/02-tech-stack.md); `bootstrap.dart`; `env/local.json` + `env/prod.json`. Fix the Android application id (GAP-204). | `flutter run -d chrome` and on Android both show a real shell, not the counter |
| `W1-T1-06` | ☐ | — | Auth screens + routing (FR-ACC-08) | `/login`, `/register`, `/forgot-password`, `/reset-password/:token`, go_router redirect driven by `authStateProvider`, Dio interceptor chain (auth → refresh → envelope → connectivity), secure token storage. All four screen states on each screen. | AC-3 passes on both platforms |
| `W1-T1-07` | ☐ | — | App shell (FR-XC-01) | `AppScaffold`, bottom nav (compact) / navigation rail (medium+), activation banner slot, sync banner slot, `LoadingState` / `EmptyState` / `ErrorState` shared widgets, light + dark theme. | Shell renders correctly at compact, medium, and expanded widths |

---

## T2 — Notes & Organisation

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W1-T2-01` | ☐ | — | Note + label domain | `NoteEntity` (with the `isPinned`/`pinnedAt` and protection invariants), `LabelEntity`, `NoteLabelEntity`, repository interfaces, DI tokens. Pure TypeScript — no framework imports. | Unit test asserts the entity invariants hold |
| `W1-T2-02` | ☐ | — | Note repository | Prisma implementation: `isDeleted: false` on every read including nested includes, ownership filtered in SQL, the pinned-first `orderBy`, paginated list via `$transaction`. | Unit test proves a soft-deleted note is never returned |
| `W1-T2-03` | ☐ | — | Drift local store | `AppDatabase` with `NotesTable`, `LabelsTable`, `NoteLabelsTable`, `PendingOperationsTable`, `SyncMetaTable`, `ConflictCopiesTable`; **one database file per account**; `appDatabaseProvider` overridden at sign-in and disposed at sign-out. | Signing in as a second account opens a different file — verified by test |
| `W1-T2-04` | ☐ | — | Repository pattern, local-first | `NoteRepositoryImpl` reading local first and refreshing from remote; write path applying locally then enqueuing. **Built local-first now so offline in Week 4 is not a rewrite** (risk R2). | Reads work with the API stopped |
| `W1-T2-05` | ☐ | — | Notes list endpoint (FR-NOTE-15, 17) | `GET /notes` with pagination, visible-set scoping (owned ∪ shared), and the deterministic ordering. | Returns the envelope with `{ items, totalCount }` in the right order |

---

## T3 — Protection, Sharing & AI (foundations)

| ID | Status | Owner | Task | Detail | Done when |
|---|:-:|---|---|---|---|
| `W1-T3-01` | ☐ | — | **Design the permission resolver** | Write `resolve(userId, noteId) → own\|edit\|read\|none` following [`07-security-permissions.md`](../../ltc-nevo-knowledge-base/07-security-permissions.md) §`sec-authorization`, with the ordering existence → permission → protection. **Design this week even though sharing ships in Week 3** — three criteria depend on there being exactly one implementation. | Resolver written, unit-tested against all four outcomes, reviewed by a second member |
| `W1-T3-02` | ☐ | — | `RedisService` | ioredis wrapper, TTL mandatory on every key, prefix-scan deletion for bulk grant revocation. | Set/get/expire/delete-by-prefix all covered by tests |
| `W1-T3-03` | ☐ | — | `StorageService` | S3-compatible client, server-generated object keys, MIME allow-list and size caps, pre-signed URLs valid 5 minutes. | Upload and pre-signed download work against MinIO |
| `W1-T3-04` | ☐ | — | `MailService` | Nodemailer with three templates (activation, reset, share). Fire-and-forget with retry so a mail failure never fails its caller. | Activation and reset mails arrive in Mailpit |

---

## End-of-week check (Sun 30/08)

```
□ M0 exit conditions met (see the header)
□ Register → sign in → Home works against the DEPLOYED API, not just localhost
□ Public /health returns ok with db and redis up
□ Both platforms run the real shell — counter demo gone
□ JWT_SECRET has no fallback; app refuses to boot without it
□ Q-01 and Q-02 closed and recorded in ../TRACKING_Nevo.md §3
□ Permission resolver designed, tested, and reviewed by two people
□ Every member has ≥2 meaningful commits this week (../TRACKING_Nevo.md §5)
□ Knowledge base updated: PLANNED → CONFIRMED for everything that now exists,
  with real file:line citations, and gaps GAP-101…104, GAP-201, GAP-204 re-checked
```
