# Design decisions

The permanent record of why each non-obvious choice was made. Code and config reference these by anchor (for example `docs/design-decisions.md#separate-repos`) instead of repeating the reasoning inline. Each entry has a status: **Decided** (in effect) or **Proposed** (the intended direction, not yet built or confirmed).

## Separate repos

**Status:** Decided.

**Decision.** The workload lives in [Nexusdeploy-App](https://github.com/harshit-kandhwey/Nexusdeploy-App) and the platform lives here. Kubeforge consumes the workload only as container images and never changes its code.

**Why.** The same workload is also deployed to ECS by Ephemeral-Deploy. Keeping it cloud-neutral and separate means it stays reusable, and Kubeforge stays focused on the platform.

**Consequences.** The contract between the two (image entrypoints, health endpoints, environment variables, one-shot tasks) is owned by Nexusdeploy-App. A change to it must land in every consumer together.

**Revisit when.** The workload needs Kubernetes-specific code, which would mean the boundary is wrong.

## Multi-cluster, not cross-cloud

**Status:** Decided.

**Decision.** "Multi-cloud" means one chart and one GitOps definition deployed to three independent clusters (EKS, GKE, AKS), each managed from Argo CD. It does not mean failover between clouds, a multi-cluster mesh, or a control plane spanning providers.

**Why.** The first is achievable and demonstrates real portability. The second group is several projects on its own and would stall the repository half-built.

**Consequences.** The differences that remain are real work and are documented as found: workload identity, ingress and load balancer annotations, storage classes, and the secrets backend behind External Secrets.

**Revisit when.** Phases 1 to 6 are complete and cross-cluster behaviour is the next thing worth learning.

## EKS first

**Status:** Decided.

**Decision.** Build the full platform on EKS before touching GKE or AKS.

**Why.** AWS is the best-known cloud here, so the learning effort goes into Kubernetes rather than cloud basics. GKE and AKS then become thin modules over a design that already works.

**Revisit when.** Never for ordering; the per-cloud modules are phase 6.

## Local first, ephemeral in the cloud

**Status:** Decided.

**Decision.** Develop and prove behaviour on kind. Cloud clusters are created for a session and destroyed afterwards by Terraform.

**Why.** Managed control planes and nodes bill from the moment they exist. Local proof is free and faster, and ephemeral clusters keep cost and credential exposure bounded.

**Consequences.** Every cloud step states its estimated cost first. Anything that must persist between sessions (state, registry, DNS) is kept minimal and documented.

**Revisit when.** A long-running environment becomes necessary, for example to demonstrate a real upgrade.

## GitOps with Argo CD

**Status:** Proposed.

**Decision.** Argo CD, in an app-of-apps layout, is the only path by which a change reaches a cluster.

**Why.** It makes Git the single source of truth and gives drift detection and rollback for free. Argo CD is also widely used, which makes the skills transferable.

**Alternatives.** Flux is comparable. The choice will be confirmed, and the comparison written here, when phase 3 starts.

## Managed data services versus in-cluster

**Status:** Proposed.

**Decision.** To be decided in phase 2. The candidates are managed Postgres and Redis (RDS and ElastiCache on AWS, and equivalents elsewhere) versus in-cluster operators.

**Why it matters.** Managed services are the usual production choice and remove backup and upgrade burden, but they tie the design to each cloud and cost money while idle. In-cluster keeps parity across clouds and works on kind.

**Revisit when.** Phase 2 begins; the answer and its trade-offs are recorded here first.
