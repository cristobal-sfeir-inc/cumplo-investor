# cumplo-investor

## Overview
Automatic investment service that receives funding-request events and places investments on
behalf of users via the Cumplo API. **Not yet implemented** — stub repo (README + LICENSE +
.gitignore only); no source code exists.

## Stack (intended)
Python 3.13, Poetry, FastAPI, Pydantic v2. Persistence: Firestore. Async fan-out: Pub/Sub
(with Cloud Tasks). Deployed on Cloud Run. Consumes `cumplo-common` for shared domain logic,
Firestore client, and Pub/Sub middleware.

## Build & Test
No Makefile or pyproject.toml exists yet. When code is scaffolded, the standard commands will be:

- Install deps: `poetry install`
- Run tests: `poetry run pytest`
- Auto-fix lint + format: `make format`
- Verify code quality (CI gate): `make lint`
- Full local CI simulation: `make check`
- Build Docker image: `make build`

## CI/CD
Only `.github/workflows/pr-title.yml` is active — enforces conventional-commit format on PR
titles (e.g. `feat: Add investment endpoint`). No lint, test, or deploy pipeline exists yet.

When code lands, the standard pattern applies: GitHub Actions `ci.yml` runs `make lint` + pytest
on every PR; Cloud Build deploys to Cloud Run on merge to `master`.

## Git workflow
- Branch prefixes: `feat/`, `fix/`, `chore/`, `ci/`. Conventional-commit subjects.
- PR title must start with an uppercase letter after the type prefix.
- `master` is protected: every change needs a **PR + code-owner review** (`@cnsfeir-reviewer`).
  **Never push directly to `master`** — it is rejected by the ruleset.

## Architecture notes
- Triggered by Pub/Sub events from `cumplo-spotter` (filtered funding-request notifications).
- Reads per-user investment preferences from Firestore to decide amount and eligibility.
- Calls the Cumplo API (authenticated via `cumplo-authenticator`) to place the investment.
- This service actually spends real money — investment logic must be gated behind user config
  and tested thoroughly before any live deployment.

## Gotchas
- **No code yet.** No Makefile, pyproject.toml, or Python source exists — this is a pure stub.
- When scaffolding, pull `cumplo-common` from the private Artifact Registry PyPI. A
  `cumplo-pypi-credentials.json` SA key is required locally (gitignored); CI uses Workload
  Identity Federation.
- Follow the multi-stage Dockerfile and `poetry.toml` patterns from `cumplo-herald` or
  `cumplo-spotter` as the Cloud Run reference.
- **Idempotency is critical.** Pub/Sub delivers at-least-once; investing twice on the same
  request would be a real financial error. Design with a dedup key from day one.

## Before committing
After making changes, run the lint/format commands and ensure they pass; check no hardcoded
secrets — before committing.
