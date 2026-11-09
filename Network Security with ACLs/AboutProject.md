### Network Security with ACLs
**Folder:** [`Network Security with ACLs`](./Network%20Security%20with%20ACLs) · **Details:** [AboutProject.md](./Network%20Security%20with%20ACLs/AboutProject.md)

A small office network split into IT, Finance, Guest and Server VLANs, with access lists controlling who can reach what. Guests are blocked from internal networks, Finance can only use the web application, and only IT can log into the router over SSH. Switch ports are locked to known devices.

**Skills shown:** VLANs and inter-VLAN routing, standard and extended ACLs, SSH, port security, negative testing.

**Tools:** Cisco Packet Tracer 8.2.2 · 1× 2911 router · 1× 2960 switch · 3× PCs · 1× server

---

### Enterprise Internet Access and Services
**Folder:** [`Enterprise Internet Access and Services`](./Enterprise%20Internet%20Access%20and%20Services) · **Details:** [AboutProject.md](./Enterprise%20Internet%20Access%20and%20Services/AboutProject.md)

An office LAN where PCs get addresses automatically, find a public website by name, and share a single ISP-facing address through PAT. Includes tests that separate DNS failures from routing and NAT failures.

**Skills shown:** DHCP, DNS, HTTP, PAT, default routing, service-level troubleshooting.

**Tools:** Cisco Packet Tracer 8.2.2 · 2× 2911 routers · 2× 2960 switches · 2× PCs · 2× servers

---

### Complete Enterprise Network Design
**Folder:** [`Complete Enterprise Network Design`](./Complete%20Enterprise%20Network%20Design) · **Details:** [AboutProject.md](./Complete%20Enterprise%20Network%20Design/AboutProject.md)

The capstone: a Nairobi headquarters with Mombasa and Kisumu branches, six VLANs, OSPF, DHCP, DNS, PAT to a simulated internet, a redundant campus core, and secure corporate and guest wireless. Security policies apply the same way to wired and wireless users, and every feature is tested with one-at-a-time fault injection.

**Skills shown:** Everything from the earlier labs combined, plus EtherChannel, Rapid PVST, WPA2 wireless, and a full acceptance test.

**Tools:** Cisco Packet Tracer 8.2.2 · 4× 2911 routers + ISP · 6× 2960 switches · 2× WRT300N · PCs, laptops, 2× servers
