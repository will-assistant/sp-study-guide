# Lab 9.6 — Optical Path Planning and Coherent Pluggable Deployment

## Objective

Design a realistic multi-span DWDM network from scratch. You'll compute span loss budgets, calculate end-to-end OSNR, select modulation formats based on reach requirements, configure coherent pluggables (400ZR/ZR+) on IOS-XR and Junos routers, and troubleshoot optical impairments. This lab bridges the gap between "I understand DWDM theory" and "I can actually design an optical path."

By the end, you'll be able to hand a fiber vendor a spec sheet and configure the router side of a coherent DWDM link — which is exactly what SP backbone engineers increasingly need to do as IPoDWDM replaces dedicated transponders.

## Prerequisites

- Completed Sections 9.1–9.5 (Transport Hierarchy through Coherent Optics)
- Understanding of dB math (power addition, loss subtraction)
- Familiarity with OSNR concepts and coherent modulation formats
- Module 4 (MPLS) and Module 6 (SR) for overlay config context

## Scenario

**TelcoWest** is deploying a new backbone connecting three cities. You're the IP/optical design engineer responsible for:

1. Validating the fiber plant meets optical requirements
2. Selecting the right pluggable technology per link
3. Configuring coherent optics on routers
4. Building a monitoring baseline
5. Diagnosing simulated optical failures

The network serves a mix of enterprise VPN, 5G backhaul, and content delivery — requiring 400G per wavelength minimum, with a growth path to 800G.

## Network Topology

```
                          680 km (8 spans)
        ┌──────────────────────────────────────────────┐
        │              LONG-HAUL LINK                   │
        │                                              │
   ┌────┴────┐     ┌─────┐     ┌─────┐          ┌────┴────┐
   │ Phoenix │     │ ILA │     │ ILA │    ...    │ Denver  │
   │  (PHX)  │─────│  1  │─────│  2  │──────────│  (DEN)  │
   │NCS-5700 │     │EDFA │     │EDFA │          │NCS-5700 │
   │ +ZR+    │     └─────┘     └─────┘          │ +ZR+    │
   └────┬────┘                                   └────┬────┘
        │ 120 km                                      │ 95 km
        │ (2 spans)                                   │ (2 spans)
        │ METRO                                       │ METRO
   ┌────┴────┐                                   ┌────┴────┐
   │ Tucson  │                                   │ C.Sprgs │
   │  (TUS)  │                                   │  (COS)  │
   │  ASR9K  │                                   │  ASR9K  │
   │ +400ZR+ │                                   │ +400ZR+ │
   └─────────┘                                   └─────────┘
```

### Physical Plant Details

| Segment | Route | Distance | Spans | Avg Span | Fiber Type | Splice+Conn Loss |
|---------|-------|----------|-------|----------|------------|-----------------|
| PHX–DEN (Long-haul) | I-40/I-25 corridor | 680 km | 8 | 85 km | G.654.E (ULL) | 0.5 dB/span |
| PHX–TUS (Metro) | I-10 corridor | 120 km | 2 | 60 km | G.652.D (SMF-28) | 0.3 dB/span |
| DEN–COS (Metro) | I-25 corridor | 95 km | 2 | 47.5 km | G.652.D (SMF-28) | 0.3 dB/span |

### Fiber Parameters

| Parameter | G.652.D (SMF-28) | G.654.E (ULL) |
|-----------|-------------------|---------------|
| Attenuation @ 1550nm | 0.20 dB/km | 0.16 dB/km |
| Chromatic Dispersion | 17 ps/(nm·km) | 20 ps/(nm·km) |
| PMD coefficient | ≤ 0.1 ps/√km | ≤ 0.04 ps/√km |
| Effective area | 80 μm² | 125 μm² |
| Zero-dispersion wavelength | ~1310 nm | ~1310 nm |

### Amplifier Specifications

| Parameter | EDFA (ILA sites) | Raman (PHX/DEN terminals) |
|-----------|-------------------|---------------------------|
| Gain range | 15–30 dB | Up to 15 dB (distributed) |
| Noise figure | 5.5 dB | ~4 dB (effective) |
| Max output power | +20 dBm (total) | — |
| Channel count | 96 (C-band, 50 GHz) | — |

### Router Platforms

| Site | Router | Pluggable Slots | Line Card |
|------|--------|----------------|-----------|
| PHX | NCS 5700 (NCS-57C3-MOD) | 2× QSFP-DD | NC57-MPA-12L-S |
| DEN | NCS 5700 (NCS-57C3-MOD) | 2× QSFP-DD | NC57-MPA-12L-S |
| TUS | ASR 9000 (ASR-9903) | 2× QSFP-DD | A99-MPA-4X400GE |
| COS | ASR 9000 (ASR-9903) | 2× QSFP-DD | A99-MPA-4X400GE |

---

## Part 1 — Span Loss Budget Calculations

### Theory Refresher

Span loss = (fiber attenuation × distance) + splice/connector losses + aging margin

Total path loss = sum of all span losses (before amplification)

The amplifiers at each ILA site compensate for the preceding span's loss. The key constraint: each amplifier must have enough gain, and the OSNR at the receiver must exceed the modulation format's threshold.

### Exercise 1.1 — Metro Link: PHX–TUS

Calculate the span loss for each of the two spans on the Phoenix–Tucson metro link.

**Given:**
- Span 1: 65 km, Span 2: 55 km (asymmetric — real fiber routes are never equal)
- Fiber: G.652.D, attenuation 0.20 dB/km
- Splice/connector loss per span: 0.3 dB
- Aging margin: 1.0 dB per span (accounts for cable repairs, temperature)
- No inline amplifiers (point-to-point with amplified terminal equipment)

<details>
<summary><strong>Solution</strong></summary>

**Span 1 (PHX → mid-point):**
```
Fiber loss      = 65 km × 0.20 dB/km = 13.0 dB
Splice/conn     = 0.3 dB
Aging margin    = 1.0 dB
─────────────────────────────────────
Total Span 1    = 14.3 dB
```

**Span 2 (mid-point → TUS):**
```
Fiber loss      = 55 km × 0.20 dB/km = 11.0 dB
Splice/conn     = 0.3 dB
Aging margin    = 1.0 dB
─────────────────────────────────────
Total Span 2    = 12.3 dB
```

**Total path loss (before amplification):**
```
Total = 14.3 + 12.3 = 26.6 dB
```

**Key insight**: With a single mid-span ILA, the ILA amplifier compensates for Span 1's loss. The receiver at TUS only needs to handle Span 2's loss plus any OSNR degradation from the ILA's noise figure. Without an ILA (direct 120 km), total unamplified loss would be 26.6 dB — pushing the limits of 400ZR but within 400ZR+ capability.

</details>

### Exercise 1.2 — Long-Haul Link: PHX–DEN

Calculate the per-span losses for the 8-span Phoenix–Denver long-haul link.

