# Service Provider — Mock Exam 1

**50 questions | Estimated time: 2 hours | All modules covered**

Covers: Foundations, IS-IS, BGP, MPLS, TE, SR, L3VPN, EVPN, Transport, Slicing, Automation, Design.

---

## Instructions

- Commit to your answer before reading the answer section
- Treat config snippets as production-relevant: wrong platform syntax is intentionally included in distractors
- Scoring: 90%+ = exam-ready | 75-89% = targeted review needed | <75% = module review required
- Time yourself — Internet Expert written is 2.5 hours for 90 questions (~1.7 min/question)

---

## Questions

---

### Q1 — IS-IS Metric Types (Foundations)

An IS-IS network uses wide metrics. A router's loopback is reachable via two paths: cost 15 via Path A (3 hops), cost 40 via Path B (2 hops). Which path does IS-IS prefer?

A) Path A — fewest hops  
B) Path B — fewest hops  
C) Path A — lowest cost (15 < 40)  
D) Path B — lowest cost (40 wins with narrow metrics only)  

---

### Q2 — IS-IS L1/L2 Routing (IGP)

A router at the L1/L2 boundary receives an L1-only route for 10.1.1.0/24 from an L1 area. When does this route get advertised into L2?

A) Never — L1 routes cannot leak into L2  
B) Automatically — L1/L2 routers redistribute L1 routes into L2 by default  
C) Only if `leak-route` is explicitly configured  
D) Only if the prefix matches the summary address configured for the L1 area  

---

### Q3 — IS-IS Authentication (IGP / Security)

IS-IS MD5 authentication is configured at the area level. What does this protect?

A) Only Hello packets (neighbor authentication)  
B) LSPs and SNPs but not Hello packets (those use interface-level auth)  
C) LSPs, SNPs, and Hello packets at the area level  
D) Only the SPF computation — MD5 is applied to the LSDB checksum  

---

### Q4 — BGP Local-Pref vs MED (BGP)

AS65001 has two uplinks to AS65000: via PE-A and PE-B. AS65001 uses LOCAL_PREF to prefer PE-A for outbound traffic. AS65000 uses MED to prefer PE-B for inbound traffic. Which statement is true?

A) LOCAL_PREF controls outbound traffic in AS65001; MED influences inbound traffic from AS65000's perspective — they operate independently  
B) MED overrides LOCAL_PREF when the two ASes disagree  
C) LOCAL_PREF is globally significant and affects AS65000's routing decision  
D) MED and LOCAL_PREF cannot both be set simultaneously  

---

### Q5 — MPLS LDP Session (MPLS Fundamentals)

Two routers run LDP. Their LDP sessions use which transport protocol and port?

A) UDP port 646 for discovery; TCP port 646 for session  
B) TCP port 646 for both discovery and session  
C) UDP port 711 for discovery; TCP port 711 for session  
D) LDP uses RSVP signaling — no separate TCP session  

---

### Q6 — MPLS Label Operations (MPLS)

A P router receives a packet with label 17005. Its LFIB entry: incoming label 17005 → swap to 17010, outgoing interface GE0/1. What operation occurs?

A) PHP — label 17005 is popped  
B) PUSH — label 17010 is pushed on top of 17005  
C) SWAP — label 17005 is replaced with 17010, packet forwarded out GE0/1  
D) POP — both labels are removed, IP lookup performed  

---

### Q7 — OSPF in VPN (L3VPN)

PE1 and PE2 both run OSPF in VRF "CORP" with the same OSPF Domain ID. CE1 advertises OSPF intra-area routes (Type 1) for its connected subnets. PE1 carries these via VPNv4 to PE2, which re-advertises into OSPF toward CE2. What OSPF route type does CE2 see for these subnets?

A) OSPF Type 1 (intra-area) — preserved through the VPN backbone  
B) OSPF Type 3 (inter-area summary) — the VPN backbone acts as a super-backbone between OSPF areas  
C) OSPF Type 5 (external) — all VPN routes appear as external  
D) OSPF Type 7 (NSSA external) — VPN routes use NSSA areas  

---

### Q8 — BGP Route Reflector Cluster (BGP)

Two RRs (RR1 and RR2) are in the same cluster. A client sends a route to RR1. RR1 reflects to RR2. What prevents a loop?

A) ORIGINATOR_ID loop prevention  
B) CLUSTER_LIST — RR2 sees the cluster-id in CLUSTER_LIST and discards the route  
C) AS_PATH loop detection  
D) TTL loop prevention in the TCP session  

---

### Q9 — SR-MPLS Forwarding (Segment Routing)

A router has SRGB 16000–23999. A remote Prefix-SID is configured with index 5. What label does a transit router use to forward packets destined to that prefix?

A) 5  
B) 16005  
C) 16000  
D) Depends on whether the transit router knows the remote SRGB  

---

### Q10 — TI-LFA Protection Capability (Segment Routing)

TI-LFA pre-computes backup paths. Which failure types CAN TI-LFA protect against when fully configured?

A) Link failures only  
B) Link and node failures, but not SRLG  
C) Link, node, and SRLG failures  
D) Only remote link failures, not adjacent link failures  

---

### Q11 — RSVP-TE Path Setup (Traffic Engineering)

An RSVP-TE tunnel is established from head-end R1 to tail-end R5. R3 fails. The tunnel has `fast-reroute` enabled. What happens within 50ms?

A) R1 detects the failure via RSVP Hello and re-signals the LSP around R3  
B) The PLR (Point of Local Repair — the router upstream of the failed link) reroutes to a pre-computed bypass tunnel immediately  
C) Traffic blackholes until RSVP RESV teardown and re-establishment  
D) IGP reconverges and RSVP resignals — minimum 1-2 seconds  

---

### Q12 — LDP-IGP Synchronization (MPLS)

LDP-IGP synchronization is enabled. A new IS-IS adjacency forms on a link. What happens to IS-IS metric for that link during LDP session establishment?

A) IS-IS uses the configured metric immediately  
B) IS-IS advertises maximum metric for the link until LDP session is fully established (labels distributed), then switches to configured metric  
C) IS-IS adjacency is delayed until LDP session is up  
D) LDP sync forces IS-IS to prefer OSPF on that link  

---

### Q13 — BGP Graceful Shutdown (BGP / Operations)

An operator wants to drain traffic from a PE before maintenance. Which BGP well-known community achieves this without disrupting the BGP session?

A) `no-export` — prevents routes from being re-advertised  
B) `graceful-shutdown` (community 65535:0) — signals neighbors to prefer alternate paths by setting LOCAL_PREF to 0 on routes with this community  
C) `no-advertise` — stops all advertisement  
D) `blackhole` (65535:666) — triggers remote triggered blackhole  

---

### Q14 — EVPN Type 2 — ARP Suppression (EVPN)

Host A (10.1.1.5) is connected to VTEP-X. Host B (10.1.1.10) is behind VTEP-Y. VTEP-X has already learned Host B's MAC/IP via EVPN. Host A sends an ARP request for 10.1.1.10. What happens?

A) VTEP-X floods the ARP request to all VTEPs in the VNI  
B) VTEP-X proxy-responds with Host B's MAC — ARP stays local  
C) VTEP-X forwards the ARP only to VTEP-Y  
D) VTEP-X drops the ARP — Host A must use default GW for inter-host communication  

---

### Q15 — Coherent Optics — Modulation (Transport)

A 400ZR module uses DP-16QAM modulation. What does increasing to DP-64QAM achieve, at what cost?

