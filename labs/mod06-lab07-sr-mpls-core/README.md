# Lab 6.7 — SR-MPLS Core with TI-LFA and Flex-Algo

## Objective

Build a production-grade SR-MPLS backbone from scratch. Start with IGP + SR prefix-SIDs, layer on TI-LFA for sub-50ms protection, deploy SR-TE policies for traffic steering, and finish with Flex-Algo for latency-optimized topologies. By the end, you'll have a network that does everything RSVP-TE + LDP did — with zero signaling protocols.

## Prerequisites

- Completed Modules 6.1–6.6 (SR-MPLS Fundamentals through Migration Strategies)
- Solid understanding of IS-IS L2 operations (Module 2.1)
- Familiarity with MPLS label operations (Module 4)

## Topology

```
                              ┌──────────┐
                              │    P1    │
                              │  (Core)  │
                              │ SID:16001│
                              └──┬───┬───┘
                      Gi0/0/0 .0 │   │ .2 Gi0/0/1
                    10.0.0.0/31  │   │ 10.0.0.2/31
                      Gi0/0/0 .1 │   │ .3 Gi0/0/0
                     ┌───────────┘   └───────────┐
                     │                           │
                ┌────┴─────┐               ┌─────┴────┐
                │    P2    │───────────────│    P3    │
                │  (Core)  │ Gi0/0/1  Gi1  │  (Core)  │
                │ SID:16002│ .4     .5     │ SID:16003│
                └┬──┬──────┘               └──────┬──┬┘
                 │  │                             │  │
        Gi0/0/2 .6  │ Gi0/0/3 .8    Gi0/0/2 .10  │  │.12 Gi0/0/3
   10.0.0.6/31 │   │ 10.0.0.8/31  10.0.0.10/31  │  │ 10.0.0.12/31
        Gi0/0/2 .7  │               Gi0/0/3 .11  │  │.13 Gi0/0/2
                 │  │                  ┌──────────┘  │
                 │  └──────────┐       │    ┌────────┘
              ┌──┴─────────────┴┐   ┌──┴────┴─────────┐
              │      PE1        │   │      PE2         │
              │     (Edge)      │   │     (Edge)       │
              │    SID:16011    │   │    SID:16012     │
              └──────┬──────────┘   └──────┬───────────┘
                     │ Gi0/0/4 .14         │ Gi0/0/4 .16
                     │ 10.0.0.14/31        │ 10.0.0.16/31
                     │ Gi0/0/0 .15         │ Gi0/0/0 .17
              ┌──────┴──────┐       ┌──────┴───────┐
              │    CE1      │       │    CE2       │
              │  (Customer) │       │  (Customer)  │
              └─────────────┘       └──────────────┘
```

**Connectivity**: PE1 dual-homed to P2 (Gi0/0/2) and P3 (Gi0/0/3). PE2 dual-homed to P3 (Gi0/0/2) and P2 (Gi0/0/3). This creates full redundancy for FRR testing.

### Design Notes

- **Full mesh P-routers**: P1, P2, P3 form a triangle (3 links). Every core router has two paths to every other.
- **Dual-homed PEs**: PE1 connects to both P2 and P3. PE2 connects to both P2 and P3. This gives redundant uplinks and enables meaningful FRR testing.
- **CEs are eBGP peers**: CE1/CE2 simulate customer routers for end-to-end VPN verification.

### Device Roles

| Device | Role | Loopback | IS-IS NET | Prefix-SID | Algo-128 SID |
|--------|------|----------|-----------|------------|---------------|
| P1 | Core | 10.255.0.1/32 | 49.0000.0102.5500.0001.00 | 16001 | 16501 |
| P2 | Core | 10.255.0.2/32 | 49.0000.0102.5500.0002.00 | 16002 | 16502 |
| P3 | Core | 10.255.0.3/32 | 49.0000.0102.5500.0003.00 | 16003 | 16503 |
| PE1 | Edge | 10.255.0.11/32 | 49.0000.0102.5500.0011.00 | 16011 | 16511 |
| PE2 | Edge | 10.255.0.12/32 | 49.0000.0102.5500.0012.00 | 16012 | 16512 |
| CE1 | Customer | 10.255.0.101/32 | — | — | — |
| CE2 | Customer | 10.255.0.102/32 | — | — | — |

### Link Addressing

