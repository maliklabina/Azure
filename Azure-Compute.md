# Azure Compute

Azure Compute provides virtualized computing resources for running applications, from single VMs to containerized workloads and serverless functions.

------------------------------------------------------------------------

## Overview

Azure Compute services enable organizations to deploy and scale applications using various compute models - from Infrastructure-as-a-Service (IaaS) to Platform-as-a-Service (PaaS) and Serverless.

### **Compute Services**

- **Virtual Machines (VMs)** - Full control, Windows/Linux
- **Virtual Machine Scale Sets** - Auto-scaling groups of VMs
- **App Service** - Managed web app hosting
- **Container Instances** - Serverless containers
- **Azure Kubernetes Service (AKS)** - Managed Kubernetes
- **Functions** - Serverless event-driven computing
- **Batch** - Large-scale parallel computing
- **Dedicated Host** - Physical servers for compliance

------------------------------------------------------------------------

## Virtual Machines (VMs)

### **VM Creation**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create VM
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys

# Create Windows VM
az vm create \
  --resource-group myResourceGroup \
  --name myWindowsVM \
  --image Win2022Datacenter \
  --admin-username azureuser \
  --admin-password Password1234!
```

### **VM Sizing**

| Category | Use Case | Examples |
|----------|----------|----------|
| **General Purpose** | Dev/test, small-medium apps | B1s, B2s, D2s_v3 |
| **Compute Optimized** | High CPU demand, batch processing | F2s_v2, F4s_v2 |
| **Memory Optimized** | Databases, caching, analytics | E2s_v3, M8m |
| **Storage Optimized** | High disk throughput | L8s_v2, L16s_v2 |
| **GPU** | AI/ML, graphics processing | NC6, ND6, NV6 |

### **VM Management Commands**

```bash
# List VMs
az vm list --resource-group myResourceGroup

# Start VM
az vm start --name myVM --resource-group myResourceGroup

# Stop VM
az vm stop --name myVM --resource-group myResourceGroup

# Deallocate VM (stop and free compute resources)
az vm deallocate --name myVM --resource-group myResourceGroup

# Delete VM
az vm delete --name myVM --resource-group myResourceGroup

# Get VM details
az vm show --name myVM --resource-group myResourceGroup

# Connect to VM
ssh -i ~/.ssh/id_rsa azureuser@<publicIP>
```

### **Disk Management**

```bash
# Create managed disk
az disk create \
  --resource-group myResourceGroup \
  --name myDisk \
  --size-gb 128 \
  --sku Premium_LRS

# Attach disk to VM
az vm disk attach \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --disk myDisk

# List disks
az disk list --resource-group myResourceGroup

# Delete disk
az disk delete --name myDisk --resource-group myResourceGroup
```

------------------------------------------------------------------------

## Virtual Machine Scale Sets (VMSS)

### **What is VMSS?**

Automatically deploy and manage a group of identical VMs that scale based on demand or schedule.

**Benefits:**
- Auto-scaling based on metrics
- High availability
- Load balancing built-in
- Uniform VM configuration

### **Create Scale Set**

```bash
# Create scale set
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --vm-sku Standard_B2s \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys

# Update instance count
az vmss scale \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --new-capacity 5
```

### **Autoscaling**

```bash
# Create autoscale settings
az monitor autoscale create \
  --resource-group myResourceGroup \
  --resource myScaleSet \
  --resource-type "Microsoft.Compute/virtualMachineScaleSets" \
  --name myAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 3

# Add scale rule (CPU > 75%)
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name myAutoscale \
  --condition "Percentage CPU > 75 avg 5m" \
  --scale out 1
```

------------------------------------------------------------------------

## App Service

### **Web App Hosting**

Managed platform for hosting web apps without managing infrastructure.

### **Create App Service Plan & Web App**

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
  --name myWebApp \
  --runtime "PYTHON|3.11"

# Deploy from git
az webapp deployment source config-zip \
  --resource-group myResourceGroup \
  --name myWebApp \
  --src myapp.zip
```

### **App Service Plans**

| Tier | Use Case | Auto-scale | Cost |
|------|----------|-----------|------|
| **Free** | Dev/test | No | Free |
| **Shared** | Shared resources | No | $ |
| **Basic** | Small apps | Manual | $ |
| **Standard** | Production apps | Yes | $$ |
| **Premium** | High performance | Yes | $$$ |

------------------------------------------------------------------------

## Container Instances

### **Azure Container Instances (ACI)**

Run containers without managing VMs - serverless containers.

### **Deploy Container**

