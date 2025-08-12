# Infrastructure as Code with Terraform 

Terraform is an open-source Infrastructure as Code (IaC) tool by **HashiCorp**, used to **provision and manage infrastructure** (cloud, platforms, services) in a declarative way.

***

## 1. Core Concepts

### Infrastructure as Code (IaC)
- Resources are defined in **human-readable configuration files** (`.tf`).
- Files can be **versioned**, **reused**, and **shared**.
- Declarative: Define **what** you want; Terraform **figures out how** to create/update/destroy** to match desired state.

### Terraform vs Ansible
| Tool | Purpose |
|---|---|
| **Terraform** | Infrastructure provisioning, orchestration (cloud, VPCs, subnets, servers). |
| **Ansible** | Configuration management & app deployment. |

Often used **together**: Terraform builds infrastructure → Ansible configures it.

### Architecture
- **Core**: Takes Config files + State → compares desired vs current state → builds execution plan.
- **Providers**: Plugins to interact with a platform API (AWS, Azure, Kubernetes, etc.).

***

## 2. Installation & Setup

### Install Terraform (macOS example)
```sh
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform -v
brew update && brew upgrade hashicorp/tap/terraform
```

### Project Structure
```sh
mkdir terraform && cd terraform
touch main.tf
```
- Optional: Install **VS Code Terraform plugin** for syntax highlight & autocomplete.

***

## 3. Providers

- Declared in `.tf` files:
```hcl
provider "aws" {
  region     = "eu-central-1"
}
```
- **Never hardcode credentials** — use:
  - Environment vars: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
  - `~/.aws/credentials` from `aws configure`

### Provider Initialization
```sh
terraform init
```
- Downloads provider into `.terraform/providers/` → `.terraform.lock.hcl` should be committed ([lock versions](https://registry.terraform.io/browse/providers)).

***

## 4. Resources & Data Sources

- **Resources**: Create/update infrastructure.
- **Data sources**: Query existing resources.

Example:
```hcl
resource "aws_vpc" "my-vpc" {
  cidr_block = "10.0.0.0/16"
}
data "aws_vpc" "default" {
  default = true
}
```

Apply:
```sh
terraform apply
```

***

## 5. Change & Destroy

- Updates: `terraform apply` shows `~` for changes, `-` for destroy.
- Destroy resource:
```sh
terraform destroy -target aws_subnet.example
```
> Recommended to change in `.tf` → re-apply to keep code & state in sync.

***

## 6. Key Commands

| Command | Function |
|---|---|
| `terraform plan` | Show changes without applying. |
| `terraform apply --auto-approve` | Apply w/o prompt. |
| `terraform destroy` | Remove all resources in config. |

***

## 7. State Management

- State saved in `terraform.tfstate` (JSON).
- Query:
```sh
terraform state list
terraform state show aws_vpc.my-vpc
```

***

## 8. Output Values

Outputs act like return values:
```hcl
output "vpc_id" {
  value = aws_vpc.my-vpc.id
}
```

***

## 9. Variables

- Declared with:
```hcl
variable "subnet_cidr_block" {
  default = "10.0.10.0/24"
}
```
- Provided via:
  - Prompt
  - CLI: `-var "subnet_cidr_block=..."`  
  - `.tfvars` file
  - Environment: `TF_VAR_avail_zone="eu-central-1a"`

Supports types: `string`, `number`, `list`, `map`, `object`, etc.

***

## 10. Credentials & Env Vars

- **Predefined**: `TF_LOG`, etc.
- **Custom**: `TF_VAR_` matches `variable ""` in `.tf`.

***

## 11. Git Best Practices

- Use separate repo for Terraform code.
- `.gitignore`:
```
**/.terraform/*
*.tfstate
*.tfstate.*
*.tfvars
```

***

## 12. EC2 Provisioning Steps (Example)

### Part 1 — Networking
Provision:
- Custom VPC
- Subnet
- Internet Gateway
- Route Table → Associate with Subnet
- Security Group (SSH, HTTP)

### Part 2 — EC2 Instance
- Get latest AMI dynamically via `data "aws_ami" { ... }`.
- Create Key Pair via Terraform or use existing.
- Associate public IP.
- Tags for naming.

### Part 3 — User Data / Bootstrap
- `user_data` inline or file:
```hcl
user_data = file("entry-script.sh")
```
Install Docker / run container on instance boot.

### Provisioners
- `file`, `remote-exec`, `local-exec`  
> Not recommended for idempotency — prefer config-management tools.

***

## 13. Modules – DRY & Reuse

### Structure
```
modules/
  subnet/
    main.tf
    variables.tf
    outputs.tf
  webserver/
    main.tf
    variables.tf
    outputs.tf
main.tf
variables.tf
outputs.tf
```

Use:
```hcl
module "subnet" {
  source              = "./modules/subnet"
  vpc_id              = aws_vpc.my-vpc.id
  ...
}
```

***

## 14. Provisioning AWS EKS with Terraform

Uses AWS community modules:
- VPC with correct tags for EKS public/private subnets.
- EKS cluster module with managed node groups.

Deploy workflow:
```sh
terraform init
terraform apply
aws eks update-kubeconfig --name myapp-eks-cluster
kubectl get nodes
```

***

## 15. CI/CD Integration with Jenkins

Pipeline stages:
1. **Provision Server**
   - Install Terraform in Jenkins agent container.
   - Run `terraform init && terraform apply`.
   - Capture `terraform output ec2_public_ip` to use in later deploy stages.
2. **Deploy Application**
   - Copy `docker-compose.yaml` + deploy script via `scp`.
   - Run `docker-compose up -d` over SSH.
   - Optionally `docker login` before pulling image.

***

## 16. Remote State

- Use an S3 backend:
```hcl
terraform {
  backend "s3" {
    bucket = "tfstate-bucket"
    key    = "app/state.tfstate"
    region = "eu-central-1"
  }
}
```
- Enables team sharing, versioning, locking (if supported).

***

## 17. Best Practices

### State
- Never manually edit `terraform.tfstate`
- Use remote + locking + versioning.
- Separate state per environment.

### Code
- Keep code in VCS, enforce CI checks.
- Apply Terraform only via pipeline.
- Use underscores `_` for identifiers.
- No hardcoding → use variables, data sources.
- Naming convention + consistent folder structure.

***

### **Key Takeaways**
- Terraform cleanly separates **desired state** from **current state** and reconciles them automatically.
- Use **modules** for reusability and cleaner organization.
- Always keep **state secure and shared** when working in teams.
- Integrates naturally into CI/CD to fully automate provisioning + deployment.
