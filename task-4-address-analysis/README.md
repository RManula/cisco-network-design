# Task 4 - Layer 2/3 Address Analysis

Traces a ping from PC-A to PC-L through the Task 2 network and records how MAC and IP addresses change at each hop.

## The Short Version

IP addresses stay the same the whole way. MAC addresses get rewritten at every router. That is how routing works at the hardware level.

IP gets the packet to the right network. MAC gets it to the right device on each individual segment. Each segment only cares about the MACs for that link, not where the packet ultimately came from or is going.

## Path

PC-A (192.168.10.3) to PC-L (192.168.30.3) via Router A then Router C.

## Address Table

| Device | Interface | Direction | Source IP | Dest IP | Source MAC | Dest MAC |
|--------|-----------|-----------|-----------|---------|------------|----------|
| PC-A | Fa0 | out | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| Router A | Fa0/1 | in | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| Router A | Se3/0 | out | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| Router C | Se2/0 | in | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| Router C | Fa0/1 | out | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |
| PC-L | Fa0 | in | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |

## What Happens at Each Hop

PC-A knows 192.168.30.3 is not on its subnet so it sends the packet to its default gateway, Router A. The destination MAC is Router A's interface, not PC-L's. PC-L's MAC is meaningless to PC-A because they are not on the same segment.

Router A receives the frame addressed to itself, strips the Layer 2 header, checks the routing table, and determines the next hop is Router C via Se3/0. It builds a new Layer 2 header with its own egress MAC as source and Router C's MAC as destination. IP addresses are untouched.

Router C does the same thing. It receives the frame, checks the routing table, finds 192.168.30.3 on its directly connected interface, does an ARP lookup for PC-L's MAC, and sends the frame directly to PC-L.

PC-L receives the frame with the same source and destination IPs PC-A set originally. The MACs are now Router C and PC-L because those are the two devices on this last segment.
