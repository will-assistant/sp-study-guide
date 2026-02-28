# SP Study Guide — Project State

> **Read this before starting work. Update when stopping.**

## Current Status
🟢 **Phase 1: Framework** — Complete
🟢 **Phase 2: Core Protocols** — Module 2, 3, 4 Complete
🟢 **Phase 3: Advanced Services** — Module 5 Complete, Module 6 Complete (6.1-6.6 + Lab 6.7), Module 7 Complete (7.1-7.5)
🟢 **Phase 4: Modern Architecture** — Module 8 Complete (8.1-8.5), Module 9 Complete (9.1-9.5 + Lab 9.6), Module 10 Complete (10.1-10.3)
🟢 **Phase 5: Operations & Design** — Module 11 Complete; Module 12 Complete (12.1–12.5); Module 1 Foundations Complete
🎉 **ALL MODULES COMPLETE** — Full SP Study Guide written and reviewed

## Last Session
- **Date**: 2026-02-22 (late evening session)
- **Work Done**:
  - **Section 12.5: Design Case Study — SP EVPN L2VPN Migration** (`modules/12-case-studies/12.5-evpn-l2vpn-migration.md`)
  - 1,149 lines (~46KB), comprehensive L2VPN-to-EVPN migration case study
  - Scope covered:
    - Fictional Tier-2 SP (TransitLink) with 5,000 VPWS PWs and 200 VPLS instances
    - Why migrate: VPLS MAC flooding, no multi-homing, signaling scale, no host mobility
    - Migration strategies: big bang (rejected), parallel overlay (primary), seamless interworking (large VPLS)
    - Phase 0: BGP EVPN activation, RT constrain, control-plane impact, RD/RT design
    - Phase 1: VPWS→EVPN-VPWS (RFC 8214) with per-service runbook and rollback
    - Phase 2: VPLS→EVPN multipoint — parallel overlay + seamless interworking for 50+ PE instances
    - Phase 3: MC-LAG→EVPN all-active multi-homing with ESI assignment and per-site runbook
    - Validation commands (IOS-XR + Junos), risk matrix (12 risks), BGP table growth analysis
    - 12-month timeline, staffing model, 5 design review questions
  - Cross-references: Module 5, Module 7, Module 12.2, Module 11.1

- **Previous Session**: 2026-02-22 (evening session 2)
- **Previous Work**:
  - **Section 12.4: Design Case Study — Internet Exchange Point Design** (`modules/12-case-studies/12.4-internet-exchange-point-design.md`)
  - 1,227 lines (~48KB), comprehensive greenfield IXP design case study
  - Scope covered:
    - Fictional scenario: BasinIX launching in Basin City (18 ISPs tromboning 600mi to nearest IXP)
    - Physical infrastructure: dual-site active-active, meet-me rooms, cross-connects, optics
    - Layer 2 fabric: traditional VLAN-based (with EVPN-VXLAN comparison and migration path)
    - Ethernet fabric: 4-switch full mesh, RSTP with BPDU guard, storm control, MAC limits
    - Route server: BIRD 2 complete config with rs client, ADD-PATH, graceful restart
    - RPKI/IRR filtering pipeline: Routinator + bgpq4 cron-generated prefix-lists per member
    - BGP peering policies: multilateral (RS) vs bilateral, community-based selective peering
    - Blackholing service: RTBH with community 65535:666, /32 only, IRR validation
    - Member configs: full IOS-XR (AS64501) and Junos (AS64502) with max-prefix, bogon filters
    - ARP/ND sponge, BCP recommendations (Euro-IX/MANRS compliance)
    - Monitoring: LibreNMS, sFlow, Alice-LG looking glass, PeeringDB integration
    - Security: MAC filtering, BPDU guard, storm control, ARP inspection, RA guard
    - Growth model: 3-year financial projection, growth triggers, fee structure
    - 5 design review questions
  - Cross-references: Module 3 (BGP), Module 4 (Peering), Module 8 (EVPN), Section 12.1

