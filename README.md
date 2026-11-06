# guestbook-helm

Helm chart for the Guestbook demo app, deployed to MicroK8s with Argo CD.

```
guestbook-helm/
├── argocd/
│   └── guestbook-application.yaml   # Argo CD Application
└── charts/guestbook/
    ├── Chart.yaml
    ├── values.yaml                  # image, replicas, NodePort 30081, ingress...
    └── templates/                   # deployment, service, ingress, hpa, helpers
```

## 1. Push to GitHub

Create an empty repo named `guestbook-helm` on GitHub, then:

```bash
cd guestbook-helm
git init
git add .
git commit -m "Add guestbook Helm chart"
git branch -M main
git remote add origin https://github.com/<your-github-user>/guestbook-helm.git
git push -u origin main
```

## 2. Point Argo CD at your repo

Edit `argocd/guestbook-application.yaml` → set `repoURL` to your repo URL, commit and push.

Private repo? Connect it first: Argo CD UI → **Settings → Repositories → Connect Repo**
(HTTPS, username + GitHub personal access token).

## 3. Deploy

If the earlier sample `guestbook` app exists, remove it first (same name):

```bash
kubectl -n argocd delete application guestbook
```

Then:

```bash
kubectl apply -f argocd/guestbook-application.yaml
kubectl -n argocd get applications
kubectl -n guestbook get pods,svc
```

Open: `http://<ubuntu2204-ip>:30081`

## 4. Update the app (GitOps)

Edit `charts/guestbook/values.yaml` (e.g. `replicaCount: 3`) → commit → push.
Argo CD syncs automatically (within ~3 min), or click **Refresh/Sync** in the UI.

## Validate locally (optional)

```bash
sudo snap install helm --classic
helm lint charts/guestbook
helm template guestbook charts/guestbook
```
