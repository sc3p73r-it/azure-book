# AZ-104: Microsoft Azure Administrator Associate
## လေ့လာရေးလမ်းညွှန်စာအုပ် (Study Guide)

> **မူရင်း Certification:** Microsoft Certified: Azure Administrator Associate  
> **စာမျက်နှာ အပ်ဒိတ်:** April 17, 2026 Skills Outline နှင့် ညီညွတ်သည်  
> **ပစ်မှတ်:** AZ-104 စာမေးပွဲကို ပထမကြိမ်တွင် အောင်မြင်နိုင်ရန်

---

## Domain 4: Implement and Manage Virtual Networking
### (15–20%) — Technically Hardest Domain

---

### ၄.၁ Virtual Network (VNet) Fundamentals

#### VNet Design Principles

```
VNet: 10.0.0.0/16 (65,536 addresses)
├── Subnet: 10.0.1.0/24 (frontend - 251 usable)
├── Subnet: 10.0.2.0/24 (backend - 251 usable)
├── Subnet: 10.0.3.0/24 (database - 251 usable)
└── GatewaySubnet: 10.0.255.0/27 (VPN/ExpressRoute gateway)
```

> ⚠️ **Azure Reserved IPs per Subnet:** First 4 addresses + last 1 address = 5 addresses reserved
> - .0 = Network address
> - .1 = Default gateway
> - .2, .3 = Azure DNS
> - .255 = Broadcast (unused but reserved)

#### VNet ဖန်တီးခြင်း

```bash
# VNet ဖန်တီးခြင်း
az network vnet create \
    --resource-group rg-network \
    --name vnet-main \
    --address-prefix 10.0.0.0/16 \
    --location southeastasia

# Subnet add ပြုလုပ်ခြင်း
az network vnet subnet create \
    --resource-group rg-network \
    --vnet-name vnet-main \
    --name subnet-frontend \
    --address-prefix 10.0.1.0/24

az network vnet subnet create \
    --resource-group rg-network \
    --vnet-name vnet-main \
    --name subnet-backend \
    --address-prefix 10.0.2.0/24
```

---

### ၄.၂ Network Security Groups (NSG)

NSG သည် inbound/outbound network traffic ကို filter ပြုလုပ်ရာတွင် သုံးသည်

#### NSG Rule Properties

| Property | ရှင်းလင်းချက် |
|----------|-------------|
| Priority | 100–4096; **lower number = higher priority** |
| Source/Destination | IP, Service Tag, Application Security Group |
| Protocol | TCP, UDP, ICMP, Any |
| Direction | Inbound, Outbound |
| Action | Allow, Deny |

#### Default Rules (cannot delete, but can override)

**Inbound:**
```
Priority 65000: Allow VirtualNetwork → VirtualNetwork (any port)
Priority 65001: Allow AzureLoadBalancer → Any (any port)
Priority 65500: Deny Any → Any (all traffic)
```

**Outbound:**
```
Priority 65000: Allow VirtualNetwork → VirtualNetwork (any port)
Priority 65001: Allow Any → Internet (any port)
Priority 65500: Deny Any → Any (all traffic)
```

#### NSG ဖန်တီးပြီး Rule ထည့်ခြင်း

```bash
# NSG ဖန်တီးခြင်း
az network nsg create \
    --resource-group rg-network \
    --name nsg-frontend

# Inbound HTTP allow rule
az network nsg rule create \
    --resource-group rg-network \
    --nsg-name nsg-frontend \
    --name Allow-HTTP \
    --priority 100 \
    --protocol Tcp \
    --direction Inbound \
    --source-address-prefix Internet \
    --destination-port-range 80 \
    --access Allow

# Inbound HTTPS allow rule
az network nsg rule create \
    --resource-group rg-network \
    --nsg-name nsg-frontend \
    --name Allow-HTTPS \
    --priority 110 \
    --protocol Tcp \
    --direction Inbound \
    --source-address-prefix Internet \
    --destination-port-range 443 \
    --access Allow

# NSG ကို subnet သို့ associate
az network vnet subnet update \
    --resource-group rg-network \
    --vnet-name vnet-main \
    --name subnet-frontend \
    --network-security-group nsg-frontend
```

#### Service Tags

Service Tags သည် specific Azure services ၏ IP ranges ကို represent ပြုလုပ်သည်

