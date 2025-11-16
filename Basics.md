# Azure Basics

Microsoft Azure is a cloud computing platform providing a wide range of services for computing, storage, networking, databases, and more.

------------------------------------------------------------------------

## Overview

Azure enables organizations to build, deploy, and manage applications through cloud-based infrastructure and platform services globally.

### **Azure Services Categories**

- **Compute** - Virtual machines, containers, serverless
- **Storage** - Blob, file shares, databases
- **Networking** - VNets, load balancers, VPN
- **Databases** - SQL, NoSQL, managed databases
- **Analytics** - Data processing, AI/ML
- **Management** - Monitoring, automation
- **Security** - Identity, encryption, compliance

------------------------------------------------------------------------

## Key Concepts

### **Subscription**

A billing entity and container for Azure resources. You can have multiple subscriptions under one account.

```bash
# List subscriptions
az account list

# Set active subscription
az account set --subscription "subscription-id"

# Get current subscription
az account show
```

### **Resource Group**

A container that holds related resources for an application or project.

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# List resource groups
az group list

# Delete resource group
az group delete --name myResourceGroup
```

### **Region**

Geographic location where Azure resources are deployed. Each region has multiple availability zones.

**Common regions:**
- East US
- West US
- Europe West
- Southeast Asia
- Japan East

------------------------------------------------------------------------

## Virtual Machines (VMs)

### **What is a VM?**

Virtualized computing resource that runs operating systems and applications - like a physical computer in the cloud.

### **Create VM**

```bash
# Create VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys

# View VMs
az vm list --resource-group myResourceGroup

# Stop VM
az vm stop --name myVM --resource-group myResourceGroup

# Start VM
az vm start --name myVM --resource-group myResourceGroup

# Delete VM
az vm delete --name myVM --resource-group myResourceGroup
```

### **VM Sizes**

| Category | Size | Use Case |
|----------|------|----------|
| General Purpose | B2s, D2s_v3 | Dev, testing, small apps |
| Compute Optimized | F2s_v2 | High CPU demand |
| Memory Optimized | E4s_v3 | Databases, caching |
| Storage Optimized | L8s_v2 | High I/O operations |
| GPU | NC6, ND6 | AI/ML, graphics |

------------------------------------------------------------------------

## Storage

### **Azure Storage Types**

Storage accounts provide various storage solutions for different needs.

#### **1. Blob Storage**

Store unstructured data - images, videos, documents, backups.

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_GRS

# Create blob container
az storage container create \
  --account-name mystorageaccount \
  --name mycontainer

# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./local-file.txt
```

#### **2. File Shares**

Managed SMB/NFS file shares - mount on multiple VMs.

```bash
# Create file share
az storage share create \
  --account-name mystorageaccount \
  --name myfileshare \
  --quota 100
```

#### **3. Table Storage**

NoSQL key-value store for structured data.

```bash
# Create table
az storage table create \
  --account-name mystorageaccount \
  --name mytable
```

#### **4. Queue Storage**

Message queue for asynchronous communication.

```bash
# Create queue
az storage queue create \
  --account-name mystorageaccount \
  --name myqueue
```

### **Storage Access Tiers**

| Tier | Use Case | Cost |
|------|----------|------|
| **Hot** | Frequent access | Higher storage, lower retrieval |
| **Cool** | Infrequent access | Lower storage, higher retrieval |
| **Archive** | Rare access | Lowest storage, slowest retrieval |

------------------------------------------------------------------------

## Networking

### **Virtual Network (VNet)**

Private network in Azure where you deploy resources.

```bash
# Create VNet
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.0.0/24

# List VNets
az network vnet list --resource-group myResourceGroup
```

### **Network Security Group (NSG)**

Firewall for controlling inbound/outbound traffic.

```bash
# Create NSG
az network nsg create \
  --resource-group myResourceGroup \
  --name myNSG

# Add inbound rule (allow SSH)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowSSH \
  --priority 100 \
  --source-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp
```

### **Public IP**

Static or dynamic IP address for external access.

```bash
# Create public IP
az network public-ip create \
  --resource-group myResourceGroup \
  --name myPublicIP \
  --sku Standard
```

------------------------------------------------------------------------

## Databases

### **Azure SQL Database**

Managed relational database - SQL Server in the cloud.

