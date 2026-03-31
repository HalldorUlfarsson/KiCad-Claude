# Audio Amplifier System — System Specification

**Project:** KiCad-Claude  
**Status:** In Progress  
**Last Updated:** 2026-03-31  
**Revision:** 0.2

---

## 1. Project Overview

A bespoke single-channel audio amplifier system designed and built in small batch quantities (< 100 units). The system is designed for line-level sources driving a single 8Ω mid-low speaker (100Hz–3kHz). Comprises discrete circuit modules developed iteratively using KiCad for schematic capture and PCB layout, with LTspice for simulation.

### Goals
- Clean, high-quality audio signal path from line-level source to speaker
- Modular design — each circuit block developed, tested and documented independently
- Simple, common, rugged components throughout — nothing esoteric
- Single 32V DC supply input — straightforward external power brick
- Fully documented design with traceable design decisions
- Small batch manufacturable — through-hole and common SMD components preferred

### Out of Scope
- Instrument-level (high impedance) inputs — not designed for, edge case only
- Digital signal processing
- Multi-channel operation
- Extended full-range audio (system optimised for 100Hz–3kHz)

---

## 2. System Architecture

### Signal Chain

```
LINE INPUT → [Input Buffer] → [Class D Power Amp] → 8Ω SPEAKER
                                       ↑
                          [Power Supply: 32V + ±15V]
```

### Power Distribution

```
32V DC INPUT (external brick)
        │
        ├──→ TPA3251 Power Stage (direct 32V)
        │
        └──→ Linear Regulators → ±15V → NE5532 Input Buffer
```

### Module Summary

| Module | Function | Key Device | Status |
|---|---|---|---|
| Input Buffer | Unity gain, high-Z in, low-Z out, line level conditioning | NE5532 | In progress |
| Power Amplifier | Class D, drives 8Ω speaker load | TPA3251 | TBD |
| Power Supply | 32V input, ±15V regulated output for buffer | 7815/7915 | TBD |

---

## 3. Modules

### 3.1 Input Buffer (PreampV1)

**Function:** Conditions the incoming line-level signal before the power amplifier. Unity gain voltage follower — preserves signal integrity, isolates source from load.

**Key Design Decisions:**
- Op-amp topology: unity gain voltage follower (output fed directly to inverting input)
- Device: NE5532 — chosen for low voltage noise (~5nV/√Hz), suited to line-level source impedances
- JFET input (TL072) considered and rejected — higher voltage noise, better suited to high-impedance sources
- Single channel

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Gain | 0dB (unity) | Voltage follower |
| Input impedance | ~100kΩ | Set by R2 bias resistor |
| Supply voltage | ±15V | Derived from 32V rail |
| Bandwidth | >20kHz | Well beyond 3kHz operating range |
| Input connector | TBD | J1 |
| Output connector | TBD | J2 |

**Components:**

| Reference | Value | Function |
|---|---|---|
| U1 | NE5532 | Op-amp, DIP-8 |
| R1 | 100Ω | Input protection |
| R2 | 100kΩ | Input bias to GND |
| R3 | 100Ω | Output isolation |
| C1, C3 | 100nF | HF power rail decoupling |
| C2, C4 | 10µF | LF power rail decoupling |

**KiCad File:** `audio-preamp/PreampV1/PreampV1.kicad_sch`  
**Simulation:** Not yet completed  
**Review Status:** Schematic draft — ERC not yet run

---

### 3.2 Power Amplifier

**Function:** Amplifies the buffered line-level signal to drive an 8Ω speaker load at up to 70W.

**Key Design Decisions:**
- Class D topology chosen over Class AB — higher efficiency (85–90% vs 50–60%), less heat, simpler thermal management
- Class AB (LM3886) considered and rejected — awkward supply voltage (±28V dual rail), working at limits for 8Ω load
- TPA3251 selected — common, well-supported, TI PurePath technology, extensive DIY reference designs
- Single 32V supply rail — simpler than dual rail, common off-the-shelf brick
- Operating range 100Hz–3kHz — Class D switching artefacts well above this, no concern

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Output power | ~70W | Into 8Ω at 32V, 70% of speaker rating |
| Speaker impedance | 8Ω | Single speaker |
| Supply voltage | 32V DC | Single rail |
| Frequency range | 100Hz–3kHz | Mid-low driver |
| Topology | Class D, BTL | Bridge-tied load |
| THD+N | TBD | To be confirmed from datasheet / simulation |

