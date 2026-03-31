# Module B — Input Buffer
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.1  
**Last Updated:** 2026-03-31  
**Status:** Schematic drafted in KiCad — ERC not yet run  

---

## Overview

The input buffer sits between the mixer (Module A) and the power amplifier (Module C). It is a unity-gain voltage follower — it passes the signal through at exactly the same level it receives it, but with a much lower output impedance. This isolates the mixer from the power amplifier, preventing the amp's input from loading down the mixer output and degrading the signal.

This is the most mature module in the project — a schematic exists in KiCad and the component choices are finalised.

---

## What It Does

- Receives the main mix output from Module A
- Buffers the signal — unity gain (0dB), no amplification
- Presents a high impedance to Module A (~100kΩ) so it does not load the mixer
- Presents a low impedance to Module C so it can drive the power amp input cleanly
- Passes the buffered signal to Module C (Power Amplifier)

---

## Signal Flow

```
Module A Output ──▶ [R1 input protection] ──▶ [U1 NE5532 voltage follower] ──▶ [R3 output isolation] ──▶ Module C Input
                                                        ↑
                                               ±15V from Module D
                                          (decoupled by C1–C4 locally)
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Gain | 0dB (unity) | Voltage follower — output = input |
| Input impedance | ~100kΩ | Set by R2 bias resistor to GND |
| Output impedance | <1Ω | Op-amp output — very low |
| Supply voltage | ±15V dual rail | From Module D |
| Input connector | TBD | J1 — 6.35mm jack or PCB header |
| Output connector | TBD | J2 — 6.35mm jack or PCB header |
| Bandwidth | >100kHz | NE5532 GBW = 10MHz, unity gain stable |
| Noise | ~5nV/√Hz input-referred | NE5532 voltage noise |
| THD | Very low | Op-amp in unity gain — minimal distortion |

---

## Components

| Reference | Value | Package | Function |
|---|---|---|---|
| U1 | NE5532 | DIP-8 | Op-amp — unity gain voltage follower |
| R1 | 100Ω | TH resistor | Input protection — limits current on ESD or fault |
| R2 | 100kΩ | TH resistor | Input bias to GND — sets input impedance |
| R3 | 100Ω | TH resistor | Output isolation — prevents oscillation with capacitive load |
| C1, C3 | 100nF | TH ceramic | HF decoupling on ±15V rails — close to U1 pins |
| C2, C4 | 10µF | TH electrolytic | LF decoupling on ±15V rails |

---

## Circuit Topology

The NE5532 is wired as a **unity-gain voltage follower** (also called a buffer):

- Non-inverting input (+) receives the signal via R1
- Output is connected directly to the inverting input (–) — 100% negative feedback
- This forces the output to track the input exactly
- R2 holds the non-inverting input at 0V DC via a 100kΩ path to GND — sets the DC operating point and defines input impedance
- R3 at the output isolates the op-amp from capacitive loads (long cables, etc.) which can cause high-frequency oscillation

---

## Why NE5532

The NE5532 was chosen over the TL072 (JFET input) for this application:

- **Voltage noise:** NE5532 = ~5nV/√Hz vs TL072 = ~18nV/√Hz. At line-level source impedances (<1kΩ), voltage noise dominates — NE5532 wins by ~3×
- **Source impedance match:** Bipolar input is optimal for low-impedance sources (line level). JFET input suits high-impedance sources (instruments, microphones) which are out of scope
- **Availability:** NE5532 is universally stocked, inexpensive, and well characterised for audio

See `DESIGN_NOTES.md` §1 for full technical rationale.

---

## Power Supply Notes

- Requires ±15V dual rail from Module D
- Local decoupling provided by C1–C4 on the PCB
- Current draw is minimal — typically <20mA total
- Power dissipation in the op-amp is negligible — no heatsink required

---

## Status & Next Steps

| Task | Status |
|---|---|
| Component selection | ✅ Complete |
| Schematic (KiCad) | ✅ Draft complete |
| ERC (Electrical Rules Check) | ⬜ Not yet run |
| LTspice simulation | ⬜ Not yet done |
| PCB layout | ⬜ Not started |
| Connector types confirmed | ⬜ TBD |

---

## Open Questions

- [ ] Confirm input/output connector types (inter-board headers vs panel jacks)
- [ ] Run ERC in KiCad and resolve any errors
- [ ] Simulate in LTspice — verify frequency response and noise
- [ ] Confirm inter-board connector spec to Module A and Module C

---

## Related Files

| File | Description |
|---|---|
| `audio-preamp/PreampV1/PreampV1.kicad_sch` | KiCad schematic |
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §1 | NE5532 vs TL072 technical rationale |
| LTspice simulation | TBD |

---

*This sheet covers Module B only. See SYSTEM_SPEC.md for full system context.*
