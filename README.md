# Crossplane Demo

This repository acts as a demo for GitOps integration between Crossplane, Kyverno in a AWS EKS Service, using Actions
for CI/CD pipelines.

Some resources (e.g. EKS cluster and IAM roles) are managed manually, not provisioned via Crossplane or Terraform (to
minimize scope).

Integration runs with every merged pull request in the default branch, and it can be configured for different
environments by creating specific branches.

## Goals

- Test Crossplane's core functionality (provisioning AWS resources) in cloud.
- Integrate Kyverno for policy enforcement on Crossplane-managed resources.
- Automate all operations using GitHub Actions.

---

## Troubleshooting

- Refer to [this page](./docs/troubleshooting.md) for common issues and solutions.
