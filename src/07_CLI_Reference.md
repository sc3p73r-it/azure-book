# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## အခန်း ၇ — PowerShell & CLI Quick Reference

### Resource Group Commands

```bash
# az cli
az group create --name rg-prod --location eastus
az group list --output table
az group delete --name rg-dev --yes --no-wait
az group show --name rg-prod

# PowerShell
New-AzResourceGroup -Name rg-prod -Location eastus
Get-AzResourceGroup | Format-Table
Remove-AzResourceGroup -Name rg-dev -Force
```

### VM Commands

```bash
# VM list/status
az vm list --output table
az vm show --resource-group rg-compute --name vm-web --output json
az vm list-sizes --location southeastasia --output table

# Start/Stop/Restart
az vm start -g rg-compute -n vm-web
az vm deallocate -g rg-compute -n vm-web
az vm restart -g rg-compute -n vm-web

# VM resize
az vm resize -g rg-compute -n vm-web --size Standard_D4s_v5

# VM extension (Custom Script)
az vm extension set \
    --resource-group rg-compute \
    --vm-name vm-web \
    --name CustomScript \
    --publisher Microsoft.Azure.Extensions \
    --settings '{"fileUris":["https://raw.githubusercontent.com/contoso/scripts/main/setup.sh"],"commandToExecute":"bash setup.sh"}'
```

### Network Commands

```bash
# Effective NSG rules စစ်ဆေးခြင်း
az network nic show-effective-nsg \
    --resource-group rg-compute \
    --name vm-web-nic \
    --output table

# Effective routes စစ်ဆေးခြင်း
az network nic show-effective-route-table \
    --resource-group rg-compute \
    --name vm-web-nic \
    --output table

# Network Watcher IP flow verify
az network watcher test-ip-flow \
    --resource-group rg-compute \
    --vm vm-web \
    --direction Inbound \
    --protocol TCP \
    --local 10.0.1.5:80 \
    --remote 203.0.113.10:4000
```

---

<a name="exam-tips"></a>
