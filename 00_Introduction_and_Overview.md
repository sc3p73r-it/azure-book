# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## အခန်း ၀ — ဤစာအုပ်အကြောင်း

ဤ Study Guide သည် SysOps၊ Linux၊ Containers နှင့် Networking ကို အကျွမ်းတဝင်ရှိသောသူများ Azure Cloud Administration သို့ ကူးပြောင်းရာတွင် ထိရောက်သောလမ်းညွှန်မှုပေးနိုင်ရန် ရေးသားထားသည်။ Microsoft ၏ Official Skills Outline ကို အချက်ကျကျ ဖော်ပြမည့်အပြင် real-world command များ၊ architect patterns နှင့် exam gotchas များပါ ထည့်သွင်းထားသည်။

### ဤစာအုပ်ကို ဘယ်သူအတွက်ရေးသားသနည်း

- Linux/VMware/on-prem sysadmin မှ cloud admin သို့ transition ဖြစ်ချင်သူ
- AWS/GCP experience ရှိပြီး Azure ထပ်မံသင်ယူချင်သူ
- AZ-104 ကို 4–8 ပတ်အတွင်း pass ဖြစ်ချင်သူ
- Azure Portal, CLI, PowerShell ကို professional အဆင့် သုံးနိုင်ချင်သူ

### သင်မှိုင်ပတ်သင့်သည့် Background Knowledge

- Networking fundamentals (IP addressing, subnetting, routing, DNS, firewalls)
- Virtualization concepts (hypervisors, VMs, storage)
- OS administration (Windows Server, Linux)
- Scripting basics (Bash, PowerShell)
- Identity concepts (LDAP, Active Directory)

---

<a name="chapter-1"></a>
## အခန်း ၁ — AZ-104 စာမေးပွဲ Overview

### Exam ဆိုင်ရာ အချက်အလက်များ

| Item | Detail |
|------|--------|
| **Exam Code** | AZ-104 |
| **Certification** | Microsoft Certified: Azure Administrator Associate |
| **Level** | Intermediate (Associate) |
| **Duration** | 100 minutes |
| **Questions** | 40–55 (approx.) |
| **Passing Score** | 700/1000 (scaled scoring) |
| **Price** | USD $165 (region ပေါ်မူတည်၍ ကွာခြားနိုင်) |
| **Languages** | English, Japanese, Korean, Chinese (Simplified/Traditional), French, Spanish, German, Italian, Portuguese (Brazil) |
| **Renewal** | 12 လတစ်ကြိမ် (free online assessment မှတဆင့်) |
| **Provider** | Pearson VUE (Onsite or Online Proctored) |

### Question Types

AZ-104 တွင် question type များ ကွဲပြားသည်:

- **Multiple Choice (Single Answer)** — အဖြေ ၁ ခုသာ မှန်သည်
- **Multiple Choice (Multiple Answers)** — အဖြေ ၂+ ခု မှန်နိုင်သည်
- **Case Studies** — ရှည်လျားသော scenario တစ်ခုအပေါ် မှတ်ဆင့်မေးသော မေးခွန်းအစုံ
- **Drag and Drop** — items များကို ကိုက်ညီသောနေရာသို့ ဆွဲထည့်ရသည်
- **Hot Area (Clickable UI)** — Azure Portal screenshot ပေါ်တွင် နှိပ်ဖြေရသည်
- **Order the Steps** — အဆင့်ဆင့် ပြုလုပ်ရသော process ကို မှန်ကန်သောအစဉ်အတိုင်း စီထားရသည်

> ⚠️ **Important:** Microsoft role-based exams တွင် exam စတင်ပြီးနောက် Microsoft Learn documentation ကို open-book access ရနိုင်သည်၊ သို့သော် extra time ထပ်မပေး၊ Q&A နှင့် Practice Assessments sections များ ဝင်ခွင့်မရ — ဒါကြောင့် full preparation မပြုဘဲ documentation ပေါ် မမှီခိုသင့်

### Exam Domain Weights

```
┌─────────────────────────────────────────────────────┬──────────────┐
│ Domain                                              │ Weight       │
├─────────────────────────────────────────────────────┼──────────────┤
│ 1. Manage Azure Identities and Governance           │ 20–25%       │
│ 2. Implement and Manage Storage                     │ 15–20%       │
│ 3. Deploy and Manage Azure Compute Resources        │ 20–25%       │
│ 4. Implement and Manage Virtual Networking          │ 15–20%       │
│ 5. Monitor and Maintain Azure Resources             │ 10–15%       │
└─────────────────────────────────────────────────────┴──────────────┘
```

**Study Time Allocation:**
- Domain 1 + Domain 3 = exam ၏ 40–50% → ဤ domain ၂ ခုကို **60% study time** သုံးပါ
- Domain 4 = technically hardest ဖြစ်သည်ဟု candidates အများစုက ဆိုသည်

### Certification Path

```
AZ-900 (Fundamentals) ── ➤ AZ-104 (Associate Admin) ── ➤ AZ-305 (Expert Architect)
                                      │
                                      ├──➤ AZ-500 (Security Engineer)
                                      └──➤ AZ-700 (Network Engineer)
```

---

<a name="domain-1"></a>
