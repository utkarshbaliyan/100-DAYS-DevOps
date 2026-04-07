# 🏗️ Terraform Complete Study Notes

> **Purpose:** Comprehensive Terraform notes for interview preparation and job applications.  
> **Author:** Self-study notes  
> **Level:** Beginner to Intermediate

---

## Table of Contents

1. [What is Terraform?](#1-what-is-terraform)
2. [What is Infrastructure as Code (IaC)?](#2-what-is-infrastructure-as-code-iac)
3. [Terraform vs Ansible](#3-terraform-vs-ansible)
4. [HCL — HashiCorp Configuration Language](#4-hcl--hashicorp-configuration-language)
5. [Terraform Lifecycle](#5-terraform-lifecycle)
6. [Core Commands: validate → plan → apply](#6-core-commands-validate--plan--apply)
7. [Connecting Terraform to AWS CLI](#7-connecting-terraform-to-aws-cli)
8. [Creating AWS Resources with Terraform](#8-creating-aws-resources-with-terraform)
9. [Terraform Commands Reference](#9-terraform-commands-reference)
10. [Terraform Workflow](#10-terraform-workflow)
11. [Best Practices for .tf Files](#11-best-practices-for-tf-files)
12. [State Management in Terraform](#12-state-management-in-terraform)
13. [State File](#13-state-file)
14. [State Locking & Remote Backend](#14-state-locking--remote-backend)
15. [S3 + DynamoDB Backend Setup](#15-s3--dynamodb-backend-setup)
16. [Variables in Terraform](#16-variables-in-terraform)
17. [Terraform Output](#17-terraform-output)
18. [Terraform Import](#18-terraform-import)
19. [Ternary Operator in Terraform](#19-ternary-operator-in-terraform)
20. [Terraform Modules](#20-terraform-modules)
21. [Terraform Workspaces](#21-terraform-workspaces)
22. [Additional Important Concepts](#22-additional-important-concepts)

---

## 1. What is Terraform?

**Terraform** is an open-source **Infrastructure as Code (IaC)** tool developed by **HashiCorp**. It allows you to define, provision, and manage infrastructure (servers, databases, networks, DNS, etc.) using a declarative configuration language called **HCL (HashiCorp Configuration Language)**.

### Key Characteristics

- **Declarative:** You describe *what* you want, not *how* to get there. Terraform figures out the steps.
- **Provider-Agnostic:** Works with AWS, Azure, GCP, Kubernetes, GitHub, Datadog, and 1000+ providers.
- **State-Driven:** Terraform keeps track of your infrastructure in a **state file** so it knows what exists and what needs to change.
- **Idempotent:** Running `terraform apply` multiple times produces the same result — it only changes what's different.

### Why Terraform?

| Problem Without Terraform | How Terraform Solves It |
|--------------------------|------------------------|
| Manual clicking in AWS Console | Automated, repeatable deployments |
| No audit trail of changes | Code-reviewed infrastructure changes via Git |
| Hard to replicate environments | Same code creates dev/staging/prod |
| Team conflicts on who changed what | State locking prevents simultaneous changes |

---

## 2. What is Infrastructure as Code (IaC)?

**Infrastructure as Code (IaC)** is the practice of managing and provisioning infrastructure through **machine-readable configuration files** rather than manual processes or interactive configuration tools.

### The Old Way (ClickOps)
- Log into AWS Console → manually create EC2, VPC, RDS, etc.
- No version control, no repeatability, human error-prone.

### The IaC Way
- Write code → commit to Git → run tool → infrastructure created automatically.

### Benefits of IaC

| Benefit | Description |
|---------|-------------|
| **Version Control** | Track every change in Git, roll back anytime |
| **Consistency** | No configuration drift between environments |
| **Speed** | Provision 100 servers in minutes |
| **Collaboration** | Teams review infrastructure like application code |
| **Documentation** | The code *is* the documentation |
| **Disaster Recovery** | Rebuild entire infrastructure from code after failure |

### Types of IaC Approaches

- **Declarative:** You define the desired end-state. The tool handles the "how". *(Terraform, CloudFormation)*
- **Imperative/Procedural:** You define step-by-step instructions. *(Ansible playbooks, shell scripts)*

---

## 3. Terraform vs Ansible

Both are popular IaC tools, but they serve **different purposes**.

| Feature | Terraform | Ansible |
|---------|-----------|---------|
| **Primary Purpose** | Infrastructure Provisioning | Configuration Management |
| **Language** | HCL (Declarative) | YAML (Procedural/Declarative hybrid) |
| **State Management** | Yes — has a state file | No — stateless |
| **Agent Required** | No (agentless) | No (agentless, uses SSH) |
| **Cloud Support** | Excellent (1000+ providers) | Good, but secondary |
| **Best For** | Creating VMs, networks, databases | Installing software, configuring OS |
| **Idempotency** | Built-in by default | Mostly, but requires care |
| **Community** | Large, growing | Very large, mature |

### Rule of Thumb: Use Both Together
1. **Terraform** to provision the infrastructure (create EC2 instance, VPC, RDS).
2. **Ansible** to configure what's on that infrastructure (install Nginx, configure firewall).

---

## 4. HCL — HashiCorp Configuration Language

HCL is the language used to write Terraform configuration files (`.tf` files).

### Core Concepts: Blocks, Parameters, Arguments

#### Block
A **block** is a container that defines a configuration object. It has a **type**, optional **labels**, and a **body**.

```hcl
# Syntax:
<BLOCK_TYPE> "<LABEL_1>" "<LABEL_2>" {
  # body (arguments go here)
}
```

#### Common Block Types

```hcl
# terraform block — configures Terraform itself
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# provider block — configures the cloud provider
provider "aws" {
  region = "us-east-1"
}

# resource block — defines infrastructure to create
resource "aws_instance" "my_server" {  # "aws_instance" = type, "my_server" = name
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

# variable block — declares input variables
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

# output block — declares output values
output "instance_ip" {
  value = aws_instance.my_server.public_ip
}

# data block — reads existing resources (not created by Terraform)
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
}

# locals block — defines local values (computed expressions)
locals {
  env    = "production"
  prefix = "myapp-${local.env}"
}

# module block — calls a module
module "vpc" {
  source = "./modules/vpc"
  cidr   = "10.0.0.0/16"
}
```

#### Arguments vs Parameters

- **Argument:** A key-value pair inside a block body. `instance_type = "t2.micro"` is an argument.
- **Parameter:** The name/key part of the argument (the "slot" the provider defines). `instance_type` is the parameter name.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-abc123"    # argument: parameter=ami, value="ami-abc123"
  instance_type = "t2.micro"     # argument: parameter=instance_type, value="t2.micro"
  
  tags = {                        # argument with map value
    Name = "MyServer"
  }
}
```

---

## 5. Terraform Lifecycle

The **Terraform resource lifecycle** controls how resources are created, updated, and destroyed.

### Resource Lifecycle Meta-Argument

```hcl
resource "aws_instance" "example" {
  ami           = "ami-abc123"
  instance_type = "t2.micro"

  lifecycle {
    create_before_destroy = true   # Create new resource before destroying old one
    prevent_destroy       = true   # Block any attempt to destroy this resource
    ignore_changes        = [tags] # Ignore changes to specific attributes
    replace_triggered_by  = [aws_security_group.example] # Force replace when dependency changes
  }
}
```

### Lifecycle Options Explained

| Option | What It Does | When to Use |
|--------|-------------|-------------|
| `create_before_destroy` | Creates replacement before deleting old | Zero-downtime updates |
| `prevent_destroy` | Errors out if you try to destroy | Production databases, critical infra |
| `ignore_changes` | Ignores drift on listed attributes | When external systems modify resources |
| `replace_triggered_by` | Forces replacement when another resource changes | Chain dependencies |

### Resource Lifecycle Phases

```
Write Config → terraform init → terraform validate → terraform plan → terraform apply → terraform destroy
                    ↓                  ↓                   ↓               ↓
             Downloads providers   Checks syntax     Shows changes    Makes changes
```

---

## 6. Core Commands: validate → plan → apply

### `terraform validate`

Checks your `.tf` files for **syntax errors and configuration mistakes** — does NOT connect to AWS.

```bash
terraform validate

# Output if valid:
# Success! The configuration is valid.

# Output if invalid:
# Error: Argument or block definition required
```

### `terraform plan`

Connects to your cloud provider, compares desired state (your code) with current state (state file + real infra), and shows you a **preview of changes** — does NOT make any changes.

```bash
terraform plan

# Output symbols:
# + create   (green) — resource will be created
# - destroy  (red)   — resource will be destroyed
# ~ update   (yellow)— resource will be modified in-place
# -/+ replace        — resource will be destroyed and recreated
```

```bash
# Save plan to a file (good practice in CI/CD)
terraform plan -out=tfplan

# Apply only that saved plan
terraform apply tfplan
```

### `terraform apply`

**Actually creates/modifies/destroys** infrastructure. Shows the plan again and asks for confirmation.

```bash
terraform apply

# Skip confirmation prompt (for automation/CI-CD):
terraform apply -auto-approve
```

---

## 7. Connecting Terraform to AWS CLI

### Step 1: Install AWS CLI

```bash
# macOS
brew install awscli

# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# Verify
aws --version
```

### Step 2: Configure AWS Credentials

```bash
aws configure
# AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
# AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
# Default region name [None]: us-east-1
# Default output format [None]: json
```

This creates `~/.aws/credentials` and `~/.aws/config`.

### Step 3: Configure Terraform Provider

```hcl
# provider.tf
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
  # Terraform automatically reads credentials from ~/.aws/credentials
  # OR from environment variables: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY
}
```

### Alternative: Environment Variables (more secure)

```bash
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_DEFAULT_REGION="us-east-1"
```

### Alternative: IAM Roles (best for production/EC2)

When running Terraform on an EC2 instance, attach an **IAM Role** to the instance. Terraform will automatically use the role's credentials — no access keys needed.

---

## 8. Creating AWS Resources with Terraform

### Project Folder Structure (Best Practice)

```
my-terraform-project/
├── main.tf          # Main resources
├── provider.tf      # Provider configuration
├── variables.tf     # Input variable declarations
├── outputs.tf       # Output declarations
├── terraform.tfvars # Variable values (DO NOT commit to Git if sensitive)
└── backend.tf       # Remote backend configuration
```

### Example: Create an EC2 Instance

```hcl
# provider.tf
provider "aws" {
  region = var.aws_region
}

# variables.tf
variable "aws_region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t2.micro"
}

# main.tf
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  tags = { Name = "main-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  tags = { Name = "public-subnet" }
}

resource "aws_security_group" "allow_ssh" {
  name   = "allow_ssh"
  vpc_id = aws_vpc.main.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.allow_ssh.id]

  tags = {
    Name        = "web-server"
    Environment = "dev"
  }
}

# outputs.tf
output "instance_id" {
  value = aws_instance.web.id
}

output "instance_public_ip" {
  value = aws_instance.web.public_ip
}
```

### Reference Syntax for Resources

```hcl
# Syntax: <resource_type>.<resource_name>.<attribute>
aws_instance.web.id
aws_instance.web.public_ip
aws_vpc.main.id
aws_subnet.public.id
```

---

## 9. Terraform Commands Reference

```bash
# SETUP
terraform init                    # Initialize project, download providers & modules
terraform init -upgrade           # Upgrade providers to latest allowed version

# PLANNING
terraform validate                # Validate configuration syntax
terraform fmt                     # Format .tf files (auto-indent and style)
terraform fmt -check              # Check formatting without making changes
terraform plan                    # Preview changes
terraform plan -out=tfplan        # Save plan to file
terraform plan -var="key=value"   # Override a variable

# APPLYING
terraform apply                   # Apply changes (with confirmation prompt)
terraform apply -auto-approve     # Apply without prompt
terraform apply tfplan            # Apply a saved plan file
terraform apply -target=resource  # Apply only a specific resource

# DESTROYING
terraform destroy                 # Destroy all managed infrastructure
terraform destroy -auto-approve   # Destroy without confirmation
terraform destroy -target=aws_instance.web  # Destroy only specific resource

# STATE
terraform show                    # Show current state in human-readable format
terraform state list              # List all resources in state
terraform state show aws_instance.web  # Show details of a specific resource
terraform state rm aws_instance.web    # Remove resource from state (doesn't delete real resource)
terraform state mv                # Move resource in state (rename)
terraform refresh                 # Sync state with real infrastructure

# WORKSPACES
terraform workspace list          # List all workspaces
terraform workspace new dev       # Create new workspace
terraform workspace select dev    # Switch to workspace
terraform workspace show          # Show current workspace
terraform workspace delete dev    # Delete workspace

# IMPORT
terraform import aws_instance.web i-1234567890abcdef0  # Import existing resource

# MISC
terraform output                  # Show all outputs
terraform output instance_ip      # Show specific output
terraform graph                   # Generate dependency graph (use with Graphviz)
terraform providers                # Show providers used
terraform version                 # Show Terraform version
```

---

## 10. Terraform Workflow

### Standard Development Workflow

```
1. Write/Edit .tf files
        ↓
2. terraform init       (first time or when providers change)
        ↓
3. terraform fmt        (format code)
        ↓
4. terraform validate   (check syntax)
        ↓
5. terraform plan       (review changes)
        ↓
6. terraform apply      (make changes)
        ↓
7. Commit code to Git
```

### Team/CI-CD Workflow

```
Developer writes code
        ↓
Push to Git branch → Open Pull Request
        ↓
CI pipeline runs: terraform fmt + validate + plan
        ↓
Team reviews plan output in PR
        ↓
PR merged to main
        ↓
CD pipeline runs: terraform apply (on main branch)
```

---

## 11. Best Practices for .tf Files

### 1. File Organization

```
project/
├── main.tf           # Core resources
├── variables.tf      # All variable declarations
├── outputs.tf        # All output declarations
├── provider.tf       # Provider + terraform block
├── backend.tf        # Remote state backend
├── data.tf           # Data sources
└── locals.tf         # Local values
```

### 2. Always Pin Provider Versions

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"   # ✅ Pinned — allows 5.x but not 6.0
    }
  }
  required_version = ">= 1.5.0"
}
```

### 3. Use Variables, Not Hard-coded Values

```hcl
# ❌ Bad
resource "aws_instance" "web" {
  instance_type = "t2.micro"
  ami           = "ami-0c55b159cbfafe1f0"
}

# ✅ Good
resource "aws_instance" "web" {
  instance_type = var.instance_type
  ami           = var.ami_id
}
```

### 4. Use `terraform.tfvars` for Variable Values

```hcl
# variables.tf — declare variables
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
}

# terraform.tfvars — assign values (gitignore if it has secrets)
instance_type = "t2.micro"
```

### 5. Tag All Resources

```hcl
locals {
  common_tags = {
    Project     = "MyApp"
    Environment = var.environment
    ManagedBy   = "Terraform"
    Owner       = "platform-team"
  }
}

resource "aws_instance" "web" {
  # ...
  tags = merge(local.common_tags, { Name = "web-server" })
}
```

### 6. Use Locals to Avoid Repetition

```hcl
locals {
  name_prefix = "${var.project}-${var.environment}"
}

resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data"
}

resource "aws_s3_bucket" "logs" {
  bucket = "${local.name_prefix}-logs"
}
```

### 7. Use `.gitignore` for Terraform

```gitignore
# .gitignore
.terraform/           # Provider binaries (auto-downloaded)
.terraform.lock.hcl   # Can be committed (lock file)
*.tfstate             # State files (use remote state instead)
*.tfstate.backup
*.tfplan              # Plan files
terraform.tfvars      # If it contains secrets
*.auto.tfvars         # If it contains secrets
```

### 8. Use `prevent_destroy` for Critical Resources

```hcl
resource "aws_db_instance" "production_db" {
  # ...
  lifecycle {
    prevent_destroy = true
  }
}
```

---

## 12. State Management in Terraform

**State management** is how Terraform tracks what it has created and manages changes over time.

### Why State Exists

Terraform needs to:
1. Know **what it already created** (avoid creating duplicates)
2. **Map your config** to real-world resources
3. Track **metadata** (like resource dependencies)
4. Improve **performance** (cache attributes instead of querying API every time)

---

## 13. State File

The **state file** (`terraform.tfstate`) is a JSON file that stores the current state of your managed infrastructure.

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "resources": [
    {
      "type": "aws_instance",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "id": "i-1234567890abcdef0",
            "ami": "ami-0c55b159cbfafe1f0",
            "instance_type": "t2.micro",
            "public_ip": "54.123.45.67"
          }
        }
      ]
    }
  ]
}
```

### Important State File Rules

- **Never edit the state file manually** — use `terraform state` commands.
- **Never commit state files to Git** — they may contain secrets (passwords, keys).
- **Use remote state** for teams — keeps everyone on the same version.
- State file is the **source of truth** for Terraform.

---

## 14. State Locking & Remote Backend

### What is State Locking?

When one user runs `terraform apply`, Terraform **locks** the state file so no other user can run `apply` simultaneously. This prevents **state corruption** from concurrent modifications.

```
User A: terraform apply → ACQUIRES LOCK → makes changes → RELEASES LOCK
User B: terraform apply → WAITS (lock held by A) → ERROR or waits → proceeds after lock released
```

### What is a Remote Backend?

By default, state is stored **locally** (`terraform.tfstate`). A **remote backend** stores state in a central, shared location.

```hcl
# backend.tf — Example: S3 Remote Backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

### Benefits of Remote Backend

| Feature | Local State | Remote State (S3) |
|---------|-------------|-------------------|
| Team collaboration | ❌ Conflicts | ✅ Shared |
| State locking | ❌ None | ✅ DynamoDB lock |
| Encryption | ❌ Plain text | ✅ S3 encryption |
| Versioning | ❌ No history | ✅ S3 versioning |
| Disaster recovery | ❌ Lost if laptop breaks | ✅ Durable |

---

## 15. S3 + DynamoDB Backend Setup

This is the standard AWS setup for team Terraform state management.

### Step 1: Create the S3 Bucket (for state storage)

```hcl
# Create this infrastructure first (in a separate bootstrap Terraform or manually)

resource "aws_s3_bucket" "terraform_state" {
  bucket = "my-company-terraform-state"

  lifecycle {
    prevent_destroy = true  # Prevent accidental deletion
  }

  tags = {
    Name    = "Terraform State"
    Purpose = "terraform-backend"
  }
}

# Enable versioning — allows rollback to previous state
resource "aws_s3_bucket_versioning" "state_versioning" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "state_encryption" {
  bucket = aws_s3_bucket.terraform_state.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block all public access
resource "aws_s3_bucket_public_access_block" "state_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

### Step 2: Create the DynamoDB Table (for state locking)

```hcl
resource "aws_dynamodb_table" "terraform_lock" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"   # MUST be named "LockID" — Terraform requires this

  attribute {
    name = "LockID"
    type = "S"   # String type
  }

  tags = {
    Name    = "Terraform State Lock"
    Purpose = "terraform-backend"
  }
}
```

### Step 3: Configure Backend in Your Project

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "production/ec2/terraform.tfstate"  # Unique path per project
    region         = "us-east-1"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

### How Locking Works

```
User A runs terraform apply
  → Terraform writes item to DynamoDB: { LockID: "production/ec2/terraform.tfstate", Info: "..." }
  → Applies changes to AWS
  → Deletes DynamoDB item (releases lock)

User B tries terraform apply at same time
  → Terraform tries to write to DynamoDB but item already exists
  → ERROR: "Error acquiring the state lock"
  → User B must wait or use -lock=false (dangerous, avoid)
```

---

## 16. Variables in Terraform

Variables make your Terraform code reusable and environment-agnostic.

### Declaring Variables

```hcl
# variables.tf

# Simple variable with default
variable "environment" {
  description = "Deployment environment"
  type        = string
  default     = "dev"
}

# Variable without default (required — must be provided)
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true   # Redacts value from logs/output
}

# Variable with validation
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"

  validation {
    condition     = contains(["t2.micro", "t2.small", "t3.micro"], var.instance_type)
    error_message = "Instance type must be t2.micro, t2.small, or t3.micro."
  }
}

# List variable
variable "availability_zones" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

# Map variable
variable "tags" {
  type = map(string)
  default = {
    Project = "MyApp"
    Team    = "Platform"
  }
}

# Object variable (structured)
variable "db_config" {
  type = object({
    instance_class    = string
    allocated_storage = number
    multi_az          = bool
  })
  default = {
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    multi_az          = false
  }
}
```

### Providing Variable Values

**Priority order (highest to lowest):**

1. `-var` flag on command line
2. `-var-file` flag on command line
3. `*.auto.tfvars` files (auto-loaded)
4. `terraform.tfvars` file (auto-loaded)
5. Environment variables (`TF_VAR_<name>`)
6. Default value in declaration
7. Interactive prompt (if no default)

```bash
# 1. Command line flag
terraform apply -var="environment=production"

# 2. Var file
terraform apply -var-file="production.tfvars"

# 3. Environment variable
export TF_VAR_db_password="supersecretpassword"
terraform apply
```

```hcl
# terraform.tfvars (auto-loaded)
environment   = "production"
instance_type = "t3.micro"

# production.tfvars (explicit, use with -var-file)
environment = "production"
db_password = "supersecretpassword"
```

### Using Variables

```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
  tags          = var.tags
}

resource "aws_db_instance" "main" {
  instance_class    = var.db_config.instance_class
  allocated_storage = var.db_config.allocated_storage
}
```

---

## 17. Terraform Output

Outputs expose values from your Terraform state — useful for sharing data between modules or displaying results.

### Declaring Outputs

```hcl
# outputs.tf

output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.web.id
}

output "instance_public_ip" {
  description = "Public IP address"
  value       = aws_instance.web.public_ip
}

# Sensitive output (hidden in logs)
output "db_connection_string" {
  description = "Database connection string"
  value       = "postgresql://${aws_db_instance.main.endpoint}/mydb"
  sensitive   = true
}

# Output from a module
output "vpc_id" {
  value = module.vpc.vpc_id
}
```

### Using Outputs

```bash
# After apply, see all outputs
terraform output

# See specific output
terraform output instance_public_ip

# Get raw value (useful for scripts)
terraform output -raw instance_public_ip

# Get JSON (useful for parsing)
terraform output -json
```

### Using Outputs Between Modules

```hcl
# Module A (networking) — defines output
output "vpc_id" {
  value = aws_vpc.main.id
}

# Module B (compute) — uses output from Module A
module "networking" {
  source = "./modules/networking"
}

resource "aws_instance" "web" {
  subnet_id = module.networking.vpc_id
}
```

---

## 18. Terraform Import

`terraform import` brings existing infrastructure (created manually or outside Terraform) under Terraform management.

### Why Use Import?

- You created resources manually in AWS Console and now want to manage them with Terraform.
- You're adopting Terraform for an existing infrastructure.

### How to Import

```bash
# Syntax:
terraform import <resource_type>.<resource_name> <real_resource_id>

# Examples:
terraform import aws_instance.web i-1234567890abcdef0
terraform import aws_s3_bucket.data my-existing-bucket-name
terraform import aws_vpc.main vpc-12345678
```

### Import Workflow

```
Step 1: Write the resource block in your .tf file (just the block, attributes can be incomplete)

resource "aws_instance" "web" {
  # attributes filled in after import
}

Step 2: Run terraform import
terraform import aws_instance.web i-1234567890abcdef0

Step 3: Run terraform plan — it will show what attributes are missing/different
terraform plan

Step 4: Fill in the resource block based on plan output and run apply
terraform apply
```

### New Import Block (Terraform 1.5+)

```hcl
# Import block — declarative, no CLI command needed
import {
  to = aws_instance.web
  id = "i-1234567890abcdef0"
}

resource "aws_instance" "web" {
  # Terraform generates config with: terraform plan -generate-config-out=generated.tf
}
```

---

## 19. Ternary Operator in Terraform

The ternary (conditional) operator lets you choose between two values based on a condition.

### Syntax

```hcl
condition ? value_if_true : value_if_false
```

### Examples

```hcl
# Choose instance type based on environment
variable "environment" { default = "dev" }

resource "aws_instance" "web" {
  instance_type = var.environment == "production" ? "t3.large" : "t2.micro"
}

# Enable multi-AZ for production only
resource "aws_db_instance" "main" {
  multi_az = var.environment == "production" ? true : false
  # Shorthand:
  multi_az = var.environment == "production"
}

# Conditional resource count (create or skip)
resource "aws_eip" "nat" {
  count = var.enable_nat_gateway ? 1 : 0
  vpc   = true
}

# Choose between two values
locals {
  bucket_name = var.environment == "production" ? "myapp-prod-data" : "myapp-dev-data"
  
  # Nested ternary (use sparingly — hurts readability)
  instance_size = var.environment == "production" ? "large" : (var.environment == "staging" ? "medium" : "small")
}
```

---

## 20. Terraform Modules

Modules are **reusable packages of Terraform configuration**. They let you group resources together and reuse them across projects or environments.

### Why Use Modules?

- **DRY (Don't Repeat Yourself):** Write once, use many times.
- **Encapsulation:** Hide complexity behind a clean interface.
- **Standardization:** Enforce best practices across teams.
- **Versioning:** Lock to a specific module version.

### Module Structure

```
modules/
└── ec2-instance/
    ├── main.tf        # Resources
    ├── variables.tf   # Input variables
    ├── outputs.tf     # Output values
    └── README.md      # Documentation
```

### Creating a Module

```hcl
# modules/ec2-instance/variables.tf
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "subnet_id" {
  type = string
}

variable "name" {
  type = string
}

# modules/ec2-instance/main.tf
resource "aws_instance" "this" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  tags          = { Name = var.name }
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-22.04-amd64-server-*"]
  }
}

# modules/ec2-instance/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}

output "public_ip" {
  value = aws_instance.this.public_ip
}
```

### Using a Module

```hcl
# main.tf (in your root project)

# Local module
module "web_server" {
  source        = "./modules/ec2-instance"
  name          = "web-server"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
}

# Use module outputs
output "web_server_ip" {
  value = module.web_server.public_ip
}

# Public registry module (Terraform Registry)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = true
}
```

### Module Sources

```hcl
# Local path
source = "./modules/ec2-instance"

# Terraform Registry
source = "terraform-aws-modules/vpc/aws"

# GitHub
source = "github.com/myorg/terraform-modules//ec2-instance"

# S3 bucket
source = "s3::https://s3.amazonaws.com/my-terraform-modules/ec2.zip"
```

---

## 21. Terraform Workspaces

Workspaces let you manage **multiple environments** (dev, staging, production) from a **single configuration**, each with its own separate state file.

### Default Workspace

Every Terraform project starts in the `default` workspace.

```bash
terraform workspace show
# default
```

### Workspace Commands

```bash
# List all workspaces
terraform workspace list
# * default
#   dev
#   staging
#   production

# Create a new workspace
terraform workspace new dev

# Switch to a workspace
terraform workspace select production

# Show current workspace
terraform workspace show

# Delete a workspace (must not be current, must be empty)
terraform workspace delete dev
```

### Using Workspace Name in Config

```hcl
# Reference current workspace
locals {
  env = terraform.workspace
}

resource "aws_instance" "web" {
  instance_type = terraform.workspace == "production" ? "t3.large" : "t2.micro"
  
  tags = {
    Name        = "web-${terraform.workspace}"
    Environment = terraform.workspace
  }
}

# Use map to select values per workspace
variable "instance_types" {
  type = map(string)
  default = {
    default    = "t2.micro"
    dev        = "t2.micro"
    staging    = "t3.small"
    production = "t3.large"
  }
}

resource "aws_instance" "web" {
  instance_type = var.instance_types[terraform.workspace]
}
```

### State Files Per Workspace

Workspaces store state in separate files automatically:

```
terraform.tfstate.d/
├── dev/
│   └── terraform.tfstate
├── staging/
│   └── terraform.tfstate
└── production/
    └── terraform.tfstate
```

### Workspaces vs. Separate Directories

| Approach | Workspaces | Separate Directories |
|----------|-----------|----------------------|
| State isolation | ✅ Yes | ✅ Yes |
| Config differences | ⚠️ Limited (conditionals) | ✅ Full flexibility |
| Complexity | Low | Higher |
| Best for | Similar envs, few differences | Very different environments |

**Many teams prefer separate directories or separate repos** for production vs dev because of the risk of accidentally applying to the wrong workspace.

---

## 22. Additional Important Concepts

### Data Sources

Fetch information about existing resources (not created by this Terraform project).

```hcl
# Get the latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Use it
resource "aws_instance" "web" {
  ami = data.aws_ami.amazon_linux.id
}

# Get current AWS account ID
data "aws_caller_identity" "current" {}
output "account_id" {
  value = data.aws_caller_identity.current.account_id
}
```

### Count and For Each (Create Multiple Resources)

```hcl
# count — create N copies
resource "aws_instance" "servers" {
  count         = 3
  ami           = "ami-abc123"
  instance_type = "t2.micro"
  tags = {
    Name = "server-${count.index}"  # server-0, server-1, server-2
  }
}

# for_each — create resources from a map or set
resource "aws_iam_user" "team" {
  for_each = toset(["alice", "bob", "charlie"])
  name     = each.value
}

resource "aws_s3_bucket" "envs" {
  for_each = {
    dev  = "us-east-1"
    prod = "us-west-2"
  }
  bucket = "myapp-${each.key}"
  # each.key = "dev" or "prod"
  # each.value = region
}
```

### Depends On

Explicitly declare dependencies (Terraform usually figures them out automatically).

```hcl
resource "aws_instance" "web" {
  depends_on = [aws_internet_gateway.main]
  # ...
}
```

### Dynamic Blocks

Generate repeated nested blocks dynamically.

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    cidr_blocks = list(string)
  }))
  default = [
    { port = 80,  cidr_blocks = ["0.0.0.0/0"] },
    { port = 443, cidr_blocks = ["0.0.0.0/0"] },
    { port = 22,  cidr_blocks = ["10.0.0.0/8"] },
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### Built-in Functions

```hcl
# String functions
upper("hello")         # "HELLO"
lower("HELLO")         # "hello"
format("hello-%s", var.env)  # "hello-dev"
join(", ", ["a","b"])  # "a, b"
split(",", "a,b,c")    # ["a", "b", "c"]
replace("hello", "l", "r")  # "herro"

# Collection functions
length(["a","b","c"])  # 3
merge({a=1}, {b=2})    # {a=1, b=2}
toset(["a","b","a"])   # {"a", "b"} — removes duplicates
flatten([[1,2],[3]])    # [1, 2, 3]
contains(["a","b"], "a")  # true
lookup({a=1, b=2}, "a", 0)  # 1

# Type conversion
tostring(42)           # "42"
tonumber("42")         # 42
tolist(toset([1,2]))   # [1, 2]

# Filesystem
file("./userdata.sh")       # Read file contents
templatefile("./init.sh", { name = var.name })  # Render template

# Encoding
base64encode("hello")        # "aGVsbG8="
jsonencode({ key = "value" })  # '{"key":"value"}'
```

### Provisioners (Use Sparingly)

Execute scripts on resources (last resort — prefer user_data or configuration management tools like Ansible).

```hcl
resource "aws_instance" "web" {
  # ...

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]

    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> inventory.txt"
  }
}
```

### Null Resource

A resource that doesn't create anything in the cloud but can run provisioners/triggers.

```hcl
resource "null_resource" "run_script" {
  triggers = {
    always_run = timestamp()
  }

  provisioner "local-exec" {
    command = "echo 'Running script'"
  }
}
```

### .terraform.lock.hcl

The **dependency lock file** pins exact provider versions.

```hcl
# .terraform.lock.hcl (auto-generated, SHOULD be committed to Git)
provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.31.0"
  constraints = "~> 5.0"
  hashes = [
    "h1:...",
  ]
}
```

---

## Quick Reference Cheat Sheet

```bash
# Daily workflow
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply

# State operations
terraform state list
terraform state show <resource>
terraform state rm <resource>

# Workspaces
terraform workspace list
terraform workspace new <name>
terraform workspace select <name>

# Debug
TF_LOG=DEBUG terraform apply   # Enable verbose logging
terraform refresh               # Sync state with real infra
```

---

## Interview Tips

1. **Explain IaC in simple terms** — "It's treating infrastructure like code — version controlled, reviewable, and repeatable."

2. **Know the difference between plan and apply** — Plan shows what WILL happen, apply makes it happen.

3. **State file is critical** — Know why it exists, why it shouldn't be in Git, and how remote backends solve this.

4. **Modules = DRY principle** — Reusable, encapsulated infrastructure components.

5. **Terraform vs Ansible** — Terraform provisions, Ansible configures.

6. **Common scenario questions:**
   - "How do you manage secrets?" → Environment variables, AWS Secrets Manager, Vault, `sensitive = true`
   - "How do you handle team collaboration?" → Remote state (S3), state locking (DynamoDB), Git branches + PR workflow
   - "What if someone manually changes a resource?" → `terraform plan` shows drift, `terraform apply` corrects it
   - "How do you manage multiple environments?" → Workspaces or separate directories with different tfvars files

---

*Notes compiled for Terraform certification and job interview preparation.*
