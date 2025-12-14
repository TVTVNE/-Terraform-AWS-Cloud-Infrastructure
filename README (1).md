# ☁️ Terraform AWS Cloud Infrastructure

![Terraform](https://img.shields.io/badge/terraform-%235835CC.svg?style=for-the-badge&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-%23FF9900.svg?style=for-the-badge&logo=amazon-aws&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)

> Enterprise-grade AWS infrastructure as code with Terraform, featuring multi-account setup, CI/CD automation, and comprehensive security controls.

[![Terraform](https://img.shields.io/badge/terraform-v1.6+-623CE4?logo=terraform)](https://www.terraform.io/)
[![AWS Provider](https://img.shields.io/badge/aws--provider-v5.0+-FF9900?logo=amazon-aws)](https://registry.terraform.io/providers/hashicorp/aws/latest)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Modules](#modules)
- [Environments](#environments)
- [Security](#security)
- [Cost Optimization](#cost-optimization)
- [CI/CD](#cicd)

## 🎯 Overview

This repository contains production-ready Terraform configurations for deploying a complete AWS infrastructure including VPC, EKS, RDS, S3, CloudFront, and more. Built with best practices, modularity, and security in mind.

**What's Included:**
- ✅ Multi-account AWS Organization setup
- ✅ Networking (VPC, Subnets, NAT, VPN)
- ✅ Compute (EKS, EC2 Auto Scaling, Lambda)
- ✅ Database (RDS Multi-AZ, DynamoDB, ElastiCache)
- ✅ Storage (S3, EFS, Backup)
- ✅ CDN & DNS (CloudFront, Route53)
- ✅ Security (IAM, KMS, Secrets Manager, WAF)
- ✅ Monitoring (CloudWatch, X-Ray)

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Organization                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Management   │  │     Dev      │  │  Production  │      │
│  │   Account    │  │   Account    │  │   Account    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    │                   │
            ┌───────▼────────┐  ┌──────▼───────┐
            │   VPC (Dev)    │  │ VPC (Prod)   │
            │                │  │              │
            │ ┌────────────┐ │  │ ┌──────────┐ │
            │ │    EKS     │ │  │ │   EKS    │ │
            │ │  Cluster   │ │  │ │ Cluster  │ │
            │ └────────────┘ │  │ └──────────┘ │
            │                │  │              │
            │ ┌────────────┐ │  │ ┌──────────┐ │
            │ │    RDS     │ │  │ │   RDS    │ │
            │ │  (Postgres)│ │  │ │(Multi-AZ)│ │
            │ └────────────┘ │  │ └──────────┘ │
            └────────────────┘  └──────────────┘
```

## ✨ Features

### Infrastructure Components

#### Networking
- **VPC**: Custom VPC with public/private subnets across 3 AZs
- **NAT Gateway**: High availability NAT configuration
- **VPN**: Site-to-Site VPN for hybrid connectivity
- **Transit Gateway**: Multi-VPC connectivity
- **PrivateLink**: Secure service access

#### Compute
- **EKS**: Managed Kubernetes cluster with node groups
- **EC2**: Auto-scaling groups with mixed instances
- **Lambda**: Serverless functions with VPC integration
- **ECS**: Container orchestration alternative

#### Database & Storage
- **RDS**: PostgreSQL/MySQL with automated backups
- **DynamoDB**: NoSQL with global tables
- **ElastiCache**: Redis/Memcached clusters
- **S3**: Versioned buckets with lifecycle policies
- **EFS**: Shared file system for EKS

#### Security & Compliance
- **IAM**: Least-privilege roles and policies
- **KMS**: Customer-managed encryption keys
- **Secrets Manager**: Automated secret rotation
- **WAF**: Web application firewall
- **GuardDuty**: Threat detection
- **Security Hub**: Compliance monitoring

#### Monitoring & Logging
- **CloudWatch**: Metrics, logs, and alarms
- **X-Ray**: Distributed tracing
- **CloudTrail**: API audit logging
- **VPC Flow Logs**: Network traffic analysis

## 🔧 Prerequisites

**Required Tools:**
- Terraform >= 1.6.0
- AWS CLI >= 2.13.0
- tfenv (recommended for version management)
- pre-commit (for git hooks)

**AWS Requirements:**
- AWS Account with admin access
- Configured AWS credentials
- S3 bucket for Terraform state
- DynamoDB table for state locking

## 🚀 Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/terraform-aws-infrastructure.git
cd terraform-aws-infrastructure
```

### 2. Configure AWS credentials

```bash
# Using AWS CLI
aws configure

# Or using environment variables
export AWS_ACCESS_KEY_ID="your-access-key"
export AWS_SECRET_ACCESS_KEY="your-secret-key"
export AWS_DEFAULT_REGION="eu-west-1"
```

### 3. Initialize Terraform backend

```bash
# Create S3 bucket for state
aws s3 mb s3://your-terraform-state-bucket --region eu-west-1

# Create DynamoDB table for locking
aws dynamodb create-table \
    --table-name terraform-state-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region eu-west-1
```

### 4. Configure backend

Edit `environments/dev/backend.tf`:

```hcl
terraform {
  backend "s3" {
    bucket         = "your-terraform-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
  }
}
```

### 5. Deploy infrastructure

```bash
# Navigate to environment
cd environments/dev

# Initialize Terraform
terraform init

# Review plan
terraform plan -out=tfplan

# Apply changes
terraform apply tfplan
```

## 📁 Project Structure

```
terraform-aws-infrastructure/
├── modules/                    # Reusable Terraform modules
│   ├── networking/
│   │   ├── vpc/
│   │   ├── subnets/
│   │   └── security-groups/
│   ├── compute/
│   │   ├── eks/
│   │   ├── ec2/
│   │   └── lambda/
│   ├── database/
│   │   ├── rds/
│   │   ├── dynamodb/
│   │   └── elasticache/
│   ├── storage/
│   │   ├── s3/
│   │   └── efs/
│   ├── security/
│   │   ├── iam/
│   │   ├── kms/
│   │   └── secrets-manager/
│   └── monitoring/
│       ├── cloudwatch/
│       └── xray/
├── environments/              # Environment configurations
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── production/
├── policies/                 # IAM and SCP policies
│   ├── iam/
│   └── scp/
├── scripts/                  # Utility scripts
│   ├── bootstrap.sh
│   ├── validate.sh
│   └── destroy.sh
├── docs/                     # Documentation
│   ├── architecture/
│   ├── runbooks/
│   └── cost-analysis/
├── .github/
│   └── workflows/
│       ├── terraform-plan.yml
│       └── terraform-apply.yml
├── .pre-commit-config.yaml
├── .tflint.hcl
└── README.md
```

## 🧩 Modules

### VPC Module

Creates a complete VPC with subnets, route tables, and NAT gateways.

```hcl
module "vpc" {
  source = "../../modules/networking/vpc"
  
  vpc_name             = "production-vpc"
  vpc_cidr             = "10.0.0.0/16"
  availability_zones   = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
  public_subnets       = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets      = ["10.0.11.0/24", "10.0.12.0/24", "10.0.13.0/24"]
  database_subnets     = ["10.0.21.0/24", "10.0.22.0/24", "10.0.23.0/24"]
  
  enable_nat_gateway   = true
  enable_vpn_gateway   = false
  enable_flow_logs     = true
  
  tags = local.common_tags
}
```

### EKS Module

Deploys a production-ready EKS cluster.

```hcl
module "eks" {
  source = "../../modules/compute/eks"
  
  cluster_name    = "production-eks"
  cluster_version = "1.28"
  
  vpc_id          = module.vpc.vpc_id
  subnet_ids      = module.vpc.private_subnets
  
  node_groups = {
    general = {
      desired_size = 3
      min_size     = 2
      max_size     = 10
      instance_types = ["t3.large"]
    }
    compute = {
      desired_size = 2
      min_size     = 1
      max_size     = 5
      instance_types = ["c5.2xlarge"]
    }
  }
  
  tags = local.common_tags
}
```

### RDS Module

Multi-AZ PostgreSQL database with automated backups.

```hcl
module "rds" {
  source = "../../modules/database/rds"
  
  identifier     = "production-postgres"
  engine         = "postgres"
  engine_version = "15.4"
  instance_class = "db.r6g.xlarge"
  
  allocated_storage     = 100
  max_allocated_storage = 1000
  
  multi_az               = true
  backup_retention_period = 30
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.database_subnets
  
  tags = local.common_tags
}
```

## 🌍 Environments

### Development
- **Purpose**: Testing and development
- **Resources**: Minimal sizing, spot instances
- **State**: `s3://terraform-state/dev/`
- **Auto-destroy**: Nightly shutdown

### Staging
- **Purpose**: Pre-production validation
- **Resources**: Medium sizing, similar to prod
- **State**: `s3://terraform-state/staging/`
- **Auto-destroy**: Weekend shutdown

### Production
- **Purpose**: Live customer-facing environment
- **Resources**: High availability, multi-AZ
- **State**: `s3://terraform-state/production/`
- **Auto-destroy**: Never

## 🔐 Security

### Best Practices Implemented

1. **Encryption**
   - S3 buckets encrypted with KMS
   - RDS encrypted at rest and in transit
   - EBS volumes encrypted
   - Secrets Manager for sensitive data

2. **Network Security**
   - Private subnets for workloads
   - Security groups with minimal access
   - NACLs for additional protection
   - VPC Flow Logs enabled

3. **IAM**
   - Least-privilege policies
   - MFA enforcement
   - Role-based access control
   - No long-term credentials

4. **Compliance**
   - AWS Config rules
   - Security Hub standards
   - GuardDuty threat detection
   - CloudTrail audit logging

### Security Scanning

```bash
# Run tfsec
tfsec .

# Run checkov
checkov -d .

# Run terraform validate
terraform validate

# Run terraform fmt
terraform fmt -recursive
```

## 💰 Cost Optimization

### Implemented Strategies

- **Right-sizing**: Instance Scheduler for dev/staging
- **Reserved Instances**: 1-year RIs for production RDS
- **Spot Instances**: For non-critical EKS workloads
- **S3 Lifecycle**: Automated transition to Glacier
- **Auto-scaling**: HPA and cluster autoscaling
- **Resource Tagging**: Cost allocation tags

### Monthly Cost Estimate

| Environment | Estimated Cost |
|------------|----------------|
| Development | $150-250 |
| Staging | $300-500 |
| Production | $1,200-2,000 |

## 🔄 CI/CD

### GitHub Actions Workflow

Automated Terraform workflows for plan and apply:

```yaml
# .github/workflows/terraform-plan.yml
name: Terraform Plan

on:
  pull_request:
    branches: [ main ]

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/setup-terraform@v2
      - name: Terraform Init
        run: terraform init
      - name: Terraform Plan
        run: terraform plan
```

### Deployment Process

1. **PR Created** → Terraform plan runs automatically
2. **PR Approved** → Manual approval required
3. **Merge to Main** → Auto-deploy to dev
4. **Tag Release** → Deploy to staging
5. **Production** → Manual approval + apply

## 🧪 Testing

```bash
# Validate configurations
terraform validate

# Format code
terraform fmt -recursive

# Security scanning
tfsec .
checkov -d .

# Cost estimation
infracost breakdown --path .
```

## 📊 Monitoring

### CloudWatch Dashboards

Access pre-configured dashboards:
- Infrastructure Overview
- EKS Cluster Metrics
- RDS Performance
- Cost and Usage

### Alerts Configured

- High CPU/Memory usage
- Failed deployments
- Database connection errors
- Cost anomalies

## 🛠️ Maintenance

### Regular Tasks

```bash
# Update Terraform version
tfenv install latest
tfenv use latest

# Update provider versions
terraform init -upgrade

# Clean up old state
terraform state list
terraform state rm <resource>
```

## 📚 Documentation

- [Module Documentation](./docs/modules/)
- [Architecture Decisions](./docs/architecture/)
- [Runbooks](./docs/runbooks/)
- [Cost Analysis](./docs/cost-analysis/)

## 🤝 Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

## 📝 License

This project is licensed under the MIT License - see [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- HashiCorp Terraform team
- AWS Solutions Architects
- Open-source community

## 📧 Contact

**Elias** - DevOps Engineer
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your Profile](https://linkedin.com/in/yourprofile)
- Email: elias@example.com

---

⭐ **Star this repo if you find it useful!** ⭐

**Need custom AWS infrastructure? I'm available for freelance projects!**
