# Terraform & Infrastructure as Code

---

## Infrastructure as Code (IaC)

**What is IaC?**
Infrastructure as Code is the practice of defining and managing infrastructure (servers, networks, databases, IAM, etc.) using code instead of clicking around in a UI or doing things manually.

**Why it matters:**
- Automates how infrastructure is created and torn down
- Makes it easy to undo or reproduce changes
- Keeps all environments (dev, staging, prod) consistent
- Reduces human error and speeds up delivery

**Problems IaC solves vs. manual AWS console work:**
- Automation — no repetitive clicking
- Consistency — same config every time
- Version control — track changes in Git
- Reproducibility — recreate infrastructure in seconds

---

## Terraform vs. Other IaC Tools

| Tool | Key Difference |
|---|---|
| **Terraform** | Multi-cloud, declarative, uses HCL |
| **CloudFormation** | AWS-only, declarative |
| **Ansible** | Primarily configuration management, not provisioning |
| **Pulumi** | Uses general-purpose programming languages (Python, TypeScript, etc.) |

---

## Terraform Core Concepts

**Declarative:** You describe the *desired state* of your infrastructure. Terraform figures out the steps to reach it — you don't write step-by-step instructions.

**Cloud-agnostic:** Works across AWS, Azure, GCP, and more using the same HCL language and workflow.

---

## Installation

```bash
# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Linux (amd64)
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Windows
choco install terraform

# Verify
terraform -version
```

**Configure AWS CLI:**
```bash
aws configure
# Enter: Access Key ID, Secret Access Key, region (e.g. us-east-1), output format (json)

# Verify access
aws sts get-caller-identity
```

---

## Basic Terraform Config Structure (`main.tf`)

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
  region = "us-east-1"
}

resource "aws_s3_bucket" "bucket" {
  bucket = "my-unique-bucket-name-2026"
}

resource "aws_instance" "instance" {
  ami           = "ami-0f5ee92e2d63afc18"
  instance_type = "t2.micro"

  tags = {
    Name = "MyInstance"
  }
}
```

---

## Core Terraform Commands

| Command | What it does |
|---|---|
| `terraform init` | Initializes the project; downloads provider plugins and modules |
| `terraform plan` | Previews what changes Terraform will make — nothing is applied |
| `terraform apply` | Creates or updates infrastructure based on the plan |
| `terraform destroy` | Deletes all resources managed by Terraform |
| `terraform show` | Displays the current state in a human-readable format |
| `terraform state list` | Lists all resources tracked in the state file |
| `terraform state show <resource>` | Shows detailed info about a specific resource in state |

**`terraform init` details:**
- Downloads the required provider plugins (e.g. AWS provider)
- Creates the `.terraform/` directory containing those plugins
- Sets up the backend configuration

---

## The Terraform Lifecycle

```
Write config (main.tf)
        ↓
terraform init       → downloads providers
        ↓
terraform plan       → shows what will change
        ↓
terraform apply      → creates/updates resources
        ↓
terraform destroy    → removes all resources
```

---

## The State File (`terraform.tfstate`)

**What it stores:**
- Resource IDs and ARNs
- Current resource attributes (IPs, tags, config)
- Mapping between Terraform config and real infrastructure
- Dependencies between resources

**Why it matters:**
- Acts as Terraform's source of truth
- Enables Terraform to detect drift and plan accurate updates
- Prevents duplicate resource creation (if it's in state, Terraform won't recreate it)

**Rules:**
- **Never manually edit** the state file — can corrupt state and cause mismatches
- **Never commit to Git** — contains sensitive data (keys, IPs, ARNs) and causes team conflicts
- For teams, use **remote state** (e.g. S3 locking)

---

## Reading `terraform plan` Output

| Symbol | Meaning |
|---|---|
| `+` | Resource will be **created** |
| `-` | Resource will be **destroyed** |
| `~` | Resource will be **updated in-place** |
| `-/+` | Resource will be **destroyed and recreated** |

**Example:** Changing an EC2 tag → `~` (in-place update, no downtime)

---
