# BGP Deep Dive — Practice Questions

Expert-level SP. 20 questions. Cover sections 3.1–3.4.

---

## Questions

---

### Q1 — BGP Path Selection (Scenario)

A PE router receives the same prefix 10.10.10.0/24 from three iBGP peers:

| Peer | Local-Pref | AS-Path | MED | Next-Hop Reachable? |
|------|-----------|---------|-----|----------------------|
| Peer A | 200 | empty | 0 | Yes (IGP cost 20) |
| Peer B | 200 | empty | 10 | Yes (IGP cost 5) |
| Peer C | 150 | empty | 0 | Yes (IGP cost 5) |

Which path does the router select as best?

A) Peer C — it shares the lowest MED with Peer A but has higher LOCAL_PREF  
B) Peer B — it has the lowest IGP cost to the next-hop after the LOCAL_PREF tie  
C) Peer A — it wins because of the lowest MED after the LOCAL_PREF tie  
D) Peer B — it wins because IGP cost is evaluated before MED in the BGP path selection order  

---

### Q2 — Route Reflector Design (Config Analysis)

```
router bgp 65000
 bgp cluster-id 10.0.0.1
 neighbor 10.1.1.1 remote-as 65000
 neighbor 10.1.1.1 route-reflector-client
 neighbor 10.1.1.2 remote-as 65000
 neighbor 10.1.1.2 route-reflector-client
 neighbor 10.1.1.3 remote-as 65000
```

`10.1.1.1` sends a route to this RR. To which neighbors will the RR reflect this route?

A) Only to `10.1.1.2` (the other RR client)  
B) To `10.1.1.2` and `10.1.1.3`  
C) To no one — RRs only reflect to clients if they have the best path  
D) To `10.1.1.1`, `10.1.1.2`, and `10.1.1.3`  

---

### Q3 — ORIGINATOR_ID Loop Prevention (Scenario)

A network has two route reflectors, RR1 and RR2, with overlapping client sets. Client A sends a prefix to RR1. RR1 reflects it to RR2. RR2 reflects it back toward Client A. What prevents Client A from accepting this reflected copy?

A) CLUSTER_LIST contains RR1's cluster-id; Client A discards it  
B) ORIGINATOR_ID is set to Client A's router-id; Client A discards it  
C) The next-hop will be unreachable after the second reflection  
D) AS_PATH loop detection stops it since the AS number repeats  

---

### Q4 — BGP Community Application (Config Analysis)

```
route-policy SET-COMMUNITY
  if destination in (10.0.0.0/8 le 24) then
    set community (65000:100) additive
  endif
  if destination in (192.168.0.0/16 le 32) then
    set community (65000:200)
  endif
end-policy
```

A route 192.168.1.0/24 with existing community (65000:50) hits this policy. What community does the route carry after the policy applies?

A) `65000:50 65000:200` — both communities retained  
B) `65000:200` — second clause replaces all communities  
C) `65000:50` — policy is a no-op since 192.168.1.0/24 doesn't match /8 le 24  
D) `65000:100 65000:200` — both clauses match; additive applies to first only  

---

### Q5 — BGP Multipath (Design)

An operator wants all equal-cost iBGP paths to a given prefix to be installed in the FIB for load-balancing. The network uses route reflectors. Which condition must be true for BGP multipath to work with RRs?

A) All paths must arrive via the same RR  
B) The next-hops must all be the same  
C) `maximum-paths ibgp` must be configured, and paths must have identical attributes (LP, AS-PATH, MED, ORIGIN)  
D) `add-path` must be enabled end-to-end on all iBGP sessions  

---

### Q6 — BGP Convergence (Troubleshooting)

After a PE-CE link failure, the CE's prefix takes 90 seconds to withdraw from all PE routers. The network runs iBGP with route reflectors. What is the most likely cause?