```
Commonly used Service Tags:
├── Internet          - All internet IP ranges
├── VirtualNetwork    - VNet address space + peered VNets
├── AzureLoadBalancer - Azure Load Balancer health probe source
├── Storage           - Azure Storage IP ranges
├── Sql               - Azure SQL IP ranges
├── AzureMonitor      - Azure Monitor & Log Analytics
├── AppService        - App Service outbound IPs
└── GatewayManager    - Azure Gateway Manager
```

---

### ၄.၃ VNet Peering

VNet Peering သည် same/different regions ရှိ VNets ၂ ခုကို private Microsoft backbone ဖြင့် connect ပြုလုပ်သည်

#### Peering Types

| Type | Description |
|------|-------------|
| **VNet Peering** | Same region |
| **Global VNet Peering** | Different regions |

```bash
# VNet Peering ဖန်တီးခြင်း (bidirectional — ၂ direction ဖြစ်ရမည်)
# VNet-A → VNet-B
az network vnet peering create \
    --resource-group rg-network \
    --name peer-a-to-b \
    --vnet-name vnet-a \
    --remote-vnet vnet-b \
    --allow-vnet-access

# VNet-B → VNet-A
az network vnet peering create \
    --resource-group rg-network \
    --name peer-b-to-a \
    --vnet-name vnet-b \
    --remote-vnet vnet-a \
    --allow-vnet-access
```

> ⚠️ **VNet Peering Caveats:**
> - Overlapping address spaces = peering **မဖြစ်နိုင်**
> - Peering သည် **non-transitive** ဖြစ်သည် (A↔B, B↔C ဆိုပါက A↔C directly communicate မပြုနိုင် — Hub-spoke architecture လိုသည်)

---

### ၄.၄ Azure DNS

#### Public DNS Zone

```bash
# Public DNS zone ဖန်တီးခြင်း
az network dns zone create \
    --resource-group rg-network \
    --name contoso.com

# A record ထည့်ခြင်း
az network dns record-set a add-record \
    --resource-group rg-network \
    --zone-name contoso.com \
    --record-set-name www \
    --ipv4-address 203.0.113.10

# CNAME record ထည့်ခြင်း
az network dns record-set cname set-record \
    --resource-group rg-network \
    --zone-name contoso.com \
    --record-set-name blog \
    --cname mywebapp.azurewebsites.net
```

#### Private DNS Zone

```bash
# Private DNS zone ဖန်တီးခြင်း
az network private-dns zone create \
    --resource-group rg-network \
    --name private.contoso.com

# VNet နှင့် link ပြုလုပ်ခြင်း (auto-registration enable)
az network private-dns link vnet create \
    --resource-group rg-network \
    --zone-name private.contoso.com \
    --name link-to-vnet-main \
    --virtual-network vnet-main \
    --registration-enabled true
```

---

### ၄.၅ Load Balancing

#### Load Balancer Comparison

| Service | Layer | Scope | SSL Termination | URL Routing | Use Case |
|---------|-------|-------|----------------|-------------|---------|
| **Azure Load Balancer** | L4 (TCP/UDP) | Regional | ❌ | ❌ | Internal/External VM LB |
| **Application Gateway** | L7 (HTTP/HTTPS) | Regional | ✅ | ✅ | Web apps, WAF |
| **Traffic Manager** | DNS-based | Global | ❌ | ❌ | Global geo-routing |
| **Azure Front Door** | L7 (HTTP/HTTPS) | Global | ✅ | ✅ | Global web apps, CDN |

#### Azure Load Balancer SKUs

| Feature | Basic | Standard |
|---------|-------|---------|
| Backend pool | Availability Set / VMSS (same) | Any VM in VNet |
| Health probes | HTTP, TCP | HTTP, HTTPS, TCP |
| Availability Zones | ❌ | ✅ |
| SLA | None | 99.99% |
| Secure by default | ❌ | ✅ (NSG required) |

