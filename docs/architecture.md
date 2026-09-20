# Architecture

The target design. Nothing here is built yet; the status of each part is tracked in the [README](../README.md#capabilities) and [ROADMAP](../ROADMAP.md). Reasons for the choices are in [design-decisions.md](design-decisions.md).

## Classification

| Dimension      | Value                                                     |
| -------------- | --------------------------------------------------------- |
| Runtime        | Distributed (three managed Kubernetes clusters plus kind) |
| Exposure       | Public HTTP ingress in demonstration environments         |
| State          | Persistent (Postgres, Redis)                              |
| Interface      | API and batch (workers)                                   |
| Criticality    | Experimental (a learning and demonstration platform)      |
| Data           | Non-sensitive, synthetic                                  |
| Deployment     | Continuous through GitOps                                 |
| Infrastructure | Cloud, ephemeral                                          |

## Boundary with the workload

Kubeforge deploys the [Nexusdeploy-App](https://github.com/harshit-kandhwey/Nexusdeploy-App) images. The workload has three roles built from one codebase:

- **API.** Gunicorn serving the REST API, with `/health` (database and Redis), `/ready` and `/metrics`.
- **Worker.** A Celery consumer. It scales on queue depth.
- **Beat.** The Celery scheduler. It must run as a single instance.

Postgres and Redis back all three. Database migrations run as a one-shot task, not on every pod start. The workload owns the exact entrypoints and environment variables; check that repository before relying on any detail listed here.

## Target layers

1. **Cluster.** Provisioned by Terraform: network, control plane, node groups, and cluster-level identity (OIDC). One module per cloud.
2. **Platform.** Installed by GitOps: ingress controller, certificate and DNS automation, External Secrets, policy engine, autoscalers, and the observability stack.
3. **Workload.** The Helm chart for the API, workers and beat, with per-cloud values files.
4. **Delivery.** GitHub Actions builds, scans, signs and pushes images using short-lived OIDC credentials. Argo CD applies what is in Git.

## Environments

- **kind (local).** Everything provable without a cloud account. The default place to develop.
- **EKS.** The first cloud target, ephemeral.
- **GKE and AKS.** Added in phase 6 as thin modules over the same chart.

## Data flow for a change

A change is merged to `main`, CI builds and signs the image and pushes it to the registry, the chart or values in Git are updated, and Argo CD in each cluster reconciles to match. A failed rollout analysis rolls the release back without manual action.
