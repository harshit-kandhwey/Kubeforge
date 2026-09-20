# Security Policy

## Supported versions

Kubeforge has no tagged release yet. Only the current state of `main` is supported.

| Version       | Supported |
| ------------- | --------- |
| `main`        | Yes       |
| Older commits | No        |

## Reporting a vulnerability

**Please do not report security vulnerabilities through public GitHub issues, pull requests, or discussions.**

Report privately via one of these channels:

- **GitHub Private Vulnerability Reporting**: use the "Report a vulnerability" button in the [Security tab](../../security/advisories/new) of this repository.
- **Email**: send details to **harshitkandhwey@gmail.com** with the subject line `[SECURITY] Kubeforge: <brief description>`.

### What to include

- A description of the vulnerability and its potential impact.
- The affected component (Terraform module, Helm chart, CI workflow, and so on).
- Steps to reproduce or a proof of concept.
- Any mitigation you have identified.

### Response timeline

This is a personal project maintained by one person, so these are targets rather than guarantees.

| Stage              | Target                                          |
| ------------------ | ----------------------------------------------- |
| Acknowledgement    | Within 72 hours                                 |
| Initial assessment | Within 7 days                                   |
| Fix or mitigation  | Depends on severity                             |
| Disclosure         | Coordinated with the reporter after a fix lands |

## Security controls in place

- **Secret scanning.** gitleaks scans staged changes in pre-commit and the full history in CI.
- **Dependency updates.** Dependabot tracks GitHub Actions.
- **Least-privilege CI.** Workflows default to read-only `contents` permission.
- **Ignored by design.** Kubeconfigs, Terraform state, `.tfvars`, keys and `.env` files are ignored by Git.

Controls for the platform itself (network policies, identity, secrets handling, policy engine) are planned and tracked in [ROADMAP.md](ROADMAP.md). They are listed here only when they exist and have been verified.

## Design notes

- Cloud clusters are ephemeral and destroyed after each session, which limits the window in which any cluster credential is valid.
- Workloads are intended to use platform-native short-lived identity rather than static cloud credentials, both in the cluster and in CI.
