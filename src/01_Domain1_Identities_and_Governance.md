# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## Domain 1: Manage Azure Identities and Governance
### (20–25%)

---

### ၁.၁ Microsoft Entra ID (formerly Azure Active Directory) Fundamentals

Microsoft Entra ID (Azure AD) သည် Microsoft ၏ cloud-based identity and access management service ဖြစ်သည်။ On-prem Active Directory Domain Services (AD DS) နှင့် တူသော်လည်း cloud-native ဖြစ်ပြီး Kerberos/LDAP မသုံး။ SAML, OAuth 2.0, OpenID Connect ကို သုံးသည်။

#### Tenant နှင့် Directory

| Concept | ရှင်းလင်းချက် |
|---------|-------------|
| **Tenant** | Entra ID ၏ dedicated instance တစ်ခု (organization တစ်ခုနှင့် ဆက်စပ်သည်) |
| **Directory** | Tenant ထဲရှိ users, groups, apps, devices များပါဝင်သောနေရာ |
| **Default Domain** | `<tenantname>.onmicrosoft.com` format ဖြင့် auto-create ဖြစ်သည် |
| **Custom Domain** | `yourcompany.com` ကဲ့သို့ custom domain add နိုင်သည်; DNS TXT record ဖြင့် verify ရသည် |

#### Entra ID SKU (License Tiers)

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| Basic Identity & SSO | ✅ | ✅ | ✅ |
| MFA (per-user) | ✅ | ✅ | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| Self-Service Password Reset | ❌ | ✅ | ✅ |
| Identity Protection | ❌ | ❌ | ✅ |
| Privileged Identity Management (PIM) | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ❌ | ✅ |

---

### ၁.၂ Users and Groups စီမံခန့်ခွဲခြင်း

#### User Types

```
Entra ID Users
├── Member Users (internal organization members)
│   ├── Cloud-only (Entra ID တွင်သာ ရှိသည်)
│   └── Synced (on-prem AD မှ Azure AD Connect ဖြင့် sync)
└── Guest Users (B2B Collaboration)
    └── External users ကို tenant သို့ invite လုပ်ထားသည်
        (၎င်းတို့ own identity provider ဖြင့် login ဝင်သည်)
```

#### User ဖန်တီးခြင်း (Portal)

1. **Entra ID** > **Users** > **New User** > **Create new user**
2. ဖြည့်ရမည်: Username, Display name, Password
3. Optional: Department, Job title, Manager, Usage location (license assign မပြုမီ ဖြည့်ရသည်)

#### User ဖန်တီးခြင်း (PowerShell)

```powershell
# Connect to Entra ID
Connect-AzureAD

# New user ဖန်တီးခြင်း
$PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
$PasswordProfile.Password = "Str0ng@Pass!"
$PasswordProfile.ForceChangePasswordNextLogin = $true

New-AzureADUser `
    -DisplayName "Ko Aung" `
    -PasswordProfile $PasswordProfile `
    -UserPrincipalName "ko.aung@contoso.com" `
    -AccountEnabled $true `
    -MailNickName "ko.aung" `
    -Department "Engineering" `
    -JobTitle "Cloud Engineer"
```

#### User ဖန်တီးခြင်း (Azure CLI)

```bash
az ad user create \
  --display-name "Ko Aung" \
  --user-principal-name ko.aung@contoso.com \
  --password "Str0ng@Pass!" \
  --force-change-password-next-sign-in true
```

#### Bulk Operations

User များကို CSV file ဖြင့် bulk create, bulk delete, bulk invite ပြုလုပ်နိုင်သည်။

**CSV Format (bulk import):**

```csv
version:v1.0
Name [displayName] Required,User name [userPrincipalName] Required,...
Ko Aung,ko.aung@contoso.com,...
Ma May,ma.may@contoso.com,...
```

#### Groups

| Group Type | Dynamic Membership | ရှင်းလင်းချက် |
|-----------|-------------------|-------------|
| **Security Group** | ✅/❌ | Azure resources access management |
| **Microsoft 365 Group** | ✅/❌ | Collaboration (Teams, SharePoint, Exchange) |
| **Assigned** | ❌ | Members ကို manually add |
| **Dynamic User** | ✅ | Attributes (dept, title) ပေါ်မူတည်၍ auto-add |
| **Dynamic Device** | ✅ | Device properties ပေါ်မူတည်၍ auto-add |

**Dynamic Group Rule Example:**

```
(user.department -eq "Engineering") and (user.accountEnabled -eq true)
```

#### Guest Users (B2B)

```powershell
# Guest user invite
New-AzureADMSInvitation `
    -InvitedUserDisplayName "External Partner" `
    -InvitedUserEmailAddress "partner@external.com" `
    -InviteRedirectUrl "https://myapp.com" `
    -SendInvitationMessage $true
