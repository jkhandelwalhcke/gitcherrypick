# GitCherryPick

GitCherryPick is a platform where open-source contributors prove skill with **evidence** rather than self-assertion, and hirers cherry-pick candidates from a ranked pool.

- **Claims:** A contributor signs in with GitHub and submits a **claim** (an evidence bundle): up to 5 of their best merged pull requests, 0 to 20 optional supporting projects, and the skills those pull requests demonstrate. One model call judges every pull request against every declared skill; supporting projects are scored arithmetically only.
- **Primary and secondary skills:** Skill standing is derived per `(user, skill)` from distinct surviving pull requests, never self-declared. **5 or more** distinct evidencing pull requests make a skill _primary_ (ranked and searchable). **1 to 4** make it _secondary_ (visible and score-contributing, never ranked). Score and standing are orthogonal—a secondary skill may outscore a primary one and remains secondary—and promotion to primary happens automatically upon the fifth distinct pull request.
- **Scores and hirers:** Three score families come out of evaluation: **per PR per skill**, **per skill**, and two user-level scores—**Overall** (depth, 0–100) and **Generalist** (breadth, unbounded). Scores change only on a rubric change or evidence change, never with age. Hirers use separately verified accounts; scorecards are gated, and contributors see only their own rank.

## Status

### What runs today

- **Local development stack (`make dev` / `backend/scripts/dev.sh`):** Starts PostgreSQL in a container (`55434`), `fakethirdparty` (`18082`, standing in for GitHub, Google, and Resend), and `api` (`8080`), seeded with test fixtures (`--reset` rebuilds the database from schema; `--rich` adds the scored population). Sign-in runs the full OAuth path (state cookie, code exchange, session mint) against the stand-in without requiring credentials from `github.com`.
- **Backend binaries (`backend/cmd/`):** Contains `api`, `evaluator`, `fakethirdparty`, `devdata`, and `jobs`.
- **Frontend (`frontend/`):** The React and TypeScript client is present in the tree and can run every screen against the local API and seeded data except for a live score appearing automatically on a newly submitted claim.

### What does not run yet

- **Evaluator in `make dev` and live model call (Stage 5):** `backend/scripts/dev.sh` does not start `cmd/evaluator`. A submitted claim stays queued until `cmd/evaluator` is run separately, and while `internal/adapter/anthropic` implements the batch submit/poll/fetch flow against `fakethirdparty`'s wire contract, both `CLAUDE.md` and `backend/scripts/dev.sh` note that Stage 5's live model call is still a stub.
- **Deployment and containerization (Stage 7):** `backend/scripts/dev.sh` runs PostgreSQL in Docker and runs the binaries on the host; Stage 7 images and orchestration are not yet implemented.

## Layout

### Top-level directories

- `.claude/` — Agent architecture rules (`Agent.md`) and bootstrap scripts.
- `.github/workflows/` — CI workflows (`check.yml` and CLA check).
- `adr/` — Approved Architecture Decision Records (`ADR-0001` through `ADR-0022`).
- `backend/` — Go backend module (`cmd/`, `internal/`, `e2e/`, `scripts/`).
- `evaluation/` — Scoring rubric specifications.
- `flow-deisgn/` — Flow design documents.
- `frontend/` — React and TypeScript client application.
- `rfc/` — Feature RFCs (`.md`) and linked schema deltas (`.schema`).
- `scripts/` — Repository utility scripts.

### Packages under `backend/internal/`

- `backend/internal/adapter/` — Infrastructure and third-party adapters implementing `port` interfaces (`anthropic`, `clock`, `crypto`, `github`, `google`, `httpx`, `money`, `places`, `resend`).
- `backend/internal/controller/` — HTTP controllers, router, middleware, and IO validation/serialization.
- `backend/internal/domain/` — Domain entities and errors; imports nothing from `internal/`.
- `backend/internal/port/` — Boundary interfaces (`Repository`, `Broker`, `AIClient`, `GitHub`, `Notifier`, `Clock`, and service interfaces).
- `backend/internal/repository/` — Storage implementations (`postgres`, embedded SQL migrations, and queue broker) translating between domain models and database rows.
- `backend/internal/scoring/` — Pure arithmetic scoring functions with no I/O.
- `backend/internal/service/` — Core business logic depending only on `port` interfaces.
