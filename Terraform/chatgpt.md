---
## 🧠 Concept Overview

Terraform and Vault integration ensures **secrets are fetched dynamically at runtime**, instead of being hardcoded in `.tf` files or environment variables.

**Key benefits:**

* No plaintext secrets in Terraform code or state files.
* Centralized secret management and audit via Vault.
* Dynamic secrets with automatic TTL and rotation.
---
## 🔐 Integration Methods

There are **3 main ways** to integrate Vault with Terraform:

| Method                                  | Description                                                                          | Use Case                                          |
| --------------------------------------- | ------------------------------------------------------------------------------------ | ------------------------------------------------- |
| 1️⃣ Terraform Vault Provider            | Use the `vault` provider to read secrets directly from Vault                         | Pull static/dynamic secrets during Terraform runs |
| 2️⃣ Vault Agent + Environment Variables | Use Vault Agent to auto-authenticate and inject secrets into Terraform’s environment | When Terraform is run from a machine or pipeline  |
| 3️⃣ Terraform Cloud/Enterprise + Vault  | Terraform Cloud integrates natively with Vault for dynamic credentials               | For enterprise setups using Terraform Cloud       |

Let’s focus on **method 1**, which is most common for DevOps engineers.
---
## ⚙️ Method 1: Terraform Vault Provider

### **Step 1: Enable Vault and store a secret**

In Vault:

```bash
vault server -dev
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'

# Store a secret
vault kv put secret/aws access_key="AKIAXXX" secret_key="abcd1234"
```

### **Step 2: Terraform Configuration**

**`providers.tf`**

```hcl
terraform {
  required_providers {
    vault = {
      source  = "hashicorp/vault"
      version = "~> 4.0"
    }
  }
}

provider "vault" {
  address = "http://127.0.0.1:8200"
  token   = "root"   # For real use, authenticate dynamically (AppRole, AWS IAM, etc.)
}
```

---

### **Step 3: Fetch secret from Vault**

**`main.tf`**

```hcl
data "vault_kv_secret_v2" "aws_creds" {
  mount = "secret"
  name  = "aws"
}

output "aws_access_key" {
  value     = data.vault_kv_secret_v2.aws_creds.data["access_key"]
  sensitive = true
}

output "aws_secret_key" {
  value     = data.vault_kv_secret_v2.aws_creds.data["secret_key"]
  sensitive = true
}
```

Run:

```bash
terraform init
terraform apply
```

✅ You’ll see Terraform fetch secrets securely from Vault (not stored in plain text in state/output).

---

## 🔁 Optional: Dynamic Secrets Example

Vault can issue **temporary credentials**, e.g., for AWS or database.

### In Vault:

```bash
vault secrets enable aws

vault write aws/config/root \
  access_key=AKIA... \
  secret_key=abcd...

vault write aws/roles/dev-role \
  credential_type=iam_user \
  policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [{"Effect": "Allow", "Action": "ec2:*", "Resource": "*"}]
}
EOF
```

### In Terraform:

```hcl
data "vault_aws_access_credentials" "creds" {
  backend = "aws"
  role    = "dev-role"
}

output "temp_access_key" {
  value     = data.vault_aws_access_credentials.creds.access_key
  sensitive = true
}
```

These credentials automatically expire based on the TTL you set in Vault — no manual rotation needed 🚀

---

## 🔄 Method 2: Vault Agent with Terraform

In CI/CD pipelines, you can run a **Vault Agent** that:

* Authenticates automatically (e.g., via AppRole or AWS IAM)
* Writes secrets to a file or environment variables
* Terraform reads them dynamically

Example:

```bash
vault agent -config=vault-agent-config.hcl
```

Then Terraform picks them up:

```bash
provider "aws" {
  access_key = env.AWS_ACCESS_KEY
  secret_key = env.AWS_SECRET_KEY
}
```

---

## 🧩 Method 3: Terraform Cloud & Vault

Terraform Cloud can natively integrate with Vault to **dynamically fetch** secrets (AWS, Azure, GCP credentials, etc.) at runtime — perfect for enterprise security.

You just configure a Vault connection in Terraform Cloud and use `vault:<path>` references in variables.

---

## 🚨 Common Pitfalls & Troubleshooting

