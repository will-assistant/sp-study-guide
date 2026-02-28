# L3VPN — Practice Questions

Expert-level SP. 20 questions. Covers sections 7.1–7.5: architecture, MP-BGP VPNv4, inter-AS, extranet, scale.

---

## Questions

---

### Q1 — RD and RT Distinction (Design)

A customer has sites in two ASes. The SP assigns RD `65000:100` to all PE-CE links for this customer. What is the problem?

A) No problem — RD just needs to be unique within a VPN, not globally  
B) The RD must be globally unique to distinguish routes from different VRFs; reusing the same RD across multiple PEs (even for the same customer) can cause routing loops and incorrect path selection at the RR  
C) RDs must be unique per PE-CE link; they cannot be reused across PEs  
D) Using an RD from AS 65000 when the customer is in another AS violates RFC 4364  

---

### Q2 — VPNv4 Route Import (Config Analysis)

```
vrf definition CUSTOMER-A
 rd 65000:100
 !
 address-family ipv4
  route-target export 65000:100
  route-target import 65000:100
  route-target import 65000:999
 exit-address-family
```

A route arrives with RT `65000:999`. What happens?

A) The route is rejected — it doesn't match the export RT  
B) The route is imported into VRF CUSTOMER-A — the import RT matches `65000:999`  
C) The route is imported only if its RD also matches `65000:100`  
D) The route is advertised out of CUSTOMER-A with RT `65000:100` added  

---

### Q3 — MP-BGP VPNv4 Next-Hop (Scenario)

PE1 and PE2 are in the same AS. PE1 imports a CE route 192.168.1.0/24 into VRF_A. It advertises the VPNv4 route to PE2 via iBGP. What is the next-hop attribute in the VPNv4 UPDATE sent to PE2?

A) PE1's loopback IP (if `next-hop-self` is configured on the iBGP session)  
B) CE1's interface IP  
C) PE1's loopback IP by default — VPNv4 iBGP updates always use the PE's loopback as next-hop  
D) The RR's loopback if route reflection is used  

---

### Q4 — VPN Label Allocation (Design)

An SP is choosing between per-VRF and per-prefix VPN label allocation on a PE with 1,000 VRFs each carrying 1,000 routes. How does this affect LFIB size?

A) Per-prefix: 1,000,000 LFIB entries. Per-VRF: 1,000 LFIB entries (one per VRF). Per-VRF requires an IP lookup after label pop; per-prefix does not.  
B) Per-prefix and per-VRF result in the same LFIB size — labels are only allocated for active routes  
C) Per-VRF always has more LFIB entries because each VRF needs multiple labels for ECMP  
D) Per-prefix: 1,000 entries. Per-VRF: 1,000,000 entries  

---

### Q5 — Inter-AS Option A (Scenario)

Two SP networks (AS65000 and AS65001) connect via an ASBR link. Both use Option A (VRF-lite). A customer route in AS65000 needs to reach the same customer in AS65001. What must exist on each ASBR?

A) A VRF for the customer on each ASBR, with a dedicated PE-PE link per VRF between ASBRs  
B) A single eBGP session between ASBRs carrying VPNv4 NLRIs  
C) An MPLS tunnel between the two ASes with label binding via LDP  
D) Route reflection between the two AS's RRs  

---

### Q6 — Inter-AS Option B (Scenario)

Inter-AS Option B uses MPLS label binding between ASBRs. An ASBR in AS65000 receives a VPNv4 route from its RR with label 16010 (VPN label) + 17005 (transport label to PE1). When the ASBR receives the VPNv4 route from the neighboring ASBR in AS65001, what does it do with the labels?

A) Allocates a new VPN label locally and swaps it for the received label; transport label is regenerated  
B) Passes the entire label stack unchanged to the AS65001 ASBR  
C) Strips both labels and re-advertises the route as a plain IP route  
D) Uses the same label values across both ASes — label consistency is required  

---

### Q7 — Inter-AS Option C (Design)

What is the key architectural difference of Inter-AS Option C vs. Options A and B?