A) Higher baud rate with same spectral efficiency  
B) More bits per symbol (higher spectral efficiency/capacity), but requires better OSNR (more noise margin), reducing maximum reach  
C) Lower power consumption with same capacity  
D) Compatibility with older DWDM amplifiers at 1,310nm  

---

### Q16 — YANG Data Modeling (Automation)

A YANG module uses `grouping` and `uses`. What is the purpose of this construct?

A) `grouping` defines a reusable set of schema nodes; `uses` instantiates that grouping at a specific point in the data model — similar to a struct/class being referenced  
B) `grouping` is for read-only data; `uses` makes it writable  
C) `grouping` defines mandatory nodes; `uses` makes them optional  
D) These are deprecated in YANG 1.1 — use `typedef` instead  

---

### Q17 — BGP Flowspec (BGP / Security)

An SP receives a DDoS attack on 203.0.113.0/24, port 443. They want to rate-limit this traffic across 50 PE routers without touching each router. What is the most scalable approach?

A) Configure ACLs on each PE via SSH  
B) Push BGP Flowspec rules via the route reflector — one rule propagates to all PEs automatically  
C) Use RSVP-TE to re-route attack traffic to a scrubbing center  
D) Announce a blackhole community for 203.0.113.0/24  

---

### Q18 — IS-IS Multi-Topology (IGP)

IS-IS MT (Multi-Topology) is required when:

A) Running IS-IS on more than 100 routers  
B) Running IPv4 and IPv6 simultaneously where not all links support both — MT allows separate topologies per address family  
C) Enabling SR-MPLS — SR requires MT for label distribution  
D) Running IS-IS in multiple areas simultaneously  

---

### Q19 — MPLS TTL Propagation (MPLS)

A network operator wants end-to-end traceroute visibility — customers should see every P router hop. What must be configured?

A) `no mpls ip propagate-ttl` — this enables traceroute visibility  
B) `mpls ip propagate-ttl` (or equivalent) — ensures the IP TTL is copied to the MPLS TTL on label imposition, allowing P routers to appear in traceroute  
C) P routers must run BGP for them to appear in traceroute  
D) MPLS traceroute always shows all P routers — no configuration needed  

---

### Q20 — SRv6 Locator Advertising (Segment Routing)

How are SRv6 locators distributed to the network so other routers can forward SRv6 packets to the correct node?

A) SRv6 locators are BGP NLRIs — distributed via BGP only  
B) SRv6 locators are advertised in the IGP (IS-IS or OSPF) as normal reachable prefixes — routers build IP routes to each other's locators  
C) SRv6 uses a dedicated SRv6-IS-IS protocol separate from standard IS-IS  
D) SRv6 locators require manual static routes on every router  

---

### Q21 — BGP Confederation vs Route Reflectors (BGP Design)

An SP needs to eliminate the full-mesh iBGP requirement. What is the main operational advantage of Route Reflectors over Confederations?

A) Route Reflectors provide better loop prevention  
B) Route Reflectors are simpler to deploy — no need to renumber into sub-ASes or change peer configs on all routers; only the RR and its clients need configuration  
C) Confederations are deprecated in modern networks  
D) Route Reflectors support more peers per router  

---

### Q22 — VPLS Split-Horizon (L2VPN)

In a VPLS full-mesh, why is split-horizon applied to pseudowire (PW) traffic?

A) To prevent IP routing loops  
B) To prevent a frame received on one PW from being forwarded to another PW — ensures frames only go toward CE-facing ports, preventing PW-to-PW forwarding loops in the full mesh  
C) Split-horizon limits MAC learning to one PW per MAC  
D) To ensure ARP replies are only sent on the originating PW  

---

### Q23 — 5G Transport — Timing (Network Slicing / 5G)

A 5G fronthaul network requires sub-100ns time synchronization between O-RU and O-DU. Which standard applies?

A) IEEE 1588v2 G.8275.2 (partial timing support, best-effort synchronization)  
B) IEEE 1588v2 G.8275.1 (full on-path support — all network nodes are timing-aware boundary clocks)  
C) SNTP — simple NTP is sufficient for 5G timing  
D) SyncE alone provides sufficient frequency synchronization for fronthaul  

---

### Q24 — NETCONF Candidate Datastore (Automation)

An operator modifies a router's config via NETCONF using the `candidate` datastore. What happens if they send a `<commit/>` followed by detecting an error?

A) The commit is automatically rolled back — NETCONF auto-reverts on error  
B) The commit is permanent. NETCONF doesn't roll back without explicit rollback commands or `commit confirmed`  
C) The candidate datastore is unlocked — the config can be re-edited  
D) NETCONF candidate stores are read-only — commit is impossible  

---

### Q25 — BGP PIC (Prefix Independent Convergence) (BGP)

BGP PIC Edge is enabled. A PE has two BGP paths to a remote VPN prefix: Path A (primary) and Path B (backup). Path A's next-hop fails. How does PIC reduce convergence time?

A) BGP re-runs path selection and installs Path B — takes ~3 seconds  
B) The backup path (Path B) is pre-installed in the FIB as a recursive backup; when Path A's next-hop becomes unreachable, the FIB immediately switches to Path B without waiting for BGP reconvergence  
C) BFD detects Path A failure and notifies BGP within 1ms  
D) PIC forces the PE to use ECMP for all VPN paths, eliminating the concept of primary/backup  

---

### Q26 — OSPF LSA Types (Foundations / IGP)

In OSPF, what type of LSA carries information about ABR-to-area boundaries and summarized inter-area networks?

A) Type 1 — Router LSA  
B) Type 2 — Network LSA  
C) Type 3 — Summary LSA (Network Summary)  
D) Type 5 — AS External LSA  

---

### Q27 — MPLS Penultimate Hop Popping (MPLS)

Why is PHP (Penultimate Hop Popping) used by default in MPLS?

A) It reduces the label stack size for all P routers  
B) It saves the egress PE from performing a double lookup: pop the outer label AND do an IP FIB lookup. With PHP, the penultimate P router pops the label, and the egress PE only does an IP lookup  
C) PHP is required for RSVP-TE to function correctly  
D) PHP allows P routers to perform IP-level QoS on the packet  

---

### Q28 — EVPN DF Election Impact (EVPN)

PE1 is elected DF for EVI 100. CE1 sends an ARP broadcast into EVI 100. Which PE handles forwarding this broadcast to the remote EVPN fabric?

A) Both PE1 and PE2 forward — all-active means both are active for all traffic  
B) PE1 (the DF) floods the BUM to the remote fabric; PE2 does not forward BUM sourced from CE1  
C) Neither PE forwards — BUM is dropped to prevent loops  
D) The RR decides which PE forwards based on BGP path selection  

---

### Q29 — gNMI Subscribe — Streaming Telemetry (Automation)

What is the difference between `SAMPLE` and `ON_CHANGE` subscription modes in gNMI?

A) SAMPLE sends data periodically at a fixed interval; ON_CHANGE sends data only when the value changes  
B) SAMPLE is for counters; ON_CHANGE is for state data  
C) SAMPLE requires the server to push; ON_CHANGE is client-pulled  
D) They are identical — the naming is implementation-specific  

---

### Q30 — SR-TE vs RSVP-TE (Traffic Engineering Design)

What is the primary operational advantage of SR-TE over RSVP-TE for traffic engineering?