A) MRAI timer (default 30s) is firing three times before the withdraw propagates  
B) The BGP scanner interval is set to 90 seconds, delaying NHT failure detection  
C) iBGP withdraw MRAI is 90 seconds by default on IOS-XR  
D) The RR holddown timer is 90 seconds  

---

### Q7 — NEXT_HOP Handling (Scenario)

Router R1 (AS 65000) peers with R2 (AS 65001) via eBGP on a point-to-point link. R2 advertises 172.16.0.0/16. R1 learns the route via eBGP and re-advertises into iBGP toward R3 (also AS 65000). R1 does NOT have `next-hop-self` configured. What is the NEXT_HOP attribute value that R3 sees?

A) R1's loopback address (iBGP always sets next-hop-self)  
B) R2's interface IP (iBGP does not modify next-hop by default for eBGP-learned routes)  
C) R3's own loopback (next-hop set to self by the advertising router)  
D) 0.0.0.0 (next-hop cleared for re-advertised routes)  

---

### Q8 — BGP Policy: MED Manipulation (Design)

An SP wants inbound traffic from a transit provider to prefer one of two physical links for prefix 203.0.113.0/24. The SP controls MED. Which approach is correct?

A) Set higher MED on the preferred link — transit provider will choose it  
B) Set lower MED on the preferred link — lower MED is preferred in BGP selection  
C) MED is ignored by default if the two links are to the same AS but different peers — use local-pref instead  
D) Set LOCAL_PREF on the eBGP session toward the preferred link  

---

### Q9 — Selective Route Announcement (Config Analysis)

```
route-policy EXPORT-TO-PEER
  if community matches-any (65000:100) then
    pass
  elseif community matches-any (65000:200) then
    drop
  else
    pass
  endif
end-policy
```

A route with communities `65000:100 65000:200` hits this policy. What happens?

A) It is advertised — the first match (`65000:100`) wins  
B) It is dropped — last match wins in IOS-XR RPL  
C) It is advertised — both communities match; policy error results in default accept  
D) It is dropped — when both match, the more specific community (`65000:200`) wins  

---

### Q10 — BGP Graceful Restart (Scenario)

A PE router running BGP GR restarts. Its neighbor holds routes for the restart timer duration. After the PE re-establishes and sends EOR (End-of-RIB), what happens to stale routes that were NOT re-advertised?

A) They are kept indefinitely until manually cleared  
B) They are removed upon receiving EOR — routes not refreshed in the new session are purged  
C) They remain until a separate stale-path timer expires, independent of EOR  
D) They are refreshed via ROUTE-REFRESH message  

---

### Q11 — AS-PATH Manipulation (Config Analysis)

A customer wants outbound traffic from AS65001 to always prefer Provider A over Provider B. Provider A is upstream of Provider B (i.e., Provider B has a longer AS-PATH to Provider A's prefix). Which action on AS65001's router achieves this?

A) Set LOCAL_PREF higher on routes received from Provider A  
B) Set LOCAL_PREF higher on routes received from Provider B  
C) Prepend AS65001 to routes advertised to Provider A  
D) Set MED lower on routes advertised to Provider A  

---

### Q12 — BGP EVPN Route Type Identification (Scenario)

An operator sees this BGP update in their EVPN overlay:

```
Route Distinguisher: 10.0.0.1:1
NLRI type: 2
MAC address: 00:50:56:ab:cd:ef
IP address: 10.1.1.100
MPLS label 1: 10000
MPLS label 2: 20000
```

What does this route type signal, and what do the two MPLS labels represent?

A) Type 2 MAC/IP route; Label 1 = L2 VNI (bridging), Label 2 = L3 VNI (IRB/routing)  
B) Type 2 MAC/IP route; Label 1 = ingress LSP, Label 2 = egress LSP  
C) Type 2 is an Ethernet Segment route; labels identify the ES and the BGW  
D) Type 2 MAC/IP route; Label 1 = MAC table index, Label 2 = ARP suppression entry  

