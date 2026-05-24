# Domain 1: Manage Azure Identities and Governance
**Weight: 20–25%**

---

## ၁.၁ Microsoft Entra ID (formerly Azure Active Directory)

### Entra ID ဆိုတာ ကဘာလဲ

Microsoft Entra ID သည် cloud-based identity and access management (IAM) service ဖြစ်ပြီး employees, partners, customers ၏ resources access ကို authenticate/authorize ပြုလုပ်သည်။

**On-prem AD DS နှင့် ကွာခြားချက်:**

| Aspect | On-prem AD DS | Microsoft Entra ID |
|--------|--------------|-------------------|
| Protocol | Kerberos, LDAP, NTLM | SAML 2.0, OAuth 2.0, OpenID Connect |
| Structure | Organizational Units (OUs) | Flat (no OUs, uses groups) |
| Query | LDAP queries | Microsoft Graph API |
| Devices | Domain-joined (GPO) | Azure AD Join, Hybrid Join |
| Trust | Forest/Domain trusts | B2B, B2C |
| Sync | N/A | Azure AD Connect (on-prem → cloud) |

### Tenant, Directory, Subscription

```
Entra ID Tenant (acme.onmicrosoft.com)
│   └── Identity store: users, groups, apps, devices
│
├── Subscription 1 (Production)   ←── billing unit
│       └── Resource Groups → Resources
│
└── Subscription 2 (Development)
        └── Resource Groups → Resources
```

**Key definitions:**

| Term | Definition |
|------|-----------|
| **Tenant** | Dedicated Entra ID instance for one organization |
| **Default Domain** | `<tenantname>.onmicrosoft.com` (auto-created) |
| **Custom Domain** | `yourcompany.com` (DNS TXT record verify လိုသည်) |
| **Directory** | Users, groups, apps, devices ပါဝင်သော container |
| **Subscription** | Azure services billing unit (tenant တစ်ခုနှင့် တွဲသည်) |

**Custom Domain ထည့်သည့် အဆင့်:**
1. Entra ID → Custom domain names → **Add custom domain**
2. Domain name ရိုက်ထည့်ပါ (`contoso.com`)
3. Microsoft က TXT/MX DNS record ပေးသည်
4. Domain registrar (GoDaddy, Namecheap) မှာ DNS record add ပါ
5. Azure Portal မှ **Verify** button နှိပ်ပါ → DNS propagation complete ဖြစ်ရမည်

### Entra ID License Tiers

| Feature | Free | Office 365 | P1 | P2 |
|---------|:----:|:----------:|:--:|:--:|
| User & Group Management | ✅ | ✅ | ✅ | ✅ |
| SSO (up to 10 apps Free, unlimited P1+) | ✅ | ✅ | ✅ | ✅ |
| MFA per-user | ✅ | ✅ | ✅ | ✅ |
| **Conditional Access Policies** | ❌ | ❌ | ✅ | ✅ |
| **SSPR (Self-Service Password Reset)** | ❌ | Cloud only | ✅ | ✅ |
| **Group-based licensing** | ❌ | ❌ | ✅ | ✅ |
| **B2B Collaboration** | Limited | Limited | ✅ | ✅ |
| **Identity Protection** (risk-based) | ❌ | ❌ | ❌ | ✅ |
| **Privileged Identity Management (PIM)** | ❌ | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ❌ | ✅ |

> 📌 **Exam Tips:** P1 = Conditional Access + SSPR ✦ P2 = Identity Protection + PIM + Access Reviews

---

## ၁.၂ Users and Groups စီမံခန့်ခွဲခြင်း

### User Types

```
Entra ID User Types
├── Member Users (internal, type = Member)
│   ├── Cloud-only          ← Entra ID တွင်သာ ရှိ; portal/CLI/PS ဖြင့် create
│   └── Synced              ← On-prem AD မှ Azure AD Connect ဖြင့် sync
│                             Authoritative source = On-prem (portal မှ edit မရ)
│
└── Guest Users (external, type = Guest)
    └── B2B Collaboration   ← Email ဖြင့် invite; own IdP ဖြင့် authenticate
                              (Microsoft, Google, Facebook, any SAML IdP)
```

