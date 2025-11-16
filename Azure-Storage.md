# Azure Storage

Azure Storage is a cloud-based storage solution providing scalable, secure, and highly available data storage services for various data types and access patterns.

------------------------------------------------------------------------

## Overview

Azure Storage provides massively scalable and secure cloud storage for data analytics, content delivery, backup, and more with multiple service options.

### **Core Services**

- **Blob Storage** - Unstructured data (images, videos, documents)
- **File Share** - Managed SMB/NFS file shares
- **Queue Storage** - Message queuing for applications
- **Table Storage** - NoSQL key-value store
- **Disk Storage** - Managed disks for VMs

------------------------------------------------------------------------

## Storage Accounts

### **Create Storage Account**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_GRS \
  --kind StorageV2 \
  --access-tier Hot
```

### **Storage Account Types**

| Type | Best For | Redundancy |
|------|----------|-----------|
| Standard_LRS | Dev/Test | Local redundancy |
| Standard_GRS | Production | Geographic redundancy |
| Standard_RAGRS | High availability | Read-access geographic redundancy |
| Premium_LRS | High performance | Local redundancy only |

### **Access Tiers**

- **Hot** - Frequent access, lower retrieval cost, higher storage cost
- **Cool** - Infrequent access, lower storage cost, higher retrieval cost
- **Archive** - Rare access, lowest storage cost, highest retrieval latency

```bash
# Change access tier
az storage account update \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --access-tier Cool
```

------------------------------------------------------------------------

## Blob Storage

### **Concepts**

Blob Storage stores unstructured data as objects in containers:

```
Storage Account
└── Blob Container
    ├── Blob 1 (Image)
    ├── Blob 2 (Video)
    └── Blob 3 (Document)
```

### **Create Container**

```bash
# Create container
az storage container create \
  --account-name mystorageaccount \
  --name mycontainer \
  --public-access blob

# List containers
az storage container list --account-name mystorageaccount
```

### **Upload & Download Blobs**

```bash
# Upload blob
az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./local-file.txt

# Download blob
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./downloaded-file.txt

