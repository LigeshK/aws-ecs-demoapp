# 📦 AWS ECS Demo — Application

> **Part 2 of 2** · Infrastructure repo → [aws-ecs-demoinfra](https://github.com/LigeshK/aws-ecs-demoinfra)

A containerized two-tier web application — Nginx frontend + Node.js backend — with a fully automated CI/CD pipeline that builds Docker images, pushes to Amazon ECR, and deploys to ECS Fargate.

---

## What This Does

Push to `main` → GitHub Actions builds both container images, tags them, pushes to ECR, and triggers a rolling ECS deployment. No manual steps.

The application itself is intentionally simple: the frontend calls the backend health endpoint and displays the response. The focus is the **deployment pipeline and container architecture**, not the app logic.

---

## Architecture

```
Developer
    │
    │  git push main
    ▼
GitHub Actions
    │
    ├─ Build frontend image  ──► ECR (frontend-latest)
    ├─ Build backend image   ──► ECR (backend-latest)
    │
    └─ Update ECS Services
           │
           ▼
    ECS Cluster (provisioned by aws-ecs-demoinfra)
           │
    ┌──────┴──────┐
    │     ALB     │  ← public endpoint
    └──────┬──────┘
           │
    ┌──────┴──────────────┐
    │                     │
 /  route            /api* route
    │                     │
Frontend Service    Backend Service
 (Nginx :80)       (Node.js :3000)
```

Frontend proxies API calls to the backend via ECS Service Discovery (`backend.ecsworkshop.local`).

---

## Application Components

### Frontend — Nginx + Static Assets

| Item | Detail |
|---|---|
| Server | Nginx |
| Port | 80 (container) · 8080 (local) |
| Proxy | `/api/*` forwarded to backend via `nginx.conf` |
| Build | Multi-stage Dockerfile |

### Backend — Node.js / Express

| Item | Detail |
|---|---|
| Runtime | Node.js 18 |
| Port | 3000 |
| Endpoint | `GET /api/health` → `{"status":"ok"}` |
| Features | CORS enabled, JSON responses |

---

## CI/CD Pipeline

```
git push main
     │
     ▼
GitHub Actions (.github/workflows/deploy.yml)
     │
     ├─ docker build frontend  →  ECR push (frontend-latest)
     ├─ docker build backend   →  ECR push (backend-latest)
     │
     ├─ ecs update-service --service frontend-service --force-new-deployment
     └─ ecs update-service --service backend-service  --force-new-deployment
```

**Required GitHub Secrets:**

| Secret | Purpose |
|---|---|
| `AWS_ACCOUNT_ID` | ECR URI construction |
| `AWS_ACCESS_KEY_ID` | AWS authentication |
| `AWS_SECRET_ACCESS_KEY` | AWS authentication |

> ECR repository (`my-app-repo`) and ECS services (`frontend-service`, `backend-service`) are provisioned by the [infrastructure repo](https://github.com/LigeshK/aws-ecs-demoinfra). Deploy that first.

---

## Local Development

### Prerequisites
- Docker
- Node.js 18+
- AWS CLI (for ECR interaction)

### Run Locally with Docker

```bash
# --- Backend ---
cd backend
docker build -t test-backend .
docker run -d -p 3000:3000 --name backend test-backend

# Verify
curl http://localhost:3000/api/health
# → {"status":"ok"}

# --- Frontend ---
cd ../frontend
docker build -t test-frontend .
docker run -d -p 8080:80 --name frontend test-frontend

# Visit http://localhost:8080
# Note: backend status shows "Error" locally — expected without ECS service discovery

# Cleanup
docker stop frontend backend && docker rm frontend backend
```

---

## Project Structure

```
aws-ecs-demoapp/
├── frontend/
│   ├── index.html          # UI
│   ├── style.css           # Styling
│   ├── script.js           # API call logic
│   ├── nginx.conf          # Proxy config → backend
│   └── Dockerfile          # Nginx container
├── backend/
│   ├── index.js            # Express server + /api/health
│   ├── package.json
│   └── Dockerfile          # Node.js container
└── .github/
    └── workflows/
        └── deploy.yml      # Build → push → deploy pipeline
```

---

## Deployment Flow (End-to-End)

```
Step 1: Deploy Infrastructure
  └─ aws-ecs-demoinfra → provisions VPC, ECS cluster, ALB, ECR

Step 2: Deploy Application (this repo)
  └─ git push → GitHub Actions → ECR push → ECS rolling update

Step 3: Access
  └─ http://<ALB-DNS-Name>   (from CloudFormation outputs)
```

---

## Infrastructure Dependency

This repo **requires** the infrastructure from [aws-ecs-demoinfra](https://github.com/LigeshK/aws-ecs-demoinfra) to be deployed first. It provisions:

- The ECR repository this pipeline pushes images to
- The ECS cluster and services this pipeline deploys to
- The ALB that routes traffic to the running containers
- IAM roles and security groups the tasks run under

---

## Tech Stack

`Node.js` · `Express` · `Nginx` · `Docker` · `Amazon ECS (Fargate)` · `Amazon ECR` · `GitHub Actions` · `AWS CLI`