### User ဖန်တီးခြင်း — Portal Steps

```
Entra ID → Users → + New User → Create new user
  ┌─ Required ─────────────────────────────────┐
  │  User principal name: ko.aung@contoso.com  │
  │  Display name:        Ko Aung              │
  │  Password:            Auto-generate / Set  │
  └────────────────────────────────────────────┘
  ┌─ Optional (Properties tab) ────────────────┐
  │  Job title, Department, Manager            │
  │  Office location, Mobile phone             │
  │  Usage location ← License assign မပြုမီ    │
  │                   ဖြည့်ရမည် (REQUIRED)      │
  └────────────────────────────────────────────┘
```

> ⚠️ **Usage location မဖြည့်ပါက license assign မပြုနိုင်** — Exam တွင် မေးတတ်သည်

### User ဖန်တီးခြင်း — PowerShell

```powershell
# Module install (ပထမဆုံးတစ်ကြိမ်)
Install-Module Microsoft.Graph -Scope CurrentUser
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Password profile object
$pwdProfile = @{
    Password = "TempP@ss123!"
    ForceChangePasswordNextSignIn = $true
}

# User create
New-MgUser `
    -DisplayName "Ko Aung" `
    -UserPrincipalName "ko.aung@contoso.com" `
    -AccountEnabled `
    -MailNickname "ko.aung" `
    -Department "Engineering" `
    -JobTitle "Cloud Engineer" `
    -UsageLocation "SG" `
    -PasswordProfile $pwdProfile

# Verify
Get-MgUser -Filter "displayName eq 'Ko Aung'" | Select DisplayName, UserPrincipalName
```

### User ဖန်တီးခြင်း — Azure CLI

```bash
# User create
az ad user create \
    --display-name "Ko Aung" \
    --user-principal-name ko.aung@contoso.com \
    --password "TempP@ss123!" \
    --force-change-password-next-sign-in true

# User list
az ad user list --output table

# User show
az ad user show --id ko.aung@contoso.com --query "{Name:displayName, UPN:userPrincipalName, Enabled:accountEnabled}"

# User delete
az ad user delete --id ko.aung@contoso.com
```

### Bulk Operations

**Bulk Create — CSV Format:**
```csv
version:v1.0
Name [displayName] Required,User name [userPrincipalName] Required,Initial Password [password] Required,Block Sign In [accountEnabled] Optional,Usage Location [usageLocation] Required,Department [department] Optional,Job Title [jobTitle] Optional
Ko Aung,ko.aung@contoso.com,TempP@ss123!,False,SG,Engineering,Cloud Engineer
Ma May,ma.may@contoso.com,TempP@ss456!,False,SG,Sales,Sales Manager
U Kyaw,u.kyaw@contoso.com,TempP@ss789!,False,SG,IT,System Admin
```

**Portal Upload:** Entra ID → Users → Bulk operations → **Bulk create** → CSV upload

### Guest Users (B2B Collaboration)

```
Guest User Lifecycle:
Admin (Portal/PS/CLI)
    │ Sends invitation
    ▼
Guest receives email → Clicks "Accept invitation"
    │ Redeems invitation
    ▼
Guest authenticates with own IdP (Gmail, Microsoft, corporate)
    │ First-time consent
    ▼
Guest added to tenant (userType = Guest)
    │ Admin assigns roles/groups
    ▼
Guest accesses assigned resources
    │ Admin can revoke anytime
    ▼
Remove user → Access revoked immediately
```

```powershell
# Guest invite (PowerShell)
New-MgInvitation `
    -InvitedUserDisplayName "Partner User" `
    -InvitedUserEmailAddress "partner@external.com" `
    -InviteRedirectUrl "https://myapps.microsoft.com" `
    -SendInvitationMessage:$true
```

```bash
# Guest invite (CLI)
az ad user invite \
    --invitation-url "https://myapps.microsoft.com" \
    --user-display-name "Partner User" \
    --user-email-address partner@external.com \
    --send-invitation true
```

### Groups

**Group Types:**

| Type | Membership | Use Case | Dynamic? |
|------|-----------|---------|---------|
| **Security Group** | Users/Devices | RBAC, licensing | ✅ User/Device |
| **Microsoft 365 Group** | Users only | Teams, SharePoint | ✅ User only |
| **Distribution List** | Users | Email distribution | ❌ |

