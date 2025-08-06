# 🚀 GitOps in Action: Automated Deployments with Argo CD & Kubernetes

Set up a GitOps workflow using ArgoCD to automatically deploy and manage a web application on a Kubernetes cluster.
Imagine pushing a change to your code, closing your laptop, and knowing that within seconds your Kubernetes cluster will:
✅ Build the new container
✅ Deploy it automatically
✅ Heal itself if anything breaks — all without you touching kubectl.

That’s the magic of GitOps — where your Git repository becomes the single source of truth, and tools like Argo CD ensure your cluster always matches that truth.

In this project, you’ll build exactly that:
A self-updating Flask web application running on Kubernetes, deployed and managed entirely through GitOps principles, with Argo CD doing all the heavy lifting.
Push code → watch it go live → break something manually → watch it fix itself.


---

## 🌟 Features

* **Git as the Single Source of Truth** – Application code and Kubernetes configurations are stored in Git.
* **Automated Continuous Delivery** – Argo CD continuously monitors the Git repository for changes and applies them to the cluster.
* **Automatic Reconciliation (Drift Detection)** – Detects and corrects manual changes to match the Git state.
* **Containerization** – Flask application containerized using Docker.
* **Kubernetes Deployment** – Application deployed and managed on Kubernetes.
* **Visual Monitoring** – Argo CD UI for real-time app health and sync status.

---

## 🛠️ Prerequisites

Ensure you have the following installed:

* **Git** – For version control
* **Docker Desktop** – With Kubernetes enabled
* **kubectl** – Kubernetes command-line tool
* **helm** – For installing Argo CD

---

## 📦 Project Setup

### 1️⃣ Create and Clone Repositories
Separating **code** from **configuration** is a core and foundational principle of the GitOps methodology.

```bash
# Create and clone your application repository
git clone https://github.com/<your-username>/my-sample-app.git

# Create and clone your GitOps repository
git clone https://github.com/<your-username>/my-gitops-repo.git
```

---

### 2️⃣ Prepare Application Code

**Application repo (`my-sample-app/`)**:

```
my-sample-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── templates/
    └── index.html
```

**GitOps repo (`my-gitops-repo/`) contains deployment & service yaml file**:

```
my-gitops-repo/
└── dev/
    └── app.manifests.yaml
```

---

### 3️⃣ Build & Push Docker Image

```bash
cd my-sample-app

# Build the Docker image (replace 'username' with your Docker Hub username)
docker build -t username/buzzgen:latest .

# Log in to Docker Hub
docker login
docker tag buzzgen:latest username/buzzgen:latest
# Push the image
docker push username/buzzgen:latest
```

---

## ⚙️ Argo CD Installation & Setup

### Step 1 – Create Namespace

```bash
kubectl create namespace argocd
```

---

### Step 2 – Install Argo CD via Helm

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd --namespace argocd
```

---

### Step 3 – Access Argo CD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open [https://localhost:8080](https://localhost:8080) in your browser.

Retrieve initial admin password:

```bash
 kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

---

## 🚀 Deploy Application via Argo CD

1. Log in to Argo CD UI.
2. Click **+ New App** → Fill in details:

**General**

* Application Name: `my-web-app`
* Project: `default`
* Sync Policy: **Automatic** (PRUNE + SELF HEAL)

**Source**

* Repository URL: `https://github.com/<your-username>/my-gitops-repo.git`
* Revision: `HEAD`
* Path: `dev`

**Destination**

* Cluster URL: `https://kubernetes.default.svc`
* Namespace: `my-web-app-ns` (Enable **AUTO-CREATE NAMESPACE**)

3. Click **CREATE** — Argo CD will sync your app automatically.

---

## 🏃 Access the Application

Port-forward the service:

```bash
kubectl port-forward svc/buzzgen-service -n my-web-app-ns 8082:80
```

Then open: [http://localhost:8082](http://localhost:8082)

---

## 💡 Demonstrating GitOps Principles

### 1. Automatic Content Update

```bash
# Edit the HTML
nano my-sample-app/templates/index.html

# Commit & push changes
git add templates/index.html
git commit -m "feat: updated homepage content via GitOps"
git push origin main

# Rebuild & push Docker image
docker build -t username/buzzgen:latest .
docker push username/buzzgen:latest
```

Argo CD will detect and deploy changes automatically.

---

### 2. Automatic Reconciliation (Drift Correction)

```bash
# Check pods
kubectl get pods -n my-web-app-ns

# Scale manually (introduce drift)
kubectl scale --replicas=3 deployment/my-app -n my-web-app-ns
```

Argo CD will detect the drift and revert to the Git-defined state.

---

## ⚠️ Troubleshooting

| Issue                                | Cause                         | Fix                                |
| ------------------------------------ | ----------------------------- | ---------------------------------- |
| `ERR_CONNECTION_REFUSED`             | Port not forwarded / in use   | Use another port (e.g., `8083:80`) |
| `jinja2.exceptions.TemplateNotFound` | HTML missing in Docker image  | Rebuild & push image               |
| `ImagePullBackOff`                   | Image not found in Docker Hub | Check name, login, push again      |
| Argo CD stuck `OutOfSync`            | Sync failure                  | Click **SYNC** in Argo CD UI       |

---

## 🧹 Cleanup

```bash
helm uninstall argocd --namespace argocd
```

---

## 🤝 Contribution

This project is a **foundational GitOps example** — feel free to fork, enhance, and integrate with other CI/CD tools.

