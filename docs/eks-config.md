# EKS Cluster Setup

This document details the resources manually created in AWS through the Management Console for the EKS cluster.

These resources support testing Crossplane and Kyverno with GitHub Actions in a cloud-based setup.

## IAM User

- **Purpose**: Provides AWS credentials for GitHub Actions to authenticate with AWS and manage resources via Crossplane.
- **Creation Steps**:
    1. Navigate to IAM > Users > Add user.
    2. Set user name: `crossplane-aws-demo`.
    3. Select "Programmatic access" (no console access).
    4. Attach policies:
        - `AmazonS3FullAccess`: Grants full access to S3 for testing Crossplane resource provisioning.
        - `AmazonEKSClusterPolicy`: Allows GitHub Actions to access the EKS cluster.
        - `IAMReadOnlyAccess`: Provides read-only access to IAM for debugging.
    5. Create user and download the CSV file with access keys.
- **Access Keys**:
    - Stored in GitHub secrets:
        - `AWS_ACCESS_KEY_ID`: `<your-access-key-id>`
        - `AWS_SECRET_ACCESS_KEY`: `<your-secret-access-key>`
- **Security Notes**:
    - Rotate access keys periodically (IAM > Users > `crossplane-aws-demo` > Security credentials).
    - Monitor CloudTrail logs for suspicious activity.

## IAM Role

- **Purpose**: Allows the EKS cluster to manage AWS resources on your behalf.
- **Creation Steps**:
    1. Navigate to IAM > Roles > Create role.
    2. Select "AWS service" > "EKS" > "EKS - Cluster".
    3. Attach policies:
        - `AmazonEKSClusterPolicy`: Grants permissions for EKS cluster operations.
    4. Set role name: `eks-cluster-role`.
    5. Create role.
- **ARN**: `arn:aws:iam::<your-account-id>:role/eks-cluster-role`
- **Notes**:
    - Used during EKS cluster creation (see below).

## IAM Role

- **Purpose**: Allows EKS worker nodes to interact with AWS services.
- **Creation Steps**:
    1. Navigate to IAM > Roles > Create role.
    2. Select "AWS service" > "EKS" > "EKS - Pod Identity".
    3. Attach policies:
        - `AmazonEC2ContainerRegistryReadOnly`: Allows pulling container images from ECR.
        - `AmazonEKS_CNI_Policy`: Supports VPC CNI networking (used if VPC CNI role is unset).
        - `AmazonEKSWorkerNodePolicy`: Grants permissions for worker nodes.
    4. Set role name: `pod-identity-cni-role`.
    5. Create role.
- **ARN**: `arn:aws:iam::<your-account-id>:role/pod-identity-cni-role`
- **Notes**:
    - Associated with the Amazon VPC CNI add-on (see below).

## IAM Role

- **Purpose**: Allows EKS worker nodes to interact with AWS services.
- **Creation Steps**:
    1. Navigate to IAM > Roles > Create role.
    2. Select "AWS service" > "EC2" > "EC2".
    3. Attach policies:
        - `AmazonEC2ContainerRegistryReadOnly`: Allows pulling container images from ECR.
        - `AmazonEKS_CNI_Policy`: Supports VPC CNI networking (used if VPC CNI role is unset).
        - `AmazonEKSWorkerNodePolicy`: Grants permissions for worker nodes.
    4. Set role name: `eks-node-role`.
    5. Create role.
- **ARN**: `arn:aws:iam::<your-account-id>:role/eks-node-role`
- **Notes**:
    - Associated with the EKS node group (see below).

## EKS Cluster

- **Purpose**: Runs Crossplane and Kyverno for testing with GitHub Actions.
- **Creation Steps**:
    1. Navigate to EKS > Clusters > Create cluster.
    2. Set cluster name: `crossplane-test`.
    3. Set custom configuration, do not use EKS Auto Mode
    4. Select Kubernetes version: `1.32` (latest stable at time of writing), standard upgrade policy.
    5. Choose cluster service role: `eks-cluster-role`.
    6. Specify networking:
        - VPC: default (or your VPC of choice).
        - Subnets: Select at least two subnets.
        - Security groups: Use default (auto-created by EKS).
        - Cluster endpoint access: "Public and private" (for GitHub Actions access).
    7. Configure logging: Disabled (optional, enable CloudWatch logging if needed).
    8. Review and create cluster.
- **Region**: `<your-region>`
- **Node Group**:
    1. Go to EKS > Clusters > `crossplane-test` > Compute > Add node group.
    2. Set node group name: `crossplane-test-nodes`.
    3. Select node IAM role: `eks-node-role`.
    4. Choose instance type: `t3.medium`.
    5. Set desired size: 2 nodes.
    6. Select subnets: Same as cluster.
    7. Create node group.
- **Add-ons**:
    - Enabled during or after cluster creation:
        - **Amazon VPC CNI**:
            - Version: Latest.
            - IAM Role: `pod-identity-cni-role`.
        - **CoreDNS**:
            - Version: Latest.
        - **Kube-Proxy**:
            - Version: Matches cluster, latest.
    - Disabled: All other add-ons to reduce complexity and costs.
- **Notes**:
    - Cluster endpoint is public for simplicity; restrict to specific CIDR ranges in production.
    - Stored in GitHub secrets:
        - `EKS_CLUSTER_NAME`: `crossplane-test`
        - `AWS_REGION`: `<your-region>`

---

## Security Best Practices

- **IAM**
    - Rotate access keys for `crossplane-aws-demo` regularly.
    - Use IRSA for production setups instead of static credentials.
- **EKS**
    - Restrict cluster endpoint access in production (e.g., private endpoint or CIDR whitelist).
    - Enable CloudWatch logging for API server, audit, and authenticator logs if monitoring is needed.
- **VPC**
    - Ensure subnets and security groups are appropriately configured for your workload.
- **Monitoring**
    - Enable CloudTrail to log API calls for all resources.
    - Review logs periodically for suspicious activity.

---

## Troubleshooting

- **IAM Issues**
    - Verify if policies are attached correctly in IAM > Users/Roles.
    - Check if GitHub Secrets match the downloaded access keys.
- **EKS Cluster Issues**
    - Ensure cluster status is "Active" in EKS console.
    - Verify node group is "Active" and nodes are running (`kubectl get nodes` in GitHub Actions logs).
- **Add-On Issues**
    - Check add-on status in EKS > Clusters > `crossplane-test` > Add-ons.
    - Ensure `pod-identity-cni-role` is associated with VPC CNI (EKS > Add-ons > Edit).

---

## Notes

- These resources are managed manually, not provisioned by Crossplane or Terraform.
- Refer to [this page](./docs/troubleshooting.md) for common issues and solutions.