```bash
# Create SQL server
az sql server create \
  --name myserver \
  --resource-group myResourceGroup \
  --location eastus \
  --admin-user sqladmin \
  --admin-password Password123!

# Create database
az sql db create \
  --resource-group myResourceGroup \
  --server myserver \
  --name mydatabase \
  --service-objective S0
```

### **Azure Cosmos DB**

Globally distributed NoSQL database.

```bash
# Create Cosmos DB account
az cosmosdb create \
  --name mycosmosdb \
  --resource-group myResourceGroup \
  --locations regionName=eastus failoverPriority=0
```

### **Azure Database for MySQL/PostgreSQL**

Managed open-source databases.

```bash
# Create PostgreSQL server
az postgres server create \
  --resource-group myResourceGroup \
  --name mypostgresserver \
  --location eastus \
  --admin-user azureuser \
  --admin-password Password123!
```

------------------------------------------------------------------------

## App Service

### **Web App Hosting**

Managed platform for hosting web applications.

```bash
# Create App Service Plan
az appservice plan create \
  --name myAppServicePlan \
  --resource-group myResourceGroup \
  --sku B1 \
  --is-linux

# Create Web App
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name mywebapp \
  --runtime "PYTHON|3.11"
```

### **App Service Plans**

| Plan | Auto-scale | Use Case |
|------|-----------|----------|
| Free | No | Dev/test |
| Shared | No | Shared resources |
| Basic | Manual | Small apps |
| Standard | Yes | Production |
| Premium | Yes | High performance |

------------------------------------------------------------------------

## Containers

### **Azure Container Instances**

Run containers without managing VMs.

```bash
# Create container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myimage:latest \
  --cpu 1 \
  --memory 1

# View logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer
```

### **Azure Container Registry**

Private registry for storing container images.

```bash
# Create registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Basic

# Push image
az acr build \
  --registry myregistry \
  --image myapp:latest .
```

------------------------------------------------------------------------

## Identity & Access

### **Azure Active Directory (Azure AD)**

Identity management service for authentication and authorization.

```bash
# Check current user
az ad user list

# Create app registration
az ad app create --display-name myapp
```

### **Role-Based Access Control (RBAC)**

Control who can access what resources.

```bash
# List role assignments
az role assignment list --resource-group myResourceGroup

# Assign role
az role assignment create \
  --assignee user@example.com \
  --role "Contributor" \
  --resource-group myResourceGroup
```

------------------------------------------------------------------------

## Monitoring & Management

### **Azure Monitor**

Monitoring and diagnostics for Azure resources.

```bash
# View metrics
az monitor metrics list \
  --resource /subscriptions/xxx/resourceGroups/rg/providers/Microsoft.Compute/virtualMachines/vm

# Create alert
az monitor metrics alert create \
  --name myalert \
  --resource-group myResourceGroup \
  --description "Alert on high CPU"
```

### **Cost Management**

Track and optimize Azure spending.

```bash
# View costs
az billing invoice list
```

------------------------------------------------------------------------

## Common Resources Hierarchy

```
Subscription
└── Resource Group
    ├── Virtual Network
    │   └── Subnet
    │       └── Network Security Group
    ├── Virtual Machine
    │   └── Public IP
    ├── Storage Account
    │   ├── Blob Container
    │   ├── File Share
    │   └── Queue
    ├── SQL Database
    └── App Service
```

------------------------------------------------------------------------

## Quick Start Example

```bash
# 1. Create resource group
az group create --name myapp --location eastus

# 2. Create VNet
az network vnet create \
  --resource-group myapp \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.0.0/24

# 3. Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myapp \
  --location eastus \
  --sku Standard_LRS

# 4. Create VM
az vm create \
  --resource-group myapp \
  --name myVM \
  --image Ubuntu2204 \
  --vnet-name myVNet \
  --subnet mySubnet \
  --public-ip-sku Standard \
  --admin-username azureuser \
  --generate-ssh-keys

# 5. View resources
az resource list --resource-group myapp
```

------------------------------------------------------------------------

## Useful Commands

```bash
# List all resources
az resource list --resource-group myResourceGroup

# Get resource details
az resource show --resource-group myResourceGroup --name myVM --resource-type "Microsoft.Compute/virtualMachines"

# Delete all resources in group
az group delete --name myResourceGroup

# View CLI version
az version

# Get help
az <command> --help
```