A) SR-TE supports more simultaneous tunnels  
B) SR-TE is stateless on transit P routers — no per-tunnel RSVP state maintained at every hop. Only the headend needs to know the path. This dramatically simplifies P router state and eliminates RSVP signaling flood  
C) SR-TE provides better bandwidth guarantees  
D) SR-TE is the only TE mechanism compatible with IPv6  

---

### Q31 — BGP NEXT_HOP Self (BGP)

When is `next-hop-self` typically required in iBGP?

A) When the PE's loopback is not in the IGP (unusual configuration)  
B) When routes are learned via eBGP and redistributed into iBGP — the eBGP next-hop (CE IP) is not reachable from remote iBGP peers, so the PE sets next-hop to its own loopback  
C) next-hop-self is always required on RR sessions  
D) When BGP communities need to be preserved  

---

### Q32 — FlexE (Network Slicing)

FlexE provides what capability that standard 100GE cannot?

A) Longer reach on fiber without amplification  
B) Hard isolation between services — sub-rating a 100GE PHY to carry 40G for Service A and 60G for Service B with guaranteed bandwidth and zero-jitter isolation at the physical layer  
C) Encryption at line rate  
D) IPv6 native forwarding without MPLS  

---

### Q33 — L3VPN — CE Connected to Two VRFs (L3VPN Design)

A CE connects to a PE via two interfaces, each in a different VRF (VRF-A and VRF-B). The CE wants routes from VRF-A to reach VRF-B locally on the PE. What is this called, and how is it achieved?

A) Route leaking — configure `import route-map` between VRFs and redistribute  
B) Route leaking — import the target VRF's route-target into the source VRF so its routes are imported  
C) VRF selection — configure a common VRF for both interfaces  
D) This is not supported — VRFs are completely isolated  

---

### Q34 — IS-IS SPF Optimization (IGP)

IS-IS SPF runs after every topology change. What optimization reduces unnecessary SPF runs in large networks?

A) SPF holddown timers with exponential backoff — rapid sequential changes trigger one delayed SPF rather than individual SPFs per change  
B) IS-IS doesn't run SPF for link-state changes, only for prefix changes  
C) IS-IS uses Dijkstra only for L1; L2 uses distance-vector  
D) IS-IS batches all changes into a 60-second SPF window by default  

---

### Q35 — BGP Extended Community RT Format (BGP / VPN)

A route-target `65000:100` is in which BGP extended community format?

A) AS4-specific: 4-byte AS + 2-byte administrator  
B) AS-specific type 0x0002: 2-byte AS number (65000) + 4-byte local-value (100)  
C) IP-address specific: 4-byte IP (65000 as an IP?) + 2-byte value  
D) Opaque community — format is undefined  

---

### Q36 — EVPN ARP Proxy at Scale (EVPN)

A large data center EVPN fabric has 10,000 VMs. Without ARP suppression, a single ARP broadcast reaches all 10,000 VMs. With EVPN ARP suppression in place, an ARP from VM-A for VM-B (already known via BGP Type 2) reaches how many network nodes?

A) All VTEPs — EVPN still floods to VTEPs even if the ARP can be suppressed  
B) Only the local VTEP — the VTEP answers the ARP locally; no underlay flooding occurs  
C) Only the VTEP behind which VM-B is attached  
D) The gateway/anycast IP, which then responds  

---

### Q37 — OSPF-TE Extensions (Traffic Engineering)

OSPF-TE (RFC 3630) extends OSPF to carry what information used by MPLS-TE path computation?

A) BGP next-hop information for inter-domain TE  
B) Link attributes: maximum bandwidth, unreserved bandwidth, TE metric, SRLG membership, and admin groups — used by CSPF (Constrained Shortest Path First) for TE path selection  
C) Only the MPLS label bindings for each TE tunnel  
D) VPN route information for L3VPN TE  

---

### Q38 — gRPC vs NETCONF (Automation)

Which statement best distinguishes gRPC/gNMI from NETCONF?

A) gRPC uses XML encoding; NETCONF uses JSON  
B) gNMI/gRPC uses Protocol Buffers over HTTP/2 (binary, efficient, streaming-native); NETCONF uses XML over SSH (text-based, request-response model, not inherently streaming)  
C) NETCONF supports push telemetry; gRPC does not  
D) gRPC requires a BGP session for transport  

---

### Q39 — RSVP-TE Bandwidth Reservation (Traffic Engineering)

An RSVP-TE tunnel requests 1Gbps on a 10Gbps link with 3Gbps already reserved. The RSVP PATH message arrives at the link. What determines if the reservation succeeds?

A) The head-end checks if 1Gbps is available before sending PATH  
B) The CSPF algorithm at the head-end verifies available bandwidth before computing the path — PATH is only sent if bandwidth is available. The receiving router admits the reservation only if local bandwidth is available (admission control)  
C) Bandwidth reservations in RSVP-TE are best-effort — the tunnel always establishes regardless of available bandwidth  
D) RSVP does not reserve bandwidth on a per-link basis — it only signals label bindings  

---

### Q40 — EVPN DCI — Site of Origin (EVPN / DCI)

Two data centers run EVPN-MPLS. A route originates in DC1 and is re-advertised by the BGW into DC2. DC2 then attempts to re-advertise the route back into DC1. What BGP attribute prevents this re-advertisement loop?

A) AS_PATH loop detection — DC1's AS number appears in the path  
B) Site-of-Origin (SoO) extended community — the BGW tags routes with the originating site; the receiving BGW discards routes with its own SoO when it would re-advertise them  
C) ORIGINATOR_ID — used within a single AS only  
D) The route-target prevents re-import  

---

### Q41 — Segment Routing — BSID (SR-TE)

An SR-TE policy on PE1 has BSID 1000001. PE2 is the "ingress" in a CsC topology. PE2 imposes label 1000001. What does PE1 do when it receives this packet?

A) Drops the packet — BSID 1000001 is not a valid MPLS label  
B) Looks up BSID 1000001 in its local LFIB/BSID table, finds the SR-TE policy, and pushes the associated segment list labels — the packet is now forwarded along the SR-TE path  
C) Forwards the packet to the next-hop based on IP routing  
D) Removes BSID 1000001 and performs PHP  

---

### Q42 — BGP UPDATE Rate (BGP Operations)

An operator notices BGP is slow to advertise new routes during a large routing table churn. MRAI (Minimum Route Advertisement Interval) is the likely cause. What is the default MRAI for iBGP?

A) 30 seconds (same as eBGP)  
B) 0 seconds — iBGP has no MRAI by default (updates sent immediately)  
C) 5 seconds  
D) 60 seconds  

---

### Q43 — L3VPN Option B — ASBR Label Allocation (L3VPN / Inter-AS)

In Inter-AS Option B, an ASBR receives a VPNv4 prefix with label 17005 from the remote AS. What label does the ASBR advertise into its own AS?

A) 17005 — labels are preserved across AS boundaries in Option B  
B) A new locally-allocated label — the ASBR allocates its own label and creates an LFIB swap entry  
C) Implicit-null (label 3) — all inter-AS labels are normalized to implicit-null  
D) No label — the ASBR strips labels at AS boundaries  

---

### Q44 — OTN Hierarchy (Transport)

In the OTN (Optical Transport Network) hierarchy, what is the relationship between OTU, ODU, and OPU?

A) OTU is the client signal; ODU is the optical channel; OPU is the frame overhead  
B) OPU (Optical Payload Unit) carries client signal; ODU (Optical Data Unit) adds maintenance/monitoring overhead; OTU (Optical Transport Unit) adds FEC and line coding — they are nested layers  
C) OTU and ODU are identical — OPU adds DWDM wavelength management  
D) ODU is a specific to SONET/SDH; OTU replaces it in modern networks  

