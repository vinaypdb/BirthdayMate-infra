## 🚀 Part 2: Kubernetes Deployment with GitOps (Argo CD)

This section outlines the complete GitOps-based deployment of the BirthdayMate application to a Kubernetes cluster using Argo CD and Minikube.

### 📦 Folder Structure

```
BirthdayMate-infra/
├── argo/
│   └── birthdaymate-app.yaml   # Argo CD Application manifest
└── k8s/
    ├── deployment.yaml         # Kubernetes Deployment
    └── services.yaml           # Kubernetes Service
```

---

### ✅ Step-by-Step Deployment Workflow

#### 1️⃣ Install & Start Minikube

```bash
minikube start
minikube ip  # Note the IP address (e.g., 192.168.49.2)
```

#### 2️⃣ Install Argo CD in Minikube

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> Access Argo CD UI:

```bash
kubectl port-forward svc/argocd-server -n argocd 8081:443
# Then visit: https://localhost:8081
```

> Get Argo CD admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

#### 3️⃣ Create Deployment & Service YAMLs

📄 `k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: birthdaymate-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: birthdaymate
  template:
    metadata:
      labels:
        app: birthdaymate
    spec:
      containers:
      - name: birthdaymate
        image: vinaypdb/birthdaymate:latest
        ports:
        - containerPort: 9090
```

📄 `k8s/services.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: birthdaymate-service
spec:
  type: NodePort
  selector:
    app: birthdaymate
  ports:
    - port: 80
      targetPort: 9090
      nodePort: 30090
```

#### 4️⃣ Apply Locally (Test Without Argo CD)

```bash
kubectl apply -f k8s/
kubectl get pods -l app=birthdaymate
kubectl get svc birthdaymate-service
```

> Access the app:

```
http://<minikube-ip>:30090
```

---

### 🔁 GitOps with Argo CD

#### 5️⃣ Create Argo CD Application

📄 `argo/birthdaymate-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: birthdaymate
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/vinaypdb/BirthdayMate-infra
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

> Apply the manifest:

```bash
kubectl apply -f argo/birthdaymate-app.yaml
```

#### 6️⃣ Argo CD Auto-Sync in Action

* Any changes pushed to the `BirthdayMate-infra` repo (under `k8s/`) will automatically sync to the cluster.
* To test, modify `deployment.yaml`, push to GitHub, and watch Argo CD auto-deploy the update.

---

### 🧪 GitOps Deployment Verification

1. ✅ Commit a change to the repo:

```bash
git add k8s/deployment.yaml
git commit -m "Test auto-sync: updated label"
git push
```

2. ✅ Argo CD will auto-sync and rollout the change in the UI.

3. ✅ You’ll see a new ReplicaSet and updated Pod on the Argo CD dashboard.

---

### 🎉 Summary

✅ We built and containerized the Go app.
✅ Set up CI with GitHub Actions and security scans.
✅ Deployed to Kubernetes via `kubectl` for local testing.
✅ Shifted to GitOps-style delivery using Argo CD.
✅ Verified auto-sync on Git changes to production state.

---

### 🙌 Author

Vinay Pedapuri ∙ [GitHub](https://github.com/vinaypdb) ∙ [Docker Hub](https://hub.docker.com/u/vinaypdb)

