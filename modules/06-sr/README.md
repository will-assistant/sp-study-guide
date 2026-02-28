# Module 6 — Segment Routing

This module focuses on the service-provider problem space around sr. It connects design intent to operations so you can make defensible architecture choices and troubleshoot production-impacting issues with confidence.

## Sections in this module

| Section | Topic | What it covers | Est. time |
|---|---|---|---:|
| 6.1 | [6.1 SR-MPLS Fundamentals](6.1-sr-mpls-fundamentals.md) | Segment Routing (SR) is the first clean-sheet rethinking of MPLS control planes in 20+ years. I... | 4 min |
| 6.2 | [6.2 — SR-TE Policies](6.2-sr-te-policies.md) | SR-TE (Segment Routing Traffic Engineering) policies are the mechanism for source-routing traff... | 4 min |
| 6.3 | [6.3 — TI-LFA (Topology-Independent Loop-Free Alternate)](6.3-ti-lfa.md) | TI-LFA (Topology-Independent Loop-Free Alternate) is the SR-MPLS implementation of IP Fast Rero... | 3 min |
| 6.4 | [6.4 SRv6 Fundamentals](6.4-srv6-fundamentals.md) | SRv6 (Segment Routing over IPv6) is the next evolution of Segment Routing — same source routing... | 4 min |
| 6.5 | [6.5 SRv6 Network Programming](6.5-srv6-network-programming.md) | SRv6 Network Programming is what separates SRv6 from "MPLS with IPv6 headers." Where SR-MPLS gi... | 5 min |
| 6.6 | [6.6 SR Migration Strategies](6.6-sr-migration-strategies.md) | Nobody rips out a running MPLS core on a Friday night. Migration from LDP, RSVP-TE, or SR-MPLS ... | 5 min |

## Prerequisites
- Module 5 TE concepts and Module 4 MPLS data-plane behavior.
- Recommended: complete [Module 5 — Traffic Engineering](../05-te/README.md) first.

## What you'll be able to do
- explain why this technology is used in SP networks
- map control-plane signals to forwarding outcomes
- verify healthy operation using platform show commands
- identify common failure patterns and likely root causes

## Key protocols/technologies covered
- MPLS, SRv6, TI-LFA

## Labs
- See [`../../labs`](../../labs) for available lab guides tied to this module.
