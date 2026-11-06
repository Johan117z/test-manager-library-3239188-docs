# Roles and Responsibilities — Manager Library

Separation of responsibilities by workstream. Each workstream maps to one repository and a group of tasks from `tasks.md`.
This is an individual exam: all roles are held by Johan117z, working through them in the order set by the MoSCoW priorities.

| Role | Repository | Tasks | Responsibilities | Assigned to |
|---|---|---|---|---|
| Backend lead | `api` | T-01 to T-17 | Laravel API: domain, use cases, persistence, auth, contract, concurrency | Johan117z |
| Frontend developer | `app` | T-22 | React app: login, catalog, members, lend, return, overdue, report | Johan117z |
| Infrastructure / DevOps | `infra` | T-20 | Docker Compose, network, volumes, empty PostgreSQL, `.env.example` | Johan117z |
| Data & tooling | `tool` | T-21 | Demo seeder through the API (>= 40 books, 10 members, 15 loans, 3 overdue) | Johan117z |
| Static site | `page` | T-23 | Public presentation site, responsive, no API calls | Johan117z |
| Documentation & QA | `docs` | all | `team/` documents, spec decisions, running each task's evidence, final delivery check | Johan117z |

## Cross-cutting responsibilities

- **Spec guardian:** checks that every change respects `constitution.md`, business rules and the API contract.
- **Delivery owner:** confirms all six forks have their latest commits on GitHub before 11:00 p.m. (Colombia).
- **Commit discipline (everyone):** small Conventional Commits in English, pushed to the fork's `main` after each task.

## Dependencies between roles

`infra` and `api` come first; `app` and `tool` depend on the API contract (T-16); `page` is independent.

## Application roles (system actors)

| Actor | Can do |
|---|---|
| `admin` | Everything: books, members, librarians, reports |
| `librarian` | Loans, returns, overdue, members, catalog queries |
| Member | Data only; does not log in |

