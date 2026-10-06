# Testing, Verification & Troubleshooting Guide

**Project Title:** Virtual Private Network for Multi-Branch Organizations  
**Document:** End-to-End Verification Procedures, CLI Outputs & Troubleshooting  
**Date:** 2026-10-06  

---

## 1. Step-by-Step Testing Methodology

```
+-----------------------------------------------------------------------------------+
|                           TEST EXECUTION WORKFLOW                                 |
+-----------------------------------------------------------------------------------+
  [Step 1] Physical & Layer 3 Local Verification (Ping Default Gateways)
     │
     ▼
  [Step 2] Public WAN Connectivity (HQ <-> ISP <-> Branch WAN Pings)
     │
     ▼
  [Step 3] VPN Tunnel Trigger (Ping from Branch PC0 -> HQ Server0)
     │
     ▼
  [Step 4] Cisco IOS Verification Commands (`show crypto ...`)
     │
     ▼
  [Step 5] Packet Tracer Simulation Mode (Inspect ISAKMP & ESP PDUs)
     │
     ▼
  [Step 6] Application Layer Testing (HTTP / FTP from PC0 to Server0)
```

---

## 2. Step 1 & 2: Local & Public WAN Baseline Verification

Before testing the VPN tunnel, verify that all physical links are up and public routing is functional.

### Test 1.1: Local Gateway Reachability
* From **PC0** (`192.168.2.10`), ping default gateway:
  ```cmd
  PC> ping 192.168.2.254
  Reply from 192.168.2.254: bytes=32 time=1ms TTL=255 (SUCCESS)
  ```
* From **Server0** (`192.168.1.10`), ping default gateway:
  ```cmd
  Server> ping 192.168.1.254
  Reply from 192.168.1.254: bytes=32 time=1ms TTL=255 (SUCCESS)
  ```

### Test 1.2: Public WAN Peer Reachability
* From **HQ-Router0**, ping Branch WAN interface across ISP:
  ```cisco
  HQ-Router0# ping 23.23.23.2
  Type escape sequence to abort.
  Sending 5, 100-byte ICMP Echos to 23.23.23.2, timeout is 2 seconds:
  !!!!!
  Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms
  ```

---

## 3. Step 3: Triggering the VPN Tunnel (End-to-End Ping)

Open the **Command Prompt** on **PC0** and ping **Server0**:

```cmd
PC> ping 192.168.1.10

Pinging 192.168.1.10 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.10: bytes=32 time=12ms TTL=126
Reply from 192.168.1.10: bytes=32 time=8ms TTL=126
Reply from 192.168.1.10: bytes=32 time=7ms TTL=126

Ping statistics for 192.168.1.10:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 7ms, Maximum = 12ms, Average = 9ms
```

> [!NOTE]
> **Why does the first ping packet time out?**  
> When the first ICMP packet hits the router, the router must:
> 1. Resolve ARP on the local segment.
> 2. Initiate IKE Phase 1 Main Mode (6 packets exchange).
> 3. Initiate IKE Phase 2 Quick Mode (3 packets exchange).
> 4. Create Security Associations (SAs) in the crypto engine.  
> This brief delay causes the first ICMP request to time out. Subsequent pings succeed immediately with a 100% success rate.

Running the ping a second time will yield:
```cmd
PC> ping 192.168.1.10

Pinging 192.168.1.10 with 32 bytes of data:

Reply from 192.168.1.10: bytes=32 time=7ms TTL=126
Reply from 192.168.1.10: bytes=32 time=6ms TTL=126
Reply from 192.168.1.10: bytes=32 time=7ms TTL=126
Reply from 192.168.1.10: bytes=32 time=6ms TTL=126

Ping statistics for 192.168.1.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

---

## 4. Step 4: Cisco IOS Verification Commands with Sample Output

Execute these commands in privileged EXEC mode (`Router#`) on **HQ-Router0** or **Branch-Router1**:

### 4.1 Check Interface Status (`show ip interface brief`)
```cisco
HQ-Router0# show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            12.12.12.1      YES manual up                    up      
FastEthernet0/1            192.168.1.254   YES manual up                    up      
```

### 4.2 Check Routing Table (`show ip route`)
```cisco
HQ-Router0# show ip route
Codes: C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 

Gateway of last resort is 12.12.12.2 to network 0.0.0.0

     12.0.0.0/28 is subnetted, 1 subnets
C       12.12.12.0 is directly connected, FastEthernet0/0
C    192.168.1.0/24 is directly connected, FastEthernet0/1
S*   0.0.0.0/0 [1/0] via 12.12.12.2
```

### 4.3 Verify IKE Phase 1 Policy (`show crypto isakmp policy`)
```cisco
HQ-Router0# show crypto isakmp policy

Global IKE policy
Protection suite of priority 10
        encryption algorithm:   AES - Advanced Encryption Standard (128 bit keys).
        hash algorithm:         Secure Hash Standard
        authentication method:  Pre-Shared Key
        Diffie-Hellman group:   #2 (1024 bit)
        lifetime:               86400 secs, no volume limit
```

### 4.4 Verify IKE Phase 1 Security Association (`show crypto isakmp sa`)
```cisco
HQ-Router0# show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id slot status
23.23.23.2      12.12.12.1      QM_IDLE              1    0 ACTIVE
```
> [!IMPORTANT]
> The state **`QM_IDLE`** (Quick Mode Idle) with status **`ACTIVE`** indicates that Phase 1 negotiation was completely successful and the management tunnel is established.