A) Option C uses RSVP-TE tunnels between ASBRs instead of BGP  
B) Option C establishes multi-hop MP-eBGP directly between PEs (or RRs) across AS boundaries, with ASBR-to-ASBR MPLS label exchange for transport but no VPN route processing at ASBRs  
C) Option C requires VRFs on ASBRs (like Option A) but with a single shared VRF for all customers  
D) Option C uses GRE tunnels between PEs instead of MPLS  

---

### Q8 — Extranet VPN (Config Analysis)

Customer A (RT import/export: `65000:100`) wants read-only access to a shared services VRF (RT export: `65000:500`). What RT configuration on Customer A's VRF achieves this?

A) Add `route-target import 65000:500` to Customer A's VRF; no changes to the shared services VRF  
B) Add `route-target export 65000:100` to shared services VRF and `route-target import 65000:100` to Customer A  
C) Both VRFs must have identical RT import/export values  
D) Use a dedicated "hub" VRF with routes redistributed between the two VRFs  

---

### Q9 — VPN Route Propagation (Troubleshooting)

PE1 learns a customer route 10.10.0.0/24 via OSPF from CE1. It doesn't appear in PE2's VRF routing table. The VPNv4 BGP session is up. What should you check first?

A) Whether the route is being redistributed from OSPF into BGP on PE1 (`redistribute ospf` under `address-family vpnv4`)  
B) Whether PE2's loopback is reachable from PE1  
C) Whether the RD on PE1 and PE2 match  
D) Whether the export RT on PE1 matches an import RT on PE2  

---

### Q10 — Carrier's Carrier (CsC) (Design)

In a Carrier's Carrier scenario, a customer SP runs MPLS within the backbone SP's network. What label behavior occurs at the customer SP's PE?

A) The customer SP PE uses the backbone SP's labels directly for forwarding  
B) The customer SP PE receives its own MPLS labels (inner stack) encapsulated within the backbone SP's MPLS stack (outer stack) — double label stacking  
C) The backbone SP strips all MPLS labels at the asPE; the customer SP runs native IP  
D) CsC requires RSVP-TE tunnels from the customer SP's PE to the backbone SP's PE  

---

### Q11 — PE-CE Protocol Choice (Design)

A CE router is managed by a third-party and the SP cannot change its configuration. The CE advertises only a default route to the PE. Which PE-CE protocol is most appropriate?

A) BGP — most flexible, any route can be exchanged  
B) Static routes on the PE — if the CE can't be configured for dynamic routing, static is the fallback  
C) RIP — simple and requires minimal CE config  
D) OSPF — redistributes automatically into BGP on the PE  

---

### Q12 — VPN Convergence (Scenario)

A PE-CE link fails. BGP withdraw takes 5 seconds. The VPN route for 10.10.10.0/24 disappears from all remote PEs within 5 seconds. However, traffic to 10.10.10.0/24 continues to blackhole for 15 additional seconds at the ingress PE. What is the likely cause?

A) The VPN label mapping hasn't been updated in the LFIB  
B) CEF/FIB is not synchronized with the RIB — `ip cef accounting` is interfering  
C) BGP Next-Hop Tracking is delayed — the VPN label's transport next-hop (the egress PE's loopback) is still reachable, so the ingress PE doesn't immediately detect the remote PE's VRF route withdrawal  
D) RSVP-TE holddown timer prevents immediate reroute  

---

### Q13 — VPN Route Distinguisher Role (Design)

Two customers, CUST-A and CUST-B, both use prefix 10.1.0.0/16 in their VPNs. PE1 carries both VRFs. How does MP-BGP distinguish the two routes when advertising them to PE2?

A) By the route-target — the RT identifies which customer the route belongs to  
B) By the VPN label — different labels per route distinguish them  
C) By the Route Distinguisher (RD) — the RD prepended to the prefix creates two unique VPNv4 NLRIs: `RD-A:10.1.0.0/16` and `RD-B:10.1.0.0/16`  
D) By the BGP community — each customer's routes carry a unique BGP community  

---

### Q14 — OSPF as PE-CE (Scenario)

