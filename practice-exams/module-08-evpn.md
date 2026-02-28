# L2VPN & EVPN — Practice Questions

Expert-level SP. 20 questions. Covers sections 8.1–8.5: VPWS, VPLS, EVPN route types, multi-homing, DCI.

---

## Questions

---

### Q1 — VPLS MAC Learning (Scenario)

In a VPLS instance, PE1 receives a frame from CE1 with source MAC 00:aa:bb:cc:dd:01 destined for unknown MAC 00:aa:bb:cc:dd:02. What does PE1 do?

A) Drops the frame — unknown unicast is not allowed in VPLS  
B) Learns the source MAC (00:aa:bb:cc:dd:01 → CE1 interface), floods the frame to all pseudowires in the VPLS instance  
C) Sends an ARP request to resolve 00:aa:bb:cc:dd:02  
D) Sends the frame only to the most recently active PE  

---

### Q2 — EVPN Route Type 2 — MAC/IP Advertisement (Config Analysis)

A VTEP advertises EVPN Route Type 2 with:
- MAC: 00:50:56:aa:bb:cc
- IP: 10.1.1.50
- VNI: 10001 (L2)
- VNI: 50001 (L3)

A remote VTEP receives this. What two things does the remote VTEP learn?

A) Only the MAC address and where to send L2 frames  
B) The MAC address (for bridging) and the host IP-to-VTEP mapping (for ARP suppression and L3 routing to 10.1.1.50)  
C) The VNI bindings only — the MAC/IP are irrelevant  
D) The BGP next-hop and nothing else  

---

### Q3 — EVPN Multi-Homing — Ethernet Segment (Scenario)

CE1 is dual-homed to PE1 and PE2 in **single-active mode**, with the same ESI (Ethernet Segment Identifier) `0x0001:0001:0001:0001:0001` on both PEs. PE1 is elected as the Designated Forwarder (DF). What does the DF election control?

A) Which PE forwards BUM (Broadcast, Unknown Unicast, Multicast) traffic toward the CE, and which PE sends known unicast toward the CE from the EVPN fabric  
B) Which PE is allowed to advertise the CE's MAC addresses into the EVPN control plane  
C) Which PE processes BGP routes for the CE  
D) Which PE maintains the ARP table for the CE's hosts  

---

### Q4 — EVPN Route Type 4 — Ethernet Segment Route (Design)

EVPN Type 4 routes are advertised by both PE1 and PE2 for the same ESI. What is the purpose of Type 4?

A) To advertise the customer's MAC addresses to remote PEs  
B) To enable DF election — PEs discover each other via Type 4, then run the DF election algorithm to select the forwarder for each VLAN/EVI  
C) To distribute IP routing information for the multi-homed CE  
D) To announce that the Ethernet Segment is administratively down  

---

### Q5 — EVPN Mass Withdrawal (Scenario)

PE1 dual-homes CE1 along with PE2 (same ESI). PE1's link to CE1 fails (the access-circuit / ES link). PE1's MPLS core connectivity is still intact. What EVPN mechanism minimizes reconvergence time?

A) Type 1 mass withdrawal — PE1 withdraws a single Type 1 route for the entire ESI, signaling to all remote PEs that all MAC/IPs reachable via PE1 are gone  
B) PE1 individually withdraws each Type 2 MAC/IP route for CE1's hosts  
C) PE2 detects PE1's failure via BFD and takes over  
D) The CE detects the failure and re-sends gratuitous ARP to flush MAC tables  

---

### Q6 — EVPN Aliasing (Design)

In EVPN all-active multi-homing, both PE1 and PE2 are active forwarders for CE1 (same ESI). Only one is the DF per EVI, but both can forward known unicast. A remote PE3 wants to load-balance traffic to CE1's MAC. What mechanism enables this without CE1 needing LACP?

