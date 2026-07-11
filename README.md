# argocd-app-config

GitOps configuration repo for `myapp`. ArgoCD watches this repo and automatically syncs any changes to the target cluster -- no manual `kubectl apply` needed.

Application code lives in a separate repo. When a new image is built there, its CI pipeline updates the image tag in this repo, which triggers a sync.

## Repository structure

```
.
└── dev/
    ├── deployment.yaml   # Deployment definition (replicas, image, ports)
    └── service.yaml      # Service exposing the deployment on port 8080
```

## How it works

1. A new commit is pushed to the application repo.
2. CI builds and pushes a new container image, then updates the image tag in `dev/deployment.yaml`.
3. ArgoCD detects the drift between this repo and the cluster state.
4. ArgoCD syncs the change -- the cluster reflects the new desired state.

## Environments

| Environment | Manifests | Notes           |
|-------------|-----------|-----------------|
| dev         | `dev/`    | Active          |

## Making changes

- **Application changes** (new features, bug fixes): make changes in the application repo. CI will update the image tag here automatically.
- **Infrastructure changes** (replicas, resources, ports, config): edit the manifests in the relevant environment folder and push. ArgoCD will apply the changes.