PE1 and CE1 run OSPF in VRF "ENTERPRISE" in area 0. CE1 has connected subnets 10.1.0.0/24 and 10.2.0.0/24 that are advertised as OSPF intra-area routes. PE2 also runs OSPF in the same VRF with the same Domain ID. PE2 receives CE1's routes via VPNv4 and imports them into its VRF. How does PE2 advertise these routes to CE2 via OSPF?

A) As OSPF Type 5 LSAs (External) — all VPN routes appear as external  
B) As OSPF Type 3 LSAs (Inter-area Summary) — the VPN backbone acts as a super-backbone; intra-area routes from one site appear as inter-area at the remote site  
C) As OSPF Type 1 LSAs (Intra-area) — the routes were intra-area at the source, so they stay intra-area  
D) PE2 doesn't redistribute VPN routes back into OSPF — manual redistribution is always required  

---

### Q15 — RT-Constrain Scale (Design)

A PE participates in 50 of 10,000 VPNs on an RR. Without RT-Constrain, the PE processes how many VPNv4 routes (assuming 1,000 routes per VPN)?

A) 50,000 (only its VPNs)  
B) 10,000,000 (all routes) — receives everything, installs only matching RT routes  
C) 0 — the RR handles all VPN route processing  
D) 1,000 (default RR sends only the PE's VPN routes)  

---

### Q16 — VPN Ping Failure (Troubleshooting)

A VPN ping from CE1 (AS65001) to CE2 (AS65001) fails. `show bgp vpnv4 unicast` on PE1 shows CE2's prefix with next-hop PE2 and a valid VPN label. `traceroute` from PE1 to PE2's loopback succeeds. What is the most likely cause of ping failure?

A) The VPN label is invalid — LFIB entry missing on P routers  
B) The transport label (LSP to PE2) is incomplete — PHP behavior is dropping the VPN label before PE2  
C) The PE-CE routing on PE2 is broken — the route exists in VPNv4 but is not imported into PE2's VRF  
D) CE2's ACL is blocking ICMP  

---

### Q17 — Hub-and-Spoke VPN (Design)

A customer wants a hub-and-spoke VPN: all spoke PEs must send traffic through the hub PE (for firewall inspection). The hub exports RT 65000:HUB. Spoke VRFs import 65000:HUB. What additional RT configuration prevents spokes from communicating directly with each other?

A) No additional config needed — hub-and-spoke is the default VPN behavior  
B) Spoke VRFs should export a spoke-specific RT (e.g., 65000:SPOKE). The hub VRF imports 65000:SPOKE. Spoke VRFs must NOT import 65000:SPOKE — they only import the hub's RT (65000:HUB). This way, spoke-to-spoke traffic only exists via the hub  
C) Apply an outbound BGP policy on the hub PE to drop direct spoke-to-spoke routes  
D) Configure split-horizon on the hub PE for VPN routes  

---

### Q18 — MPLS VPN Penultimate Hop (Scenario)

A packet from CE1 arrives at P1 (penultimate hop to PE2) with MPLS label stack [transport_label, vpn_label]. P1 performs PHP on the transport label. What does PE2 receive?

A) A plain IP packet — both labels are popped  
B) A packet with only the VPN label — PE2 uses the VPN label to identify the destination VRF  
C) A packet with both labels intact — PHP only applies to the outer label  
D) A packet with implicit-null label 3 at the top  

---

### Q19 — MPLS Traceroute in VPN (Troubleshooting)

An operator runs `traceroute mpls ipv4 vrf CUSTOMER 10.10.10.1/32`. The trace shows correct hops to the egress PE, then times out. What does this indicate?

A) The egress PE dropped the MPLS echo request — PE-CE reachability is the issue  
B) MPLS OAM is blocked by an ACL on the CE  
C) The VPN label is correctly distributed but the PE-CE segment (CE side) is unreachable or not responding to MPLS echo requests  
D) The ingress PE's VRF route for 10.10.10.1/32 is missing  

---

### Q20 — BGP-Free Core (Design)

An SP runs BGP only on PE routers. P (core) routers run only MPLS/IGP. How does a VPN packet traverse a P router without BGP?

