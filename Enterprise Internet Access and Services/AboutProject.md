# Enterprise Internet Access and Services

## Scenario
Employees get network settings automatically, reach a public website by name, and share one outside address when going to the simulated internet.

## Technologies
- DHCP with excluded addresses
- DNS (A records) and HTTP servers
- PAT (NAT overload)
- Default routing
- Packet Tracer Simulation mode for tracing traffic

## Topology

```
 PC-A   PC-B   INTERNAL-DNS
   \      |      /
        [SW-LAN]
           |
        [R-EDGE] ---- [ISP] ---- [SW-PUBLIC] ---- PUBLIC-WEB
        inside        outside                      203.0.113.10
```

## Addressing

| Device | Address | Gateway |
|---|---|---|
| R-EDGE G0/0 (inside) | 10.5.10.1/24 | n/a |
| INTERNAL-DNS | 10.5.10.10/24 | 10.5.10.1 |
| R-EDGE G0/1 (outside) | 198.51.100.2/30 | n/a |
| ISP G0/0 | 198.51.100.1/30 | n/a |
| ISP G0/1 | 203.0.113.1/24 | n/a |
| PUBLIC-WEB | 203.0.113.10/24 | 203.0.113.1 |
| PC-A, PC-B | DHCP | Learned from DHCP |

The 198.51.100.0 and 203.0.113.0 ranges are reserved documentation ranges, standing in for a WAN and a public network.

## Design decisions

| Decision | Why |
|---|---|
| Exclude 10.5.10.1 to .20 from DHCP | Keeps router and server addresses from being leased to PCs. |
| DHCP on the edge router | The router is directly on the LAN, so no helper address is needed. |
| PAT with `overload` | Many private hosts share the single outside address, with sessions told apart by port number. |
| ACL 1 selects what is translated | It chooses traffic for NAT. It is not a security filter. |
| ISP has no route to the private LAN | Replies go to the translated address on its connected WAN, so none is needed. |

## Key configuration

```
ip dhcp excluded-address 10.5.10.1 10.5.10.20
ip dhcp pool EMPLOYEES
 network 10.5.10.0 255.255.255.0
 default-router 10.5.10.1
 dns-server 10.5.10.10

access-list 1 permit 10.5.10.0 0.0.0.255
interface g0/0
 ip nat inside
interface g0/1
 ip nat outside
ip nat inside source list 1 interface g0/1 overload
ip route 0.0.0.0 0.0.0.0 198.51.100.1
```

## Validation

| Action | Expected result |
|---|---|
| PCs get an address | In 10.5.10.0/24 above .20, gateway .1, DNS .10 |
| Ping the gateway, then DNS server | Both succeed |
| Ping 203.0.113.10 | Succeeds through PAT |
| Open `http://203.0.113.10` | Loads (works without DNS) |
| Open `http://www.portfolio.test` | Loads (DNS and HTTP both work) |
| `show ip nat translations` | 10.5.10.x mapped to 198.51.100.2 |
| `show ip dhcp binding` | Both PCs listed |

## Failure tests

| Fault | What you see | What it proves |
|---|---|---|
| DNS turned off | Name fails, IP still works | The fault is the name service, not routing |
| PAT removed | Public access fails, local gateway and DNS still work | The ISP has no route to private addresses |

Restore both after each test.

## Limitations
- One edge router and one ISP, so there is no redundancy.
- NAT hides addresses but is not an access-control policy.
- The "internet" is a simulated network inside Packet Tracer.

## Learning objectives applied
- Automate addressing with DHCP and protect infrastructure addresses
- Explain how a name becomes an address (DNS) and how that differs from reaching the server (routing and HTTP)
- Configure PAT and read the NAT table
- Tell routing, translation and application problems apart from the symptoms
- Use Simulation mode to follow DHCP, DNS and HTTP exchanges

## Files
| File | Description |
|---|---|
| [`Enterprise Internet Access and Services.pkt`](./Enterprise%20Internet%20Access%20and%20Services.pkt) | Working Packet Tracer file |
