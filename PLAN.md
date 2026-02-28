# Service Provider Backbone Study Guide

## Vision
A comprehensive, expert-level training guide for service provider backbone routing and architecture. Structured like CBT Nuggets or INE courses — modular, deep, practical.

## Target Audience
- Network engineers moving into SP roles
- intermediate-level candidates eyeing expert-level certs
- Existing SP engineers wanting structured reference material

---

## Curriculum Outline

### Module 1: Foundations
1.1 SP Network Architecture Overview
1.2 Design Principles (scalability, convergence, fault isolation)
1.3 SP vs Enterprise — Fundamental Differences
1.4 Reference Topologies (regional, national, global backbones)

### Module 2: IGP at Scale
2.1 IS-IS Deep Dive (levels, areas, TLVs, SPF optimization)
2.2 OSPF in SP Networks (areas, stub types, LSA flooding control)
2.3 IGP Convergence Tuning (BFD, fast-hello, prefix prioritization)
2.4 Choosing IS-IS vs OSPF — Real-World Considerations
2.5 Lab: Multi-area IS-IS with route leaking

### Module 3: BGP — The Backbone Protocol
3.1 iBGP Scalability (route reflectors, confederations)
3.2 eBGP Peering Designs (IXP, PNI, transit)
3.3 BGP Path Selection Deep Dive (all 15+ attributes)
3.4 BGP Communities & Extended Communities
3.5 BGP Security (RPKI, ROV, BGPsec basics)
3.6 BGP Convergence & Optimization (ADD-PATH, PIC, next-hop tracking)
3.7 Lab: Route reflector hierarchy with optimal exit

### Module 4: MPLS Fundamentals
4.1 Label Switching Architecture
4.2 LDP — Label Distribution Protocol
4.3 MPLS Forwarding (PHP, UHP, label stack)
4.4 MPLS OAM (LSP ping, traceroute)
4.5 Lab: Basic MPLS core with LDP

### Module 5: Traffic Engineering
5.1 RSVP-TE Deep Dive (path setup, FRR, make-before-break)
5.2 TE Constraints (bandwidth, affinity, SRLG)
5.3 Offline vs Online Path Computation
5.4 PCE Architecture (stateful, stateless, PCE-initiated)
5.5 Lab: RSVP-TE with node/link protection

### Module 6: Segment Routing
6.1 SR-MPLS Fundamentals (prefix-SID, adjacency-SID, node-SID)
6.2 SR-TE Policies (explicit paths, flex-algo)
6.3 TI-LFA (topology-independent loop-free alternate)
6.4 SRv6 — Segment Routing over IPv6
6.5 SRv6 Network Programming (END, END.X, END.DT)
6.6 SR vs RSVP-TE — Migration Strategies
6.7 Lab: SR-MPLS with TI-LFA and flex-algo

### Module 7: L3VPN Services
7.1 MPLS L3VPN Architecture (VRF, RD, RT)
7.2 MP-BGP for VPNv4/VPNv6
7.3 Inter-AS L3VPN (Options A, B, C)
7.4 Extranet & Shared Services
7.5 L3VPN Troubleshooting Methodology
7.6 Lab: Multi-tenant L3VPN with inter-AS Option C

### Module 8: L2VPN & EVPN
8.1 Legacy L2VPN (VPWS, VPLS, H-VPLS)
8.2 EVPN Fundamentals (route types 1-5)
8.3 EVPN-MPLS vs EVPN-VXLAN
8.4 EVPN Multi-Homing (all-active, single-active)
8.5 EVPN for DCI (data center interconnect)
8.6 Lab: EVPN-MPLS with all-active multi-homing

### Module 9: Transport & Optical
9.1 SP Transport Hierarchy (access, metro, long-haul)
9.2 DWDM Fundamentals (wavelengths, amplifiers, ROADMs)
9.3 OTN — Optical Transport Network
9.4 Packet-Optical Integration
9.5 Coherent Optics (400G+)
9.6 Lab: Simulated optical path planning

