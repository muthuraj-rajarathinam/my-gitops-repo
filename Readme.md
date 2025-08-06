# 🚀 GitOps in Action: Automated Deployments with Argo CD & Kubernetes

This project provides a **hands-on, end-to-end demonstration** of GitOps principles using a simple Flask web application, Docker, Kubernetes, and Argo CD.
It showcases how **declaring your desired application state in Git** can drive fully automated and self-healing deployments to your Kubernetes cluster.

---

## 🌟 Features

* **Git as the Single Source of Truth** – All application code and Kubernetes configurations are managed in a Git repository.
* **Automated Continuous Delivery** – Argo CD continuously monitors the Git repository for changes and automatically synchronizes the Kubernetes cluster to match the desired state.
* **Automatic Reconciliation (Drift Detection)** – Argo CD detects and corrects any manual changes made directly to the cluster, ensuring the live state always matches Git.
* **Containerization** – The Flask application is containerized using Docker.
* **Kubernetes Deployment** – The application is deployed and managed on a Kubernetes cluster.
* **Visual Monitoring** – Leverage Argo CD's intuitive UI for real-time visibility into application health and synchronization status.

---

## 🛠️ Prerequisites

Ensure you have the following installed and configured:

* **Git** – For version control
* **Docker Desktop** – Includes Docker Engine and a local Kubernetes cluster (enable Kubernetes in settings)
* **kubectl** – Kubernetes command-line tool
* **Argo CD** – Installed and running on your Kubernetes cluster, with access to the Argo CD UI

---

## 🚀 Project Setup

### 1️⃣ Create two Repositories And clone it in Local

```bash
# Clone your application code & Docker files
create git repo my-sample-app

# clone your deployment.file & service.yaml to single yaml file
create git  my-gitops-repo
```
Clone it in Local

---

### 2️⃣ Prepare Your Application Code

Ensure your `my-sample-app` repository has the correct structure:

```
my-sample-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── templates/
    └── index.html
```
Ensure your `my-gitops-repo` repository has the correct structure:

```
my-gitops-repo/
└── devv/
    └── app.manifests.yaml

---

### 3️⃣ Build & Push Your Docker Image

```bash
cd my-sample-app

# Build the Docker image (replace 'Username' with your Docker Hub username)
docker build -t username/buzzgen:latest .

# Log in to Docker Hub (if not already logged in)
docker login

# Push the image to Docker Hub
docker push username/buzzgen:latest
```

---

### 4️⃣ Configure Argo CD Application

1. Open the Argo CD UI: [https://localhost:8080](https://localhost:8080)
2. Log in:

   * Username: `admin`
   * Password: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
3. Click **+ New App** or **CREATE APPLICATION**
4. Fill in details:

**General**

* Application Name: `my-web-app`
* Project: `default`
* Sync Policy: Automatic (select **PRUNE** and **SELF HEAL**)

**Source**

* Repository URL: `https://github.com/muthuraj-rajarathinam/my-gitops-repo.git`
* Revision: `HEAD`
* Path: `dev`

**Destination**

* Cluster URL: `https://kubernetes.default.svc`
* Namespace: `my-web-app-ns` (check **AUTO-CREATE NAMESPACE**)

5. Click **CREATE**
6. Argo CD will start syncing your application

---

## 🏃 Running & Accessing the Application

**Port Forward the Service:**

```bash
kubectl port-forward svc/buzzgen-service -n my-web-app-ns 8082:80
```

Leave this terminal running, then open:
[http://localhost:8082](http://localhost:8082)

---

## 💡 Demonstrating GitOps Principles

### 1. Automatic Content Update (Desired State Change)

```bash
# Edit the HTML template
nano my-sample-app/templates/index.html
# Change <h1> and <p> content

# Commit changes
cd my-sample-app
git add templates/index.html
git commit -m "feat: updated homepage content via GitOps"
git push origin main

# Rebuild & push Docker image
docker build -t muthuraj07/buzzgen:latest .
docker push muthuraj07/buzzgen:latest
```

Watch Argo CD automatically deploy the change.

---

### 2. Automatic Reconciliation (Configuration Drift)

```bash
# Check current pods
kubectl get pods -n my-web-app-ns

# Manually scale deployment (introduce drift)
kubectl scale --replicas=3 deployment/my-app -n my-web-app-ns
```

Argo CD will detect the drift and scale back to the Git-defined replica count.

---


## ⚠️ Troubleshooting

| Issue                                | Cause                               | Fix                                  |
| ------------------------------------ | ----------------------------------- | ------------------------------------ |
| `ERR_CONNECTION_REFUSED`             | Port not forwarded or in use        | Use different port (e.g., `8083:80`) |
| `jinja2.exceptions.TemplateNotFound` | HTML file missing from Docker image | Rebuild & push Docker image          |
| `ImagePullBackOff`                   | Image not found in Docker Hub       | Check image name, login, and push    |
| Argo CD stuck `OutOfSync`            | Sync failure                        | Click **SYNC** in Argo CD UI         |

---

## 🤝 Contribution

This project is a **foundational GitOps example** — feel free to fork, extend, and integrate with more CI/CD tools.

---

If you want, I can also **add badges** (Docker Hub, GitHub Actions, Kubernetes, Argo CD) at the top so your README looks even more professional.