- **Previous Session**: 2026-02-22 (evening session)
- **Previous Work**:
  - **Section 12.3: Design Case Study — 5G Mobile Backhaul / Transport Network** (`modules/12-case-studies/12.3-mobile-backhaul-5g-transport.md`)
  - 1,077 lines (~45KB), comprehensive 5G transport design case study
  - Scope covered:
    - Fictional regional carrier (Prairie Wireless) upgrading 1,200 sites from 4G to 5G
    - Four-tier converged xhaul architecture (cell site → pre-agg → agg hub → core)
    - eCPRI Option 7-2x fronthaul design with VLAN-per-sector scheme
    - FlexE channelization for fronthaul/midhaul/backhaul hard isolation
    - PTP/SyncE timing distribution (G.8275.1, 6 GNSS grandmasters, boundary clock chain)
    - SR-MPLS transport with IS-IS L1/L2, SRGB allocation, TI-LFA protection
    - Flex-Algo 128/129/130 for URLLC/eMBB/mIoT transport slicing
    - SR-TE policies with color communities for slice-aware steering
    - 6-class QoS model with IOS-XR and Junos configs
    - Anycast SID redundancy for core and agg hubs
    - Capacity planning with 3-year growth model and CapEx summary
    - 5 design review questions (fronthaul latency, timing failure, slice isolation, FWA capacity, LDP-to-SR migration)
  - Cross-references: Module 10.3 (xhaul), Module 10.2 (FlexE), Module 10.1 (slicing), Module 6 (SR)

- **Previous Session**: 2026-02-22 (nightly work session 2am)
- **Previous Work**:
  - **Section 12.2: Design Case Study — DCI with EVPN** (`modules/12-case-studies/12.2-dci-with-evpn.md`)
  - Initial draft: 725 lines (~38KB), committed to GitHub (`8ac4213`)
  - Scope covered:
    - STP-based DCI failure analysis
    - EVPN multi-site BGW architecture and underlay options (VXLAN/IP, MPLS, SR-MPLS)
    - Route-type usage (Type-1/2/3/4/5), BUM handling, ARP suppression behavior
    - L3 DCI with Type-5 policy, DR L3-only design, failure scenarios, scale planning
    - Design checklist + 5 review questions
  - **Review Findings + Fixes Applied**:
    - Adversarial and peer review identified critical control-plane issues:
      - Missing Type-4 context and over-asserted Type-1 semantics
      - Incorrect ESI/DF assumptions on generic DCI overlay links
      - Overstated Type-5 GW-IP usage
      - Invalid NX-OS L2 VNI ARP suppression example
      - DR section contradictions (L2 vs L3-only)
      - MTU wording/maths and scale-model assumptions needing correction
      - Compliance references overstated/misaligned
    - Corrected all identified issues in-place:
      - Added Type-4 route context and corrected failure-behavior language
      - Reframed multisite loop prevention as site-id/site-of-origin split-horizon
      - Corrected ARP suppression CLI examples (L2 vs L3 VNI distinction)
      - Removed contradictory DR L2 statements; made DR consistently L3-only
      - Reworked MTU and scale guidance with explicit caveats
      - Replaced hard compliance claims with internal-policy wording
      - Converted Junos section to platform-safe implementation intent (no risky copy-paste syntax)

- **Previous Session**: 2026-02-22 (nightly work session midnight)
- **Work Done**:
  - **Section 11.5: Lab — gNMI SR-TE Automation** (878 lines, ~33KB)
    - Full closed-loop lab: gnmic streaming → Python daemon → NETCONF SR-TE preference flip
    - Reviewed, 11 issues fixed (NETCONF namespace, Prometheus label, candidate DS, OC YANG enablement)
    - Commit: `46c787e`
    - SP CI/CD pipeline stages: lint → semantic validation → gated deploy → verify → rollback
    - Git-based intent model and policy unit-test examples
    - IOS-XR/Junos commit safety patterns (`commit confirmed` used as safety net, not dry-run)
    - Ring-based rollout design and role-weighted health gates
    - Telemetry-driven promotion/abort criteria with service-level probes
    - Rollback patterns: confirmed commit, checkpoint revert, Git revert redeploy
    - Failure modes + war story (green pipeline still caused outage via policy term order)
    - 5 review questions
  - **Review Findings (safety/factual fixes applied)**:
    - Corrected unsafe dry-run wording (`commit confirmed` removed from dry-run stage)
    - Corrected Junos confirmation phrasing (plain `commit` finalizes prior `commit confirmed`)
    - Reframed rollback language: explicit rollback first, timeout revert as safety net only
    - Added cross-vendor lock/checkpoint caveat and simulation-coverage caveat
    - Tightened telemetry gate realism (role-weighted BGP gates, service-path probes vs raw interface counters)
  - Local draft complete

