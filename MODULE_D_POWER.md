# Module D — Power Distribution
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.1  
**Last Updated:** 2026-03-31  
**Status:** Architecture defined — schematic not yet started  

---

## Overview

Module D is the power entry and distribution board for the entire system. It accepts 32V DC from an off-the-shelf external SMPS brick, provides input protection, and distributes power to all other modules. For the audio-sensitive circuits (Modules A and B), it generates clean ±15V rails using linear regulators. The 32V rail is passed through directly to Module C (Power Amplifier).

Module D does **not** generate voltage from mains — it only accepts, protects, filters, and distributes the 32V supplied externally.

---

## What It Does

- Accepts 32V DC input from an external power brick
- Protects the system against reverse polarity, overcurrent, and transients
- Passes 32V through to Module C (Power Amplifier)
- Generates +15V and −15V regulated rails from 32V for Modules A and B
- Distributes all rails via inter-board connectors

---

## Signal Flow

```
32V DC Brick (external)
      │
      ▼
[Barrel jack / terminal] ──▶ [Reverse polarity protection] ──▶ [Fuse] ──▶ [TVS transient protection]
      │
      ├──▶ 32V rail ──▶ Module C (Power Amplifier)
      │
      ├──▶ [7815 linear regulator] ──▶ +15V rail ──▶ Modules A and B
      │
      └──▶ [7915 linear regulator] ──▶ −15V rail ──▶ Modules A and B
```

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Input voltage | 32V DC | From external SMPS brick |
| Input connector | TBD | Barrel jack (DC) or screw terminals — confirm |
| Max input current | TBD | Sum of all module currents — calculate when specs complete |
| Input fuse | TBD | Slow-blow, sized to max system current |
| Reverse polarity protection | Yes | P-channel MOSFET or Schottky diode |
| Transient protection | TVS diode | On 32V input rail |
| 32V output | To Module C | Passed through after protection |
| +15V output | Linear regulated | 7815 or equivalent |
| −15V output | Linear regulated | 7915 or equivalent |
| Max +15V current | TBD | Mixer + buffer combined; estimate <200mA |
| Max −15V current | TBD | Estimate <200mA |
| Voltage regulator type | Linear (7815/7915) | Lower noise than SMPS — important for audio rails |
| Inter-board connectors | TBD | Keyed headers to prevent miswiring |
| PCB size | TBD | |

---

## Rail Budget (Preliminary)

| Rail | Supplied To | Estimated Current | Notes |
|---|---|---|---|
| 32V | Module C (TPA3251) | ~2.5A at 70W output | I = P_in / V = 80W / 32V |
| +15V | Module A (Mixer) + Module B (Buffer) | <100mA | Op-amps only |
| −15V | Module A (Mixer) + Module B (Buffer) | <100mA | Op-amps only |

**Total input power (estimated):** ~80W (dominated by power amp)  
**Brick specification (estimated):** 32V, ≥3A — standard laptop/industrial supply

---

## Linear Regulation — 7815 / 7915

The ±15V rails are generated using 7815 (positive) and 7915 (negative) linear regulators:

- **Why linear (not SMPS):** Linear regulators have inherently lower output noise — critical for the audio signal path in Modules A and B. Switching regulators introduce high-frequency noise that is difficult to filter completely from sensitive audio circuits
- **Why 7815/7915:** Universal availability, simple application, well characterised for decades. No exotic components required
- **Thermal consideration:** Each regulator drops (32V − 15V) = 17V. At 100mA load, dissipation = 17V × 100mA = 1.7W per regulator — manageable with a small heatsink or adequate copper area on PCB

> ⚠️ If mixer current draw increases significantly (e.g. active summing with many op-amps), thermal calculations must be revisited. Flagged as open question.

---

## Protection

### Reverse Polarity
A P-channel MOSFET in series with the positive rail, or a series Schottky diode, prevents damage if the supply brick is connected backwards. MOSFET preferred — lower voltage drop than diode.

### Fuse
A slow-blow fuse on the input rail protects the system from sustained overcurrent (short circuit, failed component). Value TBD once full current budget is known.

### TVS Diode
A transient-voltage suppressor on the 32V rail clamps voltage spikes (from plug/unplug events or supply transients) to a safe level, protecting all downstream components.

---

## External Power Brick

The 32V supply is an **off-the-shelf SMPS brick** — not designed as part of this project.

Requirements for the external brick:
- Output voltage: **32V DC** (36V also acceptable — gives more headroom to power amp)
- Output current: **≥3A** (based on preliminary rail budget — confirm when power amp spec finalised)
- Connector: TBD — barrel jack size to be confirmed and matched to Module D input connector

Common suitable supplies: industrial 32V DIN rail supplies, laptop-style 32V bricks, or bench supplies during development.

---

## Status & Next Steps

| Task | Status |
|---|---|
| Architecture defined | ✅ Complete |
| Rail budget (preliminary) | ✅ Estimated |
| Regulator selection (7815/7915) | ✅ Complete |
| Schematic (KiCad) | ⬜ Not started |
| Protection circuit design | ⬜ TBD |
| Full current budget | ⬜ Pending Module A and C specs |
| Inter-board connector spec | ⬜ TBD |
| Thermal design (regulators) | ⬜ TBD |
| Input connector type confirmed | ⬜ TBD |

---

## Open Questions

- [ ] Input connector — barrel jack (which size?) or screw terminals?
- [ ] Full current budget — requires Module A (mixer) supply current to be estimated
- [ ] Fuse value — TBD once current budget is complete
- [ ] Reverse polarity protection topology — MOSFET or diode?
- [ ] Inter-board connector spec — type, pinout, keying
- [ ] Confirm 32V is adequate for TPA3251 at 70W target, or step up to 36V
- [ ] Thermal design for 7815/7915 — PCB copper or small heatsink?

---

## Related Files

| File | Description |
|---|---|
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` §3 | Power supply architecture rationale |
| KiCad schematic | TBD |
| LTspice simulation | TBD |

---

*This sheet covers Module D only. See SYSTEM_SPEC.md for full system context.*
