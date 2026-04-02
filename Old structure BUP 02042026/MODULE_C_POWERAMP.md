# Module C — Power Amplifier
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.2  
**Last Updated:** 2026-04-02  
**Status:** Device selected — schematic not yet started

---

## Overview

The power amplifier takes the buffered line-level signal from Module B and amplifies it to drive a speaker. This module handles the highest power levels in the system and operates directly from the 32V rail from Module D. The selected device is the **TI TPA3251**, a Class D integrated power amplifier in BTL (Bridge-Tied Load) configuration, delivering up to ~80W into 8Ω from a 32V supply, with a 70W operating target.

The system is designed for a **mid-low frequency driver** covering **100Hz to 3kHz**.

---

## What It Does

- Receives buffered line-level signal from Module B
- Amplifies to speaker-level output
- Drives an 8Ω speaker load at up to ~70W
- Operates from a single 32V DC rail (from Module D)
- BTL output — both amp outputs driven differentially, speaker floats between them

---

## Signal Flow

```
Module B Out ──▶ [Input filtering / gain setting] ──▶ [TPA3251 BTL] ──▶ [LC output filter] ──▶ Speaker (8Ω)
                                                              ↑
                                                    32V from Module D
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Device | TI TPA3251 | Class D, BTL |
| Supply voltage | 32V DC | Single rail, from Module D |
| Output power | ~70W | Into 8Ω, BTL — device capable of ~80W, 70W is operating target |
| Speaker impedance | 8Ω | Confirmed |
| Frequency range | 100Hz – 3kHz | Mid-low driver application |
| Output configuration | BTL (Bridge-Tied Load) | Differential output, speaker floats |
| Input signal | ~1Vrms | From Module B |
| Input connector | TBD | Inter-board header from Module B |
| Speaker output connector | TBD | Screw terminal, binding post, or Neutrik speakON |
| Efficiency | ~85–90% | Class D |
| Heat dissipation | ~8–10W at 70W output | Small heatsink required |
| Output filter | LC low-pass | Attenuates 300–400kHz PWM carrier |
| Package | TBD | TSSOP-44 or TQFN — affects layout |

---

## Why TPA3251 / Class D

### Class AB (LM3886) was rejected:
- Requires dual rail supply (±35V or more) for 70W into 8Ω — non-standard, expensive
- LM3886 rated 38W into 8Ω from ±28V — falls short of 70W target
- At 70W output, Class AB dissipates ~60W as heat — impractical without large heatsink

### Class D advantages:
- Single positive 32V rail — off-the-shelf supply
- 85–90% efficiency — only ~8–10W to dissipate at 70W output
- Switching at 300–400kHz — well above 3kHz operating ceiling, attenuated by output filter
- BTL doubles voltage swing — 70W into 8Ω achievable from 32V

### TPA3251 vs TPA3255:
- TPA3255 designed for 36–53V and much higher power — wrong voltage range, overkill
- TPA3251 at 32V delivers ~80W into 8Ω — 70W target has good headroom
- TPA3251 is the correctly sized device

See `DESIGN_NOTES.md` §2 for full derivation and rationale.

---

## Output Filter

A mandatory LC low-pass filter between the TPA3251 switching output and the speaker:
- Attenuates 300–400kHz PWM carrier to below audible threshold
- Passes audio band (100Hz–3kHz) with minimal loss
- Component values to follow TI reference design for TPA3251

---

## Frequency Range Implications (100Hz–3kHz)

- Coupling capacitors do not need to be sized for sub-bass — smaller values acceptable
- Output filter can roll off above 3kHz without concern — does not need 20kHz extension
- A **high-pass filter below 100Hz** at the output may protect the driver from out-of-band energy — under consideration

---

## Thermal

At 70W output, ~88% efficiency:
- Input power ≈ 80W
- Dissipation ≈ 10W
- A small heatsink on the TPA3251 package is required
- PCB copper pours under thermal pad are critical — must follow TI layout guidelines closely

---

## Status & Next Steps

| Task | Status |
|---|---|
| Device selection (TPA3251) | ✅ Complete |
| Supply voltage (32V) | ✅ Confirmed |
| Output power target (70W / 8Ω) | ✅ Confirmed |
| Schematic (KiCad) | ⬜ Not started |
| Output filter design | ⬜ TBD |
| LTspice simulation | ⬜ TBD |
| PCB layout | ⬜ Not started |
| Heatsink / thermal design | ⬜ TBD |

---

## Open Questions

- [ ] Speaker output connector type — screw terminal, binding post, or speakON?
- [ ] High-pass filter below 100Hz — include for driver protection?
- [ ] Gain setting — TPA3251 gain configurable via pins; target gain TBD
- [ ] Input coupling — AC or DC coupled from Module B?
- [ ] PCB package selection — TSSOP-44 or TQFN?
- [ ] Heatsink specification and attachment method

---

## Related Files

| File | Description |
|---|---|
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §2 | Class D vs AB, TPA3251 selection, supply voltage derivation |
| `DESIGN_NOTES.md` §4 | Frequency range implications |
| TI TPA3251 datasheet | External — ti.com |
| TI TPA3251 EVM reference design | External — ti.com |
| KiCad schematic | TBD |

---

*This sheet covers Module C only. See SYSTEM_SPEC.md for full system context.*
