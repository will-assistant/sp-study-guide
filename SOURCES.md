# Authoritative Sources by Module

## Module 1: Foundations
- *SP Network Design* — Cisco Press (Alvaro Retana)
- NANOG tutorials: "Introduction to SP Architecture"
- ITU-T G.805 — Generic functional architecture of transport networks

## Module 2: IGP at Scale
- **RFC 1195** — IS-IS for IP (original)
- **RFC 5305** — IS-IS Extensions for TE
- **RFC 5308** — IS-IS for IPv6
- **RFC 7981** — IS-IS Extensions for Advertising Router Info
- **RFC 8405** — SPF Back-off algorithm
- Cisco IOS-XR IS-IS Config Guide
- Juniper Day One: IS-IS Deployment in IP/MPLS Networks
- *IS-IS: Deployment in IP Networks* — Russ White (Cisco Press, older but deep)

## Module 3: BGP
- **RFC 4271** — BGP-4 base spec
- **RFC 4456** — Route Reflection
- **RFC 5065** — Confederations
- **RFC 7454** — BGP Operations and Security (BCP 194)
- **RFC 8205** — BGPsec
- **RFC 6811** — RPKI-based ROV
- **RFC 7911** — ADD-PATH
- **RFC 7999** — Blackhole Community
- *Internet Routing Architectures* — Sam Halabi (Cisco Press)
- *BGP Design and Implementation* — Randy Zhang (Cisco Press)
- Juniper SP Study Guide (BGP chapters)
- MANRS guidelines (mutually agreed norms for routing security)

## Module 4: MPLS Fundamentals
- **RFC 3031** — MPLS Architecture
- **RFC 3032** — MPLS Label Stack Encoding
- **RFC 5036** — LDP Specification
- **RFC 4379** — LSP Ping
- *MPLS Fundamentals* — Luc De Ghein (Cisco Press)
- Juniper Day One: MPLS

## Module 5: Traffic Engineering
- **RFC 3209** — RSVP-TE Extensions
- **RFC 4090** — Fast Reroute (FRR)
- **RFC 5440** — PCE Protocol (PCEP)
- **RFC 8231** — Stateful PCE
- **RFC 8281** — PCE-Initiated LSPs
- *Traffic Engineering with MPLS* — Eric Osborne (Cisco Press)
- NANOG 67: "RSVP-TE Deployment Realities"

## Module 6: Segment Routing
- **RFC 8402** — SR Architecture
- **RFC 8660** — SR-MPLS
- **RFC 8667** — IS-IS Extensions for SR
- **RFC 8669** — OSPF Extensions for SR
- **RFC 8986** — SRv6 Network Programming
- **RFC 9252** — BGP Overlay Services with SR
- **RFC 9256** — SR Policy Architecture
- **draft-ietf-spring-srv6-srh** — SRv6 SRH
- **draft-ietf-spring-compressed-srv6** — SRv6+ (compressed SIDs)
- Cisco SP design guide: "Migrating to Segment Routing"
- Juniper Day One: Segment Routing with Junos
- *Segment Routing Part I & II* — Clarence Filsfils (Cisco Press)
- NANOG 75: "SRv6 — What, Why, How"

## Module 7: L3VPN
- **RFC 4364** — BGP/MPLS IP VPNs (L3VPN)
- **RFC 4659** — VPNv6
- **RFC 4684** — Constrained RT Distribution
- **RFC 4577** — OSPF as PE/CE
- *MPLS and VPN Architectures Vol I & II* — Pepelnjak/Guichard (Cisco Press)
- Juniper SP Study Guide (VPN chapters)

## Module 8: L2VPN & EVPN
- **RFC 4761** — VPLS using BGP
- **RFC 4762** — VPLS using LDP
- **RFC 7432** — EVPN
- **RFC 8365** — EVPN Overlay Framework
- **RFC 9135** — EVPN Multi-Homing
- **RFC 9136** — EVPN Designated Forwarder Election
- *EVPN in the Data Center* — Lukas Krattiger (Cisco Press, DC focus but excellent EVPN fundamentals)
- Juniper Day One: Data Center EVPN-VXLAN

## Module 9: Transport & Optical
- ITU-T G.709 — OTN
- ITU-T G.694.1 — DWDM Channel Spacing
- ITU-T G.698.2 — Coherent DWDM
- *Fiber Optic Reference Guide* — David Goff
- Ciena WaveLogic documentation (coherent optics)
- NANOG: "400G and Beyond — Coherent Optics in SP Networks"
- Infinera/Ciena/Nokia optical whitepapers

## Module 10: Network Slicing & 5G Transport
- 3GPP TS 28.530 — Network Slicing Management
- **RFC 8578** — DetNet Use Cases
- **RFC 8655** — DetNet Architecture
- IEEE 802.1Qbv — Time-Aware Shaper (TSN)
- FlexE Forum specifications
- Nokia: "5G Transport Networks" whitepaper
- Ericsson: "Transport Network Slicing" whitepaper
- GSMA: Network Slicing guidelines

## Module 11: Automation & Operations
- **RFC 6241** — NETCONF
- **RFC 7950** — YANG 1.1
- **RFC 8040** — RESTCONF
- gNMI specification (openconfig.net)
- OpenConfig YANG models repository
- *Network Programmability with YANG* — Benoît Claise (Cisco Press)
- Juniper Automation Documentation
- Cisco NSO documentation

## Module 12: Design Case Studies
- NANOG/RIPE/APRICOT presentation archives (real operator designs)
- PeeringDB data for IXP/peering analysis
- Submarine Cable Map (submarinecablemap.com)
- Google/Meta/AWS network architecture blog posts

---

## Cert Objective Mapping

### Cisco SP Blueprint (v5)
- [Cisco Blueprint](https://learningnetwork.cisco.com/s/ccie-service-provider)
- Focus areas: IGP, BGP, MPLS, L3VPN, L2VPN, Multicast, QoS, Automation

### IE-SP
- [Juniper Blueprint](https://www.juniper.net/us/en/training/certification/tracks/service-provider-routing-switching/jncie-sp.html)
- Focus areas: IS-IS, BGP, MPLS, L3VPN, L2 services, TE, Multicast

### Cross-Reference
| Module | IE-SP Section | IE-SP Section |
|--------|----------------|-----------------|
| 2 (IGP) | 1.0 SP Core | Layer 3 |
| 3 (BGP) | 2.0 BGP | Layer 3 |
| 4 (MPLS) | 3.0 MPLS | MPLS |
| 5 (TE) | 3.0 MPLS / TE | MPLS / TE |
| 6 (SR) | emerging (not in v5) | emerging |
| 7 (L3VPN) | 4.0 VPN | VPN |
| 8 (EVPN) | 4.0 VPN | VPN |
| 11 (Auto) | 6.0 Automation | — |