---

### Q45 — IS-IS TE Extensions — SRLG (Traffic Engineering)

SRLG (Shared Risk Link Group) information is advertised via IS-IS TE extensions. What does SRLG membership indicate, and why does it matter for TE path computation?

A) SRLGs indicate link capacity — CSPF uses SRLG to calculate bandwidth  
B) SRLGs group links that share a physical failure risk (same fiber, same duct, same conduit). CSPF can compute SRLG-diverse paths — a primary path and a backup that don't share any SRLG values, ensuring true physical diversity  
C) SRLGs are administrative groups that control which tunnels can use a link  
D) SRLG is used for MPLS-TE reoptimization scheduling  

---

### Q46 — BGP-LS (BGP Link-State) (Automation / TE)

What does BGP-LS distribute and who consumes it?

A) BGP-LS distributes BGP routing tables to SDN controllers  
B) BGP-LS distributes IGP topology information (nodes, links, prefixes, TE attributes, SR SIDs) to external consumers like PCEs or SDN controllers that need a complete network graph for path computation  
C) BGP-LS distributes MPLS label bindings across AS boundaries  
D) BGP-LS is the BGP equivalent of OSPF area LSAs  

---

### Q47 — SRv6 End.DT4 vs End.DT6 (Segment Routing / SRv6)

An SRv6 PE has both `End.DT4` and `End.DT6` SIDs for the same VRF. When is each used?

A) End.DT4 for IPv4 customers; End.DT6 for IPv6 customers — they identify the same VRF but for different address families  
B) End.DT4 is the primary; End.DT6 is a backup  
C) End.DT6 is used for L2VPN; End.DT4 for L3VPN  
D) They cannot coexist — one VRF can only have one SRv6 endpoint function  

---

### Q48 — Network Slicing — Flex-Algo (Network Slicing)

Flex-Algorithm 128 is defined with affinity constraint "exclude blue" (blue = links with maintenance status). What does this achieve?

A) Links with the "blue" admin group are removed from Flex-Algo 128's topology — IS-IS computes a separate SPT using only non-blue links. Traffic steered into Flex-Algo 128 avoids maintenance links  
B) Blue links get 10x their normal metric in Flex-Algo 128  
C) Flex-Algo 128 runs a separate Dijkstra using the interface color as the weight  
D) The "blue" admin group is reserved for management; Flex-Algo can't reference it  

---

### Q49 — BGP Bestpath Multipath ECMP (BGP Design)

An iBGP PE receives three paths to 10.0.0.0/24: Path A (next-hop 10.1.1.1, cost 10), Path B (next-hop 10.1.1.2, cost 10), Path C (next-hop 10.1.1.3, cost 20). `maximum-paths ibgp 4` is configured. All paths have identical attributes. Which paths are installed in the FIB?

A) All three — ECMP doesn't consider IGP cost  
B) Path A and Path B only — same IGP cost, different next-hops; Path C is excluded because it has higher IGP cost  
C) Only Path A — lowest next-hop IP wins as tiebreaker before multipath  
D) All three — maximum-paths 4 allows up to 4 paths regardless of IGP cost difference  

---

### Q50 — SP Design — RR Placement (BGP Design / Case Study)

An SP has 500 PEs across 5 geographic regions. They deploy 2 RRs per region (10 RRs total). Which statement about this design is correct?

A) Each PE needs iBGP sessions to all 10 RRs — full mesh among RRs and PEs  
B) Each PE needs iBGP sessions only to its 2 regional RRs. The 10 RRs form a full mesh with each other. RRs reflect routes between regions  
C) Each PE needs sessions to only 1 RR — redundancy is provided by RR failover  
D) RRs must be co-located with ASBRs — remote RR placement is unsupported  

---

## Answers and Explanations

---

### A1 — **C** — Path A (cost 15 < cost 40)

IS-IS selects the path with the **lowest total metric** — this is the fundamental Dijkstra (SPF) criterion. Hop count is irrelevant in IS-IS; wide metrics (up to 16,777,215) are compared as integers. Wide metrics support TE applications and were introduced to replace narrow metrics (max 63 per link, max 1023 total path). Path A (cost 15) wins over Path B (cost 40) regardless of hop count.

---

### A2 — **B** — L1/L2 routers redistribute L1 routes into L2 by default

L1/L2 routers automatically redistribute L1 routes into the L2 backbone. They also inject a default route into L1 areas (so L1-only routers can reach external prefixes). Route leaking (option C) refers to the non-default direction: leaking L2 routes into an L1 area, which requires explicit configuration. Without leaking, L1-only routers only have the default route toward their L1/L2 router.

---

### A3 — **B** — LSPs and SNPs but not Hello packets

IS-IS area-level authentication (`lsp-password` on IOS-XR, `area-password` on IOS) protects **LSPs and SNPs (CSNP/PSNP)** only. Hello packets require **separate interface-level authentication** (`hello-password` on IOS-XR, `isis password` under the interface on IOS). This distinction is critical:

- **Area/domain-level auth**: Protects against LSP injection and LSDB corruption — validates LSPs and SNPs
- **Interface-level auth**: Protects against neighbor spoofing — validates Hello packets
- **Best practice**: Configure BOTH levels. Area-level alone leaves Hello packets unauthenticated, allowing an attacker to form an IS-IS adjacency (but not inject LSPs)

Per ISO 10589 and RFC 5304/5310 (HMAC-MD5 and HMAC-SHA for IS-IS), authentication is applied independently to different PDU types.

---

### A4 — **A** — They operate independently

LOCAL_PREF and MED operate in different scopes:
- **LOCAL_PREF**: Intra-AS attribute, controls outbound path selection within AS65001 — AS65001 routers use LP to pick which uplink to use for outbound traffic
- **MED**: Non-transitive, influences the adjacent AS's path selection for inbound traffic into AS65001

They are independent mechanisms. AS65001 controls its outbound with LP; AS65000 controls its "inbound to AS65001" with MED. Routing can be asymmetric as a result — a common and expected outcome.

---

### A5 — **A** — UDP port 646 for discovery; TCP port 646 for session

LDP uses:
- **UDP port 646** for Link Hello (LDP discovery — multicast 224.0.0.2 on directly connected links) and Targeted Hello (unicast to non-adjacent peers)
- **TCP port 646** for LDP session establishment and label exchange

The TCP session is established between the router with the higher LDP Router-ID (becomes the "active" side). All label bindings, withdrawals, and keepalives flow over the TCP session.

---

### A6 — **C** — SWAP

MPLS label operations at a transit P router:
- **PUSH**: Impose a new label (adds to top of stack) — done at ingress PE
- **SWAP**: Replace current top label with a new label — transit P router operation
- **POP**: Remove top label — done at egress PE or penultimate P router (PHP)

The router swaps incoming label 17005 → outgoing label 17010, forwarding out GE0/1. This is standard LSR (Label Switch Router) behavior.

---

### A7 — **B** — OSPF Type 3 (inter-area summary)

Per RFC 4577 (*OSPF as PE/CE Protocol for BGP/MPLS VPNs*), the MPLS VPN backbone acts as an OSPF "super-backbone." When both PEs share the same OSPF Domain ID (which is the default), the VPN backbone preserves OSPF route types using BGP extended communities:

