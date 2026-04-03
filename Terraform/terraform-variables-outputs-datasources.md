# Terraform -- Variables, Outputs, Data Sources, and Expressions

---

## Variable Types

Terraform supports five variable types:

### `string` -- names or any text
```hcl
variable "instance_name" {
  type    = string
  default = "my-ec2"
}

# Usage
tags = { Name = var.instance_name }
```

### `number` -- counts or numeric values
```hcl
variable "instance_count" {
  type    = number
  default = 2
}

# Usage
count = var.instance_count
```

### `bool` -- conditional true/false
```hcl
variable "assign_public_ip" {
  type    = bool
  default = true
}

# Usage
associate_public_ip_address = var.assign_public_ip
```

### `list` -- ordered collection
```hcl
variable "security_groups" {
  type    = list(string)
  default = ["sg-123", "sg-456"]
}

# Usage
vpc_security_group_ids = var.security_groups
```

### `map` -- key-value pairs
```hcl
variable "s3_buckets" {
  type = map(string)
  default = {
    bucket1 = "us-east-1"
    bucket2 = "us-west-2"
  }
}

# Usage
for_each = var.s3_buckets
bucket   = each.key
region   = each.value
```

---

## Variable Files and Precedence

### Common variable files

**`terraform.tfvars`** -- auto-loaded by default:
```hcl
project_name  = "myproject"
environment   = "dev"
instance_type = "t2.micro"
```

**`prod.tfvars`** -- loaded explicitly with `-var-file`:
```hcl
project_name  = "myproject"
environment   = "prod"
instance_type = "t3.small"
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
```

### Applying with variable files
```bash
terraform plan                              # Uses terraform.tfvars automatically
terraform plan -var-file="prod.tfvars"      # Uses a specific tfvars file
terraform plan -var="instance_type=t2.nano" # CLI flag overrides everything
```

### Setting environment variables
```bash
export TF_VAR_environment="staging"
terraform plan    # env var overrides default, but not terraform.tfvars
```

> **Note:** `TF_VAR_<name>` overrides only the `default` in `variables.tf`. If the same variable is set in `terraform.tfvars`, the tfvars value takes precedence.

---

## Variable Precedence (Lowest → Highest)

| Priority (High → Low) | Source                     | Example                                    | Value   |
| --------------------- | -------------------------- | ------------------------------------------ | ------- |
| 1 (Highest)           | Command-line (`-var`)      | `terraform plan -var="environment=qa"`     | `qa`    |
| 2                     | Command-line (`-var-file`) | `terraform plan -var-file="prod.tfvars"`   | `prod`  |
| 3                     | Auto-loaded tfvars         | `terraform.tfvars → environment = "stage"` | `stage` |
| 4                     | Environment variable       | `TF_VAR_environment=uat`                   | `uat`   |
| 5 (Lowest)            | Default value              | `default = "dev"`                          | `dev`   |

---

## Outputs

### `outputs.tf` example
```hcl
output "vpc_id"              { value = aws_vpc.main.id }
output "subnet_id"           { value = aws_subnet.public.id }
output "instance_id"         { value = aws_instance.server.id }
output "instance_public_ip"  { value = aws_instance.server.public_ip }
output "instance_public_dns" { value = aws_instance.server.public_dns }
output "security_group_id"   { value = aws_security_group.main.id }
```

### Viewing outputs
```bash
terraform output                        # Show all outputs
terraform output instance_public_ip     # Show a specific output
terraform output -json                  # JSON format for scripting
```

---

## Data Sources

Use data sources to fetch existing resources dynamically instead of hardcoding values.

### Fetch the latest Amazon Linux 2 AMI
```hcl
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Usage
ami = data.aws_ami.amazon_linux.id
```

### Fetch available Availability Zones
```hcl
data "aws_availability_zones" "available" {}

# Usage -- pick the first AZ
availability_zone = data.aws_availability_zones.available.names[0]
```

### `resource` vs `data` source

| Feature           | `resource`           | `data`                         |
| ----------------- | -------------------- | ------------------------------ |
| Creates infra     | Yes                  | No                             |
| Managed by TF     | Yes                  | No                             |
| Stored in state   | Yes                  | Read-only reference            |
| Lifecycle actions | create/update/delete | read-only                      |
| Use case          | EC2, VPC, Subnet     | AMI lookup, AZs, existing VPCs |

---

## Locals

Locals define reusable internal values or expressions.

```hcl
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

### Using locals in resources
```hcl
tags = merge(local.common_tags, {
  Name = "${local.name_prefix}-server"
})
```

---

## `variable` vs `local` vs `output` vs `data`

| Block      | Purpose                                              |
| ---------- | ---------------------------------------------------- |
| `variable` | Accepts input values from the user                   |
| `local`    | Defines internal reusable values or expressions      |
| `data`     | Fetches existing resources from the provider (read-only) |
| `output`   | Displays or exports values after execution           |

---

## Built-in Functions and Conditional Expressions

### Practice in the Terraform console
```bash
terraform console
```

### String functions
```hcl
upper("terraweek")                          # "TERRAWEEK"
join("-", ["terra", "week", "2026"])        # "terra-week-2026"
format("arn:aws:s3:::%s", "my-bucket")     # "arn:aws:s3:::my-bucket"
```

### Collection functions
```hcl
length(["a", "b", "c"])                             # 3
lookup({dev = "t2.micro", prod = "t3.small"}, "dev") # "t2.micro"
toset(["a", "b", "a"])                              # removes duplicates → {"a", "b"}
```

### Networking function
```hcl
cidrsubnet("10.0.0.0/16", 8, 1)   # "10.0.1.0/24"
```

### Conditional expression
```hcl
instance_type = var.environment == "prod" ? "t3.small" : "t2.micro"
```

### Five most useful functions

| Function       | Use case                         | Example                                                        | Result               |
| -------------- | -------------------------------- | -------------------------------------------------------------- | -------------------- |
| `upper()`      | String formatting                | `upper(var.environment)`                                       | `"dev" → "DEV"`      |
| `join()`       | Combine values into a string     | `join("-", ["app", var.environment, "2026"])`                  | `"app-dev-2026"`     |
| `format()`     | Build structured strings         | `format("arn:aws:s3:::%s", "my-bucket")`                       | `"arn:aws:s3:::my-bucket"` |
| `lookup()`     | Environment-based selection      | `lookup({dev = "t2.micro", prod = "t3.small"}, "dev")`         | `"t2.micro"`         |
| `cidrsubnet()` | Network and subnet creation      | `cidrsubnet("10.0.0.0/16", 8, 1)`                              | `"10.0.1.0/24"`      |
