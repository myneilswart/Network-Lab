# Small Business Enterprise Network Lab

A simulated small business network built in Cisco Packet Tracer demonstrating VLANs, inter-VLAN routing, DHCP, ACLs, and secure SSH management. Built as part of a Network+ portfolio project.

---

## Topology

```
                        CORE-RTR (Cisco 2911)
                        192.168.99.1 (Management)
                               |
                        CORE-SW (Cisco 2960)
                        192.168.99.2 (Management)
                               |
        ┌──────────┬──────────┬──────────┬──────────┐
      VLAN 10   VLAN 20   VLAN 30   VLAN 40   VLAN 99
       HR         IT       Guest    Servers   Management
```

---

## VLAN Design

| VLAN | Name       | Subnet              | Gateway        | DNS            |
|------|------------|---------------------|----------------|----------------|
| 10   | HR         | 192.168.10.0/24     | 192.168.10.1   | 192.168.40.10  |
| 20   | IT         | 192.168.20.0/24     | 192.168.20.1   | 192.168.40.10  |
| 30   | Guest      | 192.168.30.0/24     | 192.168.30.1   | 8.8.8.8        |
| 40   | Servers    | 192.168.40.0/24     | 192.168.40.1   | 8.8.8.8        |
| 99   | Management | 192.168.99.0/24     | 192.168.99.1   | —              |

---

## IP Address Scheme

| Device     | VLAN | IP Address       | Assignment |
|------------|------|------------------|------------|
| CORE-RTR   | 99   | 192.168.99.1     | Static     |
| CORE-SW    | 99   | 192.168.99.2     | Static     |
| HR-PC1     | 10   | 192.168.10.2     | DHCP       |
| HR-PC2     | 10   | 192.168.10.3     | DHCP       |
| IT-PC1     | 20   | 192.168.20.2     | Static     |
| IT-PC2     | 20   | 192.168.20.3     | DHCP       |
| GUEST-PC1  | 30   | 192.168.30.2     | DHCP       |
| GUEST-PC2  | 30   | 192.168.30.3     | DHCP       |
| SRV-01     | 40   | 192.168.40.10    | Static     |

---

## Technologies Used

- Cisco Packet Tracer
- Cisco 2911 Router
- Cisco 2960 Switch
- VLANs and 802.1Q trunking
- Router-on-a-stick inter-VLAN routing
- DHCP (router-hosted, per VLAN)
- Extended and Standard ACLs
- SSH v2 with RSA encryption
- Management VLAN isolation (VLAN 99)

---

## Network Concepts Demonstrated

### Inter-VLAN Routing (Router-on-a-Stick)
The router uses subinterfaces on GigabitEthernet0/0 to route between VLANs. Each subinterface is assigned a VLAN ID via 802.1Q encapsulation and acts as the default gateway for that VLAN.

### DHCP
DHCP pools are hosted on the router, providing centralised address allocation across all VLANs. Static IPs are excluded for gateways, servers, and the IT management host (IT-PC1).

### Management VLAN (VLAN 99)
A dedicated management VLAN isolates administrative access to network infrastructure. Only the router (192.168.99.1) and switch (192.168.99.2) have IPs in this VLAN.

### Access Control Lists

**ACL 100 — Guest VLAN Isolation** (applied inbound on g0/0.30)
Prevents Guest VLAN from accessing HR, IT, Server, and Management VLANs while allowing internet access.

```
access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.40.0 0.0.0.255
access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.99.0 0.0.0.255
access-list 100 permit ip any any
```

**ACL 110 — Management VLAN Protection** (applied inbound on g0/0.20)
Restricts access to the management VLAN (VLAN 99) to IT-PC1 only.

```
access-list 110 permit ip host 192.168.20.2 192.168.99.0 0.0.0.255
access-list 110 deny ip any 192.168.99.0 0.0.0.255
access-list 110 permit ip any any
```

**ACL 15 — SSH Access Control** (applied to VTY lines on router and switch)
Restricts SSH management access to IT-PC1 only.

```
access-list 15 permit host 192.168.20.2
```

### SSH v2
SSH v2 is configured on both the router and switch. RSA keys are generated with a 2048-bit modulus. VTY lines are restricted to SSH only — Telnet is disabled. A 5-minute exec timeout is enforced on the router.

