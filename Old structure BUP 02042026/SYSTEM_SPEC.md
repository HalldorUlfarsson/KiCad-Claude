# Audio Amplifier System — System Specification

**Project:** KiCad-Claude  
**Status:** In Progress  
**Last Updated:** 2026-04-02  
**Revision:** 0.2

---

## 1. Project Overview

A bespoke single-channel audio amplifier system designed and built in small batch quantities (< 100 units). The system accepts 8 line-level inputs, mixes them into two effect send groups, buffers the signal, amplifies it, and drives a speaker load. All major functional blocks are implemented as separate PCBs to maximise clarity and ease of repair.

### Goals
- Clean, high-quality audio signal path from line-level sources to speaker
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
| A | Mixer PCB | 8 Nu capsule inputs; per-channel gain trim + send assign switch; two effect send buses each with master level | TBD |
| B | Buffer PCB | Unity gain, high-Z in, low-Z out, line level conditioning | In progress |
| C | Power Amp PCB | Drives speaker load from buffered signal | TBD |
| D | Power Distribution PCB | Accepts 32V brick input, distributes and protects rails to all modules | TBD |

---

## 3. Modules

### 3.1 Module A — 8-Input Mixer

**Function:** Accepts 8 Nu capsule signals via PCB headers. Each channel has a gain trim knob (level into all buses) and a 3-position send assign switch (Send 1 / Send 2 / Both). Two effect send buses each have a send master level knob. All channels sum into a main mix output to Module B.

**Key Design Decisions:**
- Inputs are Cycfi Nu capsules — active pickups, 2mm PCB headers, ~200–260mVrms output, 10kΩ output impedance
- Per-channel gain trim sets level into main mix and into send bus simultaneously
- Send assignment via 3-position switch: Send 1 only / Send 2 only / Send 1+2
- Send master level per bus — controls overall output level of each effect send independently
- Active summing (inverting summing amp) — passive rejected for 8 inputs at these levels
- Summing stage provides ~+12–14dB gain to normalise Nu output to line level

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Inputs | 8 × Nu capsule | Via 2mm pitch PCB header |
| Input signal | ~200–260mVrms | Nu capsule output |
| Per-channel controls | Gain trim knob + 3-pos send assign switch | 8 knobs + 8 switches |
| Send master controls | 2 × potentiometer | Send 1 Master + Send 2 Master |
| Total panel controls | 18 | 8 trims + 8 switches + 2 masters |
| Main mix output | 1 × line level | To Module B |
| Effect send outputs | 2 × line level | Post send-master, to FX units |
| Supply voltage | ±15V | From Module D |
| Nu capsule supply | 9V or 12V TBD, ~3.6mA | Separate regulated rail from Module D |

**Status:** TBD — architecture and control layout to be defined  
**KiCad File:** TBD  
**Simulation:** TBD

---

### 3.2 Module B — Input Buffer (PreampV1)

**Function:** Conditions the mixer output before the power amplifier. Unity gain voltage follower — preserves signal integrity, isolates source from load.

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

### 3.3 Module C — Power Amplifier

**Function:** Amplifies the buffered signal from Module B to drive an 8Ω speaker load at up to ~70W. Operates from the 32V rail from Module D using a Class D BTL topology.

**Key Design Decisions:**
- Device: TI TPA3251 — Class D, BTL configuration, 32V single rail
- Output power: ~70W into 8Ω (device capable of ~80W — 70W gives comfortable headroom)
- Speaker impedance: 8Ω confirmed
- Frequency range: 100Hz–3kHz (mid-low driver application)
- Class AB (LM3886) rejected — insufficient power at 8Ω, requires complex dual-rail supply
- TPA3255 rejected — designed for 36–53V, wrong voltage range for 32V system
- Mandatory LC output filter to attenuate 300–400kHz PWM switching carrier

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Device | TI TPA3251 | Class D, BTL |
| Supply voltage | 32V DC | Single rail, from Module D |
| Output power | ~70W | Into 8Ω, BTL |
| Speaker impedance | 8Ω | Confirmed |
| Frequency range | 100Hz–3kHz | Mid-low driver |
| Efficiency | ~85–90% | Class D |
| Heat dissipation | ~8–10W at 70W | Small heatsink required |

**Status:** Device selected — schematic not yet started  
**KiCad File:** TBD  
**Simulation:** TBD

---

### 3.4 Module D — Power Distribution

**Function:** Accepts 32V DC from an off-the-shelf external SMPS brick. Distributes power to all modules with appropriate protection and filtering. Does not generate voltage internally.