```bash
# Standard Load Balancer ဖန်တီးခြင်း
az network lb create \
    --resource-group rg-network \
    --name lb-web \
    --sku Standard \
    --frontend-ip-name frontend-ip \
    --public-ip-address pip-lb \
    --backend-pool-name backend-pool

# Health probe ဖန်တီးခြင်း
az network lb probe create \
    --resource-group rg-network \
    --lb-name lb-web \
    --name health-probe-http \
    --protocol Http \
    --port 80 \
    --path /health

# Load balancing rule ဖန်တီးခြင်း
az network lb rule create \
    --resource-group rg-network \
    --lb-name lb-web \
    --name lb-rule-http \
    --protocol Tcp \
    --frontend-port 80 \
    --backend-port 80 \
    --frontend-ip-name frontend-ip \
    --backend-pool-name backend-pool \
    --probe-name health-probe-http
```

---

### ၄.၆ VPN Gateway and ExpressRoute

#### VPN Gateway

VPN Gateway သည် Azure VNet နှင့် on-prem network ကြား encrypted IPsec/IKE VPN tunnel ဖြင့် connect ပြုလုပ်သည်

**Gateway Types:**

| Type | Use Case |
|------|---------|
| **VPN** | Site-to-site, Point-to-site VPN |
| **ExpressRoute** | ExpressRoute circuit ချိတ်ဆက်ရာတွင် |

**VPN Gateway SKUs:**

| SKU | Max S2S Tunnels | Throughput |
|-----|----------------|-----------|
| Basic | 10 | 100 Mbps |
| VpnGw1 | 30 | 650 Mbps |
| VpnGw2 | 30 | 1 Gbps |
| VpnGw3 | 30 | 1.25 Gbps |
| VpnGw4/5 | 100 | 5/10 Gbps |

```bash
# GatewaySubnet ဖန်တီးခြင်း (required)
az network vnet subnet create \
    --resource-group rg-network \
    --vnet-name vnet-main \
    --name GatewaySubnet \
    --address-prefix 10.0.255.0/27

# Public IP ဖန်တီးခြင်း
az network public-ip create \
    --resource-group rg-network \
    --name pip-vpn-gateway \
    --sku Standard

# VPN Gateway ဖန်တီးခြင်း (~30-45 minutes)
az network vnet-gateway create \
    --resource-group rg-network \
    --name vpngw-main \
    --location southeastasia \
    --public-ip-address pip-vpn-gateway \
    --vnet vnet-main \
    --gateway-type Vpn \
    --vpn-type RouteBased \
    --sku VpnGw2
```

#### ExpressRoute

ExpressRoute သည် on-prem network ကို Azure သို့ dedicated private connection ဖြင့် ချိတ်ဆက်ရာတွင် သုံးသည် (public internet မဖြတ်)

```
ExpressRoute Connection Path:
Customer Edge Router
    └── ExpressRoute Provider (Equinix, Telstra, etc.)
        └── Microsoft Edge (Meet-me location)
            └── Azure Network
```

| Feature | VPN Gateway | ExpressRoute |
|---------|------------|-------------|
| Speed | Up to 10 Gbps | 50 Mbps – 100 Gbps |
| Reliability | Internet dependent | Private, dedicated |
| Latency | Variable | Predictable, low |
| Cost | Lower | Higher |
| Setup time | Hours | Weeks/Months |
| Encryption | Built-in (IPsec) | **Not encrypted by default** |

---

### ၄.၇ Azure Bastion and Firewall

#### Azure Bastion

Bastion သည် public IP မသုံးဘဲ Azure Portal မှ RDP/SSH ဖြင့် VM ကို securely access ပေးသည်

```bash
# AzureBastionSubnet ဖန်တီးခြင်း (minimum /26 required)
az network vnet subnet create \
    --resource-group rg-network \
    --vnet-name vnet-main \
    --name AzureBastionSubnet \
    --address-prefix 10.0.253.0/26

# Bastion host ဖန်တီးခြင်း
az network bastion create \
    --resource-group rg-network \
    --name bastion-main \
    --vnet-name vnet-main \
    --public-ip-address pip-bastion \
    --sku Standard
```

#### Azure Firewall

Azure Firewall သည် cloud-native, stateful managed network firewall service ဖြစ်သည်

```
Azure Firewall Rule Types:
├── Network Rules (L4) — IP/protocol/port based
├── Application Rules (L7) — FQDN/URL based (HTTP/HTTPS)
└── NAT Rules — DNAT for inbound traffic
```

> 📌 Azure Firewall ကို Hub-Spoke topology ၏ hub VNet တွင် ထားပြီး all spokes ၏ traffic ကို route ပြုလုပ်သည်

---

<a name="domain-5"></a>