**Given:**
- Span lengths: 90, 85, 80, 88, 82, 92, 78, 85 km (realistic variation)
- Fiber: G.654.E (ULL), attenuation 0.16 dB/km
- Splice/connector loss per span: 0.5 dB (more splices in long-haul)
- Aging margin: 1.5 dB per span
- Raman pre-amp at PHX and DEN terminals (effective NF improvement ~1.5 dB)

<details>
<summary><strong>Solution</strong></summary>

| Span | Length (km) | Fiber Loss (dB) | Splice (dB) | Aging (dB) | Total (dB) |
|------|-------------|------------------|-------------|------------|------------|
| 1 | 90 | 14.4 | 0.5 | 1.5 | 16.4 |
| 2 | 85 | 13.6 | 0.5 | 1.5 | 15.6 |
| 3 | 80 | 12.8 | 0.5 | 1.5 | 14.8 |
| 4 | 88 | 14.1 | 0.5 | 1.5 | 16.1 |
| 5 | 82 | 13.1 | 0.5 | 1.5 | 15.1 |
| 6 | 92 | 14.7 | 0.5 | 1.5 | 16.7 |
| 7 | 78 | 12.5 | 0.5 | 1.5 | 14.5 |
| 8 | 85 | 13.6 | 0.5 | 1.5 | 15.6 |
| **Total** | **680** | **108.8** | **4.0** | **12.0** | **124.8** |

**Key insight**: The worst span (Span 6, 92 km, 16.7 dB) determines the minimum amplifier gain required. All EDFAs must be configured for at least 17 dB gain. The variation between spans (14.5–16.7 dB) means gain tilt management matters — auto-gain-control EDFAs will handle this, but verify per-channel power flatness at commissioning.

**Why G.654.E matters**: If this were G.652.D (0.20 dB/km), Span 6 alone would be 18.4 + 0.5 + 1.5 = 20.4 dB — near the limit of standard EDFAs. The 0.04 dB/km savings per km adds up: 0.04 × 680 = 27.2 dB total savings across the route. That's the difference between needing 7 ILAs and needing 10.

</details>

### Exercise 1.3 — Power Budget Check

For the PHX–TUS metro link using 400ZR pluggables, verify the link closes.

**Given (400ZR specs per OIF IA, worst-case analysis):**
- Tx power: -10 dBm minimum (range: -10 to +2 dBm per OIF; using minimum for worst-case)
- Rx sensitivity: -23 dBm (at pre-FEC BER threshold)
- Rx overload: +1 dBm
- Required OSNR: ~18 dB (DP-16QAM with implementation penalty, 0.1 nm reference bandwidth)

<details>
<summary><strong>Solution</strong></summary>

**Scenario A: No ILA (direct 120 km)**
```
Tx power                    = -10 dBm
Total path loss             = 26.6 dB
Rx power                    = -10 - 26.6 = -36.6 dBm
Rx sensitivity              = -23 dBm
Margin                      = -36.6 - (-23) = -13.6 dB  ← NEGATIVE
```
**Result**: Link does NOT close with 400ZR. The 120 km unamplified path exceeds 400ZR's reach by 13.6 dB.

**Scenario B: Single mid-span ILA**
```
Span 1 loss                 = 14.3 dB
ILA gain (set to match)     = 14.3 dB
ILA output per channel      = -10 dBm (restored to launch power)
Span 2 loss                 = 12.3 dB
Rx power                    = -10 - 12.3 = -22.3 dBm
Rx sensitivity              = -23 dBm
Margin                      = -22.3 - (-23) = 0.7 dB  ← BARELY closes
```
**Result**: Link closes with only 0.7 dB margin. This is too thin for production — you want ≥3 dB margin. This metro link needs **400ZR+ pluggables** (Tx power up to 0 dBm, better FEC) or a second ILA.

**Scenario C: 400ZR+ pluggables, single ILA**
```
Tx power (ZR+)              = -5 dBm (typical ZR+ enhanced power)
Span 1 loss                 = 14.3 dB
ILA gain                    = 14.3 dB
Output per ch               = -5 dBm
Span 2 loss                 = 12.3 dB
Rx power                    = -5 - 12.3 = -17.3 dBm
Rx sensitivity (ZR+)        = -25 dBm (enhanced FEC, oFEC)
Margin                      = -17.3 - (-25) = 7.7 dB  ← Healthy margin
```
**Result**: 400ZR+ with a single ILA gives 7.7 dB margin — production-ready.

**Design decision**: PHX–TUS will use **400ZR+ pluggables** with one mid-span ILA. This gives margin for:
- Future cable repairs (+1-2 dB per repair)
- Temperature-induced attenuation variation (~0.5 dB seasonal)
- Connector degradation over time
- Adding channels (loading penalty ~0.5 dB from XPM/SRS)

</details>

---

## Part 2 — OSNR Calculation and Modulation Format Selection

### Theory Refresher

OSNR degrades at every amplifier because EDFAs add amplified spontaneous emission (ASE) noise. The cascaded OSNR through N amplifiers:

```
1/OSNR_total = Σ(1/OSNR_i)  for i = 1 to N    (linear, not dB)
```

Per-amplifier OSNR (linear):
```
OSNR_i = P_in_i / (h × ν × Δν × NF_i × G_i)

Where:
  P_in_i  = signal power into amplifier i (watts)
  h       = Planck's constant (6.626 × 10⁻³⁴ J·s)
  ν       = optical frequency (~193.1 THz for C-band center)
  Δν      = reference bandwidth (12.5 GHz for 0.1 nm)
  NF_i    = noise figure of amplifier i (linear)
  G_i     = gain of amplifier i (linear)
```

**Shortcut formula** (all spans equal, same amplifier):
```
OSNR_total (dB) ≈ 58 + P_launch(dBm) - NF(dB) - span_loss(dB) - 10×log10(N)

Where:
  58      = derived from h×ν×Δν constant at 193.1 THz, 0.1 nm BW
  N       = number of amplified spans
```

### Exercise 2.1 — OSNR for PHX–DEN Long-Haul

Calculate the end-to-end OSNR for the Phoenix–Denver long-haul link.

**Given:**
- 8 spans, average span loss 15.6 dB (from Exercise 1.2)
- Per-channel launch power into each span: 0 dBm (this is the per-channel EDFA output at full loading — with 80 channels at +20 dBm total, each channel gets ~+1 dBm; we use 0 dBm as the design target). **Always design for fully loaded OSNR, not day-one.**
- EDFA noise figure: 5.5 dB
- Raman pre-amp at DEN reduces effective NF by 1.5 dB for last span

<details>
<summary><strong>Solution</strong></summary>

**Using the shortcut formula for 7 identical EDFA spans + 1 Raman-assisted span:**

EDFA-only spans (1–7):
```
OSNR_7spans = 58 + 0 - 5.5 - 15.6 - 10×log10(7)
            = 58 - 5.5 - 15.6 - 8.45
            = 28.45 dB
```