| Problem                       | Root Cause                                       | Fix                                                  |
| ----------------------------- | ------------------------------------------------ | ---------------------------------------------------- |
| `permission denied`           | Vault policy doesn’t allow access to secret path | Check Vault policy with `vault policy read <policy>` |
| `no data found at path`       | Wrong mount or secret name                       | Verify secret path with `vault kv get <path>`        |
| Secrets visible in state file | Not marked as `sensitive = true`                 | Always mark secret outputs as sensitive              |
| Token expired                 | Static token used in provider                    | Use dynamic authentication (AppRole, AWS IAM, etc.)  |

---

## 🧠 Real-Time Interview Tips

You can expect questions like:

1. *How do you prevent secrets from being exposed in Terraform state files?*
   ➤ Mark outputs as sensitive + use Vault dynamic secrets.

2. *Can Terraform write secrets back into Vault?*
   ➤ Yes, using `vault_generic_secret` resource.

3. *How can Terraform authenticate to Vault securely?*
   ➤ Use AppRole, AWS IAM, or Kubernetes Auth methods instead of static tokens.

---

---

## 🧠 What Are Terraform Workspaces?

A **workspace** in Terraform is an **isolated environment** within the same Terraform configuration directory.

Each workspace:

* Has **its own state file** (`terraform.tfstate`), stored separately.
* Uses the **same Terraform code**, but can have **different variables** or **resources** depending on the workspace.

### 🧩 In simple terms:

> Workspaces allow you to reuse the same Terraform configuration to manage multiple environments without copying your `.tf` files.

---

## 📦 Default Workspace

When you initialize Terraform, it automatically creates a **default** workspace:

```bash
terraform workspace show
# Output: default
```

All resources you create initially go into this workspace unless you create others.

---

## 🛠️ Workspace Commands

Here are the key Terraform workspace commands:

| Command                              | Description                          |
| ------------------------------------ | ------------------------------------ |
| `terraform workspace list`           | List all existing workspaces         |
| `terraform workspace show`           | Show current workspace               |
| `terraform workspace new <name>`     | Create a new workspace               |
| `terraform workspace select <name>`  | Switch to an existing workspace      |
| `terraform workspace delete <name>`  | Delete a workspace                   |
| `terraform workspace select default` | Switch back to the default workspace |

---

## 🧰 Example: Multiple Environments

Suppose you have one configuration for an AWS S3 bucket:

**`main.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "example" {
  bucket = "myapp-${terraform.workspace}-bucket"
}
```

---

### Step 1: Initialize Terraform

```bash
terraform init
```

### Step 2: Create workspaces

```bash
terraform workspace new dev
terraform workspace new prod
```

### Step 3: Deploy in each workspace

```bash
terraform workspace select dev
terraform apply
# Creates: myapp-dev-bucket

terraform workspace select prod
terraform apply
# Creates: myapp-prod-bucket
```

✅ **Result:** You’ve created two separate buckets using the same Terraform code — isolated by workspace.

---

## 🧩 Using Variables with Workspaces

You can create **different variable files** per workspace.

Example:

```
variables.tf
dev.tfvars
prod.tfvars
```

When applying:

```bash
terraform workspace select dev
terraform apply -var-file="dev.tfvars"

terraform workspace select prod
terraform apply -var-file="prod.tfvars"
```

Or use logic in Terraform:

```hcl
variable "instance_type" {
  default = "t2.micro"
}

locals {
  instance_type = terraform.workspace == "prod" ? "t3.medium" : var.instance_type
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = local.instance_type
}
```

---

## 🗂️ Workspace Directory Structure

Terraform stores workspace state files separately.

If you use a **local backend**, the structure looks like this:

```
terraform.tfstate.d/
├── dev/
│   └── terraform.tfstate
├── prod/
│   └── terraform.tfstate
```

Each directory holds its own isolated state.

---

## ☁️ Remote Workspaces (Terraform Cloud/Enterprise)

When using **Terraform Cloud** or **Enterprise**, *workspaces* represent **different runs/environments**, managed via the UI or API.

* Each workspace can connect to different variable sets and credentials.
* Perfect for managing CI/CD pipelines or multiple teams.

---

## ⚠️ Common Mistakes

| Problem                          | Root Cause                                                     | Fix                                                               |
| -------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------- |
| Resources overwrite each other   | Same resource name and identifiers (e.g., same S3 bucket name) | Add `${terraform.workspace}` in resource names                    |
| Hardcoding environment variables | Not using workspaces or conditionals                           | Use workspace-aware logic or separate `*.tfvars`                  |
| State file confusion             | Mixing up workspaces                                           | Always check with `terraform workspace show` before running apply |