**Membership Types:**

| Type | How Members Added |
|------|------------------|
| **Assigned** | Manually add/remove |
| **Dynamic User** | Attribute rules (dept, title, location) |
| **Dynamic Device** | Device properties (OS, join type) |

**Dynamic Group Rules (Examples):**

```
# Engineering team
(user.department -eq "Engineering") and (user.accountEnabled -eq true)

# Senior engineers in Singapore
(user.department -eq "Engineering") and (user.jobTitle -startsWith "Senior") and (user.usageLocation -eq "SG")

# All external guests
(user.userType -eq "Guest")

# Multiple departments (OR)
(user.department -eq "Engineering") or (user.department -eq "DevOps")

# Device-based (Windows 10+)
(device.deviceOSType -eq "Windows") and (device.deviceOSVersion -startsWith "10")
```

```bash
# Security group create
az ad group create \
    --display-name "Cloud Engineers" \
    --mail-nickname CloudEngineers

# Member add
az ad group member add \
    --group "Cloud Engineers" \
    --member-id $(az ad user show --id ko.aung@contoso.com --query id -o tsv)

# Members list
az ad group member list --group "Cloud Engineers" --output table

# Check if user is member
az ad group member check \
    --group "Cloud Engineers" \
    --member-id $(az ad user show --id ko.aung@contoso.com --query id -o tsv)
```

---

## ၁.၃ Role-Based Access Control (RBAC)

### RBAC Model

**Formula:** `Security Principal + Role Definition + Scope = Access`

```
WHO (Security Principal)        WHAT (Role Definition)     WHERE (Scope)
├── User                        ├── Actions allowed         ├── Management Group
├── Group            +          ├── NotActions (excluded) + ├── Subscription
├── Service Principal           ├── DataActions             ├── Resource Group
└── Managed Identity            └── NotDataActions          └── Individual Resource
```

**Scope Inheritance (Top → Down):**
```
Management Group  ──► assignment inherits to all subscriptions below
  └── Subscription  ──► inherits to all resource groups
        └── Resource Group  ──► inherits to all resources
              └── Resource  ──► most specific scope
```

### Built-in Roles (ကျက်မှတ်ရမည်)

| Role | Resources | Access Management | Key Note |
|------|-----------|-----------------|---------|
| **Owner** | ✅ Full | ✅ Can assign roles | Most powerful |
| **Contributor** | ✅ Full | ❌ Cannot assign | Dev/Ops standard |
| **Reader** | 👁 Read only | ❌ Cannot assign | Auditor, stakeholder |
| **User Access Admin** | ❌ Resource actions | ✅ Can assign roles | Access governance only |
| **VM Contributor** | ✅ VMs only | ❌ | No network/storage create |
| **Network Contributor** | ✅ Network only | ❌ | No VM/storage |
| **Storage Blob Data Contributor** | ✅ Blob data | ❌ | NOT storage account mgmt |
| **Storage Account Contributor** | ✅ Account mgmt | ❌ | NOT data plane access |
| **Key Vault Administrator** | ✅ Full KV | ❌ | Keys, secrets, certs |
| **Backup Operator** | ✅ Backup ops | ❌ | Cannot create vault |
| **Monitoring Contributor** | ✅ Monitoring | ❌ | Alerts, dashboards |

> ⚠️ **Critical Traps:**
> - **Owner vs Contributor** — Owner သာ role assign ပြုနိုင်; Contributor မပြုနိုင်
> - **Storage Blob Data Contributor** — Data ကိုသာ access; Storage Account ကို manage မပြုနိုင်
> - **VM Contributor** — VM ဖန်တီးနိုင်သော်လည်း VNet/Storage account ကို separately assign လိုသည်

### Role Assignment