### Password Hashing
User secrets are stored as type 5 (MD5) hashes due to Packet Tracer limitations, indicated by `secret 5` in the configuration. The `secret` keyword always hashes passwords, unlike the `password` keyword which stores them in plaintext. `service password-encryption` is not enabled as it only applies to `password` entries and provides weak reversible encryption — relying on `secret` is the correct approach.

---

## Switch Port Assignment

| Port    | VLAN | Device     |
|---------|------|------------|
| Fa0/1   | 10   | HR-PC1     |
| Fa0/2   | 10   | HR-PC2     |
| Fa0/3   | 20   | IT-PC1     |
| Fa0/4   | 20   | IT-PC2     |
| Fa0/5   | 30   | GUEST-PC1  |
| Fa0/6   | 30   | GUEST-PC2  |
| Fa0/7   | 40   | SRV-01     |
| Gi0/1   | Trunk| CORE-RTR   |

---

## Security Summary

| Control | Implementation |
|---|---|
| VLAN segmentation | 5 VLANs isolating HR, IT, Guest, Servers, Management |
| Guest isolation | ACL 100 blocks Guest from all internal VLANs |
| Management isolation | ACL 110 restricts VLAN 99 access to IT-PC1 only |
| SSH restrictions | ACL 15 on VTY lines permits only IT-PC1 |
| Telnet disabled | `transport input ssh` on all VTY lines |
| Session timeout | 5-minute exec-timeout on router VTY lines |
| Local authentication | Username/password with privilege 15 on router and switch |
| Console authentication | `login local` on console port prevents unauthenticated physical access |
| Password hashing | Secrets stored as MD5 (type 5) hashes — type 9 (Scrypt) in production |
| Native VLAN | Currently VLAN 1 — should be changed to unused VLAN in production |

---

## Files

```
/
├── configs/
│   ├── CORE-RTR.txt    
│   └── CORE-SW.txt     
└── packet-tracer/
│   ├── Network-Lab.pkt
└── screenshots/
    ├── GUEST to VLAN10 fail.png
    ├── GUEST to VLAN20 fail.png
    ├── GUEST to VLAN40 fail.png
    ├── GUEST to VLAN99 fail.png
    ├── IT-PC1 SSH_to_RTR success.png
    ├── IT-PC1 SSH_to_SW success.png
    ├── IT-PC1 to VLAN10 success.png
    ├── IT-PC1 to VLAN20 success.png
    ├── IT-PC1 to VLAN40 success.png
    ├── IT-PC1 to VLAN99 success.png
    ├── IT-PC2 SSH_to_RTR fail .png
    ├── IT-PC2 SSH_to_SW fail.png
    ├── RTR show-access-lists.png
    ├── RTR show-ip-dhcp-binding.png
    ├── RTR show-ip-interface-brief.png
    ├── RTR show-ip-ssh.png
    ├── SW show-interfaces-trunk.png
    ├── SW show-vlan-brief.png
    ├── topology.png
├── README.md

```

---

## Lessons Learned

- Always verify end-host default gateway configuration before troubleshooting ACLs or routing — a missing gateway on IT-PC1 caused significant troubleshooting time before the root cause was identified.
- `ip default-gateway` is a switch command only; routers use their routing table and the command should not be configured on a router.
- ACL direction matters — applying an ACL outbound on a management VLAN subinterface can block legitimate return traffic. Inbound ACLs on source VLANs are cleaner and easier to reason about.
- Packet Tracer has simulator limitations including no native SSH service on servers and limited `show` command support. RSA keys were successfully generated at 2048-bit. In a production environment ECDSA keys with SHA-256 (`ip ssh server algorithm hostkey ecdsa`) would be preferred.

---

## Production Improvements

In a real-world deployment the following enhancements would be made:

- ECDSA keys with SHA-256 instead of RSA/MD5
- Type 9 (Scrypt) password hashing instead of type 5 (MD5): `username admin privilege 15 algorithm-type scrypt secret <password>`
- Change native VLAN from VLAN 1 to an unused VLAN to mitigate VLAN hopping attacks
- Dedicated management workstation on VLAN 99 rather than IT VLAN
- RADIUS/TACACS+ for centralised authentication and accounting
- Port security on switch access ports to restrict MAC addresses
- DHCP snooping to prevent rogue DHCP servers
- Dynamic ARP Inspection (DAI) to prevent ARP spoofing
- Redundant links and HSRP for high availability
- Syslog server for centralised log collection