| Link | Subnet | Device A IP | Device B IP | Metric | Latency Metric |
|------|--------|-------------|-------------|--------|----------------|
| P1–P2 | 10.0.0.0/31 | .0 (P1) | .1 (P2) | 10 | 5 |
| P1–P3 | 10.0.0.2/31 | .2 (P1) | .3 (P3) | 10 | 20 |
| P2–P3 | 10.0.0.4/31 | .4 (P2) | .5 (P3) | 10 | 5 |
| P2–PE1 | 10.0.0.6/31 | .6 (P2) | .7 (PE1) | 10 | 5 |
| P2–PE2 | 10.0.0.8/31 | .8 (P2) | .9 (PE2) | 10 | 5 |
| P3–PE1 | 10.0.0.10/31 | .10 (P3) | .11 (PE1) | 10 | 15 |
| P3–PE2 | 10.0.0.12/31 | .12 (P3) | .13 (PE2) | 10 | 5 |
| PE1–CE1 | 10.0.0.14/31 | .14 (PE1) | .15 (CE1) | — | — |
| PE2–CE2 | 10.0.0.16/31 | .16 (PE2) | .17 (CE2) | — | — |

### SRGB

All routers: **16000–23999** (default SRGB, 8000 labels)

---

## Exercise 1: IS-IS + SR-MPLS Foundation (30 min)

**Goal**: Build IS-IS L2 backbone with SR-MPLS prefix-SIDs on all P/PE routers.

### Tasks

1. Configure IS-IS single-area L2-only on P1, P2, P3, PE1, PE2
   - NET addresses per table above
   - Wide metrics enabled
   - Point-to-point on all inter-router links
   - Advertise loopbacks as passive interfaces
2. Enable Segment Routing under IS-IS
   - SRGB: 16000–23999 on all routers
   - Prefix-SIDs on loopbacks per table
   - PHP (Penultimate Hop Popping) enabled (default behavior)
3. Verify IS-IS adjacencies, LSDB, and SR label tables

### IOS-XR Configuration

```
! === P1 (repeat pattern for P2, P3, PE1, PE2 with correct values) ===

router isis CORE
 is-type level-2-only
 net 49.0000.0102.5500.0001.00
 address-family ipv4 unicast
  metric-style wide
  segment-routing mpls
 !
 interface Loopback0
  passive
  address-family ipv4 unicast
   prefix-sid index 1
  !
 !
 interface GigabitEthernet0/0/0/0
  point-to-point
  address-family ipv4 unicast
   metric 10
  !
 !
 interface GigabitEthernet0/0/0/1
  point-to-point
  address-family ipv4 unicast
   metric 10
  !
 !
!
segment-routing
 global-block 16000 23999
!
```

### Junos Configuration

```
! === P1 (repeat pattern for P2, P3, PE1, PE2 with correct values) ===

! Interface families — REQUIRED for IS-IS and MPLS
set interfaces lo0 unit 0 family inet address 10.255.0.1/32
set interfaces lo0 unit 0 family iso address 49.0000.0102.5500.0001.00
set interfaces ge-0/0/0 unit 0 family inet address 10.0.0.0/31
set interfaces ge-0/0/0 unit 0 family iso
set interfaces ge-0/0/0 unit 0 family mpls
set interfaces ge-0/0/1 unit 0 family inet address 10.0.0.2/31
set interfaces ge-0/0/1 unit 0 family iso
set interfaces ge-0/0/1 unit 0 family mpls

! MPLS — required for SR label forwarding
set protocols mpls interface ge-0/0/0.0
set protocols mpls interface ge-0/0/1.0

! IS-IS
set protocols isis level 2 wide-metrics-only
set protocols isis level 1 disable
set protocols isis interface lo0.0 passive
set protocols isis interface lo0.0 level 2 metric 0
set protocols isis interface ge-0/0/0.0 point-to-point
set protocols isis interface ge-0/0/0.0 level 2 metric 10
set protocols isis interface ge-0/0/1.0 point-to-point
set protocols isis interface ge-0/0/1.0 level 2 metric 10
set protocols isis source-packet-routing srgb start-label 16000 index-range 8000
set protocols isis source-packet-routing node-segment ipv4-index 1

set routing-options router-id 10.255.0.1
```

### Validation

