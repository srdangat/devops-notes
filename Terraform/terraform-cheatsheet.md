# The Complete Terraform Cheatsheet

---

## Table of Contents

1. [Core Concepts](#1-core-concepts)
2. [Essential CLI Commands](#2-essential-cli-commands)
3. [HCL File Structure](#3-hcl-file-structure)
4. [Providers & Version Constraints](#4-providers--version-constraints)
5. [Resources & Dependencies](#5-resources--dependencies)
6. [Variables](#6-variables)
7. [Outputs](#7-outputs)
8. [Data Sources](#8-data-sources)
9. [Locals](#9-locals)
10. [Built-in Functions](#10-built-in-functions)
11. [State Management](#11-state-management)
12. [Remote Backend (S3)](#12-remote-backend-s3)
13. [Modules](#13-modules)
14. [Workspaces](#14-workspaces)
15. [Lifecycle Rules](#15-lifecycle-rules)
16. [EKS with Terraform](#16-eks-with-terraform)
17. [Best Practices](#17-best-practices)
18. [Plan Symbol Reference](#18-plan-symbol-reference)

---

## 1. Core Concepts

### Infrastructure as Code (IaC)
- Define infrastructure (servers, networks, databases, IAM) **using code** instead of clicking around in a UI.
- Provides: **automation**, **consistency**, **version control**, and easy recreation.

### Terraform vs Other Tools

| Tool | Type | Scope |
|------|------|-------|
| **Terraform** | Declarative IaC | Multi-cloud (AWS, Azure, GCP…) |
| **CloudFormation** | Declarative IaC | AWS-only |
| **Ansible** | Procedural | Configuration management |
| **Pulumi** | Imperative IaC | Multi-cloud, uses real languages |

### Key Properties
- **Declarative** — You define the *desired state*; Terraform figures out the steps to reach it.
- **Cloud-agnostic** — Same language and workflow across providers.

---

## 2. Essential CLI Commands

### Core Workflow
```bash
terraform init        # Initialize project, download providers & modules
terraform validate    # Check config for syntax/logical errors
terraform fmt         # Auto-format .tf files
terraform plan        # Preview changes before applying
terraform apply       # Create or update infrastructure
terraform destroy     # Delete all managed resources
```

### State Commands
```bash
terraform show                              # Human-readable current state
terraform state list                        # List all managed resources
terraform state show <resource>             # Detailed view of one resource
terraform state mv <old> <new>              # Rename resource in state
terraform state rm <resource>               # Remove resource from state (without destroying)
terraform import <resource_type>.<name> <id>  # Import existing resource into state
terraform force-unlock <LOCK_ID>            # Unlock a stuck state file
terraform refresh                           # Sync state with real-world resources
```

### Output Commands
```bash
terraform output                            # Show all outputs
terraform output <name>                     # Show a specific output
terraform output -json                      # JSON format (useful for scripting)
```

### Workspace Commands
```bash
terraform workspace show                    # Show current workspace
terraform workspace list                    # List all workspaces
terraform workspace new <name>              # Create a new workspace
terraform workspace select <name>           # Switch to a workspace
terraform workspace delete <name>           # Delete a workspace
```

### Misc Commands
```bash
terraform graph                             # Output dependency graph (DOT format)
terraform graph | dot -Tpng > graph.png     # Render graph to PNG (requires Graphviz)
terraform console                           # Interactive expression evaluator
terraform init -upgrade                     # Upgrade provider/module versions
```

---

## 3. HCL File Structure

### Standard Project Layout
```
project/
├── main.tf          # Resource definitions
├── variables.tf     # Input variable declarations
├── outputs.tf       # Output value declarations
├── providers.tf     # Provider and backend config
├── locals.tf        # Local values
├── terraform.tfvars # Default variable values (auto-loaded)
└── .gitignore
```

### `.gitignore` for Terraform
```gitignore
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
.terraform.lock.hcl
```

---

## 4. Providers & Version Constraints

### Provider Block
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}
```

### Version Constraint Syntax

| Constraint | Meaning |
|-----------|---------|
| `~> 5.0`  | Allow `5.x` but NOT `6.0` (most common) |
| `>= 5.0`  | Allow `5.0` and any higher version |
| `= 5.0.0` | Exactly `5.0.0` only |
| `>= 5.0, < 6.0` | Range — same as `~> 5.0` |

### Lock File
`.terraform.lock.hcl` — locks the **exact** provider version and integrity hashes. Commit this to Git.

---

## 5. Resources & Dependencies

### Resource Syntax
```hcl
resource "<TYPE>" "<NAME>" {
  # arguments
}
```

### Example — VPC + Subnet (Implicit Dependency)
```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "my-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id      # <-- implicit dependency
  cidr_block = "10.0.1.0/24"
  map_public_ip_on_launch = true
}
```

### Explicit Dependency (`depends_on`)
Use when Terraform cannot automatically detect the dependency:
```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "my-logs"

  depends_on = [aws_instance.main]    # <-- explicit dependency
}
```

**Real-world `depends_on` examples:**
- RDS depends on VPC & Subnets
- EC2 depends on IAM Role
- ACM Certificate depends on CloudFront

### Implicit vs Explicit Dependencies

| Type | How | Example |
|------|-----|---------|
| **Implicit** | Terraform detects via resource attribute references | `vpc_id = aws_vpc.main.id` |
| **Explicit** | Manually declared with `depends_on` | `depends_on = [aws_instance.main]` |

---

## 6. Variables

### Variable Declaration (`variables.tf`)
```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"
  default     = "t2.micro"
}

# No default = user must provide it
variable "project_name" {
  type = string
}
```

### Variable Types

| Type | Example |
|------|---------|
| `string` | `"t2.micro"` |
| `number` | `2` |
| `bool` | `true` |
| `list(string)` | `["sg-123", "sg-456"]` |
| `map(string)` | `{ dev = "t2.micro", prod = "t3.small" }` |

### Variable Usage
```hcl
resource "aws_instance" "server" {
  instance_type = var.instance_type    # reference a variable
}
```

### Variable Files
```hcl
# terraform.tfvars (auto-loaded)
project_name  = "my-project"
environment   = "dev"

# prod.tfvars (manually specified)
environment   = "prod"
instance_type = "t3.small"
```

Apply with a specific var-file:
```bash
terraform plan -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

### Variable Precedence (Lowest → Highest)

| Priority | Source | Example |
|----------|--------|---------|
| 5 (Lowest) | Default value in `variables.tf` | `default = "dev"` |
| 4 | Environment variable | `TF_VAR_environment=uat` |
| 3 | Auto-loaded tfvars (`terraform.tfvars`) | `environment = "stage"` |
| 2 | `-var-file` flag | `terraform plan -var-file="prod.tfvars"` |
| 1 (Highest) | `-var` CLI flag | `terraform plan -var="instance_type=t2.nano"` |

### Environment Variable Override
```bash
export TF_VAR_environment="staging"
terraform plan
```

---

## 7. Outputs

### Output Declaration (`outputs.tf`)
```hcl
output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.server.public_ip
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Referencing Module Outputs
```hcl
output "web_server_ip" {
  value = module.web_server.public_ip
}
```

---

## 8. Data Sources

Read-only lookups of **existing** resources — Terraform does NOT manage them.

### Syntax
```hcl
data "<TYPE>" "<NAME>" {
  # filters
}
```

### Example — Dynamic AMI Lookup
```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Usage in resource:
resource "aws_instance" "server" {
  ami = data.aws_ami.amazon_linux.id
}
```

### Example — Availability Zones
```hcl
data "aws_availability_zones" "available" {}

# Use the first AZ:
availability_zone = data.aws_availability_zones.available.names[0]
```

### `resource` vs `data` Source

| Feature | `resource` | `data` |
|---------|-----------|--------|
| Creates infrastructure |  Yes |  No |
| Managed by Terraform |  Yes |  No |
| Stored in state |  Yes | Read-only reference |
| Lifecycle actions | create / update / delete | read-only |
| Use case | EC2, VPC, Subnet | AMI lookup, AZs, existing VPC |

---

## 9. Locals

Reusable internal values computed once per plan. Cannot be overridden by the user.

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# Usage:
resource "aws_vpc" "main" {
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}
```

### Workspace-Aware Locals
```hcl
locals {
  environment = terraform.workspace
  name_prefix = "${var.project_name}-${local.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
```

### `variable` vs `local` vs `output` vs `data`

| Keyword | Purpose |
|---------|---------|
| `variable` | Take input values from the user |
| `local` | Define internal reusable computed values |
| `data` | Fetch existing resources from the provider (read-only) |
| `output` | Display or export values after execution |

---

## 10. Built-in Functions

Practice in `terraform console`:
```bash
terraform console
```

### String Functions
```hcl
upper("terraweek")                           # "TERRAWEEK"
lower("TERRAWEEK")                           # "terraweek"
join("-", ["terra", "week", "2026"])         # "terra-week-2026"
format("arn:aws:s3:::%s", "my-bucket")      # "arn:aws:s3:::my-bucket"
replace("hello world", " ", "-")            # "hello-world"
```

### Collection Functions
```hcl
length([1, 2, 3])                                         # 3
lookup({dev = "t2.micro", prod = "t3.small"}, "dev")      # "t2.micro"
toset(["a", "b", "a"])                                    # {"a", "b"} (removes duplicates)
merge({a = 1}, {b = 2})                                   # {a = 1, b = 2}
flatten([[1, 2], [3, 4]])                                  # [1, 2, 3, 4]
```

### Networking Function
```hcl
cidrsubnet("10.0.0.0/16", 8, 1)    # "10.0.1.0/24"
cidrsubnet("10.0.0.0/16", 8, 2)    # "10.0.2.0/24"
```

### Conditional Expression
```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

---

## 11. State Management

### What the State File Stores
- Resource IDs and ARNs
- Current resource attributes (IP, tags, etc.)
- Mapping between Terraform config and real infrastructure
- Dependencies between resources

### State File Rules
- **Never manually edit** `terraform.tfstate` — it can corrupt state and cause mismatches.
- **Never commit** to Git — it contains sensitive data (IPs, passwords, ARNs).
- The `serial` number increments with every state change.

### State Commands

```bash
# Rename a resource in state (without recreating)
terraform state mv aws_s3_bucket.old_name aws_s3_bucket.new_name

# Remove a resource from Terraform management (doesn't destroy in AWS)
terraform state rm aws_s3_bucket.my_bucket

# Import an existing AWS resource into state
terraform import aws_s3_bucket.imported my-existing-bucket-name

# Unlock a stuck state after failed apply
terraform force-unlock <LOCK_ID>

# Sync state with real-world (detect drift)
terraform refresh
```

### When to Use Each State Command

| Command | When to Use |
|---------|-------------|
| `state mv` | Rename a resource in state or move it between modules |
| `state rm` | Stop managing a resource without destroying it in AWS |
| `import` | Bring an existing AWS resource under Terraform management |
| `force-unlock` | Unlock a stuck state file after a failed operation |
| `refresh` | Sync state to detect drift from manual changes |

### `terraform import` vs Creating from Scratch

| | `terraform import` | Create from Scratch |
|-|-------------------|---------------------|
| Resource exists in AWS? |  Yes (already exists) |  No (Terraform creates it) |
| State updated |  Yes |  Yes |
| Resource created by TF |  No |  Yes |
| Use case | Migrate manual resources into TF | Standard new resource workflow |

### State Drift
Occurs when someone changes infrastructure **outside Terraform** (console, CLI, another tool).

```bash
# Detect drift:
terraform plan    # Will show diff if reality ≠ desired state

# Fix drift — Option A: reconcile (apply TF config wins):
terraform apply

# Fix drift — Option B: accept the change (update .tf files to match reality)
```

**Prevention:** Restrict console access; route all changes through CI/CD pipelines.

---

## 12. Remote Backend (S3)

### Why Remote State?
- Local state is risky — one deleted file loses everything.
- Remote state enables **team collaboration** and **state locking**.

### Setup Backend Infrastructure
```bash
# Create S3 bucket
aws s3api create-bucket \
  --bucket my-tf-state \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# Enable versioning (allows state recovery)
aws s3api put-bucket-versioning \
  --bucket my-tf-state \
  --versioning-configuration Status=Enabled

# (Optional — legacy locking) Create DynamoDB table
aws dynamodb create-table \
  --table-name tf-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

### Backend Configuration (`providers.tf`)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    encrypt        = true

    # Option A: DynamoDB locking (traditional)
    dynamodb_table = "tf-state-lock"

    # Option B: Native S3 locking (Terraform >= 1.10, cost-effective)
    use_lockfile   = true
  }
}
```

### Migrate Local → Remote State
```bash
# After adding backend block, re-initialize:
terraform init
# Terraform asks: "Copy existing state to new backend?" → Answer: yes
```

### Workspace State Paths in S3
Each workspace stores its state at:
```
env:/<workspace_name>/<key>
# e.g. env:/dev/terraweek-capstone/terraform.tfstate
```

---

## 13. Modules

### What is a Terraform Module?
A module is simply a **directory with `.tf` files**. Any `.tf` directory is a module.

- **Root Module** — Where you run Terraform commands; the entry point.
- **Child Module** — Called by the root module to perform a specific task.

### Standard Module Layout
```
modules/
└── ec2-instance/
    ├── main.tf        # Resource definitions
    ├── variables.tf   # Module inputs
    └── outputs.tf     # Module outputs
```

### Writing a Module

**`modules/ec2-instance/variables.tf`**
```hcl
variable "ami_id"             { type = string }
variable "instance_type"      { type = string; default = "t2.micro" }
variable "subnet_id"          { type = string }
variable "security_group_ids" { type = list(string) }
variable "instance_name"      { type = string }
variable "tags"               { type = map(string); default = {} }
```

**`modules/ec2-instance/main.tf`**
```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids
  tags = merge(var.tags, { Name = var.instance_name })
}
```

**`modules/ec2-instance/outputs.tf`**
```hcl
output "instance_id" { value = aws_instance.this.id }
output "public_ip"   { value = aws_instance.this.public_ip }
output "private_ip"  { value = aws_instance.this.private_ip }
```

### Calling a Module (Root `main.tf`)
```hcl
module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "my-web-server"
  tags               = local.common_tags
}

# Reuse the same module for a second instance:
module "api_server" {
  source        = "./modules/ec2-instance"
  instance_name = "my-api-server"
  # ...
}
```

### Dynamic Block (Security Group Module)
```hcl
resource "aws_security_group" "sg" {
  name   = "${var.project_name}-sg"
  vpc_id = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Using a Terraform Registry Module
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway   = false
  enable_dns_hostnames = true

  tags = local.common_tags
}
```

Registry modules download to: `.terraform/modules/`

### Module Versioning
```hcl
version = "5.1.0"          # Exact version
version = "~> 5.0"         # Any 5.x
version = ">= 5.0, < 6.0"  # Range
```

```bash
terraform init -upgrade    # Check and upgrade to newer allowed versions
```

### Module State Prefixes
```bash
terraform state list
# module.vpc.aws_vpc.this
# module.web_server.aws_instance.this
# module.web_sg.aws_security_group.this
```

---

## 14. Workspaces

### What are Workspaces?
Workspaces let you manage **multiple environments** (dev, staging, prod) from a **single codebase** using separate state files.

- **`default`** — The workspace Terraform starts with (always exists).
- Each workspace stores state in its own path.

### Quick Reference
```bash
terraform workspace show                # Show current workspace
terraform workspace list                # List all workspaces
terraform workspace new dev             # Create 'dev' workspace
terraform workspace select dev          # Switch to 'dev'
terraform workspace delete dev          # Delete 'dev' (must not be active)
```

### Using `terraform.workspace` in Config
```hcl
locals {
  environment = terraform.workspace    # "dev", "staging", or "prod"
  name_prefix = "${var.project_name}-${local.environment}"
}
```

### Deploy Per Environment
```bash
terraform workspace select dev
terraform plan  -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"

terraform workspace select staging
terraform plan  -var-file="staging.tfvars"
terraform apply -var-file="staging.tfvars"

terraform workspace select prod
terraform plan  -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

### Environment Configuration Example

| Setting | `dev` | `staging` | `prod` |
|---------|-------|-----------|--------|
| `vpc_cidr` | `10.0.0.0/16` | `10.1.0.0/16` | `10.2.0.0/16` |
| `subnet_cidr` | `10.0.1.0/24` | `10.1.1.0/24` | `10.2.1.0/24` |
| `instance_type` | `t3.micro` | `t3.small` | `c7i-flex.large` |
| `ingress_ports` | `[22, 80]` | `[22, 80, 443]` | `[80, 443]` |
| SSH allowed |  Yes |  Yes |  No |
| HTTPS allowed |  No |  Yes |  Yes |

### Workspaces vs Separate Directories

| | Workspaces | Directories |
|-|-----------|-------------|
| Codebase | One copy | One copy per env |
| State | Separate per workspace | Separate per directory |
| Complexity | Lower | Higher for large setups |
| Best for | Simple multi-env | Complex per-env configs |

---

## 15. Lifecycle Rules

```hcl
resource "aws_instance" "server" {
  # ...

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags, user_data]
  }
}
```

### Lifecycle Arguments

| Argument | What It Does | Use Case |
|----------|-------------|----------|
| `create_before_destroy = true` | Create new resource first, then destroy old one | Zero-downtime RDS/EC2 updates |
| `prevent_destroy = true` | Block `terraform destroy` for this resource | Production databases, critical S3 buckets |
| `ignore_changes = [<attr>]` | Don't manage specified attributes | EC2 tags managed externally, auto-scaling desired count |

---

## 16. EKS with Terraform

### EKS Module Call (`eks.tf`)
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access = true

  eks_managed_node_groups = {
    nodes = {
      ami_type       = "AL2_x86_64"
      instance_types = [var.node_instance_type]
      min_size       = 1
      max_size       = 3
      desired_size   = var.node_desired_count
    }
  }

  tags = {
    Environment = "dev"
    ManagedBy   = "Terraform"
  }
}
```

### EKS VPC Requirements
```hcl
# Public subnets — for internet-facing load balancers
public_subnet_tags = {
  "kubernetes.io/role/elb" = 1
}

# Private subnets — for EKS nodes/pods (no direct internet)
private_subnet_tags = {
  "kubernetes.io/role/internal-elb" = 1
}
```

### Connect `kubectl` After Apply
```bash
aws eks update-kubeconfig --name <cluster-name> --region <region>

kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

### IAM Access Entry (EKS v20 module)
```hcl
data "aws_caller_identity" "current" {}

access_entries = {
  admin = {
    principal_arn = data.aws_caller_identity.current.arn
    policy_associations = {
      admin = {
        policy_arn   = "arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy"
        access_scope = { type = "cluster" }
      }
    }
  }
}
```

### EKS Destroy Order (AWS Resources)
```bash
# Step 1: Delete Kubernetes resources first (removes AWS LoadBalancer)
kubectl delete -f k8s/nginx-deployment.yaml

# Step 2: Wait for LoadBalancer to be fully removed in AWS console

# Step 3: Destroy all Terraform resources
terraform destroy
```

---

## 17. Best Practices

### File Structure
- Separate files for each concern: `providers.tf`, `variables.tf`, `outputs.tf`, `locals.tf`, `main.tf`
- One `modules/` directory per logical concern (vpc, security-group, ec2-instance)

### State Management
- Always use a **remote S3 backend** with encryption: `encrypt = true`
- Enable state **locking** (DynamoDB or native `use_lockfile = true`)
- Never commit `*.tfstate` or `.terraform/` to Git

### Variables
- Never hardcode sensitive values — use variables
- Use `terraform.tfvars` for defaults and environment-specific `.tfvars` files
- Add `validation` blocks to catch invalid values early

### Modules
- One concern per module (single responsibility)
- Always expose outputs for values that callers may need
- Pin registry modules with version constraints
- Don't assume region, account, or environment inside modules

### Workflow
```bash
# Always follow this order:
terraform fmt        # 1. Format
terraform validate   # 2. Validate
terraform plan       # 3. Preview
terraform apply      # 4. Apply (never skip plan!)
```

### Security
- `.gitignore` must exclude: `*.tfvars`, `*.tfstate`, `.terraform/`, `.terraform.lock.hcl`
- No credentials hardcoded anywhere — use IAM roles or `aws configure`
- State encrypted at rest with `encrypt = true`
- Restrict console/manual access — all changes through CI/CD

### Tagging
```hcl
# Tag every resource consistently:
tags = {
  Name        = "${local.name_prefix}-resource"
  Environment = local.environment
  Project     = var.project_name
  ManagedBy   = "Terraform"
}
```

### Naming Convention
```
<environment>-<project>-<resource>
# Examples:
# dev-terraweek-VPC
# terraweek-prod-Server
# terraweek-staging-SG
```

### Cleanup
- Always `terraform destroy` non-production environments when not in use
- Destroy in reverse order for EKS (k8s → terraform)

---

## 18. Plan Symbol Reference

| Symbol | Meaning |
|--------|---------|
| `+` | Resource will be **created** |
| `-` | Resource will be **destroyed** |
| `~` | Resource will be **updated in-place** |
| `-/+` | Resource will be **destroyed then re-created** |

---

