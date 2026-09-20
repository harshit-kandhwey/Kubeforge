# Kubeforge

A production-grade Kubernetes platform, built hands-on: a real multi-service workload run the way a production team would, on EKS first and then on GKE and AKS, with everything defined as code.

> **Status: bootstrap.** The repository standards, design record and roadmap exist. Nothing is deployed yet. This README marks every capability as **Current** or **Planned**, and a capability moves to Current only when it works and has been demonstrated.

## Contents

- [What it is](#what-it-is)
- [Workload](#workload)
- [Architecture](#architecture)
- [Capabilities](#capabilities)
- [Phases](#phases)
- [Repository layout](#repository-layout)
- [Getting started](#getting-started)
- [Cost and safety](#cost-and-safety)
- [Documentation](#documentation)
- [Contributing, security, license](#contributing-security-license)

## What it is

Kubeforge is the platform half of a two-repo project. It answers one question: what does it take to run a small multi-service application on Kubernetes the way a production team would?

- **Infrastructure as code.** Clusters, networking and identity are created only through Terraform, never by hand.
- **GitOps delivery.** Git is the only way a change reaches a cluster.
- **Operable.** Autoscaling, health probes, rollouts with rollback, observability with an SLO, backup and restore, and an upgrade runbook.
- **Secure by default.** Non-root containers, network policies, least-privilege identity, secrets from a secrets manager, and policy checks.
- **Multi-cluster, one definition.** The same chart and GitOps setup run on EKS, GKE and AKS, and only a thin per-cloud layer differs.

**Non-goals.** This is not cross-cloud failover, a multi-cluster service mesh, or a single control plane spanning providers. Those are separate projects. See [docs/design-decisions.md](docs/design-decisions.md#multi-cluster-not-cross-cloud).

## Workload

The application is [Nexusdeploy-App](https://github.com/harshit-kandhwey/Nexusdeploy-App): a Flask project-management REST API with Celery workers and a beat scheduler, backed by Postgres and Redis. Kubeforge consumes it only as container images and never modifies its code. The contract (entrypoints, health endpoints, environment variables) is owned by that repo. See [docs/design-decisions.md](docs/design-decisions.md#separate-repos).

## Architecture

Planned target, not yet built. Details in [docs/architecture.md](docs/architecture.md).

```
   GitHub Actions (OIDC)                 Git (this repo)
   build, scan, sign, push               charts + cluster config
          │                                     │
          ▼                                     ▼
   container registry ─────────────►  Argo CD (per cluster)
                                              │
        ┌─────────────────────────────────────┼──────────────────────────┐
        ▼                                     ▼                          ▼
   EKS (AWS)                             GKE (GCP)                  AKS (Azure)
   ingress + TLS                         ingress + TLS              ingress + TLS
   API │ workers │ beat                   same chart                 same chart
   Postgres │ Redis                       per-cloud values           per-cloud values
   Prometheus │ Grafana │ Loki
```

## Capabilities

| Capability                                                  | Status  |
| ----------------------------------------------------------- | ------- |
| Repository standards, CI hygiene checks, secret scanning    | Current |
| Design-decision record and roadmap                          | Current |
| Local cluster on kind, Helm chart, probes, limits           | Planned |
| HPA on the API, KEDA on queue depth for workers, PDBs       | Planned |
| NetworkPolicies, Pod Security Standards, RBAC               | Planned |
| EKS provisioned with Terraform (VPC, node groups, OIDC)     | Planned |
| Ingress, TLS and DNS automation                             | Planned |
| CI: build, image scan, signing, push over OIDC              | Planned |
| Argo CD GitOps, app-of-apps                                 | Planned |
| Progressive delivery with automatic rollback                | Planned |
| External Secrets, policy engine                             | Planned |
| Prometheus, Grafana, Loki, alerts, SLO with burn-rate       | Planned |
| Load and chaos tests demonstrating autoscaling and recovery | Planned |
| Backup and restore drill, cluster upgrade runbook           | Planned |
| Same chart on GKE and AKS                                   | Planned |

## Phases

Detail and exit criteria are in [ROADMAP.md](ROADMAP.md).

| Phase | Scope                                             | Status  |
| ----- | ------------------------------------------------- | ------- |
| 0     | Repository foundations                            | Current |
| 1     | Local cluster and Helm chart                      | Planned |
| 2     | EKS with Terraform, ingress, TLS, secrets, CI     | Planned |
| 3     | GitOps and progressive delivery                   | Planned |
| 4     | Observability and SLO                             | Planned |
| 5     | Hardening, load, chaos, backup and upgrade drills | Planned |
| 6     | GKE and AKS                                       | Planned |

## Repository layout

Only the files that exist today are listed as current. The rest is the intended layout and is created when its phase starts.

```
.github/            CI, Dependabot, issue and PR templates          (current)
docs/               architecture and design-decision record         (current)
terraform/          modules/ (eks, gke, aks) and environments/      (planned, phase 2)
charts/             the Helm chart for the workload                 (planned, phase 1)
gitops/             Argo CD app-of-apps and ApplicationSets         (planned, phase 3)
clusters/           per-cloud values: aws/, gcp/, azure/            (planned, phase 2 and 6)
```

## Getting started

There is nothing to run yet. When phase 1 lands, this section will hold the exact commands to create the local cluster and deploy the chart, and each later phase adds its own.

Planned prerequisites: Docker, `kind`, `kubectl`, `helm`, Terraform, and the AWS CLI.

## Cost and safety

Managed clusters cost money while they exist. Every cloud cluster is ephemeral: created for a session and destroyed afterwards by Terraform, and any step that creates cloud resources states an estimated cost first. Local development runs on kind and costs nothing. No kubeconfig, Terraform state or credential is ever committed; secret scanning runs in pre-commit and in CI.

## Documentation

- [docs/architecture.md](docs/architecture.md): the target design and the boundary with Nexusdeploy-App.
- [docs/design-decisions.md](docs/design-decisions.md): why each non-obvious choice was made, with anchors.
- [ROADMAP.md](ROADMAP.md): phases, exit criteria, and what is deliberately out of scope.
- [CHANGELOG.md](CHANGELOG.md): what has shipped.

## Contributing, security, license

- [CONTRIBUTING.md](CONTRIBUTING.md) for setup and standards. This is a personal learning project, so outside contributions are welcome but not expected.
- [SECURITY.md](SECURITY.md) for reporting a vulnerability. Please do not open a public issue for one.
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for community expectations.
- Licensed under the [MIT License](LICENSE).
