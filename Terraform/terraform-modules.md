# Terraform -- Modules: Building Reusable Infrastructure

---

## Module Concepts

### Root module vs child module

| Aspect       | Root Module                                        | Child Module                                        |
| ------------ | -------------------------------------------------- | --------------------------------------------------- |
| What it is   | The main directory where Terraform commands are run | A reusable sub-directory called by the root module  |
| Purpose      | Wires everything together                          | Handles a specific task (EC2, VPC, SG, etc.)        |
| Location     | Project root                                       | `modules/<n>/` subdirectory                      |

A module is just a directory with `.tf` files -- there is nothing special about the files themselves.

---

## Standard Module Structure

```
project-root/
  main.tf               # Root module -- calls child modules
  variables.tf          # Root input variables
  outputs.tf            # Root outputs
  providers.tf          # Provider config
  modules/
    ec2-instance/
      main.tf           # EC2 resource definition
      variables.tf      # Module inputs
      outputs.tf        # Module outputs
    security-group/
      main.tf           # Security group resource definition
      variables.tf      # Module inputs
      outputs.tf        # Module outputs
```

---

## Building a Custom EC2 Module

### `modules/ec2-instance/variables.tf`
```hcl
variable "ami_id"             { type = string }
variable "instance_type"      { type = string; default = "t2.micro" }
variable "subnet_id"          { type = string }
variable "security_group_ids" { type = list(string) }
variable "instance_name"      { type = string }
variable "tags"               { type = map(string); default = {} }
```

### `modules/ec2-instance/main.tf`
```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(var.tags, {
    Name = var.instance_name
  })
}
```

### `modules/ec2-instance/outputs.tf`
```hcl
output "instance_id" { value = aws_instance.this.id }
output "public_ip"   { value = aws_instance.this.public_ip }
output "private_ip"  { value = aws_instance.this.private_ip }
```

---

## Building a Custom Security Group Module

### `modules/security-group/variables.tf`
```hcl
variable "vpc_id"        { type = string }
variable "sg_name"       { type = string }
variable "ingress_ports" { type = list(number); default = [22, 80] }
variable "tags"          { type = map(string); default = {} }
```

### `modules/security-group/main.tf`
```hcl
resource "aws_security_group" "this" {
  name   = var.sg_name
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

  tags = var.tags
}
```

The `dynamic` block loops over the `ingress_ports` list to generate repeated nested blocks automatically.

### `modules/security-group/outputs.tf`
```hcl
output "sg_id" { value = aws_security_group.this.id }
```

---

## Calling Modules from Root

### Root `main.tf`
```hcl
module "web_sg" {
  source        = "./modules/security-group"
  vpc_id        = aws_vpc.main.id
  sg_name       = "project-web-sg"
  ingress_ports = [22, 80, 443]
  tags          = local.common_tags
}

module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "project-web"
  tags               = local.common_tags
}

module "api_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t2.micro"
  subnet_id          = aws_subnet.public.id
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "project-api"
  tags               = local.common_tags
}
```

### Root `outputs.tf`
```hcl
output "web_server_ip" { value = module.web_server.public_ip }
output "api_server_ip" { value = module.api_server.public_ip }
```

### Apply commands
```bash
terraform init     # Downloads/links local modules
terraform plan     # Shows all resources from both module calls
terraform apply
```

---

## Using Public Registry Modules

Instead of writing VPC resources from scratch, use the official Terraform Registry module.

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "project-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway   = false
  enable_dns_hostnames = true

  tags = local.common_tags
}
```

Reference outputs from the registry module in other resources:
```hcl
vpc_id    = module.vpc.vpc_id
subnet_id = module.vpc.public_subnets[0]
```

### Where Terraform downloads registry modules

Registry modules are downloaded to `.terraform/modules/` on `terraform init`.

### Hand-written VPC vs Registry VPC module

| Approach             | Resources created |
| -------------------- | ----------------- |
| Hand-written VPC     | ~6 resources      |
| Registry VPC module  | ~17 resources (includes default ACLs, route tables, associations, etc.) |

---

## Module Versioning

Pin registry module versions to avoid unexpected breaking changes.

```hcl
version = "5.1.0"         # Exact version -- no surprises
version = "~> 5.0"        # Any 5.x patch/minor version
version = ">= 5.0, < 6.0" # Explicit range
```

```bash
terraform init -upgrade   # Check for and download newer versions
```

### How modules appear in state

After applying, state resource names are prefixed with the module name:
```
module.vpc.aws_vpc.this
module.web_server.aws_instance.this
module.web_sg.aws_security_group.this
```

---

## Module Best Practices

| Practice                  | Why it matters                                        |
| ------------------------- | ----------------------------------------------------- |
| Use clear names           | Resources and variables should be easy to understand  |
| Keep files organized      | Stick to `main.tf`, `variables.tf`, `outputs.tf`      |
| Use locals                | Avoid repeating the same values across resources      |
| Don't assume environment  | No hardcoded region, account IDs, or environment names |
| Validate inputs           | Add `validation` blocks so wrong values fail early    |
| Use defaults carefully    | Only provide defaults when a sensible universal value exists |
| Keep modules small        | Focused modules are easier to reuse and debug         |
| Test before using         | Run `plan` and validate before sharing or publishing  |