---

## 💡 Best Practices

✅ Use workspaces for:

* Environment separation (dev/stage/prod)
* Short-lived environments (e.g., PR testing)

❌ Avoid workspaces for:

* **Completely different infrastructures**
  (e.g., AWS vs Azure → use separate directories or repos)

✅ Always use `${terraform.workspace}` in naming to avoid collisions.
✅ Store state remotely (e.g., in S3) to safely isolate workspace states.

---

## 🧠 Interview Questions on Terraform Workspaces

| Question                                                                 | Quick Answer                                                           |
| ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| What is the purpose of Terraform workspaces?                             | To manage multiple isolated environments using the same configuration. |
| Where are workspace state files stored?                                  | In `terraform.tfstate.d/<workspace_name>/` for local backend.          |
| How can you dynamically reference a workspace in Terraform?              | Using the `terraform.workspace` variable.                              |
| What’s the difference between Terraform workspaces and multiple folders? | Workspaces share the same config, folders duplicate configs.           |
| Are workspaces recommended for multi-cloud setups?                       | No — use separate configs instead.                                     |

---

Excellent — Terraform **modules** are one of the **most powerful and reusable** features in Infrastructure as Code.
They help you organize, standardize, and scale Terraform configurations — which is a key skill for any **DevOps engineer** or **Terraform practitioner** 💪

Let’s break it down clearly 👇

---

## 🧠 What Are Terraform Modules?

A **Terraform module** is simply a **collection of `.tf` files** (resources, variables, outputs, etc.) that can be **reused** like a function or library in programming.

### 👉 Think of a module as:

> A reusable block of Terraform code that defines a piece of infrastructure.

You can use modules to manage:

* A **single resource** (like an EC2 instance)
* A **set of related resources** (like a VPC with subnets, NAT, and routing)
* **Entire environments** (like dev/prod setups)

---

## 🧩 Types of Modules

| Type                     | Description                                                   | Example                             |
| ------------------------ | ------------------------------------------------------------- | ----------------------------------- |
| **Root Module**          | The main directory where Terraform commands are run.          | Your main Terraform project folder. |
| **Child Module**         | A module called from another module using the `module` block. | Custom module for EC2 or VPC.       |
| **Public/Remote Module** | Pre-built modules from Terraform Registry or Git.             | `terraform-aws-modules/vpc/aws`     |

---

## 🗂️ Module Folder Structure

Here’s a simple module structure:

```
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    └── ec2-instance/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

---

## ⚙️ Example: Creating and Using a Module

### 1️⃣ **Create a Module** (Custom EC2 Module)

**`modules/ec2-instance/main.tf`**

```hcl
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type
  tags = {
    Name = var.instance_name
  }
}
```

**`modules/ec2-instance/variables.tf`**

```hcl
variable "ami_id" {}
variable "instance_type" {}
variable "instance_name" {}
```

**`modules/ec2-instance/outputs.tf`**

```hcl
output "instance_id" {
  value = aws_instance.this.id
}
```

---

### 2️⃣ **Call the Module in Root Configuration**

**`main.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

module "my_ec2" {
  source         = "./modules/ec2-instance"
  ami_id         = "ami-0c55b159cbfafe1f0"
  instance_type  = "t2.micro"
  instance_name  = "dev-server"
}

output "ec2_id" {
  value = module.my_ec2.instance_id
}
```

Run:

```bash
terraform init
terraform apply
```

✅ Terraform provisions the EC2 instance using your reusable module.

---

## 🌎 Using Public Modules (Terraform Registry)

Instead of writing from scratch, you can use **pre-built modules** from [Terraform Registry](https://registry.terraform.io/).

Example (AWS VPC module):

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
}
```

---

## 🧩 Module Sources

You can specify the `source` of a module in different ways:

| Source Type        | Example                                                                      |
| ------------------ | ---------------------------------------------------------------------------- |
| Local Path         | `source = "./modules/vpc"`                                                   |
| Git Repo           | `source = "git::https://github.com/org/terraform-modules.git//vpc?ref=v1.0"` |
| Terraform Registry | `source = "terraform-aws-modules/vpc/aws"`                                   |
| HTTP URL           | `source = "https://example.com/module.zip"`                                  |

---

## 🧠 Variables and Outputs in Modules

Modules communicate with the root configuration using **input variables** and **outputs**.

### Inputs → Variables you pass **to** the module

### Outputs → Values you get **from** the module

Example:

