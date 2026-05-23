# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## Domain 2: Implement and Manage Storage
### (15–20%)

---

### ၂.၁ Azure Storage Accounts

Azure Storage Account သည် Azure ၏ cloud object, file, queue, table storage service များကို host ပြုလုပ်ရာ container ဖြစ်သည်။

#### Storage Account Types

| Account Type | Supported Services | Performance | Replication |
|-------------|-------------------|-------------|-------------|
| **General Purpose v2 (GPv2)** | Blob, File, Queue, Table | Standard/Premium | All options |
| **BlockBlobStorage** | Block blobs, Append blobs | Premium | LRS, ZRS |
| **FileStorage** | Azure Files | Premium | LRS, ZRS |
| **BlobStorage (legacy)** | Blob only | Standard | LRS, GRS, RA-GRS |

> 📌 **Best Practice:** New storage accounts ဖန်တီးသည့်အခါ GPv2 ကိုသာ သုံးပါ

#### Replication Options

| Type | Full Name | Description | SLA |
|------|-----------|-------------|-----|
| **LRS** | Locally Redundant Storage | 1 datacenter, 3 copies | 99.9% |
| **ZRS** | Zone-Redundant Storage | 3 availability zones | 99.9% |
| **GRS** | Geo-Redundant Storage | 2 regions (primary=LRS, secondary=LRS) | 99.9% |
| **GZRS** | Geo-Zone-Redundant Storage | Primary=ZRS + Secondary=LRS | 99.99% |
| **RA-GRS** | Read-Access GRS | GRS + read access to secondary | 99.99% |
| **RA-GZRS** | Read-Access GZRS | GZRS + read access to secondary | 99.9999% |

> ⚠️ **Exam Trap:** RA-GRS တွင် secondary region read access ရသည်; write access မရ — failover ဖြစ်မှသာ primary ဖြစ်လာသည်

#### Storage Account ဖန်တီးခြင်း

```bash
# Storage account ဖန်တီးခြင်း
az storage account create \
    --name mystorageacct2025 \
    --resource-group rg-storage \
    --location southeastasia \
    --sku Standard_GRS \
    --kind StorageV2 \
    --access-tier Hot \
    --min-tls-version TLS1_2 \
    --allow-blob-public-access false
```

#### Network Access Control

Storage Account ကို network level တွင် ကာကွယ်ရန်:

```
Network Access Options:
├── Public Access (All Networks) — default, any IP ဝင်နိုင်
├── Selected Networks
│   ├── VNet Service Endpoints (VNet traffic allow)
│   ├── IP Rules (specific IP/CIDR allow)
│   └── Private Endpoint (VNet-only, no public endpoint)
└── Disabled (Private Endpoint only)
```

```bash
# Specific VNet ကို storage access allow
az storage account network-rule add \
    --account-name mystorageacct2025 \
    --resource-group rg-storage \
    --vnet-name vnet-main \
    --subnet subnet-app

# Specific IP allow
az storage account network-rule add \
    --account-name mystorageacct2025 \
    --resource-group rg-storage \
    --ip-address "203.0.113.0/24"
```

---

### ၂.၂ Blob Storage

#### Blob Types

| Type | Use Case |
|------|----------|
| **Block Blob** | Files, images, documents (most common) |
| **Append Blob** | Log files (efficient append operations) |
| **Page Blob** | VHD files, disks (random read/write optimized) |

#### Access Tiers

| Tier | Storage Cost | Access Cost | Use Case |
|------|-------------|------------|---------|
| **Hot** | High | Low | Frequently accessed data |
| **Cool** | Medium | Medium | Infrequent access (min 30 days) |
| **Cold** | Low | Higher | Rare access (min 90 days) |
| **Archive** | Very Low | Very High | Long-term retention (min 180 days); **offline** |

> ⚠️ **Archive tier:** Archive blobs ကို access ပြုလုပ်ရန် "rehydrate" ပြုလုပ်ရသည် (Standard: 15 hrs, High Priority: 1 hr)

