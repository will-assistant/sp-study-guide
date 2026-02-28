# Module 8 — L2VPN and EVPN

This module focuses on the service-provider problem space around l2vpn evpn. It connects design intent to operations so you can make defensible architecture choices and troubleshoot production-impacting issues with confidence.

## Sections in this module

| Section | Topic | What it covers | Est. time |
|---|---|---|---:|
| 8.1 | [8.1 — Legacy L2VPN: VPWS, VPLS, and H-VPLS](8.1-legacy-l2vpn.md) | Legacy L2VPN services are the original "Ethernet over MPLS" technologies that let service provi... | 5 min |
| 8.2 | [8.2 — EVPN Fundamentals](8.2-evpn-fundamentals.md) | EVPN (Ethernet VPN, RFC 7432) is the control plane that finally makes L2VPN services sane. Wher... | 4 min |
| 8.3 | [8.3 — EVPN-MPLS vs EVPN-VXLAN](8.3-evpn-mpls-vs-vxlan.md) | EVPN is the control plane. The data plane is your choice — MPLS or VXLAN. This isn't a religiou... | 4 min |
| 8.4 | [8.4 — EVPN Multi-Homing](8.4-evpn-multi-homing.md) | Multi-homing is where EVPN earns its reputation. VPLS gave you active/standby with STP — one li... | 4 min |
| 8.5 | [8.5 — EVPN for Data Center Interconnect (DCI)](8.5-evpn-dci.md) | Data Center Interconnect is where EVPN stops being a single-site protocol and becomes a network... | 6 min |

## Prerequisites
- Module 3 BGP policy and Module 7 service-edge concepts.
- Recommended: complete [Module 7 — MPLS L3VPN Services](../07-l3vpn/README.md) first.

## What you'll be able to do
- explain why this technology is used in SP networks
- map control-plane signals to forwarding outcomes
- verify healthy operation using platform show commands
- identify common failure patterns and likely root causes

## Key protocols/technologies covered
- EVPN, MPLS, VPLS, VXLAN

## Labs
- See [`../../labs`](../../labs) for available lab guides tied to this module.
