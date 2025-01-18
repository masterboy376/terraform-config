# Terraform Setup and Usage

## Overview

This repository contains Terraform configurations for managing infrastructure as code. It includes instructions for setting up Terraform, running it locally, and using Docker to run Terraform in a consistent environment.

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) (v1.0.0 or later)
- [Docker](https://www.docker.com/get-started)

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

### Docker Setup

1. **Install Docker**:
    Follow the instructions for your operating system on the [Docker website](https://www.docker.com/get-started).

2. **Verify Installation**:
    ```sh
    docker --version
    ```

## Using Terraform Locally

1. **Initialize Terraform**:
    ```sh
    terraform init
    ```

2. **Plan the Infrastructure**:
    ```sh
    terraform plan
    ```

3. **Apply the Configuration**:
    ```sh
    terraform apply
    ```

4. **Destroy the Infrastructure**:
    ```sh
    terraform destroy
    ```

## Running Terraform in a Docker Container

### Create a Dockerfile

1. **Create a `Dockerfile`**:
    ```Dockerfile
    # Use a base image with Terraform installed
    FROM hashicorp/terraform:latest

    # Set the working directory
    WORKDIR /workspace

    # Copy your Terraform configuration files into the container
    COPY . .

    # Run Terraform commands when the container starts
    CMD ["terraform", "apply"]
    ```

### Build and Run the Docker Image

1. **Build the Docker Image**:
    ```sh
    docker build -t my-terraform-image .
    ```

2. **Run the Docker Container**:
    ```sh
    docker run -it --name my-terraform-container my-terraform-image
    ```

### Execute Terraform Commands in Docker

1. **Initialize Terraform**:
    ```sh
    docker run -it --rm -v $(pwd):/workspace my-terraform-container terraform init
    ```

2. **Plan the Infrastructure**:
    ```sh
    docker run -it --rm -v $(pwd):/workspace my-terraform-container terraform plan
    ```

3. **Apply the Configuration**:
    ```sh
    docker run -it --rm -v $(pwd):/workspace my-terraform-container terraform apply
    ```

4. **Destroy the Infrastructure**:
    ```sh
    docker run -it --rm -v $(pwd):/workspace my-terraform-container terraform destroy
    ```

## Directory Structure

```plaintext
terraform-project/
├── main.tf
├── variables.tf
├── outputs.tf
├── Dockerfile
└── README.md
```

## Reference Notes

### Common Commands
- `terraform init` - Initialize a Terraform configuration.
- `terraform plan` - Generate and show an execution plan.
- `terraform apply` - Build or change infrastructure.
- `terraform destroy` - Destroy Terraform-managed infrastructure.
- `terraform fmt` - Reformat your configuration files.
- `terraform validate` - Check whether the configuration is valid.

### Variables and Outputs
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
  description = "The ID of the instance"
  value       = aws_instance.example.id
}
```
---
