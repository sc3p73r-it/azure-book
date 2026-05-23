# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## အခန်း ၈ — စာမေးပွဲ ပြင်ဆင်ရာတွင် အကြံပြုချက်များ

### Exam Day Checklist

- [ ] Microsoft Learn Practice Assessment ကို score 80%+ ရသည်အထိ ကျင့်ပါ
- [ ] Azure Free Account တွင် lab exercises အားလုံး ကိုယ်တိုင် ပြုလုပ်ကြည့်ပါ
- [ ] NSG rule priority logic ကို ကောင်းကောင်းနားလည်ပါ
- [ ] RBAC roles (Owner/Contributor/Reader) ၏ ကွာခြားချက်ကို မှတ်ပါ
- [ ] VNet peering non-transitive ဆိုသည်ကို နားလည်ပါ
- [ ] Load Balancer vs Application Gateway vs Traffic Manager ကွာခြားချက် မှတ်ပါ
- [ ] Storage replication types (LRS/ZRS/GRS/RA-GRS) ၏ SLA မှတ်ပါ
- [ ] VM availability options (Availability Set vs Zone) SLA မှတ်ပါ
- [ ] `deallocate` vs `stop` ကွာခြားချက် မှတ်ပါ

### Common Exam Traps

| Trap | မှန်ကန်သောအဖြေ |
|------|--------------|
| "Stop billing for a VM" | `deallocate` — `stop` မဟုတ် |
| "Block deletion but allow modification" | CanNotDelete lock |
| "Block all changes" | ReadOnly lock |
| "Limit resource creation to specific regions" | Azure Policy (Allowed Locations) |
| "Apply policy to all subscriptions at once" | Management Group policy assignment |
| "A→B peer and B→C peer — can A communicate with C?" | **No** (non-transitive) |
| "ReadOnly lock blocks VM start?" | **Yes** |
| "Owner vs Contributor" | Owner can manage access; Contributor cannot |
| "SAS vs Access Key" | SAS is time-limited; Key is permanent until rotated |
| "NSG priority 100 vs 200" | 100 wins (lower = higher priority) |

### Domain-Specific Study Priorities

```
Domain 1 — Identities & Governance:
  ✅ RBAC roles differences (Owner/Contributor/Reader/UAAdmin)
  ✅ Custom role creation
  ✅ Azure Policy effects (especially Deny vs Audit)
  ✅ Management Groups hierarchy
  ✅ Entra ID P1 vs P2 features

Domain 2 — Storage:
  ✅ Storage replication types and SLAs
  ✅ Blob access tiers and lifecycle
  ✅ SAS token types (especially User Delegation SAS)
  ✅ Storage network access control options

Domain 3 — Compute:
  ✅ Availability Set vs Availability Zones SLA
  ✅ VM deallocate vs stop
  ✅ VMSS autoscale
  ✅ ARM template / Bicep basics
  ✅ App Service tiers and deployment slots

Domain 4 — Networking (HARDEST):
  ✅ NSG rule priority and default rules
  ✅ VNet peering (non-transitive, no overlap)
  ✅ Load Balancer SKUs (Basic vs Standard)
  ✅ LB vs App Gateway vs Traffic Manager vs Front Door
  ✅ VPN Gateway vs ExpressRoute comparison
  ✅ Service Tags usage

Domain 5 — Monitor & Backup:
  ✅ Azure Monitor data types (Metrics vs Logs)
  ✅ KQL basic queries
  ✅ Alert rule configuration
  ✅ Recovery Services Vault and backup policies
  ✅ ASR concepts (RPO/RTO)
```

---

<a name="study-plan"></a>
## အခန်း ၉ — 8-Week Study Plan

ဤ Study Plan သည် Linux/SysOps background ရှိသောသူ (IT experience ၃+ years) အတွက် ရေးဆွဲထားသည်

### Week 1: Foundation & Setup