```hcl
module "ec2" {
  source         = "./modules/ec2-instance"
  ami_id         = "ami-0c55b159cbfafe1f0"
  instance_type  = "t3.micro"
  instance_name  = "prod-server"
}

output "server_id" {
  value = module.ec2.instance_id
}
```

---

## ⚡ Best Practices for Terraform Modules

✅ **1. Keep modules small and focused**

* One module should do one thing well (e.g., EC2, VPC, S3).

✅ **2. Use versioning**

* Pin module versions using `version = "x.y.z"` to prevent breaking changes.

✅ **3. Use variables with defaults**

* Avoid hardcoding values.

✅ **4. Avoid circular dependencies**

* Modules shouldn’t depend on each other in loops.

✅ **5. Use meaningful outputs**

* Only expose what’s needed (e.g., IDs, endpoints).

✅ **6. Use standardized naming**

* Consistent naming = predictable resources across environments.

---

## 🧩 Real-Time Use Case (Reusable Infra)

You can use the same module for multiple environments:

```hcl
module "dev_ec2" {
  source         = "./modules/ec2-instance"
  ami_id         = "ami-0c55b159cbfafe1f0"
  instance_type  = "t2.micro"
  instance_name  = "dev-server"
}

module "prod_ec2" {
  source         = "./modules/ec2-instance"
  ami_id         = "ami-0c55b159cbfafe1f0"
  instance_type  = "t3.large"
  instance_name  = "prod-server"
}
```

Same module, different variables → different environments 🚀

---

## ⚠️ Common Issues

| Issue                  | Root Cause                              | Fix                                                        |
| ---------------------- | --------------------------------------- | ---------------------------------------------------------- |
| “Unsupported source”   | Wrong module path or syntax             | Verify module path and quotes                              |
| Variables not found    | Missing required variables              | Define all variables or give defaults                      |
| Resource name conflict | Using same resource names               | Use `var.environment` or `${terraform.workspace}` in names |
| State file conflict    | Same backend for different environments | Use different workspaces or backend key names              |

---

## 🧠 Interview-Focused Q&A

| Question                                              | Answer                                                                 |
| ----------------------------------------------------- | ---------------------------------------------------------------------- |
| What is a Terraform module?                           | A reusable collection of Terraform resources.                          |
| What are the benefits of using modules?               | Reusability, standardization, scalability, and clean structure.        |
| How do you pass variables to a module?                | Using the `module` block with `variable_name = value`.                 |
| How can you reuse modules across projects?            | Store modules in Git or Terraform Registry and reference via `source`. |
| What’s the difference between root and child modules? | Root = main config; Child = reusable submodule called by root.         |
| Can modules depend on each other?                     | Yes, but avoid circular dependencies. Use outputs/inputs to link.      |

---

---

## 🧠 Concept Overview

Terraform provides ways to **create multiple similar resources dynamically**, instead of duplicating code.

There are **3 main constructs**:

| Feature               | Purpose                                                                   |
| --------------------- | ------------------------------------------------------------------------- |
| **`count`**           | Create *N* number of identical resources (based on integer count).        |
| **`for_each`**        | Create resources for each *unique key/value pair* in a map or set.        |
| **`for` expressions** | Used for transforming lists/maps — like loops inside variables or locals. |

---

## 🧩 1️⃣ Terraform `count`

### ✅ Definition:

`count` lets you define how many **instances** of a resource to create — using an integer.

### 🧱 Example:

```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}
```

This creates **3 EC2 instances**:

```
aws_instance.web[0]
aws_instance.web[1]
aws_instance.web[2]
```

---

### 💡 Accessing with `count.index`

You can reference each instance:

```hcl
output "instance_ids" {
  value = aws_instance.web[*].id
}
```

The `[*]` is a **splat operator**, which collects all instance IDs into a list.

---

### ⚠️ Limitation:

* Works only with **integer counts**, not unique keys.
* Changing the list order or count value may **destroy and recreate** resources (because indexes change).

---

## 🧩 2️⃣ Terraform `for_each`

### ✅ Definition:

`for_each` is used when you have a **map or a set** of values — and you want to create resources **per unique key**.

### 🧱 Example 1: Using a list of strings

```hcl
variable "server_names" {
  default = ["app1", "app2", "app3"]
}

resource "aws_instance" "web" {
  for_each = toset(var.server_names)
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = each.key
  }
}
```

This creates instances:

```
aws_instance.web["app1"]
aws_instance.web["app2"]
aws_instance.web["app3"]
```