**Key Device:** TPA3251 (TI)  
**KiCad File:** TBD  
**Simulation:** Not yet started  
**Review Status:** Not started

---

### 3.3 Power Supply

**Function:** Accepts 32V DC input, distributes to power amp directly, derives regulated ±15V for input buffer.

**Key Design Decisions:**
- Single 32V DC input — common off-the-shelf external supply, simplifies mains isolation
- Linear regulation for ±15V buffer supply — preferred over SMPS for audio, lower noise
- 7815/7915 regulators — extremely common, well-understood, robust

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Input | 32V DC | External brick |
| Output 1 | 32V (direct) | To TPA3251 power stage |
| Output 2 | +15V regulated | To NE5532 buffer V+ |
| Output 3 | -15V regulated | To NE5532 buffer V- |
| Regulation type | Linear (±15V rails) | 7815/7915 or equivalent |
| Current capacity | TBD | To be calculated from module requirements |

**KiCad File:** TBD  
**Simulation:** Not yet started  
**Review Status:** Not started

---

## 4. Global Requirements

| Parameter | Value | Notes |
|---|---|---|
| Signal type | Single channel, line level | ~1Vrms nominal |
| Speaker load | 8Ω | Mid-low driver, 100W RMS rated |
| Operating power | Max 70W (70% of speaker rating) | Safety margin |
| Supply input | 32V DC single rail | External brick |
| Component philosophy | Common, rugged, well-supported | Nothing esoteric |
| Component preference | Through-hole for prototype | SMD acceptable for production |
| PCB tool | KiCad 10 | |
| Simulation tool | LTspice | |
| Target batch size | < 100 units | |

---

## 5. Performance Targets

| Parameter | Target | Notes |
|---|---|---|
| Frequency response | 100Hz–3kHz ±1dB | Matched to mid-low driver |
| Output power | 70W into 8Ω | At 32V supply |
| THD+N | TBD | To be defined |
| SNR | TBD | To be defined |
| Input sensitivity | ~1Vrms | Line level nominal |
| Operating efficiency | >85% | Class D target |

---

## 6. Design Decisions Log

| Date | Decision | Rationale | Alternatives Considered |
|---|---|---|---|
| 2026-03-31 | NE5532 for input buffer | Low voltage noise (~5nV/√Hz), suited to line-level source impedance | TL072 — rejected, higher noise, better for high-Z sources |
| 2026-03-31 | Unity gain buffer topology | Source isolation only, no gain stage needed at input | Inverting/non-inverting gain stage — unnecessary complexity |
| 2026-03-31 | TPA3251 Class D power amp | Common, rugged, 80W into 8Ω at 32V, high efficiency, simple supply | LM3886 Class AB — rejected, awkward ±28V supply, at rating limits for 8Ω |
| 2026-03-31 | Class D topology | 85–90% efficiency, less heat, simpler thermal design, switching artefacts above 3kHz | Class AB — rejected on efficiency and supply complexity grounds |
| 2026-03-31 | 32V DC single rail supply | Common off-the-shelf brick, simpler than dual rail, covers both modules via regulation | 12V — insufficient for 70W into 8Ω; ±28V dual rail — rejected with LM3886 |
| 2026-03-31 | ±15V from linear regulators (7815/7915) | Low noise, simple, extremely common, well-understood | SMPS regulation — rejected, unnecessary noise risk for audio buffer supply |
| 2026-03-31 | KiCad 10 + LTspice toolchain | Free, professional grade, git-friendly | Altium — cost prohibitive; Fusion 360 — deferred |
| 2026-03-31 | Single channel system | Simplifies design scope | Stereo — deferred |

---

## 7. Open Questions

- [ ] TPA3251 output filter design — inductor and capacitor values for 100Hz–3kHz range
- [ ] Gain staging — does buffer output need scaling before TPA3251 input?
- [ ] 32V supply current requirement — calculate from TPA3251 max draw + regulator dissipation
- [ ] ±15V regulator heatsinking — calculate dissipation from 32V→15V drop at buffer current
- [ ] Connector types — XLR, RCA, or jack for line input? Binding posts for speaker output?
- [ ] Physical format — enclosure type, rack or desktop?
- [ ] Input Buffer ERC — run and resolve before layout
- [ ] Input Buffer LTspice simulation — verify noise and frequency response
- [ ] THD+N and SNR targets — define before power amp design begins
- [ ] TPA3251 availability and pricing — confirm for small batch before committing

---

*This document is the living specification for the KiCad-Claude audio amplifier project. Updated after every design session and committed to the project repository alongside KiCad files.*
