# Module 1 — Service Provider Foundations

This module gives you the conceptual framework for everything that follows. Written last so it can reference all later modules; read first so the rest makes sense.

## Sections in this module

- **[1.1–1.4: SP Foundations](1.1-1.4-sp-foundations.md)** — All four sections in a single document

## What you'll be able to do

- Explain what distinguishes a service provider network from an enterprise network
- Describe the three-tier P/PE/CE topology and what each tier does
- Apply the three core design principles (scalability, convergence, fault isolation) to any design decision
- Explain why IS-IS is preferred over OSPF in SP, and why MPLS is non-optional
- Describe the regional ISP, national carrier, and Tier-1 global topologies and their design differences

## Sections

### 1.1: SP Network Architecture Overview
- What a service provider network is and why it's different from enterprise
- The business model — SLAs, multi-tenancy, revenue-generating infrastructure
- Scale numbers: route tables, VRF counts, node counts at regional/national/Tier-1
- The three-tier topology (P / PE / CE)
- Where every other module fits in the protocol stack

### 1.2: Design Principles
- Scalability: why O(N²) relationships fail at SP scale, and how protocols avoid them
- Convergence: the 50ms SLA, BFD, SPF tuning, TI-LFA, and the convergence budget
- Fault isolation: control-plane and data-plane isolation at every layer
- The Day-2 framing: designing for operations, not just initial build

### 1.3: SP vs Enterprise — Fundamental Differences
- Protocol selection differences (IS-IS, BGP role, MPLS as requirement)
- Multi-tenancy as first-class design constraint (VRFs, RD/RT)
- Convergence requirements: 99.999% uptime vs 99.9%
- SP operational model: NOC, MTTR targets, revenue-generating framing

### 1.4: Reference Topologies
- Regional ISP backbone (~300–800 nodes)
- National carrier (2,000–20,000 nodes)
- Tier-1 / global backbone (20,000+ nodes)
- Topology comparison table

## Prerequisites

- Comfort with IP routing basics, subnetting, and CLI operations
- Basic understanding of BGP and IGP concepts (deepened in Modules 2–3)

## Review Questions

Five expert-caliber questions in the section file covering:
- L3VPN convergence troubleshooting
- RSVP-TE FRR vs TI-LFA design justification
- Asymmetric L3VPN blackhole diagnosis
- Post-failure BGP reconvergence analysis
- OSPF vs IS-IS SP design argument

## Cross-references

Every major concept in this module links to the module where it's covered in depth. See the cross-reference table at the end of the section file.