Raman-assisted span 8 (effective NF = 5.5 - 1.5 = 4.0 dB):
```
OSNR_span8 = 58 + 0 - 4.0 - 15.6
           = 38.4 dB
```

Combining (convert to linear, add reciprocals, convert back):
```
1/OSNR_total = 1/10^(28.45/10) + 1/10^(38.4/10)
             = 1/699.8 + 1/6918.3
             = 0.001429 + 0.000145
             = 0.001574

OSNR_total = 10 × log10(1/0.001574) = 28.03 dB
```

**Result**: End-to-end OSNR ≈ **28.0 dB**

**Modulation format selection** (required OSNR at pre-FEC BER = 4×10⁻²):

| Format | Required OSNR | Capacity | PHX–DEN Viable? |
|--------|---------------|----------|-----------------|
| DP-QPSK | ~10 dB | 200 Gbps @ 64 GBd | ✅ 18 dB margin |
| DP-8QAM | ~15 dB | 300 Gbps @ 64 GBd | ✅ 13 dB margin |
| DP-16QAM | ~18 dB | 400 Gbps @ 64 GBd | ✅ 10 dB margin (production target) |
| DP-32QAM | ~22 dB | 500 Gbps @ 64 GBd | ✅ 6 dB margin |
| DP-64QAM | ~26 dB | 600 Gbps @ 64 GBd | ⚠️ 2 dB margin (risky) |

**Design decision**: DP-16QAM (400G per wavelength) gives 10 dB of OSNR margin — production-grade with room for aging, repairs, and seasonal variation. DP-32QAM is technically feasible but eats into margin. DP-64QAM is not viable over 680 km.

**With PCS** (probabilistic constellation shaping): A PCS-capable ZR+ pluggable could run 500G at ~20 dB required OSNR, giving 8 dB margin. This is the sweet spot for maximizing capacity without sacrificing reliability.

</details>

### Exercise 2.2 — Impact of a Fiber Cut Repair

After one year in service, Span 3 (originally 80 km) suffers a fiber cut at km 45. The repair adds 0.5 dB of splice loss. What's the new OSNR?

<details>
<summary><strong>Solution</strong></summary>

**New Span 3 loss:**
```
Original   = 14.8 dB
Repair     = +0.5 dB
New total  = 15.3 dB
```

Span 3's amplifier must increase gain by 0.5 dB — within the EDFA's dynamic range.

**Recalculate OSNR**: Since the ILA at Span 3 output increases gain but the input power dropped by 0.5 dB, the per-span OSNR for Span 3 degrades slightly. However, the shortcut formula with average span loss:

```
New average span loss = (124.8 + 0.5) / 8 = 15.66 dB

OSNR_7spans = 58 + 0 - 5.5 - 15.66 - 8.45 = 28.39 dB
OSNR_span8  = 38.4 dB (unchanged — Raman-assisted)

OSNR_total ≈ 27.97 dB → ~28.0 dB
```

**Impact**: Negligible (~0.03 dB degradation). This is why we design with margin. A single fiber repair barely dents a well-engineered path.

**When to worry**: If the same span gets repaired 3-4 times, cumulative splice losses of 1.5–2.0 dB start to matter. At that point, consider cable replacement or re-routing.

</details>

### Exercise 2.3 — Channel Loading Impact

The network starts with 10 channels. Capacity demand grows to 80 channels. How does this affect per-channel OSNR?

<details>
<summary><strong>Solution</strong></summary>

**OSNR impact depends on EDFA operating mode.** This is where many engineers get confused.

**Mode 1: Constant total power** (most common in shared line systems)
Total output power is fixed (+20 dBm). Per-channel power depends on loading:
- 10 channels: +10 dBm/ch → OSNR = 58 + 10 - 5.5 - 15.6 - 8.45 = **38.45 dB**
- 80 channels: +1 dBm/ch → OSNR = 58 + 1 - 5.5 - 15.6 - 8.45 = **29.45 dB**
- 96 channels: 0 dBm/ch → OSNR = 58 + 0 - 5.5 - 15.6 - 8.45 = **28.45 dB** (our Exercise 2.1 result)

This is why Exercise 2.1 used 0 dBm/ch — it assumes near-full loading. **Always design for the worst case.**

**Mode 2: Constant per-channel power** (APC — Automatic Power Control per channel)
Each channel is maintained at a target power regardless of loading. OSNR stays constant as channels are added — until the total power hits the amplifier saturation limit.

**Real-world gotchas when going from 10 → 80 channels:**

1. **Nonlinear effects scale with total fiber power**:
   - **SRS (Stimulated Raman Scattering)**: Power tilts from short to long wavelengths (~1–2 dB tilt across C-band at full loading)
   - **XPM (Cross-Phase Modulation)**: Phase noise increases, effectively raising required OSNR by 0.5–1.5 dB
   - **FWM (Four-Wave Mixing)**: Negligible in modern dispersion-uncompensated links with G.654.E (high local dispersion suppresses FWM)

2. **Gain flatness degrades**: EDFAs have wavelength-dependent gain. With 10 channels, you can pick the flat part of the gain curve. With 80 channels, edge channels see 1–2 dB less gain.

3. **Practical mitigation**:
   - Deploy per-channel power equalization at ROADMs
   - Use gain-flattening filters in EDFAs
   - Add L-band amplifiers for >96 channels
   - Launch power optimization (back off slightly to reduce NL penalties)

**Design takeaway**: The fully loaded case is what you design for, not day-one. If you prove the link works with 10 channels and stop there, you'll get a rude surprise when channel 80 has errors.

</details>

---

## Part 3 — Pluggable Selection and Configuration

### Exercise 3.1 — Technology Selection Matrix

Based on your calculations, select the pluggable technology for each link.

<details>
<summary><strong>Solution</strong></summary>

| Link | Distance | Spans | OSNR Budget | Technology | Modulation | Capacity | Justification |
|------|----------|-------|-------------|------------|------------|----------|---------------|
| PHX–TUS | 120 km | 2 | 7.7 dB (ZR+) | 400ZR+ | DP-16QAM | 400G | 120 km with ILA too tight for standard ZR (0.7 dB margin). ZR+ gives 7.7 dB margin. |
| DEN–COS | 95 km | 2 | Ample | 400ZR | DP-16QAM | 400G | Short metro, 400ZR sufficient with single ILA. |
| PHX–DEN | 680 km | 8 | 28 dB (10ch) / 19 dB (80ch) | 400ZR+ | DP-16QAM (PCS optional) | 400G (500G PCS) | Long-haul needs enhanced FEC (oFEC), higher Tx power, tunable grid. ZR+ mandatory. |

**Why not transponders?**
- TelcoWest runs a converged IP+optical network. Dedicated transponders add cost, power, space, and operational complexity.
- ZR/ZR+ pluggables in the router eliminate an entire equipment layer.
- Exception: If you needed >96 wavelengths or needed OTN switching at intermediate sites, transponders/muxponders would return to the design.

