# Virtual Private Network for Multi-Branch Organizations

[![Project Type](https://img.shields.io/badge/Project-B.Tech%20Networking%20%2F%20Cybersecurity%20Major%20Project-blue.svg)](#)
[![Simulation Tool](https://img.shields.io/badge/Cisco%20Packet%20Tracer-6.2%20%2F%207.x%20%2F%208.x-orange.svg)](#)
[![Security Standard](https://img.shields.io/badge/Protocol-IPsec%20Site--to--Site%20VPN%20(AES%2BSHA%2BPSK)-green.svg)](#)
[![Hardware](https://img.shields.io/badge/Router-Cisco%201841%20ISR-red.svg)](#)

---

## 📌 Project Overview

In contemporary multi-branch enterprise environments, organizations must maintain secure, reliable, and transparent communications between geographically dispersed headquarters and branch offices across untrusted public telecommunication infrastructures (the Internet / ISP). Transmitting internal business data and proprietary traffic in plaintext over public networks exposes sensitive corporate assets to eavesdropping, packet alteration, session hijacking, and man-in-the-middle (MITM) attacks.

This project implements a secure **Site-to-Site Virtual Private Network (VPN)** using the **IPsec (Internet Protocol Security)** protocol suite with **IKEv1 / ISAKMP**, **AES encryption**, **SHA integrity authentication**, and **Diffie-Hellman Group 2 key exchange** on **Cisco 1841 Integrated Services Routers**. The deployment allows hosts on the Headquarters LAN (`192.168.1.0/24`) and Branch Office LAN (`192.168.2.0/24`) to communicate seamlessly through an untrusted Internet Service Provider (ISP) network without revealing internal RFC 1918 private IP addresses or unencrypted payload data.

> [!NOTE]
> **Packet Tracer Simulation File:**  
> The corresponding Cisco Packet Tracer project is located at:  
> `C:\Users\mrudu\Cisco Packet Tracer 6.2sv\saves\major project (network ).pkt`

---

## 🎯 Objectives

1. **Establish Secure Multi-Branch Connectivity:** Provide transparent Layer 3 private IPv4 communication between HQ and Branch office LANs over a simulated ISP public transit backbone.
2. **Implement Strong Cryptographic Protection:** Deploy standard IPsec security protocols (Confidentiality via AES, Integrity via SHA, Authentication via Pre-Shared Keys, and Anti-Replay via ESP sequence numbers).
3. **Isolate Public Transit from Private Addressing:** Configure realistic routing where the ISP core router (`Router2`) only routes public IP blocks (`12.12.12.0/28` and `23.23.23.0/28`) with zero knowledge of private corporate subnets.
4. **Validate End-to-End Operation:** Perform comprehensive testing including ICMP reachability, application-layer HTTP client-server access, Cisco IOS crypto state verification, and Packet Tracer simulation-mode PDU inspection.

---

## 🏛️ Network Topology & Architecture

### High-Level Architecture Diagram
![Network Topology](diagrams/network-topology.png)

### Cisco Packet Tracer Lab Simulation (`major project (network ).pkt`)
![Cisco Packet Tracer Topology](diagrams/packet-tracer-topology.png)

```
====================================================================================================
  HEADQUARTERS SITE (HQ)                  ISP TRANSIT CORE                    BRANCH SITE
  [192.168.1.0/24]                      [PUBLIC INTERNET]                  [192.168.2.0/24]
====================================================================================================

+-----------------------+                                               +-----------------------+
|        Server0        |                                               |          PC0          |
|    192.168.1.10/24    |                                               |    192.168.2.10/24    |
|   GW: 192.168.1.254   |                                               |   GW: 192.168.2.254   |
+-----------+-----------+                                               +-----------+-----------+
            | Fa0 (Straight-Through)                                                | Fa0 (Straight-Through)
            v                                                                       v
+-----------+-----------+                                               +-----------+-----------+
|    HQ Switch (2960)   |                                               |  Branch Switch (2960) |
|       [Switch0]       |                                               |       [Switch1]       |
+-----------+-----------+                                               +-----------+-----------+
            | Fa0/1 (Straight-Through)                                              | Fa0/1 (Straight-Through)
            v                                                                       v
+-----------+-----------+                                               +-----------+-----------+
|    HQ Router (1841)   |                                               |  Branch Router (1841) |
|      [Router0]        |                                               |       [Router1]       |
| LAN Fa0/1:            |                                               | LAN Fa0/0:            |
|   192.168.1.254/24    |                                               |   192.168.2.254/24    |
| WAN Fa0/0:            |                                               | WAN Fa0/1:            |
|   12.12.12.1/28       |                                               |   23.23.23.2/28       |
+-----------+-----------+                                               +-----------+-----------+
            |                                                                       |
            | WAN Fa0/0 (12.12.12.0/28)                           WAN Fa0/1 (23.23.23.0/28) |
            |                                                                       |
            +-----------------------> +-------------------+ <-----------------------+
                                      | ISP Router (1841) |
                                      |     [Router2]     |
                                      | Fa0/0: 12.12.12.2 |
                                      | Fa0/1: 23.23.23.1 |
                                      +-------------------+

   <=============================================================================>
              SECURE IPSEC ENCRYPTED VPN TUNNEL (192.168.12.0/24)
              Between Router0 (12.12.12.1) <===> Router1 (23.23.23.2)
   <=============================================================================>
```

---

## 🗂️ Project Repository Structure

```
network-vpn-project/
│
├── README.md                           # Master Project Documentation & Quickstart
├── .gitignore                          # Git Ignore Rules for Packet Tracer & OS files
│
├── topology/
│   └── topology-description.md         # Detailed Node, Cabling & Port Specifications
│
├── configs/
│   ├── HQ-Router0.txt                  # Complete Cisco IOS Configuration for HQ Router
│   ├── Branch-Router1.txt              # Complete Cisco IOS Configuration for Branch Router
│   └── ISP-Router2.txt                 # Complete Cisco IOS Configuration for ISP Router
│
├── documentation/
│   ├── network-addressing.md           # Master Subnet Calculations & Address Mapping
│   ├── vpn-configuration.md           # IPsec Protocol Deep-Dive & Cryptographic Suite
│   ├── routing.md                      # Routing Logic, Order of Operations & Dynamic Options
│   └── testing.md                      # Verification Procedures, CLI Outputs & Troubleshooting
│
└── diagrams/
    ├── network-topology.png            # Visual Network Architecture Diagram
    ├── packet-tracer-topology.png      # Cisco Packet Tracer Lab Simulation Screenshot
    └── packet-tracer-canvas.png        # Cropped Packet Tracer Topology Canvas
```

---

## 📊 IP Addressing Table

| Device Name | Device Model | Interface | IP Address | Subnet Mask | Prefix | Network Subnet | Default Gateway | Function / Role |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Router0 (HQ)** | Cisco 1841 | `Fa0/0` | `12.12.12.1` | `255.255.255.240` | `/28` | `12.12.12.0/28` | `12.12.12.2` (ISP) | WAN uplink to ISP Router2 |
| **Router0 (HQ)** | Cisco 1841 | `Fa0/1` | `192.168.1.254` | `255.255.255.0` | `/24` | `192.168.1.0/24` | N/A | LAN Gateway for HQ subnet |
| **Router0 (HQ)** | Cisco 1841 | `Tunnel0`* | `192.168.12.1` | `255.255.255.0` | `/24` | `192.168.12.0/24` | N/A | VPN Tunnel Virtual Endpoint |
| **Router1 (Branch)** | Cisco 1841 | `Fa0/0` | `192.168.2.254` | `255.255.255.0` | `/24` | `192.168.2.0/24` | N/A | LAN Gateway for Branch subnet |
| **Router1 (Branch)** | Cisco 1841 | `Fa0/1` | `23.23.23.2` | `255.255.255.240` | `/28` | `23.23.23.0/28` | `23.23.23.1` (ISP) | WAN uplink to ISP Router2 |
| **Router1 (Branch)** | Cisco 1841 | `Tunnel0`* | `192.168.12.2` | `255.255.255.0` | `/24` | `192.168.12.0/24` | N/A | VPN Tunnel Virtual Endpoint |
| **Router2 (ISP)** | Cisco 1841 | `Fa0/0` | `12.12.12.2` | `255.255.255.240` | `/28` | `12.12.12.0/28` | N/A | Public WAN connection to HQ |
| **Router2 (ISP)** | Cisco 1841 | `Fa0/1` | `23.23.23.1` | `255.255.255.240` | `/28` | `23.23.23.0/28` | N/A | Public WAN connection to Branch |
| **Server0** | Server-PT | `Fa0` | `192.168.1.10` | `255.255.255.0` | `/24` | `192.168.1.0/24` | `192.168.1.254` | Corporate HQ Web & File Server |
| **PC0** | PC-PT | `Fa0` | `192.168.2.10` | `255.255.255.0` | `/24` | `192.168.2.0/24` | `192.168.2.254` | Remote Branch Workstation |

*\*Note on Tunnel Interfaces:* `Tunnel0` interface details apply when implementing route-based GRE/IPsec encapsulation. In policy-based IPsec (Crypto Map on physical interfaces), the tunnel subnet `192.168.12.0/24` represents the logical overlay between the endpoints.

---

## 🔄 End-to-End Traffic Flow & Encapsulation Walkthrough

When **Server0** (`192.168.1.10`) communicates with **Branch PC0** (`192.168.2.10`), traffic traverses the network following these exact stages:

```
HQ Server (192.168.1.10)
    │
    ▼ [Layer 2 Frame: Ethernet II | Src MAC: Server0, Dst MAC: Switch0 / Router0]
HQ Switch (Switch0)
    │
    ▼ [Plaintext IPv4 Packet: Src: 192.168.1.10, Dst: 192.168.2.10]
HQ Router (Router0 - Fa0/1)
    │
    ├─► 1. Routing Table Lookup: Matches Default Route (0.0.0.0 via Fa0/0)
    ├─► 2. Crypto Map Inspection on Fa0/0: Matches ACL 101 (Interesting Traffic)
    └─► 3. IPsec Crypto Engine: Encrypts packet with AES-128, hashes with SHA-1 HMAC,
           and prepends Outer Public IP Header (Src: 12.12.12.1, Dst: 23.23.23.2, Proto: 50 ESP)
    │
    ▼ [Encrypted IPsec ESP Packet over Public Transit]
ISP Router (Router2)
    │
    ├─► Relays ESP packet from Fa0/0 (12.12.12.2) to Fa0/1 (23.23.23.1)
    └─► Has NO access to internal payload or private IP headers
    │
    ▼ [Encrypted Packet Ingress on Branch Fa0/1]
Branch Router (Router1)
    │
    ├─► 1. Detects ESP packet (Protocol 50) bound for local WAN IP 23.23.23.2
    ├─► 2. IPsec Engine matches SPI, verifies SHA-1 integrity, decrypts AES payload
    └─► 3. Restores original inner packet (Src: 192.168.1.10, Dst: 192.168.2.10)
    │
    ▼ [Plaintext IPv4 Packet forwarded via Fa0/0 to Switch1]
Branch Switch (Switch1)
    │
    ▼ [Frame delivered to destination workstation port]
Branch PC (PC0 - 192.168.2.10)
```

---

## ⚙️ Step-by-Step Configuration Guide

### 1. Configure ISP-Router2 (Public Transit Provider)
Apply the script from [configs/ISP-Router2.txt](file:///c:/Users/mrudu/VPN%20Tunnel%20%28major%20project%29/configs/ISP-Router2.txt):
```cisco
enable
configure terminal
hostname ISP-Router2

interface FastEthernet0/0
 description WAN - Link to HQ-Router0 Fa0/0
 ip address 12.12.12.2 255.255.255.240
 no shutdown
 exit

interface FastEthernet0/1
 description WAN - Link to Branch-Router1 Fa0/1
 ip address 23.23.23.1 255.255.255.240
 no shutdown
 exit

end
write memory
```

---

### 2. Configure HQ-Router0 (Headquarters Gateway)
Apply the script from [configs/HQ-Router0.txt](file:///c:/Users/mrudu/VPN%20Tunnel%20%28major%20project%29/configs/HQ-Router0.txt):
```cisco
enable
configure terminal
hostname HQ-Router0

! Interfaces
interface FastEthernet0/0
 description WAN - Link to ISP-Router2 Fa0/0
 ip address 12.12.12.1 255.255.255.240
 no shutdown
 exit

interface FastEthernet0/1
 description LAN - Gateway for HQ Subnet (192.168.1.0/24)
 ip address 192.168.1.254 255.255.255.0
 no shutdown
 exit

! Default Route to ISP
ip route 0.0.0.0 0.0.0.0 12.12.12.2

! IPsec Phase 1 (ISAKMP Policy)
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
 exit

! Pre-Shared Key bound to Branch WAN IP
crypto isakmp key CiscoVPN2026! address 23.23.23.2

! IPsec Phase 2 (Transform Set)
crypto ipsec transform-set TS-VPN esp-aes esp-sha-hmac
 mode tunnel
 exit

! Interesting Traffic ACL
access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255

! Crypto Map
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 23.23.23.2
 set transform-set TS-VPN
 match address 101
 exit

! Apply Crypto Map to Outgoing WAN Interface
interface FastEthernet0/0
 crypto map VPN-MAP
 exit

end
write memory
```

---

### 3. Configure Branch-Router1 (Branch Office Gateway)
Apply the script from [configs/Branch-Router1.txt](file:///c:/Users/mrudu/VPN%20Tunnel%20%28major%20project%29/configs/Branch-Router1.txt):
```cisco
enable
configure terminal
hostname Branch-Router1

! Interfaces
interface FastEthernet0/0
 description LAN - Gateway for Branch Subnet (192.168.2.0/24)
 ip address 192.168.2.254 255.255.255.0
 no shutdown
 exit

interface FastEthernet0/1
 description WAN - Link to ISP-Router2 Fa0/1
 ip address 23.23.23.2 255.255.255.240
 no shutdown
 exit

! Default Route to ISP
ip route 0.0.0.0 0.0.0.0 23.23.23.1

! IPsec Phase 1 (ISAKMP Policy)
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
 exit

! Pre-Shared Key bound to HQ WAN IP
crypto isakmp key CiscoVPN2026! address 12.12.12.1

! IPsec Phase 2 (Transform Set)
crypto ipsec transform-set TS-VPN esp-aes esp-sha-hmac
 mode tunnel
 exit

! Interesting Traffic ACL
access-list 101 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255

! Crypto Map
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 12.12.12.1
 set transform-set TS-VPN
 match address 101
 exit

! Apply Crypto Map to Outgoing WAN Interface
interface FastEthernet0/1
 crypto map VPN-MAP
 exit

end
write memory
```

---

## 🧪 Testing Procedure & Verification

For comprehensive testing steps, see [documentation/testing.md](file:///c:/Users/mrudu/VPN%20Tunnel%20%28major%20project%29/documentation/testing.md).

### 1. End-to-End Ping Test (Triggering the VPN)
Open Command Prompt on **PC0** (`192.168.2.10`) and ping **Server0** (`192.168.1.10`):
```cmd
PC> ping 192.168.1.10
```
* **First attempt:** 1 packet drop due to IKE Phase 1 / Phase 2 negotiation delay, followed by 3 successful replies.
* **Second attempt:** 4/4 replies (100% success rate).

### 2. IOS Verification Commands

```cisco
! Check interface operational states:
HQ-Router0# show ip interface brief

! Check routing table & default gateway:
HQ-Router0# show ip route

! Check Phase 1 ISAKMP Security Associations (MUST show QM_IDLE):
HQ-Router0# show crypto isakmp sa
dst             src             state          conn-id slot status
23.23.23.2      12.12.12.1      QM_IDLE              1    0 ACTIVE

! Check Phase 2 IPsec Security Associations & packet counters:
HQ-Router0# show crypto ipsec sa
    #pkts encaps: 14, #pkts encrypt: 14, #pkts digest: 14
    #pkts decaps: 14, #pkts decrypt: 14, #pkts verify: 14

! Check crypto map binding:
HQ-Router0# show crypto map
```

---

## 🛠️ Quick Troubleshooting Guide

| Issue | Cause | Fix |
| :--- | :--- | :--- |
| **`QM_IDLE` not reached in `show crypto isakmp sa`** | PSK mismatch or peer IP misconfigured | Check `crypto isakmp key` and peer IP on both routers |
| **Ping fails completely with Request Timed Out** | Missing default route or gateway IP | Check `ip route 0.0.0.0 0.0.0.0 <ISP-IP>` on both routers |
| **Phase 1 active, but Phase 2 packets not encrypting** | ACL 101 mismatch | Ensure HQ ACL is `permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255` and Branch ACL is the exact inverse |
| **Interface shows administratively down** | Interface shut down | Run `no shutdown` under the interface |

---

## 💻 Technologies & Standards Used

* **Cisco IOS Software Version:** 12.4 / 15.x
* **Router Platform:** Cisco 1841 Modular Integrated Services Router
* **Switch Platform:** Cisco Catalyst 2960-24TT
* **Network Protocols:** IPv4, ICMP, ARP, Ethernet 802.3
* **Security & VPN Standards:**
  * IPsec (RFC 4301 / RFC 2401)
  * IKEv1 / ISAKMP (RFC 2409 / RFC 2408)
  * AES (Advanced Encryption Standard - 128-bit)
  * SHA-1 HMAC (Secure Hash Algorithm - RFC 2104)
  * Diffie-Hellman Group 2 (RFC 2412)
  * ESP (Encapsulating Security Payload - RFC 4303)

---

## 👥 Authors & Academic Context

* **Project:** B.Tech Capstone / Major Networking & Cybersecurity Project
* **Theme:** Secure Multi-Branch Interconnectivity and Cryptographic Network Design
* **Cisco Packet Tracer File:** `C:\Users\mrudu\Cisco Packet Tracer 6.2sv\saves\major project (network ).pkt`
