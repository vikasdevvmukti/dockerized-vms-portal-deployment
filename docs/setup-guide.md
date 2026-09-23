# Setup Guide

This guide covers two ways to run the project: **locally with Docker Compose** (for testing) and **the full automated CI/CD path** (build → push → deploy via GitHub Actions).

---

## Prerequisites

- Docker & Docker Compose installed
- A Docker Hub account (for image storage)
- A GitHub repository with Actions enabled
- A remote Ubuntu VM reachable via SSH (for the deploy step)
- Node.js 20+ (only needed if running frontend/backend outside Docker)

---

## 1. Repository Structure

```
.
├── .github/workflows/
│   └── deploy.yml            # CI/CD pipeline (see config/deploy.yml)
├── arcis_frontend_R-D/
│   ├── Dockerfile            # see config/Dockerfile.frontend
│   ├── nginx.conf            # see config/nginx.conf
│   └── ...
├── arcis_backend_R-D/
│   ├── Dockerfile            # see config/Dockerfile.backend
│   ├── config/.config.env    # see config/.config.env.example
│   └── ...
├── docker-compose.yml        # see config/docker-compose.yml
├── config/                   # reference copies of all config files
├── docs/
│   └── setup-guide.md        # this file
└── README.md
```

---

## 2. Local Setup (run without the pipeline)

1. **Clone the repo**
   ```bash
   git clone <your-repo-url>
   cd <repo-folder>
   ```

2. **Set up backend environment variables**
   ```bash
   cp config/.config.env.example arcis_backend_R-D/.config.env
   # then edit arcis_backend_R-D/.config.env with real values
   ```

3. **Build and run both services locally**
   ```bash
   docker compose up --build
   ```
   - Frontend → `http://localhost:80`
   - Backend → `http://localhost:8082`

4. **Stop the containers**
   ```bash
   docker compose down
   ```

---

## 3. CI/CD Setup (automated build + deploy)

### a) Add GitHub Actions secrets
In your repo → **Settings → Secrets and variables → Actions**, add:

| Secret | Purpose |
|---|---|
| `DOCKER_USERNAME` | Docker Hub username |
| `DOCKER_PASSWORD` | Docker Hub access token / password |
| `REACT_APP_BASE_URL` | API base URL baked into the frontend build |
| `SERVER_IP` | Deployment VM's IP address |
| `SERVER_USERNAME` | SSH username on the VM |
| `SERVER_SSH_KEY` | Private SSH key for the VM (no passphrase) |

### b) Prepare the VM
1. Install Docker and Docker Compose on the VM.
2. Copy `config/docker-compose.yml` to `/home/ubuntu/docker-compose.yml` on the VM (update image names/tags if you rename repos).
3. Place the backend's real `.config.env` file on the VM at the path referenced in `docker-compose.yml`.
4. Make sure the VM's firewall / security group allows inbound traffic on ports `80` and `8082`.

### c) Trigger the pipeline
Push to `main`:
```bash
git push origin main
```

GitHub Actions will:
1. Build the frontend and backend Docker images.
2. Push both images to Docker Hub.
3. SSH into the VM and run:
   ```bash
   docker pull <username>/arcis_frontend_r-d:latest
   docker pull <username>/arcis_backend_r-d:latest
   docker compose -f /home/ubuntu/docker-compose.yml up -d
   ```

### d) Verify deployment
On the VM:
```bash
docker ps
```
Both `frontend` and `backend` containers should show `Up`. Then open the VM's IP in a browser to confirm the portal loads.

---

## 4. Troubleshooting

| Issue | Likely Cause |
|---|---|
| Pipeline fails at `Docker_hub_login` | Wrong `DOCKER_USERNAME` / `DOCKER_PASSWORD` secret |
| Pipeline fails at `SSH into server and deploy` | Wrong `SERVER_IP`, `SERVER_USERNAME`, or `SERVER_SSH_KEY`; VM firewall blocking SSH |
| `docker compose up -d` fails on VM | `docker-compose.yml` path incorrect, or `.config.env` missing on the VM |
| Frontend loads but API calls fail | `REACT_APP_BASE_URL` not set correctly at build time, or `nginx.conf` proxy path misconfigured |
| Containers up but portal unreachable | VM security group / firewall not allowing inbound traffic on port 80 |

---

## 5. Config File Reference

All reference config files used by this project are in [`/config`](../config):

- `Dockerfile.frontend` — multi-stage build for the React frontend, served via Nginx
- `Dockerfile.backend` — production Node.js backend build
- `docker-compose.yml` — orchestrates both containers together
- `nginx.conf` — Nginx routing/proxy config for the frontend container
- `deploy.yml` — the GitHub Actions CI/CD workflow
- `.config.env.example` — template for backend environment variables
