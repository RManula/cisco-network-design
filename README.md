# Network Design Lab: OSPF, VLANs, and Protocol Analysis

Built a small enterprise network from scratch in Cisco Packet Tracer, starting with a basic 3-subnet routed topology and building it up through OSPF dynamic routing, DHCP, VLAN segmentation, and a wireless segment. Also analyzed an SMTP packet capture in Wireshark and traced how Layer 2 and Layer 3 addresses behave across router hops.

The full topology runs 12 PCs (PC-A through PC-L) across 3 subnets, 5 switches, and 3 routers connected in a triangle via serial links.

---

## Modules

| | Module | What it covers |
|-|--------|---------------|
| 1 | [SMTP Analysis](task-1-smtp-analysis/) | Wireshark capture of a full SMTP session, TCP handshake to session close |
| 2 | [Network Design](task-2-network-design/) | 3-subnet routed network, OSPF, DHCP, failover testing |
| 3 | [VLAN Implementation](task-3-vlan/) | VLAN segmentation, Router-on-a-Stick, wireless on a dedicated VLAN |
| 4 | [L2/L3 Address Analysis](task-4-address-analysis/) | How MAC and IP addresses change at each router hop |

---

## Module 2 - Network Design

Three subnets, one router each, connected over serial links with OSPF handling all routing automatically.

### Topology

![Network Topology](task-2-network-design/screenshots/topology.png)

The network has 3 routers (R-A, R-B, R-C) forming a triangle in the center. Each router owns one subnet and connects to the other two via serial links. Subnet A is the largest with 6 PCs and 3 switches. Subnets B and C each have 3 PCs and a switch.

### Subnets

| Subnet | Network | PCs | Gateway |
|--------|---------|-----|---------|
| Subnet A | 192.168.10.0/24 | PC-A, B, C, D, E, F | 192.168.10.1 (R-A, Fa0/0) |
| Subnet B | 192.168.20.0/24 | PC-G, H, I | 192.168.20.1 (R-B, Fa0/0) |
| Subnet C | 192.168.30.0/24 | PC-J, K, L | 192.168.30.1 (R-C, Fa0/0) |

### Router Serial Interconnects

| Link | Interface | IP | Network |
|------|-----------|----|---------|
| R-A to R-B | Se2/0 | 10.0.0.1 / 10.0.0.2 | 10.0.0.0/24 |
| R-A to R-C | Se3/0 | 10.0.2.1 / 10.0.2.2 | 10.0.2.0/24 |

OSPF runs on all three routers in area 0. Each router advertises its connected networks and learns the others dynamically, no static routes anywhere. DHCP is configured on each router to serve its own subnet, so all 12 PCs get their IP, mask, gateway, and DNS automatically.

Tested failover by disabling the R-A to R-B serial link. OSPF rerouted traffic through R-C automatically within seconds, no config changes needed.

---

## Module 3 - VLAN Implementation

Same physical topology as Module 2, but with three VLANs overlaid and inter-VLAN routing handled by a single Router-on-a-Stick configuration on R-A.

### Topology

![VLAN Topology](task-3-vlan/screenshots/topology.png)

### VLAN Design

The original /24 address space was subnetted into three ranges, one per VLAN:

```
192.168.1.0/24
  192.168.1.0/25      VLAN 225  (Wi-Fi)       128 hosts
  192.168.1.128/25
    192.168.1.128/26  VLAN 75   (Group A)      62 hosts
    192.168.1.192/26  VLAN 150  (Group B)      62 hosts
```

| VLAN | ID | Subnet | Sub-interface | Members |
|------|----|--------|---------------|---------|
| VLAN 75 | 75 | 192.168.1.128/26 | Fa0/0.75, gateway .129 | PC-A, PC-D, PC-F |
| VLAN 150 | 150 | 192.168.1.192/26 | Fa0/0.150, gateway .193 | PC-B, C, E, G, H, I, J, K, L |
| VLAN 225 | 225 | 192.168.1.0/25 | Fa0/0.225, gateway .1 | Laptops via AP |

R-A handles all inter-VLAN routing on a single physical port using 802.1Q sub-interfaces. Switches use trunk ports toward R-A and access ports for end devices. The wireless AP sits on SW-C with SSID `VLAN 225 Wi-Fi` and WPA-PSK, laptops connect wirelessly and get DHCP addresses from the VLAN 225 pool.

Tested intra-VLAN pings (PC-A to PC-D), inter-VLAN pings (PC-A to VLAN 150 and 225), and interface disable/enable to confirm isolation and recovery.

---

## Module 1 - SMTP Analysis

Analyzed a Wireshark capture (`mail_sender_attachment.pcapng`) of a complete SMTP session. The session runs without TLS so the entire exchange is visible in plain text, including the full command sequence, message headers, body, and a Base64-encoded attachment. Good for seeing exactly what SMTP looks like on the wire.

Details in [task-1-smtp-analysis/](task-1-smtp-analysis/).

---

## Module 4 - L2/L3 Address Analysis

Traced a ping from PC-A to PC-L using Packet Tracer simulation mode and recorded the MAC and IP addresses at every interface along the path. IP stays the same end to end. MAC gets rewritten at each router because each segment only needs to know the two devices on that link.

Full hop-by-hop table and explanation in [task-4-address-analysis/](task-4-address-analysis/).

---

## Repo Structure

```
task-1-smtp-analysis/
  mail_sender_attachment.pcapng
task-2-network-design/
  topology.pkt            open in Cisco Packet Tracer
  ip-addressing.xlsx      IP planning spreadsheet
  screenshots/
  configs/
    Router-A.txt          OSPF + DHCP config
    Router-B.txt
    Router-C.txt
    switches/Switch-A..E.txt
task-3-vlan/
  topology.pkt
  ip-addressing.xlsx
  screenshots/
  configs/
    Router-A.txt          Router-on-a-Stick config with all sub-interfaces
    SW-A.txt, SW-B.txt, SW-C.txt
task-4-address-analysis/
  README.md
```

## Tools

Cisco Packet Tracer, Wireshark

`.pkt` files need [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer), free with a NetAcad account. `.pcapng` opens in [Wireshark](https://www.wireshark.org/).