**Key Design Decisions:**
- External 32V brick — off-the-shelf SMPS, not designed here
- Module D is a distribution and protection board only
- Linear regulation preferred for audio-sensitive rails (±15V for Modules A and B)

**Specification:**

| Parameter | Value | Notes |
|---|---|---|
| Input | 32V DC | From external brick via barrel jack or terminal |
| Output rails | TBD | 32V to power amp; ±15V regulated to mixer and buffer |
| Protection | Fuse, reverse polarity, TVS | TBD |
| Regulation | Linear (LDO or discrete) | For ±15V audio rails |

**Status:** TBD — pending power amp rail confirmation  
**KiCad File:** TBD

---

## 4. Global Requirements

| Parameter | Value | Notes |
|---|---|---|
| Signal type | Single channel, line level | ~1Vrms nominal at main mix output |
| Inputs | 8 × Cycfi Nu capsules | Via 2mm PCB headers on Module A |
| PCB architecture | 4 separate PCBs (A, B, C, D) | Independent development and repair |
| Supply input | 32V DC external brick | SMPS, off-the-shelf, ≥3A |
| Internal rails | ±15V (audio), 32V (power amp), 9/12V (Nu capsules) | Nu capsule voltage TBD |
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
- Mixer channel crosstalk between effect sends

---

## 6. Design Decisions Log

| Date | Decision | Rationale | Alternatives Considered |
|---|---|---|---|
| 2026-03-31 | NE5532 for input buffer | Low voltage noise, suited to line-level source impedance | TL072 — rejected, higher noise, better for high-Z sources |
| 2026-03-31 | Unity gain buffer topology | Source isolation only, no gain stage needed at input | Inverting/non-inverting gain stage — unnecessary complexity |
| 2026-03-31 | ±15V supply for buffer | Standard audio op-amp supply, good headroom for line level | ±12V possible but ±15V preferred for headroom |
| 2026-03-31 | KiCad 10 + LTspice toolchain | Free, professional grade, git-friendly, strong community | Altium — cost prohibitive; Fusion 360 Electronics — deferred |
| 2026-03-31 | Single channel system | Simplifies design scope | Stereo considered and deferred |
| 2026-03-31 | 4-PCB modular architecture | Independent development, testing and repair of each block | Single board — rejected, too complex to debug and repair |
| 2026-03-31 | Two effect send mix groups | Matches use case — two independent effect buses per channel | Main/monitor split, zone output — not required |
| 2026-03-31 | Active summing in mixer | Passive unsuitable for 8 Nu inputs — attenuation, crosstalk, Cycfi recommendation | Passive resistor network — rejected |
| 2026-03-31 | Gain trim + send assign switch per channel | Cleaner than 3 knobs per channel; send master handles output level per bus | 3 knobs per channel (level + send1 + send2) — rejected |
| 2026-03-31 | Send master level per send bus | Controls overall effect send output level cleanly; trim sets relative channel contribution | Per-channel send level knobs — more complex, unnecessary |
| 2026-03-31 | Nu capsules as inputs — PCB header connection | Cycfi Nu capsules connect via 2mm headers directly to PCB; no input jacks needed on panel | Panel-mounted jacks — unnecessary given capsule connector type |
| 2026-03-31 | C&K OS103011MS8QP1 SP3T slide switch for send assign | Through-hole for ruggedness, small footprint, ON-ON-ON covers all 3 states, KiCad footprint available | Toggle switch — larger footprint; rotary — unnecessary complexity |

---

## 7. Open Questions

- [ ] Mixer op-amp selection — NE5532 or TL072 for summing amps?
- [ ] Mixer gain setting — confirm target output level
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Master level on main mix output — include or omit?
- [ ] Does gain trim control level into main mix and sends simultaneously, or main mix only?
- [ ] Full supply rail requirements (pending Module A schematic current draw)
- [ ] Effect send and main output connector types — 6.35mm jack?
- [ ] Physical format — enclosure type, rack or desktop?
- [ ] Performance targets — THD, noise floor, bandwidth
- [ ] Input Buffer ERC — run and resolve before layout
- [ ] Input Buffer simulation in LTspice — verify noise and frequency response
- [ ] Module D protection spec — fusing value, TVS rating, reverse polarity topology
- [ ] Inter-board connector spec — type, pinout, keying across all four modules

---

*This document is the living specification for the KiCad-Claude audio amplifier project. It should be updated after every design session and committed to the project repository alongside KiCad files.*
