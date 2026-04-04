# Enterprise Network Lab (Beginner-Level)

## 1. Overview

This lab simulates a small enterprise network using Cisco technologies.
It demonstrates Layer 2 and Layer 3 design, segmentation, routing, security, and basic service integration (DHCP, DNS, NAT).

The network is built with:

* Multi-VLAN architecture (VLAN 10 & VLAN 20)
* Inter-VLAN routing using Router-on-a-Stick
* Layer 2 redundancy and optimization (STP + EtherChannel)
* Network services (DHCP, DNS)
* Security mechanisms (ACL, Port Security, SSH)
* Internet simulation via upstream router

---

## 2. Network Topology

```
              +-------------------+
              |   Internet Server |
              |     8.8.8.8       |
              +---------+---------+
                        |
                    8.8.8.0/24
                        |
              +---------+---------+
              |        R2         |
              |  Internet Router  |
              |   8.8.8.1         |
              +---------+---------+
                        |
                   10.0.0.0/30
                        |
              +---------+---------+
              |        R1         |
              | Router-on-a-Stick|
              +---------+---------+
                        |
                      (Trunk)
                        |
              +---------+---------+
              |        SW3        |
              |   Aggregation     |
              +----+--------+-----+
                   |        |
           (Trunk) |        | (Trunk)
                   |        |
          +--------+---+  +---+--------+
          |    SW2     |  |    SW1     |
          |  VLAN 10   |  |  VLAN 20   |
          +-----+------+  +------+-----+
                |                |
         Server_DHCP+DNS      Staff_PC
          (192.168.10.x)   (192.168.20.x)

   SW1 <======== EtherChannel (LACP) ========> SW2
```

## 3. IP Addressing Plan

| Device | Interface | IP Address       |
| ------ | --------- | ---------------- |
| R1     | G0/0.10   | 192.168.10.1/24  |
| R1     | G0/0.20   | 192.168.20.1/24  |
| R1     | G0/1      | 10.0.0.1/30      |
| R2     | G0/0      | 10.0.0.2/30      |
| R2     | G0/1      | 8.8.8.1/24       |
| Server | -         | 192.168.10.10/24 |

---

## 4. VLAN & Switching Configuration

* VLAN 10: SALES
* VLAN 20: IT

### Features Implemented

* VLAN segmentation across switches
* Trunking between switches and router
* EtherChannel (LACP - active mode)
* Spanning Tree Protocol (PVST)

### STP Optimization

* VLAN 10 root bridge → SW2
* VLAN 20 root bridge → SW1

This design enables **load balancing at Layer 2**.

---

## 5. Layer 2 Security

### Port Security

* Enabled on access ports
* Maximum MAC addresses: 2
* Violation mode: shutdown
* Sticky MAC learning

### STP Protection

* PortFast enabled on access ports
* BPDU Guard enabled to prevent rogue switches
* Root Guard configured to maintain STP topology stability

---

## 6. Inter-VLAN Routing (Router-on-a-Stick)

R1 is configured with subinterfaces:

* G0/0.10 → VLAN 10
* G0/0.20 → VLAN 20

Each VLAN has:

* Default gateway on R1
* DHCP relay using `ip helper-address`

---

## 7. DHCP & DNS Server

### Server Configuration

* IP: 192.168.10.10
* Default Gateway: 192.168.10.1

### DHCP Scopes

* VLAN 10: 192.168.10.100 – 192.168.10.200
* VLAN 20: 192.168.20.100 – 192.168.20.200

### DNS

* A record:

  * `google.com` → 8.8.8.8
  * `web_server.local` → 192.168.10.10

---

## 8. NAT (Internet Access Simulation)

* NAT overload (PAT) configured on R1
* Inside interfaces: VLAN subinterfaces
* Outside interface: G0/1

This allows internal hosts to access the simulated Internet (8.8.8.0/24).

---

## 9. Access Control (ACL)

### Named Extended ACL: `GUEST-FILTER`

Applied inbound on VLAN 20:

* Deny ICMP (echo) from VLAN 20 → VLAN 10
* Allow ICMP echo-reply
* Deny HTTP (TCP port 80) from VLAN 20
* Allow DNS (UDP 53)
* Allow DHCP (UDP 67/68)
* Permit all remaining traffic

### Behavior

* VLAN 10 → VLAN 20: allowed
* VLAN 20 → VLAN 10: restricted

---

## 10. Remote Access

* Telnet disabled
* SSH enabled (key-based authentication)
* Secure remote management via VTY lines

---

## 11. Internet Simulation (R2)

R2 acts as an upstream provider:

* Network: 8.8.8.0/24
* Static route configured to simulate Internet reachability

---

## 12. Verification & Testing

### Connectivity Tests

* PC ↔ Server communication
* Inter-VLAN routing validation
* DHCP IP assignment
* DNS resolution:

  ```
  nslookup web.local 192.168.10.10
  ```

### ACL Behavior

* VLAN 20 cannot ping VLAN 10
* VLAN 10 can initiate communication

### NAT

* Internal hosts can ping 8.8.8.8

### Layer 2

* EtherChannel operational between SW1 and SW2
* No STP loops detected

---

## 13. Key Features Summary

* Multi-VLAN segmentation
* Inter-VLAN routing (Router-on-a-Stick)
* EtherChannel (LACP)
* STP optimization (per-VLAN root bridge)
* DHCP & DNS services
* NAT (PAT overload)
* ACL-based traffic filtering
* Layer 2 security (Port Security, BPDU Guard, Root Guard)
* Secure remote access (SSH)

---

## 14. Conclusion

This lab demonstrates a complete small-scale enterprise network design, integrating switching, routing, security, and network services. It reflects practical CCNA-level knowledge and real-world network deployment concepts.

---
