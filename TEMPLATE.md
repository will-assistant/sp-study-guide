# Section Template

Use this for every sub-section (e.g., `2.1-isis-deep-dive.md`).

---

```markdown
# {Module}.{Section} — {Title}

## Overview
> One-paragraph summary. Why does this matter in an SP network?

## Prerequisites
- List prior sections/knowledge needed

## Theory

### Concepts
- Core protocol/feature mechanics
- Packet/frame formats where relevant
- Control plane vs data plane behavior

### Architecture
- Where this fits in the SP topology
- Scale considerations
- Failure domains and blast radius

### Design Considerations
- When to use (and when NOT to)
- Trade-offs and gotchas
- Vendor differences worth noting

## Configuration

### IOS-XR
```cisco
! Annotated config snippet
```

### Junos
```junos
# Annotated config snippet
```

### Nokia SR-OS (optional)
```nokia
# Annotated config snippet
```

### Key Knobs
| Parameter | Default | Recommended | Why |
|-----------|---------|-------------|-----|
| example   | value   | value       | reason |

## Verification & Monitoring

### Show Commands
| Command (IOS-XR) | Command (Junos) | What It Shows |
|-------------------|-----------------|---------------|
| `show ...`        | `show ...`      | description   |

### What Good Looks Like
- Expected states and outputs
- Key fields to check

### Telemetry Paths
- YANG model paths for streaming telemetry (if applicable)

## Troubleshooting

### Common Issues
| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| symptom | cause       | fix |

### Methodology
1. Step-by-step triage approach
2. Tools and commands in order
3. Escalation criteria

### War Stories
> Real-world scenario illustrating a non-obvious failure mode.

## Lab Exercise
- **Topology**: Reference to lab file in `labs/`
- **Objective**: What the student should achieve
- **Validation**: How to confirm success
- **Stretch Goal**: Advanced variation

## Quick Reference
- Key RFCs: RFC XXXX
- Key YANG models: `model-name`
- Cert relevance: IE-SP objective mapping

## Review Questions
1. Scenario-based question
2. Design decision question
3. Troubleshooting question

---
*Sources: [list references used]*
```
