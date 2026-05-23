# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## Domain 3: Deploy and Manage Azure Compute Resources
### (20–25%)

---

### ၃.၁ Virtual Machines

#### VM Size Families

| Family | ရှင်းလင်းချက် | Example |
|--------|-------------|---------|
| **A-series** | Basic, entry-level, dev/test | A2_v2 |
| **B-series** | Burstable, cost-effective for variable workloads | B2s |
| **D-series** | General purpose, balanced CPU/memory | D4s_v5 |
| **E-series** | Memory optimized | E8s_v5 |
| **F-series** | Compute optimized | F8s_v2 |
| **G-series** | Large memory, large local SSD | G5 |
| **H-series** | HPC, high CPU performance | H16r |
| **L-series** | Storage optimized, high disk throughput | L8s_v3 |
| **M-series** | Memory optimized, very large workloads | M128s |
| **N-series** | GPU-enabled | NC6, NV6 |

#### VM ဖန်တီးခြင်း (CLI)

```bash
# VM ဖန်တီးခြင်း
az vm create \
    --resource-group rg-compute \
    --name vm-webserver \
    --image Ubuntu2204 \
    --size Standard_B2s \
    --admin-username azureuser \
    --ssh-key-values ~/.ssh/id_rsa.pub \
    --vnet-name vnet-main \
    --subnet subnet-web \
    --public-ip-sku Standard \
    --nsg-rule SSH

# VM start/stop
az vm start --resource-group rg-compute --name vm-webserver
az vm deallocate --resource-group rg-compute --name vm-webserver
az vm restart --resource-group rg-compute --name vm-webserver
```

> ⚠️ `stop` command သည် VM ကို stop ဖြစ်သော်လည်း **deallocate မပြုလုပ်** → compute cost ဆက်ကုန်သည်။ Cost save ရန် `deallocate` သုံးပါ

#### VM Availability Options

| Option | SLA | ရှင်းလင်းချက် |
|--------|-----|-------------|
| No redundancy | 95% | Single VM |
| **Availability Set** | 99.95% | Same datacenter, different fault/update domains |
| **Availability Zone** | 99.99% | Different physical datacenters in same region |
| **VMSS with Zones** | 99.99%+ | Auto-scaling + zone distribution |

**Availability Set Concepts:**
- **Fault Domain (FD):** Shared power/network rack — max 3 FDs
- **Update Domain (UD):** VMs that may be rebooted together during maintenance — max 20 UDs

```bash
# Availability Set ဖန်တီးခြင်း
az vm availability-set create \
    --name avset-web \
    --resource-group rg-compute \
    --platform-fault-domain-count 2 \
    --platform-update-domain-count 5

# VM ဖန်တီးပြီး Availability Set ထဲ ထည့်ခြင်း
az vm create \
    --resource-group rg-compute \
    --name vm-web-01 \
    --availability-set avset-web \
    --image Win2022Datacenter \
    --size Standard_D2s_v5
```

---

### ၃.၂ VM Disks

#### Disk Types

| Type | Latency | IOPS/Throughput | Best For |
|------|---------|-----------------|---------|
| **Ultra Disk** | Sub-ms | 160,000 IOPS / 2,000 MB/s | HPC, SAP HANA, Tier-1 DBs |
| **Premium SSD v2** | Sub-ms | 80,000 IOPS / 1,200 MB/s | Production databases |
| **Premium SSD (P-series)** | Single-ms | 20,000 IOPS / 900 MB/s | Production workloads |
| **Standard SSD (E-series)** | Low-ms | 6,000 IOPS / 750 MB/s | Light production, test |
| **Standard HDD (S-series)** | Variable | 2,000 IOPS / 500 MB/s | Dev/test, backup |

#### Managed vs Unmanaged Disks

> ✅ **Always use Managed Disks** — Storage account management မပြုဘဲ Azure က disk ကို manage ပြုလုပ်သည်; SLA ပိုမြင့်; simpler

#### Azure Disk Encryption

```bash
# Key Vault ဖန်တီးပြီး disk encryption key ကို store
az keyvault create \
    --name kv-disks-prod \
    --resource-group rg-compute \
    --location southeastasia \
    --enabled-for-disk-encryption

# VM disk encryption enable
az vm encryption enable \
    --resource-group rg-compute \
    --name vm-webserver \
    --disk-encryption-keyvault kv-disks-prod \
    --volume-type All
```

---

### ၃.၃ VM Scale Sets (VMSS)

VM Scale Sets (VMSS) သည် identical VM instances များကို auto-scale ဖြင့် horizontally scale ပြုလုပ်ရာတွင် သုံးသည်

```bash
# VMSS ဖန်တီးခြင်း
az vmss create \
    --resource-group rg-compute \
    --name vmss-web \
    --image Ubuntu2204 \
    --vm-sku Standard_D2s_v5 \
    --instance-count 3 \
    --admin-username azureuser \
    --ssh-key-values ~/.ssh/id_rsa.pub \
    --zones 1 2 3 \
    --upgrade-policy-mode Automatic

# Autoscale rule ဖန်တီးခြင်း (CPU 70% တက်ပါက scale out)
az monitor autoscale create \
    --resource-group rg-compute \
    --resource vmss-web \
    --resource-type Microsoft.Compute/virtualMachineScaleSets \
    --name autoscale-web \
    --min-count 2 \
    --max-count 10 \
    --count 3

az monitor autoscale rule create \
    --resource-group rg-compute \
    --autoscale-name autoscale-web \
    --condition "Percentage CPU > 70 avg 5m" \
    --scale out 2
```