**Growth path to 800G**:
- Metro links: Upgrade to 800ZR (DP-16QAM at ~130 GBd) when available — same QSFP-DD800 form factor
- Long-haul: 800G would require DP-16QAM at ~130 GBd, needing ~21 dB OSNR. With 80 channels (19 dB available), 800G won't close on PHX–DEN without either:
  - C+L band (doubling channels across two bands, higher per-channel power)
  - Additional Raman amplification at mid-span ILAs
  - Shorter span design (more ILAs)

</details>

### Exercise 3.2 — IOS-XR Configuration: PHX Router (NCS 5700)

Configure the PHX router's coherent pluggable for the long-haul link to Denver.

**Requirements:**
- 400ZR+ pluggable in QSFP-DD port 0/0/0/0
- Frequency: 193.1 THz (C-band channel 61, 50 GHz grid)
- Modulation: DP-16QAM
- FEC: oFEC (enhanced, required for ZR+)
- Tx power: 0 dBm (into amplified line system)
- Loopback disabled
- Performance monitoring enabled

<details>
<summary><strong>Solution</strong></summary>

```
!! PHX — NCS 5700 Coherent Pluggable Configuration
!! Interface: FourHundredGigE0/0/0/0 (QSFP-DD, ZR+ pluggable)

! --- Layer 1: Optics Controller ---
controller Optics0/0/0/0
 transmit-power 0            ! 0 dBm into amplified line
 dwdm-carrier 100MHz-grid frequency 1931000  ! 193.1 THz (100 MHz units)
 port-mode 400G
 modulation dp-16qam
 fec ofec                    ! oFEC for ZR+ (14.8% overhead, 11.1 dB NCG)
 cd-min -5000                ! CD compensation range: -5000 to +50000 ps/nm
 cd-max 50000                ! Covers ~2500 km of G.654.E
 performance-monitoring enable
 pm 30-sec optics report opr otp osnr cd dgd
 pm 15-min optics report opr otp osnr cd dgd
 pm 24-hour optics report opr otp osnr cd dgd
 alarm-threshold osnr 20     ! Alarm if OSNR drops below 20 dB (2 dB above DP-16QAM threshold with impl. penalty)
!

! --- Layer 2: Coherent DSP Controller ---
controller CoherentDSP0/0/0/0
 performance-monitoring enable
 pm 30-sec fec report ec-bits pre-fec-ber post-fec-ber
 pm 15-min fec report ec-bits pre-fec-ber post-fec-ber
 alarm-threshold pre-fec-ber 1.25e-2   ! Pre-FEC BER alarm (approaching oFEC cliff)
!

! --- Layer 3: Interface ---
interface FourHundredGigE0/0/0/0
 description "PHX-DEN | 400ZR+ | 193.1 THz | Ch61 | Long-haul 680km"
 mtu 9216
 ipv4 address 10.1.0.0/31
 ipv6 address 2001:db8:1::0/127
 carrier-delay up 2000       ! 2 sec hold-up (optical transients)
 carrier-delay down 0        ! Immediate down detection
 load-interval 30
 no shutdown
!

! --- LLDP for neighbor verification ---
lldp
!

! --- Performance Monitoring Thresholds ---
! These create syslog alerts when optical parameters degrade
controller Optics0/0/0/0
 alarm-threshold cd-high 48000   ! CD approaching compensation limit
 alarm-threshold dgd 25          ! DGD >25 ps = PMD concern
!
```

**Key configuration notes:**

1. **Frequency in 100 MHz units**: `1931000` = 193.1 THz. Common mistake: using GHz (193100) or THz (193.1) — IOS-XR expects 100 MHz granularity.

2. **CD range**: Setting `cd-min -5000` and `cd-max 50000` tells the DSP the expected CD range. Over 680 km of G.654.E (20 ps/nm/km), expected CD ≈ 13,600 ps/nm. The DSP compensates automatically, but the range must encompass the actual value or the DSP won't converge.

3. **oFEC vs cFEC**: ZR+ uses oFEC (Open FEC, per OIF IA), not concatenated FEC (cFEC). oFEC provides ~11.1 dB NCG vs ~10.2 dB for hard-decision FEC. This 0.9 dB difference translates to ~50 km additional reach.

4. **Carrier delay**: `up 2000` prevents interface flapping during optical transients (amplifier restarts, protection switches). `down 0` ensures fast failure detection for IGP convergence.

5. **OSNR alarm at 20 dB**: DP-16QAM requires ~18 dB with implementation penalty. Setting the alarm at 20 dB gives 2 dB warning before errors start.

</details>

### Exercise 3.3 — Junos Configuration: DEN Router (Equivalent)

Configure the Denver side in Junos (PTX/ACX/MX with coherent pluggable).

<details>
<summary><strong>Solution</strong></summary>

```
## DEN — Junos Coherent Pluggable Configuration
## Interface: et-0/0/0 (QSFP-DD, ZR+ pluggable)

chassis {
    fpc 0 {
        pic 0 {
            port 0 {
                channel-speed 400g;
            }
        }
    }
}

interfaces {
    et-0/0/0 {
        description "DEN-PHX | 400ZR+ | 193.1 THz | Ch61 | Long-haul 680km";
        mtu 9216;
        optics-options {
            wavelength-frequency 193.10;     /* THz — Junos uses THz directly */
            modulation dp-16qam;
            tx-power 0;                      /* 0 dBm */
            laser-enable;
            fec ofec;
            /* CD compensation auto-detected by DSP — no manual range needed in Junos */
        }
        hold-time up 2000 down 0;
        unit 0 {
            family inet {
                address 10.1.0.1/31;
            }
            family inet6 {
                address 2001:db8:1::1/127;
            }
        }
    }
}

protocols {
    lldp {
        interface et-0/0/0;
    }
    oam {
        ethernet {
            link-fault-management {
                interface et-0/0/0;
            }
        }
    }
}

/* Performance monitoring thresholds */
event-options {
    policy OSNR-DEGRADED {
        events snmp_trap_link_alarm;
        then {
            event-script osnr-alert.slax;
        }
    }
}
```

**IOS-XR vs Junos differences for coherent:**

| Parameter | IOS-XR | Junos |
|-----------|--------|-------|
| Frequency units | 100 MHz (integer: 1931000) | THz (decimal: 193.10) |
| Controller model | Optics + CoherentDSP (separate) | optics-options (unified) |
| CD range | Manual `cd-min`/`cd-max` | Auto-detected |
| FEC config | Under controller | Under optics-options |
| PM config | Granular per-interval | Event-driven + SNMP |
| Alarm thresholds | Per-parameter under controller | Event policies |

</details>

### Exercise 3.4 — Metro Link Configuration: PHX–TUS (400ZR+)

Configure the PHX router's metro link to Tucson using 400ZR+ (needed for the 120 km reach per Exercise 1.3).

<details>
<summary><strong>Solution</strong></summary>

