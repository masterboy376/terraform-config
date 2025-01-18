#### AWS Infrastructure with Terraform

## Overview

This repository contains Terraform configurations for infrastructure on AWS. It includes EC2 instances, S3 buckets, VPCs, and RDS databases.

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) (v1.0.0 or later)
- [AWS CLI](https://aws.amazon.com/cli/)

## Why Use CLI Tools Alongside Terraform?

### Initial Setup and Configuration
Before Terraform can manage resources, you often need to authenticate and configure access using CLI tools. For example, setting up AWS CLI with `aws configure` stores credentials that Terraform uses.

### Manual Management and Debugging
While Terraform is great for automated, declarative infrastructure management, sometimes you might need to manually inspect, manage, or debug resources directly. CLI tools provide commands to query, modify, and troubleshoot resources interactively.

### Service-Specific Features
Some cloud provider features or configurations might not be fully supported by Terraform, especially new or niche services. CLI tools often have more comprehensive and immediate access to all provider APIs.

### Integration and Scripting
CLI tools can be used in scripts and automation outside of Terraform. For example, you might use CLI commands in a CI/CD pipeline to manage resources, handle deployments, or collect logs.

### Supplemental Tasks
CLI tools can perform tasks that are tangential but necessary to infrastructure management, such as uploading files to cloud storage, managing IAM policies, or setting environment variables.

#### Example Scenarios
- **Authenticating with CLI**:
    ```sh
    aws configure
    ```

- **Manual Resource Inspection**:
    ```sh
    aws ec2 describe-instances --instance-ids i-0123456789abcdef0
    ```

- **Running Supplemental Commands**:
    ```sh
    aws s3 cp file.txt s3://my-bucket/
    ```

## Installation and Setup

### Terraform Installation

1. **Download Terraform**:
    ```sh
    curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
    sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
    sudo apt-get update && sudo apt-get install terraform
    ```

2. **Verify Installation**:
    ```sh
    terraform version
    ```

### AWS Setup

1. **Configure AWS CLI**:
    ```sh
    aws configure
    ```

2. **Initialize Terraform**:
    ```sh
    terraform init
    ```

### Configuration

**main.tf**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.0.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "environment" {
  description = "The environment for the resources"
  type        = string
  default     = "development"
}

resource "aws_vpc" "example" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name        = "example-vpc"
    Environment = var.environment
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.example.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"
  tags = {
    Name        = "example-public-subnet"
    Environment = var.environment
  }
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.example.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name        = "example-private-subnet"
    Environment = var.environment
  }
}

resource "aws_instance" "example" {
  ami           = "ami-0abcdef1234567890" # Replace with a valid AMI ID
  instance_type = "t2.micro"
  key_name      = "your-key-pair" # Replace with your key pair name
  subnet_id     = aws_subnet.public.id
  tags = {
    Name        = "example-instance"
    Environment = var.environment
  }
}

resource "aws_s3_bucket" "example" {
  bucket    = "example-bucket-2025" # Replace with a unique bucket name
  acl       = "private"
  versioning {
    enabled = true
  }
  tags = {
    Name        = "example-s3-bucket"
    Environment = var.environment
  }
}

resource "aws_db_subnet_group" "example" {
  name       = "example-db-subnet-group"
  subnet_ids = [
    aws_subnet.public.id,
    aws_subnet.private.id
  ]
  tags = {
    Name        = "example-db-subnet-group"
    Environment = var.environment
  }
}

resource "aws_db_instance" "example" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  name                 = "exampledb"
  username             = "admin"
  password             = "password" # Replace with a secure password
  db_subnet_group_name = aws_db_subnet_group.example.name
  skip_final_snapshot  = true
  tags = {
    Name        = "example-db-instance"
    Environment = var.environment
  }
}

# Backend Configuration for State Management
terraform {
  backend "s3" {
    bucket = "your-terraform-state-bucket"
    key    = "path/to/terraform.tfstate"
    region = "us-east-1"
  }
}
```

## Directory Structure

```plaintext
aws-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
├── backend.tf
└── README.md
```

### Reference Notes

#### Common Commands
- `terraform init` - Initialize a Terraform configuration.
- `terraform plan` - Generate and show an execution plan.
- `terraform apply` - Build or change infrastructure.
- `terraform destroy` - Destroy Terraform-managed infrastructure.
- `terraform fmt` - Reformat your configuration files.
- `terraform validate` - Check whether the configuration is valid.

#### Variables and Outputs
Define variables in `variables.tf`:
```hcl
variable "instance_type" {
  description = "Type of instance to create"
  type        = string
  default     = "t2.micro"
}
```

Output values in `outputs.tf`:
```hcl
output "instance_id" {
  description = "The ID of the EC2 instance"
  value       = aws_instance.example.id
}
```
---
   