```
! IOS-XR
show isis adjacency                            ! All L2 adjacencies UP
show isis segment-routing label table          ! Verify prefix-SID→label mapping
show mpls forwarding                           ! Labels programmed in LFIB
show cef 10.255.0.12/32                        ! Should show SR label (16012)
ping 10.255.0.12 source 10.255.0.1             ! End-to-end IP reachability
traceroute sr-mpls 10.255.0.12/32              ! Verify label-switched path

! Junos
show isis adjacency
show spring-traffic-engineering lsp
show route table inet.3                        ! SR labels for all loopbacks
show route 10.255.0.12/32 extensive            ! Should show SPRING next-hop
show mpls label usage
ping 10.255.0.12 source 10.255.0.1
```

### Expected Results

- 7 IS-IS L2 adjacencies (P1↔P2, P1↔P3, P2↔P3, P2↔PE1, P2↔PE2, P3↔PE1, P3↔PE2)
- Each router has prefix-SIDs for all 5 P/PE loopbacks in its label table
- `show cef 10.255.0.12/32` on P1 shows label 16012 with ECMP next-hops to P2 and P3
- `traceroute sr-mpls` from PE1 to PE2 shows label swaps at each hop, PHP at penultimate hop

---

## Exercise 2: TI-LFA Fast Reroute (30 min)

**Goal**: Enable TI-LFA on all interfaces for sub-50ms link and node protection.

### Tasks

1. Enable TI-LFA on all IS-IS interfaces
2. Configure BFD on all inter-router links
   - Minimum interval: 100ms
   - Multiplier: 3 (300ms detection)
3. Tune IS-IS SPF timers for fast convergence
   - Initial wait: 50ms
   - Secondary wait: 200ms
   - Max wait: 5000ms
4. Verify backup paths are computed for all prefixes
5. Test failover by shutting a link and measuring convergence

### IOS-XR Configuration

```
router isis CORE
 address-family ipv4 unicast
  fast-reroute per-prefix
  fast-reroute per-prefix ti-lfa
  spf-interval initial-wait 50 secondary-wait 200 maximum-wait 5000
  lsp-gen-interval initial-wait 50 secondary-wait 200 maximum-wait 5000
 !
 interface GigabitEthernet0/0/0/0
  bfd minimum-interval 100
  bfd multiplier 3
  bfd fast-detect ipv4
  ! NOTE: For virtual routers (XRv9000, vMX), increase to 300-1000ms
  !       to avoid false BFD flaps from hypervisor scheduling jitter.
 !
 ! (repeat BFD config for all inter-router interfaces)
 !
!
```

### Junos Configuration

```
set protocols isis interface ge-0/0/0.0 level 2 post-convergence-lfa node-protection
set protocols isis interface ge-0/0/1.0 level 2 post-convergence-lfa node-protection

! Repeat for all inter-router interfaces

set protocols isis spf-options delay 50 holddown 5000 rapid-runs 3
set protocols isis overload timeout 300

! BFD (directly under interface, NOT under family inet)
set protocols isis interface ge-0/0/0.0 bfd-liveness-detection minimum-interval 100
set protocols isis interface ge-0/0/0.0 bfd-liveness-detection multiplier 3
set protocols isis interface ge-0/0/1.0 bfd-liveness-detection minimum-interval 100
set protocols isis interface ge-0/0/1.0 bfd-liveness-detection multiplier 3
```

### Validation

```
! IOS-XR
show isis fast-reroute summary                  ! Target: 100% coverage
show isis fast-reroute 10.255.0.12/32 detail    ! Backup path details
show cef 10.255.0.12/32                         ! Primary + backup next-hop
show bfd session                                ! All sessions UP

! Junos
show isis backup spf results
show isis backup coverage
show route 10.255.0.12/32 extensive             ! Look for "Next hop type: Router"
                                                ! with backup next-hop listed
show bfd session
```

### Failover Test

```
! On P2, shut the P2↔P3 link:
! IOS-XR: interface GigabitEthernet0/0/0/1 → shutdown
! Junos:  set interfaces ge-0/0/1 disable

! From PE1, continuous ping to PE2:
ping 10.255.0.12 source 10.255.0.11 count 1000 interval 10

! Expected: 0-3 lost packets (sub-50ms failover via TI-LFA backup)
! Then IS-IS reconverges within ~250ms (SPF recalculation)
```

### Expected Results

- TI-LFA coverage: 100% (all prefixes have backup paths)
- Every prefix in `show cef` shows both primary and backup next-hops
- Backup paths use label stacks (potentially 2-3 labels for node protection)
- Link failure: ≤3 pings lost at 10ms interval = sub-30ms failover
- BFD detects failure in ~300ms worst case; TI-LFA pre-computed backup installs in <1ms