```
!! PHX — Metro Link to Tucson (400ZR+)
!! ZR+ needed: 120 km with single ILA gives only 0.7 dB margin with standard ZR

controller Optics0/0/0/1
 transmit-power -5            ! ZR+ Tx power (-5 dBm, conservative — tunable up to +1 dBm)
 dwdm-carrier 100MHz-grid frequency 1931500  ! 193.15 THz (different channel from DEN link)
 port-mode 400G
 modulation dp-16qam
 fec ofec                     ! oFEC for ZR+ (better coding gain than cFEC)
 performance-monitoring enable
 pm 30-sec optics report opr otp osnr
 alarm-threshold osnr 20     ! Higher threshold OK — metro link has good margin with ZR+
!

controller CoherentDSP0/0/0/1
 performance-monitoring enable
!

interface FourHundredGigE0/0/0/1
 description "PHX-TUS | 400ZR+ | 193.15 THz | Metro 120km"
 mtu 9216
 ipv4 address 10.1.1.0/31
 ipv6 address 2001:db8:1:1::0/127
 carrier-delay up 500        ! Shorter for metro (less optical complexity)
 carrier-delay down 0
 load-interval 30
 no shutdown
!
```

**400ZR vs 400ZR+ configuration differences:**

| Aspect | 400ZR | 400ZR+ |
|--------|-------|--------|
| Tx power | -10 to +2 dBm (OIF range) | Tunable, up to +1 to +3 dBm (vendor-dependent) |
| Modulation | Fixed DP-16QAM | Configurable (QPSK through 16QAM) |
| FEC | Fixed cFEC | Configurable (cFEC or oFEC) |
| Grid | 75 GHz minimum | Flex-grid capable |
| Config complexity | Minimal — it's standardized | More knobs — more things to get wrong |

This is the beauty of 400ZR for metro: plug it in, set the frequency, done. The OIF standard means a Cisco pluggable in slot A talks to a Juniper pluggable in slot B without vendor-specific tuning.

</details>

---

## Part 4 — Optical Monitoring and Baseline

### Exercise 4.1 — Baseline Measurement Collection

After the network is commissioned, collect baseline optical parameters.

**IOS-XR verification commands:**

```
!! Per-interface optical parameters
show controllers optics 0/0/0/0

!! Expected output (annotated):
!!
!!  Controller State: Up
!!  Transport Admin State: In Service
!!
!!  Laser State: On
!!  LED State: Green
!!
!!  Optics Status:
!!    Optics Type: QSFP-DD ZR+
!!    Wavelength: 1552.524 nm / 193.10 THz
!!
!!    Alarm Status:
!!      Detected Alarms: None
!!
!!    Optical Power:
!!      Tx Power: 0.00 dBm          ← Should match configured value
!!      Rx Power: -15.60 dBm        ← Within Rx dynamic range
!!
!!    Optical Quality:
!!      OSNR: 28.1 dB               ← Matches our calculation!
!!      CD: 13600 ps/nm             ← ~680km × 20 ps/nm/km (G.654.E)
!!      DGD: 2.8 ps                 ← Well within DP-16QAM tolerance
!!      PDL: 0.3 dB                 ← Low — good fiber/connector quality
!!
!!    FEC:
!!      FEC Type: oFEC
!!      Pre-FEC BER: 2.1e-3         ← Healthy (threshold ~1.25e-2)
!!      Post-FEC BER: 0.0e0         ← Zero errors after FEC — perfect
!!      EC Bits: 142857142          ← Normal — FEC is doing its job

!! Coherent DSP status
show controllers coherentDSP 0/0/0/0

!! Historical performance monitoring (trending)
show controllers optics 0/0/0/0 pm current 30-sec optics 1
show controllers optics 0/0/0/0 pm history 15-min optics 1

!! All optics summary (quick health check)
show controllers optics brief
```

**Junos equivalent:**

```
## Per-interface optical parameters
show interfaces et-0/0/0 media

## Optics diagnostics
show interfaces diagnostics optics et-0/0/0

## Expected key fields:
##   Laser output power                  :  0.00 dBm
##   Module temperature                  :  45 C
##   Receiver signal average optical power: -15.60 dBm
##   OSNR                                :  28.1 dB
##   Chromatic Dispersion                :  13600 ps/nm
##   Differential Group Delay            :  2.8 ps
##   Pre-FEC BER                         :  2.1e-03
##   Post-FEC BER                        :  0.0e+00

## Historical PM
show interfaces diagnostics optics et-0/0/0 lane
show oam ethernet link-fault-management et-0/0/0
```

### Exercise 4.2 — Build the Baseline Table

Document the day-one baseline. This table becomes your golden reference for troubleshooting.

<details>
<summary><strong>Solution</strong></summary>

| Parameter | PHX→DEN | DEN→PHX | PHX→TUS | TUS→PHX | DEN→COS | COS→DEN |
|-----------|---------|---------|---------|---------|---------|---------|
| Tx Power (dBm) | 0.0 | 0.0 | -10.0 | -10.0 | -10.0 | -10.0 |
| Rx Power (dBm) | -15.6 | -15.2 | -17.3 | -16.8 | -14.5 | -14.2 |
| OSNR (dB) | 28.1 | 28.4 | 34.2 | 34.8 | 36.1 | 36.5 |
| CD (ps/nm) | 13600 | 13600 | 2040 | 2040 | 1615 | 1615 |
| DGD (ps) | 2.8 | 3.1 | 1.2 | 1.1 | 0.9 | 0.8 |
| Pre-FEC BER | 2.1e-3 | 1.8e-3 | 4.2e-5 | 3.8e-5 | 1.1e-5 | 9.8e-6 |
| Post-FEC BER | 0 | 0 | 0 | 0 | 0 | 0 |
| Module Temp (°C) | 45 | 43 | 38 | 37 | 36 | 35 |

**Key observations from baseline:**

1. **CD is always positive at 1550 nm**: At wavelengths above the zero-dispersion point (~1310 nm), chromatic dispersion is positive regardless of propagation direction. PHX→DEN and DEN→PHX both accumulate ~13,600 ps/nm. The DSP compensates automatically. Metro values: PHX→TUS = 120 km × 17 ps/nm/km (G.652.D) = 2,040 ps/nm; DEN→COS = 95 × 17 = 1,615 ps/nm.

2. **DGD asymmetry is normal**: PMD is stochastic (random birefringence in fiber). PHX→DEN (2.8 ps) vs DEN→PHX (3.1 ps) is typical. If DGD suddenly jumps >10 ps, suspect a stressed fiber (tight bend, damaged cable).

3. **Pre-FEC BER correlates with distance**: Long-haul (2.1e-3) >> metro (4.2e-5) >> short metro (1.1e-5). This is expected — more amplifier noise, more fiber impairments.

4. **Module temperature**: Long-haul pluggables run hotter (45°C) than metro (36°C) because the DSP works harder compensating for more CD and noise. If temp exceeds 70°C, check airflow and power budget.

