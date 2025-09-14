# Fintech-Blueprint Clonable Org: Golden Template & CI/CD Gates Overview

## Purpose

This repository set provides a clonable organization scaffold and a golden template microservice (`example-service`) intended to accelerate new service onboarding. It contains standardized CI/CD gates, shared libraries, governance ADRs, and GitOps manifests for staging.

## Repo List

- infra: Platform infrastructure, ADRs, and platform-level policies.
- platform-libs: Shared libraries (auth, tracing, idempotency, outbox, utils).
- contracts: API contract schemas and shared protobufs/openapi.
- concierge: Service that coordinates onboarding and infra tasks.
- db-manager: Database lifecycle and migrations service.
- api-gateway: Edge routing and request orchestration.
- catalog: Service catalog and metadata.
- design-system: Shared UI components (storybook, tokens).
- example-service: Golden template microservice (FastAPI) with CI/CD.
- .github: Organization-level GitHub assets (workflows, runbooks, dependabot).

## Golden Template Usage (example-service)

1. Clone the repo:

```bash
git clone git@github.com:Fintech-Blueprint/example-service.git
cd example-service
```

2. Create a virtual environment and install dependencies:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

3. Run locally:

```bash
uvicorn src.application.main:app --reload --port 8000
# then open http://localhost:8000/healthz
```

4. Build and run with Docker:

```bash
docker build -t example-service:local .
docker run -p 8000:8000 example-service:local
```

5. Deploy to staging (via GitOps): create a branch, update manifests under `k8s/` and open a PR targeting `main`.

## CI/CD Gates

- Unit Tests: Runs `pytest` unit suite; must pass before merging.
- Contract Tests: Validates OpenAPI/contract compatibility with shared schemas.
- Lint: Runs `flake8`/`ruff` and enforces style rules.
- SAST: Static application security scans (e.g., bandit, semgrep).
- Docker Build: Confirms container builds successfully and validates multi-stage cache.
- SBOM & Signing: Generates SBOM and runs image signing (cosign) as part of release pipeline.
- Staging Canary: Deploys changes to the `staging-env` ArgoCD application for canary verification.

Each PR is expected to pass all required checks listed above (see the branch protection runbook for exact status-check names).

## Notes

- Branch protection configuration was intentionally skipped and must be applied by an org admin. See `branch_protection_runbook.md` for exact `gh` CLI commands.
- Secret scanning and certain enforcement features may require plan upgrades or web UI confirmation; validate in the org settings.
- Some repository-level features (required signatures) are not included due to plan limitations.

## Contact / Support

- Default platform admin contact: `platform-admins@fintech-blueprint.example` (replace with your support alias).
- For emergency access, contact org-admins via the GitHub org team or directly reach out to `security@fintech-blueprint.example`.