> **Virtual Router Note**: BFD at 100ms may flap on resource-constrained VMs. If you see BFD instability, increase to 300ms × 3 (900ms detection). The TI-LFA mechanics work identically — only detection time changes.

### Topology Analysis: Protection Scenarios

| Failed Link | Protected Prefix | Backup Path | Label Stack |
|-------------|-----------------|-------------|-------------|
| P2↔P3 | 10.255.0.3 (P3) from P2 | P2→P1→P3 | {16003} |
| P2↔PE1 | 10.255.0.11 (PE1) from P2 | P2→P3→PE1 | {16011} |
| P1↔P2 (node-protect P2) | 10.255.0.11 from P1 | P1→P3→PE1 | {16011} |
| P2↔P3 + P2↔PE2 (node-protect P2) | 10.255.0.12 from P1 | P1→P3→PE2 | {16012} |

---

## Exercise 3: SR-TE Policies (45 min)

**Goal**: Create SR-TE policies that steer traffic along explicit paths, overriding the IGP shortest path.

### Scenario

PE1 needs to send traffic to PE2 via P1 (the "northern" path through the core) instead of the direct P2→PE2 path. This simulates steering traffic through a scrubbing center, monitoring tap, or preferred transit path.

### Tasks

1. Create an SR-TE policy on PE1 that steers traffic to PE2 via P1
   - Explicit segment list: PE1→P2→P1→P3→PE2 (or PE1→P3→P1→P2→PE2)
   - Binding SID: 24001
2. Create a second policy with a dynamic path via PCE (or on-box CSPF)
   - Constraint: avoid link P2↔P3
   - Binding SID: 24002
3. Steer VRF traffic onto the SR-TE policy using color communities
4. Verify forwarding uses the explicit path

### IOS-XR Configuration

```
! === PE1: Explicit path via P1 (northern route) ===

segment-routing
 traffic-eng
  segment-list SL-VIA-P1
   index 10 mpls label 16002    ! → P2
   index 20 mpls label 16001    ! → P1
   index 30 mpls label 16003    ! → P3
   index 40 mpls label 16012    ! → PE2
  !
  policy STEER-TO-PE2-VIA-P1
   color 100 end-point ipv4 10.255.0.12
   candidate-paths
    preference 200
     explicit segment-list SL-VIA-P1
    !
   !
   binding-sid mpls 24001
  !
  !
  ! === Dynamic policy: avoid P2↔P3 ===
  segment-list SL-AVOID-P2P3
   index 10 mpls label 16001    ! → P1 (forces path through P1)
   index 20 mpls label 16012    ! → PE2
  !
  policy STEER-TO-PE2-AVOID-P2P3
   color 200 end-point ipv4 10.255.0.12
   candidate-paths
    preference 100
     explicit segment-list SL-AVOID-P2P3
    !
   !
   binding-sid mpls 24002
  !
 !
!
```

### Junos Configuration

```
set protocols source-packet-routing segment-list SL-VIA-P1 hop1 label 16002
set protocols source-packet-routing segment-list SL-VIA-P1 hop2 label 16001
set protocols source-packet-routing segment-list SL-VIA-P1 hop3 label 16003
set protocols source-packet-routing segment-list SL-VIA-P1 hop4 label 16012

set protocols source-packet-routing source-routing-path STEER-TO-PE2-VIA-P1 to 10.255.0.12
set protocols source-packet-routing source-routing-path STEER-TO-PE2-VIA-P1 color 100
set protocols source-packet-routing source-routing-path STEER-TO-PE2-VIA-P1 binding-sid 24001
set protocols source-packet-routing source-routing-path STEER-TO-PE2-VIA-P1 primary SL-VIA-P1

set protocols source-packet-routing source-routing-path STEER-TO-PE2-AVOID-P2P3 to 10.255.0.12
set protocols source-packet-routing source-routing-path STEER-TO-PE2-AVOID-P2P3 color 200
set protocols source-packet-routing source-routing-path STEER-TO-PE2-AVOID-P2P3 binding-sid 24002
set protocols source-packet-routing source-routing-path STEER-TO-PE2-AVOID-P2P3 primary SL-AVOID-P2P3

set protocols source-packet-routing segment-list SL-AVOID-P2P3 hop1 label 16001
set protocols source-packet-routing segment-list SL-AVOID-P2P3 hop2 label 16012
```

### Color-Based Steering (ODN)

To steer VRF traffic onto the SR-TE policy, attach a color extended community via BGP:

