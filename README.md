# Vprofile EKS Infrastructure

This repository contains Terraform configuration to provision a complete Amazon EKS (Elastic Kubernetes Service) cluster with all necessary components for production-grade Kubernetes workloads.

## Architecture Overview

- **VPC**: Custom VPC with public subnets across multiple availability zones
- **EKS Cluster**: Managed Kubernetes control plane
- **Node Group**: Auto-scaling EC2 worker nodes
- **IRSA**: IAM Roles for Service Accounts with OIDC provider
- **EBS CSI Driver**: AWS addon for EBS volume management

## Features

- ✅ Multi-AZ public subnets with internet gateway
- ✅ EKS cluster with managed node group
- ✅ IRSA (IAM Roles for Service Accounts) support
- ✅ EBS CSI Driver addon for persistent volume management
- ✅ Terraform remote state in S3
- ✅ Modular and scalable design

## Prerequisites

- AWS CLI configured with appropriate credentials
- Terraform >= 1.0
- kubectl installed locally
- AWS account with necessary permissions

## File Structure

```
.
├── backend.tf           # S3 remote state configuration
├── main.tf              # Primary infrastructure resources
├── variables.tf         # Variable definitions and defaults
├── outputs.tf           # Output values
├── .gitignore          # Git ignore patterns
└── README.md           # This file
```

## Configuration

### Variables

Key variables can be customized in `variables.tf`:

- `region`: AWS region (default: us-east-1)
- `cluster_name`: EKS cluster name (default: vprofile-eks-cluster)
- `vpc_cidr`: VPC CIDR block (default: 10.0.0.0/16)
- `public_subnets`: Public subnet CIDRs (default: 10.0.1.0/24, 10.0.2.0/24)
- `availability_zones`: AZ list (default: us-east-1a, us-east-1b)
- `instance_type`: EC2 node type (default: t3.large)
- `min_size`: Minimum node count (default: 1)
- `max_size`: Maximum node count (default: 2)
- `desired_size`: Desired node count

## Usage

### Initialize Terraform

```bash
terraform init
```

### Plan Infrastructure

```bash
terraform plan
```

### Apply Configuration

```bash
terraform apply
```

### Destroy Infrastructure

```bash
terraform destroy
```

## Accessing the Cluster

After applying Terraform, configure kubectl:

```bash
aws eks update-kubeconfig --region $(terraform output -raw region) --name $(terraform output -raw cluster_name)
```

Verify cluster access:

```bash
kubectl get nodes
kubectl get pods -A
```

## EBS CSI Driver

The EBS CSI driver addon is automatically configured with:
- IRSA setup for secure credential handling
- OIDC provider association
- Appropriate IAM permissions

No manual Kubernetes service account creation is needed - AWS addon handles this automatically.

## Remote State

Terraform state is stored in S3 bucket: `gitops-terraformcode831`

Ensure this bucket exists and you have appropriate S3 permissions.

## Outputs

After successful apply, the following outputs are available:

- `cluster_endpoint`: EKS API endpoint
- `cluster_name`: EKS cluster name
- `cluster_arn`: EKS cluster ARN

View outputs:

```bash
terraform output
```

## Troubleshooting

### State File Issues

If encountering state-related issues, ensure S3 bucket and credentials are configured properly.

### OIDC Provider Errors

The OIDC provider is automatically created from the EKS cluster's identity endpoint. If TLS certificate validation fails, ensure the cluster is fully provisioned before applying.

### Node Group Not Scaling

Check Auto Scaling Group (ASG) configuration and CloudWatch logs for node group events.

## Security Considerations

- Public subnets are used for demo purposes; consider private subnets with NAT gateways for production
- Restrict API endpoint access using security groups
- Enable logging for EKS cluster control plane
- Use IAM roles and IRSA for pod authentication

## Contributing

When modifying infrastructure:
1. Run `terraform fmt` to format code
2. Run `terraform validate` to check syntax
3. Create a plan and review changes before applying
4. Document any new variables or resources

## License

This project is provided as-is for infrastructure provisioning.