---

### Q13 — BGP Flowspec (Design)

An SP wants to block traffic destined to 192.0.2.0/24 port 80 from a specific source range 10.0.0.0/8 across all PE routers, without touching each device individually. What is the most scalable approach?

A) Distribute ACLs via NETCONF to each PE  
B) Use BGP Flowspec — encode the filter in a BGP UPDATE (AFI 1, SAFI 133) and push via RR  
C) Use BGP Communities to trigger per-PE policy via route-maps  
D) Use RSVP signaling to install filter states along the path  

---

### Q14 — BGP ADD-PATH (Scenario)

An iBGP network with route reflectors is suffering from suboptimal routing: all clients receive only the RR's best path, missing alternate equal-cost paths. The operator enables `bgp additional-paths send` on the RR and `receive` on clients. What changes?

A) The RR now sends multiple paths per prefix; clients can install all received paths in their RIB  
B) Clients now send all their paths to the RR; RR selects the overall best  
C) The RR sends a ROUTE-REFRESH requesting all paths from clients  
D) ADD-PATH allows clients to override the RR's best-path selection  

---

### Q15 — BGP Confederation (Scenario)

A large SP uses BGP confederations with member AS 65001, 65002, 65003 inside confederation AS 100. A router in 65001 receives a route with AS_PATH `(65002 65003) 200 300`. How does an eBGP peer of the confederation (external to AS 100) see this AS_PATH?

A) `100 200 300` — confederation sub-AS numbers are stripped  
B) `65001 65002 65003 200 300` — all sub-AS numbers preserved  
C) `100 65002 65003 200 300` — only the confedAS is prepended  
D) `200 300` — only external AS hops visible  

---

### Q16 — BGP Optimal Route Reflection (ORR) (Design)

Traditional route reflection causes suboptimal routing because the RR picks its own best path, not the best path from each client's perspective. What does BGP Optimal Route Reflection (ORR) solve, and how?

A) ORR forces all clients to use the same path, eliminating routing asymmetry  
B) ORR makes the RR compute best-path per client using the client's IGP topology view (via BGP-LS), then advertise the client-optimal path  
C) ORR eliminates route reflectors entirely by using the RR only for route storage, not path selection  
D) ORR uses CLUSTER_LIST to prefer closer clusters, automatically picking shorter paths  

---

### Q17 — BGP Bestpath Tie-Breaking (Scenario)

Two iBGP paths to 10.0.0.0/24 are otherwise identical (same LP, AS-PATH length, MED, ORIGIN, weight). Path A's next-hop resolves via OSPF cost 10; Path B's resolves via OSPF cost 10 (same). Path A comes from router-id 10.0.0.1; Path B from 10.0.0.2. What breaks the tie?

A) Path A — lower next-hop IP address  
B) Path A — lower cluster-list length  
C) Path A — lower router-id / BGP peer address  
D) Path B — higher router-id is preferred as a tiebreaker  

---

### Q18 — BGP RT-Constrain (Design)

A PE in a large VPN network participates in 10 of 5,000 VPNs. Without RT-Constrain, how many VPNv4 routes does the PE receive from the RR?

A) Only routes for its 10 VPNs  
B) All 5,000 VPN routes — RT filtering only happens at the RR if RT-Constrain is configured  
C) All routes, but the PE only installs routes matching its VRF import RTs  
D) No VPNv4 routes — the PE must request them via ROUTE-REFRESH  

---

### Q19 — BGP Attributes and Transit (Scenario)

AS65100 is a transit provider. A route enters from AS65200 with MED=50. AS65100 passes the route to AS65300. Should AS65300 see MED=50?

A) Yes — MED is transitive and propagates through transit providers  
B) No — MED is non-transitive; it is reset to 0 or stripped when advertised to eBGP peers  
C) Yes, unless the transit provider explicitly strips it with a route-policy  
D) Only if the transit provider has `bgp bestpath med always-compare` configured  

