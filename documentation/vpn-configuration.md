# IPsec VPN Configuration & Architecture Guide

**Project Title:** Virtual Private Network for Multi-Branch Organizations  
**Document:** IPsec Protocol Deep-Dive, Cryptographic Suite & IOS Implementation  
**Date:** 2026-10-06  

---

## 1. Executive Summary & Security Objectives

IPsec (Internet Protocol Security) is a standards-based suite of open protocols defined by the IETF (RFC 4301) to provide cryptographic security services at the Network Layer (Layer 3) of the OSI model. In this project, an IPsec Site-to-Site VPN is established between **HQ-Router0** and **Branch-Router1** across the untrusted transit network of **ISP-Router2**.

### Core Security Services Provided:
1. **Confidentiality:** Encryption of private user payloads using AES (Advanced Encryption Standard) to prevent eavesdropping by ISP or third-party attackers.
2. **Data Integrity:** Hashing via SHA-1 HMAC (Hashed Message Authentication Code) ensures data cannot be modified in transit without detection.
3. **Peer Authentication:** Identity verification using Pre-Shared Keys (PSK) before establishing any encrypted session.
4. **Anti-Replay Protection:** Sequence numbering in ESP prevents malicious actors from intercepting and retransmitting recorded packets.

---

## 2. IPsec VPN Two-Phase Architecture

```
+-----------------------------------------------------------------------------------+
|                           PHASE 1: IKE / ISAKMP SA                                |
|  - Authenticate peers (Pre-Shared Key)                                            |
|  - Agree on cryptographic algorithms (AES, SHA, DH-2)                             |
|  - Establish secure bidirectional management tunnel (ISAKMP SA / UDP 500)         |
+-----------------------------------------------------------------------------------+
                                         │
                                         ▼
+-----------------------------------------------------------------------------------+
|                              PHASE 2: IPSEC SA                                    |
|  - Negotiate data security parameters (Transform Set: ESP-AES + ESP-SHA-HMAC)     |
|  - Agree on Interesting Traffic (Proxy ACLs 101)                                  |
|  - Establish unidirectional IPsec SAs (SPIs) for encrypted user payload           |
+-----------------------------------------------------------------------------------+
```

---

## 3. Phase 1: IKE / ISAKMP Negotiation Deep-Dive

Internet Key Exchange (IKEv1) / Internet Security Association and Key Management Protocol (ISAKMP) operates on **UDP port 500**. Its primary goal is to authenticate the two peer routers and securely establish a shared master key to protect subsequent Phase 2 negotiations.

### 3.1 The HAGLE Negotiation Framework
For Phase 1 to succeed, both routers must agree on five critical parameters:

| Parameter | Parameter Name | Value Selected in Project | IOS Configuration Keyword | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **H** | **Hash Algorithm** | SHA-1 (160-bit) | `hash sha` | Verifies data integrity and message origin. |
| **A** | **Authentication** | Pre-Shared Key (PSK) | `authentication pre-share` | Authenticates peer router identities. |
| **G** | **Diffie-Hellman Group** | Group 2 (1024-bit) | `group 2` | Generates shared secret keys over insecure public channel. |
| **L** | **Lifetime** | 86,400 seconds (24h) | `lifetime 86400` | Expiration time before re-keying Phase 1 SA. |
| **E** | **Encryption** | AES-128 | `encr aes` | Symmetrical cipher for protecting Phase 1 messages. |

### 3.2 Phase 1 Configuration Breakdown

#### HQ-Router0:
```cisco
! Enter Phase 1 ISAKMP policy configuration
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
 exit

! Pre-Shared Key bound to Branch WAN IP (23.23.23.2)
crypto isakmp key CiscoVPN2026! address 23.23.23.2
```

#### Branch-Router1:
```cisco
! Enter matching Phase 1 ISAKMP policy configuration
crypto isakmp policy 10
 encr aes
 hash sha
 authentication pre-share
 group 2
 lifetime 86400
 exit

! Pre-Shared Key bound to HQ WAN IP (12.12.12.1)
crypto isakmp key CiscoVPN2026! address 12.12.12.1
```

---

## 4. Phase 2: IPsec Security Associations & Transform Sets

Once Phase 1 is in the `QM_IDLE` (active) state, IKE initiates Phase 2 (Quick Mode) to negotiate the actual IPsec Security Associations (SAs) that protect user payload traffic.

### 4.1 Encapsulating Security Payload (ESP)
We utilize **ESP (IP Protocol 50)** rather than Authentication Header (AH / IP Protocol 51) because AH provides integrity but **no encryption**. ESP provides both robust encryption and cryptographic integrity.