- **Previous Session**: 2026-02-19 (nightly work session 1am)
- **Work Done**:
  - **Section 11.3: SR-TE Controller Integration** (~911 lines, 28KB)
    - PCE architecture (stateless vs stateful, roles, delegation)
    - PCEP deep dive (session establishment, message types, capabilities)
    - BGP-LS topology distribution (AFI 16388/SAFI 71, NLRI types, IOS-XR + Junos configs)
    - Cisco SR-PCE server + PCC configuration
    - SR policy with PCE delegation and explicit fallback
    - On-Demand Nexthop (ODN) — color ext-community workflow, IOS-XR + Junos
    - Juniper Paragon Pathfinder capabilities + PCEP client config + REST API (illustrative)
    - Cisco Crosswork/XTC overview
    - Binding SID — stitching, allocation strategy (fixed: SRLB ≠ BSID range)
    - Disjoint path computation (fixed: `disjoint-path group-id` syntax)
    - Auto-bandwidth (note: per-policy in SR-TE, not global)
    - Telemetry integration with PCE (gnmic → controller feedback loop)
    - PCEP HA (redundant PCE, delegation cleanup timer)
    - Troubleshooting (PCEP session, delegation, BGP-LS TED, stale PCE-initiated)
    - PCE placement design guidance
    - War story: PCE reoptimization loop (telemetry-driven oscillation)
    - 5 review questions

  - **Review Findings (10 fixes applied)**:
    - Peer (B+): Found Type 40 SR-ERO fabricated (fixed), disjointness block syntax wrong (CRITICAL — fixed), PCNotf→PCNtf, auto-BW global block wrong, `clear pce-delegation` fabricated, local-block for BSID wrong, `show ospf distribute link-state` wrong
    - Adversarial: Found `sr-te-capability` Junos knob likely fabricated (HIGH — removed), `show pce ipv4 lsp` wrong (MEDIUM — fixed), TE Policy NLRI misleading (MEDIUM — fixed), `bw-objective min-hops` fabricated (already self-caught + fixed before reviews), review Q2 "3 minutes" reframed as explicitly abnormal (MEDIUM — fixed)
    - All CRITICAL and HIGH issues resolved. 10 total fixes applied.
  - Commit 4f4c00b (local only — GitHub suspended)

- **Previous Session**: 2026-02-18 (nightly work session 3am)
- **Work Done**:
  - **Section 11.2: Streaming Telemetry** (~33KB)
    - Polling vs streaming, dial-in vs dial-out, gNMI Subscribe deep dive
    - Sensor paths (OC + native), IOS-XR MDT + Junos JTI configs
    - gnmic YAML + Telegraf TOML pipeline configs
    - Scale math (500 PEs), HA patterns, Prometheus alerting
    - 10 review fixes applied (Junos analytics native-only, SubscriptionMode enum, etc.)
    - Commit 1e15345 (local only — GitHub suspended)

- **Previous Session**: 2026-02-18 (nightly work session 12am)
- **Previous Work**:
  - **Section 11.1: Model-Driven Networking** (~30KB)
    - YANG data modeling (constructs, model ecosystem, discovery)
    - NETCONF deep dive (datastores, candidate commit, merge vs replace, XPath)
    - RESTCONF HTTP mapping (verbs, URL structure, comparison)
    - gNMI operations (Get/Set/Subscribe modes)
    - Protocol comparison (NETCONF vs RESTCONF vs gNMI)
    - IOS-XR + Junos configs (NETCONF/gRPC/RESTCONF enable, session limits)
    - Tools + code examples (ncclient, gnmic)
    - Architecture patterns (declarative, event-driven, hybrid)
    - War story: phantom candidate lock
  - Review fixes: ncclient confirm_timeout param, removed double config wrapper,
    fixed Junos gRPC same-port bug, corrected RFC citations (8529→9657 for BGP),
    removed invalid IOS-XR ssh server netconf command, XPath filter fixed,
    added YANG import caveat, OC vs native path distinction

- **Previous Session**: 2026-02-17 (nightly work session 1am)
  - **Section 10.3: 5G Xhaul Requirements** (~32KB, ~700 lines)
    - Disaggregated gNB (O-RU/O-DU/O-CU), 3GPP functional splits (Option 2/7-2x/8)
    - CPRI vs eCPRI comparison with bandwidth tables
    - Fronthaul/midhaul/backhaul requirements matrix
    - Converged xhaul topology (access/pre-agg/agg/core tiers)
    - Transport slicing with Flex-Algo + BGP-CT
    - PTP/SyncE timing architecture (G.8275.1/G.8275.2)
    - IOS-XR + Junos configs (QoS, PTP, SyncE, Flex-Algo, L3VPN)
    - Troubleshooting matrix + war story (asymmetric fiber PTP failure)
    - 5 review questions

