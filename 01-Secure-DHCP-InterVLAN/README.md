# 🏢 Secure Enterprise Network with Inter-VLAN Routing & DHCP Snooping

## 📐 Network Topology Diagram
![Network Topology](topology.png)
*Figure 1: Cisco Packet Tracer implementation showing R1 Router, SW1 Switch, and segmented endpoint subnets.*

## 📝 Project Overview
This project demonstrates the design and deployment of a secure, segmented corporate local area network (LAN). It leverages a **Router-on-a-Stick (RoaS)** topology to provide efficient inter-VLAN 
routing while ensuring mitigation against rogue DHCP attacks using advanced Layer 2 hardening techniques.

### 🔑 Key Infrastructure Objectives:
- **Logical Segmentation:** Isolating network infrastructure into distinct staff and guest access environments.
- **Dynamic IP Allocation:** Automating client addressing locally via central routing resources.
- **Layer 2 Security Hardening:** Mitigating network vulnerabilities (man-in-the-middle attacks) via mitigation protocols.

---

## 🛠️ Protocol & Feature Implementation Detail

### 1. Inter-VLAN Routing (Router-on-a-Stick)
- **Sub-Interfaces:** Segmented `GigabitEthernet 0/0` into logical sub-interfaces to match operational parameters.
- **Encapsulation:** Enforced **IEEE 802.1Q** trunk encapsulation headers on the gateway path.

### 2. Network Services (DHCP Architecture)
- Centralized IP address pools configured directly on gateway engine `R1` to handle automated address management parameters:
  - **STAFF Pool:** Allotted to internal corporate assets inside local subnets.
  - **GUEST Pool:** Isolated environment handling visitor or untrusted network devices.

### 3. Layer 2 Performance & Spanning Tree
- Migrated legacy STP convergence metrics over to **Rapid Per-VLAN Spanning Tree Plus (Rapid-PVST+)**. This drops topology stabilization delays drastically during physical link failures.

### 4. Infrastructure Security (DHCP Snooping Hardening)
- Enabled global **DHCP Snooping** boundaries across active operational scopes (`VLAN 10-20`).
- Hardened access interfaces by treating them as untrusted lines, actively dropping rogue server packets.
- Defined trunk interface `GigabitEthernet 0/1` facing the authentic DHCP engine as an explicitly **Trusted Port**.
- **Important Cisco Fix:** Applied `no ip dhcp snooping information option` to disable Option 82 injection, preventing Cisco R1 from dropping endpoint discovery packets.

---

## 🔢 Addressing & Logical Architecture Matrix

| Device Name | Interface / Scope | Assigned VLAN | Subnet Boundary | IP Address / Gateway | Mode |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | `g0/0.10` | VLAN 10 | 192.168.10.0/24 | 192.168.10.1 | Gateway (dot1Q) |
| **R1** | `g0/0.20` | VLAN 20 | 192.168.20.0/24 | 192.168.20.1 | Gateway (dot1Q) |
| **SW1** | `Fa0/1, Fa0/3` | VLAN 10 (Office_staff) | 192.168.10.0/24 | DHCP Allocated | Access |
| **SW1** | `Fa0/2, Fa0/4` | VLAN 20 (guest) | 192.168.20.0/24 | DHCP Allocated | Access |
| **SW1** | `Gi0/1` | Multi-VLAN | Trunk (10, 20) | N/A | Trunk (**Trusted**) |

---

## 💾 Core Device Configurations (CLI Highlights)

### Router (R1) Inter-VLAN & DHCP Setup:
```ios
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
ip dhcp pool STAFF
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
!
ip dhcp pool guest
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
```

### Switch (SW1) Layer 2 Hardening & Security Setup:
```ios
spanning-tree mode rapid-pvst
!
vlan 10
 name Office_staff
vlan 20
 name guest
!
ip dhcp snooping
ip dhcp snooping vlan 10-20
no ip dhcp snooping information option
!
interface GigabitEthernet0/1
 switchport mode trunk
 ip dhcp snooping trust
```

---

## 🔍 Validation & Operational Verification
- ✅ **Dynamic Lease Validation:** Endpoints inside both VLAN blocks successfully request IP scopes securely.
- ✅ **Trunk Enforcement:** `show interfaces trunk` validates proper transport of tagging parameters across nodes.
- ✅ **Snooping Verification:** Untrusted interfaces successfully filter out malicious rogue DHCP broadcast responses.