A) Route Type 1 (Ethernet Auto-Discovery per ESI) is used for aliasing — PE3 can send traffic to either PE1 or PE2 for CE1's MAC, even if only one PE originally advertised the MAC/IP Type 2 route  
B) PE3 uses ECMP natively based on the two different BGP next-hops in the Type 2 routes  
C) CE1 advertises its own MAC to both PEs simultaneously, enabling ECMP  
D) Aliasing requires STP — the non-root PE is put in forwarding state for all VLANs  

---

### Q7 — EVPN Type 5 — IP Prefix Route (Scenario)

An EVPN spine router receives a Type 5 route for 10.0.0.0/8 with GW-IP 192.168.1.1 and L3 VNI 50001. What does this route represent?

A) A host /32 MAC/IP advertisement compressed to a subnet  
B) An IP prefix route (subnet or aggregate) used for inter-subnet routing — the GW-IP identifies the next-hop gateway within the L3 VNI, and the L3 VNI identifies the routing domain  
C) A Type 5 route is only used for DCI — it doesn't participate in local EVPN forwarding  
D) An external route redistributed from OSPF into EVPN, only valid for Type-5 gateway  

---

### Q8 — EVPN Integrated Routing and Bridging (IRB) — Asymmetric vs Symmetric (Design)

What is the key difference between asymmetric and symmetric IRB in EVPN-VXLAN?

A) Asymmetric IRB requires both a L2 VNI and L3 VNI on every VTEP; symmetric only needs L2 VNI  
B) In asymmetric IRB, the ingress VTEP performs both routing and bridging — it looks up the destination MAC, encapsulates with the destination L2 VNI, and forwards; the egress VTEP only bridges. In symmetric IRB, both ingress and egress VTEPs route — traffic is encapsulated with the L3 VNI on the wire, and only the relevant local L2 VNIs are needed per VTEP  
C) Symmetric IRB is not standards-based and is vendor-specific  
D) Asymmetric IRB uses Type 5 routes; symmetric uses Type 2 routes exclusively  

---

### Q9 — VPLS Flooding vs EVPN ARP Suppression (Design)

In a VPLS network with 200 PEs and 1,000 hosts per VPN, what is the BUM traffic impact vs. an EVPN network with ARP suppression?

A) VPLS and EVPN have identical BUM traffic — flooding is required for both  
B) VPLS floods BUM to all 199 remote PEs. EVPN with ARP suppression builds a local ARP cache from Type 2 MAC/IP routes — ARP requests are answered locally, eliminating ARP flood across the MPLS/IP underlay  
C) EVPN eliminates all BUM traffic including unknown unicast  
D) VPLS uses multicast for BUM, which is more efficient than EVPN's unicast replication  

---

### Q10 — H-VPLS (Hierarchical VPLS) (Design)

What problem does H-VPLS solve compared to flat VPLS?

A) H-VPLS reduces MPLS label overhead by using LDP instead of RSVP-TE  
B) H-VPLS reduces the full-mesh pseudowire requirement. In flat VPLS, every PE needs a pseudowire to every other PE (O(N²) scale). H-VPLS introduces a two-tier hierarchy: MTU (access) nodes connect via spoke PWs to U-PE (hub) nodes, which maintain the full mesh. MTUs don't need PW to every remote PE  
C) H-VPLS enables IPv6 support in VPLS  
D) H-VPLS uses BGP for MAC learning instead of data-plane flooding  

---

### Q11 — EVPN Split-Horizon (Scenario)

In EVPN all-active multi-homing, PE1 and PE2 both connect to CE1 via the same ESI. PE1 receives a BUM frame from CE1. What prevents PE1 from sending the flooded frame back to CE1 via PE2?

A) STP is run between PE1 and PE2 to prevent loops  
B) The ESI label (split-horizon label) — PE2 receives the flooded BUM from PE1 with the ESI label. PE2 sees the ESI matches its own local ESI for CE1 and does NOT forward the frame to CE1 (split-horizon by ESI)  
C) PE1 marks the frame with a loop-prevention community; PE2 discards it  
D) The BGP route-target is used to identify the originating PE and filter  

