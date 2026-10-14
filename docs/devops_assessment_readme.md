# DevOps Engineer — Hands-On Assessment

A production-style platform built with React, Node.js, and PostgreSQL, containerized via Docker, orchestrated using Kubernetes, and managed through automated CI/CD workflows.

---

## 1. Prerequisites & One-Command Startup (Tasks 1 & 2)

### Prerequisites
* Docker & Docker Compose
* Kubernetes cluster (Kind, Minikube, or AWS EC2 K8s node)
* kubectl CLI
* Git

### One-Command Startup
* **Local Docker Compose (Task 1):**
  ```bash
  docker compose up --build
  ```
* **Local Kubernetes Deployment (Task 2 & 5):**
  ```bash
  kubectl apply -f k8s/ -n preprod
  ```

---

## 2. Request Path & Kubernetes Service Discovery (Task 3)

### Full Request Path
* **Flow:** Browser ➔ Ingress Controller (NGINX) ➔ Kubernetes Service (ClusterIP/NodePort) ➔ Backend/Frontend Pod ➔ PostgreSQL Database.

### Kubernetes DNS Resolution
* **Internal Resolution:** Services communicate inside the cluster using Fully Qualified Domain Names (FQDN) following this standard pattern: `[service-name].[namespace].svc.cluster.local` (e.g., `backend-service.preprod.svc.cluster.local`).

### Debugging Scenarios
* **Ingress is reachable, but backend is not:** Check pod health (`kubectl get pods -n preprod`), inspect Ingress routing rules (`kubectl describe ingress`), and review backend container logs (`kubectl logs`).
* **Frontend cannot talk to backend:** Verify the API base URL environment variable in the frontend and test internal pod connectivity using `curl` or `nslookup`.

---

## 3. CI/CD & Multi-Environment Deployment (Task 4)

### Pipeline Architecture
* **Automation:** A GitHub Actions workflow (`.github/workflows/ci-cd.yml`) automates building dependencies, testing, containerizing, tagging with branch names and short commit SHAs, pushing to a container registry, and deploying to matching namespaces:
  * `master` ➔ `dev` namespace
  * `pre-release/v*` ➔ `preprod` namespace
  * `release/v*` ➔ `release` namespace

### CI Secrets & Rollback Strategy
* **Secrets Management:** Sensitive data (registry credentials and base64-encoded `KUBECONFIG_DATA`) are stored securely as GitHub Actions Secrets and injected dynamically at runtime.
* **Rollback Strategy:** Immediate recovery using Kubernetes native revision control:
  ```bash
  kubectl rollout undo deployment/backend-deployment -n preprod
  ```

---

## 4. Platform Services & Read/Write DB Routing (Task 5)

### Platform Services
* **Redis:** Deployed via `redis-deployment.yaml` and `redis-service.yaml` for caching and session handling.
* **Queues & Search:** Integrated service definitions for background workers and search platform.

### Read/Write Database Split
* **Environment Configuration:** Configured separate environment variables (`DB_WRITE_URL` for primary mutations and `DB_READ_URL` for read replicas).
* **Replication Lag Note:** Due to asynchronous replication, all write mutations (`INSERT`, `UPDATE`, `DELETE`) and critical consistent reads are strictly routed to the primary writer instance.

---

## 5. Observability, Scaling & Rate Limiting (Task 6)

* **Metrics & Logging:** Prometheus & Grafana configured for monitoring backend resource usage, request rates, and error rates. Fluent Bit DaemonSet ships container logs to Loki/Elasticsearch.
* **Autoscaling:** Horizontal Pod Autoscaler (`hpa-backend.yaml`) scales backend pods based on CPU utilization.
* **Rate Limiting:** Configured at the ingress layer to prevent API abuse and control traffic spikes.

---

## 6. Future Evolution
* **Microservices Scale:** To scale into a full microservices architecture, we can introduce an advanced API Gateway (Kong/Traefik), split the monolithic backend into dedicated domain services, and implement a service mesh (Istio) for enhanced security and distributed tracing.