```

---

### ၁.၃ Role-Based Access Control (RBAC)

#### RBAC Fundamentals

Azure RBAC သည် resource access ကို granular ဖြင့် ထိန်းချုပ်ရာတွင် သုံးသည်။

**Core Components:**

```
Security Principal (Who?)
├── User
├── Group  
├── Service Principal (App identity)
└── Managed Identity

Role Definition (What actions?)
├── Actions: ["Microsoft.Compute/virtualMachines/*"]
├── NotActions: ["Microsoft.Compute/virtualMachines/delete"]
├── DataActions: ["Microsoft.Storage/storageAccounts/blobServices/containers/blobs/read"]
└── NotDataActions: [...]

Scope (Where?)
├── Management Group
├── Subscription
├── Resource Group
└── Resource (most specific)
```

#### Built-in Roles (အဓိကသိသင့်သောများ)

| Role | ကွာခြားချက် |
|------|------------|
| **Owner** | Full access + can manage access to others |
| **Contributor** | Full access to resources, but **cannot** manage access |
| **Reader** | View only; no modifications |
| **User Access Administrator** | Access ကိုသာ manage ပြုလုပ်နိုင်; resource ကို မ manage |
| **Virtual Machine Contributor** | VM manage ပြုလုပ်နိုင်; network/storage ကို separate access လိုသည် |
| **Storage Blob Data Contributor** | Blob storage data read/write/delete; storage account manage မရ |
| **Network Contributor** | Network resources manage ပြုလုပ်နိုင် |

> ⚠️ **Exam Trap:** Owner နှင့် Contributor ၏ ကွာခြားချက် — Owner သာ access assignment ပြုလုပ်နိုင်သည်; Contributor မပြုနိုင်

#### Custom Roles

```json
{
  "Name": "VM Operator",
  "Description": "Can start and stop VMs",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId}"
  ]
}
```

```powershell
# Custom role ဖန်တီးခြင်း
New-AzRoleDefinition -InputFile "vm-operator-role.json"

# Role assign ပြုလုပ်ခြင်း
New-AzRoleAssignment `
    -SignInName "ko.aung@contoso.com" `
    -RoleDefinitionName "VM Operator" `
    -ResourceGroupName "rg-production"
```

#### Effective Permissions စစ်ဆေးခြင်း

```bash
# User ၏ assignments ကြည့်ရန်
az role assignment list --assignee ko.aung@contoso.com --output table

# Scope တစ်ခုအတွင်း assignments ကြည့်ရန်
az role assignment list \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-prod \
    --output table
```

> 📌 **Key Concept:** Deny assignments ကို Azure Policy မှ configure ပြုလုပ်နိုင်သည်; explicit deny ရှိပါက allow role assignments များကို override ပြုလုပ်သည်

---

### ၁.၄ Subscriptions and Governance

#### Azure Management Hierarchy

```
Azure Account (Billing Account)
└── Management Groups
    └── Subscriptions
        └── Resource Groups
            └── Resources
```

**Management Group အကျိုးကျေးဇူး:**
- Multiple subscriptions ကို policy/RBAC ဖြင့် centrally govern ပြုလုပ်နိုင်
- Organization hierarchy ကို reflect ပြုလုပ်နိုင် (Corp → Divisions → BUs)
- Root Management Group = tenant ၏ top level

#### Azure Policy

Azure Policy သည် resource compliance ကို enforce ပြုလုပ်ရာတွင် သုံးသည်။

**Policy Effects (Effect များသိသင့်):**

