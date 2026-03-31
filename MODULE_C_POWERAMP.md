# Module C — Power Amplifier
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.1  
**Last Updated:** 2026-03-31  
**Status:** Device selected — schematic not yet started  

---

## Overview

The power amplifier takes the buffered line-level signal from Module B and amplifies it to a level capable of driving a speaker. This module handles the highest power levels in the system and operates directly from the 32V rail supplied by Module D. The selected device is the **TI TPA3251**, a Class D integrated power amplifier.

The system is designed for a **mid-low frequency driver** covering **100Hz to 3kHz**, delivering up to **70W into 8Ω**.

---

## What It Does

- Receives buffered line-level signal from Module B
- Amplifies it to speaker-level output
- Drives an 8Ω speaker load at up to ~70W
- Operates from a single 32V DC rail (from Module D)
- Uses BTL (Bridge-Tied Load) output configuration

---

## Signal Flow

```
Module B Output ──▶ [Input filtering] ──▶ [TPA3251 Class D amp] ──▶ [Output LC filter] ──▶ Speaker (8Ω)
                                                   ↑
                                          32V from Module D
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Device | TI TPA3251 | Class D, BTL configuration |
| Supply voltage | 32V DC single rail | From Module D |
| Output power | ~70W | Into 8Ω, BTL, at 32V |
| Speaker impedance | 8Ω | Design target |
| Frequency range | 100Hz – 3kHz | Mid-low driver application |
| Configuration | BTL (Bridge-Tied Load) | Both outputs driven differentially |
| Input signal | Line level from Module B | ~1Vrms nominal |
| Input connector | TBD | PCB header from Module B |
| Output connector | TBD | Screw terminal or speakON to speaker |
| Efficiency | ~85–90% | Class D advantage over Class AB |
| Heat dissipation | ~8–10W | At 70W output — small heatsink required |
| Output filter | LC low-pass | Attenuates 300–400kHz switching noise |
| Package | TBD | TPA3251 available in TSSOP-44 and TQFN |

---

## Why TPA3251

### Why Class D (not Class AB / LM3886)

The LM3886 (Class AB) was the initial candidate and was rejected:

- **Supply voltage problem:** 70W into 8Ω requires a peak output of ~33V. LM3886 needs a dual rail supply (±V) exceeding peak output — this means ±35V or more, requiring a bespoke transformer
- **Output power:** LM3886 is rated 38W into 8Ω from ±28V — falls short of the 70W target
- **Heat:** Class AB at 70W output dissipates ~60W as heat — significant heatsinking needed
- **Supply complexity:** Dual rail supply is considerably more complex than the single-rail 32V used here

Class D advantages for this application:
- Single positive rail supply — 32V, off-the-shelf
- 85–90% efficiency vs 50–60% for Class AB — ~8W dissipation instead of ~60W
- Switching artefacts at 300–400kHz — well above the 3kHz operating ceiling, attenuated by output LC filter
- BTL configuration doubles voltage swing — 70W from 32V is achievable

### Why TPA3251 (not TPA3255)

| | TPA3251 | TPA3255 |
|---|---|---|
| Supply voltage | 21–37V | 36–53V |
| Power at 32V into 8Ω | ~80W | Not rated at 32V |
| Power headroom at 70W | Good — 10W margin | N/A |
| Suitability | ✅ Right-sized | ❌ Overkill, wrong voltage range |

The TPA3255 is designed for higher supply voltages and much higher power levels — it would require a higher supply voltage and adds unnecessary complexity. TPA3251 is the correct device for this application.

See `DESIGN_NOTES.md` §2 for full technical rationale including the supply voltage derivation.

---

## Output Filter

Class D amplifiers require an output LC low-pass filter between the switching output and the speaker. This:
- Attenuates the 300–400kHz PWM switching carrier
- Passes the audio band (100Hz–3kHz) with minimal attenuation
- Is a mandatory part of the PCB design — not optional

Filter values to be determined during schematic design, following TI reference design guidelines for TPA3251.

---

## Frequency Range Implications

Designing for 100Hz–3kHz (not full audio bandwidth) simplifies several aspects:

- Coupling capacitors do not need to be sized for sub-bass extension — smaller values acceptable
- Output filter roll-off above 3kHz is acceptable — does not need to extend to 20kHz
- A **high-pass filter below 100Hz** at the output may be worth adding to protect the driver from out-of-band energy — flagged as future consideration

---

## Power and Thermal

At 70W output, 32V supply, 88% efficiency:

- Input power from supply = 70W / 0.88 ≈ 80W
- Heat dissipated = 80W − 70W = ~10W
- A small heatsink on the TPA3251 package is required
- PCB copper pours under the thermal pad are critical — follow TI layout guidelines

---

## Status & Next Steps

| Task | Status |
|---|---|
| Device selection (TPA3251) | ✅ Complete |
| Supply voltage confirmed (32V) | ✅ Complete |
| Output power target (70W / 8Ω) | ✅ Complete |
| Schematic (KiCad) | ⬜ Not started |
| Output filter design | ⬜ TBD |
| LTspice simulation | ⬜ TBD |
| PCB layout | ⬜ Not started |
| Heatsink / thermal design | ⬜ TBD |
| Speaker connector type | ⬜ TBD |

---

## Open Questions

- [ ] Speaker output connector — screw terminal, binding post, or Neutrik speakON?
- [ ] High-pass filter at output below 100Hz — include or omit?
- [ ] Gain setting — TPA3251 gain is configurable via pins; target gain TBD
- [ ] Input coupling — AC or DC coupled from Module B?
- [ ] PCB package — TSSOP-44 or TQFN? (affects layout complexity)
- [ ] Heatsink specification

---

## Related Files

| File | Description |
|---|---|
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §2 | Class D vs AB rationale, TPA3251 selection, supply voltage derivation |
| `DESIGN_NOTES.md` §4 | Frequency range implications |
| TI TPA3251 datasheet | External reference |
| TI TPA3251 reference design | External reference — EVM schematic |
| KiCad schematic | TBD |
| LTspice simulation | TBD |

---

*This sheet covers Module C only. See SYSTEM_SPEC.md for full system context.*
