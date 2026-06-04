# Task 2 - Multi-Subnet Network Design

Three-subnet routed network built in Cisco Packet Tracer. Each subnet has a dedicated router, routers connect over serial links, OSPF handles routing, and DHCP handles IP allocation.

## Topology

![Network Topology](screenshots/topology.png)

## IP Addressing

| Subnet | Network | Usable Range | Gateway | DNS |
|--------|---------|-------------|---------|-----|
| Subnet A | 192.168.10.0/24 | .1 to .254 | 192.168.10.1 | 192.168.10.1 |
| Subnet B | 192.168.20.0/24 | .1 to .254 | 192.168.20.1 | 192.168.20.1 |
| Subnet C | 192.168.30.0/24 | .1 to .254 | 192.168.30.1 | 192.168.30.1 |

## Router Serial Interconnects

| Link | Interface on A | Remote Interface | Network |
|------|---------------|-----------------|---------|
| A to B | Se2/0 - 10.0.0.1 | Se2/0 - 10.0.0.2 | 10.0.0.0/24 |
| A to C | Se3/0 - 10.0.2.1 | Se3/0 - 10.0.2.2 | 10.0.2.0/24 |

## Routing

All three routers run OSPF process 1 in area 0. Each router advertises its directly connected networks and learns the rest automatically. No static routes.

```
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 0
 network 10.0.2.0 0.0.0.255 area 0
```

OSPF was the right call here because of failover. If a serial link goes down, OSPF recalculates and reroutes without any manual changes. Tested this by disabling the A-B link and confirming traffic rerouted through C automatically.

## DHCP

Each router serves as DHCP for its own subnet. PCs get their IP, mask, gateway, and DNS on boot.

```
ip dhcp pool SubnetA
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.1
```

## Simulation Tests

Normal path, PC-A to PC-H:

```
PC-A -> Switch -> Router A -> Router B -> Switch -> PC-H
```

With Router A-B link disabled:

```
PC-A -> Router A -> Router C -> Router B -> PC-H
```

OSPF converged and rerouted without any config changes.

## Limitations

Give OSPF and DHCP a few seconds to converge on first boot before testing. If you ping immediately after startup it will likely fail. For critical devices, static IPs avoid this.

There are no redundant physical links between routers. OSPF handles single link failures but if two links go down at once, connectivity breaks. VLANs were not used here, so all devices in a subnet share the same broadcast domain. That gets addressed in Task 3.

## Files

| File | Description |
|------|-------------|
| topology.pkt | Packet Tracer simulation |
| ip-addressing.xlsx | IP allocation spreadsheet |
| configs/Router-A.txt | Router A config (OSPF + DHCP for Subnet A) |
| configs/Router-B.txt | Router B config |
| configs/Router-C.txt | Router C config |
| configs/switches/ | Switch A through E configs |