---

### Q12 — EVPN DCI — Border Gateway (Scenario)

Two data centers, DC1 and DC2, run EVPN-VXLAN internally. An EVPN Border Gateway (BGW) connects them. A Type 2 route from DC1 host 10.1.1.1 must reach DC2. What does the BGW do with the route?

A) Strips the VXLAN encapsulation and re-encapsulates with MPLS for the WAN  
B) Re-originates the Type 2 route (or Type 5 equivalent) with the BGW's own loopback as next-hop, normalizing VNI/RT values for the inter-DC fabric — the original VTEP IP is not exposed across the DCI link  
C) Passes the Type 2 route unchanged with the original VTEP as next-hop  
D) Converts the Type 2 MAC/IP to a Type 5 IP prefix for efficiency on the DCI link  

---

### Q13 — EVPN Route Type 1 — Ethernet Auto-Discovery (Config Analysis)

When is an EVPN Type 1 (A-D) route sent "per EVI" vs. "per ESI"?

A) Per-ESI routes handle DF election; per-EVI routes handle aliasing and mass withdrawal  
B) Per-ESI routes handle mass withdrawal (one withdraw per ESI brings down all EVIs); per-EVI routes handle aliasing (advertising that a PE is reachable for an ES in a specific EVI for load-balancing)  
C) There is no distinction — Type 1 routes are always per-ESI  
D) Per-EVI routes are only used in VXLAN; per-ESI in MPLS  

---

### Q14 — EVPN Multihoming — Active-Active vs Single-Active (Design)

What is the fundamental difference between EVPN all-active and single-active multi-homing?

A) All-active: both PEs can forward traffic from the CE simultaneously. Single-active: only one PE forwards (the DF), the other is standby  
B) All-active: only one PE is the DF. Single-active: both PEs forward simultaneously  
C) All-active requires STP; single-active uses DF election  
D) All-active is only supported with VXLAN; single-active works with MPLS  

---

### Q15 — EVPN-MPLS vs EVPN-VXLAN (Design)

An SP backbone uses MPLS. A new data center uses IP-only leaf-spine fabric. They need to bridge L2 across the WAN. What encapsulation is used where?

A) MPLS everywhere — the DC fabric must run MPLS  
B) VXLAN in the DC fabric; MPLS on the WAN. Border/gateway routers perform VXLAN→MPLS encapsulation translation  
C) VXLAN everywhere — the SP WAN adopts VXLAN for simplicity  
D) GRE on the WAN with VXLAN inside  

---

### Q16 — EVPN ARP/ND Suppression Detail (Scenario)

Host A (10.1.1.10) ARPs for Host B (10.1.1.20). VTEP-A has learned Host B's MAC/IP via EVPN Type 2. What happens?

A) VTEP-A floods the ARP request to all remote VTEPs  
B) VTEP-A intercepts the ARP request, looks up its local EVPN-derived ARP cache for 10.1.1.20, and responds with Host B's MAC on behalf of VTEP-B — the ARP never crosses the underlay  
C) VTEP-A forwards the ARP to VTEP-B which responds directly  
D) VTEP-A drops the ARP — EVPN-VXLAN doesn't support ARP  

---

### Q17 — VPWS vs EVPN VPWS (Design)

Traditional VPWS (RFC 8077) uses LDP or RSVP for pseudowire signaling. EVPN VPWS (RFC 8214) uses BGP. What advantage does EVPN VPWS provide over traditional VPWS for multi-homing?

A) EVPN VPWS has lower latency because BGP converges faster than LDP  
B) EVPN VPWS supports multi-homed CE attachment (all-active or single-active) natively — the CE can be connected to two PEs with automatic failover. Traditional VPWS doesn't have native multi-homing support  
C) EVPN VPWS uses smaller labels (16-bit vs 20-bit)  
D) Traditional VPWS requires RSVP-TE; EVPN VPWS doesn't require MPLS at all  

