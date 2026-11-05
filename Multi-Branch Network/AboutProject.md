# Multi-Branch Network with OSPF

## Scenario
A company has offices in Nairobi (HQ), Mombasa and Kisumu. Each office has its own LAN, and the three offices must reach each other. The routers are connected in a triangle so that if one link fails, traffic still has another way through.

Instead of typing every route by hand (static routing), the routers use **OSPF** to discover each other's networks and recalculate the best path when something changes.

## Technologies
- IPv4 addressing and subnetting (/24 LANs, /30 point-to-point links)
- Router interface configuration
- OSPFv2, single area (Area 0)
- Passive interfaces
- Basic Layer 2 switching (default VLAN 1)
- Verification with `show` commands, `ping` and `tracert`

## Topology

```
                    [MSA]  Mombasa
                  G0/1   G0/2
                   /       \
        10.3.0.0/30         10.3.0.4/30
                 /           \
             G0/1             G0/1
           [HQ] ------------- [KSM]
        Nairobi   10.3.0.8/30   Kisumu
             G0/2             G0/2

Each router also connects through G0/0 to a local switch and PC.
```

## Addressing

**LANs**

| Site | Router (G0/0) | PC | Gateway |
|---|---|---|---|
| HQ (Nairobi) | 10.3.10.1/24 | 10.3.10.10 | 10.3.10.1 |
| MSA (Mombasa) | 10.3.20.1/24 | 10.3.20.10 | 10.3.20.1 |
| KSM (Kisumu) | 10.3.30.1/24 | 10.3.30.10 | 10.3.30.1 |

**WAN links (router to router)**

| Link | Subnet | Side A | Side B |
|---|---|---|---|
| HQ ↔ MSA | 10.3.0.0/30 | HQ G0/1: 10.3.0.1 | MSA G0/1: 10.3.0.2 |
| MSA ↔ KSM | 10.3.0.4/30 | MSA G0/2: 10.3.0.5 | KSM G0/1: 10.3.0.6 |
| KSM ↔ HQ | 10.3.0.8/30 | KSM G0/2: 10.3.0.9 | HQ G0/2: 10.3.0.10 |

**OSPF router IDs:** HQ `1.1.1.1` · MSA `2.2.2.2` · KSM `3.3.3.3`

## Design decisions

| Decision | Why |
|---|---|
| /30 subnets on WAN links | A router-to-router link only needs 2 usable addresses, so a /30 wastes nothing. |
| Triangle layout | Every site has two paths to every other site, so one cable failure doesn't cut anyone off. |
| OSPF instead of static routes | Routes are learned automatically and adjust after a failure without manual changes. |
| Manual router IDs | Fixed, predictable IDs make neighbor tables easier to read and troubleshoot. |
| Passive interface on G0/0 | The LAN still gets advertised, but no OSPF messages are sent toward PCs. This is cleaner and safer. |
| One broad `network 10.3.0.0 0.0.255.255 area 0` | One statement covers every 10.3.x.x interface. This is fine in a small isolated lab. A larger network would use more specific statements. |

## Configuration

**HQ**
```
hostname HQ
interface GigabitEthernet0/0
 ip address 10.3.10.1 255.255.255.0
interface GigabitEthernet0/1
 ip address 10.3.0.1 255.255.255.252
interface GigabitEthernet0/2
 ip address 10.3.0.10 255.255.255.252
router ospf 1
 router-id 1.1.1.1
 passive-interface GigabitEthernet0/0
 network 10.3.0.0 0.0.255.255 area 0
```

**MSA**
```
hostname MSA
interface GigabitEthernet0/0
 ip address 10.3.20.1 255.255.255.0
interface GigabitEthernet0/1
 ip address 10.3.0.2 255.255.255.252
interface GigabitEthernet0/2
 ip address 10.3.0.5 255.255.255.252
router ospf 1
 router-id 2.2.2.2
 passive-interface GigabitEthernet0/0
 network 10.3.0.0 0.0.255.255 area 0
```

**KSM**
```
hostname KSM
interface GigabitEthernet0/0
 ip address 10.3.30.1 255.255.255.0
interface GigabitEthernet0/1
 ip address 10.3.0.6 255.255.255.252
interface GigabitEthernet0/2
 ip address 10.3.0.9 255.255.255.252
router ospf 1
 router-id 3.3.3.3
 passive-interface GigabitEthernet0/0
 network 10.3.0.0 0.0.255.255 area 0
```

## Validation

| Check | Command | Expected result |
|---|---|---|
| Interfaces up | `show ip interface brief` | All used interfaces `up/up` |
| Neighbors formed | `show ip ospf neighbor` | 2 neighbors per router, state `FULL` |
| Routes learned | `show ip route` | Remote LANs shown with `O` (OSPF), local LAN with `C` |
| End-to-end reachability | `ping` between all three PCs | All succeed |
| Path taken | `tracert` from HQ PC to MSA PC | Goes directly via the HQ–MSA link |

## Failure and recovery
1. **Baseline:** `tracert` from the HQ PC to the Mombasa PC takes the direct HQ → MSA path (lowest cost).
2. **Fault:** `shutdown` on HQ `G0/1` (the HQ–MSA link).
3. **Expected result:** After a brief convergence delay, OSPF reroutes traffic HQ → KSM → MSA. HQ loses its direct neighbor but keeps a route to the Mombasa LAN. A few pings may be lost while OSPF reconverges.
4. **Recovery:** `no shutdown` on the same interface. The direct path returns.

## Limitations
- This tests the loss of **one link**, not the loss of a whole router.
- Each LAN has a single switch and a single router, so these are still single points of failure.
- Single-area OSPF suits a small network. A large one would use multiple areas.
- No security features (ACLs, SSH, port security) are configured in this lab.
- Everything runs inside the Packet Tracer simulator.

## Learning objectives applied
- Plan and apply an IPv4 addressing scheme with LAN and point-to-point subnets
- Configure router interfaces and host gateways
- Explain and configure OSPF: process ID, router ID, Area 0, network statements and wildcard masks
- Use passive interfaces to stop routing traffic on user-facing ports
- Read neighbor tables and routing tables to confirm OSPF is working
- Demonstrate redundancy and convergence by breaking a link and tracing the new path
- Follow a build order: interfaces first, test, then routing

## Files
| File | Description |
|---|---|
| [`Multi-Branch Network with OSPF.pkt`](./Multi-Branch%20Network%20with%20OSPF.pkt) | Working Packet Tracer file (opens in 8.2.2 or later) |
