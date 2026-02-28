# Lab 2.1 — Multi-Area IGP: IS-IS and OSPF Side by Side

## Objective
Build a service provider backbone running IS-IS L1/L2 hierarchy, then configure OSPF on a parallel set of interfaces (ships-in-the-night), and practice controlled migration from OSPF to IS-IS.

## Topology

```
                          ┌──────────┐
                          │   P1     │
                          │ (Core)   │
                          │ L2 only  │
                          └────┬─────┘
                               │ ge-0/0/0
                               │ 10.0.0.0/31
                          ge-0/0/0
                     ┌─────────┴──────────┐
                     │                    │
                ┌────┴─────┐        ┌─────┴────┐
                │   P2     │        │    P3    │
                │ (Core)   │────────│  (Core)  │
                │ L2 only  │ge1  ge1│ L2 only  │
                └────┬─────┘        └─────┬────┘
                     │ 10.0.0.4/31        │ 10.0.0.6/31
                     │                    │
                ┌────┴─────┐        ┌─────┴────┐
                │   PE1    │        │   PE2    │
                │ L1/L2    │        │  L1/L2   │
                │ Area 49.0001      │ Area 49.0002
                └────┬─────┘        └─────┬────┘
                     │                    │
                ┌────┴─────┐        ┌─────┴────┐
                │   CE1    │        │   CE2    │
                │ (Access)  │        │ (Access) │
                │ L1 only  │        │ L1 only  │
                └──────────┘        └──────────┘
```

### Device Roles

| Device | Role | IS-IS Level | Loopback | NET |
|--------|------|-------------|----------|-----|
| P1 | Core | L2 | 10.255.0.1/32 | 49.0000.0100.2550.0001.00 |
| P2 | Core | L2 | 10.255.0.2/32 | 49.0000.0100.2550.0002.00 |
| P3 | Core | L2 | 10.255.0.3/32 | 49.0000.0100.2550.0003.00 |
| PE1 | Edge | L1/L2 | 10.255.0.11/32 | 49.0001.0100.2550.0011.00 |
| PE2 | Edge | L1/L2 | 10.255.0.12/32 | 49.0002.0100.2550.0012.00 |
| CE1 | Access | L1 | 10.255.0.101/32 | 49.0001.0100.2550.0101.00 |
| CE2 | Access | L1 | 10.255.0.102/32 | 49.0002.0100.2550.0102.00 |

### Link Addressing

| Link | Subnet | Device A IP | Device B IP |
|------|--------|-------------|-------------|
| P1–P2 | 10.0.0.0/31 | .0 | .1 |
| P1–P3 | 10.0.0.2/31 | .2 | .3 |
| P2–P3 | 10.0.0.4/31 | .4 | .5 |
| P2–PE1 | 10.0.0.6/31 | .6 | .7 |
| P3–PE2 | 10.0.0.8/31 | .8 | .9 |
| PE1–CE1 | 10.0.0.10/31 | .10 | .11 |
| PE2–CE2 | 10.0.0.12/31 | .12 | .13 |

## Exercises

### Exercise 1: IS-IS Multi-Level Deployment (45 min)

**Goal**: Build a working IS-IS topology with L1/L2 hierarchy.

**Tasks**:
1. Configure IS-IS on all devices with the NETs specified above
2. Set P1, P2, P3 as L2-only
3. Set PE1, PE2 as L1/L2 (area boundary routers)
4. Set CE1, CE2 as L1-only
5. Use wide metrics on all nodes
6. Enable point-to-point on all inter-router links
7. Set overload-on-startup 300 on all P/PE routers

**Validation**:
```
# Verify adjacencies
show isis adjacency          # All expected adjacencies UP
show isis neighbors detail   # Confirm L1/L2 roles correct

# Verify LSDB
show isis database           # Check LSP count per level
show isis database detail    # Verify TLV contents

# Verify routing
show route isis              # All loopbacks reachable
ping 10.255.0.X source 10.255.0.Y   # End-to-end reachability
```

**Expected Results**:
- P1↔P2, P1↔P3, P2↔P3: L2 adjacencies
- P2↔PE1: L2 adjacency (PE1 is L1/L2, P2 is L2-only → forms L2)
- P3↔PE2: L2 adjacency
- PE1↔CE1: L1 adjacency
- PE2↔CE2: L1 adjacency
- CE1 should see a default route from PE1 (L1 attached bit)
- All loopbacks reachable from all nodes

