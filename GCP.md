# GCP Infrastructure with Terraform

## Overview

This file contains Terraform configurations for infrastructure on GCP. It includes Virtual Machines (VMs), Blob Storage, Virtual Networks, and SQL Databases.

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) (v1.0.0 or later)
- [GCP SDK](https://cloud.google.com/sdk/docs)

## Why Use CLI Tools Alongside Terraform?

### Initial Setup and Configuration
Before Terraform can manage resources, you often need to authenticate and configure access using CLI tools. For example, setting up GCP SDK with `gcloud auth login` stores credentials that Terraform uses.

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
    gcloud auth login
    gcloud config set project YOUR_PROJECT_ID
    ```

- **Manual Resource Inspection**:
    ```sh
    gcloud compute instances describe INSTANCE_NAME
    ```

- **Running Supplemental Commands**:
    ```sh
    gsutil cp file.txt gs://my-bucket/
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

### GCP Setup

1. **Authenticate with GCP**:
    ```sh
    gcloud auth login
    gcloud config set project YOUR_PROJECT_ID
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
    google = {
      source  = "hashicorp/google"
      version = "4.0.0"
    }
  }
}

provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "us-central1"
}

variable "environment" {
  description = "The environment for the resources"
  type        = string
  default     = "development"
}

resource "google_compute_network" "vpc_network" {
  name                    = "vpc-network"
  auto_create_subnetworks = false
  tags = {
    Environment = var.environment
  }
}

resource "google_compute_subnetwork" "public_subnet" {
  name          = "public-subnet"
  network       = google_compute_network.vpc_network.id
  ip_cidr_range = "10.0.1.0/24"
  region        = "us-central1"
  tags = {
    Environment = var.environment
  }
}

resource "google_compute_subnetwork" "private_subnet" {
  name          = "private-subnet"
  network       = google_compute_network.vpc_network.id
  ip_cidr_range = "10.0.2.0/24"
  region        = "us-central1"
  tags = {
    Environment = var.environment
  }
}

resource "google_compute_instance" "vm_instance" {
  name         = "vm-instance"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }
  network_interface {
    subnetwork = google_compute_subnetwork.public_subnet.id
  }
  tags = {
    Environment = var.environment
  }
}

resource "google_storage_bucket" "bucket" {
  name          = "example-bucket-2025" # Replace with a unique bucket name
  location      = "US"
  force_destroy = true
  versioning {
    enabled = true
  }
  tags = {
    Environment = var.environment
  }
}

resource "google_sql_database_instance" "default" {
  name             = "example-db-instance"
  database_version = "MYSQL_8_0"
  settings {
    tier = "db-f1-micro"
  }
  tags = {
    Environment = var.environment
  }
}

resource "google_sql_database" "database" {
  name     = "exampledb"
  instance = google_sql_database_instance.default.name
  tags = {
    Environment = var.environment
  }
}

resource "google_sql_user" "users" {
  name     = "admin"
  instance = google_sql_database_instance.default.name
  password = "password" # Replace with a secure password
  tags = {
    Environment = var.environment
  }
}

# Backend Configuration for State Management
terraform {
  backend "gcs" {
    bucket  = "your-terraform-state-bucket"
    prefix  = "terraform/state"
  }
}
```

## Directory Structure

```plaintext
gcp-terraform/
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
  default     = "n1-standard-1"
}
```

Output values in `outputs.tf`:
```hcl
output "instance_id" {
  description = "The ID of the Compute Engine instance"
  value       = google_compute_instance.vm_instance.id
}
```
