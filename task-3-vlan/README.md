# VLAN Implementation

Takes the Task 2 topology and adds VLAN segmentation across the existing switches. PCs are split into three VLANs based on their role, inter-VLAN routing is handled by a single Router-on-a-Stick configuration on R-A, and a wireless segment is added for laptops via an access point on SW-C.

## Topology

![VLAN Topology](screenshots/topology.png)

Same physical layout as Task 2, but now SW-A, SW-B, and SW-C are configured with VLAN-aware ports and trunk links toward R-A. The access point connects to SW-C and bridges wireless clients into VLAN 225.

## VLAN Design

The original /24 address space from Task 2 was replaced with a subnetted range that gives each VLAN its own network:

```
192.168.1.0/24
  192.168.1.0/25      VLAN 225  (Wi-Fi)    hosts: .2 to .126
  192.168.1.128/25
    192.168.1.128/26  VLAN 75   (Group A)  hosts: .130 to .190
    192.168.1.192/26  VLAN 150  (Group B)  hosts: .194 to .254
```

| VLAN | ID | Subnet | Gateway | Members |
|------|----|--------|---------|---------|
| VLAN 75 | 75 | 192.168.1.128/26 | 192.168.1.129 (Fa0/0.75) | PC-A, PC-D, PC-F |
| VLAN 150 | 150 | 192.168.1.192/26 | 192.168.1.193 (Fa0/0.150) | PC-B, C, E, G, H, I, J, K, L |
| VLAN 225 | 225 | 192.168.1.0/25 | 192.168.1.1 (Fa0/0.225) | Laptops via AP |

PC-A, PC-D, and PC-F are spread across different switches but land in the same VLAN. That is the point of VLANs, logical grouping regardless of physical location.

## Router-on-a-Stick

Instead of running three separate cables from the switches to R-A, a single FastEthernet0/0 port is divided into three sub-interfaces using 802.1Q tagging. Each sub-interface handles one VLAN and acts as its default gateway.

```
interface FastEthernet0/0.75
 encapsulation dot1Q 75
 ip address 192.168.1.129 255.255.255.192

interface FastEthernet0/0.150
 encapsulation dot1Q 150
 ip address 192.168.1.193 255.255.255.192

interface FastEthernet0/0.225
 encapsulation dot1Q 225
 ip address 192.168.1.1 255.255.255.128
```

All three came up as up/up in simulation.

## Switches

Each switch has access ports for end devices (one VLAN per port) and trunk ports toward R-A that carry tagged frames for all three VLANs. SW-C also has a trunk toward the AP.

## DHCP

R-A is the DHCP server for all three VLANs. The first few IPs in each range are excluded for static use:

```
ip dhcp excluded-address 192.168.1.1   192.168.1.10
ip dhcp excluded-address 192.168.1.129 192.168.1.139
ip dhcp excluded-address 192.168.1.193 192.168.1.200
```

## Wireless

The access point connects to SW-C. Laptops connect wirelessly, get a DHCP address from the VLAN 225 pool, and can reach other VLANs through R-A.

| Setting | Value |
|---------|-------|
| SSID | VLAN 225 Wi-Fi |
| Security | WPA-PSK |
| VLAN | 225 |
| DHCP range | 192.168.1.2 to 192.168.1.126 |

## Connectivity Tests

Intra-VLAN, PC-A to PC-D (both VLAN 75): working, frames stay on the same VLAN without going through R-A.

Inter-VLAN, PC-A (VLAN 75) to PC-B (VLAN 150): working, traffic goes up to R-A via trunk, gets routed at Fa0/0.75 to Fa0/0.150, comes back down.

Inter-VLAN, PC-A (VLAN 75) to laptop (VLAN 225): working, same path through R-A.

Disabled Fa0/0.75 sub-interface: all VLAN 75 traffic stopped, VLAN 150 and 225 unaffected. Re-enabled and connectivity restored.

## How This Differs from Task 2

Task 2 had one broadcast domain per subnet and simple routing between them. Here, VLANs create logical segments that are independent of the physical switch ports. PC-A and PC-D are on different switches but in the same VLAN. PC-A and PC-B are on the same switch but in different VLANs and cannot talk without going through R-A. The subnets are also much smaller, using /26 and /25 instead of /24 across the board.

The main limitation is that Router-on-a-Stick puts all inter-VLAN traffic through a single physical link to R-A. A Layer 3 switch would handle this more efficiently but was not available in the lab environment.

## Files

| File | Description |
|------|-------------|
| topology.pkt | Cisco Packet Tracer simulation |
| ip-addressing.xlsx | Subnetting and VLAN planning spreadsheet |
| configs/Router-A.txt | Router-on-a-Stick config, all sub-interfaces and DHCP pools |
| configs/SW-A.txt | Switch A, access and trunk port config |
| configs/SW-B.txt | Switch B |
| configs/SW-C.txt | Switch C, includes trunk to AP |
