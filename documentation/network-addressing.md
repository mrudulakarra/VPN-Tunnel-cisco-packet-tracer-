# Network Addressing Plan & Subnet Design

**Project Title:** Virtual Private Network for Multi-Branch Organizations  
**Document:** Master Addressing Table & Subnet Calculations  
**Date:** 2026-10-06  

---

## 1. Master IP Addressing Table

| Device Name | Device Model | Interface | IP Address | Subnet Mask | Prefix | Wildcard Mask | Network Subnet | Default Gateway | Description / Role |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Router0 (HQ)** | Cisco 1841 | `Fa0/0` | `12.12.12.1` | `255.255.255.240` | `/28` | `0.0.0.15` | `12.12.12.0/28` | `12.12.12.2` (ISP) | WAN connection to ISP Router2 |
| **Router0 (HQ)** | Cisco 1841 | `Fa0/1` | `192.168.1.254` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.1.0/24` | N/A | LAN Gateway for HQ subnet |
| **Router0 (HQ)** | Cisco 1841 | `Tunnel0`* | `192.168.12.1` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.12.0/24` | N/A | Virtual VPN Tunnel Endpoint (HQ) |
| **Router1 (Branch)** | Cisco 1841 | `Fa0/0` | `192.168.2.254` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.2.0/24` | N/A | LAN Gateway for Branch subnet |
| **Router1 (Branch)** | Cisco 1841 | `Fa0/1` | `23.23.23.2` | `255.255.255.240` | `/28` | `0.0.15` | `23.23.23.0/28` | `23.23.23.1` (ISP) | WAN connection to ISP Router2 |
| **Router1 (Branch)** | Cisco 1841 | `Tunnel0`* | `192.168.12.2` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.12.0/24` | N/A | Virtual VPN Tunnel Endpoint (Branch) |
| **Router2 (ISP)** | Cisco 1841 | `Fa0/0` | `12.12.12.2` | `255.255.255.240` | `/28` | `0.0.0.15` | `12.12.12.0/28` | N/A | Public WAN port towards HQ Router0 |
| **Router2 (ISP)** | Cisco 1841 | `Fa0/1` | `23.23.23.1` | `255.255.255.240` | `/28` | `0.0.0.15` | `23.23.23.0/28` | N/A | Public WAN port towards Branch Router1 |
| **Server0** | Generic Server | `Fa0` | `192.168.1.10` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.1.0/24` | `192.168.1.254` | Corporate HQ File/Web Server |
| **PC0** | Generic PC | `Fa0` | `192.168.2.10` | `255.255.255.0` | `/24` | `0.0.0.255` | `192.168.2.0/24` | `192.168.2.254` | Branch Office Client Workstation |
| **Switch0** | Catalyst 2960 | `VLAN 1` | Unmanaged / DHCP optional | `255.255.255.0` | `/24` | N/A | `192.168.1.0/24` | `192.168.1.254` | HQ Layer-2 Switch |
| **Switch1** | Catalyst 2960 | `VLAN 1` | Unmanaged / DHCP optional | `255.255.255.0` | `/24` | N/A | `192.168.2.0/24` | `192.168.2.254` | Branch Layer-2 Switch |

*\*Note:* `Tunnel0` interface is used if implementing Route-Based / GRE-over-IPsec VPN. For standard Policy-Based IPsec (Crypto Map on FastEthernet), the virtual tunnel subnet represents the logical overlay between the endpoints.

---

## 2. Subnet Breakdown & Mathematical Analysis

### 2.1 HQ LAN Subnet (`192.168.1.0/24`)
* **Network ID:** `192.168.1.0`
* **Subnet Mask:** `255.255.255.0` (`11111111.11111111.11111111.00000000`)
* **Wildcard Mask:** `0.0.0.255` (used in Cisco ACLs)
* **Total Host Addresses:** $2^8 = 256$
* **Usable Host Addresses:** $256 - 2 = 254$
* **Usable Host Range:** `192.168.1.1` – `192.168.1.254`
* **Broadcast Address:** `192.168.1.255`
* **Standard RFC 1918 Scope:** Private Class C Subnet

---

### 2.2 Branch LAN Subnet (`192.168.2.0/24`)
* **Network ID:** `192.168.2.0`
* **Subnet Mask:** `255.255.255.0`
* **Wildcard Mask:** `0.0.0.255`
* **Total Host Addresses:** $2^8 = 256$
* **Usable Host Addresses:** $254$
* **Usable Host Range:** `192.168.2.1` – `192.168.2.254`
* **Broadcast Address:** `192.168.2.255`
* **Standard RFC 1918 Scope:** Private Class C Subnet

---

### 2.3 HQ-to-ISP WAN Subnet (`12.12.12.0/28`)
* **Network ID:** `12.12.12.0`
* **Subnet Mask:** `255.255.255.240` (`11111111.11111111.11111111.11110000`)
* **Prefix Length:** `/28` (28 network bits, 4 host bits)
* **Block Size (Increment):** $256 - 240 = 16$
* **Total IP Addresses:** $2^4 = 16$
* **Usable Host Addresses:** $16 - 2 = 14$
* **Usable Host Range:** `12.12.12.1` – `12.12.12.14`
* **Broadcast Address:** `12.12.12.15`
* **Assigned Endpoints:**
  * HQ Router0 Fa0/0: `12.12.12.1`
  * ISP Router2 Fa0/0: `12.12.12.2`

---

### 2.4 Branch-to-ISP WAN Subnet (`23.23.23.0/28`)
* **Network ID:** `23.23.23.0`
* **Subnet Mask:** `255.255.255.240`
* **Prefix Length:** `/28`
* **Block Size (Increment):** $16$
* **Total IP Addresses:** $16$
* **Usable Host Addresses:** $14$
* **Usable Host Range:** `23.23.23.1` – `23.23.23.14`
* **Broadcast Address:** `23.23.23.15`
* **Assigned Endpoints:**
  * ISP Router2 Fa0/1: `23.23.23.1`
  * Branch Router1 Fa0/1: `23.23.23.2`

---

### 2.5 Virtual VPN Tunnel Subnet (`192.168.12.0/24`)
* **Network ID:** `192.168.12.0`
* **Subnet Mask:** `255.255.255.0`
* **Assigned Endpoints:**
  * HQ Router Tunnel0: `192.168.12.1`
  * Branch Router Tunnel0: `192.168.12.2`

---

## 3. End-Host Static Configuration Checklist

### 3.1 HQ Server0 Configuration (Cisco Packet Tracer)
1. Click **Server0** in the workspace.
2. Navigate to **Desktop** -> **IP Configuration**.
3. Set the following parameters:
   * **Configuration Mode:** `Static`
   * **IP Address:** `192.168.1.10`
   * **Subnet Mask:** `255.255.255.0`
   * **Default Gateway:** `192.168.1.254`
   * **DNS Server:** `192.168.1.10` (or leave blank)
4. Verify HTTP and FTP services are turned **ON** in the **Services** tab to allow application-layer testing across the VPN.

### 3.2 Branch PC0 Configuration (Cisco Packet Tracer)
1. Click **PC0** in the workspace.
2. Navigate to **Desktop** -> **IP Configuration**.
3. Set the following parameters:
   * **Configuration Mode:** `Static`
   * **IP Address:** `192.168.2.10`
   * **Subnet Mask:** `255.255.255.0`
   * **Default Gateway:** `192.168.2.254`
   * **DNS Server:** `192.168.1.10`
5. Open **Command Prompt** to run connectivity tests.

---

## 4. Verification Check

To confirm IP settings inside Packet Tracer CLI:
```cisco
HQ-Router0# show ip interface brief
Branch-Router1# show ip interface brief
ISP-Router2# show ip interface brief
```
Ensure all required interfaces display status `up` and protocol `up`.