```bash
# Role assign — user to resource group
az role assignment create \
    --assignee ko.aung@contoso.com \
    --role "Contributor" \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-production

# Role assign — group to subscription
az role assignment create \
    --assignee {group-object-id} \
    --role "Reader" \
    --scope /subscriptions/{sub-id}

# Role assign — managed identity to resource
az role assignment create \
    --assignee {managed-identity-client-id} \
    --role "Storage Blob Data Contributor" \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-data/providers/Microsoft.Storage/storageAccounts/mystorageacct

# List assignments
az role assignment list \
    --assignee ko.aung@contoso.com \
    --output table

# Remove assignment
az role assignment delete \
    --assignee ko.aung@contoso.com \
    --role "Contributor" \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-production
```

### Custom Roles

Built-in roles မကိုက်ညီသောအခါ custom roles ဖန်တီးနိုင်သည်

```json
{
  "Name": "VM Power Operator",
  "Description": "Can start, stop, restart VMs but cannot create or delete",
  "Actions": [
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/deallocate/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/instanceView/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read"
  ],
  "NotActions": [],
  "DataActions": [],
  "NotDataActions": [],
  "AssignableScopes": [
    "/subscriptions/{subscriptionId}",
    "/subscriptions/{subscriptionId}/resourceGroups/rg-production"
  ]
}
```

```powershell
# Create custom role
New-AzRoleDefinition -InputFile "vm-power-operator.json"

# List custom roles
Get-AzRoleDefinition | Where-Object { $_.IsCustom -eq $true } | Select Name, Description

# Update custom role
$role = Get-AzRoleDefinition "VM Power Operator"
$role.Actions.Add("Microsoft.Compute/virtualMachines/extensions/read")
Set-AzRoleDefinition -Role $role

# Delete custom role
Remove-AzRoleDefinition -Name "VM Power Operator"
```

```bash
# CLI equivalent
az role definition create --role-definition vm-power-operator.json
az role definition list --custom-role-only true --output table
az role definition update --role-definition @updated-role.json
az role definition delete --name "VM Power Operator"
```

**Custom Role Limits:**
- Subscription တစ်ခုတွင် max **5,000** custom roles
- `AssignableScopes` တွင် Management Groups, Subscriptions, Resource Groups သတ်မှတ်နိုင်သည်

### Managed Identities

Managed Identity သည် credentials (passwords/keys) မသုံးဘဲ Azure services တို့ authenticated ဖြစ်ရန် Azure ကိုယ်တိုင် manage ပြုလုပ်သော identity ဖြစ်သည်

```
Types:
┌── System-Assigned Managed Identity
│   ├── Resource (VM, App Service) နှင့် tied ဖြစ်သည်
│   ├── Resource delete ဖြစ်ရင် identity ပါ delete ဖြစ်သည်
│   └── One resource = one identity only
│
└── User-Assigned Managed Identity
    ├── Standalone resource အဖြစ် create ပြုလုပ်သည်
    ├── Multiple resources ကို assign ပြုနိုင်သည်
    └── Resource delete ဖြစ်သော်လည်း identity ကျန်ရစ်သည်
```

```bash
# System-assigned: VM တွင် enable
az vm identity assign \
    --resource-group rg-compute \
    --name vm-webserver

# User-assigned: create standalone identity
az identity create \
    --resource-group rg-identity \
    --name id-webapp

# Assign to VM
az vm identity assign \
    --resource-group rg-compute \
    --name vm-webserver \
    --identities /subscriptions/{sub-id}/resourceGroups/rg-identity/providers/Microsoft.ManagedIdentity/userAssignedIdentities/id-webapp

# Grant identity permission (Storage Blob access)
az role assignment create \
    --assignee $(az identity show --name id-webapp -g rg-identity --query clientId -o tsv) \
    --role "Storage Blob Data Reader" \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-data/providers/Microsoft.Storage/storageAccounts/mystorageacct
```

---

## ၁.၄ Subscriptions and Management Groups

### Management Groups Hierarchy

```
Root Management Group (Tenant Root)
├── Corp Management Group
│   ├── Production Management Group
│   │   ├── Sub-Prod-001 (Production workloads)
│   │   └── Sub-Prod-002 (DR workloads)
│   └── Non-Production Management Group
│       ├── Sub-Dev-001 (Development)
│       └── Sub-Test-001 (Testing/Staging)
└── Platform Management Group
    ├── Sub-Identity-001 (Identity/AD)
    ├── Sub-Connectivity-001 (Hub networking)
    └── Sub-Management-001 (Monitoring, Security)
```

