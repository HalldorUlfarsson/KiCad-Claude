# Audio Amplifier System — System Specification

**Project:** KiCad-Claude  
**Status:** In Progress  
**Last Updated:** 2026-03-31  
**Revision:** 0.1

---

## 1. Project Overview

A bespoke single-channel audio amplifier system designed and built in small batch quantities (< 100 units). The system is designed for line-level sources and comprises discrete circuit modules developed iteratively using KiCad for schematic capture and PCB layout, with LTspice for simulation.

### Goals
- Clean, high-quality audio signal path from line-level source to speaker
- Modular design — each circuit block developed, tested and documented independently
- Fully documented design with traceable design decisions
- Small batch manufacturable — through-hole and common SMD components preferred

### Out of Scope
- Instrument-level (high impedance) inputs — not designed for, edge case only
- Digital signal processing
- Multi-channel operation

---

## 2. System Architecture

### Signal Chain

```
LINE INPUT → [Input Buffer] → [Power Amplifier] → SPEAKER OUTPUT
                                     ↑
                              [Power Supply]
```

### Module Summary

| Module | Function | Status |
|---|---|---|
| Input Buffer | Unity gain, high-Z in, low-Z out, line level conditioning | In progress |
| Power Amplifier | Drives speaker load from buffered signal | TBD |
| Power Supply | Provides regulated rails to all modules | TBD |

---

## 3. Modules

### 3.1 Input Buffer (PreampV1)

**Function:** Conditions the incoming line-level signal before the power amplifier. Unity gain voltage follower — preserves signal integrity, isolates source from load.

**Key Design Decisions:**
- Op-amp topology: unity gain voltage follower (output fed directly to inverting input)
- Device: NE5532 — chosen for low voltage noise (~5nV/√Hz), suitable for line-level source impedances
- JFET input (TL072) considered and rejected — higher voltage noise, better suited to high-impedance sources
- Single channel

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Gain | 0dB (unity) | Voltage follower |
| Input impedance | ~100kΩ | Set by R2 bias resistor |
| Supply voltage | ±15V | Dual rail |
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

**Function:** Amplifies the buffered signal to drive a speaker load.

> ⚠️ **TBD** — Architecture, topology, and specifications to be defined.

Open questions:
- Output power target (watts)?
- Speaker impedance (4Ω, 8Ω)?
- Topology — Class A, Class AB, Class D?
- Discrete transistor or integrated (e.g. LM3886)?

---

### 3.3 Power Supply

**Function:** Provides regulated DC rails to all system modules.

> ⚠️ **TBD** — To be designed after amplifier architecture is confirmed.

Open questions:
- Supply rails required (±15V confirmed for buffer — power amp may need additional rails)
- Transformer or SMPS?
- Regulation approach — linear preferred for audio
- Chassis-mounted or PCB-mounted?

---

## 4. Global Requirements

| Parameter | Value | Notes |
|---|---|---|
| Signal type | Single channel, line level | ~1Vrms nominal |
| Supply architecture | Dual rail (±) | Voltage TBD pending power amp spec |
| Component preference | Through-hole for prototype | SMD acceptable for production |
| PCB tool | KiCad 10 | |
| Simulation tool | LTspice | |
| Target batch size | < 100 units | |

---

## 5. Performance Targets

> ⚠️ **TBD** — To be defined before power amplifier design begins.

Targets to establish:
- Frequency response (e.g. 20Hz–20kHz ±0.5dB)
- THD+N (e.g. < 0.01% at 1kHz)
- Noise floor (e.g. > 90dB SNR)
- Output power
- Channel crosstalk (N/A — single channel)

---

## 6. Design Decisions Log

| Date | Decision | Rationale | Alternatives Considered |
|---|---|---|---|
| 2026-03-31 | NE5532 for input buffer | Low voltage noise, suited to line-level source impedance | TL072 — rejected, higher noise, better for high-Z sources |
| 2026-03-31 | Unity gain buffer topology | Source isolation only, no gain stage needed at input | Inverting/non-inverting gain stage — unnecessary complexity |
| 2026-03-31 | ±15V supply for buffer | Standard audio op-amp supply, good headroom for line level | ±12V possible but ±15V preferred for headroom |
| 2026-03-31 | KiCad 10 + LTspice toolchain | Free, professional grade, git-friendly, strong community | Altium — cost prohibitive; Fusion 360 Electronics — deferred pending commercial need |
| 2026-03-31 | Single channel system | Simplifies design scope | Stereo considered and deferred |

---

## 7. Open Questions

- [ ] Power amplifier topology and output power target
- [ ] Speaker load impedance
- [ ] Full supply rail requirements (pending power amp spec)
- [ ] Connector types — XLR, RCA, jack?
- [ ] Physical format — enclosure type, rack or desktop?
- [ ] Performance targets — THD, noise floor, bandwidth
- [ ] Input Buffer ERC — run and resolve before layout
- [ ] Input Buffer simulation in LTspice — verify noise and frequency response

---

*This document is the living specification for the KiCad-Claude audio amplifier project. It should be updated after every design session and committed to the project repository alongside KiCad files.*
