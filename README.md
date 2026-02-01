# home-gitops

This repository defines the GitOps control plane for my home Kubernetes cluster.

It contains Argo CD `Application` manifests that declaratively manage all workloads deployed to the cluster. No application manifests are applied manually; everything is reconciled from Git.

---

## What this repository is responsible for

- Declaring Argo CD `Application` resources
- Defining which repositories are deployed to the cluster
- Controlling namespaces, sync behaviour, and ordering
- Acting as the single source of truth for cluster state

This repository does **not** contain application code or Helm charts.

---

## GitOps model

Each application lives in its **own repository** and provides:
- a Helm chart
- default values
- application-specific configuration

This repository references those repos via Argo CD.

GitHub → Argo CD → Kubernetes


## Repository structure

```.
├── namespaces/
│ └── jellyfin.yaml
│
├── applications/
│ ├── jellyfin.yaml
│ └── pilotedge-audio-fetcher.yaml
│
└── README.md
```
---

## Applications managed

### Jellyfin
- Repository: `home-media`
- Type: Deployment
- Namespace: `jellyfin`
- Role: Media consumer

### PilotEdge Audio Fetcher
- Repository: `pilotedge-audio-fetcher`
- Type: CronJob
- Namespace: `jellyfin`
- Role: Audio producer (owns PVC)

Both applications intentionally deploy into the same namespace to allow shared access to persistent storage.

---

## Namespace model

Namespaces are managed declaratively and created via Argo CD.

Currently defined namespaces:
- `jellyfin` – media services and producers

---

## Storage ownership

Persistent volumes are **owned by the application that produces the data**.

- Producers define PVCs
- Consumers mount them read-only
- Applications can be replaced without data loss

This avoids coupling storage lifecycle to a specific consumer.

---

## Sync behaviour

All applications use automated sync:

- `prune: true`
- `selfHeal: true`

This ensures cluster state always converges back to Git.

Manual changes in the cluster are treated as drift.

---

## Usage

Once Argo CD is installed and bootstrapped, applying this repository is sufficient to recreate the entire cluster state.

This repo is intended to be boring, stable, and rarely changed.

---

## Notes

- Designed for homelab / learning use
- Currently tested on kind and k3s