---

### Q20 — BGP TTL Security (GTSM) (Config/Security)

An SP wants to prevent spoofed BGP sessions from external IPs. They configure GTSM (Generalized TTL Security Mechanism) on all eBGP sessions. Which statement about GTSM is correct?

A) GTSM sets the IP TTL on outgoing BGP packets to 1; neighbors must also set TTL to 1  
B) GTSM sets outgoing TTL to 255 and rejects incoming BGP packets with TTL < 254, preventing spoofed packets that would require multi-hop routing  
C) GTSM uses MD5 authentication on the TCP session to prevent unauthorized BGP opens  
D) GTSM limits BGP session establishment to the configured neighbor's /32 only  

---

## Answers and Explanations

---

### A1 — Answer: **C**

**Explanation:** BGP path selection order:
1. Highest WEIGHT (Cisco only) — all equal
2. Highest LOCAL_PREF — Peers A and B have LP=200; Peer C has LP=150 → **Peer C eliminated**
3. Prefer locally originated routes — not applicable
4. Shortest AS-PATH — all empty, tied
5. Lowest ORIGIN — all same, tied
6. Lowest MED — **Peer A has MED=0, Peer B has MED=10** → **Peer A wins**

Peer A is selected. MED=0 is lower (preferred) than MED=10. IGP cost doesn't factor in because MED already broke the tie between A and B. The common trap (option D) is thinking IGP metric is compared before MED — MED comes first in the standard selection order.

---

### A2 — Answer: **B**

**Explanation:** RR reflection rules:
- Route from a **client** → reflect to all other **clients** AND all **non-clients**
- Route from a **non-client** → reflect to all **clients** only

`10.1.1.1` is a client. Its route is reflected to:
- `10.1.1.2` (other client) ✓
- `10.1.1.3` (non-client) ✓

The RR does NOT reflect back to the originating client (`10.1.1.1`). So: `10.1.1.2` and `10.1.1.3`.

---

### A3 — Answer: **B**

**Explanation:** ORIGINATOR_ID is set by the **first RR** to reflect a route, and is set to the **originating client's router-id**. When Client A receives the reflected copy (RR1 → RR2 → Client A path), it sees ORIGINATOR_ID = its own router-id. BGP discards routes where ORIGINATOR_ID matches the local router-id. This prevents routing loops in multi-RR topologies. CLUSTER_LIST (option A) handles loops between RR clusters, but within a single cluster, ORIGINATOR_ID is the mechanism.

---

### A4 — Answer: **B**

**Explanation:** In IOS-XR RPL (Route Policy Language), a `set community` **without** the `additive` keyword **replaces** all existing communities. The route 192.168.1.0/24 matches the second `if` clause. The policy executes `set community (65000:200)` — this replaces the existing `65000:50`. Result: only `65000:200`. The first clause (`65000:100`) requires a match on 10.0.0.0/8 le 24; 192.168.1.0/24 does not match.

---

### A5 — Answer: **C**

**Explanation:** BGP multipath with iBGP requires:
1. `maximum-paths ibgp N` configured
2. All candidate paths must be "equal" per BGP: same LOCAL_PREF, same AS-PATH length, same ORIGIN, same MED (or MED comparison disabled for multipath)
3. Next-hops must be **different** (otherwise they're the same path)

ADD-PATH (option D) enables the RR to *advertise* multiple paths, but multipath *installation* in the FIB requires `maximum-paths ibgp`. ADD-PATH alone doesn't install multiple paths in the FIB.

---

### A6 — Answer: **B**

**Explanation:** MRAI (Minimum Route Advertisement Interval) applies to UPDATE messages, not WITHDRAW messages. Withdrawals are sent immediately. The 90-second delay points to **BGP Next-Hop Tracking (NHT) / BGP scanner**. The BGP scanner (default ~60s on some platforms, configurable) periodically checks next-hop reachability. If the IGP hasn't converged or NHT isn't event-driven, BGP may not detect next-hop failure until the scanner fires. A 90s scanner interval is the closest match. On modern IOS-XR, NHT is event-driven via `bgp nexthop trigger-delay`, so 90s suggests this is disabled or set high.

---

### A7 — Answer: **B**

**Explanation:** By default, when a router receives a prefix via eBGP and re-advertises it into iBGP, the **NEXT_HOP is not changed** — it remains the eBGP peer's interface IP (R2's IP). This is the standard iBGP behavior per RFC 4271. R3 must be able to resolve R2's IP via IGP or static route for the route to be usable. This is why `next-hop-self` is commonly configured on iBGP sessions — without it, external next-hops can be unreachable from remote iBGP peers.