#### Orchestration Modes

| Mode | ရှင်းလင်းချက် |
|------|-------------|
| **Uniform** | Identical VM instances; auto-scale optimized |
| **Flexible** | Mixed VM types; manual scale support; better HA |

---

### ၃.၄ ARM Templates and Bicep

#### ARM Template Structure

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "vm-web",
      "metadata": { "description": "VM name" }
    }
  },
  "variables": {
    "location": "[resourceGroup().location]"
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-09-01",
      "name": "[parameters('vmName')]",
      "location": "[variables('location')]",
      "properties": { ... }
    }
  ],
  "outputs": {
    "vmId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

#### Bicep (ARM Template ၏ Simplified Language)

```bicep
// Bicep ဖြင့် storage account ဖန်တီးခြင်း
param storageAccountName string = 'mystorage${uniqueString(resourceGroup().id)}'
param location string = resourceGroup().location
param skuName string = 'Standard_LRS'

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: skuName
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
    minimumTlsVersion: 'TLS1_2'
    allowBlobPublicAccess: false
  }
}

output storageAccountId string = storageAccount.id
```

```bash
# Bicep file ကို deploy ပြုလုပ်ခြင်း
az deployment group create \
    --resource-group rg-storage \
    --template-file storage.bicep \
    --parameters storageAccountName=mystorage2025
```

---

### ၃.၅ Containers on Azure

#### Azure Container Instances (ACI)

ACI သည် VM management မပြုဘဲ single containers ကို serverless ဖြင့် run ရာတွင် သုံးသည်

```bash
# Container instance ဖန်တီးခြင်း
az container create \
    --resource-group rg-containers \
    --name myapp-container \
    --image nginx:latest \
    --dns-name-label myapp-2025 \
    --ports 80 \
    --cpu 1 \
    --memory 1.5 \
    --os-type Linux \
    --restart-policy Always

# Container logs ကြည့်ရန်
az container logs \
    --resource-group rg-containers \
    --name myapp-container

# Container ထဲ exec ဝင်ရန်
az container exec \
    --resource-group rg-containers \
    --name myapp-container \
    --exec-command "/bin/bash"
```

#### Azure Kubernetes Service (AKS)

AKS သည် Kubernetes cluster ကို managed ဖြစ်အောင် Azure က control plane ကို operate ပြုလုပ်သည်

```bash
# AKS cluster ဖန်တီးခြင်း
az aks create \
    --resource-group rg-containers \
    --name aks-production \
    --node-count 3 \
    --node-vm-size Standard_D2s_v5 \
    --enable-managed-identity \
    --network-plugin azure \
    --zones 1 2 3 \
    --generate-ssh-keys

# kubectl credentials ရယူခြင်း
az aks get-credentials \
    --resource-group rg-containers \
    --name aks-production

# Cluster status စစ်ဆေးခြင်း
kubectl get nodes
kubectl get pods --all-namespaces
```

#### Container Registry (ACR)

```bash
# ACR ဖန်တီးခြင်း
az acr create \
    --resource-group rg-containers \
    --name myacrregistry2025 \
    --sku Premium \
    --admin-enabled false

# Docker image ကို ACR သို့ push ပြုလုပ်ခြင်း
az acr login --name myacrregistry2025
docker tag myapp:latest myacrregistry2025.azurecr.io/myapp:latest
docker push myacrregistry2025.azurecr.io/myapp:latest

# AKS ကို ACR နှင့် attach ပြုလုပ်ခြင်း
az aks update \
    --resource-group rg-containers \
    --name aks-production \
    --attach-acr myacrregistry2025
```

---

### ၃.၆ App Service

#### App Service Plans

| Tier | Features | Scale |
|------|---------|-------|
| **Free (F1)** | Dev only, 60 min/day CPU | Shared |
| **Shared (D1)** | Custom domain | Shared |
| **Basic (B1-B3)** | Custom domain, SSL | Dedicated, manual scale |
| **Standard (S1-S3)** | Staging slots, auto-scale | Dedicated, auto-scale |
| **Premium (P1v3-P3v3)** | Zone redundancy, VNet integration | Dedicated, auto-scale |
| **Isolated (I1v2-I3v2)** | Private network, dedicated hardware | App Service Environment |

```bash
# App Service Plan ဖန်တီးခြင်း
az appservice plan create \
    --name asp-production \
    --resource-group rg-webapp \
    --location southeastasia \
    --sku P2v3 \
    --is-linux

# Web App ဖန်တီးခြင်း
az webapp create \
    --resource-group rg-webapp \
    --plan asp-production \
    --name mywebapp-2025 \
    --runtime "NODE:20-lts"

# Deployment slot ဖန်တီးခြင်း
az webapp deployment slot create \
    --name mywebapp-2025 \
    --resource-group rg-webapp \
    --slot staging

# Slot swap (Blue/Green deployment)
az webapp deployment slot swap \
    --name mywebapp-2025 \
    --resource-group rg-webapp \
    --slot staging \
    --target-slot production
```

---

<a name="domain-4"></a>
