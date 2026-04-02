# Module B — Input Buffer
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.1  
**Last Updated:** 2026-04-02  
**Status:** Schematic drafted in KiCad — ERC not yet run

---

## Overview

The input buffer sits between the mixer (Module A) and the power amplifier (Module C). It is a unity-gain voltage follower — it passes the signal through at the same level it receives it, but with a much lower output impedance. This isolates the mixer output from the power amplifier input, preventing loading and preserving signal integrity.

This is the most mature module in the project — a schematic exists in KiCad and component choices are finalised.

---

## What It Does

- Receives the main mix output from Module A (~1Vrms nominal)
- Buffers the signal at unity gain (0dB) — no amplification
- Presents high input impedance (~100kΩ) — does not load Module A
- Presents very low output impedance (<1Ω) — drives Module C cleanly
- Passes the buffered signal on to Module C

---

## Signal Flow

```
Module A Out ──▶ [R1 100Ω input protection] ──▶ [U1 NE5532 voltage follower] ──▶ [R3 100Ω output isolation] ──▶ Module C
                                                         ↑
                                                ±15V from Module D
                                           (decoupled by C1–C4 on PCB)
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Gain | 0dB (unity) | Voltage follower — output = input |
| Input impedance | ~100kΩ | Set by R2 bias resistor to GND |
| Output impedance | <1Ω | Op-amp output — very low |
| Supply voltage | ±15V dual rail | From Module D |
| Input connector | TBD | J1 — inter-board header or 6.35mm jack |
| Output connector | TBD | J2 — inter-board header or 6.35mm jack |
| Bandwidth | >100kHz | NE5532 GBW = 10MHz, unity gain stable |
| Input-referred noise | ~5nV/√Hz | NE5532 voltage noise |

---

## Components

| Reference | Value | Package | Function |
|---|---|---|---|
| U1 | NE5532 | DIP-8 | Op-amp — unity gain voltage follower |
| R1 | 100Ω | TH resistor | Input protection — limits current on ESD or fault |
| R2 | 100kΩ | TH resistor | Input bias to GND — defines input impedance |
| R3 | 100Ω | TH resistor | Output isolation — prevents oscillation with capacitive load |
| C1, C3 | 100nF | TH ceramic | HF decoupling on ±15V rails |
| C2, C4 | 10µF | TH electrolytic | LF decoupling on ±15V rails |

---

## Circuit Topology

The NE5532 is wired as a unity-gain voltage follower:
- Non-inverting input (+) receives the signal via R1
- Output is connected directly back to the inverting input (–) — 100% negative feedback
- This forces output to track input exactly
- R2 holds the non-inverting input at 0V DC — sets DC operating point and input impedance
- R3 at the output isolates the op-amp from capacitive loads (long cables etc.) which can cause high-frequency oscillation

---

## Why NE5532

At line-level source impedances (Module A output ~1kΩ or lower), voltage noise dominates. NE5532 at ~5nV/√Hz is approximately 3× better than TL072 (~18nV/√Hz). TL072 JFET input suits high-impedance sources (instruments, mics) — not applicable here.

See `DESIGN_NOTES.md` §1 for full technical rationale.

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

- [ ] Confirm connector types — inter-board headers to Module A and C, or panel jacks?
- [ ] Run ERC in KiCad and resolve any errors before layout
- [ ] LTspice simulation — verify frequency response and noise
- [ ] Confirm Module A output level (~1Vrms) is within comfortable range for buffer input

---

## Related Files

| File | Description |
|---|---|
| `audio-preamp/PreampV1/PreampV1.kicad_sch` | KiCad schematic |
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §1 | NE5532 vs TL072 rationale |

---

*This sheet covers Module B only. See SYSTEM_SPEC.md for full system context.*
