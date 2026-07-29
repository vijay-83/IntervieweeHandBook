
# 01_Azure_Networking_Part_001.md

## Q001-Q010 Azure Networking Fundamentals

### Q001 What is Azure VNet?
Answer: A logically isolated virtual network providing private connectivity, IP management, routing, and security boundaries.

### Q002 What is a Subnet?
Answer: A segmented IP range inside a VNet used to isolate workloads.

### Q003 What is CIDR?
Answer: CIDR defines IP ranges using prefix notation (e.g. 10.0.0.0/16).

### Q004 Public vs Private IP
- Public: Internet reachable.
- Private: Internal communication only.

### Q005 What is an NSG?
Stateful L3/L4 packet filtering for inbound/outbound traffic.

### Q006 NSG Rule Priority
Lower number = higher priority. First match wins.

### Q007 What is an ASG?
Logical grouping of VM NICs for simpler NSG rules.

### Q008 What is UDR?
Custom routing table to override Azure system routes.

### Q009 What is a Route Table?
Collection of routes associated with subnets.

### Q010 What are Azure System Routes?
Routes automatically created by Azure for VNet communication.

---
Next Command

NEXT_NET_002