```bash
# Create container
az container create \
  --resource-group myResourceGroup \
  --name myContainer \
  --image myregistry.azurecr.io/myapp:1.0 \
  --cpu 1 \
  --memory 1 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username username \
  --registry-password password \
  --environment-variables LOG_LEVEL=INFO \
  --ports 80 443

# View container status
az container show \
  --resource-group myResourceGroup \
  --name myContainer

# View logs
az container logs \
  --resource-group myResourceGroup \
  --name myContainer
```

------------------------------------------------------------------------

## Azure Functions

### **Serverless Computing**

Event-driven compute service - pay only for execution time.

### **Function Types**

- **HTTP Trigger** - Response to HTTP requests
- **Timer Trigger** - Scheduled execution
- **Queue Trigger** - Process queue messages
- **Blob Trigger** - React to storage changes
- **Event Hub Trigger** - Stream processing
- **Service Bus Trigger** - Message processing

### **Create Function**

```bash
# Create function app
az functionapp create \
  --resource-group myResourceGroup \
  --name myFunctionApp \
  --storage-account mystorageaccount \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4

# Create function
func new --name myFunction --template "HTTP trigger"

# Publish function
func azure functionapp publish myFunctionApp
```

------------------------------------------------------------------------

## Batch Computing

### **Azure Batch**

Large-scale parallel computing for scientific and engineering workloads.

**Use cases:**
- Financial modeling
- Image processing
- Video encoding
- Machine learning training

### **Create Batch Account**

```bash
# Create batch account
az batch account create \
  --name myBatchAccount \
  --resource-group myResourceGroup \
  --location eastus \
  --storage-account mystorageaccount
```

------------------------------------------------------------------------

## Dedicated Host

### **Azure Dedicated Host**

Dedicated physical servers for compliance and licensing requirements.

**Benefits:**
- License mobility (Windows Server, SQL Server)
- Compliance requirements
- Cost savings for multi-core licenses
- VM isolation

### **Create Dedicated Host**

```bash
# Create host group
az dedicated-host group create \
  --resource-group myResourceGroup \
  --name myHostGroup \
  --platform-fault-domain-count 2 \
  --location eastus \
  --sku ESv3-Type1

# Create dedicated host
az dedicated-host create \
  --resource-group myResourceGroup \
  --host-group-name myHostGroup \
  --name myDedicatedHost \
  --sku ESv3-Type1 \
  --platform-fault-domain 1
```

------------------------------------------------------------------------

## Best Practices

- **Right-size VMs** - Choose appropriate size for workload
- **Use managed disks** - Better performance and security
- **Enable auto-shutdown** - Dev/test VMs to save costs
- **Use scale sets** - Auto-scaling for consistency
- **Implement monitoring** - Track VM health and performance
- **Use tags** - Organize and manage resources
- **Backup critical data** - Regular snapshots and backups
- **Security groups** - Restrict inbound/outbound traffic
- **Update regularly** - Security patches and updates
- **Use spot instances** - Cost optimization for non-critical workloads

------------------------------------------------------------------------

## Cost Optimization

- **Deallocate stopped VMs** - Stop doesn't free compute
- **Use reserved instances** - 1-year or 3-year discounts
- **Spot instances** - Up to 90% discount for interruptible workloads
- **Hybrid benefit** - Use existing licenses
- **Auto-shutdown** - Automatically stop dev/test VMs
- **Right-sizing** - Avoid oversized VMs
- **Migrate to PaaS** - Use App Service instead of VMs when possible
- **Monitor usage** - Use Azure Cost Management

------------------------------------------------------------------------

## Common Commands Reference

```bash
# VM operations
az vm create --resource-group rg --name vm --image Ubuntu2204
az vm start --name vm --resource-group rg
az vm stop --name vm --resource-group rg
az vm deallocate --name vm --resource-group rg
az vm delete --name vm --resource-group rg

# Scale Set operations
az vmss create --resource-group rg --name ss --image Ubuntu2204
az vmss scale --name ss --resource-group rg --new-capacity 5
az vmss update --name ss --resource-group rg

# App Service operations
az appservice plan create --name plan --resource-group rg --sku B1
az webapp create --name app --plan plan --resource-group rg

# Container operations
az container create --name container --image image:tag --resource-group rg
az container show --name container --resource-group rg
az container logs --name container --resource-group rg

# Function operations
az functionapp create --name app --storage-account sa --resource-group rg
az functionapp delete --name app --resource-group rg
```

------------------------------------------------------------------------

## Troubleshooting

| Issue | Solution |
|-------|----------|
| VM won't start | Check allocation error, verify resources available |
| SSH connection refused | Check NSG rules, verify public IP |
| High CPU usage | Scale up VM size or use VMSS for horizontal scaling |
| Disk full | Add new disk, resize existing disk |
| Out of storage quota | Delete unused resources, clean up storage |
| Container not starting | Check image exists, verify credentials, check logs |