*Note*: In L3VPN/VPNv4 context (RFC 4364), PEs set next-hop to their own loopback by default for VPNv4 routes. This question is about **global table** iBGP behavior, not VPN.

---

### A8 — Answer: **B**

**Explanation:** MED signals preference to an **adjacent AS** — a *lower* MED is preferred. To steer inbound traffic to a preferred link, advertise a **lower MED** on that link. The upstream provider compares MEDs (if it chooses to, and if they come from the same sub-AS) and prefers the lower value. MED is "inbound traffic engineering" — you're telling the neighbor which path you prefer for inbound. LOCAL_PREF (option D) is for intra-AS outbound selection and isn't sent to eBGP peers.

---

### A9 — Answer: **A**

**Explanation:** IOS-XR RPL evaluates `if/elseif/else` in order; **first match wins**. The route has community `65000:100`, which matches the first `if` clause → `pass`. The `elseif` clause is never evaluated. The route is advertised.

---

### A10 — Answer: **B**

**Explanation:** BGP Graceful Restart (RFC 4724 Section 4.1): When the helper router receives EOR from the restarted peer, it removes any previously marked stale routes that were NOT re-advertised in the new session. Per the RFC: routes "not carried in the new session SHOULD be removed by the receiving speaker at the time of receipt of the End-of-RIB marker."

The **stale-path timer** (e.g., 360s on IOS-XR) is a safety net for when the restarting peer **never sends EOR** — if EOR doesn't arrive within this timer, all stale routes are purged. But when EOR IS received, stale route cleanup happens at that point, not after a subsequent timer.

---

### A11 — Answer: **A**

**Explanation:** For **outbound** traffic selection (choosing which upstream to send traffic to), LOCAL_PREF is the dominant tool. Set higher LOCAL_PREF on routes **received from** Provider A → routes via Provider A are preferred. The AS-PATH manipulation (option C) would make routes *to* AS65001 look longer to Provider A — affecting inbound, not outbound. MED (option D) is not sent to eBGP peers in standard configs.

---

### A12 — Answer: **A**

**Explanation:** EVPN Type 2 (MAC/IP Advertisement Route) carries both a MAC address and optionally an IP address for ARP suppression and host routing. When two MPLS labels are present: **Label 1** is the L2 service label (identifies the EVI/bridge domain — used for bridging), and **Label 2** is the L3 service label (identifies the IP VRF — used for IRB/routing). This enables integrated routing and bridging (IRB) at the PE.

*Terminology note*: The question uses MPLS label notation (labels 10000 and 20000), which is correct for EVPN-MPLS. In EVPN-VXLAN, these would be VNIs (24-bit VXLAN Network Identifiers) rather than MPLS labels, but the semantic distinction (L2 service identifier vs. L3 VRF identifier) is the same in both encapsulations.

---

### A13 — Answer: **B**

