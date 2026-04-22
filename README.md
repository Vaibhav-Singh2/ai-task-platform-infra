# AI Task Processing Platform 🚀

A highly scalable, production-ready AI Task Processing Platform built using the MERN stack (MongoDB, Express.js, React/Next.js, Node.js), a Python worker, Redis, and orchestrated with Docker, Kubernetes, and Argo CD for GitOps.

- **Application Repository**: [https://github.com/Vaibhav-Singh2/ai-task-platform](https://github.com/Vaibhav-Singh2/ai-task-platform)
- **Infrastructure Repository**: [https://github.com/Vaibhav-Singh2/ai-task-platform-infra](https://github.com/Vaibhav-Singh2/ai-task-platform-infra)

## 📖 Overview
This application provides users with an authentication-secured interface to submit text processing tasks (e.g., uppercase, lowercase, reverse string, word count). The actual processing is handled completely asynchronously: 
- Tasks are immediately queued into **Redis**.
- A scalable **Python Worker** pulls from the queue, executes the task, and logs the execution.
- Real-time updates are streamed directly to the frontend using **Server-Sent Events (SSE)** via a Redis Pub/Sub mechanism.
- Failed tasks are automatically retried and, if permanently failed, pushed to a **Dead Letter Queue (DLQ)**.

---

---

## 🏗️ Architecture & Documentation
For the system architecture document, worker scaling strategy, and the PRD compliance checklist, please refer to the documentation hosted in the main [Application Repository](https://github.com/Vaibhav-Singh2/ai-task-platform).

---

## 🛠️ Technology Stack
* **Frontend:** Next.js (React), Tailwind CSS, TypeScript
* **Backend:** Node.js, Express.js, TypeScript, Mongoose
* **Worker:** Python 3.12, Redis-py
* **Datastore:** MongoDB, Redis
* **Infrastructure:** Docker (multi-stage builds), Kubernetes (k3s / minikube compatible)
* **CI/CD:** GitHub Actions, Argo CD

---

## 🚀 Getting Started

### 1. Local Development (Docker Compose)
The easiest way to spin up the entire application locally for development is using Docker Compose.

1. Navigate to the companion Application repository's `docker` folder.
2. Copy `.env.example` to `.env` and fill in your secrets (e.g., `JWT_SECRET`, `OPENAI_API_KEY`).
3. Run the following command:
   ```bash
   docker-compose up -d --build
   ```
4. Access the frontend at `http://localhost:3000`.

### 2. Production Deployment (Kubernetes + Argo CD)
The system is fully containerized and configured for Kubernetes deployment using a strict GitOps methodology via Argo CD.

**Prerequisites:**
- A running Kubernetes cluster (e.g., k3s, minikube, or Docker Desktop).
- `kubectl` configured.

**Steps:**
1. **Install Argo CD:**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Deploy the Infrastructure:**
   This repository contains all `Deployment`, `Service`, `ConfigMap`, `Secret`, and `Ingress` manifests. Apply the Argo CD application definition to automatically sync the cluster:
   ```bash
   kubectl apply -f argocd-app.yaml
   ```

3. **Access Argo CD Dashboard:**
   Port-forward the Argo CD UI to your localhost:
   ```bash
   kubectl port-forward svc/argocd-server -n argocd 8080:443
   ```
   * Login at `https://localhost:8080` using `admin` and the initial secret (`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`).

---

## 🔒 Security Posture
- **Passwords:** Hashed using `bcrypt` with a high cost factor (12 rounds).
- **Authentication:** Stateless `JWT` tokens.
- **Headers:** Secured via the `helmet` middleware.
- **Rate Limiting:** IP-based request throttling using `express-rate-limit`.
- **Database:** Full injection protection utilizing `express-mongo-sanitize`.
- **Containerization:** All Dockerfiles utilize hardened `non-root` users (`node` and `workeruser`).

---

## 📂 Repository Structure
- `/base/` - Contains Kustomize base manifests for frontend, backend, worker, redis, and mongodb.
- `argocd-app.yaml` - Argo CD application definition for auto-syncing this repository.
