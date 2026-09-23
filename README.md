<img width="1920" height="1011" alt="Screenshot from 2026-09-23 14-38-24" src="https://github.com/user-attachments/assets/cee93add-a698-4dd7-9b62-b8dda96f582d" /># Dockerized CI/CD Deployment Pipeline for a VMS Web Portal

A self-initiated DevOps project exploring **Docker-based deployment** for a Video Management System (VMS) web portal, as an alternative to a direct VM deployment workflow. The project automates the entire path from a `git push` to a live, running application using **GitHub Actions**, **Docker**, and **Docker Compose**.

> Personal / practice project — built to strengthen hands-on CI/CD and containerization skills outside of production work.

---

## Overview

The VMS portal has two independent services — a **frontend** and a **backend** — each containerized separately. On every push to `main`, a GitHub Actions pipeline builds both Docker images, pushes them to Docker Hub, then SSHes into a remote VM to pull the new images and redeploy the app via Docker Compose — with zero manual steps.

## Architecture / Workflow

```
git push (main)
      │
      ▼
GitHub Actions triggered
      │
      ├── Job 1: build_and_push
      │     ├── Checkout code
      │     ├── Login to Docker Hub
      │     ├── Build + push frontend image
      │     └── Build + push backend image
      │
      └── Job 2: deploy (needs: build_and_push)
            └── SSH into VM
                  ├── docker pull frontend:latest
                  ├── docker pull backend:latest
                  └── docker compose up -d
                              │
                              ▼
                  Live VMS portal running on the VM
```

## Tech Stack

| Category | Tools |
|---|---|
| Containerization | Docker, multi-stage Dockerfiles |
| CI/CD | GitHub Actions |
| Orchestration | Docker Compose |
| Image Registry | Docker Hub |
| Deployment Target | Remote Ubuntu VM (via SSH) |
| Web Server | Nginx (serving the built frontend) |
| Backend Runtime | Node.js |

## Key Highlights

- **Multi-stage Dockerfile** for the frontend — builds the React app in a `node` stage, then serves the static build through a lightweight `nginx:alpine` stage.
- **Separate, independent Dockerfiles** for frontend and backend, so each service builds and scales independently.
- **Fully automated CI/CD** — a single `git push` triggers build, push, and deployment with no manual intervention.
- **Secrets-driven configuration** — Docker Hub credentials, SSH host/key, and environment variables are all injected via GitHub Actions secrets, never hardcoded.
- **One-command redeployment** on the VM using Docker Compose, keeping frontend and backend in sync.

---

## Screenshots

### 1. Dockerfiles — Frontend & Backend
Multi-stage build for the frontend (React build → Nginx serve) and a lean production build for the backend.

<img width="1617" height="483" alt="Screenshot from 2026-09-23 14-36-51" src="https://github.com/user-attachments/assets/50f909e5-7598-496b-9f90-7e9a6ec5fe41" />


### 2. GitHub Actions Workflow (`deploy.yml`)
Defines the two jobs — `build_and_push` and `deploy` — and how they chain together.

<img width="1632" height="927" alt="Screenshot from 2026-09-23 14-37-25" src="https://github.com/user-attachments/assets/2abbdf91-2faa-419a-8b0d-4bfc2d4ce232" />


### 3. Docker Compose Configuration
Orchestrates the frontend and backend containers together on the VM.

<img width="1632" height="927" alt="Screenshot from 2026-09-23 14-37-34" src="https://github.com/user-attachments/assets/d0e7a5db-c3ec-4182-99fb-5ac05c105ff0" />


### 4. Successful Pipeline Run — Deploy Step Expanded
The `deploy` job's log, showing the pipeline pulling both images and running `docker compose up -d` on the VM automatically.

<img width="1919" height="967" alt="Screenshot from 2026-09-23 14-39-54" src="https://github.com/user-attachments/assets/61de8cc9-d401-468e-a535-06301ba6589a" />


### 5. Containers Running on the VM
`docker ps` output confirming both containers are up and healthy after deployment.

<img width="1920" height="1030" alt="Screenshot from 2026-09-23 14-37-45" src="https://github.com/user-attachments/assets/f74a2bfe-c5cf-44bb-9052-6eef04aa7a46" />


### 6. Live Portal
The deployed VMS portal, live and accessible in the browser.

<img width="1920" height="1011" alt="Screenshot from 2026-09-23 14-38-24" src="https://github.com/user-attachments/assets/8bd46d82-a44a-4f3e-9c5c-5975ebd7f495" />


---

## What I Learned

- Structuring a multi-service app with independent Dockerfiles and a shared Docker Compose file.
- Writing multi-job GitHub Actions workflows with job dependencies (`needs:`).
- Safely handling secrets (Docker Hub credentials, SSH keys, env vars) in CI/CD without exposing them in code.
- Automating remote deployment over SSH as part of a pipeline, instead of manually logging in to deploy.

## Notes

- Credentials, VM IP, and SSH keys are managed entirely through GitHub Actions **encrypted secrets** — none are present in the repository.
- This project intentionally mirrors a real production deployment pattern (build → push → pull → compose up) on a smaller, self-hosted scale.