**Store this baseline in your NMS/monitoring system. Set alarms at:**
- Rx power: ±3 dB from baseline
- OSNR: -3 dB from baseline
- Pre-FEC BER: 10× increase from baseline
- DGD: >2× baseline or >25 ps absolute
- Module temp: >65°C warning, >75°C critical

</details>

---

## Part 5 — Troubleshooting Scenarios

### Scenario 5.1 — Gradual OSNR Degradation

**Symptom**: Over 3 months, the PHX→DEN link's OSNR drops from 28.1 dB to 22.5 dB. Pre-FEC BER increases from 2.1e-3 to 8.2e-3. No alarms yet, but the trend is concerning.

**Question**: What's causing this? What's your diagnostic procedure?

<details>
<summary><strong>Solution</strong></summary>

**Step 1: Check if it's both directions**
```
show controllers optics 0/0/0/0    ! PHX→DEN
```
If DEN→PHX OSNR is also degraded → shared infrastructure problem (fiber plant, amplifiers)
If DEN→PHX is fine → transmitter or single-direction fiber issue

**Step 2: Check per-span EDFA performance**
Contact the optical operations team (or check your amplifier management system) for:
- Per-amplifier output power (should be stable)
- Per-amplifier gain (should match span loss)
- ASE noise floor (should not be rising)

**Step 3: Identify the degrading span**
With optical channel monitoring (OCM) at each ILA, check per-channel power at each amplifier output. A single degrading span will show:
- Normal power at ILA inputs before the bad span
- Low power at the ILA input after the bad span
- That ILA's gain will have increased to compensate (if auto-gain)

**Most likely causes of gradual OSNR degradation:**

1. **Connector degradation** (most common): Dust, oxidation, or mechanical stress at patch panels. Fix: clean and re-terminate.

2. **Fiber micro-bend**: New construction near the cable route (seasonal — winter frost heave, summer road work). Fix: re-route or re-bury affected segment.

3. **EDFA aging**: Pump laser degradation reduces gain. The amplifier compensates by increasing pump current, which increases noise figure. Fix: replace pump module.

4. **New channel loading**: If channels were added, per-channel power dropped (Part 2, Exercise 2.3). This is expected and should correlate with provisioning records.

5. **Raman pump degradation**: If the terminal Raman amplifier's pump is weakening, the effective NF improvement drops. This would show as degradation specifically on the last span.

**Step 4: Take action**
```
! Check current vs baseline
show controllers optics 0/0/0/0 pm history 24-hour optics 1

! If connector suspected, request fiber plant maintenance:
! - OTDR trace from each ILA (compare to commissioning trace)
! - Connector inspection with fiber microscope
! - Clean and remate suspect connectors
```

**The 5.6 dB degradation (28.1 → 22.5) most likely cause**: Two spans have degraded — perhaps a connector issue at ILA-3 (adding 2 dB) and EDFA pump aging at ILA-6 (increasing NF by 1.5 dB), compounded by 15 new channels being added (reducing per-channel power by 1.8 dB). Each cause alone is minor; together they eat the margin.

</details>

### Scenario 5.2 — Sudden Link Failure with Fast Recovery

**Symptom**: PHX→DEN goes down for exactly 12 seconds, then recovers. It happens at 2:47 AM. OSNR and power look normal after recovery.

**Question**: What happened?

<details>
<summary><strong>Solution</strong></summary>

**12 seconds + 2:47 AM + clean recovery = optical protection switch or amplifier restart.**

**Most likely: EDFA auto-restart at an ILA site**

ILA amplifiers sometimes restart due to:
- Power supply glitch (UPS transfer)
- Firmware auto-update (yes, some EDFAs have auto-update schedules at odd hours)
- Transient laser safety shutdown (reflected power spike from a connector)

**Timeline of a typical EDFA restart:**
```
T+0s:     EDFA detects fault, shuts down pump laser
T+0-1s:   Downstream spans go dark (no amplification)
T+1-5s:   EDFA runs self-test, pump laser re-ignition
T+5-10s:  Gain stabilization (AGC loop settles)
T+10-12s: Downstream EDFAs re-acquire signal, AGC settles
T+12s:    End-to-end link stable, coherent DSP locks
```

**Why the router interface didn't go down (if carrier-delay up = 2000ms)**:
If the outage was <2 seconds at the router level, the carrier-delay timer would mask it. But 12 seconds exceeds the 2-second hold-up, so the interface would have flapped.

**Check on the router:**
```
show logging | include "0/0/0/0"
! Look for:
!   Interface state change: FourHundredGigE0/0/0/0 -> Down (02:47:xx)
!   Interface state change: FourHundredGigE0/0/0/0 -> Up (02:47:xx)

show controllers optics 0/0/0/0 alarm-log
! Look for: LOS (Loss of Signal) at 02:47
```

**Check the amplifier management system:**
- Which ILA restarted? The first ILA downstream of the restart will show the earliest LOS alarm.
- Was it a planned restart? Check the ILA event log for firmware update or configuration push.

**What to do:**
1. If it was a one-time power glitch: note it, monitor, no action needed
2. If it repeats: investigate the ILA site's power infrastructure
3. Add the event to the maintenance log with OTDR baseline comparison
4. If customer-affecting: report the 12-second outage per SLA terms

**The IGP (IS-IS/OSPF) impact:**
With BFD at 300ms × 3 = 900ms detection, the BFD session would have timed out at ~T+1s. IS-IS would reconverge traffic to alternate paths within 50ms of BFD-down (if TI-LFA is deployed — which it should be from Lab 6.7). Customer traffic disruption: ~1 second, not 12. The 12 seconds is just how long it takes the optical link to fully recover.

</details>

### Scenario 5.3 — Wavelength Conflict After ROADM Change

**Symptom**: After a fiber maintenance window, the PHX→DEN link shows -40 dBm Rx power (was -15.6 dBm). The link is effectively down. PHX→TUS is fine.

**Question**: Diagnose and fix.

<details>
<summary><strong>Solution</strong></summary>

**-40 dBm = no usable signal. The wavelength is being blocked or misrouted.**

**Step 1: Verify Tx is still transmitting**
```
show controllers optics 0/0/0/0
! Check: Tx Power should still show 0.0 dBm
! If Tx is 0 dBm but Rx is -40 dBm, the problem is in the fiber plant
```

**Step 2: Check the maintenance change log**
What was done during the maintenance window? Common culprits:
- ROADM express path was changed (wavelength 193.1 THz dropped from the pass-through)
- Fiber patch panel was re-arranged (wrong port)
- Amplifier was replaced and came up in a different gain mode

**Step 3: Verify wavelength provisioning on ROADMs**
At each ROADM/ILA between PHX and DEN, verify that channel 61 (193.1 THz) is in the pass-through or express path. A single ROADM blocking the channel will kill the entire path.