**Explanation:** BGP Flowspec (RFC 5575) encodes traffic filters (match on prefix, protocol, port, etc.) as BGP NLRIs and distributes them via the BGP control plane. A single Flowspec route pushed via a route reflector can program filters on all PEs simultaneously without per-device configuration. This is the canonical SP DDoS mitigation technique. NETCONF (option A) would require per-device pushes; Communities (option C) trigger policy but don't encode fine-grained L4 match criteria.

---

### A14 — Answer: **A**

**Explanation:** BGP ADD-PATH (RFC 7911) allows a BGP speaker to advertise multiple paths for the same prefix. With `additional-paths send` on the RR, it sends all its eligible paths (not just the best) to clients. With `receive` on clients, they accept and store multiple paths in their RIB. Clients can then select among received paths locally. This is the solution for path diversity and PIC (Prefix Independent Convergence) in iBGP networks.

---

### A15 — Answer: **A**

**Explanation:** BGP confederation sub-AS numbers (confed-AS segments) are **stripped** when routes are advertised to external eBGP peers outside the confederation. The external peer only sees the confederation AS (100) and the external AS-PATH hops (200, 300). This is defined in RFC 5065. Sub-AS numbers are only meaningful within the confederation.

---

### A16 — Answer: **B**

**Explanation:** ORR (Optimal Route Reflection, RFC 9107) addresses a fundamental RR limitation: the RR selects best-path based on its own IGP position, then advertises that path to all clients. A client far from the RR might have a shorter IGP path to a different next-hop. ORR fixes this by having the RR compute best-path from each *client's* IGP perspective using the full IGP topology (typically distributed via BGP-LS). Each client gets its topologically-optimal path.

---

### A17 — Answer: **C**

**Explanation:** BGP tie-breaking order after IGP metric tie:
1. **Oldest path** (prefer most stable/established path) — this is the default on most platforms (IOS-XR, Junos). The tiebreaker prevents route oscillation by favoring stable paths.
2. Lowest **router-id** (or ORIGINATOR_ID if route-reflected)
3. Lowest **peer address**

In this question, both paths arrive at the same time (exam assumption), so the oldest-path step doesn't break the tie. Next: router-id comparison. Path A from 10.0.0.1; Path B from 10.0.0.2. Lower router-id wins → **Path A**.

*Platform note*: On Cisco, `bgp bestpath compare-routerid` enables explicit router-id comparison. Without it, the oldest path always wins and router-id is never evaluated. The question implicitly assumes simultaneous arrival or `compare-routerid` enabled.

---

### A18 — Answer: **B**

**Explanation:** Without RT-Constrain (RFC 4684), the RR sends **all VPNv4 routes** to every PE, regardless of whether the PE participates in those VPNs. The PE receives and processes all routes but only *installs* routes matching its VRF import RTs into local VRF tables (what option C describes — but C asks about receiving, and the PE does receive all routes). RT-Constrain allows PEs to advertise their RT membership to the RR, so the RR only sends routes with matching RTs — eliminating the wasted control-plane overhead.

---

### A19 — Answer: **B**

**Explanation:** MED (`MULTI_EXIT_DISC`) is a **non-transitive** optional attribute (RFC 4271). When a BGP speaker advertises a route to an eBGP peer, MED is **reset to 0 or not included** by default unless explicitly configured to pass it through. This prevents MED from leaking across multiple ASes. Option C (strip with policy) would only matter if the default was to pass it — but the default is not to pass it.

---

### A20 — Answer: **B**

**Explanation:** GTSM (RFC 5082) sets the outgoing IP TTL to **255** (maximum). The receiver requires incoming BGP packets to have TTL ≥ 254 (meaning they traversed at most 1 hop). A spoofed BGP packet coming from a distant host would arrive with TTL much lower (each hop decrements TTL by 1 from the source's starting value). Since BGP typically runs over directly-connected links (or 1 hop), legitimate packets arrive at TTL 254–255. This prevents remote spoofed BGP SYN floods. GTSM is distinct from MD5 TCP authentication (option C), which authenticates sessions but doesn't prevent connection attempts.
