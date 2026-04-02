# KiCad-Claude — Master Project Document
**Project:** KiCad-Claude Audio Amplifier  
**Last Updated:** 2026-04-02  
**Revision:** 0.3

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [System Architecture](#2-system-architecture)
3. [Module A — 8-Input Mixer](#3-module-a--8-input-mixer)
4. [Module B — Input Buffer](#4-module-b--input-buffer)
5. [Module C — Power Amplifier](#5-module-c--power-amplifier)
6. [Module D — Power Distribution](#6-module-d--power-distribution)
7. [Global Requirements](#7-global-requirements)
8. [Performance Targets](#8-performance-targets)
9. [Design Notes — Technical Rationale](#9-design-notes--technical-rationale)
10. [Design Decisions Log](#10-design-decisions-log)
11. [Open Questions](#11-open-questions)
12. [Workflow & Session Protocol](#12-workflow--session-protocol)

---
---

# SYSTEM SPECIFICATION
*Sections 1–8 form the system specification. Each module section (3–6) can be extracted as a standalone document for review or handoff.*

---

## 1. Project Overview

A bespoke single-channel audio amplifier system designed and built in small batch quantities (< 100 units). The system accepts 8 signals from Cycfi Research Nu modular single-string pickups, mixes them into a main output and two effect send groups, buffers the signal, amplifies it, and drives a speaker load. All major functional blocks are implemented as separate PCBs to maximise clarity and ease of repair.

### Goals
- Clean, high-quality audio signal path from Nu capsule sources to speaker
- Modular PCB architecture — each circuit block developed, tested and documented independently
- 8-input mixing with two independent effect send groups
- Fully documented design with traceable design decisions
- Small batch manufacturable — through-hole and common SMD components preferred

### Out of Scope
- Instrument-level (high impedance) inputs — not designed for, edge case only
- Digital signal processing
- Multi-channel speaker output

---

## 2. System Architecture

### Signal Chain

```
NU CAPSULES (×8)
      │
      ▼
[A — 8-Input Mixer]
  Per channel: Gain Trim knob + Send Assign switch (Send 1 / Send 2 / Both)
  Per send bus: Send 1 Master knob, Send 2 Master knob
      │              │                        │
      │     [Send 1 Master] ──▶ FX Unit 1     │
      │                    [Send 2 Master] ──▶ FX Unit 2
      ▼
[B — Input Buffer]
      │
      ▼
[C — Power Amplifier]
      │
      ▼
SPEAKER OUTPUT (8Ω)

[D — Power Distribution] ──▶ 32V → C | ±15V → A, B | 9/12V → Nu capsules
      ▲
32V DC brick (off-the-shelf external SMPS)
```

### Module Summary

| Module | PCB | Function | Status |
|---|---|---|---|
| A | Mixer PCB | 8 Nu capsule inputs; gain trim + send assign switch per channel; two effect send buses with master level | Architecture defined |
| B | Buffer PCB | Unity gain, high-Z in, low-Z out, line level conditioning | Schematic drafted |
| C | Power Amp PCB | TPA3251 Class D, 70W into 8Ω, 32V single rail | Device selected |
| D | Power Distribution PCB | 32V brick input, protection, ±15V linear regulated, Nu capsule rail | Architecture defined |

---

## 3. Module A — 8-Input Mixer

*This section can be extracted as a standalone MODULE_A_MIXER.md for review or handoff.*

**Status:** Architecture defined — schematic not yet started  
**KiCad File:** TBD  
**Simulation:** TBD

### Overview

The mixer accepts 8 signals from Nu modular single-string pickups (Cycfi Research), combines them into a main mix output (to Module B), and independently routes each channel to two effect send buses. Each channel has a gain trim knob and a 3-position send assign switch (Send 1 / Send 2 / Both). The two effect send buses each have a master level knob. Active summing throughout — passive mixing is not suitable for 8 inputs at these signal levels.

18 controls on the front panel: 8 gain trim knobs, 8 send assign switches, 2 send master knobs.

### Input Sources — Nu Capsules

| Parameter | Value | Notes |
|---|---|---|
| Output voltage | 580–740mV peak-to-peak | ~200–260mVrms — below standard 1Vrms line level |
| Output impedance | 10kΩ | Higher than typical line-level sources |
| Supply voltage | 5V–18V DC | Reverse polarity protected internally |
| Supply current | 450µA per capsule | × 8 = 3.6mA total |
| Connectivity | 2mm pitch PCB header | Mounts directly to PCB — no panel input jacks needed |
| Frequency response | 20Hz–20kHz | Flat, uncoloured |

Nu output (~200–260mVrms) is below standard line level. The active summing stage provides ~+12–14dB gain to normalise to ~1Vrms before Module B.

Nu capsules require a clean regulated 5–18V supply at 3.6mA total. Module D provides a dedicated low-noise rail for this. Voltage TBD (9V or 12V).

### Channel Controls

| Control | Type | Function |
|---|---|---|
| **Gain Trim** | Potentiometer | Sets signal level for this channel — feeds main mix and send buses simultaneously |
| **Send Assign** | SP3T slide switch | Routes channel to Send 1 only / Send 2 only / Send 1+2 |

| Bus Control | Type | Function |
|---|---|---|
| **Send 1 Master** | Potentiometer | Overall output level of Effect Send 1 bus → FX Unit 1 |
| **Send 2 Master** | Potentiometer | Overall output level of Effect Send 2 bus → FX Unit 2 |

### Signal Flow

```
Nu #1 ──[Gain Trim]──┬──────────────────────────────── Main Mix Summing Amp ──▶ Main Out → Module B
                     │
                     └──[Send Assign]──┬──▶ Send 1 Summing Amp ──[Send 1 Master]──▶ Send 1 Out → FX Unit 1
                                       ├──▶ Send 2 Summing Amp ──[Send 2 Master]──▶ Send 2 Out → FX Unit 2
                                       └──▶ Both (centre position)

Nu #2–8 ──▶ (same structure × 8)
```

Three independent active inverting summing amplifiers — main mix, Send 1 bus, Send 2 bus.

### Send Assign Switch — Selected Component

**C&K OS103011MS8QP1** — SP3T, ON-ON-ON, through-hole vertical, PC pin

| Parameter | Value |
|---|---|
| Manufacturer | C&K (Littelfuse) |
| Part number | OS103011MS8QP1 |
| Configuration | SP3T — Single Pole, 3 Throw |
| Switch function | ON-ON-ON — all 3 positions active |
| Positions | 1: Send 1 only / 2: Send 1+2 / 3: Send 2 only |
| Mounting | Through-hole vertical — top actuated through slot in panel ✅ Confirmed |
| Pin pitch | 2mm |
| Travel | 2mm per position |
| Current rating | 100mA at 12VDC |
| Body size | 8.2mm × 7.5mm |
| KiCad footprint | Available via SnapMagic |
| Digikey | OS103011MS8QP1-ND |
| Unit cost | ~$0.50–$0.83 |

> ⚠️ The "Both" centre position connects the pole to a junction node feeding both send summing amp inputs. Must be handled explicitly in PCB routing.

### Specification

| Parameter | Value | Notes |
|---|---|---|
| Inputs | 8 × Nu capsule | Via 2mm pitch PCB header |
| Input signal level | ~200–260mVrms | Nu capsule output |
| Input impedance | TBD | Must suit 10kΩ Nu output impedance |
| Per-channel controls | Gain trim knob + SP3T slide switch | 8 knobs + 8 switches |
| Send master controls | 2 × potentiometer | One per send bus |
| Total panel controls | 18 | 8 trims + 8 switches + 2 masters |
| Main mix output | 1 × ~1Vrms | To Module B |
| Effect send outputs | 2 × line level | Post send-master, to external FX units |
| Output connectors | TBD | 6.35mm jack or inter-board header |
| Summing architecture | Active inverting summing amp | 3 buses |
| Gain | TBD | ~+12–14dB to normalise Nu output |
| Op-amp supply | ±15V | From Module D |
| Nu capsule supply | TBD voltage, ~3.6mA | Dedicated clean rail from Module D |

### Physical Notes

8 columns, each with a gain trim knob and a C&K OS103011MS8QP1 slide switch (top-actuated through panel slot). Send master knobs grouped at one end of the panel. Nu capsules connect directly to PCB headers — no input jacks on the front panel.

Panel slot dimensions must match OS103011MS8QP1 actuator profile — confirm from datasheet before panel fabrication.

### Open Questions — Module A
- [ ] Op-amp selection — NE5532 or TL072 for summing amps?
- [ ] Exact gain setting — confirm target output level at main out
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Does gain trim control level into main mix and sends simultaneously, or main mix only?
- [ ] Output connector types — main out and effect sends
- [ ] Master level control on main output — include or omit?
- [ ] Any clip / level indication per channel or master?
- [ ] PCB dimensions and mechanical format

---

## 4. Module B — Input Buffer

*This section can be extracted as a standalone MODULE_B_BUFFER.md for review or handoff.*

**Status:** Schematic drafted in KiCad — ERC not yet run  
**KiCad File:** `audio-preamp/PreampV1/PreampV1.kicad_sch`  
**Simulation:** Not yet completed

### Overview

Unity-gain voltage follower between Module A (mixer output) and Module C (power amp input). Preserves signal integrity, isolates source from load — high impedance input, very low impedance output.

### Signal Flow

```
Module A Out ──▶ [R1 100Ω] ──▶ [U1 NE5532 voltage follower] ──▶ [R3 100Ω] ──▶ Module C
                                          ↑
                                 ±15V from Module D
                              (decoupled by C1–C4 on PCB)
```

### Specification

| Parameter | Value | Notes |
|---|---|---|
| Gain | 0dB (unity) | Voltage follower — output = input |
| Input impedance | ~100kΩ | Set by R2 bias resistor to GND |
| Output impedance | <1Ω | Op-amp output |
| Supply voltage | ±15V dual rail | From Module D |
| Input connector | TBD | J1 |
| Output connector | TBD | J2 |
| Bandwidth | >100kHz | NE5532 GBW = 10MHz |
| Input-referred noise | ~5nV/√Hz | NE5532 voltage noise |

### Components

| Reference | Value | Package | Function |
|---|---|---|---|
| U1 | NE5532 | DIP-8 | Op-amp — unity gain voltage follower |
| R1 | 100Ω | TH | Input protection |
| R2 | 100kΩ | TH | Input bias to GND — sets input impedance |
| R3 | 100Ω | TH | Output isolation — prevents oscillation with capacitive load |
| C1, C3 | 100nF | TH ceramic | HF decoupling on ±15V rails |
| C2, C4 | 10µF | TH electrolytic | LF decoupling on ±15V rails |

### Status & Next Steps

| Task | Status |
|---|---|
| Component selection | ✅ Complete |
| Schematic (KiCad) | ✅ Draft complete |
| ERC | ⬜ Not yet run |
| LTspice simulation | ⬜ Not yet done |
| PCB layout | ⬜ Not started |
| Connector types | ⬜ TBD |

### Open Questions — Module B
- [ ] Confirm connector types — inter-board headers or panel jacks?
- [ ] Run ERC and resolve any errors before layout
- [ ] LTspice simulation — verify frequency response and noise

---

## 5. Module C — Power Amplifier

*This section can be extracted as a standalone MODULE_C_POWERAMP.md for review or handoff.*

**Status:** Device selected — schematic not yet started  
**KiCad File:** TBD  
**Simulation:** TBD

### Overview

TI TPA3251 Class D integrated power amplifier in BTL configuration. Delivers ~80W into 8Ω from 32V (70W operating target). Designed for a mid-low frequency driver covering 100Hz–3kHz.

### Signal Flow

```
Module B Out ──▶ [Input filtering / gain setting] ──▶ [TPA3251 BTL] ──▶ [LC output filter] ──▶ Speaker (8Ω)
                                                              ↑
                                                    32V from Module D
```

### Specification

| Parameter | Value | Notes |
|---|---|---|
| Device | TI TPA3251 | Class D, BTL |
| Supply voltage | 32V DC | Single rail, from Module D |
| Output power | ~70W | Into 8Ω — device capable of ~80W |
| Speaker impedance | 8Ω | Confirmed |
| Frequency range | 100Hz–3kHz | Mid-low driver |
| Output configuration | BTL | Differential, speaker floats |
| Input signal | ~1Vrms | From Module B |
| Efficiency | ~85–90% | Class D |
| Heat dissipation | ~8–10W at 70W output | Small heatsink required |
| Output filter | LC low-pass | Attenuates 300–400kHz PWM carrier |
| Package | TBD | TSSOP-44 or TQFN |

### Status & Next Steps

| Task | Status |
|---|---|
| Device selection (TPA3251) | ✅ Complete |
| Supply voltage (32V) | ✅ Confirmed |
| Output power (70W / 8Ω) | ✅ Confirmed |
| Schematic (KiCad) | ⬜ Not started |
| Output filter design | ⬜ TBD |
| LTspice simulation | ⬜ TBD |
| PCB layout | ⬜ Not started |
| Heatsink / thermal design | ⬜ TBD |

### Open Questions — Module C
- [ ] Speaker output connector — screw terminal, binding post, or speakON?
- [ ] High-pass filter below 100Hz — include for driver protection?
- [ ] Gain setting — TPA3251 gain configurable via pins; target TBD
- [ ] Input coupling — AC or DC coupled from Module B?
- [ ] PCB package — TSSOP-44 or TQFN?
- [ ] Heatsink specification

---

## 6. Module D — Power Distribution

*This section can be extracted as a standalone MODULE_D_POWER.md for review or handoff.*

**Status:** Architecture defined — schematic not yet started  
**KiCad File:** TBD

### Overview

Power entry and distribution board. Accepts 32V DC from an off-the-shelf external SMPS brick, provides protection, and distributes power to all modules. Does not generate voltage from mains.

### Signal Flow

```
32V DC Brick (external)
      │
      ▼
[Input connector] ──▶ [Reverse polarity protection] ──▶ [Fuse] ──▶ [TVS clamp]
      │
      ├──▶ 32V ──────────────────────────────────────────▶ Module C (TPA3251)
      ├──▶ [7815 linear regulator] ──▶ +15V ────────────▶ Modules A and B (op-amps)
      ├──▶ [7915 linear regulator] ──▶ −15V ────────────▶ Modules A and B (op-amps)
      └──▶ [Low-noise LDO, TBD] ──▶ 9V or 12V ──────────▶ Module A (Nu capsules)
```

### Specification

| Parameter | Value | Notes |
|---|---|---|
| Input voltage | 32V DC | External SMPS brick |
| Input connector | TBD | Barrel jack or screw terminals |
| Input fuse | TBD | Slow-blow, sized to full system current |
| Reverse polarity | P-channel MOSFET | Lower drop than Schottky diode |
| Transient protection | TVS diode | On 32V input rail |
| 32V output | Pass-through to Module C | After protection |
| +15V output | 7815 linear regulated | Op-amps in Modules A and B |
| −15V output | 7915 linear regulated | Op-amps in Modules A and B |
| Nu capsule rail | TBD voltage, ~3.6mA | Low-noise LDO to Module A |
| Inter-board connectors | TBD | Keyed headers to prevent miswiring |

### Rail Budget (Preliminary)

| Rail | To | Estimated Current | Notes |
|---|---|---|---|
| 32V | Module C | ~2.5A at 70W output | P_in ≈ 80W; I = 80W / 32V |
| +15V | Modules A + B | <150mA | Summing amps + buffer |
| −15V | Modules A + B | <150mA | Summing amps + buffer |
| Nu rail (9/12V) | Module A | ~3.6mA | 450µA × 8 capsules |

**Estimated brick spec:** 32V, ≥3A (≥3.5A recommended).

### Status & Next Steps

| Task | Status |
|---|---|
| Architecture defined | ✅ Complete |
| Rail budget (preliminary) | ✅ Estimated |
| 7815/7915 for ±15V | ✅ Selected |
| Nu capsule rail identified | ✅ Required — regulator TBD |
| Schematic (KiCad) | ⬜ Not started |
| Protection circuit design | ⬜ TBD |
| Full current budget | ⬜ Pending Module A schematic |
| Inter-board connector spec | ⬜ TBD |

### Open Questions — Module D
- [ ] Input connector — barrel jack (size?) or screw terminals?
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Nu capsule regulator selection — low-noise LDO spec
- [ ] Fuse value — confirm once full current budget known
- [ ] Inter-board connector spec — type, pinout, keying across all four modules
- [ ] Thermal design — heatsink or PCB copper for 7815/7915?
- [ ] 32V adequate or step up to 36V for more headroom?

---

## 7. Global Requirements

| Parameter | Value | Notes |
|---|---|---|
| Signal type | Single channel, line level | ~1Vrms nominal at main mix output |
| Inputs | 8 × Cycfi Nu capsules | Via 2mm PCB headers on Module A |
| PCB architecture | 4 separate PCBs (A, B, C, D) | Independent development and repair |
| Supply input | 32V DC external brick | SMPS, off-the-shelf, ≥3A |
| Internal rails | ±15V (audio), 32V (power amp), 9/12V (Nu capsules) | Nu voltage TBD |
| Component preference | Through-hole for prototype | SMD acceptable for production |
| PCB tool | KiCad 10 | |
| Simulation tool | LTspice | |
| Target batch size | < 100 units | |

---

## 8. Performance Targets

> ⚠️ TBD — to be defined before power amplifier design begins.

- Frequency response (e.g. 20Hz–20kHz ±0.5dB)
- THD+N (e.g. < 0.01% at 1kHz)
- Noise floor (e.g. > 90dB SNR)
- Output power
- Mixer channel crosstalk between effect sends

---
---

# DESIGN NOTES — TECHNICAL RATIONALE
*This section records the extended reasoning behind key decisions. The specification above records what was decided; this records why.*

---

## 9. Design Notes — Technical Rationale

### 9.1 Input Buffer — NE5532 vs TL072

**The core question:** bipolar (NE5532) vs JFET (TL072) op-amp input topology.

**Bipolar (NE5532):** requires ~200nA bias current — creates ~20mV DC offset with 100kΩ bias resistor, manageable with coupling capacitor. Voltage noise ~5nV/√Hz — excellent for audio at low source impedances.

**JFET (TL072):** picoamp bias current — no DC offset, no loading of high-impedance sources. Voltage noise ~18nV/√Hz — 3× worse than NE5532. Lower current noise, which matters at high source impedances.

**Decision — NE5532:** system is designed for line-level sources (<1kΩ source impedance). At these impedances, voltage noise dominates — NE5532's advantage is significant. Instrument-level inputs are explicitly out of scope.

**Note for Module A summing amps:** the 10kΩ Nu capsule output impedance is higher than typical line level. At 10kΩ, current noise becomes more relevant — TL072 may be reconsidered for the summing stage. This remains an open question.

---

### 9.2 Power Amplifier — Class D vs Class AB, TPA3251 Selection

**Why not Class AB (LM3886):**
- 70W into 8Ω requires peak output ~33V → LM3886 needs ±35V dual rail — non-standard, expensive
- LM3886 rated 38W into 8Ω from ±28V — falls short of 70W target
- At 70W output, Class AB dissipates ~60W as heat — impractical heatsinking

**Why Class D:**
- Single positive 32V rail — off-the-shelf supply
- 85–90% efficiency — only ~8–10W dissipated at 70W output
- Switching at 300–400kHz — well above 3kHz operating ceiling, attenuated by LC output filter
- BTL configuration doubles voltage swing — 70W into 8Ω achievable from 32V

**TPA3251 vs TPA3255:**
- TPA3255 designed for 36–53V, much higher power — wrong voltage range, overkill
- TPA3251 at 32V delivers ~80W into 8Ω — 70W target has comfortable headroom

**Supply voltage derivation:**
```
V_supply ≥ √(2 × P × R) + headroom
V_supply ≥ √(2 × 70 × 8) + headroom
V_supply ≥ 33.5V + headroom
```
32V is slightly below theoretical maximum but BTL effectively doubles voltage swing, making 70W achievable. 36V would give more headroom.

---

### 9.3 Power Supply Architecture

**Single rail vs dual rail:** Class D does not require dual rail — BTL output bridges the load between two half-bridges creating a floating differential output. Single rail simplifies everything.

**Linear regulation for ±15V:** SMPS rejected — linear regulators have inherently lower noise, critical for audio signal path. 7815/7915: universally available, simple, decades of characterisation. Thermal: each regulator drops 17V at operating current. At 150mA: 17V × 0.15A = 2.55W per regulator — small heatsink or PCB copper pours required.

**External SMPS brick:** eliminates mains isolation design complexity and regulatory considerations. Off-the-shelf bricks are reliable and inexpensive. Linear regulators on Module D provide the final audio-quality regulation.

---

### 9.4 Frequency Range — 100Hz to 3kHz

Designed for a mid-low driver. Implications:
- Coupling capacitors don't need to be sized for sub-bass — smaller values acceptable
- Output filter can roll off above 3kHz — no need for 20kHz extension
- Class D switching artefacts (300–400kHz) are 100× above ceiling — completely benign
- High-pass filter below 100Hz for driver protection — under consideration

---

### 9.5 Nu Capsule Inputs

**Why this changes the design:** Nu capsules are active devices with an integrated preamp — not passive pickups. Their 10kΩ output impedance is higher than typical line sources, and their 200–260mVrms output is ~14dB below standard line level.

**Why active summing is mandatory:** Cycfi Research explicitly advise against passive mixing beyond 3–4 inputs. With 8 inputs, passive summing causes ~18dB attenuation, channel-to-channel crosstalk, and loss of individual outputs.

**Gain requirement:** ~+12–14dB at the summing stage to normalise Nu output to ~1Vrms.

**Nu capsule power:** dedicated low-noise LDO on Module D required — Cycfi specify "clean, well-regulated power source". Candidate voltage: 9V (conventional for pickup circuits) or 12V.

---

### 9.6 Mixer Control Architecture

**Why not 3 knobs per channel (24 total):** An earlier design used level + Send 1 + Send 2 knobs per channel. Rejected in favour of:
- Gain trim per channel (sets level into all buses simultaneously)
- SP3T send assign switch per channel (routes to Send 1 / Send 2 / Both)
- Send master level per bus (2 knobs total)

Result: 18 controls instead of 24. Cleaner panel, fewer parts, clearer signal flow.

**C&K OS103011MS8QP1 switch selection:** SP3T, ON-ON-ON, through-hole vertical, 2mm pitch, 8.2mm × 7.5mm body. Meets all requirements: 3 active positions, through-hole for ruggedness, small footprint, top-actuated through panel slot, KiCad footprint available.

---

### 9.7 4-PCB Modular Architecture

Single board rejected. Separate boards (A, B, C, D) chosen for:
- **Independent development and testing** — each module validated in isolation
- **Ease of repair** — failed module replaced without disturbing system
- **Design clarity** — each PCB has a single, well-defined function
- **Risk separation** — high-power Module C (32V, 70W) physically isolated from sensitive audio signal path

---
---

# PROJECT LOG

---

## 10. Design Decisions Log

| Date | Decision | Rationale | Alternatives Considered |
|---|---|---|---|
| 2026-03-31 | NE5532 for input buffer | Low voltage noise, suited to line-level source impedance | TL072 — rejected, higher noise, better for high-Z sources |
| 2026-03-31 | Unity gain buffer topology | Source isolation only, no gain stage needed at input | Inverting/non-inverting gain stage — unnecessary complexity |
| 2026-03-31 | ±15V supply for buffer | Standard audio op-amp supply, good headroom | ±12V possible but ±15V preferred |
| 2026-03-31 | KiCad 10 + LTspice toolchain | Free, professional grade, git-friendly | Altium — cost prohibitive |
| 2026-03-31 | Single channel system | Simplifies design scope | Stereo — deferred |
| 2026-03-31 | 4-PCB modular architecture | Independent development, testing and repair | Single board — too complex to debug and repair |
| 2026-03-31 | Two effect send buses | Matches use case — two independent effect buses | Main/monitor split — not required |
| 2026-03-31 | Active summing in mixer | Passive unsuitable for 8 Nu inputs — attenuation, crosstalk | Passive resistor network — rejected |
| 2026-03-31 | Gain trim + send assign switch per channel | Cleaner than 3 knobs per channel; send master handles bus level | 3 knobs per channel — rejected |
| 2026-03-31 | Send master level per send bus | Controls overall effect send output level cleanly | Per-channel send level knobs — unnecessary |
| 2026-03-31 | Nu capsules via PCB header | 2mm headers mount directly to PCB; no input jacks needed | Panel-mounted jacks — unnecessary |
| 2026-04-02 | C&K OS103011MS8QP1 SP3T slide switch | Through-hole, small footprint, ON-ON-ON, top-actuated, KiCad footprint available | Toggle — larger; rotary — unnecessary complexity |
| 2026-04-02 | External 32V brick, Module D distributes | Simplifies design, off-the-shelf reliability | Internal SMPS — unnecessary complexity |
| 2026-04-02 | Merged single master document | Reduces cross-file update errors; sections clearly delimited for later extraction | Separate per-module files — retained as extraction target |

---

## 11. Open Questions

### System-level
- [ ] Physical format — enclosure type, rack or desktop?
- [ ] Performance targets — THD, noise floor, bandwidth
- [ ] Inter-board connector spec — type, pinout, keying across all four modules
- [ ] Effect send and main output connector types — 6.35mm jack?

### Module A
- [ ] Op-amp selection — NE5532 or TL072 for summing amps?
- [ ] Exact gain setting — confirm target output level at main out
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Does gain trim control level into main mix and sends simultaneously, or main mix only?
- [ ] Master level control on main output — include or omit?
- [ ] Any clip / level indication per channel or master?

### Module B
- [ ] Connector types — inter-board headers or panel jacks?
- [ ] Run ERC in KiCad — resolve before layout
- [ ] LTspice simulation — verify frequency response and noise

### Module C
- [ ] Speaker output connector — screw terminal, binding post, or speakON?
- [ ] High-pass filter below 100Hz — include for driver protection?
- [ ] Gain setting — TPA3251 gain configurable via pins; target TBD
- [ ] Input coupling — AC or DC coupled from Module B?
- [ ] PCB package — TSSOP-44 or TQFN?
- [ ] Heatsink specification

### Module D
- [ ] Input connector — barrel jack (size?) or screw terminals?
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Nu capsule regulator — low-noise LDO spec
- [ ] Fuse value — confirm once full current budget known
- [ ] Thermal design — heatsink or PCB copper for 7815/7915?
- [ ] 32V adequate or step up to 36V?

---
---

# WORKFLOW & META
*Session protocol and toolchain reference. Not part of the system specification.*

---

## 12. Workflow & Session Protocol

### Toolchain

| Tool | Version | Purpose |
|---|---|---|
| KiCad | 10.0.0 | Schematic capture, PCB layout |
| LTspice | 26.0.1 (macOS) | Analog circuit simulation |
| Claude.ai project chat | — | Design decisions, documentation, research, memory |
| Claude Code | 2.1.88 | Terminal — KiCad/LTspice file editing only |
| git | 2.39.5 | Version control |

### Repository

**URL:** https://github.com/HalldorUlfarsson/KiCad-Claude  
**Local path:** `/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude`

### Division of Labour

**Claude.ai project chat** — all design thinking:
- Design decisions, architecture, component research
- Documentation writing and review
- Persistent memory across all sessions
- Produces this master document as output at end of session

**Terminal** — pushing to GitHub:
- Download `KiCad-Claude.md` from chat outputs
- Copy into local repo folder
- Run one git command

**Claude Code** — file and schematic work only:
- Editing KiCad `.kicad_sch` and LTspice `.asc` files directly
- Only needed when we move into schematic phase

### End of Session — "Save and Push to GitHub"

Claude produces `KiCad-Claude.md` as a download.

Copy it into the local repo folder, then paste into Terminal:

```bash
cd "/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude" && git add -A && git commit -m "Session $(date '+%Y-%m-%d') — docs updated" && git push
```

### Starting a Claude Code Session (schematic phase)

```bash
cd "/Users/hau/Library/CloudStorage/GoogleDrive-halldorion@gmail.com/My Drive/2026/KiCad-Claude"
claude
```

Brief Claude Code at session start:
```
Read KiCad-Claude.md. This is the KiCad-Claude project master document. Familiarise yourself with the current design state before we start.
```

### KiCad MCP Server (future use)

Config: `~/Library/Application Support/Claude/claude_desktop_config.json`
```json
{
  "mcpServers": {
    "kicad": {
      "command": "/Users/hau/kicad-mcp/.venv/bin/python3",
      "args": ["/Users/hau/kicad-mcp/main.py"],
      "env": { "KICAD_SEARCH_PATHS": "~/Documents/KiCad" }
    }
  }
}
```
Reinstall: `cd ~/kicad-mcp && brew install uv && make install`

### GitHub PAT

Stored in Claude.ai project memory. **Expires: 2026-06-29.**  
Before expiry: generate a new fine-grained PAT for `HalldorUlfarsson/KiCad-Claude` (read/write contents) and paste it to Claude to update memory.

### Schematic Workflow (when we get there)

1. Design agreed in Claude.ai chat
2. Claude Code creates/edits `.kicad_sch` directly in repo
3. Open in KiCad — verify visually, run ERC
4. Simulate in LTspice if needed
5. End-of-session push as above

### Human Verification Gates

- [ ] Spec sign-off — requirements agreed before schematic work
- [ ] Schematic review — ERC clean, biasing and decoupling checked
- [ ] Pre-layout simulation — LTspice confirms behaviour
- [ ] Pre-fab layout review — ground planes, trace widths, thermal relief