### 4.2 Tunnel Mode vs. Transport Mode
* **Tunnel Mode (Default & Configured):** The entire original IP packet (Header + Payload) is encrypted, and a completely new outer IP header is prepended with public routable addresses (`12.12.12.1` -> `23.23.23.2`). This is required for Site-to-Site VPNs where private host addresses (`192.168.1.10`) cannot be routed across the public internet.
* **Transport Mode:** Only the payload is encrypted; original IP headers remain exposed. Used in end-to-end host-to-host scenarios.

### 4.3 Phase 2 Transform Set Configuration
```cisco
! Configured on both HQ-Router0 and Branch-Router1:
crypto ipsec transform-set TS-VPN esp-aes esp-sha-hmac
 mode tunnel
 exit
```

---

## 5. Identifying Interesting Traffic with ACLs

An extended Access Control List (ACL) defines the "interesting traffic" that triggers the router to initiate the VPN tunnel and encrypt matching packets.

### Symmetric ACL Requirement:
The ACL on the local router must be an exact cryptographic mirror of the ACL on the remote router.

* **HQ-Router0 ACL:**
  ```cisco
  access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
  ```
  *(Matches packets originating from HQ LAN destined for Branch LAN)*

* **Branch-Router1 ACL:**
  ```cisco
  access-list 101 permit ip 192.168.2.0 0.0.0.255 192.168.1.0 0.0.0.255
  ```
  *(Matches packets originating from Branch LAN destined for HQ LAN)*

---

## 6. Crypto Map: Gluing All Components Together

A Crypto Map is the central binding structure in Cisco IOS that associates:
1. The remote peer's public IP address (`set peer`)
2. The Phase 2 Transform Set (`set transform-set`)
3. The Interesting Traffic ACL (`match address`)

### 6.1 Crypto Map Configuration
```cisco
! HQ-Router0:
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 23.23.23.2
 set transform-set TS-VPN
 match address 101
 exit

interface FastEthernet0/0
 crypto map VPN-MAP
 exit
```

```cisco
! Branch-Router1:
crypto map VPN-MAP 10 ipsec-isakmp
 set peer 12.12.12.1
 set transform-set TS-VPN
 match address 101
 exit

interface FastEthernet0/1
 crypto map VPN-MAP
 exit
```

---

## 7. Packet Encapsulation Structure in Transit

When PC0 (`192.168.2.10`) sends an ICMP Echo Request to Server0 (`192.168.1.10`), the packet structure transforms across the network:

### Inside LAN (Plaintext):
```
[Ethernet Header] [Source: 192.168.2.10 | Dest: 192.168.1.10] [ICMP Echo]
```

### In Public Transit Across ISP (Encapsulated IPsec ESP Tunnel Mode):
```
+-----------------------------------------------------------------------------------------------+
| New Public IP Header    | ESP Header | Encrypted Inner IP Packet       | ESP Trailer | ESP ICV |
| Src: 23.23.23.2         | SPI: 0x... | Src: 192.168.2.10               | Pad, Length | HMAC    |
| Dest: 12.12.12.1        | Seq: 001   | Dest: 192.168.1.10 [ICMP Data]  | Next: IP(4) | SHA-1   |
| Proto: 50 (ESP)         |            | (AES-128 Ciphertext)            |             |         |
+-----------------------------------------------------------------------------------------------+
```

The ISP Router2 only inspects the outer IP header (`23.23.23.2` -> `12.12.12.1`). It has no visibility into the inner private IP addresses or payload data.

---

## 8. Verification & Debugging Commands

| Verification Command | Purpose | Expected Output / Success Indicator |
| :--- | :--- | :--- |
| `show crypto isakmp policy` | Verify configured Phase 1 parameters | Displays AES, SHA, Pre-share, Group 2, 86400s |
| `show crypto isakmp sa` | Check Phase 1 tunnel status | State should be `QM_IDLE` and status `ACTIVE` |
| `show crypto ipsec sa` | Check Phase 2 tunnel & encryption stats | `#pkts encaps > 0`, `#pkts decaps > 0` |
| `show crypto map` | Verify interface binding & peer IP | Interfaces: FastEthernet0/0 (or 0/1) active |
| `debug crypto isakmp` | Real-time debug of Phase 1 exchange | Shows 6 main-mode packet exchanges |
| `debug crypto ipsec` | Real-time debug of Phase 2 exchange | Shows Quick Mode SA creation |
