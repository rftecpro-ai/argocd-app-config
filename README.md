# argocd-app-config

GitOps configuration repo for `myapp`. ArgoCD watches this repo and automatically syncs any changes to the target cluster -- no manual `kubectl apply` needed.

Application code lives in a separate repo. When a new image is built there, its CI pipeline updates the image tag in this repo, which triggers a sync.

## Repository structure

```
.
└── dev/
    ├── application.yaml  # ArgoCD Application resource (registers this repo with ArgoCD)
    ├── deployment.yaml   # Deployment definition (replicas, image, ports)
    └── service.yaml      # Service exposing the deployment on port 8080
```

## ArgoCD Application

`application.yaml` is the ArgoCD `Application` CRD that registers this repo with ArgoCD. It is applied once to bootstrap the setup -- after that, ArgoCD manages the rest automatically.

| Field          | Value                                             |
|----------------|---------------------------------------------------|
| App name       | myapp-argo-application                            |
| Watched repo   | https://github.com/rftecpro-ai/argocd-app-config |
| Watched path   | `dev/`                                            |
| Target cluster | https://kubernetes.default.svc (in-cluster)       |
| Target namespace | myapp (auto-created if missing)                 |
| Sync policy    | Automated, self-heal enabled, prune enabled       |

> **Note:** With `selfHeal: true`, ArgoCD will revert any manual changes made directly to the cluster. With `prune: true`, resources removed from this repo will also be deleted from the cluster. All changes should go through this repo.

## How it works

1. Apply `dev/application.yaml` once to register the app with ArgoCD (bootstrap step).
2. ArgoCD begins watching the `dev/` path in this repo.
3. A new commit is pushed to the application repo.
4. CI builds and pushes a new container image, then updates the image tag in `dev/deployment.yaml`.
5. ArgoCD detects the drift between this repo and the cluster state.
6. ArgoCD syncs the change -- the cluster reflects the new desired state.

## Environments

| Environment | Manifests | Notes  |
|-------------|-----------|--------|
| dev         | `dev/`    | Active |

## Making changes

- **Application changes** (new features, bug fixes): make changes in the application repo. CI will update the image tag here automatically.
- **Infrastructure changes** (replicas, resources, ports, config): edit the manifests in the relevant environment folder and push. ArgoCD will apply the changes.
- **Do not apply changes directly to the cluster** -- ArgoCD will revert them. This repo is the single source of truth.
