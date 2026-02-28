# Segment Routing — Practice Questions

Expert-level SP. 20 questions. Covers SR-MPLS, SR-TE, TI-LFA, SRv6. Sections 6.1–6.6.

---

## Questions

---

### Q1 — Prefix-SID Assignment (Config Analysis)

```
router isis 1
 address-family ipv4 unicast
  segment-routing mpls
  !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid absolute 16001
```

The SRGB on all routers is 16000–23999. What label does a remote router use to forward traffic to this router's loopback?

A) 1 (the offset from the base)  
B) 16001 (absolute value as configured)  
C) 16000 (base of SRGB)  
D) Depends on the remote router's SRGB base  

---

### Q2 — Adjacency-SID Behavior (Scenario)

Router R1 has adjacency-SIDs allocated for its links to R2 (Adj-SID 24001) and R3 (Adj-SID 24002). A packet arrives at R1 with label stack [24001, 16005]. What does R1 do?

A) Pop 24001, forward to R2 with label 16005 intact  
B) Swap 24001 for 16005, forward via shortest path to the node with Prefix-SID 16005  
C) Pop both labels and forward IP natively  
D) Swap 24001 for an implicit null label and drop 16005  

---

### Q3 — SR-TE Explicit Path (Config Analysis)

An SR-TE policy is configured with segment list `[16001, 16002, 24003]` where 16001 and 16002 are Prefix-SIDs and 24003 is an Adj-SID. What does the path encoding achieve vs. using only Prefix-SIDs?

A) Nothing — Adj-SIDs and Prefix-SIDs are interchangeable in segment lists  
B) The Adj-SID 24003 forces the packet over a specific link from the node with Prefix-SID 16002, rather than ECMP over any link  
C) The Adj-SID overrides MPLS TTL processing at that hop  
D) Adj-SIDs cannot appear in SR-TE segment lists  

---

### Q4 — TI-LFA (Scenario)

Router P1 has neighbors P2, P3, and P4. Primary path: P1→P2→P3→PE1. P2 fails. P4 is in P1's P-space (reachable from P1 without traversing P2). TI-LFA pre-computes a backup path. The post-convergence path (after P2's failure) is P1→P4→P3→PE1. P1 has an auto-allocated Adj-SID 24010 for its link to P4. What segment list does P1 install in the TI-LFA repair path?

A) Just the Prefix-SID of PE1 — re-route via P4 automatically since P2 is gone  
B) Adj-SID 24010 (P1→P4 link), then Prefix-SID of P3 — forces exit via P4, then normal SPF to PE1  
C) Prefix-SIDs of P4, P3, PE1 in order  
D) Adj-SID of P1→P4, then Adj-SID of P4→P3, then Prefix-SID of PE1  

---

### Q5 — SRGB and SRLB (Design)

An operator needs to use Binding SIDs (BSID) for SR-TE policies. Where should BSIDs be allocated from?

A) From the SRGB (Shared Reserve Global Block) — same as Prefix-SIDs  
B) From the SRLB (Segment Routing Local Block) — BSIDs are local to the router  
C) Dynamically from the MPLS label range above 1,000,000  
D) From the SRGB but with the S-bit (sharing bit) cleared  

---

### Q6 — SR-TE Coloring (Scenario)

Traffic from VRF "Gold" should travel along an SR-TE policy with color 100. The BGP next-hop for the destination is 10.0.0.1. How does the headend router steer traffic into the correct SR-TE policy?

A) Match on destination prefix and apply a static route pointing to the SR policy  
B) BGP extended community `color:100` is carried on the VPNv4 route; the headend matches color 100 + BGP next-hop 10.0.0.1 → selects SR policy with endpoint 10.0.0.1 and color 100  
C) Traffic is steered via QoS DSCP marking — color 100 maps to DSCP EF  
D) The headend uses a Flowspec rule to redirect Gold traffic into the SR-TE tunnel  