### Module 10: Network Slicing & 5G Transport
10.1 Network Slicing Concepts
10.2 FlexE — Flexible Ethernet
10.3 5G Xhaul Requirements (fronthaul, midhaul, backhaul)
10.4 TSN — Time-Sensitive Networking in SP
10.5 SR + Network Slicing Integration

### Module 11: Automation & Operations
11.1 Model-Driven Networking (YANG, NETCONF, gRPC)
11.2 Telemetry (streaming vs polling, gNMI)
11.3 SR-TE Controller Integration
11.4 CI/CD for Network Config
11.5 Lab: gNMI telemetry with SR policy automation

### Module 12: Design Case Studies
12.1 Regional ISP Backbone Design
12.2 National Carrier Network
12.3 Content Provider WAN
12.4 Mobile Operator Convergence
12.5 Submarine Cable Landing Architecture

---

## Emerging Technologies to Track

| Technology | Status | Relevance |
|------------|--------|-----------|
| SRv6+ (compressed SIDs) | Emerging | Reduces header overhead |
| IGP Flex-Algo | Deployed | Multiple topologies, one IGP |
| BGP-CT (Classful Transport) | Draft | Next-gen inter-domain TE |
| DetNet | Emerging | Deterministic networking |
| RIFT | Experimental | Routing in Fat Trees |
| Path-aware networking | Research | SCION, etc. |

---

## Research Sources

### Vendor Documentation
- Cisco SP design guides (CCO)
- Juniper TechLibrary
- Nokia Network Developer Portal
- Arista SP deployment guides

### Standards
- IETF RFCs (BGP, MPLS, SR, EVPN)
- ITU-T (optical transport)
- IEEE (Ethernet, TSN)

### Training References
- INE IE-SP curriculum
- Juniper IE-SP syllabus
- CBT Nuggets SP track

### Real-World Sources
- NANOG presentations
- RIPE meeting archives
- APRICOT/APNIC content
- Vendor blogs (Cisco, Juniper, Nokia)

---

## Build Plan

### Phase 1: Framework (Week 1)
- [ ] Finalize module structure
- [ ] Create template for each section
- [ ] Set up lab environment specs (EVE-NG/CML)
- [ ] Identify authoritative sources per topic

### Phase 2: Core Protocols (Weeks 2-4)
- [ ] Module 2: IGP at Scale
- [ ] Module 3: BGP
- [ ] Module 4: MPLS Fundamentals

### Phase 3: Advanced Services (Weeks 5-7)
- [ ] Module 5: Traffic Engineering
- [ ] Module 6: Segment Routing
- [ ] Module 7: L3VPN

### Phase 4: Modern Architecttic (Weeks 8-10)
- [ ] Module 8: L2VPN & EVPN
- [ ] Module 9: Transport & Optical
- [ ] Module 10: Network Slicing

### Phase 5: Operations & Design (Weeks 11-12)
- [ ] Module 11: Automation
- [ ] Module 12: Case Studies
- [ ] Module 1: Foundations (write last, informed by rest)

### Nightly Build Tasks
Each night session can tackle:
1. Research one sub-section topic
2. Write/refine one section draft
3. Create one lab topology
4. Review and update emerging tech section
5. Cross-reference with cert objectives

---

## Deliverables

1. **Study Guide** — Markdown docs per module (printable)
2. **Lab Topologies** — EVE-NG/CML configs
3. **Quick Reference Cards** — Cheat sheets per protocol
4. **Practice Questions** — Scenario-based problems
5. **Config Snippets** — Vendor-specific (Cisco IOS-XR, Junos, Nokia SR-OS)

---

## Notes

- Vendor-neutral concepts first, vendor-specific implementation second
- Every section should have: Theory → Config → Verification → Troubleshooting
- Real-world war stories where possible
- Keep updated as tech evolves (SRv6, etc.)
