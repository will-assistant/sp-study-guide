# Module 3 — BGP at SP Scale

This module focuses on the service-provider problem space around bgp. It connects design intent to operations so you can make defensible architecture choices and troubleshoot production-impacting issues with confidence.

## Sections in this module

| Section | Topic | What it covers | Est. time |
|---|---|---|---:|
| 3.1 | [3.1 — BGP Fundamentals at SP Scale](3.1-bgp-fundamentals-at-sp-scale.md) | BGP is the protocol that holds the internet together. Full stop. In a service provider, BGP doe... | 3 min |
| 3.2 | [3.2 — iBGP Design: Route Reflectors & Confederations](3.2-ibgp-design.md) | BGP's full-mesh requirement for iBGP is the single biggest scalability problem in the protocol.... | 2 min |
| 3.3 | [3.3 — eBGP Peering: IX, Transit & Customers](3.3-ebgp-peering.md) | eBGP peering is where the money is. Your peering strategy — who you peer with, where, and under... | 3 min |
| 3.4 | [3.4 — BGP Policy & Traffic Engineering](3.4-bgp-policy-and-traffic-engineering.md) | BGP is the only protocol where you can influence routing decisions across autonomous systems yo... | 3 min |

## Prerequisites
- Module 2 IGP underlay and route-policy basics.
- Recommended: complete [Module 2 — IGP at Scale](../02-igp/README.md) first.

## What you'll be able to do
- explain why this technology is used in SP networks
- map control-plane signals to forwarding outcomes
- verify healthy operation using platform show commands
- identify common failure patterns and likely root causes

## Key protocols/technologies covered
- BGP

## Labs
- See [`../../labs`](../../labs) for available lab guides tied to this module.
