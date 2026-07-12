# My AWS DevOps Project

A complete end-to-end DevOps/SRE learning project built from scratch by **Vinay Kumar**, covering cloud hosting, containerization, CI/CD, Kubernetes, monitoring, and Infrastructure as Code.

## 👤 About

This project started as a simple personal portfolio website and evolved into a full production-grade DevOps pipeline — demonstrating real-world practices used by SRE and DevOps teams.

- **Author**: Vinay Kumar
- **Role**: Site Reliability Engineer
- **Email**: vinaykottour@gmail.com

---

## 🏗️ Project Evolution

```
Static HTML site on EC2
        ↓
Dockerized application
        ↓
Multi-environment CI/CD (dev/staging/prod)
        ↓
Docker Hub image registry with rollback support
        ↓
Kubernetes on AWS EKS
        ↓
Prometheus + Grafana monitoring
        ↓
Kubernetes Secrets management
        ↓
Terraform (Infrastructure as Code)
```

---

## 🛠️ Tech Stack

| Category | Tools |
|----------|-------|
| Cloud Provider | AWS (EC2, EKS, Load Balancer, IAM) |
| Web Server | Nginx |
| Containerization | Docker, Docker Hub |
| Orchestration | Kubernetes (AWS EKS) |
| CI/CD | GitHub Actions |
| Monitoring | Prometheus, Grafana |
| Infrastructure as Code | Terraform |
| Version Control | Git, GitHub |
| OS | Ubuntu (EC2 instances) |

---

## 📁 Repository Structure

```
my-project/
├── index.html                     # Portfolio website (HTML/CSS)
├── Dockerfile                     # Container build instructions
├── deployment.yaml                # Kubernetes Deployment config
├── service.yaml                   # Kubernetes Service (LoadBalancer)
├── README.md                      # This file
└── .github/
    └── workflows/
        ├── deploy-dev.yml         # CI/CD pipeline for dev branch
        ├── deploy-staging.yml     # CI/CD pipeline for staging branch
        └── deploy-prod.yml        # CI/CD pipeline for main/prod branch
```

---

## 🌍 Environments

| Environment | Branch | Deploys To | Trigger |
|-------------|--------|-----------|---------|
| Dev | `dev` | Dev EC2 (Docker container) | Push to `dev` |
| Staging | `staging` | Staging EC2 (Docker container) | Push to `staging` |
| Prod | `main` | AWS EKS (Kubernetes cluster) | Push to `main` |

---

## 🔄 CI/CD Pipeline Flow

```
Developer pushes code to dev branch
        ↓
GitHub Actions builds Docker image
        ↓
Pushes image to Docker Hub (tagged :dev / :staging / :latest + version number)
        ↓
Deploys to target environment:
  - Dev/Staging → SSH into EC2, docker pull + docker run
  - Prod → kubectl set image (triggers Kubernetes rolling update)
        ↓
Merge dev → staging → main promotes code through environments
```

### Docker Hub Tags
```
vinaykottour/my-portfolio
├── :dev       → latest dev build
├── :staging   → latest staging build
├── :latest    → current production build
└── :<run_number>  → versioned builds for rollback (e.g. :12, :13)
```

---

## 🐳 Docker

The app is containerized using a lightweight Nginx Alpine base image:

```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## ☸️ Kubernetes (AWS EKS)

Production runs on a managed Kubernetes cluster on AWS EKS.

**Cluster details:**
- Cluster name: `my-portfolio-cluster`
- Region: `us-east-1`
- Node type: `t3.small`
- Nodegroup: `my-nodes-v2` (min 1, max 3, auto-scaling enabled)

**Key features implemented:**
- **Deployment** with 2 replicas for high availability
- **Pod Anti-Affinity** — ensures replicas never run on the same node, so a single node failure doesn't take down the whole app
- **Rolling Update strategy** tuned (`maxSurge: 0`, `maxUnavailable: 1`) to work safely alongside anti-affinity on a small cluster
- **LoadBalancer Service** — exposes the app via an AWS Elastic Load Balancer with a public URL
- **Kubernetes Secrets** — sensitive values (e.g. admin email, API key) injected as environment variables at runtime instead of being hardcoded

**Useful commands:**
```bash
kubectl get nodes                          # list worker nodes
kubectl get pods -o wide                   # see pods and which node they're on
kubectl get svc portfolio-service          # get the public Load Balancer URL
kubectl rollout status deployment/portfolio-deployment   # watch a deployment update
kubectl rollout undo deployment/portfolio-deployment     # rollback to previous version
```

---

## 🔁 Rollback Strategy

Every prod deploy is tagged with both `:latest` and a unique version number (`${{ github.run_number }}`), so any previous version can be restored:

**Manual rollback (fastest, emergency use):**
```bash
docker stop portfolio && docker rm portfolio
docker pull vinaykottour/my-portfolio:<version>
docker run -d -p 80:80 --name portfolio vinaykottour/my-portfolio:<version>
```

**GitHub Actions rollback (tracked, team-visible):**
A `rollback.yml` workflow accepts a version number as input and redeploys that version to prod via `workflow_dispatch` — triggered manually from the Actions tab.

---

## 📊 Monitoring — Prometheus & Grafana

Installed on the EKS cluster via Helm (`kube-prometheus-stack`), providing:

- **Prometheus** — scrapes and stores metrics from nodes, pods, and the Kubernetes API
- **Grafana** — pre-built dashboards for cluster health, namespace/pod resource usage, and node metrics
- **Alerting** — a custom alert rule (`Portfolio Pod Down Alert`) watches the count of Ready portfolio pods and fires if it drops below the expected replica count, verified by manually deleting a pod and observing Kubernetes self-heal while the alert correctly fired and resolved

```bash
helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring
kubectl get pods -n monitoring
kubectl get svc monitoring-grafana -n monitoring   # get Grafana's public URL
```

---

## 🔐 Kubernetes Secrets

Sensitive configuration is stored using native Kubernetes Secrets rather than being hardcoded in YAML or Docker images:

```bash
kubectl create secret generic portfolio-secret \
  --from-literal=admin-email=vinaykottour@gmail.com \
  --from-literal=api-key=MySecretApiKey123
```

Referenced in `deployment.yaml` via `secretKeyRef` and injected as environment variables at container startup. Note: values are base64-encoded (not encrypted) — for true production security this would be layered with AWS KMS or a dedicated secrets manager.

---

## 🏗️ Infrastructure as Code — Terraform

Terraform is being introduced to manage infrastructure declaratively instead of manual console clicks or one-off CLI commands:

```bash
terraform init      # set up working directory and provider plugins
terraform plan       # preview changes before applying
terraform apply       # create/update infrastructure
terraform destroy    # tear down everything Terraform manages, safely and in order
```

Currently used to manage a test EC2 instance as a learning exercise, with plans to eventually manage the EKS cluster itself.

---

## 💰 Cost Awareness

Since AWS resources bill by the hour, the following are stopped/deleted when not actively in use:
- EC2 instances (dev, staging, prod, k8s-control-server)
- EKS cluster and its nodegroup (~$0.16/hr combined when running)
- Load Balancers (created automatically by Kubernetes Services)

---

## 📬 Contact

- **Email**: vinaykottour@gmail.com
- **Role**: Site Reliability Engineer