### 4.5 Verify IPsec Phase 2 Security Associations (`show crypto ipsec sa`)
```cisco
HQ-Router0# show crypto ipsec sa

interface: FastEthernet0/0
    Crypto map tag: VPN-MAP, local addr 12.12.12.1

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.1.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (192.168.2.0/255.255.255.0/0/0)
   current_peer 23.23.23.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 14, #pkts encrypt: 14, #pkts digest: 14
    #pkts decaps: 14, #pkts decrypt: 14, #pkts verify: 14
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts no crypto_map: 0, #pkts internal err: 0

     inbound esp sas:
      spi: 0x4A2B81F0(1244365296)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2001, flow_id: 1, sibling_flags 80000040, crypto map: VPN-MAP
        sa timing: remaining key lifetime (k/sec): (4492780/3540)

     outbound esp sas:
      spi: 0x9D14C3E2(2635383778)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2002, flow_id: 2, sibling_flags 80000040, crypto map: VPN-MAP
        sa timing: remaining key lifetime (k/sec): (4492780/3540)
```
> [!TIP]
> Confirm that **`#pkts encaps`** and **`#pkts decaps`** are incrementing with each ping or HTTP request sent across the VPN.

### 4.6 Verify Crypto Map Configuration (`show crypto map`)
```cisco
HQ-Router0# show crypto map
Crypto Map "VPN-MAP" 10 ipsec-isakmp 
        Peer = 23.23.23.2
        Extended IP access list 101
            access-list 101 permit ip 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255
        Current peer: 23.23.23.2
        Security association lifetime: 4608000 kilobytes/3600 seconds
        PFS (Diffie-Hellman group): none
        Transform sets={ 
                TS-VPN: { esp-aes esp-sha-hmac  } , 
        }
        Interfaces using crypto map VPN-MAP:
                FastEthernet0/0
```

---

## 5. Step 5: Packet Tracer Simulation Mode Analysis

To observe the cryptographic packet flow in Cisco Packet Tracer:

1. Click **Simulation** mode (bottom-right icon or press `Shift + S`).
2. In the **Event List Filters**, select `ICMP`, `ISAKMP`, and `IPsec (ESP)`.
3. From **PC0**, send a ping to `192.168.1.10`.
4. Click **Capture / Forward** repeatedly:
   * Notice the initial **ISAKMP** UDP 500 packets exchanged between `12.12.12.1` and `23.23.23.2`.
   * Once ISAKMP completes, observe the packets change to **IPsec (ESP)** protocol 50 traversing `ISP-Router2`.
   * Click on the PDU packet on the ISP router: Inspect the **Inbound PDU Details** to verify that the private IP payload is completely encapsulated inside an ESP header.

---

## 6. Step 6: Application Layer Verification (HTTP / Web Browser)

1. Open **PC0** in Cisco Packet Tracer.
2. Navigate to **Desktop** -> **Web Browser**.
3. In the URL field, type: `http://192.168.1.10` and press **Go**.
4. **Expected Result:** The default Cisco Packet Tracer Server welcome web page displays successfully, proving full Layer 7 communication through the encrypted Layer 3 IPsec VPN tunnel.

---

## 7. Comprehensive Troubleshooting Matrix

| Issue / Symptom | Probable Cause | Diagnostic Command | Remediation Step |
| :--- | :--- | :--- | :--- |
| **Ping from PC0 to Server0 gives "Destination Host Unreachable"** | Default gateway missing or wrong on PC0 / Server0 | `ipconfig` on PC / Server | Set Gateway to `192.168.2.254` (PC0) and `192.168.1.254` (Server0). |
| **Ping gives 100% "Request Timed Out"** | Default route missing on HQ or Branch router | `show ip route` | Add `ip route 0.0.0.0 0.0.0.0 12.12.12.2` (HQ) and `ip route 0.0.0.0 0.0.0.0 23.23.23.1` (Branch). |
| **`show crypto isakmp sa` returns blank / no output** | No interesting traffic sent, or WAN link down | `ping 23.23.23.2` from HQ | Ping end-to-end (`PC0 -> Server0`) to trigger tunnel initiation. |
| **`show crypto isakmp sa` stuck in `MM_NO_STATE` or `MM_KEY_EXCH`** | Pre-shared key mismatch or crypto policy mismatch | `show crypto isakmp policy` | Verify `crypto isakmp key CiscoVPN2026!` matches on both sides and remote peer IP is correct. |
| **Phase 1 active (`QM_IDLE`), but Phase 2 fails (`#pkts encaps: 0`)** | Transform set mismatch or ACL mirror mismatch | `show crypto ipsec sa`, `show access-lists` | Ensure ACL 101 on HQ matches `permit ip 192.168.1.0 ... 192.168.2.0 ...` and Branch matches `permit ip 192.168.2.0 ... 192.168.1.0 ...`. |
| **Crypto map not encrypting** | Crypto map not bound to outgoing WAN interface | `show crypto map` | Apply `crypto map VPN-MAP` under `interface FastEthernet0/0` (HQ) or `FastEthernet0/1` (Branch). |
| **ISP drops packets** | Unencrypted private packets hitting ISP without VPN | `debug ip packet` on ISP | Verify crypto map is active so packets reach ISP with outer public IPs. |
