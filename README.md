# aws-terraform-vpc-module

Reusable Terraform module for provisioning a multi-Availability Zone (multi-AZ) AWS Virtual Private Cloud (VPC) with isolated public and private subnet tiers and outbound internet routing.

## Architecture

* **VPC (`aws_vpc`)**: Configurable IPv4 CIDR block (default `10.0.0.0/16`) with DNS resolution and DNS hostnames enabled.
* **Public Subnets (`aws_subnet`)**: Distributed across multiple Availability Zones (`us-east-1a`, `us-east-1b`), configured with auto-assigned public IPv4 addresses on launch and routed to the Internet Gateway.
* **Private Subnets (`aws_subnet`)**: Distributed across the same Availability Zones without public IP assignment or direct internet routes, isolating internal workloads.
* **Internet Gateway & Routing (`aws_internet_gateway`, `aws_route_table`, `aws_route_table_association`)**: Attaches an Internet Gateway to the VPC and associates only the public subnets with a route table directing `0.0.0.0/0` traffic to the gateway.

## Repository Structure

```text
.
├── main.tf                  # Root module configuration
├── variables.tf             # Root input variables
├── outputs.tf               # Root outputs
├── versions.tf              # Terraform and AWS provider version constraints
├── .terraform.lock.hcl      # Provider dependency lockfile
├── .gitignore               # Excludes state files, .terraform/, and local *.tfvars
└── modules/
    └── vpc/
        ├── main.tf          # VPC, subnets, IGW, and route table resources
        ├── variables.tf     # Child module input variables
        └── outputs.tf       # Child module resource IDs
```

## Usage

```hcl
module "vpc" {
  source = "./modules/vpc"

  vpc_cidr             = "10.0.0.0/16"
  azs                  = ["us-east-1a", "us-east-1b"]
  public_subnet_cidrs  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnet_cidrs = ["10.0.10.0/27", "10.0.10.32/27"]

  tags = {
    Name        = "prod-core"
    Environment = "prod"
    ManagedBy   = "terraform"
  }
}
```

## Inputs

| Name | Description | Type | Required |
| :--- | :--- | :--- | :--- |
| `vpc_cidr` | IPv4 CIDR block for the VPC | `string` | Yes |
| `azs` | List of Availability Zones for subnet distribution | `list(string)` | Yes |
| `public_subnet_cidrs` | List of CIDR blocks for public subnets | `list(string)` | Yes |
| `private_subnet_cidrs` | List of CIDR blocks for private subnets | `list(string)` | Yes |
| `tags` | Map of tags applied to VPC resources (falls back safely if `Name` is omitted) | `map(string)` | Yes |

## Outputs

| Name | Description |
| :--- | :--- |
| `vpc_id` | ID of the created VPC |
| `public_subnet_ids` | List of IDs for the public subnets |
| `private_subnet_ids` | List of IDs for the private subnets |

## Quick Start

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
```