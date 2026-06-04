# Cisco Network Infrastructure Design and Protocol Analysis

Four networking modules built in Cisco Packet Tracer and Wireshark. Covers protocol analysis, multi-subnet routing with OSPF, VLAN segmentation with Router-on-a-Stick, and OSI Layer 2/3 address behavior across hops.

---

## Modules

| Module | Topic |
|--------|-------|
| [1 - SMTP Analysis](task-1-smtp-analysis/) | Wireshark capture of a full SMTP email session |
| [2 - Network Design](task-2-network-design/) | Three-subnet routed network with OSPF and DHCP |
| [3 - VLAN Implementation](task-3-vlan/) | VLAN segmentation and inter-VLAN routing |
| [4 - Address Analysis](task-4-address-analysis/) | How MAC and IP addresses behave across router hops |

---

## Module 1 - SMTP Protocol Analysis

Analysis of a Wireshark capture (`mail_sender_attachment.pcapng`) showing a complete SMTP session, from the TCP handshake through to session teardown. The email includes an attachment sent as Base64 in the DATA payload.

The session runs without TLS so the full content, sender, recipient, and attachment are all visible in plain text. Good for understanding exactly what SMTP looks like on the wire before encryption is added.

Files: [task-1-smtp-analysis/](task-1-smtp-analysis/)

---

## Module 2 - Multi-Subnet Network Design

Three-subnet routed network with a router per subnet, serial interconnects between routers, OSPF for dynamic routing, and DHCP on each router for automatic IP allocation.

### Topology

![Network Topology](task-2-network-design/screenshots/topology.png)

### Addressing

| Subnet | Network | Gateway |
|--------|---------|---------|
| Subnet A | 192.168.10.0/24 | 192.168.10.1 (Router A) |
| Subnet B | 192.168.20.0/24 | 192.168.20.1 (Router B) |
| Subnet C | 192.168.30.0/24 | 192.168.30.1 (Router C) |

### Router Serial Links

| Link | Router A | Remote | Network |
|------|----------|--------|---------|
| A to B | Se2/0 - 10.0.0.1 | Se2/0 - 10.0.0.2 | 10.0.0.0/24 |
| A to C | Se3/0 - 10.0.2.1 | Se3/0 - 10.0.2.2 | 10.0.2.0/24 |

OSPF is used instead of static routing so the network recovers automatically from link failures. Tested by disabling the A-B serial link, OSPF rerouted traffic through Router C with no manual changes.

Files: [task-2-network-design/](task-2-network-design/)

---

## Module 3 - VLAN Implementation

Extends the Module 2 topology with three VLANs and a Router-on-a-Stick setup. The original /24 was subnetted to give each VLAN its own address range.

### Topology

![VLAN Topology](task-3-vlan/screenshots/topology.png)

### Subnetting

```
192.168.1.0/24
  192.168.1.0/25    VLAN 225 (Wi-Fi laptops)
  192.168.1.128/25
    192.168.1.128/26  VLAN 75  (PC-A, PC-D, PC-F)
    192.168.1.192/26  VLAN 150 (remaining PCs)
```

### VLAN Table

| VLAN | ID | Subnet | Sub-interface | Devices |
|------|----|--------|---------------|---------|
| VLAN 75 | 75 | 192.168.1.128/26 | Fa0/0.75 | PC-A, PC-D, PC-F |
| VLAN 150 | 150 | 192.168.1.192/26 | Fa0/0.150 | Remaining PCs |
| VLAN 225 | 225 | 192.168.1.0/25 | Fa0/0.225 | Laptops via AP |

Router A handles inter-VLAN routing on a single physical interface using 802.1Q sub-interfaces. The access point is on VLAN 225 with WPA-PSK, SSID `VLAN 225 Wi-Fi`.

Files: [task-3-vlan/](task-3-vlan/)

---

## Module 4 - Layer 2/3 Address Analysis

Traces a ping from PC-A (`192.168.10.3`) to PC-L (`192.168.30.3`) through the network and records how MAC addresses change at each router while IP addresses stay constant end-to-end.

| Device | Interface | Direction | Source IP | Dest IP | Source MAC | Dest MAC |
|--------|-----------|-----------|-----------|---------|------------|----------|
| PC-A | Fa0 | out | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| Router A | Fa0/1 | in | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| Router A | Se3/0 | out | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| Router C | Se2/0 | in | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| Router C | Fa0/1 | out | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |
| PC-L | Fa0 | in | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |

Files: [task-4-address-analysis/](task-4-address-analysis/)

---

## Repo Structure

```
README.md
Report.docx
task-1-smtp-analysis/
  mail_sender_attachment.pcapng
task-2-network-design/
  topology.pkt
  ip-addressing.xlsx
  screenshots/
  configs/
    Router-A.txt, Router-B.txt, Router-C.txt
    switches/Switch-A..E.txt
task-3-vlan/
  topology.pkt
  ip-addressing.xlsx
  screenshots/
  configs/
    Router-A.txt, SW-A.txt, SW-B.txt, SW-C.txt
task-4-address-analysis/
  README.md
```

## Tools

Cisco Packet Tracer, Wireshark, OSPF, DHCP, IEEE 802.1Q

Open `.pkt` files with [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer) (free NetAcad account required).  
Open `.pcapng` with [Wireshark](https://www.wireshark.org/).