- [ ] Azure Free Account ဖန်တီးပါ ($200 credit)
- [ ] Azure Portal, CLI, PowerShell ကို install/configure ပြုလုပ်ပါ
- [ ] Domain 1: Entra ID Users, Groups, RBAC (Lab 1 ပြုလုပ်ပါ)
- [ ] Microsoft Learn: [AZ-104 Prerequisites](https://learn.microsoft.com/en-us/training/paths/az-104-administrator-prerequisites/) ၂ modules

### Week 2: Governance & Compliance

- [ ] Domain 1 (cont): Azure Policy, Management Groups, Tags, Cost Management
- [ ] RBAC custom roles practice
- [ ] Policy Initiative configuration
- [ ] Resource Locks testing
- [ ] Microsoft Learn: Manage identities and governance path

### Week 3: Storage Deep Dive

- [ ] Domain 2: Storage accounts, replication, network rules
- [ ] Blob tiers, lifecycle management, AzCopy
- [ ] Azure Files, File Sync
- [ ] SAS tokens (all types)
- [ ] Lab 2 ပြုလုပ်ပါ
- [ ] Microsoft Learn: Implement storage

### Week 4: Compute Resources

- [ ] Domain 3: VM creation, sizing, availability
- [ ] VM Disks (managed, encryption)
- [ ] VMSS, autoscale
- [ ] ARM Templates + Bicep
- [ ] ACI, AKS basics, ACR
- [ ] App Service, deployment slots
- [ ] Microsoft Learn: Deploy and manage Azure compute

### Week 5: Networking (Part 1)

- [ ] Domain 4: VNets, subnets, address planning
- [ ] NSG (priority, default rules, Service Tags)
- [ ] VNet Peering (local, global)
- [ ] Azure DNS (public, private)
- [ ] Lab 3 ပြုလုပ်ပါ
- [ ] Microsoft Learn: Implement virtual networking

### Week 6: Networking (Part 2)

- [ ] Domain 4 (cont): Load Balancer (Basic vs Standard)
- [ ] Application Gateway, WAF
- [ ] Traffic Manager, Front Door comparison
- [ ] VPN Gateway, ExpressRoute
- [ ] Azure Firewall, Bastion
- [ ] Network Watcher diagnostics
- [ ] Microsoft Learn: Connect services together

### Week 7: Monitor, Backup & Review

- [ ] Domain 5: Azure Monitor, Metrics, Log Analytics
- [ ] KQL queries practice
- [ ] Alerts and Action Groups
- [ ] Azure Backup, Recovery Services Vault
- [ ] Azure Site Recovery concepts
- [ ] **First full Practice Assessment** → weak areas မှတ်ပါ

### Week 8: Final Revision & Exam

- [ ] Weak areas ကို ပြန်လေ့လာပါ
- [ ] **3 full mock exams** (Whizlabs, MeasureUp, or Microsoft official)
- [ ] Exam traps list ပြန်သုံးသပ်ပါ
- [ ] Exam day logistics ပြင်ဆင်ပါ (Pearson VUE scheduling)
- [ ] **Exam day: REST + confidence!**

---

## အပ်ဒိတ်နှင့် ကိုးကားအချက်များ

| Resource | Link |
|----------|------|
| Official Certification Page | https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/ |
| AZ-104 Study Guide (Official) | https://aka.ms/AZ104-StudyGuide |
| Microsoft Learn Path | https://learn.microsoft.com/en-us/training/paths/az-104-administrator-prerequisites/ |
| Practice Assessment (Free) | https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/practice/assessment |
| Azure CLI Docs | https://docs.microsoft.com/en-us/cli/azure/ |
| Azure PowerShell Docs | https://docs.microsoft.com/en-us/powershell/azure/ |
| Azure Architecture Center | https://docs.microsoft.com/en-us/azure/architecture/ |
| Schedule Exam | https://learn.microsoft.com/en-us/credentials/certifications/schedule-through-pearson-vue |

---

> **💡 Final Note:** AZ-104 သည် theoretical knowledge ထက် hands-on experience ကို test ပြုလုပ်သော exam ဖြစ်သည်။ Concepts တိုင်းကို Azure Portal တွင် ကိုယ်တိုင်ပြုလုပ်ကြည့်ခြင်းသည် pass ဖြစ်နိုင်ချေကို အများကြီးတိုးစေသည်။ **Good luck! 🚀**