---

### Q7 — SRv6 SID Structure (Design)

An operator allocates SRv6 Locator `2001:db8:1::/48`. A specific node function "End.DT4" (L3 VPN decapsulation for IPv4) gets SID `2001:db8:1:0:1::/80`. What do the parts of this SID represent?

A) `/48` = locator (routed to this node), `0:1` = function code (End.DT4 for this VPN), remaining bits = argument  
B) `/48` = AS number, `0:1` = node ID, remaining = service ID  
C) The entire 128-bit SID is a Prefix-SID; the structure is only meaningful to the originating router  
D) `/48` = SRGB base, `0:1` = offset into SRGB  

---

### Q8 — SRv6 vs SR-MPLS (Design Choice)

An SP is considering migrating from SR-MPLS to SRv6. Which statement best describes a key tradeoff?

A) SRv6 requires larger packet headers (IPv6 SRH) but eliminates MPLS label stacks entirely and enables native IPv6 transport and programmability  
B) SRv6 is strictly superior — it requires no header overhead and simplifies data-plane processing  
C) SR-MPLS is required for all SP use cases; SRv6 only works for DC fabrics  
D) SRv6 requires all hardware to support MPLS, same as SR-MPLS  

---

### Q9 — TI-LFA P-Space and Q-Space (Scenario)

For a link failure between R1 and R2, TI-LFA computes:
- P-space of R1 (protecting router): nodes reachable from R1 without using R1→R2
- Q-space of R1 (relative to failure): nodes from which PE1 (protected destination) is reachable without using R1→R2

If the P-space and Q-space do not intersect, what does TI-LFA do?

A) Falls back to RSVP-TE for this destination  
B) Computes a node-protecting path using multiple adjacency SIDs  
C) Uses a repair node in the Extended P-space, encoding additional Prefix-SIDs to "tunnel" to the Q-space boundary  
D) Marks the destination as unprotectable and relies on IGP reconvergence  

---

### Q10 — SR-MPLS PHP and Penultimate Hop Popping (Config Analysis)

An operator sets a node's Prefix-SID with `explicit-null`. What changes vs. default PHP behavior?

A) PHP is suppressed — the penultimate hop swaps to label 3 (implicit-null is default) and the egress pops; explicit-null instead carries label 0, preserving EXP/TC bits to the egress  
B) The penultimate hop sends two copies of the packet — one with label popped, one with label 0  
C) Explicit-null disables SR for that prefix and falls back to LDP  
D) The penultimate hop keeps the Prefix-SID label intact all the way to the egress  

---

### Q11 — SR Policy Candidate Paths (Scenario)

An SR-TE policy has three candidate paths: preference 100 (PCE-computed), preference 50 (explicit segment list), preference 10 (dynamic IGP-based). The PCE path goes down. What happens?

A) The SR policy goes down and traffic blackholes until PCE recovers  
B) Traffic immediately falls back to the next valid candidate path (preference 50)  
C) The dynamic path (preference 10) is always used — explicit paths are only used when dynamic fails  
D) The operator must manually switch to a backup policy  

---

### Q12 — SRv6 End.X Function (Scenario)

An SRv6 SID with function `End.X` is assigned to a specific interface. What does this SID do?

A) It's a cross-connect function — any packet with this SID as active segment is forwarded out the specific interface associated with End.X  
B) End.X is the basic endpoint function — it decrements the SL (Segments Left) counter and advances to the next segment  
C) End.X performs L3VPN decapsulation for IPv6 customers  
D) End.X signals to adjacent routers that the node is reachable via extended SRH  

---

### Q13 — IS-IS SR Distribution (Config Analysis)

```
router isis 1
 address-family ipv4 unicast
  segment-routing mpls
  !
```

A router has this config but no `prefix-sid` under any interface. What SR information does this router advertise in IS-IS?