**Management Group Limits:**
- Max **6 levels** deep (root level not counted)
- Max **10,000** management groups per tenant
- Each MG can have max **1,000** subscriptions

```powershell
# Management Group create
New-AzManagementGroup -GroupId "mg-production" -DisplayName "Production"

# Subscription add to MG
New-AzManagementGroupSubscription -GroupId "mg-production" -SubscriptionId "{sub-id}"

# MG hierarchy list
Get-AzManagementGroup -Expand -Recurse
```

---

## ၁.၅ Azure Policy

### Policy Architecture

```
Policy Definition (single rule)
    ↓ Group into
Policy Initiative / Set (collection of definitions)
    ↓ Apply as
Policy Assignment (at scope: MG/Sub/RG)
    ↓ Evaluate
Compliance Results (compliant / non-compliant)
    ↓ Optionally
Remediation Task (fix non-compliant resources)
```

### Policy Effects (ကျက်မှတ်ရမည်)

| Effect | When Applied | Action | Use Case |
|--------|-------------|--------|---------|
| **Disabled** | Before evaluation | Skip entirely | Testing |
| **Append** | During create/update | Add properties | Force tags, IP rules |
| **Deny** | Before create/update | Block request | Region restriction |
| **Audit** | After create/update | Log warning only | Visibility, reporting |
| **AuditIfNotExists** | After create | Audit if related resource missing | Check diagnostic settings |
| **DeployIfNotExists** | After create | Deploy related resource | Auto-add monitoring agent |
| **Modify** | During create/update | Add/modify/remove properties | Tag inheritance |

> 📌 **Evaluation Order:** Disabled → Append → Deny → Audit → AuditIfNotExists → DeployIfNotExists → Modify

### Common Built-in Policies

```
Governance:
"Allowed locations"                   → Region restriction (Deny)
"Allowed resource types"              → Only specific services (Deny)
"Allowed VM SKUs"                     → VM size restriction (Deny)
"Require a tag and its value"         → Mandatory tags (Deny)
"Inherit a tag from resource group"   → Tag propagation (Modify)

Security:
"Storage accounts should use HTTPS"   → Enforce TLS (Deny/Audit)
"VMs should use managed disks"        → No unmanaged disks (Audit)
"MFA should be enabled on accounts"   → Security baseline (Audit)

Cost:
"Not allowed resource types"          → Block expensive services (Deny)
"Allowed storage account SKUs"        → Limit premium storage (Deny)
```

### Policy Assignment (CLI)

```bash
# List available built-in policies
az policy definition list --query "[?policyType=='BuiltIn'].{Name:displayName,ID:name}" --output table

# Assign "Allowed locations" policy
az policy assignment create \
    --name "AllowedRegions" \
    --display-name "Allowed Azure Regions" \
    --policy "e765b5de-1225-4ba3-bd56-1ac6695d345d" \
    --params '{"listOfAllowedLocations":{"value":["eastasia","southeastasia","centralindia"]}}' \
    --scope /subscriptions/{sub-id}

# Check compliance
az policy state list \
    --resource /subscriptions/{sub-id}/resourceGroups/rg-production \
    --query "[?complianceState=='NonCompliant'].{Resource:resourceId,Policy:policyDefinitionName}" \
    --output table

# Create remediation task
az policy remediation create \
    --name "RemediateTagInheritance" \
    --policy-assignment AllowedRegions \
    --resource-group rg-production
```

### Policy Initiative Example

```json
{
  "properties": {
    "displayName": "Security Baseline",
    "description": "Collection of security policies",
    "policyDefinitions": [
      {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/...storage-https...",
        "parameters": { "effect": { "value": "Deny" } }
      },
      {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/...vm-managed-disk...",
        "parameters": { "effect": { "value": "Audit" } }
      }
    ]
  }
}
```

---

## ၁.၆ Resource Locks

### Lock Types

| Lock | Read | Write | Delete | Common Use |
|------|:----:|:-----:|:------:|-----------|
| **CanNotDelete** | ✅ | ✅ | ❌ | Production resources protection |
| **ReadOnly** | ✅ | ❌ | ❌ | Critical configs (Key Vault, VNet) |

