# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## အခန်း ၆ — Hands-On Lab Exercises

### Lab 1: Identity and Access Management

**Objective:** Entra ID တွင် user/group ဖန်တီးပြီး RBAC configure ပြုလုပ်ခြင်း

```bash
# 1. Resource Group ဖန်တီးခြင်း
az group create --name rg-lab-01 --location southeastasia

# 2. User ဖန်တီးခြင်း
az ad user create \
    --display-name "Lab User 01" \
    --user-principal-name labuser01@$(az account show --query tenantId -o tsv).onmicrosoft.com \
    --password "Lab@Pass123!" \
    --force-change-password-next-sign-in false

# 3. Security Group ဖန်တီးခြင်း
az ad group create \
    --display-name "Cloud Engineers" \
    --mail-nickname CloudEngineers

# 4. User ကို Group ထဲ ထည့်ခြင်း
az ad group member add \
    --group CloudEngineers \
    --member-id $(az ad user show --id labuser01@... --query id -o tsv)

# 5. Group ကို Resource Group တွင် Contributor role assign
az role assignment create \
    --assignee $(az ad group show --group CloudEngineers --query id -o tsv) \
    --role Contributor \
    --scope /subscriptions/$(az account show --query id -o tsv)/resourceGroups/rg-lab-01
```

---

### Lab 2: Storage Configuration

**Objective:** Storage account ဖန်တီး၊ blob upload ပြုလုပ်ပြီး SAS ဖြင့် access ပေးခြင်း

```bash
# 1. Storage Account ဖန်တီးခြင်း
STORAGE_NAME="labstorage$(cat /dev/urandom | tr -dc 'a-z0-9' | head -c 8)"
az storage account create \
    --name $STORAGE_NAME \
    --resource-group rg-lab-02 \
    --sku Standard_LRS \
    --kind StorageV2

# 2. Container ဖန်တီးခြင်း
az storage container create \
    --name files \
    --account-name $STORAGE_NAME

# 3. File upload
echo "Hello Azure!" > test.txt
az storage blob upload \
    --container-name files \
    --name test.txt \
    --file test.txt \
    --account-name $STORAGE_NAME

# 4. SAS token ဖန်တီးပြီး URL generate
EXPIRY=$(date -u -d "+2 hours" '+%Y-%m-%dT%H:%MZ')
SAS=$(az storage blob generate-sas \
    --account-name $STORAGE_NAME \
    --container-name files \
    --name test.txt \
    --permissions r \
    --expiry $EXPIRY \
    --output tsv)
echo "https://$STORAGE_NAME.blob.core.windows.net/files/test.txt?$SAS"
```

---

### Lab 3: Virtual Network Setup

**Objective:** Hub-Spoke VNet topology ဖန်တီး၊ peering configure ပြုလုပ်ခြင်း

```bash
# Hub VNet ဖန်တီးခြင်း
az network vnet create -g rg-lab-03 -n vnet-hub --address-prefix 10.0.0.0/16
az network vnet subnet create -g rg-lab-03 --vnet-name vnet-hub -n AzureFirewallSubnet --address-prefix 10.0.1.0/26
az network vnet subnet create -g rg-lab-03 --vnet-name vnet-hub -n AzureBastionSubnet --address-prefix 10.0.2.0/26

# Spoke VNets ဖန်တီးခြင်း
az network vnet create -g rg-lab-03 -n vnet-spoke-dev --address-prefix 10.1.0.0/16
az network vnet subnet create -g rg-lab-03 --vnet-name vnet-spoke-dev -n subnet-app --address-prefix 10.1.1.0/24

az network vnet create -g rg-lab-03 -n vnet-spoke-prod --address-prefix 10.2.0.0/16
az network vnet subnet create -g rg-lab-03 --vnet-name vnet-spoke-prod -n subnet-app --address-prefix 10.2.1.0/24

# VNet Peering (Hub ↔ Spoke Dev)
az network vnet peering create -g rg-lab-03 -n hub-to-dev --vnet-name vnet-hub --remote-vnet vnet-spoke-dev --allow-vnet-access
az network vnet peering create -g rg-lab-03 -n dev-to-hub --vnet-name vnet-spoke-dev --remote-vnet vnet-hub --allow-vnet-access

# VNet Peering (Hub ↔ Spoke Prod)
az network vnet peering create -g rg-lab-03 -n hub-to-prod --vnet-name vnet-hub --remote-vnet vnet-spoke-prod --allow-vnet-access
az network vnet peering create -g rg-lab-03 -n prod-to-hub --vnet-name vnet-spoke-prod --remote-vnet vnet-hub --allow-vnet-access
```

---

<a name="cli-reference"></a>
