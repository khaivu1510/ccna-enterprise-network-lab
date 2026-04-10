# Enterprise Network Lab (Beginner-Level)

---

## 1. Overview

This lab simulates a small enterprise network using Cisco technologies.  
It demonstrates VLAN segmentation, HSRP redundancy, routing, security policies, and network services.

### Key Technologies
- Multi-VLAN architecture (VLAN 10 & VLAN 20)
- Inter-VLAN routing with HSRP
- DHCP service on router (R1)
- Layer 2 STP + EtherChannel
- NAT overload (PAT)
- ACL-based traffic filtering
- SSH secure management
- Internet simulation via upstream router

---

## 2. Network Topology

             +----------------------+
             |   Internet Router    |
             +----------+-----------+
                      |     |
          10.0.0.0/30 |     |  11.0.0.0/30
                      |     |
                +-----+-----+-----+
                |                 |
         R1 (HSRP + DHCP)      R2 (HSRP)
         VLAN10: .2            VLAN10: .3
         VLAN20: .2            VLAN20: .3
                \             /
                 \           /
                  +---------+
                  |   SW3   |
                  +----+----+
                       |
                +------+------+
                |             |
               SW1           SW2
            (VLAN 20)     (VLAN 10)
        192.168.20.0/24 192.168.10.0/24   
                |             |
             PC/User       DNS Server
                          192.168.10.10

    SW1 <======== EtherChannel (LACP) ========> SW2

---

## 3. IP Addressing Plan

### HSRP Virtual Gateways

| VLAN   | Virtual IP         |
|--------|--------------------|
| VLAN 10 | 192.168.10.254     |
| VLAN 20 | 192.168.20.254     |

### Device IPs

| Device   | Interface | IP Address        |
|----------|----------|-------------------|
| R1       | G0/0.10  | 192.168.10.2/24   |
| R1       | G0/0.20  | 192.168.20.2/24   |
| R2       | G0/0.10  | 192.168.10.3/24   |
| R2       | G0/0.20  | 192.168.20.3/24   |
| Internet | To R1    | 10.0.0.1/30       |
| Internet | To R2    | 11.0.0.1/30       |
| DNS Server | -      | 192.168.10.10/24  |

---

## 4. HSRP Configuration

- VLAN 10 gateway: `192.168.10.254`
- VLAN 20 gateway: `192.168.20.254`

### Behavior
- R1 active for VLAN 10 (primary)
- R2 active for VLAN 20 (load balancing design)
- Automatic failover if a router fails

---

## 5. VLAN & Switching Configuration

- VLAN 10: SALES
- VLAN 20: IT
- Trunk links between switches and routers
- EtherChannel (LACP)
- STP PVST optimization

---

## 6. Layer 2 Security

- Port Security (sticky MAC, max 2 MACs)
- BPDU Guard enabled
- PortFast enabled
- Root Guard configured

---

## 7. Inter-VLAN Routing

Both R1 and R2 use Router-on-a-Stick:
G0/0.10 → VLAN 10
G0/0.20 → VLAN 20


All hosts use HSRP virtual gateway:
192.168.10.254
192.168.20.254


---

## 8. DHCP (ON ROUTER R1)

DHCP service is provided directly by **R1**

### DHCP Pools
- VLAN 10: `192.168.10.4 – 192.168.10.253`
- VLAN 20: `192.168.20.4 – 192.168.20.253`

### Default Gateway
- VLAN 10 → `192.168.10.254`
- VLAN 20 → `192.168.20.254`

### DNS Option
- `192.168.10.10`

> DHCP relay (`ip helper-address`) is NOT required since R1 provides DHCP directly.

---

## 9. DNS Server

- IP: `192.168.10.10`
- Gateway: `192.168.10.254`

### DNS Records
- `google.com` → `8.8.8.8`

---

## 10. NAT (Internet Access)

- PAT (NAT overload) configured on active router
- Inside: VLAN subinterfaces
- Outside: link to Internet router
- Enables internal hosts to access external network

---

## 11. ACL (Security Policy)

### VLAN 20 Restrictions
- ❌ Cannot ping VLAN 10
- ❌ Cannot SSH to routers

### VLAN 10 Permissions
- ✅ Can ping VLAN 20
- ✅ Can SSH into R1 and R2

---

## 12. Remote Access

- SSH enabled on R1 and R2
- Telnet disabled
- Access restricted to VLAN 10 only

---

## 13. Internet Simulation

Internet router acts as ISP simulation:

- `10.0.0.0/30 ↔ R1`
- `11.0.0.0/30 ↔ R2`

---

## 14. Verification & Testing

### DHCP
- Clients in VLAN 10 & VLAN 20 receive IP from R1

### HSRP
- Virtual gateways reachable:
  - `192.168.10.254`
  - `192.168.20.254`

### ACL
- VLAN 20 blocked from VLAN 10
- VLAN 10 fully accessible

### NAT
- Internal devices can reach external network (e.g. `8.8.8.8`)

---

## 15. Key Features Summary

- VLAN segmentation (10/20)
- HSRP redundancy gateway
- DHCP centralized on R1
- STP + EtherChannel
- NAT overload (PAT)
- ACL security policies
- SSH secure management
- Internet simulation via ISP router

---

## 16. Conclusion

This lab demonstrates a realistic enterprise network design with redundancy (HSRP), centralized DHCP service, VLAN segmentation, and security enforcement aligned with CCNA-level concepts.
