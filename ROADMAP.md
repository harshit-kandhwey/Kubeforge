# Roadmap

Sequencing and exit criteria. Versions and order are intent, not commitments; [CHANGELOG.md](CHANGELOG.md) records what has actually shipped and [docs/design-decisions.md](docs/design-decisions.md) records why. A phase is complete only when its exit criteria are demonstrated, not when the code is written.

## Principles

- **Local first.** Everything that can be proven on kind is proven there before a cloud cluster exists.
- **One cloud, then the rest.** EKS is built end to end before GKE or AKS is touched.
- **Only what is built is claimed.** README, resume and repository description change when a phase completes, never before.
- **Everything is code.** No manual console or CLI changes to anything Terraform or GitOps manages.

## Phase 0: Repository foundations (current)

Repository standards, CI hygiene checks, secret scanning, architecture outline and design-decision record.

Exit criteria: CI is green on `main`, a fresh clone passes `pre-commit run --all-files`, and the README states truthfully what exists.

## Phase 1: Local cluster and Helm chart

- kind cluster and a Helm chart for the API, worker, beat, Postgres and Redis.
- Liveness, readiness and startup probes, resource requests and limits, graceful shutdown.
- HPA on the API, KEDA on queue depth for workers, PodDisruptionBudgets, topology spread.
- Migrations as a Job, not per-pod startup.
- NetworkPolicies, Pod Security Standards, non-root and read-only containers.

Exit criteria: a load test makes the HPA and KEDA scale out and back in, and killing pods and a node does not drop requests. Both are recorded in the docs with the measured numbers.

## Phase 2: EKS with Terraform

- Terraform modules and environments, remote state with locking, OIDC identity for workloads.
- Ingress, TLS and DNS automation, External Secrets backed by Secrets Manager.
- CI that builds, scans, signs and pushes images using short-lived OIDC credentials.
- Ephemeral by default: create for a session, destroy afterwards.

Exit criteria: one command creates the cluster and deploys the workload, one command destroys it, and the cost of a session is measured and documented.

## Phase 3: GitOps and progressive delivery

- Argo CD with an app-of-apps layout so Git is the only deploy path.
- Argo Rollouts with blue-green or canary and an automatic rollback on failed analysis.

Exit criteria: a bad release is rolled back automatically, and the whole demonstration is reproducible from a clean cluster.

## Phase 4: Observability and SLO

- kube-prometheus-stack, Grafana dashboards, Loki logs.
- Alerts that each have an owner and an expected response, and an SLO with burn-rate alerting.

Exit criteria: an induced failure fires the intended alert and nothing else.

## Phase 5: Hardening and drills

- Policy engine rules, image and manifest scanning gates.
- Backup and restore drill, and a written cluster-upgrade runbook that has been followed once.
- Chaos experiments and a cost view.

Exit criteria: a restore has been performed from backup on a clean cluster and an upgrade has been executed following the runbook.

## Phase 6: GKE and AKS

- Thin `gke` and `aks` Terraform modules and per-cloud values.
- Identity, ingress, storage class and secrets-backend differences documented as they are found.

Exit criteria: the same chart and GitOps definition deploy the workload on all three clouds.

## Out of scope

Cross-cloud failover, a multi-cluster service mesh, a single control plane across providers (Cluster API or Crossplane), and running the platform for other teams. These are deliberately excluded; see [docs/design-decisions.md](docs/design-decisions.md#multi-cluster-not-cross-cloud).