### 🧱 Example 2: Using a map

```hcl
variable "servers" {
  default = {
    dev  = "t2.micro"
    prod = "t3.medium"
  }
}

resource "aws_instance" "web" {
  for_each = var.servers
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value

  tags = {
    Name = each.key
  }
}
```

Creates:

```
aws_instance.web["dev"]
aws_instance.web["prod"]
```

---

### 💡 Accessing with `each.key` and `each.value`

* `each.key` → current key (like “dev”)
* `each.value` → current value (like “t2.micro”)

---

### ⚠️ Advantages over `count`

| Feature            | `count`             | `for_each`                |
| ------------------ | ------------------- | ------------------------- |
| Input type         | Integer             | Map or Set                |
| Identifier         | `count.index`       | `each.key` / `each.value` |
| Resource reference | Numeric index       | Named key                 |
| Stability          | Index order matters | Stable — key-based        |
| Readability        | Simple for numbers  | Better for unique items   |

✅ **Use `count`** → when identical resources (e.g., 3 EC2 instances)
✅ **Use `for_each`** → when unique configurations or keys (e.g., prod/dev/test)

---

## 🧩 3️⃣ Terraform `for` Expressions

### ✅ Definition:

`for` expressions are like loops inside **variables**, **locals**, or **outputs** to transform lists/maps.

---

### 🧱 Example 1: Transforming a list

```hcl
variable "servers" {
  default = ["dev", "stage", "prod"]
}

locals {
  server_names = [for name in var.servers : upper(name)]
}

output "servers_uppercase" {
  value = local.server_names
}
```

✅ Output:

```
["DEV", "STAGE", "PROD"]
```

---

### 🧱 Example 2: Transforming a map

```hcl
variable "instances" {
  default = {
    dev  = "t2.micro"
    prod = "t3.medium"
  }
}

locals {
  instance_tags = {
    for name, type in var.instances :
    name => "Environment: ${name}, Type: ${type}"
  }
}

output "tags" {
  value = local.instance_tags
}
```

✅ Output:

```
{
  dev  = "Environment: dev, Type: t2.micro"
  prod = "Environment: prod, Type: t3.medium"
}
```

---

### 🧱 Example 3: With Conditional Filtering

```hcl
locals {
  large_instances = {
    for name, type in var.instances : name => type
    if type == "t3.medium"
  }
}
```

✅ Filters only items where instance type = `t3.medium`.

---

## 🧩 Combining them together (real-time example)

Let’s use all three in a real-world setup 👇

**Use Case:** Deploy EC2 instances based on environment data.

```hcl
variable "environments" {
  default = {
    dev  = "t2.micro"
    qa   = "t2.small"
    prod = "t3.medium"
  }
}

locals {
  env_tags = [for env, type in var.environments : "${env}-${type}"]
}

resource "aws_instance" "servers" {
  for_each = var.environments

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = each.value

  tags = {
    Name = "server-${each.key}"
  }
}

output "instance_ids" {
  value = [for key, inst in aws_instance.servers : inst.id]
}

output "env_tags" {
  value = local.env_tags
}
```

✅ Terraform creates EC2s per environment and outputs instance IDs dynamically.

---

## ⚠️ Common Pitfalls

| Problem                                | Root Cause                     | Fix                                    |
| -------------------------------------- | ------------------------------ | -------------------------------------- |
| Error: “Invalid for_each argument”     | List passed instead of map/set | Convert using `toset()` or use `count` |
| Resource replaced on reorder           | Using `count` with list        | Use `for_each` (key-based stability)   |
| Cannot use both `count` and `for_each` | Terraform doesn’t allow both   | Choose one per resource                |
| Duplicate keys                         | Same key in map                | Keys in `for_each` must be unique      |

---

## 🧠 Interview-Focused Q&A

| Question                                                     | Answer                                                                |
| ------------------------------------------------------------ | --------------------------------------------------------------------- |
| What’s the difference between `count` and `for_each`?        | `count` = numeric repetition; `for_each` = key/value-based iteration. |
| Can we use both `count` and `for_each` in the same resource? | No, only one allowed.                                                 |
| When to use `for_each` over `count`?                         | When you want stable, key-based creation (e.g., map or set).          |
| What’s a `for` expression in Terraform?                      | A loop-like syntax for transforming or filtering lists/maps.          |
| How to prevent recreation when list order changes?           | Use `for_each` instead of `count`.                                    |

---
