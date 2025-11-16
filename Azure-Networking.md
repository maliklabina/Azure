# Azure Networking

Azure Networking provides a comprehensive set of services for building, managing, and securing network infrastructure in the cloud with connectivity, security, and performance features.

------------------------------------------------------------------------

## Overview

Azure Networking enables organizations to connect resources securely, control traffic flow, and optimize network performance across cloud and hybrid environments.

### **Key Networking Services**

- **Virtual Networks (VNets)** - Private networks in Azure
- **Subnets** - Segmentation within VNets
- **Network Security Groups (NSGs)** - Firewall rules
- **Application Gateway** - Layer 7 load balancing
- **Load Balancer** - Layer 4 load balancing
- **VPN Gateway** - Site-to-site connectivity
- **ExpressRoute** - Dedicated private connections
- **DNS** - Azure DNS services
- **Content Delivery Network (CDN)** - Global content delivery

------------------------------------------------------------------------

## Virtual Networks (VNets)

### **Concepts**

A Virtual Network is an isolated network environment in Azure where you can deploy resources.

### **Create VNet**

```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create VNet
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet \
  --address-prefix 10.0.0.0/16 \
  --subnet-name mySubnet \
  --subnet-prefix 10.0.0.0/24
```

### **VNet Peering**

Connect two VNets together:

```bash
# Create second VNet
az network vnet create \
  --resource-group myResourceGroup \
  --name myVNet2 \
  --address-prefix 10.1.0.0/16 \
  --subnet-name mySubnet2 \
  --subnet-prefix 10.1.0.0/24

# Create peering
az network vnet peering create \
  --name myVNetToVNet2Peering \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --remote-vnet myVNet2 \
  --allow-vnet-access
```

### **VNet Architecture**

```
VNet (10.0.0.0/16)
├── Subnet1 (10.0.1.0/24)
│   ├── VM1
│   └── VM2
├── Subnet2 (10.0.2.0/24)
│   ├── VM3
│   └── AKS Cluster
└── Gateway Subnet (10.0.255.0/27)
    └── VPN Gateway
```

------------------------------------------------------------------------

## Subnets

### **Subnet Planning**

```
Private Subnets:
- Database tier: 10.0.1.0/24
- Application tier: 10.0.2.0/24
- Data tier: 10.0.3.0/24

Public Subnets:
- Front-end tier: 10.0.100.0/24
```

### **Create Subnets**

```bash
# Create subnet
az network vnet subnet create \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet2 \
  --address-prefix 10.0.2.0/24

# Associate NSG with subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet2 \
  --network-security-group myNSG
```

### **Service Endpoints**

Allow VNet resources to access Azure services privately:

```bash
# Enable service endpoint for subnet
az network vnet subnet update \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --name mySubnet \
  --service-endpoints Microsoft.Storage Microsoft.Sql
```

------------------------------------------------------------------------

## Network Security Groups (NSGs)

### **NSG Rules**

Control inbound and outbound traffic:

```bash
# Create NSG
az network nsg create \
  --resource-group myResourceGroup \
  --name myNSG

# Add inbound rule (allow HTTP)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowHTTP \
  --priority 100 \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 80 \
  --access Allow \
  --protocol Tcp

# Add inbound rule (allow SSH)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name AllowSSH \
  --priority 101 \
  --source-address-prefixes 203.0.113.0/24 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp

# Add outbound rule (deny all)
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNSG \
  --name DenyAllOutbound \
  --priority 100 \
  --direction Outbound \
  --source-address-prefixes '*' \
  --destination-address-prefixes '*' \
  --access Deny
```

### **NSG Best Practices**

- Create separate NSGs per tier
- Use application security groups
- Enable NSG flow logs
- Regularly audit rules
- Remove unused rules

------------------------------------------------------------------------

## Load Balancing

### **Azure Load Balancer (Layer 4)**

Distribute traffic at the transport layer:

```bash
# Create public IP
az network public-ip create \
  --resource-group myResourceGroup \
  --name myPublicIP

# Create load balancer
az network lb create \
  --resource-group myResourceGroup \
  --name myLB \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool

# Create health probe
az network lb probe create \
  --resource-group myResourceGroup \
  --lb-name myLB \
  --name myHealthProbe \
  --protocol tcp \
  --port 80

# Create load balancing rule
az network lb rule create \
  --resource-group myResourceGroup \
  --lb-name myLB \
  --name myLoadBalancingRule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontEnd \
  --backend-pool-name myBackEndPool \
  --probe-name myHealthProbe
```

### **Application Gateway (Layer 7)**

Advanced routing based on URL and hostname:

```bash
# Create application gateway
az network application-gateway create \
  --name myAppGateway \
  --resource-group myResourceGroup \
  --capacity 2 \
  --sku Standard_v2 \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myPublicIP \
  --vnet-name myVNet \
  --subnet mySubnet \
  --cert-password Azure123456!

# Add backend pool
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --name myBackendPool \
  --resource-group myResourceGroup \
  --servers 10.0.1.4 10.0.1.5

# Add routing rule
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name myRoutingRule \
  --priority 100 \
  --resource-group myResourceGroup
```

### **Load Balancer vs Application Gateway**

| Feature | Load Balancer | Application Gateway |
|---------|---------------|-------------------|
| Layer | 4 (Transport) | 7 (Application) |
| Protocol | TCP/UDP | HTTP/HTTPS |
| Routing | Port-based | URL path, hostname |
| SSL Termination | No | Yes |
| Complexity | Simple | Advanced |

------------------------------------------------------------------------

## VPN Gateway

### **Site-to-Site VPN**

Connect on-premises network to Azure VNet:

```bash
# Create VPN gateway
az network vnet-gateway create \
  --name myVPNGateway \
  --resource-group myResourceGroup \
  --vnet myVNet \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1

# Create local network gateway
az network local-gateway create \
  --gateway-ip-address 203.0.113.1 \
  --name myLocalNetworkGateway \
  --resource-group myResourceGroup \
  --local-address-prefix 192.168.0.0/16

# Create VPN connection
az network vpn-connection create \
  --name myVPNConnection \
  --resource-group myResourceGroup \
  --vnet-gateway1 myVPNGateway \
  --local-gateway2 myLocalNetworkGateway \
  --shared-key Azure123456!
```

### **Point-to-Site VPN**

Connect individual clients to Azure VNet:

```bash
# Create point-to-site configuration
az network vnet-gateway vpn-client generate \
  --resource-group myResourceGroup \
  --name myVPNGateway \
  --authentication-method EAPTLS
```

------------------------------------------------------------------------

## ExpressRoute

### **Dedicated Private Connectivity**

Establish private connection to Azure without using the internet:

```bash
# Create circuit
az network express-route create \
  --bandwidth 50 \
  --name myCircuit \
  --peering-location "Silicon Valley" \
  --provider "Equinix" \
  --resource-group myResourceGroup \
  --sku-family MeteredData \
  --sku-tier Standard

# Get circuit status
az network express-route show \
  --name myCircuit \
  --resource-group myResourceGroup
```

### **ExpressRoute Peering**

Configure private and Microsoft peering:

```
Private Peering:
├── Azure resources
├── Direct connectivity
└── 10 Gbps bandwidth

Microsoft Peering:
├── Microsoft services (Office 365, Dynamics)
├── Public IP addressing
└── Bidirectional access
```

------------------------------------------------------------------------

## Azure DNS

### **Create DNS Zone**

```bash
# Create DNS zone
az network dns zone create \
  --resource-group myResourceGroup \
  --name contoso.com

# Add DNS record
az network dns record-set a add-record \
  --resource-group myResourceGroup \
  --zone-name contoso.com \
  --record-set-name www \
  --ipv4-address 203.0.113.10

# Add CNAME record
az network dns record-set cname set-record \
  --resource-group myResourceGroup \
  --zone-name contoso.com \
  --record-set-name alias \
  --cname www.contoso.com
```

### **Private DNS Zones**

DNS resolution within VNets:

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group myResourceGroup \
  --name internal.contoso.com

# Link to VNet
az network private-dns link vnet create \
  --resource-group myResourceGroup \
  --zone-name internal.contoso.com \
  --name myVNetLink \
  --virtual-network myVNet \
  --registration-enabled true
```

------------------------------------------------------------------------

## Traffic Manager

### **Route Traffic Across Regions**

```bash
# Create Traffic Manager profile
az network traffic-manager profile create \
  --name myTrafficManager \
  --resource-group myResourceGroup \
  --routing-method Performance \
  --unique-dns-name mytrafficmanager

# Add endpoint
az network traffic-manager endpoint create \
  --name myEndpoint1 \
  --profile-name myTrafficManager \
  --resource-group myResourceGroup \
  --target myapp-eastus.azurewebsites.net \
  --type azureEndpoints
```

------------------------------------------------------------------------

## Content Delivery Network (CDN)

### **Distribute Content Globally**

```bash
# Create CDN profile
az cdn profile create \
  --resource-group myResourceGroup \
  --name myCDN \
  --sku Standard_Microsoft

# Create endpoint
az cdn endpoint create \
  --name myEndpoint \
  --profile-name myCDN \
  --resource-group myResourceGroup \
  --origin myapp.azurewebsites.net
```

------------------------------------------------------------------------

## Network Topology & Design

### **Hub-and-Spoke Architecture**

```
Hub VNet (10.0.0.0/16)
├── Contains: Shared services, firewalls
├── Peered to Spoke1
├── Peered to Spoke2
└── Peered to Spoke3

Spoke VNets
├── Spoke1 (10.1.0.0/16) - Production
├── Spoke2 (10.2.0.0/16) - Development
└── Spoke3 (10.3.0.0/16) - Testing
```

### **DMZ Architecture**

```
Internet → Load Balancer → DMZ Subnet (Public)
                              ↓
                          Firewall
                              ↓
                        Application Subnet (Private)
                              ↓
                        Database Subnet (Private)
```

------------------------------------------------------------------------

## Network Monitoring & Diagnostics

### **Network Watcher**

Monitor and troubleshoot network issues:

```bash
# Create Network Watcher
az network watcher configure \
  --resource-group myResourceGroup \
  --locations eastus

# Enable flow logging
az network watcher flow-log create \
  --resource-group myResourceGroup \
  --name myFlowLog \
  --nsg myNSG \
  --storage-account mystorageaccount

# Test connectivity
az network watcher test-connectivity \
  --resource-group myResourceGroup \
  --source-resource myVM \
  --destination-address 203.0.113.10 \
  --destination-port 443
```

### **Azure Monitor Integration**

Track network metrics:

- Data in/out
- Packet drops
- Connection count
- Health probe status

------------------------------------------------------------------------

## Common Commands

```bash
# List VNets
az network vnet list --resource-group myResourceGroup

# List subnets
az network vnet subnet list --resource-group myResourceGroup --vnet-name myVNet

# List NSGs
az network nsg list --resource-group myResourceGroup

# View NSG rules
az network nsg rule list --resource-group myResourceGroup --nsg-name myNSG

# Delete resources
az network vnet delete --resource-group myResourceGroup --name myVNet

# Get resource details
az network vnet show --resource-group myResourceGroup --name myVNet
```