| Effect | ရှင်းလင်းချက် |
|--------|-------------|
| **Deny** | Non-compliant request ကို block ပြုလုပ်သည် |
| **Audit** | Non-compliant resources ကို log ထဲ record ရိုက်သည်; block မပြုလုပ် |
| **Append** | Resource properties ကို request မပြုမီ add ပြုလုပ်သည် |
| **DeployIfNotExists** | Condition မပြည့်မှ related resource deploy ပြုလုပ်သည် |
| **AuditIfNotExists** | Condition မပြည့်မှ log ထဲ record ရိုက်သည် |
| **Modify** | Resource properties ကို create/update တွင် add/modify/remove ပြုလုပ်သည် |
| **Disabled** | Policy effect ကို disable ပြုလုပ်သည် (testing) |

**Common Built-in Policy Examples:**

```
"Allowed locations" → Resources ကို specific region တွင်သာ ဖန်တီးခွင့်ပြုသည်
"Allowed VM SKUs" → VM size limit ချပေးသည်
"Require a tag and its value" → Resource tag enforce ပြုလုပ်သည်
"Inherit a tag from the resource group" → RG tag ကို resource သို့ propagate ပြုလုပ်သည်
```

**Policy Initiative (Policy Set):**

Policy ၁ ခုစီကို individual assign မပြုဘဲ related policies များကို Initiative (set) တစ်ခုအဖြစ် group ပြုလုပ်ပြီး single assignment ဖြင့် apply ပြုလုပ်နိုင်သည်။

```bash
# Policy assignment ဖန်တီးခြင်း
az policy assignment create \
    --name "AllowedLocations" \
    --policy "e56962a6-4747-49cd-b67b-bf8b01975c4f" \
    --params '{"listOfAllowedLocations":{"value":["eastasia","southeastasia"]}}' \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-prod
```

#### Resource Locks

Resource Locks သည် accidental deletion/modification ကို ကာကွယ်သည်။

| Lock Type | ရှင်းလင်းချက် |
|-----------|-------------|
| **CanNotDelete** | Read + Modify ပြုလုပ်နိုင်သည်; Delete မပြုနိုင် |
| **ReadOnly** | Read only; Modify နှင့် Delete မပြုနိုင် |

> ⚠️ **Important:** ReadOnly lock သည် VM ကို stop/start လုပ်ခြင်းပင် block ပြုလုပ်သည်

```powershell
# Resource Group တစ်ခုတွင် lock ချထားခြင်း
New-AzResourceLock `
    -LockName "ProductionLock" `
    -LockLevel CanNotDelete `
    -ResourceGroupName "rg-production" `
    -LockNotes "Production resources - do not delete"

# Lock ဖျက်ရန်
Remove-AzResourceLock `
    -LockName "ProductionLock" `
    -ResourceGroupName "rg-production"
```

#### Tags

Tags သည် resources ကို categorize ပြုလုပ်ရာတွင် metadata အဖြစ် သုံးသည် (cost tracking, environment, team)

```bash
# Resource Group ကို tag ပေးခြင်း
az group update \
    --name rg-production \
    --tags Environment=Production CostCenter=Engineering Owner=DevTeam

# Tag ဖြင့် resources ကို filter ပြုလုပ်ခြင်း
az resource list --tag Environment=Production --output table
```

> 📌 **Key Points:**
> - Tags ကို resource group မှ resources သို့ automatically inherit မပြုလုပ်
> - Azure Policy မှ "Inherit a tag from resource group" effect ဖြင့် inherit enforce ပြုလုပ်နိုင်
> - Resource တစ်ခုတွင် maximum **50 tags** ပေးနိုင်သည်

#### Cost Management

```bash
# Current month cost ကြည့်ရန် (CLI)
az consumption usage list \
    --billing-period-name 202501 \
    --output table

# Budget ဖန်တီးခြင်း
az consumption budget create \
    --budget-name "MonthlyBudget" \
    --amount 1000 \
    --time-grain Monthly \
    --start-date 2025-01-01 \
    --end-date 2025-12-31 \
    --resource-group rg-production
```

#### Self-Service Password Reset (SSPR)

SSPR ကို enable ပြုလုပ်ရာတွင် Entra ID P1 license လိုသည်။

```
SSPR Authentication Methods:
├── Email (external email)
├── Mobile Phone (SMS/voice)
├── Authenticator App
├── Office Phone
└── Security Questions (least recommended)
```

**Key Configuration:**
- **Scope:** None / Selected (specific group) / All
- **Number of methods required to reset:** 1 or 2

---

<a name="domain-2"></a>