- **Intra-area routes** (Type 1/2 at source) → **Type 3 (inter-area summary)** at the remote PE. The VPN backbone is conceptually "between areas."
- **External routes** (Type 5/7 at source) → remain **Type 5** at the remote site.

Since CE1's routes are intra-area (connected subnets, OSPF Type 1), PE2 advertises them to CE2 as **Type 3 inter-area summaries**. This is NOT simple BGP-to-OSPF redistribution (which WOULD produce Type 5) — RFC 4577 explicitly preserves OSPF semantics.

Sham-links can further elevate these to Type 1 (intra-area) by creating a virtual adjacency between PEs within the same OSPF area.

---

### A8 — **B** — CLUSTER_LIST loop prevention

Within a cluster, ORIGINATOR_ID prevents the original client from accepting its own reflected route. Between clusters, **CLUSTER_LIST** is the loop prevention: each RR appends its cluster-id when reflecting. When RR2 receives a route reflected by RR1, it checks CLUSTER_LIST — if its own cluster-id is present, it discards the route. This prevents inter-cluster loops in hierarchical RR designs.

---

### A9 — **B** — 16005

With SRGB base = 16000 and Prefix-SID index = 5:
**Label = SRGB_base + index = 16000 + 5 = 16005**

The index-based Prefix-SID is globally consistent: every router in the SR domain with SRGB 16000–23999 uses label 16005 to reach this destination. The Prefix-SID index 5 is added to the local SRGB base: 16000 + 5 = 16005. With a uniform SRGB across the network (16000–23999 on all routers), every router uses the same label value (16005) for this destination. If routers had different SRGB bases, they would compute different labels — this is why uniform SRGB is recommended in SR deployments.

---

### A10 — **C** — Link, node, and SRLG failures

TI-LFA (Topology Independent Loop-Free Alternate) can protect against all three failure types when fully configured:
- **Link protection**: Backup around a failed link — enabled with basic TI-LFA config
- **Node protection**: Backup around a failed node — requires `tiebreaker node-protecting` on IOS-XR
- **SRLG protection**: Backup avoiding shared-risk groups — requires `tiebreaker srlg-disjoint` + SRLG definitions

*Platform note*: On IOS-XR, basic `fast-reroute per-prefix ti-lfa` provides **link protection only**. Node and SRLG protection require additional tiebreaker configuration. The question asks about TI-LFA's full capability, not the default-enabled scope.

Repair paths are encoded as SR segment lists computed offline using SPF on the post-convergence topology.

---

### A11 — **B** — PLR reroutes via bypass tunnel immediately

RSVP-TE Fast Reroute (RFC 4090):
- The **PLR (Point of Local Repair)** is the router directly upstream of the failure
- PLR pre-establishes a bypass tunnel around the protected link/node
- When the PLR detects the failure, it immediately reroutes traffic onto the bypass tunnel
- The head-end (R1) simultaneously re-signals the primary LSP (this takes seconds)
- During re-signaling, traffic flows via the bypass tunnel

**50ms caveat**: Sub-50ms restoration requires **BFD** for failure detection (BFD link detection can be <10ms with aggressive timers). Without BFD, RSVP Hello timeout defaults to ~3×3s = 9 seconds. The question specifies "within 50ms" — this is achievable with BFD enabled. With RSVP Hello only, convergence is seconds, not 50ms.

FRR provides ~50ms restoration *with BFD*; without BFD, detection alone takes seconds.

---

### A12 — **B** — Maximum metric until LDP is up

LDP-IGP synchronization (RFC 5443) prevents routing traffic onto a link before MPLS is ready:
- When an IS-IS adjacency forms, IS-IS immediately advertises the link at **maximum metric** (2^24-2 for wide metrics)
- Traffic is steered away from this link by IGP
- When LDP session is established and labels are exchanged, IS-IS reverts to the configured metric
- Traffic flows on the link with full MPLS support

Without this, traffic could be forwarded to a label-switched hop that hasn't yet established LDP, causing a black hole.

---

### A13 — **B** — Community `graceful-shutdown` (65535:0)

BGP Graceful Shutdown (RFC 8326):
- Operator tags routes on the draining PE with community `GRACEFUL_SHUTDOWN` (65535:0)
- Receiving peers are configured to set LOCAL_PREF=0 for routes with this community
- BGP re-runs path selection — the draining PE's routes lose to all alternatives
- Traffic migrates to alternate paths before maintenance begins
- The BGP session remains up throughout — no disruption, just traffic drain

This is superior to session teardown because it's pre-emptive, graceful, and reversible.

---

### A14 — **B** — VTEP-X proxy-responds with Host B's MAC

EVPN ARP suppression (proxy ARP at VTEP):
- VTEP-X receives the ARP request locally from Host A
- VTEP-X checks its EVPN ARP cache (populated by BGP Type 2 MAC/IP routes)
- Host B's MAC was learned via Type 2 from VTEP-Y
- VTEP-X generates an ARP reply on behalf of Host B, sending it back to Host A locally
- The ARP never traverses the VXLAN underlay — VTEP-Y doesn't even see the ARP

This is one of EVPN's biggest operational wins: ARP flooding (historically a major BUM traffic contributor) is eliminated.

---

### A15 — **B** — Higher bits per symbol, shorter reach

Optical modulation tradeoff:
- **DP-16QAM**: 8 bits per symbol (4 bits per polarization × 2 polarizations) — moderate spectral efficiency, moderate reach (~1,000 km for standard coherent)
- **DP-64QAM**: 12 bits per symbol (6 bits per polarization × 2 polarizations) — 50% more capacity at same baud rate, but requires ~6dB better OSNR
- Higher order modulation is more sensitive to noise — maximum reach decreases significantly
- PCS (Probabilistic Constellation Shaping) can partially recover reach with higher-order modulation
- 400ZR uses DP-16QAM at 400G for ~100km reach; DP-64QAM would require better amplifiers/shorter spans

---

### A16 — **A** — `grouping` = reusable schema; `uses` = instantiate

YANG `grouping` defines a reusable schema fragment (a set of nodes, like a template). `uses` references a grouping and instantiates its nodes at the current position in the schema. Example:
```yang
grouping address {
  leaf ip { type inet:ipv4-address; }
  leaf prefix-length { type uint8; }
}
container interface {
  uses address;  // instantiates ip + prefix-length here
}
```
This prevents schema duplication and is fundamental to YANG model design.

---

### A17 — **B** — BGP Flowspec via RR

BGP Flowspec (RFC 5575) encodes traffic match criteria (destination prefix + L4 match + rate-limit action) as BGP NLRIs:
- SAFI 133 (ipv4 flowspec) — the "filter rules" are BGP routes
- Distributed via the route reflector like any BGP route
- All 50 PEs receive the Flowspec UPDATE and install the filter in hardware
- Operator pushes one rule to the RR → all PEs react within BGP convergence time

This is the canonical SP DDoS mitigation / traffic steering tool. Much faster than SSH-to-50-PEs and removes human error.

---

### A18 — **B** — Not all links support both IPv4 and IPv6

IS-IS Multi-Topology (MT-IS-IS, RFC 5120) runs separate SPF instances for different topologies. Without MT, IS-IS has a single topology — a link either exists or doesn't for all address families. MT is needed when:
- Some links carry IPv4 only, some IPv6 only, some both
- Running IS-IS MT-IPv4 (topology 0) and MT-IPv6 (topology 2) separately allows the SPF for each AF to use only links that support that AF
- Without MT, running IPv4 and IPv6 with different reachable link sets causes routing inconsistency