```
! IOS-XR — PE1 route-policy for VRF CUSTOMER-A
! IOS-XR 7.5+: use extcommunity-set color; older: extcommunity-set opaque
extcommunity-set color COLOR-100
  100
end-set
!
route-policy SET-COLOR-100
  set extcommunity color COLOR-100
  pass
end-policy
!
router bgp 65000
 vrf CUSTOMER-A
  neighbor 10.0.0.15
   address-family ipv4 unicast
    route-policy SET-COLOR-100 in
   !
  !
 !
!
```

```
! Junos — PE1 policy for VRF CUSTOMER-A
set policy-options community COLOR-100 members color:0:100
set policy-options policy-statement SET-COLOR-100 term 1 then community add COLOR-100
set policy-options policy-statement SET-COLOR-100 term 1 then accept

set routing-instances CUSTOMER-A protocols bgp group CE1 import SET-COLOR-100
```

### Validation

```
! IOS-XR
show segment-routing traffic-eng policy all          ! Both policies UP
show segment-routing traffic-eng policy color 100    ! Explicit path details
show segment-routing traffic-eng forwarding policy STEER-TO-PE2-VIA-P1
show cef vrf CUSTOMER-A <CE2-prefix>                 ! Should show BSID 24001

! Trace the actual path
traceroute sr-mpls 10.255.0.12/32 policy color 100  ! Should go PE1→P2→P1→P3→PE2

! Junos
show spring-traffic-engineering lsp all
show spring-traffic-engineering lsp name STEER-TO-PE2-VIA-P1 detail
show route table CUSTOMER-A.inet.0 <CE2-prefix> extensive  ! Check next-hop
```

### Expected Results

- Policy `STEER-TO-PE2-VIA-P1` (color 100): UP, 4-label stack
- Policy `STEER-TO-PE2-AVOID-P2P3` (color 200): UP, 2-label stack
- VRF traffic to CE2 prefixes uses BSID 24001 → forwarded via P2→P1→P3→PE2
- Traceroute confirms the non-shortest path through P1
- Without the policy, traffic would take the direct PE1→P2→PE2 or PE1→P3→PE2 path

---

## Exercise 4: Flex-Algo — Latency-Optimized Topology (45 min)

**Goal**: Deploy Flex-Algorithm 128 with latency-based metric to compute an independent topology overlay. Traffic steered to Algo 128 follows the lowest-latency path, which may differ from the lowest-IGP-metric path.

### Background

In this topology, all IGP metrics are 10 (equal cost). But the **latency metrics** differ:

- P1↔P2: 5ms, P2↔P3: 5ms, P2↔PE1: 5ms, P2↔PE2: 5ms, P3↔PE2: 5ms
- P1↔P3: **20ms** (simulating a long-haul link)
- P3↔PE1: **15ms** (simulating a sub-optimal path)

With standard IGP metrics, PE1→PE2 has two equal-cost paths. With latency metrics, the optimal path is PE1→P2→PE2 (10ms) vs PE1→P3→PE2 (20ms) vs PE1→P2→P1→P3→PE2 (35ms).

### Tasks

1. Configure delay metric on all IS-IS interfaces (per table above)
2. Define Flex-Algorithm 128 with min-delay objective on all P/PE routers
3. Assign Algo-128 prefix-SIDs on all loopbacks (16501–16512 per table)
4. Verify Algo-128 computes a separate SPF tree
5. Create an SR-TE policy that uses Algo-128 SIDs
6. Compare forwarding between Algo-0 (IGP metric) and Algo-128 (latency)

### IOS-XR Configuration

```
! === All P/PE routers: Define Flex-Algo 128 ===

router isis CORE
 flex-algo 128
  metric-type delay
  advertise-definition
 !
 interface GigabitEthernet0/0/0/0
  address-family ipv4 unicast
   metric 10
  !
 !
 ! (all interfaces keep IGP metric 10)
 !
 interface Loopback0
  address-family ipv4 unicast
   prefix-sid index 1              ! Algo-0 (already configured)
   prefix-sid algorithm 128 index 501  ! Algo-128
  !
 !
!

! === Configure static delay for lab (real networks use TWAMP probes) ===

performance-measurement
 interface GigabitEthernet0/0/0/0
  delay-measurement
   advertise-delay 0               ! Advertise any change (threshold = 0)
  !
 !
 interface GigabitEthernet0/0/0/1
  delay-measurement
   advertise-delay 0
  !
 !
!
! Static delay values via IS-IS link attributes (IOS-XR 7.5+)
router isis CORE
 interface GigabitEthernet0/0/0/0
  address-family ipv4 unicast
   delay metric 5000               ! 5ms in microseconds
  !
 !
 interface GigabitEthernet0/0/0/1
  address-family ipv4 unicast
   delay metric 20000              ! 20ms (P1↔P3, the long-haul link)
  !
 !
!
```

