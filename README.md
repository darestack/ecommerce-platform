# ecommerce-platform

> This repo demonstrates a complete Docker-based CI/CD pipeline.
> The app (React frontend + Express.js API) is the deployment subject — the pipeline is the product.

**Context:** Part of the [devops-labs Module-3 Capstone 5](https://github.com/darestack/devops-labs/tree/main/Module-3/capstone-project-5) — full implementation notes, evidence, and troubleshooting guide live there.

---

## CI/CD Pipeline Overview

```
Push to main
  │
  ├── CI (ci.yml): checkout → npm ci → test → build
  │     Runs on: every push to main + every PR
  │
  ├── Image publish (docker-publish.yml): on app changes merged to main
        ├── Build API image  → push to GHCR (ghcr.io/.../api:latest + :sha)
        └── Build Webapp image → push to GHCR (ghcr.io/.../webapp:latest + :sha)
  │
  └── Deploy workflow (deploy.yml): pulls images and runs Docker Compose on EC2
        Requires EC2 secrets and fresh deployment evidence before promotion
```

### Pipeline Details

| File | Trigger | What It Does |
|---|---|---|
| `.github/workflows/ci.yml` | Every push + PR | `npm ci` → `npm test` → `npm run build` for both api and webapp |
| `.github/workflows/docker-publish.yml` | Push to `main` | Multi-arch Docker build via Buildx, push to GHCR with `latest` and `sha` tags |
| `.github/workflows/deploy.yml` | Successful CI workflow | Pulls GHCR images and runs Docker Compose on EC2 when secrets are configured |

## Evidence

| Evidence | Link |
|---|---|
| Latest passing CI run | [GitHub Actions run 25606532931](https://github.com/darestack/ecommerce-platform/actions/runs/25606532931) |
| Docker Publish run | [GitHub Actions run 18215811001](https://github.com/darestack/ecommerce-platform/actions/runs/18215811001) |

This repo is a companion app for the `devops-labs` Capstone 5 write-up. Keep it
as supporting evidence unless the project is refreshed with current screenshots,
deployment logs, and a stronger README.

**Key implementation decisions:**
- `actions/setup-node@v6` with `cache: npm` — avoids re-downloading dependencies on every run
- `docker/setup-buildx-action` + `docker/build-push-action` — enables layer caching and multi-platform builds
- GHCR auth via `secrets.GITHUB_TOKEN` — no long-lived credentials stored
- Images tagged with both `latest` and commit SHA — enables rollback to any prior build

---

## Application Stack

| Layer | Technology |
|---|---|
| Frontend | React, React Router, Axios |
| Backend | Node.js, Express.js, JWT auth |
| Containerisation | Docker, Docker Compose |
| Registry | GitHub Container Registry (GHCR) |
| Deployment target | AWS EC2 |

---

## How to Run Locally

**Prerequisites:** Docker and Docker Compose installed.

```bash
git clone https://github.com/darestack/ecommerce-platform.git
cd ecommerce-platform
export JWT_SECRET=your_secret_key
docker-compose up --build
```

- Frontend: http://localhost:3000
- API health check: http://localhost:3001/health

---

## Key Challenges Solved

**Docker cache export failure:** Default GitHub Actions Docker driver does not support
cache export. Fixed by adding `docker/setup-buildx-action` before the build step.

**GHCR package creation permission denied:** `GITHUB_TOKEN` requires explicit
`packages: write` permission in the workflow. Added to job-level permissions block.