---

### Q18 — MAC Mobility (Scenario)

Host 00:aa:bb:cc:dd:01 was learned at VTEP-1 (Type 2 route: next-hop VTEP-1). The host migrates to VTEP-2. VTEP-2 learns the MAC locally and advertises a new Type 2 route. What mechanism handles this without permanent routing inconsistency?

A) All remote VTEPs flush their MAC tables when they receive a duplicate Type 2  
B) EVPN MAC Mobility — VTEP-2 increments the MAC Mobility sequence number in its Type 2 route. Remote VTEPs prefer the higher sequence number, replacing VTEP-1's stale entry. VTEP-1 eventually withdraws its Type 2 when it stops seeing the MAC locally  
C) Both VTEP-1 and VTEP-2 routes coexist — ECMP is used  
D) The MAC address is marked as duplicate and both routes are withdrawn  

---

### Q19 — EVPN with OSPF Redistribution (Scenario)

A PE running EVPN imports routes from OSPF into the EVPN L3 VNI. The OSPF routes appear as Type 5 in EVPN. A remote VTEP receives the Type 5 route. What encapsulation is used to forward traffic from the remote VTEP to the originating PE?

A) VXLAN with L2 VNI — the Type 5 route uses the same VNI as the L2 domain  
B) VXLAN with L3 VNI — Type 5 routes are IP prefix routes forwarded via the VRF, encapsulated using the L3 VNI (routing domain identifier)  
C) Native IP — Type 5 routes bypass VXLAN encapsulation  
D) MPLS — Type 5 routes always use MPLS regardless of underlay  

---

### Q20 — EVPN Designated Forwarder Election Algorithm (Scenario)

PE1 and PE2 share ESI `0x0001:0001:0001:0001:0001` for a multi-homed CE. For EVI 100, the default DF election algorithm (modulo) is used. PE1 has ordinal 0, PE2 has ordinal 1 (sorted by BGP next-hop). EVI 100 uses `(EVI mod number_of_PEs) = 100 mod 2 = 0`. Which PE is the DF for EVI 100?

A) PE2 — ordinal 1 wins for all EVIs  
B) PE1 — ordinal 0 matches the result of `100 mod 2 = 0`  
C) Both PEs are DF simultaneously — DF is a per-flow concept  
D) Neither — modulo election only applies to VPLS  

---

## Answers and Explanations

---

### A1 — Answer: **B**

**Explanation:** VPLS performs **data-plane MAC learning** exactly like a traditional Ethernet switch. When PE1 receives a frame:
1. **Learns** the source MAC → maps it to the incoming pseudowire/interface
2. **Looks up** the destination MAC in the MAC table
3. If unknown (not in table) → **floods** the frame to all other pseudowires in the VPLS instance (including all remote PEs)

This flooding behavior is a key scalability problem with VPLS — BUM traffic is replicated to every PE in the instance, regardless of whether any local CE needs it. EVPN solves this via ARP suppression and BGP-based MAC distribution.

---

### A2 — Answer: **B**

