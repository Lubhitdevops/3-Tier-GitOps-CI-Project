---

# 🚀 Node.js CI – GitOps Workflow with Argo CD

This repository implements a **GitOps-driven CI/CD pipeline** for a **3-Tier Node.js Application** using **GitHub Actions**, **Docker**, **Trivy**, **Gitleaks**, and **Argo CD**.

The CI pipeline builds and pushes Docker images, performs security scans, and automatically updates a CD (Continuous Deployment) repository that triggers **Argo CD** to deploy changes to Kubernetes.

---

## 🏗 Project Overview

### 🧩 Architecture

This setup uses **two repositories** under a GitOps model:

| Repo Type         | Repository                              | Description                                                                                               |
| ----------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| **CI Repository** | `Lubhitdevops/3-Tier-GitOps-CI-Project` | Contains the Node.js frontend (`client`) and backend (`api`) code, along with GitHub Actions CI workflow. |
| **CD Repository** | `Lubhitdevops/3-Tier-GitOps-CD-Project` | Holds Kubernetes manifests (YAML files) monitored by **Argo CD** for automated deployments.               |

---

## ⚙️ CI Workflow Summary

### 🔹 1. **Code Compilation Check**

Verifies syntax for frontend (`client/`) and backend (`api/`) code:

```bash
find . -name "*.js" -exec node --check {} +
```

### 🔹 2. **Secret Scanning (Gitleaks)**

Ensures no hardcoded credentials or API keys are committed:

```bash
gitleaks detect --source ./client --exit-code 1
gitleaks detect --source ./api --exit-code 1
```

### 🔹 3. **Filesystem Vulnerability Scan (Trivy)**

Scans local project files for vulnerabilities before building:

```bash
trivy fs . --severity CRITICAL,HIGH
```

### 🔹 4. **Build and Push Docker Images**

Builds and pushes images for both services:

* **Backend:** `lubhitdocker/actions-backend:<commit-sha>`
* **Frontend:** `lubhitdocker/actions-frontend:<commit-sha>`

### 🔹 5. **Docker Image Vulnerability Scan**

Scans built Docker images for vulnerabilities:

```bash
trivy image lubhitdocker/actions-backend:<commit-sha>
trivy image lubhitdocker/actions-frontend:<commit-sha>
```

### 🔹 6. **Update CD Repository**

Automatically updates the Kubernetes manifests in the **CD repository** to reference the new image tags:

```bash
yq -i '.spec.template.spec.containers[].image = "lubhitdocker/actions-backend:<tag>"' backend.yaml
```

Argo CD then detects the change and redeploys the application.

---

## 🌐 Continuous Deployment with Argo CD

Once the CD repository (`3-Tier-GitOps-CD-Project`) is updated, **Argo CD** automatically:

1. Detects changes in the repo
2. Syncs new manifests
3. Deploys the updated application on the Kubernetes cluster

---

## ☁️ Infrastructure Setup

### 🖥 1. **Create EC2 Instance**

Create an EC2 instance on AWS to host your Kubernetes cluster and Argo CD:

```bash
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.medium \
  --key-name my-key \
  --security-group-ids sg-xxxxxxx
```

### ⚙️ 2. **Install Docker & Kubernetes**

```bash
sudo apt update && sudo apt install docker.io -y
curl -sfL https://get.k3s.io | sh -
kubectl get nodes
```

### 🚀 3. **Install Argo CD**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 🌍 4. **Access Argo CD Dashboard**

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access via browser: [https://localhost:8080](https://localhost:8080)

---

## 🧰 Technologies Used

| Category              | Tools / Technologies |
| --------------------- | -------------------- |
| **Version Control**   | Git, GitHub          |
| **Programming**       | Node.js, JavaScript  |
| **CI/CD Automation**  | GitHub Actions       |
| **Containerization**  | Docker               |
| **Security Scanning** | Gitleaks, Trivy      |
| **Orchestration**     | Kubernetes           |
| **GitOps Deployment** | Argo CD              |
| **Cloud Provider**    | AWS EC2              |
| **YAML Processor**    | yq                   |
| **OS / Platform**     | Ubuntu 22.04         |

---

## 🔒 Environment Variables & Secrets

| Name                 | Type            | Description                       |
| -------------------- | --------------- | --------------------------------- |
| `DOCKERHUB_USERNAME` | GitHub Variable | Docker Hub username               |
| `DOCKERHUB_PASSWORD` | GitHub Secret   | Docker Hub access token           |
| `CD_REPO_TOKEN`      | GitHub Secret   | Token to update CD repo manifests |

---

## 📈 Workflow Dependency Graph

```
compile
  ↓
gitleaks-scan
  ↓
trivy_fs_scan
  ↓
build_backend_docker_image_and_push
build_frontend_docker_image_and_push
  ↓
trivy_image_scan
  ↓
update_cd_repo → Argo CD syncs → Deployed on Kubernetes
```

---

## 📊 Workflow Summary Diagram

```
   ┌────────────┐
   │  Developer │
   └──────┬─────┘
          │ Push (main)
          ▼
   ┌──────────────┐
   │  GitHub CI   │
   │ (Actions)    │
   └──────┬───────┘
          │ Build, Scan, Push
          ▼
   ┌──────────────┐
   │  CD Repo     │
   │ (Manifest Update) │
   └──────┬───────┘
          │ Argo CD Sync
          ▼
   ┌──────────────┐
   │ Kubernetes   │
   │ Deployment   │
   └──────────────┘
```

---

## ✅ Final Outcome

✔️ Secure and automated Node.js CI/CD pipeline
✔️ Docker images automatically scanned and pushed
✔️ CD repo manifests updated automatically
✔️ Argo CD continuously deploys latest images to Kubernetes

---

## 🙌 Acknowledgements

Special thanks to:

* **Argo Project** – for making GitOps seamless and declarative.
* **Aqua Security** – for the Trivy vulnerability scanner.
* **Gitleaks** – for improving open-source security.
* **GitHub Actions Team** – for simplifying CI/CD automation.
* **Node.js Community** – for an amazing developer ecosystem.

---

## 📜 License

This project is licensed under the **MIT License** – you are free to use, modify, and distribute it with attribution.

```
MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

---

## 🧠 Author

**Lubhit Mawar**
Cloud Enthusiast | GitOps Practitioner
📦 [Docker Hub](https://hub.docker.com/u/lubhitdocker)
💻 [GitHub Profile](https://github.com/Lubhitdevops)

