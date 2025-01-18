# Azure Infrastructure with Terraform

## Overview

This repository contains Terraform configurations to infrastructure on Azure. It includes Virtual Machines (VMs), Blob Storage, Virtual Networks, and SQL Databases.

## Prerequisites

- [Terraform](https://www.terraform.io/downloads.html) (v1.0.0 or later)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

## Why Use CLI Tools Alongside Terraform?

### Initial Setup and Configuration
Before Terraform can manage resources, you often need to authenticate and configure access using CLI tools. For example, setting up Azure CLI with `az login` stores credentials that Terraform uses.

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
    az login
    az account set --subscription YOUR_SUBSCRIPTION_ID
    ```

- **Manual Resource Inspection**:
    ```sh
    az vm show --resource-group RESOURCE_GROUP_NAME --name VM_NAME
    ```

- **Running Supplemental Commands**:
    ```sh
    az storage blob upload --account-name ACCOUNT_NAME --container-name CONTAINER_NAME --name BLOB_NAME --file FILE_PATH
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

### Azure Setup

1. **Authenticate with Azure**:
    ```sh
    az login
    az account set --subscription YOUR_SUBSCRIPTION_ID
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
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "3.0.0"
    }
  }
}

provider "azurerm" {
  features {}
}

variable "environment" {
  description = "The environment for the resources"
  type        = string
  default     = "development"
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_subnet" "public" {
  name                 = "public-subnet"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.1.0/24"]
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_subnet" "private" {
  name                 = "private-subnet"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.example.name
  address_prefixes     = ["10.0.2.0/24"]
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_network_interface" "example" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.public.id
    private_ip_address_allocation = "Dynamic"
  }
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_virtual_machine" "example" {
  name                  = "example-vm"
  location              = azurerm_resource_group.example.location
  resource_group_name   = azurerm_resource_group.example.name
  network_interface_ids = [azurerm_network_interface.example.id]
  vm_size               = "Standard_B1s"

  storage_os_disk {
    name              = "example-os-disk"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }

  storage_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }

  os_profile {
    computer_name  = "example-vm"
    admin_username = "adminuser"
    admin_password = "P@ssw0rd123!" # Replace with a secure password
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

  tags = {
    Environment = var.environment
  }
}

resource "azurerm_storage_account" "example" {
  name                     = "examplestorageacc"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_sql_server" "example" {
  name                         = "examplesqlserver"
  resource_group_name          = azurerm_resource_group.example.name
  location                     = azurerm_resource_group.example.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = "P@ssw0rd123!" # Replace with a secure password
  tags = {
    Environment = var.environment
  }
}

resource "azurerm_sql_database" "example" {
  name                = "exampledb"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  server_name         = azurerm_sql_server.example.name
  edition             = "Basic"
  tags = {
    Environment = var.environment
  }
}

# Backend Configuration for State Management
terraform {
  backend "azurerm" {
    resource_group_name  = "your-resource-group"
    storage_account_name = "yourstorageaccount"
    container_name       = "terraform-state"
    key                  = "terraform.tfstate"
  }
}
```

## Directory Structure

```plaintext
azure-terraform/
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
variable "vm_size" {
  description = "Size of the virtual machine"
  type        = string
  default     = "Standard_B1s"
}
```

Output values in `outputs.tf`:
```hcl
output "vm_id" {
  description = "The ID of the Virtual Machine"
  value       = azurerm_virtual_machine.example.id
}
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.
---
