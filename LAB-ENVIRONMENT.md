# Lab Environment Specifications

## Platform Options

### Primary: EVE-NG (Recommended)
- **Why**: Runs on Proxmox or similar hypervisor, supports multi-vendor
- **Host**: Dedicate a Proxmox VM — 32GB RAM, 8 vCPU, 100GB disk minimum
- **Nested virt**: Must enable in Proxmox VM settings

### Alternative: Cisco CML
- Better native IOS-XR support
- Licensing cost (~$200/yr personal)
- Runs on same Proxmox infra

### Lightweight: Containerlab
- Docker-based, fast spin-up
- Great for SR Linux (Nokia) and FRR
- Limited IOS-XR/Junos support (needs licensed images)

---

## Required Images

| Vendor | Image | Version | RAM/Node | Use Cases |
|--------|-------|---------|----------|-----------|
| Cisco | IOS-XRv 9000 | 7.x+ | 4GB | Core/PE routers, SR, BGP, MPLS |
| Cisco | IOS-XE (CSR1000v) | 17.x | 3GB | CE routers, basic L3 |
| Juniper | vMX | 23.x+ | 4GB | Core/PE, BGP, MPLS, EVPN |
| Juniper | vSRX | 23.x | 2GB | CE/edge |
| Nokia | SR Linux | latest | 1GB | Modern SR, EVPN (Containerlab) |
| FRR | FRRouting | latest | 256MB | Lightweight route sims |

### Optional
| Vendor | Image | Version | RAM/Node | Use Cases |
|--------|-------|---------|----------|-----------|
| Cisco | NX-OS (Nexus9300v) | 10.x | 4GB | DC/DCI scenarios |
| Arista | vEOS | 4.3x | 2GB | DC/DCI alternative |

---

## Reference Topologies

### Topo 1: Small SP Core (Modules 2-4)
```
         [P1]---[P2]
        / |  \  / |  \
     [PE1] [PE2] [PE3] [PE4]
       |     |     |     |
     [CE1] [CE2] [CE3] [CE4]
```
- 2 P routers, 4 PE routers, 4 CE routers
- IS-IS L2 core, LDP or SR-MPLS
- RAM: ~24GB total (6x XRv + 4x XE)

### Topo 2: Multi-AS (Modules 3, 7)
```
  [AS65001]----[AS65002]----[AS65003]
   PE1-P1-PE2   PE3-P2-PE4   PE5-P3-PE6
```
- 3 autonomous systems, inter-AS VPN testing
- RAM: ~36GB total

### Topo 3: Full SP (Modules 5-8)
```
        [RR1]     [RR2]
         |  \    /  |
   [PE1]-[P1]--[P2]-[PE4]
   [PE2]-[P3]--[P4]-[PE5]
         [PE3]  [PE6]
```
- 4 P routers, 6 PE routers, 2 RRs
- TE, SR, L3VPN, EVPN
- RAM: ~48GB (need beefy Proxmox node)

### Topo 4: Containerlab Quick (Any module)
```yaml
# containerlab topology for quick SR Linux labs
name: sp-quick
topology:
  nodes:
    pe1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
    pe2:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
    p1:
      kind: nokia_srlinux
      image: ghcr.io/nokia/srlinux:latest
  links:
    - endpoints: ["pe1:e1-1", "p1:e1-1"]
    - endpoints: ["pe2:e1-1", "p1:e1-2"]
    - endpoints: ["pe1:e1-2", "pe2:e1-2"]
```

---

## Lab File Convention

Each lab goes in `labs/` with this naming:
```
labs/
  mod02-lab01-isis-multiarea.md      # Instructions
  mod02-lab01-isis-multiarea/        # Configs directory
    topology.yaml                     # EVE-NG or Containerlab topo
    pe1-initial.cfg
    pe2-initial.cfg
    ...
    pe1-solution.cfg                  # Solution configs
```

---

## Quick Start Checklist

- [ ] EVE-NG VM deployed on Proxmox
- [ ] IOS-XRv 9000 image uploaded
- [ ] vMX image uploaded
- [ ] CSR1000v image uploaded
- [ ] Containerlab installed (for quick labs)
- [ ] SR Linux pulled (`docker pull ghcr.io/nokia/srlinux`)
- [ ] Management network configured (OOB)
- [ ] SSH keys distributed to lab nodes
