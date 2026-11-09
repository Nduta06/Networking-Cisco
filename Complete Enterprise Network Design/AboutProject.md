# Complete Enterprise Network Design

## Scenario
A company with a Nairobi headquarters and branches in Mombasa and Kisumu. The design combines the techniques from the earlier labs into one network: VLANs, OSPF, DHCP, DNS, address translation, security policy, redundancy and wireless.

## Technologies
- VLANs, 802.1Q trunks and inter-VLAN routing
- OSPF (single area) with a default route from HQ
- EtherChannel (LACP) and Rapid PVST
- DHCP and DNS (internal and public)
- PAT at a dedicated edge router
- Extended and standard ACLs, SSH, port security
- WPA2 wireless on WRT300N access points

## Topology (summary)

```
 Simulated internet:  PUBLIC-SRV - SW-PUBLIC - ISP - EDGE
                                                      |  VLAN 99 (transit)
 HQ campus:    HQ -- D1 ==Po1== D2        (ASW dual-homed to D1 and D2)
                                            wired endpoints + 2 APs on ASW
 WAN triangle: HQ -- MBA -- KSM -- HQ       (each branch: router, switch, PC)
```

See `topology.png` for the full labeled diagram.

## Addressing

| Network | Subnet | Notes |
|---|---|---|
| HQ IT, Finance, Staff, Servers, Guest | 10.8.10.0, .20.0, .30.0, .40.0, .50.0 (/24) | HQ gateway is .1 in each |
| HQ transit VLAN 99 | 10.8.99.0/24 | HQ .1, EDGE .2 |
| HQ–MBA / MBA–KSM / KSM–HQ | 10.8.0.0, .0.4, .0.8 (/30) | WAN triangle |
| Mombasa / Kisumu LANs | 10.8.60.0/24, 10.8.70.0/24 | DHCP at the branch |
| EDGE–ISP | 198.51.100.0/30 | EDGE .2, ISP .1 |
| Public LAN | 203.0.113.0/24 | PUBLIC-SRV .10 |

Switch management addresses are 10.8.10.2 to .4 in VLAN 10. Clients exclude .1 to .20 from DHCP. OSPF router IDs: HQ 1.1.1.1, MBA 2.2.2.2, KSM 3.3.3.3.

## Design decisions

| Decision | Why |
|---|---|
| Separate transit VLAN 99 to EDGE | Keeps the internet edge off any client network. |
| ISP kept out of OSPF | Internal routes are not shared with the outside. |
| `passive-interface default` | OSPF only forms neighbors on the WAN links. |
| `default-information originate` on HQ | Branches learn the internet route automatically, and only while HQ has one. |
| EtherChannel between D1 and D2 | Adds bandwidth and survives the loss of one member link. |
| ASW dual-homed | Spanning tree blocks one path and keeps the other as backup. |
| Policy applied per VLAN | A wireless guest gets exactly the same restrictions as a wired guest. |
| Port security only on single-device access ports | AP and uplink ports carry many MACs and are left out. |

## Security policy

| Group | Allowed | Denied |
|---|---|---|
| Guest (wired and wireless) | DHCP, internet | Internal 10.x, 172.16.x and 192.168.x ranges |
| Finance | DNS and HTTP to the internal server, internet | Everything else in the server VLAN |
| IT | SSH to all routers and switches | n/a |
| Everyone else | Internal and public web | SSH to routers |

## Validation

| Check | Expected result |
|---|---|
| Clients renew DHCP | Correct subnet, gateway and DNS, no duplicates |
| Internal site by name | Loads for IT, Staff and both branches |
| Finance opens internal site, pings server | Page loads, ping denied by policy |
| Public site by name from every client | Loads, translations created on EDGE |
| Guests ping internal addresses | All denied; public site still works |
| IT connects by SSH | Works on all routers and HQ switches |
| Staff connects by SSH to HQ | Fails (VTY ACL) |
| `show ip ospf neighbor` on HQ, MBA, KSM | Two FULL neighbors each |
| `show etherchannel summary` | Po1 healthy |

## Fault tests

| Fault | Result |
|---|---|
| Shut HQ–MBA link | Branch still reaches intranet and public site through KSM |
| Shut one Po1 member link | Bundle stays up on the other link |
| Shut the active ASW uplink | STAFF regains access over the alternate path |
| Turn off internal DNS | Internal names fail, IP still works, guest public DNS unaffected |
| Wrong wireless key | Association fails until corrected |
| Swap the IT PC on ASW | Port-security violation, then documented recovery |

## Limitations
- HQ gateway, EDGE and ISP are single devices, so their failure still takes services down. Redundant gateways (HSRP) are covered in a separate lab.
- This is a Cisco Packet Tracer simulation, not production equipment.
- WPA2 Personal uses a shared passphrase, not enterprise 802.1X authentication.
- Lab-only credentials are used throughout.

## Learning objectives applied
- Design a multi-site addressing plan with VLANs and point-to-point links
- Combine OSPF, default routing and PAT into one working design
- Build redundant Layer 2 paths with EtherChannel and spanning tree
- Apply one consistent security policy to wired and wireless users
- Secure management with SSH and restrict who may use it
- Run a full acceptance test, then inject faults one at a time and recover
- State clearly what is redundant and what is not

## Files
| File | Description |
|---|---|
| [`Complete Enterprise Network Design.pkt`](./Complete%20Enterprise%20Network%20Design/Complete%20Enterprise%20Network%20Design.pkt) | Working Packet Tracer file |
