# Layer 2 and Layer 3 Address Analysis

Used Packet Tracer simulation mode to trace a ping from PC-A to PC-L and recorded the MAC and IP addresses at every interface along the path. The goal was to understand how Layer 2 and Layer 3 addresses behave differently as a packet moves through a routed network.

## The Core Difference

Layer 3 (IP) addresses identify the source and destination of the communication end to end. They do not change anywhere along the path.

Layer 2 (MAC) addresses identify devices on a single network segment. They get rewritten at every router because each segment is a separate link, and the router needs to address the next device on that link specifically.

IP tells the packet where it ultimately needs to go. MAC tells it who to hand it to right now.

## Traced Path

Source: PC-A, IP 192.168.10.3, in Subnet A  
Destination: PC-L, IP 192.168.30.3, in Subnet C  
Route taken: PC-A, R-A (Router0), R-C (Router4), PC-L

## Address at Each Hop

| Device | Interface | Direction | Source IP | Dest IP | Source MAC | Dest MAC |
|--------|-----------|-----------|-----------|---------|------------|----------|
| PC-A | Fa0 | out | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| R-A | Fa0/1 | in | 192.168.10.3 | 192.168.30.3 | 0030.F28D.A517 | 000A.F3C5.7D08 |
| R-A | Se3/0 | out | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| R-C | Se2/0 | in | 192.168.10.3 | 192.168.30.3 | 000A.F3C5.7D08 | 0002.175A.D5C8 |
| R-C | Fa0/1 | out | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |
| PC-L | Fa0 | in | 192.168.10.3 | 192.168.30.3 | 0002.175A.D5C8 | 0002.4469.D303 |

## What Happens at Each Step

**PC-A sending the packet**

PC-A checks its routing table. 192.168.30.3 is not on its subnet, so it sends the packet to its default gateway, R-A at 192.168.10.1. PC-A does an ARP lookup for R-A's MAC and sets that as the destination MAC. PC-L's MAC address is irrelevant at this point because PC-A and PC-L are not on the same segment.

**R-A forwarding to R-C**

R-A receives the frame on Fa0/1, addressed to its own MAC so it accepts it. It strips the Layer 2 header and looks at the IP destination, 192.168.30.3. Its routing table says next hop is R-C via Se3/0. R-A builds a new Layer 2 header for the serial link, puts its own Se3/0 MAC as the source and R-C's MAC as the destination. IP addresses stay the same.

**R-C delivering to PC-L**

R-C receives the frame on Se2/0. Same process: strip the Layer 2 header, look up 192.168.30.3 in the routing table, find it on the directly connected Fa0/1 network. R-C does an ARP for PC-L's MAC, builds the final Layer 2 header, and delivers the frame. The IP addresses PC-A set at the start arrive at PC-L unchanged.

## Why This Matters

MAC addresses are only meaningful within a single broadcast domain. Once a packet crosses a router, the previous MAC pair is completely discarded and replaced with the addresses of the two devices on the new segment. This is why you cannot route based on MAC addresses and why IP was designed to be the persistent identifier across the whole network.
