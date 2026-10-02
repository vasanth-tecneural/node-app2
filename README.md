# 📝 Node Blog Application

A full-stack blog application built with **Node.js + Express + React**, deployed through a complete DevOps pipeline — Docker, Kubernetes (Minikube), Horizontal Pod Autoscaling, CI/CD via GitHub Actions, and load-tested with k6.

> This project reimplements a Python/Flask CRUD service from scratch in the Node.js ecosystem and takes it through the entire DevOps lifecycle.

---

## 🌐 Live Demo

> Run locally via Minikube — see [Getting Started](#-getting-started) below.

**Docker Hub Image:** [`manches300/node-blog-app:latest`](https://hub.docker.com/r/manches300/node-blog-app)

---

## 🧱 Tech Stack

| Layer | Technology |
|---|---|
| Backend | Node.js 20, Express 4.18 |
| Frontend | React 19, Vite |
| Database | PostgreSQL 15, Sequelize 6 ORM |
| Containerization | Docker (multi-stage build), Docker Compose |
| Orchestration | Kubernetes v1.28 (Minikube) |
| Autoscaling | Horizontal Pod Autoscaler (HPA) |
| Load Testing | k6 |
| CI/CD | GitHub Actions (self-hosted runner) |

---

## ✨ Features

- ✅ Full **CRUD** operations — Create, Read, Update, Delete blog posts
- ✅ React SPA served directly from Express (single port `5000`)
- ✅ **Multi-stage Docker build** — lean Alpine-based runtime image
- ✅ **Kubernetes deployment** with liveness & readiness probes
- ✅ **HPA** — auto-scales from 2 to 10 replicas based on CPU/memory
- ✅ **Secrets management** — sensitive config stored in Kubernetes Secrets, never in Git
- ✅ **Self-healing** — pods replaced automatically within ~11 seconds of failure
- ✅ **CI/CD pipeline** — automated build, push to Docker Hub, and deploy on every push to `main`

---

## 📁 Project Structure

```
Node_blog_application/
├── app/                    # Express backend source
├── frontend/               # React + Vite frontend
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment
│   ├── service.yaml        # NodePort Service
│   ├── hpa.yaml            # Horizontal Pod Autoscaler
│   └── postgres.yaml       # PostgreSQL Deployment + ClusterIP Service
├── .github/
│   └── workflows/
│       └── ci.yml          # GitHub Actions CI/CD pipeline
├── loadtest.js             # k6 load test script
├── Dockerfile              # Multi-stage Docker build
├── docker-compose.yaml     # Local development stack
├── app.js                  # Express entry point
└── .env.example            # Environment variable template
```

---

## 🚀 Getting Started

### Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Node.js 20+](https://nodejs.org/)

---

### Option 1 — Run with Docker Compose (Local Dev)

```bash
# Clone the repo
git clone https://github.com/manches3003/Node_blog_application.git
cd Node_blog_application

# Copy and configure environment variables
cp .env.example .env

# Start the full stack (app + postgres)
docker compose up --build
```

App will be available at **http://localhost:5000**

> ⚠️ If your database password contains `@`, it must be percent-encoded as `%40` in the `DATABASE_URL`. See [Notes](#-notes) below.

---

### Option 2 — Deploy to Kubernetes (Minikube)

```bash
# Start Minikube
minikube start --driver=docker

# Enable metrics server (required for HPA)
minikube addons enable metrics-server

# Pull the image into Minikube
minikube image pull manches300/node-blog-app:latest

# Create Kubernetes Secret with your credentials
kubectl create secret generic node-blog-secrets \
  --from-literal=DATABASE_URL="postgresql://postgres:password@postgres:5432/node_blog_db" \
  --from-literal=SECRET_KEY="your-secret-key"

# Deploy PostgreSQL
kubectl apply -f k8s/postgres.yaml

# Deploy the application
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/hpa.yaml --server-side=true

# Wait for rollout
kubectl rollout status deployment node-blog-app --timeout=120s

# Get the app URL
minikube service node-blog-app --url
```

> On Windows with Docker driver, keep the terminal open — the tunnel must stay active for the URL to remain accessible.

---

## 🌐 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/posts` | Get all posts |
| `POST` | `/api/posts` | Create a new post |
| `GET` | `/api/posts/:id` | Get a single post |
| `PUT` | `/api/posts/:id` | Update a post |
| `DELETE` | `/api/posts/:id` | Delete a post |
| `GET` | `/health` | Health check (used by probes) |

---

## ⚙️ Kubernetes Architecture

```
Internet
    │
    ▼
NodePort Service (port 30080)
    │
    ▼
Deployment: node-blog-app
├── Pod 1 (node-blog-app)  ─────┐
└── Pod 2 (node-blog-app)       ├── ClusterIP: postgres:5432
        ▲                       │
        │                       ▼
  HPA (min 2 / max 10)    PostgreSQL Pod
  CPU: 50% threshold
  Memory: 70% threshold
```

**Resource limits per pod:**

| | Request | Limit |
|---|---|---|
| CPU | 100m | 500m |
| Memory | 128Mi | 256Mi |

---

## 📊 Load Test Results (k6)

Tested with **500 concurrent virtual users** over **100 seconds** (5-stage ramp).

| Metric | Value | Threshold | Status |
|---|---|---|---|
| Total Requests | 77,834 | — | ✅ |
| Throughput | 771 req/s | — | ✅ |
| Avg Response Time | 3.83 ms | < 2000 ms | ✅ |
| Median (p50) | 1.66 ms | — | ✅ |
| p90 | 5.78 ms | — | ✅ |
| p95 | 12.05 ms | < 2000 ms | ✅ |
| Max Response Time | 134.39 ms | — | ✅ |
| Error Rate | 0.00% | < 50% | ✅ |
| Max Virtual Users | 500 | >= 500 | ✅ |

### Run the load test yourself

```bash
# Install k6: https://k6.io/docs/get-started/installation/
k6 run loadtest.js
```

---

## 🔁 CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/ci.yml`) triggers on every push to `main`:

1. **Test** — Install dependencies, run `npm test`
2. **Build** — Install frontend deps, run Vite build
3. **Docker** — Login to Docker Hub, build & push `manches300/node-blog-app:latest`
4. **Deploy** — Setup Minikube, apply all manifests via `kubectl`

> A **self-hosted GitHub Actions runner** is required because standard hosted runners cannot run a Minikube cluster.

---

## 🔒 Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```env
NODE_ENV=production
PORT=5000
DATABASE_URL=postgresql://postgres:YOUR_PASSWORD@postgres:5432/node_blog_db
SECRET_KEY=your-secret-key-change-in-production
```

In Kubernetes, these are injected from the `node-blog-secrets` Secret — never committed to the repository.

---

## 📝 Notes

### Password URL Encoding
If your PostgreSQL password contains special characters like `@`, you **must** percent-encode them in the `DATABASE_URL`:

```
@ → %40
```

### Minikube Tunnel (Windows)
On Windows with the Docker driver, the terminal running `minikube service node-blog-app --url` must stay open. Closing it will make the URL inaccessible.

### HPA and Metrics Server
The HPA requires the Minikube metrics-server addon. Enable it with:
```bash
minikube addons enable metrics-server
```

---

## 🐛 Known Challenges & Fixes

| Challenge | Fix |
|---|---|
| `ENOENT` error on startup | Dockerfile was copying frontend build to `/app/public` but `app.js` referenced `/app/frontend/dist` — fixed by aligning the static path |
| Docker token error in CI | Docker Desktop access issue during GitHub Actions — regenerated the Docker Hub token |
| Minikube in GitHub Actions | Standard hosted runners can't run Minikube — switched to a self-hosted runner |
| `@` in DB password | Percent-encoded as `%40` in all `DATABASE_URL` values |

---

## 👨‍💻 Authors

- **Keshav Virajbhai Kansara** — [100007269@stud.srh-university.de](mailto:100007269@stud.srh-university.de)


---

## 📄 License

This project is for academic purposes. Feel free to reference or fork it for learning.

---

<p align="center">
  Made with ☕ and way too many <code>kubectl get pods</code> commands
</p>