---

### A19 — **B** — `mpls ip propagate-ttl` (IOS syntax; platform-specific)

*Note: IOS syntax is `mpls ip propagate-ttl` (enabled by default; disable with `no mpls ip propagate-ttl`). IOS-XR syntax is `mpls ip-ttl-propagate disable` to disable (enabled by default). Junos controls this under `no-propagate-ttl` stanza. The concept is universal; the command varies by platform.*

Without TTL propagation:
- IP TTL is set to 255 at MPLS label imposition (ingress PE)
- P routers decrement the MPLS TTL (not the IP TTL)
- At egress PE, MPLS TTL is copied back to IP TTL only if propagation is enabled
- Result: Traceroute shows no P router hops — they're invisible

With `mpls ip propagate-ttl`:
- IP TTL is copied to MPLS TTL at imposition
- P routers decrement MPLS TTL; when it reaches 1, they send ICMP TTL-exceeded
- P routers appear in traceroute

SPs often **disable** TTL propagation (use `no mpls ip propagate-ttl`) to **hide** their core topology from customers.

---

### A20 — **B** — Advertised in the IGP as reachable prefixes

SRv6 locators are standard IPv6 prefixes. They are:
1. Configured on the router's loopback or special interface
2. Redistributed into the IGP (IS-IS or OSPF) as reachable IPv6 prefixes
3. Other routers build IPv6 routes to each other's locators via the IGP

When a packet arrives at a router with the active SID matching the router's locator, the router processes it locally (applies the function). If the locator doesn't match locally, the router forwards the IPv6 packet toward the locator's IGP next-hop. This is the elegant simplicity of SRv6 — it's just IPv6 routing.

---

### A21 — **B** — RRs are operationally simpler

Route Reflectors vs Confederations:
- **RR**: Only the RR's configuration changes (add clients). Client routers may not even know they're behind an RR. No AS renumbering. Simpler.
- **Confederations**: Every router must be renumbered into a sub-AS. All iBGP configs must be updated. EBGP semantics apply within the confederation. Much more complex to deploy on a live network.
- **Both** eliminate the full-mesh requirement
- RRs are vastly more common in production SP networks. Confederations are rare — mostly academic or legacy.

---

### A22 — **B** — Prevents PW-to-PW forwarding loops

VPLS split-horizon rule: **A frame received on a pseudowire is never forwarded to another pseudowire.** It can only be forwarded to CE-facing interfaces. This prevents the classic bridging loop that would occur if PE1 sent a flooded frame to PE2 via PW, and PE2 then forwarded it to PE3 via another PW, and PE3 to PE1, etc. The full-mesh of PWs requires split-horizon to prevent this loop.

---

### A23 — **B** — G.8275.1 (full on-path support)

5G timing requirements:
- **Fronthaul** (O-RU to O-DU): Sub-100ns synchronization required (Option 7-2x split)
- **G.8275.1**: All network nodes are timing-aware (BC or T-GM). Achieves sub-100ns end-to-end. Required for fronthaul.
- **G.8275.2**: Partial timing support — only some nodes are timing-aware. Maximum accuracy ~1µs — suitable for midhaul/backhaul but not fronthaul.
- **SyncE** alone provides frequency synchronization but not phase/time synchronization — insufficient for fronthaul.

---

### A24 — **B** — Commit is permanent without explicit rollback

NETCONF candidate/running datastore behavior:
- `<edit-config target="candidate">` modifies the candidate datastore (not running)
- `<commit/>` applies candidate to running — this is **permanent**
- NETCONF does not auto-rollback on error after commit
- Use `<commit><confirmed/>` with `<confirm-timeout>` for automatic rollback if no `<commit/>` is sent within the timeout
- `<discard-changes/>` resets candidate to running (before commit)

Always use `commit confirmed` for risky changes with a reasonable timeout (e.g., 300 seconds).

---

### A25 — **B** — Pre-installed backup FIB entry

BGP PIC (Prefix Independent Convergence) works by:
1. When two paths exist, **both** are computed during BGP path selection
2. Path A (primary) is installed as the main FIB entry
3. Path B (backup) is pre-installed as a **recursive backup** in the FIB — it's ready but not actively used
4. When Path A's next-hop becomes unreachable (detected by FIB-level NHT or BFD):
   - The FIB **atomically switches** to Path B — no BGP reconvergence required
   - Convergence = FIB update time (~milliseconds), not BGP processing time (~seconds)

Without PIC, BGP would need to re-run path selection after detecting the failure — takes 1-3+ seconds at scale.

---

### A26 — **C** — Type 3 (Summary LSA)

OSPF LSA types:
- **Type 1** (Router LSA): Links within an area — generated by every router
- **Type 2** (Network LSA): Multi-access network segment — generated by DR
- **Type 3** (Summary LSA): Inter-area prefix advertisements — generated by ABRs
- **Type 4** (ASBR Summary): Points to an ASBR in another area
- **Type 5** (AS External): External routes (from ASBR redistribution)
- **Type 7** (NSSA External): External routes within an NSSA area

ABRs generate Type 3 LSAs to advertise routes from one area into another, enabling inter-area routing.

---

### A27 — **B** — Saves egress PE from double lookup

PHP exists for efficiency at the egress PE:
- Without PHP: P-router swaps labels to egress PE. Egress PE receives packet with MPLS label. Egress PE must: (1) pop MPLS label, (2) IP FIB lookup in VRF or global table. Two operations.
- With PHP: Penultimate P-router pops the outer label, sends IP (or inner MPLS stack) to egress PE. Egress PE receives IP packet or inner-label stack. Only one lookup needed.

This was a meaningful optimization on older hardware. Modern ASICs handle double lookups efficiently, but PHP remains the default for backward compatibility.

---

### A28 — **B** — PE1 (DF) floods BUM; PE2 does not

DF election controls BUM forwarding **toward the CE**. For BUM traffic sourced **from** CE1 and going into the EVPN fabric:
- The DF (PE1 for EVI 100) is responsible for flooding BUM to the EVPN fabric
- PE2 (non-DF) receives the same BUM from CE1 but **drops it** — only DF forwards
- This prevents duplicate BUM delivery to remote VTEPs (which would receive it from both PE1 and PE2 otherwise)

For **known unicast** traffic (in all-active mode), both PE1 and PE2 can forward simultaneously.

---

### A29 — **A** — SAMPLE = periodic; ON_CHANGE = event-driven

gNMI subscription modes:
- **SAMPLE**: Subscribe to a path; the server sends the current value every `sample_interval` regardless of changes. Good for counters, statistics, periodic monitoring.
- **ON_CHANGE**: Server sends an update only when the value changes. Good for state data (interface status, BGP state), low-frequency events. Reduces bandwidth vs SAMPLE for slowly-changing data.
- **ONCE**: Get current value once, subscription closes — equivalent to a Get operation.

ON_CHANGE is event-driven and more bandwidth-efficient for sparse updates; SAMPLE is predictable but potentially wasteful.

---

### A30 — **B** — Stateless transit routers

SR-TE vs RSVP-TE fundamental difference:
- **RSVP-TE**: Every P router maintains per-tunnel RSVP state (PATH state, RESV state). 10,000 tunnels = 10,000 state entries per P router. Signaling complexity, refresh floods, scalability limits.
- **SR-TE**: P routers are **stateless**. They only perform MPLS label operations (swap). No per-tunnel state. The headend encodes the full path as a label stack. Only the headend needs to know the path. Massively more scalable.

