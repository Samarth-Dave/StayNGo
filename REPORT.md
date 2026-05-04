# StayNGo — DevOps Project Report

> A full-stack hotel booking application containerized, orchestrated, and monitored using industry-standard DevOps tools.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Part 1 — Dockerization](#part-1--dockerization)
4. [Part 2 — CI/CD with Jenkins](#part-2--cicd-with-jenkins)
5. [Part 3 — Code Quality with SonarQube](#part-3--code-quality-with-sonarqube)
6. [Part 4 — Kubernetes Deployment](#part-4--kubernetes-deployment)
7. [Part 5 — Monitoring with Prometheus & Grafana](#part-5--monitoring-with-prometheus--grafana)
8. [Screenshots Checklist](#screenshots-checklist)
9. [Architecture Diagram](#architecture-diagram)

---

## Project Overview

**StayNGo** is a hotel booking web application with a Next.js frontend and a Node.js/Supabase backend. The project was built with a complete DevOps pipeline covering containerization, CI/CD, static code analysis, Kubernetes orchestration, and real-time monitoring.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js |
| Backend | Node.js + Supabase |
| Containerization | Docker + Docker Compose |
| CI/CD | Jenkins |
| Code Quality | SonarQube |
| Orchestration | Kubernetes (Docker Desktop) |
| Monitoring | Prometheus + Grafana + Node Exporter |
| Metrics Collection | kube-state-metrics, cAdvisor |

---

## Part 1 — Dockerization

### What We Did
- Created `Dockerfile` for the frontend (Next.js) and backend (Node.js)
- Created `docker-compose.yml` to run both services together locally
- Used environment variables for Supabase credentials
- Tested both services running in containers communicating over a Docker network

### Key Files
- `Dockerfile` (frontend)
- `Dockerfile` (backend)
- `docker-compose.yml`

### Screenshot
![Docker Compose Running](assets/Screenshot%202026-05-04%20115428.png)

---

## Part 2 — CI/CD with Jenkins

### What We Did
- Set up Jenkins running locally on port `8080`
- Created a `Jenkinsfile` with a multi-stage pipeline:
  - **Clone** — pulls code from GitHub
  - **Build** — builds Docker images for frontend and backend
  - **Test** — runs automated tests
  - **SonarQube Analysis** — runs static code analysis
  - **Deploy** — deploys containers
- Configured GitHub webhook / manual trigger to run the pipeline
- Jenkins builds Docker images and pushes them to the local Docker daemon

### Key Files
- `Jenkinsfile`

### Screenshots
![Jenkins Pipeline View](assets/Screenshot%202026-05-04%20115450.png)
![Jenkins Build Success](assets/Screenshot%202026-05-04%20115533.png)

---

## Part 3 — Code Quality with SonarQube

### What We Did
- Integrated SonarQube into the Jenkins pipeline
- SonarQube scans the codebase for:
  - Code smells
  - Bugs
  - Vulnerabilities
  - Code coverage
- Configured `sonar-project.properties` for both frontend and backend
- Results are visible on the SonarQube dashboard at `http://localhost:9000`

### Key Files
- `sonar-project.properties`
- `Jenkinsfile` (SonarQube stage)

### Screenshot
![SonarQube Dashboard](assets/Screenshot%202026-05-04%20144353.png)
![SonarQube Analysis Result](assets/Screenshot%202026-05-04%20144406.png)

---

## Part 4 — Kubernetes Deployment

### What We Did
- Migrated from Docker Compose to Kubernetes (K8s) on Docker Desktop
- Created Kubernetes manifest files in the `k8s/` folder:
  - `frontend-deployment.yaml` — 2 replicas of Next.js frontend
  - `backend-deployment.yaml` — 2 replicas of Node.js backend
  - `grafana.yaml` — Grafana monitoring dashboard
  - `promethus.yaml` — Prometheus metrics server
  - `node-exporter.yaml` — DaemonSet for node-level metrics
  - `kube-state-metrics.yaml` — Pod/cluster state metrics
  - `prometheus-rbac.yaml` — RBAC permissions for Prometheus
- Services are exposed using `NodePort` for frontend/backend and `ClusterIP` for internal monitoring
- Frontend accessible at `http://localhost:32000`, backend at `http://localhost:32001`
- Scaled frontend and backend to **3 replicas each** to demonstrate horizontal scaling

### Key Commands
```bash
kubectl apply -f k8s/
kubectl get pods
kubectl scale deployment stayngo-frontend --replicas=3
kubectl scale deployment stayngo-backend --replicas=3
```

### Screenshot
![All Pods Running](assets/Screenshot%202026-05-04%20144937.png)
![Kubernetes Services](assets/Screenshot%202026-05-04%20145452.png)

---

## Part 5 — Monitoring with Prometheus & Grafana

### What We Did

#### Prometheus Setup
- Deployed Prometheus as a Kubernetes Deployment
- Configured `prometheus.yml` via ConfigMap with 3 scrape jobs:
  1. `node-exporter` — scrapes hardware/OS metrics from port `9100`
  2. `kube-state-metrics` — scrapes Kubernetes object state from port `8080`
  3. `kubernetes-cadvisor` — scrapes per-container CPU/memory from the K8s API
- Added `ClusterRole` + `ServiceAccount` (RBAC) so Prometheus can access the K8s API

#### Node Exporter Setup
- Deployed as a **DaemonSet** — automatically runs one pod per node
- Exposes hardware metrics: CPU, memory, disk, network
- In a multi-node cluster, one node-exporter pod would run per node automatically

#### kube-state-metrics Setup
- Deployed with full RBAC permissions
- Exposes Kubernetes object metrics: pod status, deployment replicas, container states

#### Grafana Setup
- Deployed Grafana on port `32000` (via NodePort)
- Connected Prometheus as the data source (`http://prometheus:9090`)
- Imported **Dashboard ID 1860** (Node Exporter Full) showing:
  - CPU usage: **64.8%**
  - RAM usage: **83%**
  - Swap usage: **78.3%**
  - Network traffic (Rx/Tx kb/s)
  - Disk space usage per mount
  - System uptime: **4.7 hours**

### Prometheus Targets (All UP ✅)
| Job | Endpoint | Status |
|---|---|---|
| node-exporter | http://node-exporter:9100/metrics | UP |
| kube-state-metrics | http://kube-state-metrics:8080/metrics | UP |
| kubernetes-cadvisor | https://kubernetes.default.svc/api/v1/nodes/.../cadvisor | UP |

### Screenshots
![Prometheus Targets All UP](assets/Screenshot%202026-05-04%20151058.png)
![Grafana Node Exporter Dashboard](assets/Screenshot%202026-05-04%20151600.png)
![Grafana CPU and Memory Graphs](assets/Screenshot%202026-05-04%20153255.png)
![Grafana Network Traffic](assets/Screenshot%202026-05-04%20153328.png)
![Grafana Disk Space](assets/Screenshot%202026-05-04%20153444.png)

---

## Screenshots Checklist

All screenshots are stored in the `assets/` folder. Here is what each screenshot should show:

| # | Screenshot Needed | File in assets/ |
|---|---|---|
| 1 | Docker containers running (`docker ps`) | `Screenshot 2026-05-04 115428.png` |
| 2 | Jenkins pipeline stages view | `Screenshot 2026-05-04 115450.png` |
| 3 | Jenkins build console output / success | `Screenshot 2026-05-04 115533.png` |
| 4 | SonarQube dashboard overview | `Screenshot 2026-05-04 144353.png` |
| 5 | SonarQube project analysis result | `Screenshot 2026-05-04 144406.png` |
| 6 | `kubectl get pods` — all pods running | `Screenshot 2026-05-04 144937.png` |
| 7 | `kubectl get services` — all services | `Screenshot 2026-05-04 145452.png` |
| 8 | Prometheus targets page — all UP | `Screenshot 2026-05-04 151058.png` |
| 9 | Grafana home / dashboard list | `Screenshot 2026-05-04 151600.png` |
| 10 | Grafana Node Exporter Full dashboard | `Screenshot 2026-05-04 153255.png` |
| 11 | Grafana CPU Basic graph | `Screenshot 2026-05-04 153328.png` |
| 12 | Grafana Memory Basic graph | `Screenshot 2026-05-04 153444.png` |
| 13 | Grafana Network Traffic graph | `Screenshot 2026-05-04 153558.png` |
| 14 | Grafana Disk Space graph | `Screenshot 2026-05-04 153608.png` |
| 15 | StayNGo app running in browser | `image.png` |

> ⚠️ **Missing screenshots to capture now:**
> - StayNGo frontend running at `http://localhost:32000`
> - `kubectl get nodes` showing `docker-desktop` node
> - `kubectl get deployments` showing scaled replicas
> - SonarQube quality gate result (passed/failed)

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker Desktop (K8s)                      │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐                       │
│  │  Frontend    │    │   Backend    │                       │
│  │  (x3 pods)   │───▶│  (x3 pods)  │──▶ Supabase (cloud)   │
│  │  Next.js     │    │  Node.js    │                       │
│  └──────────────┘    └──────────────┘                       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Monitoring Stack                        │   │
│  │                                                      │   │
│  │  Node Exporter ──▶ Prometheus ──▶ Grafana Dashboard  │   │
│  │  (DaemonSet)       (ConfigMap)    (Dashboard 1860)   │   │
│  │  kube-state-metrics ──▶ Prometheus                   │   │
│  │  cAdvisor (K8s API) ──▶ Prometheus                   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
         ▲
    Jenkins CI/CD + SonarQube (running on host)
```

---

*Report generated for StayNGo DevOps Project — May 2026*
