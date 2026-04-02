# Module D — Power Distribution
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.2  
**Last Updated:** 2026-04-02  
**Status:** Architecture defined — schematic not yet started

---

## Overview

Module D is the power entry and distribution board. It accepts 32V DC from an off-the-shelf external SMPS brick, provides protection, and distributes power to all other modules. It generates ±15V regulated rails for the audio circuitry (Modules A and B op-amps) and a clean low-voltage rail for the Nu capsule pickups. The 32V rail passes through directly to Module C.

Module D does not generate voltage from mains — it only accepts, protects, filters, and distributes.

---

## What It Does

- Accepts 32V DC from external SMPS brick
- Protects against reverse polarity, overcurrent, and voltage transients
- Passes 32V to Module C (Power Amplifier)
- Generates +15V and −15V regulated rails for Modules A and B (op-amps)
- Generates a clean low-voltage rail (9V or 12V TBD) for the 8 Nu capsules
- Distributes all rails via inter-board connectors

---

## Signal Flow

```
32V DC Brick (external)
      │
      ▼
[Input connector] ──▶ [Reverse polarity protection] ──▶ [Fuse] ──▶ [TVS transient clamp]
      │
      ├──▶ 32V ──────────────────────────────────────────────▶ Module C (TPA3251)
      │
      ├──▶ [7815 linear regulator] ──▶ +15V ──────────────▶ Modules A and B (op-amps)
      │
      ├──▶ [7915 linear regulator] ──▶ −15V ──────────────▶ Modules A and B (op-amps)
      │
      └──▶ [Low-noise regulator, TBD] ──▶ 9V or 12V ──────▶ Module A (Nu capsule supply)
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Input voltage | 32V DC | External SMPS brick |
| Input connector | TBD | Barrel jack or screw terminals |
| Input fuse | TBD | Slow-blow, sized to full system current |
| Reverse polarity | Yes | P-channel MOSFET or Schottky diode |
| Transient protection | TVS diode | On 32V input rail |
| 32V rail | Pass-through to Module C | After protection |
| +15V rail | Linear regulated (7815) | For op-amps in Modules A and B |
| −15V rail | Linear regulated (7915) | For op-amps in Modules A and B |
| Nu capsule rail | TBD voltage, ~3.6mA | Low-noise regulator, to Module A |
| Inter-board connectors | TBD | Keyed headers to prevent miswiring |

---

## Rail Budget (Preliminary)

| Rail | Supplied To | Estimated Current | Notes |
|---|---|---|---|
| 32V | Module C (TPA3251) | ~2.5A at 70W output | P_in ≈ 80W; I = 80W / 32V |
| +15V | Modules A + B op-amps | <150mA | Summing amps + buffer |
| −15V | Modules A + B op-amps | <150mA | Summing amps + buffer |
| Nu rail (9/12V) | Module A — 8 Nu capsules | ~3.6mA | 450µA × 8 capsules |

**Estimated brick spec:** 32V, ≥3A. A standard 32V / 3.5–5A industrial or laptop-style brick is suitable.

---

## Linear Regulation — ±15V (7815 / 7915)

- **Why linear:** Inherently lower output noise than SMPS — critical for audio signal path
- **Why 7815/7915:** Universal availability, simple, decades of characterisation
- **Thermal:** Each regulator drops 17V (32V − 15V). At 150mA: 17V × 0.15A = 2.55W per regulator — requires a small heatsink or generous PCB copper pours

---

## Nu Capsule Supply Rail

The 8 Nu capsules require a clean, regulated supply between 5V and 18V at ~3.6mA total. Cycfi Research specify a "clean, well-regulated power source" — this rules out an unregulated tap and argues for a dedicated low-dropout regulator.

- **Candidate voltage:** 9V or 12V (both within Nu 5–18V range; 9V is conventional for pickup circuits)
- **Regulator:** Low-noise LDO (e.g. LM7809 or similar)
- **Current:** 3.6mA — trivial thermally, but regulator noise spec matters more than current capacity here
- **Route to Module A** via inter-board connector alongside ±15V rails

---

## Protection

**Reverse polarity:** P-channel MOSFET in series with positive rail — lower voltage drop than Schottky diode, recommended.

**Fuse:** Slow-blow on input, sized to maximum system current. Value TBD once rail budget is finalised.

**TVS diode:** Clamps transient spikes on 32V input rail — protects all downstream devices from plug/unplug events and supply transients.

---

## External Brick Requirements

| Parameter | Requirement |
|---|---|
| Output voltage | 32V DC |
| Output current | ≥3A (≥3.5A recommended for margin) |
| Regulation | Adequate — linear regs provide final audio-rail quality |
| Connector | TBD — barrel jack size to match Module D input |

Standard 32V industrial DIN rail supplies or laptop-style bricks are suitable. A bench supply can be used during development.

---

## Status & Next Steps

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
| Input connector type | ⬜ TBD |

---

## Open Questions

- [ ] Input connector — barrel jack (size?) or screw terminals?
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Nu capsule regulator selection — low-noise LDO spec
- [ ] Fuse value — confirm once full current budget is known
- [ ] Reverse polarity topology — MOSFET or diode?
- [ ] Inter-board connector spec — type, pinout, keying across all four modules
- [ ] Thermal design — heatsink or PCB copper for 7815/7915?
- [ ] 32V adequate or step up to 36V for more power amp headroom?

---

## Related Files

| File | Description |
|---|---|
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §3 | Power supply architecture rationale |
| Nu capsule datasheet | https://www.cycfi.com/projects/nu-capsule/ |
| KiCad schematic | TBD |

---

*This sheet covers Module D only. See SYSTEM_SPEC.md for full system context.*