**ReadOnly Lock ၏ Hidden Effects (Exam Trap!):**
```
Resource         ReadOnly Lock ကြောင့် block ဖြစ်သော actions
─────────        ─────────────────────────────────────────────
VM               Start, Stop, Restart, Resize, Add disk
Storage Account  Regenerate access keys, change tier
App Service      Scale up/down, deploy code
Key Vault        Add/delete secrets
SQL DB           Scale DTUs/vCores
```

> ⚠️ ReadOnly lock = VM ကို deallocate/start ပင် block ဖြစ်သည် — Exam မှာ မကြာမကြာ မေးသည်

```bash
# Lock create (resource group level)
az lock create \
    --name "ProductionLock" \
    --resource-group rg-production \
    --lock-type CanNotDelete \
    --notes "Production resources - delete requires manager approval"

# Lock create (subscription level)
az lock create \
    --name "SubReadOnly" \
    --lock-type ReadOnly \
    --resource-group "" \
    --scope /subscriptions/{sub-id}

# Lock list
az lock list --resource-group rg-production --output table

# Lock delete (admin/owner required)
az lock delete \
    --name "ProductionLock" \
    --resource-group rg-production

# PowerShell equivalent
New-AzResourceLock -LockName "ProdLock" -LockLevel CanNotDelete -ResourceGroupName "rg-production"
Get-AzResourceLock -ResourceGroupName "rg-production"
Remove-AzResourceLock -LockName "ProdLock" -ResourceGroupName "rg-production"
```

**Lock Hierarchy:**
- Parent scope lock → child resources ကို inherit ဖြစ်သည်
- Subscription lock → all RGs + resources ကို affect ဖြစ်သည်
- Lock delete ပြုလုပ်ရန် Owner or User Access Admin role လိုသည်

---

## ၁.၇ Tags

### Tagging Strategy

**Recommended Tag Set:**

| Tag Key | Example Values | Purpose |
|---------|---------------|---------|
| `Environment` | Production, Staging, Dev | Lifecycle management |
| `CostCenter` | CC-1234, Engineering | Cost allocation |
| `Owner` | ko.aung@contoso.com | Accountability |
| `Application` | webapp-sales, api-inventory | App mapping |
| `DataClassification` | Public, Internal, Confidential | Compliance |
| `BackupPolicy` | Daily, Weekly, None | Backup management |
| `CreatedDate` | 2025-01-15 | Resource age tracking |
| `ExpiryDate` | 2026-01-15 | Temporary resource cleanup |

**Tag Limits:**
- Resource တစ်ခုတွင် max **50 tags**
- Tag key: max **512 characters** (storage accounts: 128)
- Tag value: max **256 characters**
- Tags are **not inherited** from RG to resources (Policy မပါဘဲ)

```bash
# Tag a resource group
az group update \
    --name rg-production \
    --tags Environment=Production CostCenter=Engineering Owner=ops@contoso.com

# Tag an individual resource
az resource tag \
    --resource-group rg-production \
    --name vm-webserver \
    --resource-type Microsoft.Compute/virtualMachines \
    --tags Environment=Production BackupPolicy=Daily

# Query resources by tag
az resource list \
    --tag Environment=Production \
    --output table

# Remove a specific tag
az resource tag \
    --resource-group rg-production \
    --name vm-webserver \
    --resource-type Microsoft.Compute/virtualMachines \
    --tags "" # removes all tags

# PowerShell — update tags without removing existing
$tags = (Get-AzResource -Name vm-webserver -ResourceGroupName rg-production).Tags
$tags["BackupPolicy"] = "Daily"
Set-AzResource -ResourceGroupName rg-production -Name vm-webserver -ResourceType "Microsoft.Compute/virtualMachines" -Tag $tags -Force
```

### Tag Inheritance via Policy

Tags ကို resource group မှ resources သို့ automatically propagate ပြုလုပ်ရန် Policy သုံးသည်

```bash
# Assign "Inherit a tag from resource group" policy
az policy assignment create \
    --name "InheritEnvTag" \
    --policy "cd3aa116-8754-49c9-a813-ad46d90f91a7" \
    --params '{"tagName":{"value":"Environment"}}' \
    --scope /subscriptions/{sub-id}/resourceGroups/rg-production
```

