# Module 2 — IGP at Scale

This module focuses on the service-provider problem space around igp. It connects design intent to operations so you can make defensible architecture choices and troubleshoot production-impacting issues with confidence.

## Sections in this module

| Section | Topic | What it covers | Est. time |
|---|---|---|---:|
| 2.1 | [2.1 — IS-IS Deep Dive](2.1-isis-deep-dive.md) | IS-IS is the IGP of choice for the vast majority of service provider backbones worldwide. Unlik... | 2 min |
| 2.2 | [2.2 — OSPF in SP Networks](2.2-ospf-in-sp-networks.md) | OSPF is the world's most deployed IGP — but in service provider backbones, it plays second fidd... | 3 min |
| 2.3 | [2.3 — IGP Convergence Tuning](2.3-igp-convergence-tuning.md) | Convergence isn't a single event — it's a pipeline. Failure detection → notification → SPF calc... | 2 min |
| 2.4 | [2.4 — IS-IS vs OSPF: The Real-World Decision Framework](2.4-isis-vs-ospf-decision-framework.md) | Every network engineer eventually faces the question: IS-IS or OSPF? In service provider networ... | 2 min |

## Prerequisites
- Module 1 foundations; familiarity with link-state routing concepts.

## What you'll be able to do
- explain why this technology is used in SP networks
- map control-plane signals to forwarding outcomes
- verify healthy operation using platform show commands
- identify common failure patterns and likely root causes

## Key protocols/technologies covered
- IS-IS, OSPF

## Labs
- See [`../../labs`](../../labs) for available lab guides tied to this module.