### Junos Configuration

```
! === All P/PE routers: Define Flex-Algo 128 ===

! Flex-Algo definition lives under routing-options on Junos
set routing-options flex-algorithm 128
set routing-options flex-algorithm 128 definition metric-type delay-metric
set routing-options flex-algorithm 128 definition priority 128

! Algo-128 SID (Algo-0 index already set above)
set protocols isis source-packet-routing node-segment ipv4-index 1
set protocols isis source-packet-routing flex-algorithm 128 node-segment ipv4-index 501

! Delay metric — Junos uses te-metric for Flex-Algo delay computation
! Real deployments use TWAMP/Y.1731; static te-metric works for lab
set protocols isis interface ge-0/0/0.0 level 2 te-metric 5000
set protocols isis interface ge-0/0/1.0 level 2 te-metric 20000
```

> **Note**: Real deployments measure delay dynamically with TWAMP or Y.1731. Static delay values are used here for lab determinism.

### Validation

```
! IOS-XR
show isis flex-algo 128                             ! Algo definition + participating nodes
show isis flex-algo 128                             ! Algo definition + participating nodes
show segment-routing local-block                    ! Verify Algo-128 SID allocation
show cef label <algo128-label> detail               ! Forwarding entry (use label from above)

! Compare paths:
show isis route 10.255.0.12/32                      ! Algo-0: ECMP via P2 and P3
show isis route 10.255.0.12/32 flex-algo 128        ! Algo-128: via P2 only (lower latency)

! Junos
show spring-traffic-engineering lsp                 ! SR-TE LSP status
show route table inet.3 10.255.0.12/32 extensive    ! Check Flex-Algo path selection
show route table inet.3 10.255.0.12/32 extensive    ! Check which algo
show route label 16512                              ! Algo-128 label for PE2
```

### Steering Traffic to Algo-128

Use Algo-128 SIDs in an SR-TE policy for latency-sensitive traffic:

```
! IOS-XR — PE1: Latency-optimized policy to PE2

segment-routing
 traffic-eng
  segment-list SL-LATENCY-PE2
   index 10 mpls label 16512       ! PE2 Algo-128 SID (latency-optimal)
  !
  policy LATENCY-TO-PE2
   color 300 end-point ipv4 10.255.0.12
   candidate-paths
    preference 100
     explicit segment-list SL-LATENCY-PE2
    !
   !
  !
 !
!
```

```
! Junos — PE1: Latency-optimized policy to PE2

set protocols source-packet-routing segment-list SL-LATENCY-PE2 hop1 label 16512
set protocols source-packet-routing source-routing-path LATENCY-TO-PE2 to 10.255.0.12
set protocols source-packet-routing source-routing-path LATENCY-TO-PE2 color 300
set protocols source-packet-routing source-routing-path LATENCY-TO-PE2 primary SL-LATENCY-PE2
```

### Expected Results

| Metric Type | PE1→PE2 Best Path | Total Cost | Next-Hops from PE1 |
|-------------|-------------------|------------|---------------------|
| Algo-0 (IGP) | PE1→P2→PE2 or PE1→P3→PE2 | 20 (equal) | ECMP: P2, P3 |
| Algo-128 (delay) | PE1→P2→PE2 | 10ms | P2 only |

- Algo-128 eliminates the P3 path because PE1→P3 = 15ms + P3→PE2 = 5ms = 20ms total
- PE1→P2 = 5ms + P2→PE2 = 5ms = **10ms** total — strictly better
- `show isis route 10.255.0.12/32 flex-algo 128` should show single next-hop via P2
- Label 16512 forwards via the latency-optimal path only

---

## Exercise 5: Troubleshooting Scenarios (30 min)

Inject these faults one at a time. Diagnose and fix each before moving to the next.

### Fault 1: SRGB Mismatch

Change P3's SRGB to 17000–24999 (different from everyone else's 16000–23999).

**Questions:**
- Does IS-IS adjacency break?
- Can PE1 still reach PE2 via P3?
- What does `show isis segment-routing label table` show on P3 vs P1?
- What label does P1 push to reach P3's loopback?