A) Nothing — SR isn't active without at least one Prefix-SID  
B) Only the SRGB range in IS-IS TLV 242 (Router Capability) — no Prefix-SID TLVs and no Adjacency-SIDs  
C) The SRGB range (TLV 242) AND auto-allocated Adjacency-SIDs for each IS-IS adjacency — but no Prefix-SID TLVs  
D) The router advertises the global SRGB only if `mpls traffic-eng router-id` is also configured  

---

### Q14 — SR-TE PCE Delegation (Scenario)

An SR-TE policy is PCE-initiated. The PCE loses connectivity to the headend. The policy has `pce-delegation-fallback-timer 30`. What happens after 30 seconds?

A) The SR-TE policy is deleted from the headend  
B) The headend retains the last known PCE-provided path until PCE reconnects  
C) The headend falls back to a locally configured candidate path (e.g., dynamic or explicit) if one exists  
D) The headend removes the policy and re-signals via RSVP-TE as a fallback  

---

### Q15 — SR-MPLS Interworking with LDP (Scenario)

An SR-MPLS domain interworks with an LDP domain. An SR-MPLS packet exits the SR domain and enters the LDP domain at the border router. The border router runs both SR and LDP. What does the border router do with the incoming SR label?

A) Drops the packet — SR and LDP are incompatible  
B) Maps the Prefix-SID to the corresponding LDP-allocated label for the same prefix and swaps the label  
C) Removes the MPLS label and sends native IP into the LDP domain  
D) Encapsulates the SR packet in GRE before forwarding into the LDP domain  

---

### Q16 — TI-LFA Scope (Design)

TI-LFA provides which type(s) of protection?

A) Link protection only  
B) Link protection and node protection  
C) Link protection, node protection, and SRLG protection  
D) Node protection only — link failures are handled by fast-reroute  

---

### Q17 — Binding SID in SR-TE (Scenario)

An operator assigns BSID 1000001 to an SR-TE policy on PE1. PE2 wants to steer traffic into this policy without knowing PE1's internal segment list. How does PE2 use the BSID?

A) PE2 sends traffic with just label 1000001; PE1 receives it, matches BSID, and applies the full segment list internally  
B) PE2 must advertise BSID 1000001 in BGP-LS for it to be usable  
C) PE2 can't use a BSID — BSIDs are only locally significant on PE1  
D) PE2 sends a PCEP request to allocate a remote BSID on PE1  

---

### Q18 — SRv6 Compression (uSID) (Design)

Standard SRv6 SIDs are 128 bits. What problem does uSID (Micro-Segment ID) address, and how?

A) uSID packs multiple short (16-bit) segments into a single 128-bit IPv6 DA, reducing SRH overhead and allowing more hops per packet without header extension  
B) uSID compresses SRH by using 32-bit segment IDs instead of 128-bit IDs  
C) uSID eliminates the IPv6 header entirely, using only the SRH for forwarding  
D) uSID reduces the IPv6 header to 40 bytes by removing optional extension headers  

---

### Q19 — SR-TE Disjoint Paths (Scenario)

Two SR-TE policies are configured as a disjoint pair (same disjoint group). The first policy uses path P1→P2→P3→PE1. What constraint applies to the second policy in the same group?

A) The second path must use the same intermediate nodes (for redundancy)  
B) The second path must not share any node (node-disjoint) or link (link-disjoint) with the first, depending on the disjointness level configured  
C) The second policy must use RSVP-TE since SR can't guarantee disjointness  
D) Disjoint paths are only supported with PCE — not with local SR-TE computation  

---

### Q20 — SR Migration Strategy (Scenario)

An operator is migrating from LDP to SR-MPLS on a live network. They want zero-traffic impact during migration. Which strategy is recommended?

A) Disable LDP globally on all routers simultaneously, then enable SR — LDP-SR coexistence isn't needed  
B) Enable SR on all routers first (in LDP/SR coexistence mode), then gradually shift traffic to SR paths, then disable LDP  
C) Replace all routers at once with SR-capable hardware  
D) Use RSVP-TE as a bridge between LDP and SR during migration  

