# Multi-Area OSPF Enterprise Routing Lab

## Overview

This project demonstrates the design and implementation of a complex enterprise routing topology using OSPFv2 as the Interior Gateway Protocol (IGP), combined with NAT/PAT for external connectivity simulation.

**The lab focuses on**:

- Multi-area OSPF hierarchical design
- DR/BDR election manipulation in Area 0
- Stub and Totally NSSA area implementation
- Virtual link configuration (for discontiguous Area 0 scenarios)
- NAT/PAT and ACL configuration for internet simulation
- Passive interface tuning for routing optimization
- End-to-end connectivity validation and troubleshooting

This topology simulates a realistic enterprise network with internal segmentation and controlled external access.

**Key Technical Challenges Solved**:

- Packet Tracer Limitation Workarounds: Successfully transitioned from a Point-to-Multipoint (P2MP) Non-Broadcast design to a Broadcast Multi-Access design on Ethernet segments to ensure stable adjacencies within the simulation environment.

- Timer Optimization: Implemented custom OSPF Hello and Dead intervals (doubled from defaults to 20/80) to simulate stability requirements for low-bandwidth or high-latency links.

- Routing Table Optimization: Utilized Stub and Totally NSSA area types to reduce Link State Database (LSDB) overhead and minimize routing table size on branch office routers.

---

## Network Topology

The network is segmented into multiple OSPF areas:

- **Area 0 (Backbone)** – The core transit area connecting all Area Border Routers (ABRs).
- **Area 2** – A standard OSPF area connecting localized segments to the backbone.
- **Area 3(Branch Stub)** – A multi-access segment (R3, R9, R10) configured as a Stub Area.
    - Behavior: Blocks external Type-5 LSAs; R3 (ABR) injects an O*IA default route.
- **Area 51(Totally NSSA)** – An optimized area for external route injection.
    - Behavior: Uses a Type-7 to Type-5 LSA translation at the ABR.
- **Discontiguous Area 0** – Linked virtually to Backbone over transit Area 2
- **ABRs** - Inter-area routing (R1, R2, R3, R4, R7)
- **ASBR** - External route injection (R5)
- **R1** – Edge router performing NAT/PAT toward ISP

📌 Topology Diagram:

<img width="1066" height="745" alt="01-network-topology" src="https://github.com/user-attachments/assets/08f1224b-ce86-4af9-988a-da148490189b" />

---

## OSPF Design Architecture

### OSPF Process

- OSPF Version: OSPFv2
- Process ID: 1
- Routing Type: Link-State

---

### Area Designation

| Area | Type | Purpose |
|------|------|--------|
| 0 | Backbone | Core routing domain |
| 2 | Standard | Internal segmentation |
| 3 | Stub Area | Reduced LSDB, no external routes |
| 51 | Totally NSSA | External route injection controlled |
| Discontiguous 0 | Virtual Extension of Area 0 | Transits over Area 2 |

---

## Key OSPF Features Implemented

---

### 1. DR / BDR Election (Area 0)

To control OSPF adjacency behavior, router priorities were manually adjusted:

- R3: Priority 250 → DR
- R2: Priority 200 → BDR

After configuration, the OSPF process was reset to force re-election:

```bash
clear ip ospf process
yes
```
This ensured deterministic election behavior in the broadcast domain.

---

### 2. Stub Area Configuration (Area 3)

Area 3 was configured as a Stub Area to reduce routing overhead and prevent external LSAs from being flooded.

Effects:

- No Type 5 LSAs allowed
- Default route injected automatically
- Reduced routing table size

---

### 3. Totally NSSA Configuration (Area 51)

Area 51 was configured as a Totally NSSA, allowing controlled external route injection while limiting inter-area complexity.

Characteristics:

- Accepts Type 7 LSAs
- Blocks external Type 5 LSAs
- Default route injected from ABR
- Supports limited external redistribution

---

### 4. Passive Interface Configuration

To optimize OSPF adjacency formation and reduce unnecessary neighbor relationships:

- R7 G0/1 was configured as a passive interface

Effect:

- Prevents OSPF hello packets on user-facing segment
- Maintains network advertisement without adjacency formation

---

### 5. NAT / PAT Configuration (R1 Edge Router)

R1 was configured as the edge router providing NAT/PAT translation toward the ISP.

**Issue Observed**

Internal nodes experienced:

- Failed ICMP echo replies to external networks

**Solution Implemented**

- PAT (Port Address Translation) was configured to ensure all internal traffic(listed in the ACL) could share a single public IP.
- ACL (Access Control List) to allow access to ISP to only desired networks in the organization.

**NAT Configuration Summary**
- Inside interface: LAN-facing interface
- Outside interface: ISP-facing interface
- NAT overload enabled

**Result**
- Stable outbound connectivity restored
- ICMP traffic successfully translated and returned
- ISP-side packet loss issue resolved

---

### Routing Verification

**OSPF Neighbor Status**
- Full adjacency achieved across all configured areas
- DR/BDR roles confirmed in Area 0

**Routing Table Behavior**
- Inter-area routes properly summarized
- Stub area default route installed
- NSSA external routes correctly translated

---

## Troubleshooting & Engineering Notes (The "Gold" Section)

This project required active troubleshooting of OSPF adjacency states:

| Issue encountered | Root Cause | Resolution |
| --- | --- | --- |
| **Invalid Input at '^'** | Packet Tracer lacks support for `ip ospf network point-to-multipoint non-broadcast`. | Reverted to **Broadcast** network type and utilized manual DR priority to achieve similar logical behavior. |
| **Stuck in 2WAY/DROTHER** | OSPF "Wait" timer (80s) and non-preemptive election prevented R3 from becoming DR. | Performed a coordinated `clear ip ospf process` and interface cycle to force a fresh election with R3 as the victor. |
| **Stuck in EXSTART** | Database Description (DBD) packet exchange failure due to initial timer mismatches. | Synchronized Hello/Dead intervals (20/80) across all neighbors in Area 3. |
| **Empty Routing Table (R9)** | Stub flag mismatch; R3 was configured as Stub but R9 was "Normal." | Standardized the `area 3 stub` command across all Area 3 participants to allow LSDB synchronization. |

---

## Key Networking Concepts Demonstrated

- OSPF multi-area hierarchical design
- DR/BDR election manipulation
- Stub and Totally NSSA area behavior
- ABR/ASBR role separation
- Link-State Database optimization
- Virtual link usage (if applicable in topology)
- NAT/PAT (Port Address Translation)
- Enterprise edge routing behavior
- OSPF troubleshooting and convergence analysis

---

## Author

- Natnael Haile
