# Terraform -- State Management and Remote Backends

---

## Inspecting State

```bash
terraform show                              # Full state in human-readable format
terraform state list                        # All resources tracked by Terraform
terraform state show aws_instance.<name>    # Every attribute of the instance
terraform state show aws_vpc.<name>         # Every attribute of the VPC
```

### What does Terraform track in state?

Terraform stores far more than what you define. For an EC2 instance, the state includes:

- **Identity:** `ami`, `instance_type`, `tags`, `key_name`
- **Networking:** `private_ip`, `public_ip`, `private_dns`, `public_dns`, `subnet_id`, `vpc_security_group_ids`, `primary_network_interface_id`
- **Storage:** `root_block_device`, `volume_id`, `volume_size`, `volume_type`, `delete_on_termination`

### The `serial` number

The `serial` number in `terraform.tfstate` represents how many times the state has been updated. It increments with every change, helping Terraform detect conflicts and prevent stale overwrites.

---

## Remote Backend with S3

Storing state locally is risky -- a single deleted file means losing all tracked infrastructure. Use an S3 backend with DynamoDB for locking.

### 1. Create the backend infrastructure

```bash
# Create S3 bucket
aws s3api create-bucket \
  --bucket my-tf-state-bucket \
  --region ap-south-1 \
  --create-bucket-configuration LocationConstraint=ap-south-1

# Enable versioning (allows recovery of previous state)
aws s3api put-bucket-versioning \
  --bucket my-tf-state-bucket \
  --versioning-configuration Status=Enabled

# Create DynamoDB table for state locking
aws dynamodb create-table \
  --table-name tf-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region ap-south-1
```

### 2. Add the backend block

```hcl
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "dev/terraform.tfstate"
    region         = "ap-south-1"
    dynamodb_table = "tf-state-lock"
    encrypt        = true
  }
}
```

### 3. Migrate state

```bash
terraform init
# Terraform will ask: "Do you want to copy existing state to the new backend?" → say yes
```

After migration:
- The S3 bucket will contain `dev/terraform.tfstate`
- The local `terraform.tfstate` will be empty or gone
- `terraform plan` should show no changes

### Local vs Remote State

| Aspect          | Local State              | Remote State (S3 + DynamoDB)        |
| --------------- | ------------------------ | ----------------------------------- |
| Location        | `terraform.tfstate` file | S3 bucket                           |
| Locking         | No locking               | DynamoDB prevents concurrent writes |
| Team access     | Single user only         | Shared across team                  |
| Versioning      | Manual backups only      | S3 versioning built-in              |
| Risk            | File loss = disaster     | Durable, recoverable                |

---

## State Locking

State locking prevents two users from running `terraform apply` simultaneously, which could corrupt the state file.

When a lock is held, any other `terraform plan` or `apply` will show:
```
Error: Error acquiring the state lock

Terraform can't acquire the state lock because DynamoDB says the state is
already locked (ConditionalCheckFailedException).
```

### Releasing a stale lock

If a process crashes and leaves a stale lock:
```bash
terraform force-unlock <LOCK_ID>
```

> Locking is critical in team environments -- it prevents concurrent writes that could cause unintended or corrupted infrastructure changes.

---

## Importing Existing Resources

Not everything starts with Terraform. Use `terraform import` to bring manually created resources under Terraform management.

### Steps to import a resource

1. **Write the resource block** (just the minimal config first):
```hcl
resource "aws_s3_bucket" "imported" {
  bucket = "my-existing-bucket"
}
```

2. **Run the import command:**
```bash
terraform import aws_s3_bucket.imported my-existing-bucket
```

3. **Check the state:**
```bash
terraform state list    # The imported resource should now appear
```

4. **Run `terraform plan`:**
   - "No changes" → import was perfect
   - If changes appear → update your `.tf` config to match reality, then plan again

### `terraform import` vs creating from scratch

| Aspect           | `terraform import`                             | Create from scratch                    |
| ---------------- | ---------------------------------------------- | -------------------------------------- |
| What it does     | Brings an existing resource under TF management | Terraform creates a new resource       |
| State update     | Adds to state only (no infra change)           | Both state and resource are created    |
| Use case         | Migrating existing manual resources            | Standard workflow for new infra        |

---

## State Surgery: `mv` and `rm`

### Rename a resource in state

```bash
terraform state list
terraform state mv aws_s3_bucket.old_name aws_s3_bucket.new_name
```
Update the `.tf` file to match the new name, then run `terraform plan` -- it should show no changes.

### Remove a resource from state

```bash
terraform state rm aws_s3_bucket.new_name
```
Terraform no longer tracks the resource, but it still exists in AWS. Use this when you want to stop managing a resource without deleting it.

### Re-import after removal

```bash
terraform import aws_s3_bucket.new_name my-existing-bucket
```

### When to use each command

| Command         | When to use                                                              |
| --------------- | ------------------------------------------------------------------------ |
| `state mv`      | Renaming a resource in config without destroying and recreating it       |
| `state rm`      | Removing a resource from TF management while keeping it alive in the cloud |
| `import`        | Bringing an existing cloud resource under Terraform management           |
| `force-unlock`  | Releasing a stuck lock after a failed or crashed operation               |
| `refresh`       | Syncing state with real-world resources to detect drift                  |

---

## State Drift

State drift happens when someone changes infrastructure outside of Terraform -- through the AWS console, CLI, or another tool.

### Detecting drift

```bash
terraform plan
# Shows a diff if reality no longer matches the desired state
```

### Resolving drift

**Option A -- Reconcile (force reality back to config):**
```bash
terraform apply
```

**Option B -- Accept the drift (update config to match reality):**
Update your `.tf` files to reflect the manual change, then run `terraform plan` until it shows "No changes."

### Preventing drift in production

- Restrict direct console access for all team members
- Route all infrastructure changes through version-controlled CI/CD pipelines
- Use `terraform plan` in PRs to review changes before applying