### Exercise 2: IS-IS Convergence Tuning (30 min)

**Goal**: Optimize IS-IS for fast convergence.

**Tasks**:
1. Configure BFD on all inter-router links (interval 100ms, multiplier 3)
2. Tune SPF timers: initial-wait 50ms, secondary-wait 200ms, max-wait 5000ms
3. Tune LSP generation: initial-wait 50ms, secondary-wait 200ms, max-wait 5000ms
4. Enable TI-LFA on all interfaces
5. Verify FRR backup paths are computed

**Validation**:
```
# BFD
show bfd session              # All sessions UP
show bfd session detail       # Confirm interval/multiplier

# Convergence
show isis fast-reroute summary    # TI-LFA coverage percentage
show isis route <prefix> detail   # Check backup path exists

# Test: Shut an interface, measure convergence
# (Use timestamps on ping to estimate)
```

### Exercise 3: Ships-in-the-Night Migration (45 min)

**Goal**: Add OSPF alongside IS-IS, then migrate traffic from OSPF to IS-IS.

**Tasks**:

**Phase 1: Add OSPF**
1. Configure OSPF Area 0 on all P-router links (P1↔P2, P1↔P3, P2↔P3)
2. Configure OSPF Area 1 on P2↔PE1, PE1↔CE1
3. Configure OSPF Area 2 on P3↔PE2, PE2↔CE2
4. Make Area 1 and Area 2 totally stubby
5. All loopbacks in OSPF
6. Verify OSPF neighbors and full routing table

**Phase 2: Verify Dual-Stack**
1. Confirm both IS-IS and OSPF routes exist in RIB
2. Identify which protocol is active for each prefix (check AD/preference)
3. Document the current active protocol per-prefix

**Phase 3: Controlled Cutover**
1. Adjust IS-IS distance/preference to be preferred over OSPF
2. Verify traffic shifts to IS-IS paths
3. Confirm no routing loops or black holes
4. Monitor for 5 minutes

**Phase 4: Remove OSPF**
1. Remove OSPF configuration from all nodes
2. Verify no route loss
3. Confirm IS-IS-only operation

**Validation at Each Phase**:
```
# Phase 2: Which IGP is active?
show route 10.255.0.1/32 detail    # Check protocol & preference

# Phase 3: Confirm shift
show route isis                     # IS-IS routes should be active
show route ospf                     # OSPF routes should be backup/inactive

# Phase 4: Clean state
show route isis                     # All routes present
show protocols ospf                 # Should be empty/unconfigured
```

### Exercise 4: Troubleshooting Scenarios (30 min)

Inject these faults one at a time and diagnose:

1. **Fault 1**: Misconfigure the NET on PE1 (wrong area ID). What symptoms appear? How do L1 and L2 adjacencies change?

2. **Fault 2**: Set a metric of 16777214 on the P1↔P2 link. What happens to traffic flow? Does TI-LFA handle it correctly?

3. **Fault 3**: Enable IS-IS authentication on P2 but not P1. What adjacency state do you see? How does it differ between L1 and L2?

4. **Fault 4**: Set `overload-bit` on P2. What happens to transit traffic? Which routes are still reachable via P2?

## Stretch Goals

1. **Add SR-MPLS**: Enable Segment Routing with prefix SIDs on all loopbacks. Verify label programming and forwarding.
2. **Multi-Topology**: Configure IPv6 addresses and enable IS-IS multi-topology. Verify independent IPv4/IPv6 SPF computation.
3. **Flex-Algo**: Define Flex-Algorithm 128 with a latency metric. Assign it to a subset of nodes. Verify independent topology computation.

## Platform Notes

This lab works on:
- **EVE-NG / GNS3**: Cisco IOS-XRv, Juniper vMX/vSRX, Nokia vSR
- **Containerlab**: `vrnetlab` images for IOS-XR, Junos, SR-OS
- **Cisco CML**: IOSv, IOS-XRv9k, CSR1000v
- **Juniper vLabs**: Free vMX access (labs.juniper.net)

See `LAB-ENVIRONMENT.md` in project root for platform setup instructions.

---
*Estimated total time: 2.5-3 hours*
*Difficulty: Intermediate-Advanced (intermediate SP level)*