---

## ၁.၈ Cost Management

### Azure Cost Management Tools

```
Azure Cost Management + Billing
├── Cost Analysis    ← Usage breakdown by resource, tag, service
├── Budgets          ← Spending limits with alerts
├── Alerts           ← Cost/credit/quota notifications
├── Advisor          ← Cost optimization recommendations
└── Export           ← Scheduled cost data export to Storage
```

### Budget + Alert Setup

```bash
# Create monthly budget with alert at 80%
az consumption budget create \
    --budget-name "MonthlyEngineeringBudget" \
    --amount 5000 \
    --time-grain Monthly \
    --start-date 2025-01-01 \
    --end-date 2025-12-31 \
    --resource-group rg-production \
    --category Cost

# Add alert notification
az consumption budget notification create \
    --budget-name "MonthlyEngineeringBudget" \
    --threshold 80 \
    --threshold-type Forecasted \
    --contact-emails ops@contoso.com finance@contoso.com \
    --operator GreaterThan
```

---

## ၁.၉ Self-Service Password Reset (SSPR)

SSPR သည် users ကို IT helpdesk မကိုးဘဲ password ကိုယ်တိုင် reset ပြုလုပ်ခွင့်ပေးသည်

**License Requirement:** Entra ID P1 (or P2)

**SSPR Methods:**

| Method | Security Level | Notes |
|--------|---------------|-------|
| Authenticator App | 🔴 Highest | Microsoft/3rd party |
| Phone (SMS) | 🟡 Medium | SMS code |
| Email | 🟡 Medium | External email only |
| Office Phone | 🟡 Medium | Voice call |
| Security Questions | 🟢 Lowest | Not recommended for production |

**Configuration:**
- **Scope:** None / Selected group / All
- **Required methods count:** 1 or 2 (2 = more secure)
- **Administrator accounts:** Always require 2 methods regardless of setting

```
SSPR Flow:
User forgets password
    ↓
Go to https://aka.ms/sspr
    ↓
Enter email + CAPTCHA
    ↓
Verify identity (registered method/s)
    ↓
Set new password
    ↓
Password reset complete + notification email sent
```

---

## ၁.၁၀ Conditional Access (P1 Required)

Conditional Access သည် "if-then" policies ဖြင့် access ကို control ပြုလုပ်သည်

```
Signal (Conditions)              Controls (Access)
├── User/Group                   ├── Block Access
├── Device platform/state        ├── Grant Access (with conditions)
├── IP location                  │   ├── Require MFA
├── Application                  │   ├── Compliant device
├── Real-time risk               │   └── Approved apps
└── Session                      └── Session controls
```

**Common CA Policy Examples:**

```
Policy 1: Require MFA for all admins
- Users: Global Admins, Security Admins
- Cloud apps: All
- Access: Grant — Require MFA

Policy 2: Block non-corporate devices
- Users: All
- Device platforms: iOS, Android (unmanaged)
- Access: Block

Policy 3: Block legacy authentication
- Users: All
- Client apps: Other clients (basic auth)
- Access: Block
```

---

## Domain 1 — Key Takeaways

```
✅ Must Know:
   • Entra ID ≠ On-prem AD DS (different protocols, flat structure)
   • License: P1 = Conditional Access + SSPR | P2 = Identity Protection + PIM
   • Owner = Contributor + access management permission
   • Custom roles = JSON definition + AssignableScopes
   • Management Groups = multi-subscription governance
   • Policy effects order: Disabled → Append → Deny → Audit → DeployIfNotExists
   • ReadOnly lock blocks VM start/stop/deallocate
   • Tags NOT auto-inherited (need Policy for propagation)
   • SSPR needs P1; User's Usage Location needed before license assign

⚠️ Common Exam Traps:
   • "Allow modification but block deletion" → CanNotDelete (not ReadOnly)
   • "Block all changes including VM start" → ReadOnly
   • "Can Contributor assign roles?" → NO, only Owner can
   • "VM Contributor creates a VM — can they attach existing VNet?" → NO, need Network Contributor too
   • "Apply policy to 50 subscriptions at once" → Management Group policy assignment
```