**Most likely cause: Channel 61 was dropped from a ROADM's cross-connect during maintenance.**

Someone was working on a different channel but accidentally included channel 61 in a batch operation. Or the ROADM firmware was updated and the cross-connect table didn't persist correctly.

**Step 4: Fix**
```
! On the ROADM (vendor-specific CLI — example for a Ciena 6500):
! 1. Check channel plan:
show wavelength-plan

! 2. Verify channel 34 express path:
show cross-connect channel 61

! 3. If missing, re-provision:
wavelength-plan express channel 61 input-port LINE-WEST output-port LINE-EAST
commit
```

**Step 5: Verify recovery on the router**
```
show controllers optics 0/0/0/0
! Rx power should return to ~-15.6 dBm within 10-30 seconds
! OSNR should match baseline
! Pre-FEC BER should return to ~2.1e-3
```

**Lessons:**
1. Always verify the **full wavelength plan** after maintenance, not just the channels you touched
2. ROADM changes should include a post-maintenance optical power sweep (automated in good OSS systems)
3. If your router has telemetry streaming, you'd catch this in seconds:
   ```
   ! gNMI subscription for real-time optical monitoring
   sensor-group OPTICS
    sensor-path openconfig-platform:components/component/optical-channel/state
   !
   subscription OPTICAL-HEALTH
    sensor-group-id OPTICS sample-interval 10000  ! 10 seconds
   ```

</details>

### Scenario 5.4 — Pre-FEC BER Oscillation

**Symptom**: The PHX–TUS metro link shows pre-FEC BER oscillating between 1e-5 and 3e-3 with a ~6 hour period. OSNR is stable. Temperature is stable. It started 2 weeks ago.

**Question**: What's causing the oscillation?

<details>
<summary><strong>Solution</strong></summary>

**Periodic BER oscillation with stable OSNR + stable temperature = polarization-dependent effect.**

The 6-hour period is the key clue. Common causes of periodic optical impairment:

1. **Solar heating cycle** (12h/24h period): Fiber on aerial spans or in shallow conduit expands/contracts with temperature, changing PMD and birefringence. But our fiber is buried (I-10 corridor), and the period is 6 hours, not 24.

2. **Tidal/groundwater effects** (12.4h period): Coastal/riverside routes see periodic ground pressure changes. I-10 through the Arizona desert = not this.

3. **Traffic vibration** (rush hour pattern): If fiber runs along a highway or rail corridor, heavy traffic causes micro-vibrations that modulate birefringence. I-10 between Phoenix and Tucson carries significant truck traffic with peaks at ~6 AM, ~noon, ~6 PM, ~midnight (trucking schedules).

**Most likely: Mechanical vibration from I-10 truck traffic modulating fiber birefringence.**

The coherent DSP compensates for slowly-varying PMD, but rapid fluctuations (vibration-induced) can exceed the DSP's tracking speed, causing momentary BER spikes.

**Verification:**
```
show controllers optics 0/0/0/1 pm history 24-hour optics 1
! Plot DGD (Differential Group Delay) over 24 hours
! If DGD correlates with BER oscillation, it's PMD/vibration

! Also check PDL (Polarization-Dependent Loss)
! Rapid PDL fluctuation = birefringence changes
```

**Fix options:**
1. **No action needed** if post-FEC BER stays at zero (the FEC handles it)
2. If post-FEC errors appear: increase the DSP's polarization tracking speed (vendor-specific, may require pluggable firmware update)
3. Long-term: work with the fiber provider to identify the specific cable segment near the vibration source and request conduit reinforcement or re-route

**Why OSNR is stable but BER oscillates:**
OSNR measures average noise power in a bandwidth around the channel. It doesn't capture fast polarization effects. The pre-FEC BER is more sensitive because it sees instantaneous bit errors from polarization transients that the OSNR measurement averages out.

**This is a real-world scenario**: Many SP engineers see periodic BER patterns and assume it's an equipment problem. It's almost always environmental — temperature, vibration, or water. Understanding this saves weeks of troubleshooting.

</details>

---

## Part 6 — Wavelength Planning

### Exercise 6.1 — C-Band Channel Plan

Design the initial channel plan for the PHX–DEN long-haul link.

**Requirements:**
- 10 channels initially, expandable to 80
- 50 GHz spacing (standard grid per ITU-T G.694.1)
- Avoid band edges (channels 1–4 and 93–96 have higher EDFA noise figure)
- Guard band around 1550.12 nm (OSC channel)

<details>
<summary><strong>Solution</strong></summary>

**C-band channel numbering** (50 GHz grid, ITU-T G.694.1):
```
Channel 1:  196.10 THz  (1528.77 nm) ← High-frequency edge
Channel 96: 191.35 THz  (1566.72 nm) ← Low-frequency edge
Center:     193.725 THz (1547.72 nm)
```

**OSC (Optical Supervisory Channel)**: Typically at 1510 nm or 1625 nm (out-of-band). If OSC is at 1550.12 nm (legacy design), place a guard band at channels matching ~193.21 THz.

**Initial 10-channel allocation** (spread across band for even EDFA loading):

| Channel # | Frequency (THz) | Wavelength (nm) | Use |
|-----------|-----------------|-----------------|-----|
| 15 | 195.40 | 1534.25 | PHX–DEN core |
| 23 | 195.00 | 1537.40 | PHX–DEN core |
| 31 | 194.60 | 1540.56 | PHX–DEN core |
| 39 | 194.20 | 1543.73 | PHX–DEN core |
| 47 | 193.80 | 1546.92 | PHX–DEN core |
| 54 | 193.45 | 1549.72 | PHX–DEN core |
| 61 | 193.10 | 1552.52 | PHX–DEN core (primary — our config uses this) |
| 69 | 192.70 | 1555.75 | PHX–DEN core |
| 77 | 192.30 | 1558.98 | PHX–DEN core |
| 85 | 191.90 | 1562.23 | PHX–DEN core |

**Design rationale:**
1. **Spread across the band**: Channels every ~8 slots. This prevents gain ripple from concentrated loading and ensures even SRS (Stimulated Raman Scattering) distribution.
2. **Avoid edges**: Channels 5–90 only. Edge channels (1–4, 93–96) see 1–2 dB higher noise figure.
3. **Growth path**: Fill in between existing channels. Adding 8 channels between each pair gives 80 total at 50 GHz spacing.
4. **Channel 61 as "primary"**: 193.1 THz (1552.52 nm), near C-band center, good EDFA performance, used in our router configs.

**For the metro links (PHX–TUS, DEN–COS):**
Use separate channel allocations to avoid conflicts if these links ever interconnect at a ROADM:
- PHX–TUS: Channels 16, 24, 32, 40 (start with 4)
- DEN–COS: Channels 48, 56, 64, 72 (start with 4)

</details>

### Exercise 6.2 — Flex-Grid Planning (Future 800G)

When TelcoWest upgrades to 800G channels, the spectrum requirements change.