- **Previous Session**: 2026-02-15 (nightly work session 3am)
  - **Section 10.2: FlexE — Flexible Ethernet** (~38KB, ~813 lines)
    - OIF FlexE spec history (1.0 through 3.0, 800G PHYs, 100G calendar slots)
    - FlexE shim architecture within IEEE 802.3 PCS layer
    - Three core operations: bonding, sub-rating, channelization (with ASCII diagrams)
    - Overhead frame structure (8 OH blocks/frame, 32-frame multiframe, ~104.8µs/frame, ~3.35ms/multiframe)
    - Conceptual configs for IOS-XR, Junos, Huawei NE40E (with strong caveats on syntax verification)
    - FlexE vs Flex-Algo comparison table for slice architecture decisions
    - Three transport modes (unaware, termination, aware)
    - Design guidance: when to deploy FlexE vs enhanced-soft slicing
    - Bandwidth planning (5G/25G/100G slot granularity)
    - Troubleshooting matrix + war story (calendar slot numbering mismatch)
    - 5 review questions (scenario, design, troubleshooting, comparison, architecture)
  - **Review Results**:
    - Peer review (B+): Caught overhead timing error (132µs was wrong), frame structure terminology confusion (period vs frame), FlexE 3.0 date flagged as unverifiable (confirmed valid via OIF source)
    - Adversarial review: CRITICAL — overhead timing fabricated (fixed to OIF spec ~104.77µs), multiframe timing cascade error (fixed to ~3.35ms), frame structure conflation (fixed with clear period/frame/multiframe hierarchy). HIGH — vendor configs likely conceptual (strengthened caveats), FlexE 3.0 missing 100G slot feature (added), Huawei syntax mismatch (corrected to FlexE-100GE model)
    - All CRITICAL and HIGH issues fixed
  - Commit b9050b0, pushed to GitHub

- **Stopped At**: Section 10.2 complete. Next: Section 10.3 (5G Xhaul Requirements).

## Next Actions
1. ~~Section 12.3: Mobile Backhaul / 5G Transport (case study)~~ ✅ Complete
2. ~~Section 12.4: Internet Exchange Point Design (case study)~~ ✅ Complete
3. ~~Section 12.5: SP EVPN L2VPN Migration (case study)~~ ✅ Complete (2026-02-22 evening)
4. Module 1: Foundations (write last) ← **NEXT TASK**

## Key Files
- `PLAN.md` — Full curriculum outline and build plan
- `TEMPLATE.md` — Section template
- `modules/` — Individual module content
- `labs/` — Lab topologies and configs

## Notes
- Junos Flex-Algo lives under `protocols isis source-packet-routing flex-algorithm`
- IOS-XR Flex-Algo affinities require `affinity-map` under `segment-routing`
- NCS 5500 and ASR 9000 have different QoS architectures — always note platform caveats
- FlexE Groups = bonded PHYs; FlexE Clients = sub-rate channels
- FlexE vendor configs are poorly publicly documented — always add verification caveats
- FlexE overhead: 8 OH blocks per frame (~104.8µs), 32 frames per multiframe (~3.35ms)
- FlexE 3.0 (May 2025): 800G PHYs + 100G calendar slot granularity

## TTS Audio Generation
- **Date**: 2026-02-28
- **Audio Completed**:
  - Module 0: Introduction (5:30)
  - Module 1: Foundations Part 1 (4:40) + Part 2 (10:14)
  - Module 2: IGP — 4 sections (34:43)
  - Module 3: BGP — 4 sections (36:49)
  - Module 4: MPLS — 4 sections (in progress)
- **Audio Location**: `tts/audio/{module}/` (gitignored)
- **Pipeline**: `tts/` directory has scripts and pronunciation dictionary
- **Voice**: Kokoro `will` preset, 1.1x speed, MP3 128kbps

## Content Validation Standards
- **RFC-first**: All protocol descriptions must cite and align with the authoritative RFC
- **Multi-vendor**: Cisco IOS-XE, Junos, and open-source implementations treated equally
- **No vendor lock-in**: Distinguish vendor-specific behavior from RFC-mandated behavior
- **Operational best practices**: Validated against NANOG/RIPE presentations and real SP operational experience
- **Review checklist** (run after adding/modifying modules):
  1. RFC citations present for every protocol described
  2. Default behaviors documented per-vendor (IOS-XE, Junos) with RFC reference
  3. Vendor-specific features clearly labeled (e.g., "Cisco-proprietary: Weight")
  4. Troubleshooting uses standard tools defined in RFCs, not just vendor CLIs
  5. War stories are realistic and teach RFC-grounded lessons
  6. No single-vendor bias in "recommended" configurations
