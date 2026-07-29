
# 01_Azure_Networking_Part_002.md

Version: 1.0
Questions: Q011-Q020

## Q011 What is VNet Peering?
Connects two Azure VNets over the Microsoft backbone with low latency. Peered VNets are not transitive.

Production: Hub VNet peered with multiple spoke VNets hosting AKS, ASE, and databases.

Azure CLI:
```bash
az network vnet peering create --name hub-to-spoke --resource-group rg-hub --vnet-name hub-vnet  --remote-vnet /subscriptions/<id>/resourceGroups/rg-spoke/providers/Microsoft.Network/virtualNetworks/spoke-vnet  --allow-vnet-access
```

Best Practice:
- Use hub-spoke architecture.
- Plan non-overlapping CIDRs.

---

## Q012 Can VNet Peering span regions?
Yes. Global VNet Peering connects VNets in different Azure regions over Microsoft's backbone.

---

## Q013 Is VNet Peering transitive?
No. If A peers with B and B peers with C, A cannot automatically reach C.

---

## Q014 VNet Peering vs VPN Gateway
Peering uses Azure backbone and offers lower latency. VPN encrypts traffic over tunnels and supports hybrid connectivity.

---

## Q015 What is Hub-Spoke Architecture?
A central hub hosts shared services (Firewall, Bastion, DNS). Spokes host workloads such as AKS and applications.

---

## Q016 What is Azure VPN Gateway?
Managed VPN service providing Site-to-Site, Point-to-Site, and VNet-to-VNet connectivity.

---

## Q017 What is ExpressRoute?
Private dedicated connectivity between on-premises and Azure without traversing the public Internet.

---

## Q018 VPN Gateway vs ExpressRoute
VPN: Internet-based, encrypted, lower cost.
ExpressRoute: Private circuit, predictable latency, higher bandwidth.

---

## Q019 What is Azure Bastion?
Managed jump service providing browser-based RDP/SSH without exposing public IPs on virtual machines.

---

## Q020 Why avoid overlapping address spaces?
Overlapping CIDRs prevent routing between VNets, VPNs, and ExpressRoute-connected networks.

Next Command:
NEXT_NET_003
