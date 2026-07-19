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

## Code Quality
- Auto-fix lint + format: `make format`
- Verify (CI gate): `make lint`

`make lint` runs Ruff (check-only), Ruff format check, basedpyright, and docformatter — all at
120-char line length. The same command runs in CI on every PR. Fix failures locally before pushing.

## Tech Stack
Python 3.13, FastAPI, Pydantic v2, Poetry. Persistence via `cumplo_common.database.firestore`.
Auth via `cumplo_common.dependencies` (authenticate, is_admin). Pub/Sub via PubSubMiddleware.

## Gotchas
- `cumplo-common` is pulled from a private Artifact Registry PyPI (`cumplo-pypi`). Locally you
  need `cumplo-pypi-credentials.json` (gitignored); CI uses Workload Identity Federation.
- FAST002 and B008 are intentionally ignored — this service uses FastAPI's `Depends(...)` pattern
  throughout.
- `IS_TESTING` env var switches from Cloud Logging to basicConfig — set it when running locally
  without GCP credentials.

## Before committing
- [ ] Run `make format`, then `make lint` and ensure it passes.
- [ ] No secrets or credential files committed (`cumplo-pypi-credentials.json` is gitignored).
