# Top 10 Essential Azure Services with Terraform Examples

## Table of Contents

1. [Virtual Machines (VMs)](#1-azure-virtual-machines)
2. [Azure Blob Storage](#2-azure-blob-storage)
3. [Azure SQL Database](#3-azure-sql-database)
4. [Azure Functions](#4-azure-functions)
5. [Virtual Network (VNet)](#5-azure-virtual-network)
6. [Azure Active Directory (AAD)](#6-azure-active-directory)
7. [Azure Kubernetes Service (AKS)](#7-azure-kubernetes-service)
8. [Azure App Service](#8-azure-app-service)
9. [Azure Cosmos DB](#9-azure-cosmos-db)
10. [Azure Load Balancer](#10-azure-load-balancer)

## Prerequisites

- Azure Subscription
- Azure CLI installed and logged in
- Terraform installed

## 1. Azure Virtual Machines {#1-azure-virtual-machines}

### Overview of Azure Virtual Machines

Azure VMs provide on-demand, scalable computing resources with flexible pricing options.

### Azure VM Terraform Example

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "East US"
}

resource "azurerm_virtual_network" "main" {
  name                = "example-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "main" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_linux_virtual_machine" "example" {
  name                = "example-machine"
  resource_group_name = azurerm_resource_group.example.name
  location            = azurerm_resource_group.example.location
  size                = "Standard_F2"
  admin_username      = "adminuser"
  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  admin_ssh_key {
    username   = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }


  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }


  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
}
```

## 2. Azure Blob Storage {#2-azure-blob-storage}

### Overview of Azure Blob Storage

Azure Blob Storage is Microsoft's object storage solution for the cloud, optimized for storing massive amounts of unstructured data.

### Blob Storage Terraform Example

```hcl
resource "azurerm_storage_account" "example" {
  name                     = "examplestorageacct"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "example" {
  name                  = "examplecontainer"
  storage_account_name  = azurerm_storage_account.example.name
  container_access_type = "private"
}
```

## 3. Azure SQL Database {#3-azure-sql-database}

### Overview of Azure SQL Database

Fully managed relational database with auto-scale, integral intelligence, and robust security.

### SQL Database Terraform Example

```hcl
resource "azurerm_sql_server" "example" {
  name                         = "examplesqlserver"
  resource_group_name          = azurerm_resource_group.example.name
  location                     = azurerm_resource_group.example.location
  version                      = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = "H@Sh1CoR3!"
}

resource "azurerm_sql_database" "example" {
  name                = "examplesqldb"
  resource_group_name = azurerm_resource_group.example.name
  location           = azurerm_resource_group.example.location
  server_name        = azurerm_sql_server.example.name
  edition            = "Standard"
  requested_service_objective_name = "S0"
}
```

## 4. Azure Functions {#4-azure-functions}

### Overview of Azure Functions

Event-driven serverless compute platform that can solve complex orchestration problems.

### Functions Terraform Example

```hcl
resource "azurerm_function_app" "example" {
  name                       = "example-function-app"
  location                   = azurerm_resource_group.example.location
  resource_group_name        = azurerm_resource_group.example.name
  app_service_plan_id        = azurerm_app_service_plan.example.id
  storage_account_name       = azurerm_storage_account.example.name
  storage_account_access_key = azurerm_storage_account.example.primary_access_key
  os_type                    = "linux"
  version                    = "~3"

  app_settings = {
    "FUNCTIONS_WORKER_RUNTIME" = "node"
  }
}
```

## 5. Azure Virtual Network {#5-azure-virtual-network}

### Overview of Azure Virtual Network

Provides private networking for your Azure resources.

### VNet Terraform Example

```hcl
resource "azurerm_virtual_network" "example" {
  name                = "example-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}
```

## Next Steps

1. Run `az login` to authenticate with Azure
2. Run `terraform init` to initialize the provider
3. Run `terraform plan` to see the execution plan
4. Run `terraform apply` to create the resources

## Best Practices

- Use Azure Resource Manager (ARM) templates for complex deployments
- Implement Azure Policy for governance
- Use managed identities instead of service principals when possible
- Enable diagnostic settings for all resources
- Use Azure Monitor for observability