A) P routers run a lightweight version of BGP (LDP-BGP) to process VPN labels  
B) VPN packets are encapsulated in GRE across the core — P routers forward GRE transparently  
C) P routers forward based on the MPLS transport label (LSP to the egress PE); they never see the VPN label (which is the inner label). The transport label is swapped at each P hop  
D) P routers install VPN routes via OSPF redistribution from PE  

---

## Answers and Explanations

---

### A1 — Answer: **B**

**Explanation:** The Route Distinguisher (RD) must be **globally unique** per RFC 4364. Its purpose is to make overlapping customer IP prefixes unique in the VPNv4 address space so the RR can distinguish them. If two PEs advertise the same RD for the same prefix, the RR sees them as the same route and may perform incorrect path selection (selecting one and discarding the other — even if they're from different sites). Best practice: RD = `AS:PE_id` or `PE_loopback:VRF_id` — unique per PE+VRF combination.

---

### A2 — Answer: **B**

**Explanation:** Route import is based on **RT matching**: if any of the route's RT extended communities match any import RT in the VRF, the route is imported. RT `65000:999` matches the `route-target import 65000:999` statement → route imported into CUSTOMER-A. The RD (option C) is not checked during import — it's only used for uniqueness in BGP. The export RT (option A) is unrelated to import decisions.

---

### A3 — Answer: **C**

**Explanation:** For VPNv4 iBGP, PE1 sets the NEXT_HOP to **its own loopback IP** by default. This differs from normal iBGP where next-hop is not changed. RFC 4364 requires the PE to set next-hop to its own loopback in VPNv4 advertisements — this is the address used to establish the transport LSP. The CE's IP (option B) would be unreachable from remote PEs. `next-hop-self` (option A) is redundant for VPNv4 because PEs automatically set next-hop to their loopback.

---

### A4 — Answer: **A**

**Explanation:** Label allocation strategies:
- **Per-prefix**: One LFIB entry per VPN route. 1,000 VRFs × 1,000 routes = **1,000,000 LFIB entries**. No IP lookup needed at egress — the label identifies the exact VRF+prefix combination.
- **Per-VRF (per-CE or per-VRF)**: One label per VRF. 1,000 VRFs = **1,000 LFIB entries**. After the VPN label is popped, the egress PE must do an IP FIB lookup in the VRF to find the correct next-hop.

Per-VRF is memory-efficient but requires an extra IP lookup. Per-prefix is memory-intensive but enables MPLS-based forwarding without IP lookup. Most modern platforms support per-prefix by default.

---

### A5 — Answer: **A**

**Explanation:** Inter-AS Option A (VRF-to-VRF) treats the ASBR-ASBR link as a CE-CE connection. Each customer requires:
- A dedicated VRF on each ASBR
- A dedicated logical/physical sub-interface between the ASBRs for each customer VRF
- PE-CE routing (static, BGP, OSPF) between ASBRs per VRF

This is simple but doesn't scale — N customers = N sub-interfaces and N routing instances between ASBRs. No VPNv4 is exchanged between ASBRs; everything is standard IPv4 in a VRF.

---

### A6 — Answer: **A**

**Explanation:** In Option B, ASBRs exchange **VPNv4 NLRIs** (including VPN labels) via eBGP. The ASBR in AS65000 receives a VPNv4 route with the AS65001 VPN label. It:
1. **Allocates a new local VPN label** for the same VPNv4 route
2. Advertises the route into AS65000 with the **new local label**
3. In its LFIB, maps: receive-new-label → swap to AS65001's label → forward to AS65001 ASBR

Labels are **not preserved across AS boundaries** in Option B — each ASBR allocates its own labels. Transport labels are also regenerated independently within each AS.

---

### A7 — Answer: **B**

**Explanation:** Inter-AS Option C (BGP multihop): 
- PEs or RRs establish **multi-hop MP-eBGP** sessions directly across AS boundaries
- ASBRs exchange labeled IPv4 routes (the PE loopbacks) so transport LSPs can be built between PEs across ASes
- ASBRs do **not** process VPN routes — they only handle the transport label exchange for PE reachability
- VPN routes flow PE-to-PE (or RR-to-RR) directly without VRF processing at ASBRs

This is more scalable than Options A/B (no per-customer ASBR state) but requires multi-hop eBGP and PE loopback reachability across ASes.

---

### A8 — Answer: **A**

**Explanation:** Extranet VPN implementation:
- Shared services VRF exports RT `65000:500`
- Customer A VRF imports RT `65000:500` → Customer A **receives** shared services routes
- Customer A does NOT export `65000:500`, so shared services VRF does NOT receive Customer A's routes (one-way access)
- Customer A's own routes (RT `65000:100`) remain inaccessible to shared services

No changes needed to the shared services VRF — just add an import RT on the customer side. This is the classic "extranet" or "shared services" pattern.

---

### A9 — Answer: **D**

**Explanation:** The most common cause of VPN route not appearing at PE2 is **RT mismatch**: PE1's export RT doesn't match any of PE2's import RTs. Check:
1. `show bgp vpnv4 unicast vrf CUST rd` on PE1 — is the route in PE1's BGP with the right RT?
2. `show bgp vpnv4 unicast` on PE2 (or RR) — does the route arrive at all?
3. `show bgp vpnv4 unicast vrf CUST` on PE2 — is it imported?

If the route exists in PE2's BGP table but not the VRF, the RT import config is wrong. If it's not in BGP at all, check redistribution and BGP session. Option A (redistribute) is for PE1 ingestion, which wouldn't explain a route being in PE1's BGP but not PE2's VRF.

---

### A10 — Answer: **B**

**Explanation:** In CsC (Carrier's Carrier, RFC 4364 Section 10), the backbone SP carries the customer SP's MPLS traffic. The customer SP's PE-to-PE LSP labels are carried as the **inner** label stack, encapsulated within the backbone SP's **outer** label stack. The backbone SP P-routers only see the outer labels. At the backbone SP's egress PE (the PE connected to the customer SP's PE), the outer labels are popped, and the customer SP's MPLS packet (inner stack) is delivered. This enables the customer SP to run MPLS end-to-end across the backbone SP without the backbone SP needing to understand the customer's labels.

---

### A11 — Answer: **B**

**Explanation:** If the CE can't be reconfigured (third-party managed, minimal access), **static routes** on the PE are the practical fallback. The PE has a static route to the CE's subnet(s) and redistributes them into VRF BGP. The CE has a static default pointing to the PE. Static routing requires no CE configuration changes. BGP (option A) would require CE configuration. OSPF (option D) would also require CE-side config for redistribution.

---

### A12 — Answer: **C**

**Explanation:** When PE2 withdraws the VPN route from BGP (due to PE-CE link failure), the ingress PE1 removes the route from its VRF RIB. However, if PE1 has a static or other default route into that VRF, or if BGP next-hop tracking is delayed, traffic can continue hitting PE1 with no valid VRF route → blackhole. The 15-second residual blackhole suggests PE1's CEF/FIB hasn't been fully updated, or there's a default route in the VRF that's absorbing traffic. In practice, this is often a stale CEF entry or VPN FIB synchronization delay.

---

### A13 — Answer: **C**

**Explanation:** The RD's sole purpose is to make overlapping IP prefixes unique in the global BGP VPNv4 table. A VPNv4 route is the combination: `<RD><IP prefix>`. With different RDs:
- `65000:100 / 10.1.0.0/16` → CUST-A's route
- `65000:200 / 10.1.0.0/16` → CUST-B's route

These are distinct NLRIs in BGP. The RT (option A) determines import/export filtering but doesn't distinguish routes within BGP's table. The VPN label (option B) enables forwarding post-import but doesn't help the RR distinguish routes.

---

### A14 — Answer: **B**

**Explanation:** Per RFC 4577 (*OSPF as PE/CE Protocol for BGP/MPLS VPNs*), the MPLS VPN backbone acts as an OSPF "super-backbone" connecting all customer sites. When the source and destination PEs use the same OSPF Domain ID (which is the default), the VPN backbone preserves the original OSPF route type using the OSPF Route Type extended community in BGP:

- **Intra-area routes** (Type 1/2 at the source site) → advertised as **Type 3 (inter-area summary)** at the remote PE. The VPN backbone is "between areas" from the customer OSPF perspective.
- **External routes** (Type 5/7 at the source site) → remain **Type 5 (external)** at the remote site.
- **Inter-area routes** (Type 3 at the source site) → remain **Type 3** at the remote site.

This is NOT simple BGP redistribution — RFC 4577 specifically preserves OSPF semantics. Without sham-links, intra-area routes can never appear as Type 1 at the remote site (option C is wrong). With sham-links, intra-area routes CAN be restored as Type 1.

---

### A15 — Answer: **B**

**Explanation:** Without RT-Constrain, the RR sends **all VPNv4 routes** to every PE — 10,000 VPNs × 1,000 routes = **10,000,000 routes** sent to and processed by every PE. The PE's control plane must parse, filter, and discard 9,950,000 routes that don't match its 50 VRFs. This is a massive waste of CPU, memory, and iBGP bandwidth. RT-Constrain (RFC 4684) solves this: PEs advertise their RT membership; the RR only sends routes with matching RTs.

---

### A16 — Answer: **C**

**Explanation:** Work through the layers: The VPNv4 route exists on PE1, the LSP to PE2 works (traceroute succeeds), PE1 can reach PE2. The failure is in PE2's VRF → CE2. The VPN route is in PE1's BGP (so it was advertised by PE2). Check on PE2:
- `show ip route vrf CUSTOMER 10.10.x.x` — is the CE route in the VRF?
- `show ip bgp vpnv4 vrf CUSTOMER` — is the route imported?
- `show ip route vrf CUSTOMER` — is there a PE2-CE2 route?

A broken PE2-CE2 routing session (option C) means PE2 knows its customer routes exist but they're stale (CE came up but PE-CE BGP/OSPF isn't running). Traffic reaches PE2 but has no way to reach CE2.