---

## Answers and Explanations

---

### A1 — Answer: **B**

**Explanation:** `prefix-sid absolute 16001` configures the label **directly** as 16001. With an `absolute` Prefix-SID, all routers use label 16001 regardless of their local SRGB base — the label is self-consistent network-wide as long as 16001 falls within every router's SRGB. With `index` (the alternative), the label = SRGB_base + index. Here, absolute is used, so the label is literally 16001.

---

### A2 — Answer: **A**

**Explanation:** An Adjacency-SID is a **local label** that identifies a specific outgoing interface/adjacency. When R1 receives a packet with top label 24001 (its own Adj-SID to R2), it **pops** the top label and forwards the remaining label stack (16005) **out the specific interface toward R2**. The Adj-SID is not swapped — it's consumed (popped) at the local router. The next router R2 receives the packet with only label 16005 and processes it normally.

---

### A3 — Answer: **B**

**Explanation:** Prefix-SIDs route to a destination node using ECMP (the shortest-path set from IS-IS/OSPF). An Adj-SID identifies a **specific interface** on a specific router, pinning the packet to an exact link. By placing an Adj-SID at the end of the segment list (after the Prefix-SID of the source node), you specify which exact link to use for the final hop — useful for link-specific TE, SRLG avoidance, or bandwidth reservation on a specific interface.

---

### A4 — Answer: **B**

**Explanation:** TI-LFA encodes the post-convergence path as a **segment list**. For this scenario:
- P1 must reach P3 (or beyond) via P4 without using P2
- The repair path needs an Adj-SID for P1→P4 (to force the first hop over the backup link), followed by the Prefix-SID of P3 (to route from P4 to P3 and onward normally)

This is the canonical TI-LFA repair: an Adj-SID pinning the first hop, then Prefix-SIDs for the post-convergence path. Just using a Prefix-SID for PE1 (option A) would still route via P2 (which is down).

---

### A5 — Answer: **B**

**Explanation:** BSIDs are allocated from the **SRLB (Segment Routing Local Block)** — a range of labels that are local to the router (typically 15000–15999 or operator-defined). BSIDs are not globally significant; they're locally meaningful only to the headend router that instantiates the SR-TE policy. The SRGB is for globally-routable Prefix-SIDs. Mixing BSIDs into the SRGB would create label collision with Prefix-SIDs.

---

### A6 — Answer: **B**

**Explanation:** BGP Color-Based Traffic Steering: A VPNv4 route can carry an extended community `color:100`. When the headend router receives this route with next-hop 10.0.0.1, it looks for an SR-TE policy with **endpoint=10.0.0.1 and color=100**. If found, traffic is steered into that policy. This is the "On-Demand Nexthop" (ODN) mechanism — the color community triggers automatic policy selection. No static routes or policy-maps needed; the color in BGP drives the steering decision.

---

### A7 — Answer: **A**

**Explanation:** SRv6 SID structure (per RFC 8986):
- **Locator** (`2001:db8:1::/48`): Globally routable prefix — advertised in the IGP to attract packets to this node. Think of it as the "node address."
- **Function** (`0:1` — bits 49–80): Identifies the endpoint behavior. `End.DT4` = decapsulate SRv6 and forward to IPv4 VRF table. Each VRF gets a unique function code.
- **Argument** (remaining bits): Optional, used for additional context (e.g., specific flow or entropy).

The locator is routed; the function is processed locally; arguments are passed to the function.

---

### A8 — Answer: **A**