**Question**: How wide is an 800G channel, and how many fit in the C-band with flex-grid?

<details>
<summary><strong>Solution</strong></summary>

**800G channel requirements:**
- Modulation: DP-16QAM at ~130 GBd (single-carrier) or dual-carrier 2×65 GBd
- Single-carrier 130 GBd spectral width: ~137 GHz (including roll-off and guard band)
- Required slot: 150 GHz on flex-grid (12× 12.5 GHz slots)
- With 75 GHz flex-grid: 2 slots per 800G channel

**C-band capacity at 800G:**
```
C-band usable bandwidth: ~4.8 THz (191.3–196.1 THz)

Fixed 50 GHz grid:  Not possible — 800G doesn't fit in 50 GHz
Fixed 75 GHz grid:  64 channels × 400G = 25.6 Tbps (400G only)
Flex-grid 150 GHz:  32 channels × 800G = 25.6 Tbps
Flex-grid 75+150:   Mix of 400G (75 GHz) and 800G (150 GHz)
```

**Migration strategy:**
1. **Day 1 (now)**: 50 GHz grid, 400G per channel, up to 96 channels = 38.4 Tbps
2. **Day 2 (800G available)**: Convert high-demand channels to 800G using flex-grid. Each 800G replaces 3× 50 GHz slots. Net capacity increase only if you have spare spectrum.
3. **Day 3 (C+L band)**: Add L-band amplifiers, doubling spectrum to ~9.6 THz. Run 800G across both bands = ~51 Tbps.

**The flex-grid gotcha**: Upgrading from 50 GHz fixed grid to flex-grid requires ROADM hardware that supports flex-grid WSS (Wavelength Selective Switch). Many deployed ROADMs only support fixed 50/100 GHz. This is a capital expense planning decision that must happen at initial deployment.

**TelcoWest recommendation**: Deploy flex-grid-capable ROADMs now even though you're using 50 GHz initially. The marginal cost increase is ~15%, but retrofitting later requires truck rolls to every ILA site.

</details>

---

## Part 7 — Design Review Checklist

Before signing off on any optical path design, verify every item:

### Physical Layer
- [ ] Fiber type documented per span (G.652.D / G.654.E / G.655)
- [ ] Span losses calculated with aging margin
- [ ] OTDR traces archived for every span (commissioning baseline)
- [ ] Connector types consistent (LC/UPC or LC/APC — never mix)
- [ ] Splice loss budget includes future repairs (2–3 per span over 20 years)

### Optical Layer
- [ ] End-to-end OSNR calculated for fully loaded channel count (not day-one)
- [ ] OSNR margin ≥3 dB above modulation format threshold
- [ ] EDFA gain range covers worst-case span loss + aging
- [ ] Raman required? (spans >90 km on G.652.D or >110 km on G.654.E)
- [ ] SRS tilt calculated for max channel loading
- [ ] PMD budget verified (especially for aerial/legacy fiber)

### Router/Pluggable Layer
- [ ] Pluggable compatible with router line card (check qualified optics list!)
- [ ] Frequency configured correctly (units match platform: MHz vs THz)
- [ ] Tx power matches line system design (too high = nonlinear penalty, too low = OSNR loss)
- [ ] FEC type matches pluggable capability (oFEC for ZR+, cFEC for ZR)
- [ ] CD compensation range set (IOS-XR) or verified auto (Junos)
- [ ] Carrier-delay tuned (up = suppress optical transients, down = fast failure detect)
- [ ] MTU set for jumbo frames (9216 for MPLS/VPN traffic)
- [ ] Performance monitoring enabled with alarm thresholds

### Monitoring
- [ ] Baseline optical parameters documented
- [ ] Telemetry streaming configured (gNMI + sensor paths)
- [ ] Alarm thresholds set (OSNR, Rx power, pre-FEC BER, temperature)
- [ ] Trend dashboards configured in NMS (Grafana/LibreNMS/vendor NMS)
- [ ] Escalation runbook for optical alarms (who to call at 3 AM)

### Documentation
- [ ] Wavelength plan spreadsheet with growth allocation
- [ ] Fiber route diversity analysis (SRLGs mapped)
- [ ] Amplifier site access procedures (keys, contacts, safety)
- [ ] Vendor support contracts cover pluggables AND line system
- [ ] Disaster recovery procedure for cable cut (reroute plan, expected restoration time)

---

## Summary

This lab covered the full lifecycle of optical path planning in a modern SP network:

1. **Span loss budgets** — Calculate per-span and end-to-end loss with realistic parameters
2. **OSNR engineering** — Determine achievable OSNR, select modulation formats, account for channel loading
3. **Technology selection** — Choose between 400ZR and 400ZR+ based on link requirements
4. **Router configuration** — Configure coherent pluggables on IOS-XR and Junos with production-grade parameters
5. **Baseline and monitoring** — Establish golden reference values and alarm thresholds
6. **Troubleshooting** — Diagnose real-world optical failures from symptom to root cause
7. **Wavelength planning** — Design channel plans for current and future capacity

### Key Takeaways

- **Design for the worst case**: Fully loaded channel count, with aging margin, after 3 fiber repairs. If it works then, it'll work day one.
- **400ZR is beautiful in its simplicity**: Fixed modulation, fixed FEC, fixed power. Plug and go. Use it for every metro link where it reaches.
- **400ZR+ is necessary evil for long-haul**: More knobs = more ways to misconfigure. But you need the tunable power, selectable modulation, and oFEC for anything beyond ~120 km.
- **OSNR is king**: Every design decision ultimately comes down to "does the OSNR close?" Learn to calculate it in your head (58 + launch - NF - span_loss - 10log(N)) and you'll never be surprised.
- **Environmental effects are real**: Temperature, vibration, water, and construction affect fiber optics in ways that don't show up in PowerPoint. The best optical engineers think about geography as much as physics.

### What's Next

- **Module 10**: Network Slicing & 5G Transport — how all this physical layer feeds into network programmability
- **Module 11**: Automation & Operations — gNMI streaming, closed-loop optical remediation, zero-touch provisioning

---

## References

- OIF 400ZR Implementation Agreement (OIF-400ZR-01.0)
- OIF 400ZR+ Framework (OIF-400ZR+-01.0)
- ITU-T G.694.1 — Spectral grids for WDM applications: DWDM frequency grid
- ITU-T G.652 — Characteristics of a single-mode optical fibre and cable
- ITU-T G.654 — Characteristics of a cut-off shifted single-mode optical fibre and cable
- ITU-T G.698.2 — Multi-channel DWDM applications with single-channel optical interfaces
- Cisco NCS 5700 Configuration Guide — Coherent Pluggable Optics
- Juniper Junos OS — Configuring Coherent Optics (PTX/ACX/MX Series)
- IEEE 802.3cu — 100/400 Gbps Ethernet over SMF using coherent optics
- Infinera White Paper: "Optical Path Computation for IP-Optical Convergence"