---

### A17 — Answer: **B**

**Explanation:** Hub-and-spoke design:
- Hub VRF: `export 65000:HUB`, `import 65000:SPOKE`
- Spoke VRFs: `export 65000:SPOKE`, `import 65000:HUB`

With this config:
- Spokes receive hub routes (import HUB) → can send traffic to hub
- Hub receives spoke routes (import SPOKE) → can send traffic to any spoke
- Spokes do NOT import `65000:SPOKE` → spokes don't receive each other's routes → no direct spoke-to-spoke communication

All spoke-to-spoke traffic must traverse the hub (for firewall inspection). This is the textbook hub-and-spoke RT configuration.

---

### A18 — Answer: **B**

**Explanation:** PHP (Penultimate Hop Popping) removes the **outermost transport label** at the penultimate P router. The egress PE (PE2) receives a packet with only the **VPN label** remaining. PE2 uses the VPN label to:
1. Look up the label in its LFIB → identifies the destination VRF
2. Pop the VPN label
3. Do an IP FIB lookup within the VRF
4. Forward to CE

The VPN label is preserved across the P network — only the transport label is popped at PHP.

---

### A19 — Answer: **C**

**Explanation:** MPLS OAM traceroute works hop-by-hop through the P network using LSP echo requests. If the trace completes through the P network and reaches the egress PE but times out beyond the PE, the MPLS LSP itself is intact. The timeout indicates the PE-CE segment is unreachable or the CE doesn't respond to MPLS echo requests (which is expected — CE typically doesn't run MPLS OAM). The troubleshooting should shift to PE-CE routing and CE reachability.

---

### A20 — Answer: **C**

**Explanation:** BGP-free core is the defining advantage of MPLS L3VPN. P routers only have:
- IGP routes (for LSP next-hop reachability)
- LFIB (MPLS label swap entries for transport LSPs)

They never see VPN routes or VPN labels. A VPN packet arrives at P1 with label stack [transport_label, vpn_label]. P1 does a label lookup on the transport label, swaps it to the next-hop transport label, and forwards. The VPN label (inner label) is completely opaque to P routers — they treat it as "just another label" in the stack. This allows the SP core to remain BGP-free and scale independently of VPN customer count.
