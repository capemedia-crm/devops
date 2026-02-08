# DevOps – Fleet manifests and deployment

Central repo for **Kubernetes / Rancher Fleet** manifests. Each app has its own Fleet bundle; one GitRepo in Fleet points at this repo and deploys all bundles.

## Layout

```
devops/
├── content-service/          # Fleet bundle
│   ├── fleet.yaml
│   └── manifests/
├── user-service/             # (add when migrating from app repo)
│   ├── fleet.yaml
│   └── manifests/
└── README.md
```

- **App directories** (e.g. `content-service/`) are Fleet bundle roots: each contains a `fleet.yaml` and `manifests/`.
- **GitRepo** in Fleet should reference this repo and list each bundle path, e.g. `paths: ["content-service", "user-service"]`.

## Fleet GitRepo example

```yaml
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: capemedia-apps
  namespace: fleet-default
spec:
  repo: https://github.com/capemedia-crm/devops
  branch: main
  paths:
    - content-service
    # - user-service
  pollingInterval: 60s
  # secretRef: for private repo
```

## Per-app notes

- **content-service**: Image `ghcr.io/capemedia-crm/content-service:dev` (or `:main`). Requires secret `content-service-env` and optional `ghcr-pull-secret` in namespace `content-service`.
- Add **user-service** (and others) as sibling bundle dirs when ready.

## CI/CD flow

- **App repos** (content-service, user_service, …): CI builds and pushes images to GHCR (e.g. `:dev`, `:main`, `:sha-xxx`). No manifests in app repos.
- **This repo**: Only manifest changes. Fleet syncs from here and deploys. To roll a new image, either use a moving tag (`:dev`) so no commit here is needed, or have app CI update the image tag in this repo and commit.
