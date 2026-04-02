# Module A — 8-Input Mixer
**Project:** KiCad-Claude Audio Amplifier  
**Revision:** 0.2  
**Last Updated:** 2026-04-02  
**Status:** Architecture defined — schematic not yet started

---

## Overview

The mixer accepts 8 signals from Nu modular single-string pickups (Cycfi Research), combines them into a main mix output (to Module B), and independently routes each channel to two effect send buses. Each channel has a gain trim knob and a 3-position send assign switch (Send 1 / Send 2 / Both). The two effect send buses each have a master level knob. The mixer uses active summing throughout — passive mixing is not suitable for 8 inputs at these signal levels.

This is the most mechanically complex module, with 18 controls on the front panel: 8 gain trim knobs, 8 send assign switches, and 2 send master knobs.

---

## Input Sources — Nu Capsules

The 8 inputs come from **Cycfi Research Nu modular single-string pickups**. These are active pickups with an integrated low-noise preamplifier, designed for per-string pickup on guitars and similar instruments.

Key Nu capsule electrical characteristics relevant to this design:

| Parameter | Value | Notes |
|---|---|---|
| Output voltage | 580–740mV peak-to-peak | ~200–260mVrms — below standard 1Vrms line level |
| Output impedance | 10kΩ | Higher than typical line-level sources |
| Supply voltage | 5V–18V DC | Reverse polarity protected internally |
| Supply current | 450µA per capsule | × 8 capsules = 3.6mA total |
| Connectivity | 2mm pitch PCB header | Designed to mount directly to PCB |
| Frequency response | 20Hz–20kHz | Flat, uncoloured |

**Signal level note:** Nu output (~200–260mVrms) is below standard line level (~1Vrms). The active summing stage will include gain (~+12–14dB) to normalise the output to line level before Module B.

**Power for Nu capsules:** 8 capsules require a clean, regulated 5–18V supply at ~3.6mA total. Cycfi specifies a "clean, well-regulated power source". Module D should provide a dedicated low-noise rail for this. Voltage TBD (9V or 12V are natural choices).

---

## Channel Controls — Terminology

Each of the 8 channels has the following controls:

| Control | Type | Function |
|---|---|---|
| **Gain Trim** | Potentiometer (knob) | Sets the signal level for this channel — acts as a trim before the main mix and before the send assignment switch. Compensates for level differences between pickups. |
| **Send Assign** | 3-position switch | Routes this channel to Effect Send 1 only, Effect Send 2 only, or both Send 1 + Send 2. Does not affect the main mix. |

**Effect Send Master levels** (one per send bus, not per channel):

| Control | Type | Function |
|---|---|---|
| **Send 1 Master** | Potentiometer (knob) | Master output level for the entire Effect Send 1 bus — controls the overall send level to FX Unit 1 |
| **Send 2 Master** | Potentiometer (knob) | Master output level for the entire Effect Send 2 bus — controls the overall send level to FX Unit 2 |

**Total per-channel controls: 8 × gain trim knob + 8 × 3-position switch = 16 controls**  
**Plus: 2 × send master knobs = 18 controls total on the panel**

This is a simpler and cleaner layout than 3 knobs per channel — the gain trim sets the relative level of each pickup in all buses simultaneously, and the switch determines which effect send(s) that channel feeds.

---

## Signal Flow

```
Nu #1 ──[Gain Trim]──┬────────────────────────────── Main Mix Summing Amp ──▶ Main Out → Module B
                     │
                     └──[Send Assign Switch]──┬──▶ Send 1 Summing Amp ──[Send 1 Master]──▶ Send 1 Out → FX Unit 1
                                              ├──▶ Send 2 Summing Amp ──[Send 2 Master]──▶ Send 2 Out → FX Unit 2
                                              └──▶ Both (switch position 3)

Nu #2–8 ──▶ (same structure repeated × 8)
```

- **Gain trim** controls the channel level entering all buses — main mix and sends simultaneously
- **Send assign switch** (3-position): Send 1 only / Send 2 only / Send 1+2
- **Send master** controls the overall output level of each send bus — one knob per send, not per channel
- All channels feed the main mix regardless of send assignment
- Three summing amplifiers: main mix, Send 1 bus, Send 2 bus

---

## Summing Architecture — Active

Passive summing is explicitly not used. Cycfi Research advise against passive mixing for more than 3–4 Nu inputs due to: signal attenuation per added channel, channel crosstalk, and loss of individual outputs.

**Active inverting summing amplifier** (op-amp based) for each bus:
- Full channel isolation — no crosstalk between inputs
- Gain set at the summing stage to compensate for Nu sub-line output level
- Clean, predictable summing

