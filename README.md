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

## Structure

- **`.crossplane/compositions/`**: Defines Crossplane CompositeResourceDefinitions (XRDs) and Compositions for custom
  resources.
- **`.crossplane/managed-resources/`**: Contains instances of composite resources (e.g., S3 buckets).
- **`.crossplane/policies/`**: Holds Kyverno policies for resource validation.
- **`.crossplane/providers/`**: Configures the AWS provider for Crossplane.

---

## Setup Instructions

### Prerequisites

- AWS account with EKS cluster manually provisioned (refer to [this page](./docs/eks-config.md) for more info).
- GitHub Secrets configured:
    - `AWS_ACCESS_KEY_ID`: AWS Access Key ID
    - `AWS_SECRET_ACCESS_KEY`: AWS Secret Access Key
- GitHub Variables configured:
    - `AWS_EKS_CLUSTER_NAME`: Name of the EKS cluster
    - `AWS_REGION`: AWS region

## Usage

- Push changes to the `main` branch to trigger the `main.yml` workflow.
- Check GitHub Actions logs for execution details.

### 1. Setup EKS Access

- GitHub Actions workflow: [aws-eks-access.yml](./.github/workflows/aws-eks-access.yml)
- Configures AWS credentials and EKS access.

### 2. Install Crossplane

- GitHub Actions workflow: [crossplane-install.yml](./.github/workflows/crossplane-install.yml)
- Installs Crossplane on the EKS cluster.

### 3. Configure the AWS Provider

- GitHub Actions workflow: [aws-provider-config.yml](./.github/workflows/aws-provider-config.yml)
- Sets up the AWS provider for Crossplane.

### 4. Apply Compositions

- GitHub Actions workflow: [composition-apply.yml](./.github/workflows/composition-apply.yml)
- Applies Crossplane compositions for S3 buckets.

### 5. Install Kyverno

- GitHub Actions workflow: [kyverno-install.yml](./.github/workflows/kyverno-install.yml)
- Installs Kyverno on the EKS cluster.

### 6. Test Crossplane

- GitHub Actions workflow: [crossplane-test.yml](./.github/workflows/crossplane-test.yml)
- Tests S3 bucket provisioning via compositions.

### 7. Test Kyverno

- GitHub Actions workflow: [kyverno-test.yml](./.github/workflows/kyverno-test.yml)
- Tests Kyverno policy enforcement on Crossplane-managed resources.

### 8. Manually Managed Resources

- EKS cluster is manually provisioned (refer to [this page](./docs/eks-config.md) for more info).
- IAM roles and permissions are managed directly in AWS.

---

## Troubleshooting

- Refer to [this page](./docs/troubleshooting.md) for common issues and solutions.