SR-TE also eliminates RSVP signaling storms during IGP reconvergence — a major RSVP-TE operational headache.

---

### A31 — **B** — When eBGP next-hop is unreachable from iBGP peers

Standard iBGP rule: next-hop is not modified (RFC 4271). But when a PE learns a route via eBGP from a CE, the next-hop is the CE's IP. Remote iBGP peers can't reach the CE IP directly (it's behind the local PE). Solution: `next-hop-self` — the PE sets next-hop to its own loopback in iBGP updates. Remote PEs then route to the local PE (reachable via IGP), and the local PE does the CE forwarding. This is standard PE configuration for Internet edge and VPN services.

---

### A32 — **B** — Hard bandwidth isolation at the physical layer

FlexE (Flexible Ethernet, OIF spec):
- Standard 100GE carries all traffic as one shared channel — no hard isolation
- FlexE creates a **calendar slot** mechanism at the PHY layer (OIF FlexE shim between MAC and PCS)
- Each slot (minimum 5G, 25G, 50G, or 100G in FlexE 3.0) is exclusively allocated to a client
- Clients (sub-rate channels) have **guaranteed bandwidth** and are **isolated** from each other — congestion in one client cannot impact another
- This is "hard slicing" vs "soft slicing" (QoS) — QoS can be preempted under congestion; FlexE cannot

---

### A33 — **B** — Route leaking via RT import

VRF-to-VRF route leaking on the same PE:
- VRF-A exports RT `65000:A`
- VRF-B exports RT `65000:B`
- To let VRF-B receive VRF-A's routes: add `route-target import 65000:A` to VRF-B
- To let VRF-A receive VRF-B's routes: add `route-target import 65000:B` to VRF-A

This works because the PE processes RT-based import on locally instantiated VRFs. The routes don't need to traverse the MPLS core — the PE imports them locally. Used for shared services VPNs and extranet connectivity.

---

### A34 — **A** — SPF holddown with exponential backoff

IS-IS SPF optimization (3-timer model):
- **Initial delay** (e.g., 50ms): Wait briefly after first topology change before running SPF (in case more changes arrive)
- **Hold interval**: Minimum time between SPF runs — starts short, doubles with each successive trigger (exponential backoff)
- **Maximum hold**: Cap on the exponential backoff (e.g., 5-10 seconds)

During rapid link flaps, this prevents CPU-intensive SPF from running continuously. The exponential backoff adapts to instability — flapping networks trigger slower SPF runs, stable networks respond quickly.

---

### A35 — **B** — AS-specific type 0x0002 (2-byte AS + 4-byte value)

BGP Extended Community formats (RFC 4360):
- **Type 0x0002**: AS-specific — 2-byte Type + 2-byte global admin (AS number, up to 65535) + 4-byte local admin (value)
- `65000:100` = AS 65000, local value 100
- **Type 0x0102**: AS4-specific (for 4-byte AS numbers) — 2-byte Type + 4-byte AS + 2-byte value
- **Type 0x0102** would be needed for 4-byte AS like 131072:100

Since 65000 fits in 2 bytes, it uses Type 0x0002. This is the most common RT format in production networks.

---

### A36 — **B** — Only the local VTEP

With full ARP suppression active:
- VTEP has VM-B's MAC/IP in its local EVPN ARP table (learned from BGP Type 2)
- VM-A's ARP stays at VTEP-A — answered locally
- Zero underlay packets generated for this ARP
- VTEP-B never sees the ARP request
- Network nodes involved: 1 (local VTEP)

Without ARP suppression: 10,000 hosts × ARP storm = potentially millions of ARP packets across the underlay per minute. ARP suppression is mission-critical at data center scale.

---

### A37 — **B** — Link TE attributes for CSPF

OSPF-TE (RFC 3630) adds Opaque LSAs (Type 10) carrying:
- **Maximum bandwidth**: Physical link capacity
- **Maximum reservable bandwidth**: How much is available for RSVP-TE
- **Unreserved bandwidth**: Per-CT (class-type) available bandwidth
- **TE metric**: Separate metric for TE path computation (independent of OSPF metric)
- **Admin groups** (affinity): Color bits for include/exclude constraints
- **SRLG**: Shared Risk Link Group membership

CSPF uses all these attributes to find a path satisfying tunnel constraints (bandwidth, affinity, SRLG avoidance).

---

### A38 — **B** — gRPC/protobufs vs XML/SSH

Key protocol differences:
| | gNMI/gRPC | NETCONF |
|--|--|--|
| Transport | HTTP/2 (TLS) | SSH |
| Encoding | Protocol Buffers (binary) | XML (text) |
| Model | Streaming-native, bidirectional | Request-response |
| Efficiency | ~10-100x more compact than XML | Verbose |
| Telemetry | Native (SUBSCRIBE mode) | Not native |

gRPC/gNMI is the modern choice for telemetry and high-frequency operations. NETCONF remains standard for configuration management (especially with YANG models), where verbosity is acceptable.

---

### A39 — **B** — CSPF at headend + admission control at each hop

RSVP-TE bandwidth reservation process:
1. **Headend CSPF**: Computes a path with sufficient bandwidth (checks TE-DB which tracks reserved bandwidth per link)
2. **PATH message**: Sent along the computed path with bandwidth request in SENDER_TSPEC
3. **Admission control** at each hop: Each router checks if it can admit the requested bandwidth. If yes, reserves it and forwards PATH. If no, rejects with PathErr.
4. **RESV message**: Flows backward from tail-end to head-end, confirming reservation
5. Labels allocated during RESV flow

Without CSPF or with stale TE-DB, PATH could be sent over a link without available bandwidth — admission control at the link is the final safety check.

---

### A40 — **B** — Site-of-Origin (SoO) extended community

SoO prevents routing loops in multi-homed VPN topologies and DCI:
- When a BGW originates or re-originates a route, it tags it with an SoO community identifying the originating site (e.g., `soo:65000:1` for DC1)
- When the route arrives at DC2's BGW, DC2 re-originates it into its fabric
- If DC2's BGW receives the route back (loop), it checks: does the SoO match my DC2 site? No → might re-advertise (loop). Do I see DC1's SoO on a route coming back to DC1? Yes → discard.
- BGW configured to not re-advertise routes with its own (DC1's) SoO back into DC1

SoO is also used in CE multi-homing to prevent "tromboning."

---

### A41 — **B** — PE1 looks up BSID, applies segment list

Binding SID operation:
1. PE2 imposes label 1000001 (the BSID for PE1's SR-TE policy)
2. PE1 receives packet with top label 1000001
3. PE1 looks up 1000001 in its BSID/LFIB table → finds SR-TE policy P1
4. PE1 **pops** BSID label and **pushes** the full segment list from policy P1 (e.g., [16001, 16002, 24003])
5. Packet is now forwarded along the SR-TE path

BSID abstracts the path from external headends — PE2 doesn't need to know PE1's internal segment list. This is the foundation of hierarchical SR-TE and inter-domain TE.

---

### A42 — **B** — iBGP MRAI = 0 seconds (vendor default)

MRAI (Minimum Route Advertisement Interval) prevents route flap amplification:
- **eBGP default MRAI**: 30 seconds — limits how fast BGP advertises updates to external peers
- **iBGP default MRAI**: **0 seconds** — iBGP updates are sent immediately without delay (per major vendor implementations)

*RFC caveat*: RFC 4271 Section 9.2.1.1 recommends 30 seconds as the default MRAI for all sessions, but notes it "MAY be set to 0 seconds for iBGP sessions." All major vendors (Cisco IOS, IOS-XR, Junos) default to 0 for iBGP in practice. The Internet Expert exam tests the practical implementation, not the RFC recommendation. If asked specifically about RFC 4271's recommended value, the answer is 30 seconds for all sessions.

Rationale: iBGP is internal; convergence speed matters more than dampening internal flaps. eBGP MRAI prevents oscillating routes from propagating to the entire internet at 30-second intervals.

---

### A43 — **B** — New locally-allocated label

Inter-AS Option B label behavior:
- ASBR1 (AS65000) receives VPNv4 route from ASBR2 (AS65001) with label 17005
- ASBR1 allocates a **new label** (e.g., 17100) for this VPNv4 route
- ASBR1 advertises the VPNv4 route into AS65000 with label 17100
- ASBR1 LFIB: when it receives a packet with label 17100, swap to 17005 and forward to ASBR2

Labels are not globally significant — they're locally assigned at each router/AS boundary. This is the fundamental MPLS label scoping rule.

---

### A44 — **B** — Nested layers: OPU inside ODU inside OTU

OTN layer hierarchy (outer to inner):
- **OTU** (Optical Transport Unit): Outermost — adds FEC (Reed-Solomon or enhanced FEC) and line coding. Rate variants: OTU1 (2.5G), OTU2 (10G), OTU3 (40G), OTU4 (100G+).
- **ODU** (Optical Data Unit): Adds operations/maintenance overhead, TCM (Tandem Connection Monitoring), delay measurement. Enables service-level monitoring and OAM.
- **OPU** (Optical Payload Unit): Carries the actual client signal (Ethernet, SONET, other). Maps the client into the OTN frame.

OTN frames are 4×4080-byte structure with 3824-byte payload (OPU) + overhead + FEC.

---

### A45 — **B** — Physical failure risk groups for diverse path computation

SRLG purpose and use:
- Links sharing the same physical infrastructure (fiber, duct, conduit, right-of-way, building) are assigned the same SRLG value
- Example: Two fiber cables in the same underground conduit both get SRLG 100 — if the conduit is cut, both fail
- CSPF (Constrained Shortest Path First) can compute **SRLG-diverse paths**: primary and backup paths that share no SRLG values
- Without SRLG awareness, two "geographically diverse" paths might actually share physical infrastructure and both fail under the same backhoe incident

SRLG is critical for true resilience in SP backbone design.

---

### A46 — **B** — IGP topology distribution to PCEs/controllers

BGP-LS (RFC 7752) exports network topology into BGP:
- **AFI 16388, SAFI 71**
- Carries: Node NLRIs (router-id, capabilities), Link NLRIs (metrics, TE attributes, adjacency info), Prefix NLRIs (reachable prefixes)
- Also carries SR SIDs (Prefix-SIDs, Adj-SIDs, SRGBs, SRLBs, SRLG)
- BGP-LS sessions typically go from routers to a route reflector or directly to a PCE/SDN controller

The PCE (Path Computation Element) needs a complete network graph to compute optimal/constrained paths. BGP-LS provides this without requiring the PCE to run IS-IS/OSPF directly.

---

### A47 — **A** — End.DT4 for IPv4 VRF, End.DT6 for IPv6 VRF

SRv6 endpoint functions for L3VPN:
- **End.DT4** (Decapsulate and do Table Lookup for IPv4): Remove SRv6 encapsulation, do IPv4 FIB lookup in the specified VRF table. Used when the payload is IPv4.
- **End.DT6** (Decapsulate and do Table Lookup for IPv6): Same, but for IPv6 payload in VRF.
- **End.DT46**: Handle both IPv4 and IPv6 in the same VRF (newer, avoids needing two SIDs per VRF)

A dual-stack VPN customer would use End.DT46. Pure IPv4 customer = End.DT4. Pure IPv6 = End.DT6. Each VRF+AF combination gets its own SID.

---

### A48 — **A** — Blue links removed from Flex-Algo 128's topology

Flex-Algorithm with affinity constraints:
- Admin groups (affinity bits) are assigned to links (e.g., "blue" bit = link is under maintenance)
- Flex-Algo 128 is defined with `exclude blue` constraint
- IS-IS computes a **separate SPT** for Flex-Algo 128 where blue links are pruned from the topology
- Traffic steered into Flex-Algo 128 (via SR-TE policy or BGP-CT) follows paths that never traverse blue/maintenance links
- Flex-Algo can run multiple independent SPFs with different constraints (latency-optimized, bandwidth-constrained, maintenance-avoiding)

This enables network slicing at the IGP level without requiring PCE or RSVP-TE signaling.

---

### A49 — **B** — Path A and Path B only (same IGP cost)

BGP multipath with `maximum-paths ibgp`:
- Candidate paths must be "equal" for multipath: same LOCAL_PREF, same AS-PATH, same ORIGIN, same MED
- Additionally, paths must have the **same IGP cost** to the next-hop (by default, unless `unequal-cost` or `multipath relax` is configured)
- Path A (cost 10) and Path B (cost 10) → **equal IGP cost** → both installed → 2-path ECMP
- Path C (cost 20) → **higher IGP cost** → excluded from multipath even if all BGP attributes are equal

Result: 2-way ECMP with Path A and B. Path C is the tiebreaker backup (lower router-id would select A or B as best).

---

### A50 — **B** — Each PE to 2 regional RRs; RRs full-mesh with each other

Standard hierarchical RR design:
- **Regional RRs**: Each PE peers only with its 2 regional RRs (iBGP sessions = 2 per PE)
- **Inter-region**: The 10 RRs form a full mesh with each other (45 sessions among RRs) — or hierarchical RR clusters if even larger
- **Route propagation**: PE in Region A → Regional RR A1 → Full mesh to RR B1 → Region B PEs
- **Redundancy**: 2 RRs per region means loss of 1 RR doesn't isolate PEs

Alternative: Hierarchical RRs (super-RRs above regional RRs) for very large networks. 500 PEs × 2 sessions = 1,000 RR-client sessions total — very manageable.

---

## Score Interpretation

**Count your correct answers:**

| Score | Percentage | Assessment |
|-------|-----------|------------|
| 45-50 | 90-100% | **Exam ready** — book the exam |
| 38-44 | 76-88% | **Strong** — review wrong topics, re-test in 2 weeks |
| 30-37 | 60-75% | **Solid base** — targeted module study needed |
| 20-29 | 40-59% | **Build foundation** — back to module reading first |
| <20 | <40% | **Start over** — work through modules 1-12 systematically |

**Review wrong answers by module:**
- Q1, Q3, Q18, Q26, Q34: IGP (Module 2)
- Q4, Q8, Q13, Q21, Q31, Q35, Q42: BGP (Module 3)
- Q5, Q6, Q12, Q19, Q27: MPLS (Module 4)
- Q11, Q37, Q39, Q45: Traffic Engineering (Module 5)
- Q2, Q9, Q10, Q30, Q32, Q41, Q48: Segment Routing (Module 6)
- Q7, Q33, Q43: L3VPN (Module 7)
- Q14, Q22, Q28, Q36, Q40, Q47: L2VPN/EVPN (Module 8)
- Q15, Q44: Transport/Optical (Module 9)
- Q23, Q32, Q48: Network Slicing/5G (Module 10)
- Q16, Q24, Q29, Q38, Q46: Automation (Module 11)
- Q17, Q49, Q50: Design (Module 12)
