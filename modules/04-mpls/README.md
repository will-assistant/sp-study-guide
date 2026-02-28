# Module 4 — MPLS Core Operations

This module focuses on the service-provider problem space around mpls. It connects design intent to operations so you can make defensible architecture choices and troubleshoot production-impacting issues with confidence.

## Sections in this module

| Section | Topic | What it covers | Est. time |
|---|---|---|---:|
| 4.1 | [4.1 — LDP & Label Distribution](4.1-ldp-and-label-distribution.md) | Label Distribution Protocol (LDP) is the workhorse signaling protocol that builds Label Switche... | 3 min |
| 4.2 | [4.2 — RSVP-TE: Traffic Engineered Label Switching](4.2-rsvp-te.md) | RSVP-TE (Resource Reservation Protocol — Traffic Engineering) is MPLS's answer to the question:... | 3 min |
| 4.3 | [4.3 — Label Operations: Push, Swap, Pop, and the Label Stack](4.3-label-operations.md) | MPLS forwarding boils down to three operations: push, swap, and pop. That sounds simple — and a... | 3 min |
| 4.4 | [4.4 — MPLS OAM & Troubleshooting](4.4-mpls-oam-and-troubleshooting.md) | MPLS is a beautiful forwarding paradigm right up until something breaks and you realize you can... | 3 min |

## Prerequisites
- Modules 2–3; stable IGP and BGP control-plane understanding.
- Recommended: complete [Module 3 — BGP at SP Scale](../03-bgp/README.md) first.

## What you'll be able to do
- explain why this technology is used in SP networks
- map control-plane signals to forwarding outcomes
- verify healthy operation using platform show commands
- identify common failure patterns and likely root causes

## Key protocols/technologies covered
- LDP, MPLS, RSVP-TE

## Labs
- See [`../../labs`](../../labs) for available lab guides tied to this module.
