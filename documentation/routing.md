# Routing Architecture & Order of Operations

**Project Title:** Virtual Private Network for Multi-Branch Organizations  
**Document:** Routing Logic, Packet Flow & IOS Order of Operations  
**Date:** 2026-10-06  

---

## 1. Routing Overview

In an enterprise VPN deployment, routing determines how packets navigate between internal local area networks, edge gateways, and public carrier infrastructures.

In this topology:
1. **HQ-Router0** utilizes a **Static Default Route** (`0.0.0.0/0`) pointing to ISP gateway `12.12.12.2`.
2. **Branch-Router1** utilizes a **Static Default Route** (`0.0.0.0/0`) pointing to ISP gateway `23.23.23.1`.
3. **ISP-Router2** maintains only directly connected routes for the public subnets `12.12.12.0/28` and `23.23.23.0/28`. It holds **NO routes** for private corporate subnets (`192.168.1.0/24` or `192.168.2.0/24`).

---

## 2. Cisco IOS Crypto & Routing Order of Operations

A frequent point of confusion in IPsec networking is how routers decide to encrypt packets versus routing them normally.

```
+-----------------------------------------------------------------------------------+
|                        CISCO IOS PACKET FORWARDING PIPELINE                       |
+-----------------------------------------------------------------------------------+
                                         │
  1. Ingress Packet Received on LAN Interface (e.g., Fa0/0 on Branch-Router1)
                                         │
                                         ▼
  2. Routing Table Lookup (FIB / CEF)
     - Target IP: 192.168.1.10
     - Lookup matches Default Route: 0.0.0.0 0.0.0.0 via Fa0/1 (Next-hop: 23.23.23.1)
     - Egress interface determined: FastEthernet0/1
                                         │
                                         ▼
  3. Egress Interface Inspection for Crypto Map
     - Router detects 'crypto map VPN-MAP' applied to FastEthernet0/1
                                         │
                                         ▼
  4. Access-List (ACL 101) Evaluation
     - Does packet (Src: 192.168.2.10, Dst: 192.168.1.10) match ACL 101?
     - YES -> Packet classified as INTERESTING TRAFFIC
                                         │
                                         ▼
  5. IPsec Encapsulation Engine
     - Checks if active IPsec SA exists. If NO, trigger IKE Phase 1 & Phase 2.
     - Once SAs are active: Original packet is encrypted using AES-128.
     - New outer IP header prepended (Src: 23.23.23.2, Dst: 12.12.12.1, Proto: 50 ESP).
                                         │
                                         ▼
  6. Egress Transmission to ISP
     - Encrypted ESP packet transmitted over physical wire to ISP-Router2 (Fa0/1).
```

---

## 3. ISP Core Routing Isolation

```
+------------------+                    +------------------+                    +------------------+
|    HQ-Router0    |                    |    ISP-Router2   |                    |  Branch-Router1  |
|                  |                    |                  |                    |                  |
| Public IP:       |                    | Fa0/0: 12.12.12.2|                    | Public IP:       |
| 12.12.12.1/28    |<==================>| Fa0/1: 23.23.23.1|<==================>| 23.23.23.2/28    |
+------------------+   12.12.12.0/28    +------------------+   23.23.23.0/28    +------------------+
```

### ISP Routing Table (`show ip route` on ISP-Router2):
```
Gateway of last resort is not set

     12.0.0.0/28 is subnetted, 1 subnets
C       12.12.12.0 is directly connected, FastEthernet0/0
     23.0.0.0/28 is subnetted, 1 subnets
C       23.23.23.0 is directly connected, FastEthernet0/1
```

### Why this design is realistic:
* **RFC 1918 Private IP Protection:** The public internet drops private IP packets. In this lab, if plaintext packets with source `192.168.2.10` reach the ISP without encryption, the ISP would discard them because it has no route to `192.168.1.0/24`.
* **Encapsulation Bypass:** Because the outer IP header uses public IPs `12.12.12.1` and `23.23.23.2`, the ISP effortlessly forwards the packets using its directly connected interface table.

---

## 4. Comparison: Policy-Based IPsec vs. Route-Based GRE/IPsec

| Feature | Policy-Based IPsec (Crypto Map) | Route-Based (GRE over IPsec / VTI) |
| :--- | :--- | :--- |
| **Interface Type** | Applied directly to physical WAN interface | Uses virtual `Tunnel0` interface |
| **Traffic Trigger** | Extended Access List (ACL) matching | Route table / Next-Hop pointing to Tunnel |
| **Multicast / Routing Protocols** | Does **NOT** support multicast (No OSPF/EIGRP) | **Supports multicast** (OSPF, EIGRP, BGP across tunnel) |
| **Configuration Complexity** | Simple, standard for site-to-site | Requires Tunnel config + IPsec profile |
| **Packet Tracer 6.2sv Stability**| **100% Reliable & Fully Supported** | Supported with specific IOS images |
| **Project Usage** | **Primary implementation for this project** | Included as reference in configs & docs |

---

## 5. Dynamic Routing Extension (OSPF over GRE Reference)

If dynamic routing across the tunnel is required in an extended scenario:
```cisco
! On HQ-Router0:
interface Tunnel0
 ip address 192.168.12.1 255.255.255.0
 tunnel source FastEthernet0/0
 tunnel destination 23.23.23.2
 exit

router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 exit
```

```cisco
! On Branch-Router1:
interface Tunnel0
 ip address 192.168.12.2 255.255.255.0
 tunnel source FastEthernet0/1
 tunnel destination 12.12.12.1
 exit

router ospf 1
 network 192.168.2.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.255 area 0
 exit
```
