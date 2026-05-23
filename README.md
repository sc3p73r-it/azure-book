# AZ-104: Microsoft Azure Administrator Associate
## စာအုပ်မာတိကာ (Table of Contents)

> **Certification:** Microsoft Certified: Azure Administrator Associate  
> **Exam Code:** AZ-104  
> **Skills Outline:** April 17, 2026 (Latest)  
> **Passing Score:** 700 / 1000 · **Duration:** 100 min · **Questions:** 40–55

---

## ဖိုင်များ စာရင်း

| ဖိုင် | အကြောင်းအရာ | Domain Weight |
|------|------------|--------------|
| [00 — Introduction & Overview](#00) | Exam format, question types, domain weights | — |
| [01 — Domain 1: Identities & Governance](#01) | Entra ID, RBAC, Policy, Subscriptions | 20–25% |
| [02 — Domain 2: Storage](#02) | Storage Accounts, Blob, Files, SAS | 15–20% |
| [03 — Domain 3: Compute](#03) | VMs, VMSS, Containers, App Service | 20–25% |
| [04 — Domain 4: Networking](#04) | VNet, NSG, LB, VPN, Firewall | 15–20% |
| [05 — Domain 5: Monitor & Backup](#05) | Azure Monitor, Log Analytics, ASR | 10–15% |
| [06 — Hands-On Labs](#06) | Lab exercises (Identity, Storage, Networking) | — |
| [07 — CLI & PowerShell Reference](#07) | Quick command reference sheet | — |
| [08 — Exam Tips & Study Plan](#08) | Exam traps, 8-week study plan | — |

---

<a name="00"></a>
## 📄 00 — Introduction & Exam Overview
**ဖိုင်:** `00_Introduction_and_Overview.md`

- စာအုပ်အကြောင်း — ဤ guide ကို ဘယ်သူ့အတွက် ရေးသားသနည်း
- လိုအပ်သော Background Knowledge
- Exam အချက်အလက်များ (Duration, Score, Price, Languages)
- Question Types (MCQ, Case Study, Drag & Drop, Hot Area)
- Domain Weight Chart
- Certification Path (AZ-900 → AZ-104 → AZ-305/AZ-500/AZ-700)

---

<a name="01"></a>
## 📄 01 — Domain 1: Manage Azure Identities and Governance
**ဖိုင်:** `01_Domain1_Identities_and_Governance.md` · **Weight: 20–25%**

### ၁.၁ Microsoft Entra ID Fundamentals
- Tenant, Directory, Default/Custom Domain
- Entra ID License Tiers (Free / P1 / P2) feature comparison

### ၁.၂ Users and Groups စီမံခန့်ခွဲခြင်း
- User Types (Member, Guest/B2B, Cloud-only, Synced)
- User ဖန်တီးခြင်း (Portal, PowerShell, Azure CLI)
- Bulk Operations (CSV import/delete/invite)
- Group Types (Security, M365, Assigned, Dynamic)
- Dynamic Group membership rules
- Guest User (B2B) invite

### ၁.၃ Role-Based Access Control (RBAC)
- Security Principal, Role Definition, Scope hierarchy
- Built-in Roles (Owner / Contributor / Reader / UAAdmin)
- Custom Role ဖန်တီးခြင်း (JSON + PowerShell)
- Effective Permissions စစ်ဆေးခြင်း (CLI)
- Deny Assignments

### ၁.၄ Subscriptions and Governance
- Azure Management Hierarchy (Account → MG → Sub → RG → Resource)
- Azure Policy effects (Deny, Audit, Append, DeployIfNotExists, Modify)
- Policy Initiative (Policy Set)
- Resource Locks (CanNotDelete vs ReadOnly)
- Tags (limits, inheritance, Azure Policy enforcement)
- Cost Management & Budgets
- Self-Service Password Reset (SSPR)

---

<a name="02"></a>
## 📄 02 — Domain 2: Implement and Manage Storage
**ဖိုင်:** `02_Domain2_Storage.md` · **Weight: 15–20%**

### ၂.၁ Azure Storage Accounts
- Storage Account Types (GPv2, BlockBlobStorage, FileStorage)
- Replication Options (LRS / ZRS / GRS / GZRS / RA-GRS / RA-GZRS) + SLA
- Storage Account ဖန်တီးခြင်း (CLI)
- Network Access Control (Public / Selected Networks / Private Endpoint)

### ၂.၂ Blob Storage
- Blob Types (Block, Append, Page)
- Access Tiers (Hot / Cool / Cold / Archive) + costs
- Archive Rehydration (Standard 15h vs High Priority 1h)
- Lifecycle Management Policy (JSON)

### ၂.၃ Storage Access နှင့် Security
- SAS Token Types (Account, Service, User Delegation)
- Access Keys vs SAS vs Entra ID vs Managed Identity comparison
- User Delegation SAS ဖန်တီးခြင်း (recommended)

### ၂.၄ Azure Files and File Sync
- Azure Files (SMB 3.0, NFS 4.1) + Linux mount command
- Azure File Sync architecture (Agent → Sync Group → Cloud Endpoint)
- Cloud Tiering concept

### ၂.၅ Data Transfer
- AzCopy commands (upload, copy, SAS-based)
- Azure Import/Export Service (Import Job, Export Job)

---

<a name="03"></a>
## 📄 03 — Domain 3: Deploy and Manage Azure Compute Resources
**ဖိုင်:** `03_Domain3_Compute.md` · **Weight: 20–25%**

### ၃.၁ Virtual Machines
- VM Size Families (A, B, D, E, F, N series)
- VM ဖန်တီး၊ start/stop/deallocate (CLI)
- `stop` vs `deallocate` ကွာခြားချက် (billing!)
- Availability Options: No Redundancy / Availability Set / AZ / VMSS+Zones
- Fault Domain vs Update Domain

### ၃.၂ VM Disks
- Disk Types (Ultra / Premium SSD v2 / Premium SSD / Standard SSD / HDD)
- Managed Disks (always use)
- Azure Disk Encryption (Key Vault + CLI)

### ၃.၃ VM Scale Sets (VMSS)
- VMSS ဖန်တီးခြင်း (zones, upgrade policy)
- Autoscale rules (CPU-based scale out/in)
- Orchestration Modes (Uniform vs Flexible)

### ၃.၄ ARM Templates and Bicep
- ARM Template structure (parameters, variables, resources, outputs)
- Bicep syntax + deploy command
- Template functions (resourceGroup().location, uniqueString)

### ၃.၅ Containers on Azure
- ACI (Azure Container Instances) — serverless containers
- AKS (Azure Kubernetes Service) — managed Kubernetes
- ACR (Azure Container Registry) — private image registry
- AKS ↔ ACR integration (attach-acr)

### ၃.၆ App Service
- App Service Plan Tiers (Free/Shared/Basic/Standard/Premium/Isolated)
- Web App ဖန်တီးပြီး runtime configure ပြုလုပ်ခြင်း
- Deployment Slots (staging → production swap)

---

<a name="04"></a>
## 📄 04 — Domain 4: Implement and Manage Virtual Networking
**ဖိုင်:** `04_Domain4_Networking.md` · **Weight: 15–20% ⚠️ Hardest Domain**

### ၄.၁ Virtual Network (VNet) Fundamentals
- Address space design, subnet planning
- Azure reserved IPs per subnet (5 addresses)
- VNet + Subnet ဖန်တီးခြင်း (CLI)

### ၄.၂ Network Security Groups (NSG)
- Rule properties (Priority, Source, Protocol, Direction, Action)
- Default inbound/outbound rules
- NSG ဖန်တီး၊ rules ထည့်ပြီး subnet associate (CLI)
- Service Tags (Internet, VirtualNetwork, AzureLoadBalancer, Storage ...)

### ၄.၃ VNet Peering
- Local Peering vs Global VNet Peering
- Bidirectional peering setup (CLI)
- Non-transitive behavior (A↔B, B↔C → A↔C မဖြစ်)
- Overlapping address space restriction

### ၄.၄ Azure DNS
- Public DNS Zone (A record, CNAME)
- Private DNS Zone + VNet link (auto-registration)

### ၄.၅ Load Balancing
- Azure LB vs App Gateway vs Traffic Manager vs Front Door comparison
- Standard vs Basic LB SKU differences
- Load Balancer ဖန်တီး၊ health probe, LB rule (CLI)

### ၄.၆ VPN Gateway and ExpressRoute
- VPN Gateway SKUs (throughput comparison)
- GatewaySubnet requirement
- VPN Gateway ဖန်တီးခြင်း (CLI)
- VPN vs ExpressRoute feature comparison (speed, reliability, encryption)

### ၄.၇ Azure Bastion and Firewall
- Bastion (AzureBastionSubnet /26 requirement, Portal SSH/RDP)
- Azure Firewall rule types (Network, Application, NAT)
- Hub-Spoke topology ၌ Firewall placement

---

<a name="05"></a>
## 📄 05 — Domain 5: Monitor and Maintain Azure Resources
**ဖိုင်:** `05_Domain5_Monitor_and_Backup.md` · **Weight: 10–15%**

### ၅.၁ Azure Monitor
- Monitor data types (Platform Metrics, Custom Metrics, Activity Logs, Resource Logs)
- Metrics query (CLI)

### ၅.၂ Log Analytics Workspace
- Workspace ဖန်တီး (PerGB2018 SKU, retention)
- KQL queries (Perf, SecurityEvent, Heartbeat)

### ၅.၃ Alerts
- Metric Alert ဖန်တီးခြင်း (CPU > 80%)
- Action Group (Email + SMS)

### ၅.၄ Azure Backup
- Recovery Services Vault ဖန်တီးခြင်း
- VM backup enable, on-demand backup, restore
- Backup Policy settings (frequency, retention, instant restore)

### ၅.၅ Azure Site Recovery (ASR)
- Replication architecture (source → target region)
- RPO vs RTO concepts
- Failover workflow

---

<a name="06"></a>
## 📄 06 — Hands-On Lab Exercises
**ဖိုင်:** `06_Hands_On_Labs.md`

### Lab 1: Identity and Access Management
Resource Group ဖန်တီး → User ဖန်တီး → Security Group ဖန်တီး → Member add → RBAC assign

### Lab 2: Storage Configuration
Storage Account ဖန်တီး → Container ဖန်တီး → Blob upload → SAS URL generate

### Lab 3: Virtual Network Setup
Hub VNet + Spoke VNets ဖန်တီး → Subnets configure → Bidirectional VNet Peering setup

---

<a name="07"></a>
## 📄 07 — CLI & PowerShell Quick Reference
**ဖိုင်:** `07_CLI_Reference.md`

- Resource Group commands (create, list, delete, show)
- VM commands (list, show, start, deallocate, restart, resize, extension)
- Network commands (effective NSG rules, effective routes, IP flow verify)

---

<a name="08"></a>
## 📄 08 — Exam Tips & Study Plan
**ဖိုင်:** `08_Exam_Tips_and_Study_Plan.md`

### Exam Day Checklist
Pass မဖြစ်ခင် ပြင်ဆင်ထားသင့်သော ၁၀ items

### Common Exam Traps
`stop` vs `deallocate` · Lock types · VNet peering transitive · Owner vs Contributor · NSG priority logic နှင့် အခြား ဘာသာရပ်ဆိုင်ရာ traps များ

### Domain-Specific Study Priorities
Domain တိုင်းအတွက် ဦးစားပေး မှတ်သင့်သော topics များ

### 8-Week Study Plan
| Week | Focus |
|------|-------|
| Week 1 | Azure setup + Entra ID Users/Groups/RBAC |
| Week 2 | Azure Policy, Governance, Management Groups |
| Week 3 | Storage deep dive + Lab 2 |
| Week 4 | VM, VMSS, Containers, ARM/Bicep, App Service |
| Week 5 | VNet, NSG, Peering, DNS + Lab 3 |
| Week 6 | LB, App Gateway, VPN, ExpressRoute, Firewall |
| Week 7 | Monitor, Backup, ASR + First Mock Exam |
| Week 8 | Weak area review + 3 full mock exams + Exam day |

---

## 🔗 သုံးသင့်သော Official Resources

| Resource | Link |
|----------|------|
| Certification Page | https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/ |
| AZ-104 Study Guide (Official PDF) | https://aka.ms/AZ104-StudyGuide |
| Free Practice Assessment | https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/practice/assessment |
| Microsoft Learn Path | https://learn.microsoft.com/en-us/training/paths/az-104-administrator-prerequisites/ |
| Schedule Exam (Pearson VUE) | https://learn.microsoft.com/en-us/credentials/certifications/schedule-through-pearson-vue |
| Azure Free Account | https://azure.microsoft.com/en-us/free/ |
