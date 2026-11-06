# Delivery Notes — Manager Library

Short explanation of what was built, why, and what was left out, so every choice can be justified.

## 1. How the spec was applied (SDD)

- The spec in `spec-python/` was read in the order the README suggests: constitution, spec, plan, architecture, data model, API contract, tasks, ADRs.
- Business behaviour was kept exactly as specified: user stories HU-01 to HU-10, business rules RN-1 to RN-6, the API contract and the data model.
- Technology decisions were translated: FastAPI/SQLAlchemy/Alembic -> Laravel routes, controllers and migrations; pytest -> PHPUnit; Python frontend notes -> React with Vite.
- Where the spec was silent or inconsistent, the gap was written down with a decision and a reason (`spec-gaps-and-decisions.md`), instead of guessing silently.
- The work followed `tasks.md` order: skeleton and health, migration, auth, books, members, loans, returns, overdue, report, error contract. A task counted as done only when its evidence ran.

## 2. Planning documents (in `team/`)

| Document | Purpose |
|---|---|
| `roles-and-responsibilities.md` | Who does what; individual exam, so one person holds every role |
| `functional-requirements.md` | What the system does, taken from HU-01..HU-10 |
| `non-functional-requirements.md` | NFR-1..NFR-5 from the spec plus constitution principles |
| `user-stories.md` | Stories with acceptance criteria |
| `moscow.md` | Priorities: Must = tasks T-01..T-17 (the spec's own "core"), Should, Could, Won't |
| `definition-of-done.md` | A task is done only when its evidence exists and can be run |
| `spec-gaps-and-decisions.md` | Eight spec gaps with the decision taken for each |

## 3. Key decisions and why

1. **Last copy of a book:** conditional update inside a transaction plus the range CHECK, instead of a version column. The data model has no version column and Article X says the data model wins; the conditional update satisfies NFR-2 (one request gets 201, the other 409).
2. **CHECK constraints in the first migration:** `ck_book_total_positive` and `ck_user_role` were moved from "pending T-18" into the initial migration, and `data-model.md` was updated in the same change (Article VI).
3. **Six tables only:** Laravel's own `migrations` table was moved to the `public` schema so the `library` schema holds exactly the six tables of the spec.
4. **Password never in logs:** a test for NFR-3 found that PHP stack traces were writing the clear password to the log. It was fixed by not logging business errors and by omitting function arguments from traces.
5. **Cover upload:** PHP only reads multipart files on POST, so clients send `POST` with `_method=PUT`. This is documented as a known limitation.
6. **Overdue loans in the seeder (gap 8):** the contract has no way to set a past loan date, so the seeder would need a test-only clock override. The decision is recorded.
7. **Error body:** every error uses the single body `code`, `message`, `details`, covered by a contract test over all protected routes.

## 4. What was left out, and why

| Left out | Priority | Reason |
|---|---|---|
| `tool` demo seeder (T-21) | Could | Depends on the finished API and on gap 8; not part of the core |
| Full `docker compose` in `infra` (T-20) | Should | `infra` starts the empty PostgreSQL; API and app each have their own compose file |
| Architecture tests (T-19) | Could | Skipped for time |
| Search load test (NFR-1) | Could | Needs the seeder |
| Cover and librarian screens in the React app | Could | Endpoints exist; no screens |
| Race on the 3-book member limit | Known limit | The spec marks that rule as application-only; only the last-copy race is protected in the database |

## 5. How to verify

- Database: in `infra`, copy `.env.example` to `.env` and run `docker compose up -d`; the database starts empty.
- API and app: see the README of each repo for the commands and the evidence of each task.
- Real credentials are never in the repositories; only `.env.example` files are versioned.

## 6. Tools

The work was done with AI assistance (Claude Code) driven by the spec, with the evidence of each task run and reviewed.
