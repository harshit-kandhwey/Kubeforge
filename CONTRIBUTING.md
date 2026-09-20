# Contributing to Kubeforge

Kubeforge is a personal, hands-on learning project, so contributions are welcome but not expected. This document covers setup, the workflow, and the standards the repository holds itself to.

## Project overview

Kubeforge is the platform half of a two-repo project: it runs the [Nexusdeploy-App](https://github.com/harshit-kandhwey/Nexusdeploy-App) workload on Kubernetes the way a production team would. See [README.md](README.md) for what exists today and [ROADMAP.md](ROADMAP.md) for what is planned.

## Setup

```bash
pip install pre-commit
pre-commit install            # runs the hooks on every commit
pre-commit run --all-files    # first-time check of the whole repository
```

Prettier formats the Markdown, YAML and JSON in this repository through the pre-commit hook, so no separate Node install is needed. To format by hand:

```bash
npx prettier --write "**/*.md"
```

Docs use `"proseWrap": "never"` (see `.prettierrc.json`): one line per paragraph, no hard wrapping.

Tooling for later phases (`kind`, `kubectl`, `helm`, Terraform, the AWS CLI) is documented in the README as each phase lands.

## Before opening a PR

```bash
pre-commit run --all-files
```

CI runs the same hooks, plus a full-history secret scan (the pre-commit gitleaks hook only sees staged changes), so a clean local run predicts a green CI.

## Branch and PR workflow

| Branch       | Purpose                                                   |
| ------------ | --------------------------------------------------------- |
| `main`       | Always green. Reflects only what has been built and shown |
| `feature/**` | Your work. A PR into `main` runs CI                       |

1. Create a branch from `main`: `git checkout -b feature/my-change`.
2. Make the change, with docs and tests where they apply.
3. Run the checks above.
4. Open a PR into `main`. CI must pass before merging.

Prefer small, reviewable changes. Use a merge commit for anything with real intermediate structure, and reserve squash for single-purpose automated changes such as dependency bumps.

## Standards

- **Infrastructure only through code.** Anything Terraform or GitOps manages is changed only through that code. A manual console or CLI fix is reverted by the next apply.
- **Modules and environments stay separate.** Reusable modules take variables and hold no environment-specific values. Environments only instantiate modules.
- **No secrets, ever.** No credentials, kubeconfigs, Terraform state or `.tfvars` in Git. Secret scanning runs in pre-commit and CI, and it is a backstop, not a substitute for care.
- **Least privilege by default** for every role, identity and permission grant.
- **A new guard is not trusted until it has failed.** For every test, policy or CI gate, plant the bug it should catch, confirm it fails with a specific, named failure, remove the plant, and confirm it passes.
- **State what changed and why.** Comments explain the non-obvious why. Longer rationale goes into [docs/design-decisions.md](docs/design-decisions.md) and is referenced by anchor.
- **Claim only what is built.** The README, roadmap and resume change when a phase is demonstrated, never before.
- **Any step that creates cloud resources states its estimated cost first.**

### Commit messages

Conventional commits:

```
<type>: <short summary>

<optional body explaining why, not what>
```

Types: `feat`, `fix`, `perf`, `refactor`, `test`, `chore`, `docs`, `ci`.

## Security

If you discover a vulnerability, do not open a public issue. See [SECURITY.md](SECURITY.md).
