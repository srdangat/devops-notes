# Terraform: Providers, Resources, and Dependencies

## Provider Version Constraints

Version constraints control which provider versions Terraform can use.

| Constraint | Meaning |
| ---------- | ------- |
| `~> 5.0`   | Allows any 5.x version, but not 6.0 or higher |
| `>= 5.0`   | Allows 5.0 and any higher version  |
| `= 5.0.0`  | Allows only exactly version 5.0.0 |

---

## The Provider Lock File (`.terraform.lock.hcl`)

- Locks the **exact provider version** used in the project.
- Ensures Terraform always installs the same version that satisfies the version constraint.
- Stores **cryptographic hashes** to verify provider integrity — confirming the binary is authentic and unmodified.
- Should be committed to version control so all team members use the same provider version.

---

## Implicit Dependencies

Terraform automatically determines the creation order of resources by analyzing **attribute references** between them.

When a resource references an attribute of another (e.g., `vpc_id = aws_vpc.main.id`), Terraform infers that the referenced resource must be created first.

**Example dependency chain for a VPC setup:**

| Relationship | Reference |
| --- | --- |
| Subnet → VPC | `aws_subnet` references `aws_vpc.id` |
| Internet Gateway → VPC | `aws_internet_gateway` references `aws_vpc.id` |
| Route Table → VPC | `aws_route_table` references `aws_vpc.id` |
| Route Table → Internet Gateway | `aws_route_table` references `aws_internet_gateway.id` |
| Route Table Association → Subnet | references `aws_subnet.id` |
| Route Table Association → Route Table | references `aws_route_table.id` |

---

## Explicit Dependencies (`depends_on`)

Used when Terraform **cannot automatically detect** a dependency because there is no direct attribute reference between resources.

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = "my-app-logs"

  depends_on = [aws_instance.main]
}
```

**When to use `depends_on`:**

- **RDS depends on VPC & Subnets** — Ensure the VPC and subnets exist before creating the RDS database.
- **EC2 depends on IAM Role** — Ensure an IAM role is fully created before attaching it to an EC2 instance.
- **ACM Certificate depends on CloudFront** — Ensure a certificate is issued before attaching it to a CloudFront distribution.

---

## Implicit vs. Explicit Dependencies

| Type | How it works | Example |
| --- | --- | --- |
| **Implicit** | Terraform detects automatically via attribute references | EC2 instance referencing a security group ID |
| **Explicit** | Manually declared with `depends_on` | Auto Scaling Group explicitly depending on a Launch Template |

---

## Lifecycle Rules

The `lifecycle` block controls how Terraform manages the creation, update, and deletion of a resource.

### `create_before_destroy`

By default, Terraform destroys a resource before creating its replacement. Setting `create_before_destroy = true` reverses this — the new resource is created first, then the old one is destroyed.

```hcl
lifecycle {
  create_before_destroy = true
}
```

**Use case:** Replacing an RDS instance or EC2 instance with zero downtime.

---

### `prevent_destroy`

Protects a resource from accidental deletion. Terraform will raise an error and block any operation that would destroy the resource.

```hcl
lifecycle {
  prevent_destroy = true
}
```

**Use case:** Protecting a production S3 bucket or database containing critical data.

---

### `ignore_changes`

Tells Terraform to ignore changes to specific attributes during updates. Useful for attributes managed outside of Terraform (e.g., manually).

```hcl
lifecycle {
  ignore_changes = [tags, security_groups]
}
```

**Use case:** Ignoring EC2 instance tags or security group rules that are updated manually.

---

## Destroy Order

Terraform destroys resources in **reverse dependency order** — the opposite of how it creates them. Resources with no dependents are destroyed first, working back toward foundational resources like the VPC.
