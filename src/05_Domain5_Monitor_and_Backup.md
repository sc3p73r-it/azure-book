# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## Domain 5: Monitor and Maintain Azure Resources
### (10–15%)

---

### ၅.၁ Azure Monitor

Azure Monitor သည် Azure resources ၏ telemetry data (metrics, logs) ကို collect, analyze, act ပြုလုပ်ရာ central platform ဖြစ်သည်

#### Monitor Data Types

```
Azure Monitor Data Types:
├── Metrics (numerical time-series data)
│   ├── Platform Metrics (automatic, free)
│   └── Custom Metrics (application-defined)
└── Logs (structured/unstructured text data)
    ├── Activity Logs (subscription-level operations)
    ├── Resource Logs (resource-level diagnostic data)
    └── Entra ID Logs (sign-in, audit)
```

#### Metrics Explorer

```bash
# Metric query (CLI)
az monitor metrics list \
    --resource /subscriptions/{sub-id}/resourceGroups/rg-compute/providers/Microsoft.Compute/virtualMachines/vm-web \
    --metric "Percentage CPU" \
    --interval PT5M \
    --start-time 2025-01-01T00:00:00Z \
    --end-time 2025-01-01T23:59:59Z
```

---

### ၅.၂ Log Analytics Workspace

Log Analytics Workspace သည် Azure Monitor Logs ကို store, query ပြုလုပ်ရာ repository ဖြစ်သည်

```bash
# Log Analytics Workspace ဖန်တီးခြင်း
az monitor log-analytics workspace create \
    --resource-group rg-monitoring \
    --workspace-name law-production \
    --location southeastasia \
    --sku PerGB2018 \
    --retention-time 90
```

#### KQL (Kusto Query Language) — Basic Queries

```kql
// Last 1 hr ၏ VM performance data
Perf
| where TimeGenerated > ago(1h)
| where Computer == "vm-webserver"
| where CounterName == "% Processor Time"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart

// Failed login attempts
SecurityEvent
| where TimeGenerated > ago(24h)
| where EventID == 4625  // Failed logon
| summarize FailedLogins = count() by Computer, Account
| order by FailedLogins desc

// VM heartbeat check (machines that stopped reporting)
Heartbeat
| where TimeGenerated > ago(1h)
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(5m)
```

---

### ၅.၃ Alerts

```bash
# Metric Alert ဖန်တီးခြင်း (CPU > 80%)
az monitor metrics alert create \
    --name "High CPU Alert" \
    --resource-group rg-monitoring \
    --scopes /subscriptions/{sub-id}/resourceGroups/rg-compute/providers/Microsoft.Compute/virtualMachines/vm-web \
    --condition "avg Percentage CPU > 80" \
    --window-size 5m \
    --evaluation-frequency 1m \
    --action /subscriptions/{sub-id}/resourceGroups/rg-monitoring/providers/microsoft.insights/actionGroups/ag-ops-team \
    --severity 2

# Action Group ဖန်တီးခြင်း (Email + SMS)
az monitor action-group create \
    --resource-group rg-monitoring \
    --name ag-ops-team \
    --short-name ops \
    --email-receiver name=OpsEmail email=ops@contoso.com \
    --sms-receiver name=OpsSMS country-code=65 phone-number=87654321
```

---

### ၅.၄ Azure Backup

#### Recovery Services Vault

```bash
# Recovery Services Vault ဖန်တီးခြင်း
az backup vault create \
    --resource-group rg-backup \
    --name rsv-production \
    --location southeastasia

# VM backup enable
az backup protection enable-for-vm \
    --resource-group rg-backup \
    --vault-name rsv-production \
    --vm vm-webserver \
    --policy-name DefaultPolicy

# On-demand backup
az backup protection backup-now \
    --resource-group rg-backup \
    --vault-name rsv-production \
    --container-name IaasVMContainer;iaasvmcontainerv2;rg-compute;vm-webserver \
    --item-name vm-webserver \
    --backup-management-type AzureIaasVM \
    --retain-until 31-01-2026

# VM restore
az backup restore restore-disks \
    --resource-group rg-backup \
    --vault-name rsv-production \
    --container-name IaasVMContainer;iaasvmcontainerv2;rg-compute;vm-webserver \
    --item-name vm-webserver \
    --rp-name <recovery-point-name> \
    --storage-account mystorageacct2025
```

#### Backup Policies

| Setting | Description |
|---------|-------------|
| **Frequency** | Daily (up to 4/day), Weekly |
| **Retention** | Daily (1-180 days), Weekly (1-520 weeks), Monthly/Yearly |
| **Instant Restore** | 1-5 days (snapshot retention for fast recovery) |

---

### ၅.၅ Azure Site Recovery (ASR)

ASR သည် VMs နှင့် physical servers ကို secondary region သို့ replicate ပြုလုပ်ပြီး disaster recovery ပြုလုပ်နိုင်သည်

```
ASR Workflow:
Source Region (Primary)          Target Region (Secondary)
├── VM (vm-webserver)    →       ├── Replica VM (standby)
├── Storage              →       └── Cache Storage
└── VNet                 →           VNet (pre-configured)

RTO: Recovery Time Objective (how quickly systems restored)
RPO: Recovery Point Objective (how much data loss acceptable)
```

```bash
# ASR: Source VM ကို secondary region သို့ replicate enable
az site-recovery protection-container-mapping create \
    --resource-group rg-asr \
    --vault-name rsv-asr \
    --fabric-name fabric-source \
    --protection-container-name container-source \
    --name mapping-primary-secondary \
    --policy-name ReplicationPolicy \
    --target-protection-container /subscriptions/.../targetContainer
```

---

<a name="labs"></a>