**Key Insight**: Different SRGBs work because each router advertises its own SRGB via IS-IS. When forwarding toward a prefix-SID with index N, each hop computes the **outgoing** label as: `downstream_neighbor_SRGB_base + N`. Example — P1 sends to P3 (index 3) via P2:
- P1 pushes label `P2_SRGB_base + 3` = 16000 + 3 = **16003** (P2's SRGB is unchanged)
- P2 receives 16003, swaps to `P3_SRGB_base + 3` = 17000 + 3 = **17003** (using P3's advertised SRGB)
- P3 receives 17003 → recognizes its own prefix-SID → delivers

IS-IS adjacency does NOT break. Forwarding works because label computation is per-hop using the downstream neighbor's advertised SRGB. The label values just differ at each hop. This is the critical concept from Section 6.1.

### Fault 2: TI-LFA Label Stack Depth

On PE1, set `max-label-stack-depth 1` (if your platform supports it, or simulate by understanding the constraint).

**Questions:**
- Which prefixes lose TI-LFA protection?
- Why do node-protecting backup paths require deeper label stacks?
- What's the minimum stack depth needed for full protection in this topology?

### Fault 3: SR-TE Policy Down

On PE1, create a policy with a segment list that references a non-existent prefix-SID (e.g., label 16099):

```
segment-list BROKEN
  index 10 mpls label 16099
  index 20 mpls label 16012
```

**Questions:**
- What state does the policy show? (Should be DOWN — segment not resolvable)
- What happens to traffic that was steered to this policy's color?
- Does it fall back to IGP, or black-hole? (Answer depends on autoroute vs ODN configuration)

### Fault 4: Flex-Algo Participation

Remove Flex-Algo 128 configuration from P2 only. Leave it on all other routers.

**Questions:**
- Does P2 still forward traffic for Algo-128 SIDs?
- What happens to PE1→PE2 Algo-128 path? (P2 was the optimal latency path)
- Does Algo-128 SPF recompute? What's the new path?
- What does `show isis flex-algo 128` show on P1? (P2 should be missing from participants)

### Fault 5: BFD + TI-LFA Interaction

Disable BFD on P2's link to P3 (one side only — asymmetric BFD).

**Questions:**
- Does BFD session come up? (No — both ends must be configured)
- What's the failure detection time without BFD? (IS-IS holdtime: 30s default on both IOS-XR and Junos — hello 10s × multiplier 3)
- How does this affect TI-LFA? (TI-LFA backup paths are still computed, but activation is delayed)
- What's the real-world impact? (Potentially 30-90 seconds of black hole before FRR kicks in)

---

## Exercise 6: End-to-End Integration (30 min)

**Goal**: Combine everything — VRF + SR-MPLS + TI-LFA + Flex-Algo — into a production-like deployment.

### Tasks

1. Configure VRF `CUSTOMER-A` on PE1 and PE2
   - RD: 65000:100
   - RT import/export: 65000:100
2. Configure eBGP PE-CE with CE1 and CE2
   - CE1 AS 65001, CE2 AS 65002
   - CE1 advertises 192.168.1.0/24, CE2 advertises 192.168.2.0/24
3. Configure MP-BGP VPNv4 between PE1 and PE2 (direct iBGP or via RR — your choice)
4. Verify end-to-end CE1↔CE2 reachability over SR-MPLS transport
5. Steer CUSTOMER-A traffic to use Algo-128 (latency-optimized) path
6. Shut a core link and verify sub-50ms failover with TI-LFA

### IOS-XR Configuration (PE1)

```
vrf CUSTOMER-A
 address-family ipv4 unicast
  import route-target
   65000:100
  !
  export route-target
   65000:100
  !
 !
!
interface GigabitEthernet0/0/0/4
 vrf CUSTOMER-A
 ipv4 address 10.0.0.14 255.255.255.254
!
router bgp 65000
 address-family vpnv4 unicast
 !
 neighbor 10.255.0.12                          ! PE2 loopback
  remote-as 65000
  update-source Loopback0
  address-family vpnv4 unicast
   next-hop-self                                ! PE sets itself as VPN next-hop
  !
 !
 vrf CUSTOMER-A
  rd 65000:100
  address-family ipv4 unicast
  !
  neighbor 10.0.0.15
   remote-as 65001
   address-family ipv4 unicast
    route-policy PASS-ALL in
    route-policy PASS-ALL out
   !
  !
 !
!
route-policy PASS-ALL
  pass
end-policy
!
```

### Junos Configuration (PE1)

```
! Global AS number (required for BGP)
set routing-options autonomous-system 65000

set routing-instances CUSTOMER-A instance-type vrf
set routing-instances CUSTOMER-A interface ge-0/0/4.0
set routing-instances CUSTOMER-A route-distinguisher 65000:100
set routing-instances CUSTOMER-A vrf-target target:65000:100
! vrf-table-label: allocate VPN label for this VRF (required for MPLS VPN forwarding)
set routing-instances CUSTOMER-A vrf-table-label
set routing-instances CUSTOMER-A protocols bgp group CE1 type external
set routing-instances CUSTOMER-A protocols bgp group CE1 peer-as 65001
set routing-instances CUSTOMER-A protocols bgp group CE1 neighbor 10.0.0.15

set protocols bgp group iBGP-VPN type internal
set protocols bgp group iBGP-VPN local-address 10.255.0.11
set protocols bgp group iBGP-VPN family inet-vpn unicast
set protocols bgp group iBGP-VPN neighbor 10.255.0.12
```

### Validation

```
! Full end-to-end test
! From CE1: ping 192.168.2.1 source 192.168.1.1

! IOS-XR (PE1)
show bgp vpnv4 unicast vrf CUSTOMER-A             ! CE2 prefixes learned
show cef vrf CUSTOMER-A 192.168.2.0/24             ! Label stack: VPN label + SR label
show segment-routing traffic-eng policy color 300  ! Latency policy active

! Junos (PE1)
show route table CUSTOMER-A.inet.0
show route table CUSTOMER-A.inet.0 192.168.2.0/24 extensive  ! Check label stack
```

### Final Validation Checklist

| Test | Command | Expected |
|------|---------|----------|
| IS-IS adjacencies | `show isis adjacency` | 7 L2 adjacencies UP |
| SR labels | `show isis segment-routing label table` | 5 prefix-SIDs: 16001, 16002, 16003, 16011, 16012 |
| TI-LFA coverage | `show isis fast-reroute summary` | 100% |
| BFD sessions | `show bfd session` | All UP, 100ms interval |
| SR-TE policies | `show segment-routing traffic-eng policy all` | All UP |
| Flex-Algo 128 | `show isis flex-algo 128` | 5 participating nodes |
| VPN reachability | `ping vrf CUSTOMER-A 192.168.2.1` | Success |
| Failover | Shut link, measure loss | ≤3 packets at 10ms interval |

---

## Stretch Goals

1. **Add PCE**: Deploy a PCE (Cisco XTC or open-source PathMan) and convert the explicit SR-TE policies to PCE-delegated paths. Verify PCEP session establishment and path computation.

2. **Flex-Algo 129 (Affinity)**: Define a second Flex-Algo that avoids "long-haul" links (using admin groups / affinities). Assign affinity colors to links and verify independent topology.

3. **Microloop Avoidance**: Enable SR microloop avoidance (local delay or RIB update delay) and demonstrate how it prevents transient loops during convergence.

4. **SRv6 Migration**: Following the patterns from Section 6.6, migrate one PE from SR-MPLS to SRv6 while maintaining connectivity. Verify interworking via the SR-MPLS↔SRv6 gateway pattern.

5. **BGP-LS Export**: Enable BGP-LS on all routers, export the topology to an external controller (or just verify the BGP-LS table), and correlate the link-state database with the IS-IS LSDB.

## Platform Notes

This lab works on:
- **EVE-NG / GNS3**: Cisco IOS-XRv9000 (7.x+), Juniper vMX 21.x+
- **Containerlab**: `vrnetlab` images for IOS-XR, Junos
- **Cisco CML**: IOS-XRv9k (requires 8GB+ RAM per node)
- **Juniper vLabs**: Free vMX access (labs.juniper.net)

**Minimum hardware per node:**
- IOS-XRv9000: 4 vCPU, 8GB RAM, 50GB disk
- vMX: 2 vCPU (RE) + 2 vCPU (PFE), 4GB RAM
- Total lab: ~48GB RAM recommended (7 nodes)

See `LAB-ENVIRONMENT.md` in project root for platform setup instructions.

---

*Estimated total time: 3.5–4 hours*
*Difficulty: Advanced (IE-SP / IE-SP level)*
*Covers: Sections 6.1 (SR-MPLS), 6.2 (SR-TE), 6.3 (TI-LFA), 6.6 (Migration concepts), 7.1 (L3VPN basics)*