**Explanation:** Key SR-MPLS vs SRv6 tradeoffs:
- **SRv6 advantage**: Native IPv6, eliminates MPLS labels, richer programmability (SRH allows carrying arbitrary per-hop instructions), easy interop with IPv6-only hardware
- **SRv6 disadvantage**: The SRH adds significant overhead (each segment = 16 bytes; 4 segments = 64 bytes of header). This is real overhead on the wire. Some hardware also has limited SRH processing capability.
- **SR-MPLS advantage**: Compact (4 bytes/label), proven hardware support, easy LDP interop
- **SR-MPLS disadvantage**: Limited programmability, MPLS overhead scales with path depth

Neither is universally superior — the choice depends on hardware capabilities and use case.

---

### A9 — Answer: **C**

**Explanation:** When P-space and Q-space don't intersect, there's no single node from which the destination is reachable without using the protected link AND that P1 can reach without using the failed link. TI-LFA handles this by using the **extended P-space** (nodes reachable from P1's neighbors, excluding the failed link). A node in the extended P-space that's also in Q-space is chosen as a **repair node**. The repair label stack encodes: Adj-SID to reach the neighbor, then Prefix-SID to reach the repair node (tunneling through extended P-space to the Q-space boundary). This is called "extended P-space" repair in TI-LFA (distinct from the older Remote LFA / RFC 7490 mechanism, which TI-LFA supersedes).

---

### A10 — Answer: **A**

**Explanation:** By default, SR uses **implicit-null** (label 3) signaling. The egress PE signals label 3 to its penultimate-hop neighbor in its LDP/SR label mapping. When the penultimate P router receives label 3 as the binding for the final hop, it **pops the top label entirely** (removes it from the stack) — label 3 is a signal to pop, not an actual value imposed in the data plane. The egress PE receives a packet with the remaining inner labels (or bare IP). The MPLS EXP/TC bits are lost when the label is popped, breaking end-to-end QoS marking.

With **explicit-null** (label 0 for IPv4, label 2 for IPv6), the egress PE signals label 0 instead of label 3. The penultimate hop **swaps** the top label to label 0 (instead of popping). The egress PE receives a packet with label 0 on top, processes it as a pop, but crucially the EXP/TC bits were preserved in the label 0 header all the way to the egress. Critical for end-to-end QoS in MPLS networks.

---

### A11 — Answer: **B**

**Explanation:** SR-TE candidate paths have preferences. The **highest-preference** valid (operational) candidate path is used. When the PCE path (preference 100) goes down, the policy falls back to the next valid candidate: preference 50 (explicit). This happens immediately without operator intervention. This is the designed resilience model for SR-TE — multiple candidate paths provide automatic failover. If preference 50 also fails, preference 10 (dynamic) would be used.

---

### A12 — Answer: **A**

**Explanation:** `End.X` is the **Layer 3 Cross-Connect** function in SRv6. A packet arriving with the active segment matching an End.X SID is forwarded out the specific **next-hop interface** associated with that End.X SID (analogous to an SR-MPLS Adjacency-SID). The SL (Segments Left) is decremented, and the next segment becomes active. End.X enables precise link-level steering in SRv6, equivalent to Adj-SIDs in SR-MPLS.

---

### A13 — Answer: **C**

**Explanation:** With `segment-routing mpls` enabled under IS-IS but no Prefix-SIDs configured:
- The router advertises its **SRGB** in IS-IS TLV 242 (Router Capability TLV) — other routers know this router's label range
- The router **auto-allocates Adjacency-SIDs** for each of its IS-IS adjacencies — these appear in IS-IS TLV 22 (Extended IS Reachability) with the Adj-SID sub-TLV
- Without a `prefix-sid` statement, no Prefix-SID TLVs are generated for the loopback

Option B is partially correct (SRGB IS advertised) but incomplete — it omits auto-allocated Adj-SIDs. Option C captures both facts. This intermediate state is common during SR deployment: routers participate in SR topology (with Adj-SIDs for hop-by-hop steering) but aren't yet reachable via globally-routed Prefix-SIDs.

---

### A14 — Answer: **C**

**Explanation:** For PCE-initiated policies, when PCE connectivity is lost:
- The `pce-delegation-fallback-timer` counts down
- After expiry, the headend **takes back control** and uses a locally-configured candidate path (explicit or dynamic) if one exists
- If no fallback candidate path exists, the policy goes down

This prevents traffic blackholes when PCE fails — the headend maintains connectivity using the best available local path until PCE recovers and re-delegates.

---

### A15 — Answer: **B**

**Explanation:** SR-MPLS / LDP interworking uses **label mapping**: the border router, running both SR and LDP, maps SR Prefix-SIDs to LDP FEC labels and vice versa. A packet entering from SR with label 16001 (Prefix-SID for destination X) is mapped to the LDP label for the same prefix (e.g., 17500). The border router swaps 16001→17500 and forwards into the LDP domain. In the reverse direction, LDP labels map to SR Prefix-SIDs. This is defined in RFC 8661 (*Segment Routing MPLS Interworking with LDP*) and is supported on IOS-XR via `segment-routing mpls / sr-prefer` mode.

---

### A16 — Answer: **C**

**Explanation:** TI-LFA (Topology-Independent Loop-Free Alternate) provides:
- **Link protection**: Route around a failed link
- **Node protection**: Route around a failed node (protect against the neighbor going down entirely)
- **SRLG protection**: Route around shared-risk link groups — if multiple links share physical infrastructure (same fiber, same duct), SRLG-aware TI-LFA avoids all of them in the repair path

This coverage — without requiring RSVP-TE or offline path computation — is a key SR-MPLS advantage over traditional IP FRR.

---

### A17 — Answer: **A**

**Explanation:** A Binding SID works as a **tunnel endpoint label**. PE1 advertises BSID 1000001 via BGP-LS or signaling to controllers/headends. PE2 imposes label 1000001 on packets it wants to steer. When PE1 receives a packet with top label 1000001 (matching its local BSID), PE1 **swaps that label for the full segment list** of the associated SR-TE policy and forwards accordingly. The BSID abstracts the internal path from PE2 — PE2 only needs to know the BSID, not the 4-segment label stack inside the policy.

---

### A18 — Answer: **A**

**Explanation:** Standard SRv6 SRH overhead is significant: each segment in the SRH is 16 bytes (full IPv6 address). A 4-node path requires 64 bytes of SRH + 8 bytes SRH header = 72 bytes of overhead per packet. **uSID (Micro-SID)** addresses this by allocating short identifiers (typically 16 bits) within a single 128-bit IPv6 DA. Multiple micro-segments are packed into the destination address itself — no SRH needed for many use cases. The DA is read 16 bits at a time; each hop shifts the DA left by 16 bits. This achieves 8 segments per packet with zero SRH overhead.

---

### A19 — Answer: **B**

**Explanation:** Disjoint path groups (configured via PCE or locally) enforce that two policies in the same group use **non-overlapping resources**. The level of disjointness is configurable:
- **Link-disjoint**: No shared links between the two paths
- **Node-disjoint**: No shared nodes (implies link-disjoint)
- **SRLG-disjoint**: No shared risk groups

If the first policy uses P1→P2→P3→PE1, the second policy must use a path that doesn't share P2, P3, or their connecting links (depending on disjoint level). This is the foundation of path redundancy for critical services.

---

### A20 — Answer: **B**

**Explanation:** The recommended SR migration strategy is the **incremental/coexistence approach**:
1. Enable SR on all routers (SR + LDP coexistence, SR-prefer or LDP-prefer configurable)
2. SR and LDP run simultaneously — no traffic disruption
3. Shift traffic to SR paths incrementally (route-policy, prefer-SR config, or just wait for SR convergence)
4. Verify SR is carrying traffic
5. Disable LDP router by router

This is zero-impact because the network carries traffic via LDP while SR paths are being validated. The simultaneous cutover (option A) would cause momentary disruption during the LDP teardown.
