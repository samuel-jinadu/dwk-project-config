# dwk-project-config

Kubernetes manifests and deploy automation for the DWK 2026 project (exercise 4.10).

Application code lives in [dwk-project-app](https://github.com/samuel-jinadu/dwk-project-app).

## Layout

| Path | Purpose |
|---|---|
| `gitops/base/` | Base Kustomize overlays (todo-app, todo-backend, broadcaster, cronjobs) |
| `gitops/staging/` | Staging overlay — `LOG_ONLY=true`, no backup |
| `gitops/production/` | Production overlay — backup CronJob + public HTTPRoutes |
| `gitops/log-output/` | `exercises` namespace log-output app |
| `argocd/` | Argo CD Application CRs (deployed by `deploy-gke.sh` §9) |
| `gke-manifests/` | Cluster-level: Gateway, canary Rollout, RBAC, HTTPRoute, AnalysisTemplate |
| `manifests/namespaces/` | Namespace definitions |
| `postgres/` | StatefulSet + SOPS-encrypted credentials |
| `broadcaster/manifests/` | SOPS-encrypted webhook secret |
| `helm/` | Helm values for kube-prometheus-stack, Loki |
| `deploy-gke.sh` | End-to-end deploy (cluster + namespaces + Gateway + Argo CD + Argo Rollouts + NATS + apps) |

## Deploy

```bash
./deploy-gke.sh                                   # → staging (default)
TARGET_NS=production TARGET_TAG=v1.2.3 ./deploy-gke.sh   # → production
```

Requires: `kubectl`, `helm`, `sops`, `age`, `gcloud` (or the floci-gcp emulator).

## Secrets

Encrypted with [SOPS](https://github.com/getsops/sops) using an age recipient declared in `.sops.yaml`. The age private key is **never** committed — provide it via `SOPS_AGE_KEY` env var (or `~/.config/sops/age/keys.txt`) before running any script that decrypts. To re-encrypt after editing a plaintext secret, run `./encrypt.sh`.

## CI (`.github/workflows/`)

- `emulator-smoke-test.yaml` — full end-to-end test against the floci-gcp emulator (staging + production + backup + broadcaster fan-out). Gated by `ENABLE_EMULATOR_SMOKE_TEST=true` repo variable + `SOPS_AGE_KEY` secret.
- `deploy-gke.yaml` — real-GKE deploy on tag push. Gated by `DISABLE_GKE_DEPLOY` variable + GCP WIF secrets.
- `delete-env.yaml` — deletes the per-branch namespace on branch deletion.
