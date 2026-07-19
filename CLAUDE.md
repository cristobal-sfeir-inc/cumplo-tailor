# cumplo-tailor

## Overview
FastAPI service that manages user configuration for the Cumplo API project — investment filters,
notification channels (WhatsApp, IFTTT, webhooks), credentials, and Gmail subscription handling.
Deployed on Cloud Run; depends on `cumplo-common` for shared domain models, Firestore client,
and auth middleware.

## Build & Run
- Install deps: `poetry install`
- Run locally: `docker-compose up -d cumplo-tailor`
- Build image: `make build`
- Update common: `make update-common`

Python 3.13, Poetry 2.4.x.

## Code Quality
Three tools enforced at 120 cols — all three must agree before merging.

| Tool | Role | Command |
|---|---|---|
| Ruff | Lint + format | `make lint` / `make format` |
| basedpyright | Type checking | included in `make lint` |
| docformatter | Docstring style | included in `make lint` |

`make format` auto-fixes; `make lint` verifies (same as CI, no fixes applied).

## Testing
No automated test suite exists yet — test coverage is a known reliability gap. When adding
features or fixing bugs, include tests in `tests/` to cover the changed path. Run with
`poetry run pytest`.

## CI / CD
- **PR gate:** `.github/workflows/lint.yml` runs `make lint` on every pull request.
  The workflow authenticates to Artifact Registry via Workload Identity Federation before
  installing dependencies. A passing lint gate is required before merge.
- **Deploy:** Google Cloud Build deploys on push to `master` (lint is NOT re-run by Cloud Build).
  The PR gate is the only enforcement point — never merge a branch that fails `make lint`.

## Git Workflow
- Branch prefixes: `feat/`, `fix/`, `chore/`, `ci/`. Conventional-commit subjects (`feat:`,
  `fix:`, …).
- `master` is protected by the `not-cumplo-audit-gate` ruleset: every change requires a
  PR + code-owner review (`@cnsfeir-reviewer`). **Never push to `master` directly.**
- Open one focused PR per issue. Match existing patterns before introducing new ones.

## Gotchas
- `cumplo-common` is pulled from a private Artifact Registry PyPI (`cumplo-pypi`). Locally you
  need `cumplo-pypi-credentials.json` (gitignored); CI uses Workload Identity Federation with a
  read-scoped service account (`vars.READ_SA`).
- `FAST002` and `B008` are intentionally ignored — this service uses FastAPI's `Depends(...)`
  pattern throughout.
- `IS_TESTING` env var switches from Cloud Logging to basicConfig — set it when running locally
  without GCP credentials.
- **`cumplo-common` blast radius:** every service depends on it. Changes there affect all Cloud
  Run services — keep common changes minimal and backward-compatible.

## Before Committing
- [ ] Run `make format`, then `make lint` and ensure it passes.
- [ ] No secrets or credential files committed (`cumplo-pypi-credentials.json` is gitignored).
- [ ] Version not required (this is a service, not a published library).