# List blobs
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer
```

### **Blob Types**

- **Block Blob** - Text and binary files (optimal for most uses)
- **Page Blob** - Random read/write (VHD disks)
- **Append Blob** - Append operations (logs)

### **Blob Lifecycle Management**

Automatically transition blobs between access tiers:

```json
{
  "rules": [
    {
      "name": "archive-old-blobs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
```

### **Blob Snapshots**

Create point-in-time copies:

```bash
# Create snapshot
az storage blob snapshot \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt

# List snapshots
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include s
```

------------------------------------------------------------------------

## File Share Storage

### **Create File Share**

```bash
# Create file share
az storage share create \
  --account-name mystorageaccount \
  --name myfileshare \
  --quota 100

# List shares
az storage share list --account-name mystorageaccount
```

### **Mount File Share**

#### **Linux/macOS**

```bash
# Mount SMB share
sudo mount -t cifs //mystorageaccount.file.core.windows.net/myfileshare /mnt/azure \
  -o username=Azure\mystorageaccount,password=StorageAccountKey,vers=3.0,dir_mode=0777,file_mode=0777,sec=ntlmssp
```

#### **Windows PowerShell**

```powershell
# Map network drive
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\mystorageaccount.file.core.windows.net\myfileshare"
```

### **Upload Files to Share**

```bash
# Upload file
az storage file upload \
  --account-name mystorageaccount \
  --share-name myfileshare \
  --source ./local-file.txt

# List files
az storage file list \
  --account-name mystorageaccount \
  --share-name myfileshare \
  --path /
```

------------------------------------------------------------------------

## Queue Storage

### **Create Queue**

```bash
# Create queue
az storage queue create \
  --account-name mystorageaccount \
  --name myqueue

# List queues
az storage queue list --account-name mystorageaccount
```

### **Send & Receive Messages**

```bash
# Send message
az storage message put \
  --account-name mystorageaccount \
  --queue-name myqueue \
  --content "Hello, Queue!"

# Receive message
az storage message get \
  --account-name mystorageaccount \
  --queue-name myqueue

# Delete message
az storage message delete \
  --account-name mystorageaccount \
  --queue-name myqueue \
  --id <message-id> \
  --pop-receipt <receipt>

# Clear queue
az storage queue delete \
  --account-name mystorageaccount \
  --name myqueue
```

### **Queue Use Cases**

- Decoupling application components
- Load leveling
- Asynchronous task processing
- Inter-service communication

------------------------------------------------------------------------

## Table Storage

### **Create Table**

```bash
# Create table
az storage table create \
  --account-name mystorageaccount \
  --name mytable

# List tables
az storage table list --account-name mystorageaccount
```

### **Insert Entities**

```bash
# Insert entity
az storage entity insert \
  --account-name mystorageaccount \
  --table-name mytable \
  --entity PartitionKey=partition1 RowKey=row1 Name=John Age=30
```

### **Query Entities**

```bash
# Query all entities
az storage entity query \
  --account-name mystorageaccount \
  --table-name mytable

# Query with filter
az storage entity query \
  --account-name mystorageaccount \
  --table-name mytable \
  --filter "Age gt 25"
```

------------------------------------------------------------------------

## Managed Disks

### **Create Managed Disk**

```bash
# Create disk
az disk create \
  --resource-group myResourceGroup \
  --name myDisk \
  --size-gb 128 \
  --sku Premium_LRS

# Attach to VM
az vm disk attach \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --disk myDisk
```

### **Disk Types**

| Type | Use Case | Performance |
|------|----------|-------------|
| Standard_LRS | Dev/Test | 500 IOPS |
| StandardSSD_LRS | Web apps | 4,000 IOPS |
| Premium_LRS | Databases | 20,000 IOPS |
| UltraSSD_LRS | Mission-critical | 160,000 IOPS |

### **Disk Snapshots**

```bash
# Create snapshot
az snapshot create \
  --resource-group myResourceGroup \
  --name mySnapshot \
  --source myDisk

# Create disk from snapshot
az disk create \
  --resource-group myResourceGroup \
  --name myDiskFromSnapshot \
  --source mySnapshot
```

------------------------------------------------------------------------

## Security & Access Control

### **Storage Account Keys**

Shared keys for authentication:

```bash
# Get account keys
az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount

# Regenerate keys (rotate for security)
az storage account keys renew \
  --resource-group myResourceGroup \
  --account-name mystorageaccount \
  --key primary
```

### **Shared Access Signatures (SAS)**

Time-limited, permission-scoped tokens:

```bash
# Generate account SAS
az storage account generate-sas \
  --account-name mystorageaccount \
  --account-key <storage-key> \
  --expiry 2025-12-31 \
  --permissions racwd \
  --resource-types sco \
  --services bfqt

# Generate blob SAS
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2025-12-31
```

### **Azure AD Authentication**

Use managed identities and RBAC:

```bash
# Assign role
az role assignment create \
  --assignee <managed-identity-id> \
  --role "Storage Blob Data Contributor" \
  --scope /subscriptions/<subscription>/resourceGroups/<group>/providers/Microsoft.Storage/storageAccounts/<account>
```

### **Encryption**

```bash
# Enable customer-managed keys
az storage account update \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --encryption-key-name myKey \
  --encryption-key-vault /subscriptions/<sub>/resourceGroups/<group>/providers/Microsoft.KeyVault/vaults/<vault>
```

------------------------------------------------------------------------

## Replication & Redundancy

### **Local Redundancy (LRS)**
```
Data replicated 3 times within single data center
```

### **Zone Redundancy (ZRS)**
```
Data replicated across 3 availability zones in region
```

### **Geographic Redundancy (GRS)**
```
Primary: 3 replicas in primary region
Secondary: 3 replicas in paired region (read-only)
```

### **Read-Access Geographic Redundancy (RAGRS)**
```
Primary: 3 replicas in primary region
Secondary: 3 replicas in paired region (readable)
```

------------------------------------------------------------------------

## Monitoring & Diagnostics

### **Enable Storage Analytics**

```bash
# Enable logging
az storage logging update \
  --account-name mystorageaccount \
  --services b \
  --log d \
  --retention 7
```

### **Metrics**

Track storage performance:

```bash
# Enable metrics
az storage metrics update \
  --account-name mystorageaccount \
  --services b \
  --retention 7
```

### **Monitor in Azure Portal**

- Capacity trends
- Request patterns
- Error rates
- Latency metrics

------------------------------------------------------------------------

## Azure Storage Explorer

Cross-platform tool for managing storage:

```bash
# Download and install Azure Storage Explorer
# https://azure.microsoft.com/features/storage-explorer/

# Features:
# - Browse containers and blobs
# - Upload/download files
# - Manage access keys
# - Edit table entities
# - Monitor queues
```

------------------------------------------------------------------------

## Common Commands

```bash
# List storage accounts
az storage account list --resource-group myResourceGroup

# Show storage account details
az storage account show --name mystorageaccount --resource-group myResourceGroup

# Update storage account properties
az storage account update --name mystorageaccount --resource-group myResourceGroup --default-action Deny

# Get connection string
az storage account show-connection-string --name mystorageaccount --resource-group myResourceGroup

# Delete storage account
az storage account delete --name mystorageaccount --resource-group myResourceGroup

# Change replication
az storage account update --name mystorageaccount --sku Standard_RAGRS
```

------------------------------------------------------------------------