**Explanation:** EVPN Type 2 (MAC/IP Advertisement) carries both the MAC address AND an IP address (when the host's IP is known). Remote VTEPs use this to:
1. **Populate their MAC table**: Forward L2 frames for this MAC to the advertising VTEP (using L2 VNI 10001)
2. **Populate their ARP cache**: When a local host ARPs for 10.1.1.50, the VTEP answers locally without flooding — it knows the MAC is at the remote VTEP
3. **Install an L3 host route**: Traffic to 10.1.1.50 can be routed directly to the advertising VTEP using the L3 VNI (50001), enabling anycast gateway routing

---

### A3 — Answer: **A**

**Explanation:** In EVPN **single-active** multi-homing, the DF controls all traffic forwarding to/from the CE:
- **BUM from fabric → CE**: Only the DF forwards BUM to CE (prevents duplicate delivery)
- **Known unicast from fabric → CE**: Only the DF forwards (single-active means one PE is active at a time)
- **Traffic from CE → fabric**: CE sends all traffic on the active link (to the DF); the non-DF link is standby

In **all-active** multi-homing (different scenario):
- CE sends/receives on both PE links simultaneously
- DF still controls BUM **from fabric → CE** (to prevent duplicates reaching CE)
- Known unicast from fabric can go to either PE (aliasing enables load-balancing)
- BUM from CE → fabric: Both PEs CAN forward, but split-horizon (ESI label) prevents loops

DF election is per-EVI (or per-VLAN), so different VLANs may have different DFs for load-balancing even in all-active mode.

---

### A4 — Answer: **B**

**Explanation:** EVPN Type 4 (Ethernet Segment Route) serves two purposes:
1. **ES discovery**: PEs with the same ESI discover each other — a PE learns which other PEs share its multi-homed CE
2. **DF election**: After discovering peer PEs via Type 4, all PEs run the DF election algorithm (default: modulo hash on EVI ID) to determine who is the DF per EVI

Type 4 contains the ESI, the IP address of the advertising PE, and EVPN-specific attributes. Without Type 4, multi-homing PEs can't coordinate DF election.

---

### A5 — Answer: **A**

**Explanation:** EVPN **mass withdrawal** (RFC 7432) uses Type 1 (Ethernet Auto-Discovery per ESI route) for efficient convergence. When PE1 loses its uplink:
- PE1 **withdraws a single Type 1 per-ESI route**
- Remote PEs receive this single withdrawal and know ALL MAC/IPs reachable via PE1 for this ESI are now invalid
- Without mass withdrawal, PE1 would need to withdraw each Type 2 (MAC/IP) individually — potentially thousands of routes

This reduces convergence time from O(N_MACs) withdrawals to O(1) withdrawal. PE2 (still connected to CE1) continues advertising its own Type 2 routes — traffic reroutes to PE2.

---

### A6 — Answer: **A**

**Explanation:** **EVPN Aliasing** allows remote PEs to load-balance traffic to a multi-homed CE even if only one PE originally advertised a particular MAC's Type 2 route. Here's how it works:
- PE1 advertises: Type 2 for MAC-X (its next-hop) + Type 1 per-EVI route (advertising ESI reachability)
- PE2 also advertises Type 1 per-EVI for the same ESI
- Remote PE3 receives Type 2 (next-hop PE1) but ALSO sees Type 1 from PE2 (same ESI)
- PE3 uses PE2 as an **alias** — can forward traffic to PE2 for MAC-X even though PE2 never sent a Type 2 for it
- This enables ECMP to the CE across both PEs without requiring the CE to run LACP or send traffic on both links

---

### A7 — Answer: **B**

**Explanation:** EVPN Type 5 (IP Prefix Route) is used for **IP prefix advertisement** in EVPN L3VPN contexts — subnets, aggregates, or external routes (from OSPF/BGP redistribution). Unlike Type 2 (host /32 or /128), Type 5 advertises longer-match prefixes like 10.0.0.0/8.
- **GW-IP**: The IP address of the gateway within the L3 domain — where traffic to this prefix should be sent after decapsulation
- **L3 VNI**: Identifies the routing domain (VRF)
- **BGP next-hop**: The VTEP/PE to tunnel the packet to

Type 5 enables routing between different L2 domains and external networks within the EVPN fabric.

---

### A8 — Answer: **B**

**Explanation:**
- **Asymmetric IRB**: Ingress VTEP does BOTH routing AND bridging. Receives frame in L2 VNI → routes to destination subnet → looks up destination host MAC → encapsulates with destination L2 VNI → forwards. The egress VTEP only bridges. Requires ALL L2 VNIs on ALL VTEPs (even VNIs for subnets with no local hosts). Simple but memory-intensive.
- **Symmetric IRB**: Ingress VTEP routes to L3 VNI → sends packet with L3 VNI to egress VTEP → egress VTEP routes within L3 VNI → bridges into local L2 VNI. Both ends do routing. Only requires local L2 VNIs + shared L3 VNI. More scalable; standard for large fabrics.

---

### A9 — Answer: **B**

**Explanation:** VPLS flooding: Every BUM frame is replicated to every pseudowire. 200 PEs × unknown ARP rate = massive BUM replication. EVPN ARP suppression works because:
1. Every host's IP-to-MAC binding is distributed via Type 2 MAC/IP routes at learn time
2. VTEPs maintain a local ARP table populated from EVPN control plane
3. When Host A ARPs for Host B, the local VTEP looks up its ARP cache and responds directly
4. ARP requests never traverse the underlay network

BUM traffic is NOT completely eliminated (unknown unicast still floods until MAC is learned), but ARP (a major BUM component) is suppressed.

---

### A10 — Answer: **B**

**Explanation:** Flat VPLS full-mesh: with N PEs, each PE needs N-1 pseudowires = O(N²) total PWs. At 1,000 PEs = ~500,000 pseudowires network-wide — unmanageable.

H-VPLS hierarchy:
- **Access layer (n-PE/MTU)**: Customer-facing PEs with spoke pseudowires to U-PE
- **Core layer (u-PE)**: Core PEs maintain the full mesh among themselves
- MTU only has 1 PW (to its U-PE) instead of N-1 PWs

MTU/n-PE doesn't need BGP or a full mesh — it's essentially a "remote VPLS access switch." This reduces PW scale dramatically while keeping VPLS semantics.

---

### A11 — Answer: **B**

**Explanation:** EVPN split-horizon for multi-homing uses an **ESI label** (Ethernet Segment Identifier label):
1. PE1 receives BUM from CE1 (ESI=X)
2. PE1 floods to all remote PEs, including PE2, carrying the ESI label for ESI=X
3. PE2 receives the flooded BUM with ESI label indicating origin ESI=X
4. PE2 recognizes ESI=X is its own local ESI (connected to CE1)
5. PE2 **does not forward the frame to CE1** — split-horizon applied

This prevents CE1 from receiving its own broadcast back via the other multi-homing PE, which would cause loops.

---

### A12 — Answer: **B**

**Explanation:** A DCI Border Gateway (BGW) is a **route policy + re-origination point**:
- Routes from DC1's EVPN fabric (with DC1 VNIs and VTEP IPs) are re-originated by BGW with:
  - BGW's own loopback as new next-hop
  - Translated/normalized VNI values if DC1 and DC2 use different VNIs
  - Appropriate RT values for cross-DC import

The remote DC2 VTEPs never see DC1's internal VTEP addresses — traffic goes to BGW, which bridges to the remote DC. This provides topology hiding and VNI translation.

---

### A13 — Answer: **B**

**Explanation:** Type 1 has two sub-types:
- **Per-ESI** (AD route per ES): One route per ESI, no EVI tag. Used for **mass withdrawal** — withdrawing this single route signals loss of all MAC/IP reachability via this ES across all EVIs.
- **Per-EVI** (AD route per ES per EVI): One route per ESI per EVI. Used for **aliasing** — advertising that this PE can forward traffic for this ESI in this specific EVI (enabling remote PEs to use this PE as an alias for MAC/IP not directly advertised here).

Understanding this distinction is critical for multi-homing and troubleshooting.

---

### A14 — Answer: **A**

**Explanation:**
- **All-active**: Both PE1 and PE2 are DFs for traffic from CE1. CE1 can send and receive traffic on both links simultaneously. Remote PEs can load-balance to both PE1 and PE2 (via aliasing). Provides link-level load balancing and fast failover.
- **Single-active**: Only the elected DF forwards traffic from the CE. The non-DF PE is in standby — it maintains the BGP adjacency and ESI info but drops forwarded traffic from CE. Simpler but doesn't provide per-flow load balancing.

---

### A15 — Answer: **B**

**Explanation:** This is the canonical SP/DC interconnect architecture:
- **DC fabric**: IP-only leaf-spine → VXLAN (UDP encapsulation, no MPLS needed)
- **WAN/SP core**: SR-MPLS or LDP → MPLS labels
- **BGW/PE border router**: Terminates VXLAN from the DC, re-encapsulates as MPLS for the WAN (EVPN-MPLS). Performs VXLAN VNI ↔ MPLS label mapping.

Many implementations use EVPN as the common control plane for both — the same BGP EVPN routes are exchanged, only the encapsulation (data-plane) differs.

---

### A16 — Answer: **B**

**Explanation:** EVPN ARP suppression is one of its highest-value features. When VTEP-A receives an ARP request from Host A for 10.1.1.20:
1. VTEP-A checks its EVPN-derived ARP table (populated by Type 2 MAC/IP routes from VTEP-B)
2. If found → VTEP-A sends an ARP reply with Host B's MAC on behalf of VTEP-B
3. The ARP request is never forwarded across the VXLAN underlay
4. Host A gets the MAC immediately; no BUM flooding occurs

This eliminates ARP storms in large-scale EVPN fabrics and reduces underlay bandwidth significantly.

---

### A17 — Answer: **B**

**Explanation:** Traditional VPWS is a point-to-point pseudowire — one CE port to one PE-end. Multi-homing requires proprietary extensions (e.g., RFC 6870 PW redundancy) which are limited. EVPN VPWS (RFC 8214) natively supports:
- **All-active multi-homing**: CE dual-homed to two PEs, both active, with load-balancing
- **Single-active multi-homing**: CE dual-homed, one active for redundancy
- Automatic failover using the EVPN control plane and Type 1 mass withdrawal

This is a significant improvement for customer multi-homing in point-to-point L2 services.

---

### A18 — Answer: **B**

**Explanation:** MAC Mobility (RFC 7432 Section 15) handles host migration between VTEPs:
1. VTEP-2 detects the MAC locally (gratuitous ARP, data-plane learning, or EVPN local learn)
2. VTEP-2 advertises Type 2 with the **MAC Mobility Extended Community** containing an incremented **sequence number** (one higher than VTEP-1's sequence, or starting at 1 if VTEP-1 had no sequence)
3. Remote VTEPs receive both Type 2 routes (from VTEP-1 and VTEP-2 for the same MAC)
4. Higher sequence number wins → remote VTEPs update next-hop to VTEP-2
5. VTEP-1 eventually withdraws its Type 2 when the MAC ages out locally

This enables seamless VM live migration in data center environments.

---

### A19 — Answer: **B**

**Explanation:** EVPN Type 5 routes are IP prefix routes associated with a **Layer 3 VNI** (L3 VNI) that identifies the routing domain (VRF). When a remote VTEP forwards traffic toward a Type 5 prefix:
1. Traffic is VXLAN-encapsulated with the **L3 VNI**
2. The outer IP destination is the advertising VTEP's loopback
3. The advertising VTEP receives the VXLAN packet, decapsulates, sees L3 VNI → routes in the VRF
4. Looks up the GW-IP (from the Type 5 route) and forwards accordingly

L2 VNI is for bridging (Type 2 MAC routes); L3 VNI is for routing (Type 5 and inter-subnet Type 2 with IP).

---

### A20 — Answer: **B**

**Explanation:** Default EVPN DF election algorithm (RFC 7432, updated by RFC 8584):

**Modulo-based**: 
- Sort all PEs sharing the ESI by their IP address (ordinal assignment: PE1=0, PE2=1 in this example)
- For each EVI: DF = PE with ordinal `(EVI_ID mod N)` where N = number of PEs in the ESI
- EVI 100: `100 mod 2 = 0` → ordinal 0 = PE1

PE1 is the DF for EVI 100. PE2 would be DF for EVI 101 (`101 mod 2 = 1`). This naturally load-balances DFs across EVIs without manual configuration.