#### Lifecycle Management

```json
{
  "rules": [
    {
      "name": "MoveToArchive",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 },
            "delete": { "daysAfterModificationGreaterThan": 365 }
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

---

### ၂.၃ Storage Access နှင့် Security

#### Shared Access Signatures (SAS)

SAS သည် storage resources ကို limited access (specific time, IP, permissions) ဖြင့် access ပေးနိုင်သော signed URL ဖြစ်သည်

| SAS Type | Revocation | Description |
|----------|-----------|-------------|
| **Account SAS** | Access key rotate ဖြင့် | Storage account level |
| **Service SAS** | Access key rotate ဖြင့် | Specific service (blob/file/queue/table) |
| **User Delegation SAS** | Entra ID permissions ဖြင့် | **Recommended; Entra credentials ဖြင့် sign** |

```bash
# User Delegation SAS ဖန်တီးခြင်း (recommended)
az storage blob generate-sas \
    --account-name mystorageacct2025 \
    --container-name mycontainer \
    --name myfile.txt \
    --permissions r \
    --expiry 2025-12-31T00:00:00Z \
    --auth-mode login \
    --as-user \
    --output tsv
```

#### Access Keys vs SAS vs Entra ID

| Method | Revocation | Granularity | Best For |
|--------|-----------|------------|---------|
| Access Keys | Rotate key | Full account | Admin/automation (with caution) |
| SAS | Expire time / key rotate | Fine-grained | Temporary access sharing |
| Entra ID | Revoke role | Role-based | Applications, users |
| Managed Identity | N/A (no credentials) | Role-based | Azure services accessing storage |

---

### ၂.၄ Azure Files and File Sync

#### Azure Files

Azure Files သည် SMB 3.0 (Windows, Linux, macOS), NFS 4.1 (Linux/macOS) protocols ဖြင့် mount ပြုလုပ်နိုင်သော managed file shares ဖြစ်သည်

```bash
# File share ဖန်တီးခြင်း
az storage share create \
    --name myfileshare \
    --account-name mystorageacct2025 \
    --quota 100

# Linux တွင် mount ပြုလုပ်ခြင်း (SMB)
sudo mount -t cifs //mystorageacct2025.file.core.windows.net/myfileshare /mnt/azure \
    -o vers=3.0,username=mystorageacct2025,password=<key>,dir_mode=0777,file_mode=0777
```

#### Azure File Sync

Azure File Sync သည် on-prem Windows Server ကို Azure Files cloud share နှင့် sync ပြုလုပ်ပေးသည်

```
Architecture:
On-premises Windows Server
    └── Azure File Sync Agent (installed)
        └── Registered Server
            └── Sync Group
                └── Cloud Endpoint (Azure File Share)
                    └── Server Endpoints (local paths)
```

**Cloud Tiering:** Local disk space ကိုချွေတာရန် infrequently accessed files ကို cloud သို့ tier ပြုလုပ်သည် (local တွင် stub file ကျန်ရစ်သည်)

---

### ၂.၅ Data Transfer

#### AzCopy

```bash
# Blob upload
azcopy copy '/local/path/data.csv' \
    'https://mystorageacct2025.blob.core.windows.net/mycontainer/data.csv' \
    --recursive

# Container ကို container သို့ copy
azcopy copy \
    'https://src.blob.core.windows.net/srccontainer' \
    'https://dst.blob.core.windows.net/dstcontainer' \
    --recursive

# SAS token ဖြင့် login မပြုဘဲ
azcopy copy 'localfile.txt' \
    'https://account.blob.core.windows.net/container/file.txt?<SAS_TOKEN>'
```

#### Azure Import/Export Service

Terabyte level data ကို physical drive များဖြင့် ship ပြုလုပ်ပြီး transfer ပြုလုပ်ရာတွင် သုံးသည်

- **Import Job:** Local data → Physical drives → Microsoft datacenter → Azure Storage
- **Export Job:** Azure Storage → Physical drives → Ship to customer

---

<a name="domain-3"></a>