**Op-amp device:** TBD — NE5532 (bipolar, low voltage noise) or TL072 (JFET, suits 10kΩ source impedance) under consideration. Decision to be made during schematic phase.

---

## Specification

| Parameter | Value | Notes |
|---|---|---|
| Inputs | 8 × Nu capsule | Via 2mm pitch PCB header |
| Input signal level | ~200–260mVrms | Nu capsule output |
| Input impedance | TBD | Must suit 10kΩ Nu output impedance |
| Per-channel controls | 1 × gain trim knob + 1 × SP3T slide switch | Level trim and send assignment |
| Send master controls | 2 × potentiometer | Send 1 Master, Send 2 Master — one per send bus |
| Total panel controls | 18 | 8 trims + 8 switches + 2 send masters |
| Main mix output | 1 × line level (~1Vrms) | All channels summed — to Module B |
| Effect send outputs | 2 × line level | To external effects units, post send-master |
| Output connectors | TBD | 6.35mm jack (TRS) or inter-board header |
| Summing architecture | Active — inverting summing amp | 3 separate summing buses |
| Gain | TBD | ~+12–14dB to normalise Nu output to 1Vrms |
| Op-amp supply | ±15V | From Module D |
| Nu capsule supply | TBD voltage, ~3.6mA | Separate clean rail from Module D |

---

## Send Assign Switch — Selected Component

**C&K OS103011MS8QP1** — SP3T, ON-ON-ON, through-hole vertical, PC pin

| Parameter | Value |
|---|---|
| Manufacturer | C&K (Littelfuse) |
| Part number | OS103011MS8QP1 |
| Configuration | SP3T — Single Pole, 3 Throw |
| Switch function | ON-ON-ON — all 3 positions are active |
| Positions | 1: Send 1 only / 2: Send 1+2 / 3: Send 2 only |
| Mounting | Through-hole vertical — top actuated through slot in panel | ✅ Confirmed |
| Pin pitch | 2mm |
| Travel | 2mm per position |
| Current rating | 100mA at 12VDC — well within audio signal levels |
| Body size | 8.2mm × 7.5mm |
| KiCad footprint | Available via SnapMagic / KiCad library |
| Digikey | OS103011MS8QP1-ND |
| Approximate unit cost | ~$0.50–$0.83 |

**Switch position logic:**

| Position | Connects | Result |
|---|---|---|
| 1 | Pole → Throw 1 | Send 1 only |
| 2 | Pole → Throw 2 | Send 1 + 2 (centre position — pole feeds a junction) |
| 3 | Pole → Throw 3 | Send 2 only |

> ⚠️ The "Both" (Send 1+2) position requires the centre throw to connect to both send buses. This needs to be handled in the PCB routing — the centre pin connects to a node that feeds into both summing amp inputs. Confirm wiring during schematic phase.

**Orientation confirmed:** Vertical (OS103011MS8QP1) — switch actuator protrudes upward through a slot in the panel. The right-angle variant (OS103011MA7QP1) is not used.

---

## Open Questions

- [ ] Op-amp selection — NE5532 or TL072 for summing amps?
- [ ] Exact gain setting — confirm target output level at main out
- [ ] Nu capsule supply voltage — 9V or 12V?
- [ ] Does gain trim control level in both main mix and send buses, or main mix only?
- [ ] Output connector types — main out and effect sends
- [ ] Master level control on main output — include or omit?
- [ ] Any clip / level indication per channel or master?
- [ ] PCB dimensions and mechanical format

---

## Physical Notes

Per-channel panel layout: 8 columns, each with a **gain trim knob** and a **C&K OS103011MS8QP1 SP3T slide switch** (top-actuated through a slot in the panel). The two **send master knobs** are grouped together at one end of the panel.

The Nu capsules connect directly to PCB headers — no input jacks on the front panel. The panel carries only the controls and output connectors, keeping it clean and uncluttered.

The slide switch actuator protrudes upward through a panel slot — the PCB sits behind the panel with the switch body mounted vertically. Panel slot dimensions must match the OS103011MS8QP1 actuator profile; confirm from datasheet before panel fabrication.

---

## Related Files

| File | Description |
|---|---|
| `SYSTEM_SPEC.md` | System-level context |
| `DESIGN_NOTES.md` | Extended technical rationale |
| Nu capsule datasheet | https://www.cycfi.com/projects/nu-capsule/ |
| KiCad schematic | TBD |
| LTspice simulation | TBD |

---

*This sheet covers Module A only. See SYSTEM_SPEC.md for full system context.*
