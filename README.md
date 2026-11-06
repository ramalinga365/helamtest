# helamtest

Helm chart for the Guestbook demo app, deployed to MicroK8s with Argo CD.

```
helamtest/
├── argocd/
│   └── guestbook-application.yaml   # Argo CD Application
└── charts/guestbook/
    ├── Chart.yaml
    ├── values.yaml                  # image, replicas, NodePort 30081, ingress...
    └── templates/                   # deployment, service, ingress, hpa, helpers
```

## 1. Push to GitHub

Create an empty repo named `helamtest` on GitHub, then:

```bash
cd helamtest
git init
git add .
git commit -m "Add guestbook Helm chart"
git branch -M main
git remote add origin https://github.com/ramalinga365/helamtest.git
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

## Versioning (Git tags)

Each release is a Git tag (`v1.0.0`, `v1.1.0`, ...). Argo CD deploys the tag set in
`targetRevision`, so a deployment is pinned to an exact, unchangeable version.

| Tag | Changes |
|---|---|
| v1.0.0 | gb-frontend:v5, 3 replicas, NodePort 30081 |

### Release a new version

1. Make your change in `charts/guestbook/values.yaml` (or templates).
2. Bump `version:` in `charts/guestbook/Chart.yaml` (e.g. `1.1.0`).
3. Commit, tag and push:
   ```bash
   git commit -am "Release v1.1.0: <what changed>"
   git tag -a v1.1.0 -m "v1.1.0: <what changed>"
   git push origin main --tags
   ```
4. Deploy it: Argo CD UI → app → **APP DETAILS → EDIT** → **Target Revision** = `v1.1.0` → **SAVE** → **SYNC**.

### Roll back

Set **Target Revision** back to the previous tag (e.g. `v1.0.0`) → **SAVE** → **SYNC**.

### Version numbers (SemVer)

- `MAJOR` (2.0.0) – breaking change
- `MINOR` (1.1.0) – new feature / config change
- `PATCH` (1.0.1) – small fix

List versions: `git tag -l` or GitHub → **Tags**.
