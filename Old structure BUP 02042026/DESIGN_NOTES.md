# Design Notes — Technical Rationale
**Project:** KiCad-Claude Audio Amplifier  
**Last Updated:** 2026-04-02

This document captures the extended technical reasoning behind key design decisions. The SYSTEM_SPEC.md records *what* was decided; this file records *why* in depth.

---

## 1. Input Buffer — NE5532 vs TL072

### The Core Question
Op-amp input stage topology — bipolar (NE5532) vs JFET (TL072) — has real consequences for a buffer circuit. The choice depends primarily on source impedance and noise requirements.

### Bipolar Input (NE5532)
Bipolar transistor input stages require a small bias current (~200nA for NE5532) to flow into the input pins. With a 100kΩ bias resistor this creates ~20mV DC offset — manageable with a coupling capacitor or bias compensation resistor. The advantage is **voltage noise**: NE5532 achieves ~5nV/√Hz, which is excellent for audio. At line-level source impedances (typically <1kΩ), voltage noise dominates and bipolar wins clearly.

### JFET Input (TL072)
JFET inputs have bias currents in the picoamp range — effectively zero. This means virtually no DC offset from bias current and no loading of high-impedance sources. However, voltage noise is ~18nV/√Hz — roughly 3× worse than the NE5532. The TL072 has lower **current noise**, which matters when source impedances are high (instrument pickups, piezo transducers, microphone capsules).

### Decision
**NE5532 selected.** System is designed for line-level sources (<1kΩ source impedance). At these impedances, voltage noise dominates — NE5532's 5nV/√Hz advantage is significant. Instrument-level inputs are explicitly out of scope.

---

## 2. Power Amplifier — Class D vs Class AB, TPA3251 Selection

### Why Not Class AB (LM3886)
The LM3886 was the initial candidate — a well-regarded monolithic power amp with decades of audio use. It was rejected for two reasons:

**Supply voltage:** Delivering 70W into 8Ω requires a peak output voltage of ~33V. The LM3886 needs a dual rail supply (±V) where the rail voltage must exceed the peak output. This means ±35V or higher — an awkward, non-standard supply voltage requiring a bespoke or expensive transformer arrangement.

**Operating point:** At 8Ω, the LM3886 is working at its limits. The datasheet rates it at 68W into 4Ω and 38W into 8Ω from ±28V — falling short of the 70W target. Pushing harder risks reliability.

### Why Class D
For a 100Hz–3kHz mid-low driver, Class D is well suited:
- **Efficiency:** 85–90% vs 50–60% for Class AB. At 70W output, Class AB dissipates ~60W as heat requiring significant heatsinking. Class D dissipates ~8W — a small heatsink suffices.
- **Switching noise:** Class D switches at 300–400kHz. The output filter attenuates this well above our 3kHz operating ceiling — no audible artefacts.
- **Supply simplicity:** Class D operates from a single rail, not a dual rail. Dramatically simpler supply design.

### TPA3251 Selection
The TPA3251 (TI PurePath) was selected over the higher-power TPA3255 because:
- At 32V supply, TPA3251 delivers ~80W into 8Ω — our 70W operating point sits comfortably below clipping with good headroom
- The TPA3255 is designed for higher voltages (36–53V) and much higher power — overkill and would push supply voltage requirements higher
- Both share the same architecture; the TPA3251 is the right-sized device for this application
- Extensive DIY community support and TI reference designs available

### Supply Voltage Selection — 32V
Physics determines the minimum supply voltage for a given output power into a given load:

```
V_supply ≥ √(2 × P × R) + headroom
V_supply ≥ √(2 × 70 × 8) + headroom
V_supply ≥ 33.5V + headroom
```

32V is slightly below theoretical maximum for 70W but the TPA3251 in BTL (bridge-tied load) configuration effectively doubles the voltage swing, making 70W achievable. 32V is a standard, widely available supply voltage — 36V would also work and gives more headroom if needed.

---

## 3. Power Supply Architecture

### Single Rail vs Dual Rail
A dual rail supply (±V) is conventional for Class AB amplifiers because the output stage requires both positive and negative swing referenced to a centre ground. Class D does not — it operates from a single positive rail and the BTL output bridges the load between two half-bridges, creating a floating differential output. Single rail simplifies everything: one transformer winding, one rectifier, one filter capacitor bank, one voltage to regulate.

### Linear Regulation for ±15V Audio Rails
The 32V supply rail is stepped down to ±15V for the op-amp stages (Modules A and B) using 7815/7915 linear regulators. SMPS regulators were considered and rejected:
- Linear regulators have inherently lower noise — important for the audio signal path
- 7815/7915 are among the most common components in electronics — universally available, well characterised, simple to apply
- The power dissipation is manageable: (32V − 15V) × I_load. At <150mA combined load, dissipation is ~2.55W per regulator — handled with a small heatsink or PCB copper pours

### Thermal Consideration
The 7815/7915 drop 17V at operating current. If current draw rises (additional op-amp circuitry added in future), heatsinking requirements must be recalculated. Flagged as an open question.

### External SMPS Brick
The decision to use an off-the-shelf 32V brick rather than designing an internal mains supply was deliberate:
- Eliminates mains isolation design complexity and regulatory considerations
- Off-the-shelf SMPS bricks are reliable, well-filtered, and inexpensive
- The linear regulators on Module D provide the final audio-quality regulation — the brick just needs to be adequate, not audiophile-grade

---

## 4. Frequency Range — 100Hz to 3kHz

The system is explicitly designed for a mid-low driver covering 100Hz–3kHz. This has several design implications:
- **No extended low frequency response needed** — no subwoofer-grade coupling capacitors required, smaller values acceptable
- **No extended high frequency response needed** — output filter can roll off above 3kHz without concern
- **Class D switching artefacts** (300–400kHz) are 100× above the operating ceiling — completely benign
- **Speaker protection** — a high-pass filter at the amplifier output below 100Hz protects the driver from out-of-band energy; flagged as a future consideration

---

## 5. Input Sources — Cycfi Nu Capsules

### What the Nu Capsule Is
The Cycfi Research Nu is an active modular single-string pickup with an integrated low-noise preamplifier. Each capsule is a self-contained active device — not a passive pickup requiring an instrument-level input stage.

### Key Electrical Characteristics
- **Output voltage:** 580–740mV peak-to-peak (~200–260mVrms) — below standard line level (~1Vrms / 2.83Vpk-pk)
- **Output impedance:** 10kΩ — higher than typical line-level sources (<1kΩ)
- **Supply:** 5–18V DC at ~450µA per capsule (3.6mA for 8 capsules total)
- **Connection:** 2mm pitch PCB header — designed to mount directly to a PCB

### Why This Changes the Mixer Design
The 10kΩ output impedance has implications for the summing amplifier input design. At this source impedance, current noise becomes more significant than for low-impedance line sources. This is one reason the op-amp selection for the summing amps (NE5532 vs TL072) remains an open question — TL072's JFET input has lower current noise and may be better suited to the 10kΩ Nu output impedance, despite its higher voltage noise.

### Why Active Summing is Mandatory
Cycfi Research explicitly advise against passive mixing for more than 3–4 Nu inputs. With 8 inputs, passive summing would cause:
- Signal attenuation proportional to number of channels (~−18dB for 8 inputs)
- Channel-to-channel crosstalk through the shared summing resistor network
- Inability to have both mixed and individual outputs simultaneously

Active inverting summing amplifiers are used for all three buses (main mix, Send 1, Send 2).

### Gain Requirement
Nu output (~200mVrms) is approximately 14dB below standard line level (1Vrms). The summing stage gain is set to compensate — approximately +12 to +14dB — to deliver a normalised line-level output to Module B. Exact gain to be confirmed during schematic design.

### Nu Capsule Power Supply
Cycfi specify a "clean, well-regulated power source". A dedicated low-noise LDO regulator on Module D supplies the capsules — not a raw tap from the 32V rail. Candidate voltage is 9V (conventional for pickup circuits) or 12V. This remains an open decision.

---

## 6. Mixer Architecture — Control Layout

### Why Not Three Knobs Per Channel
An earlier design considered level, Send 1, and Send 2 knobs per channel (24 potentiometers total). This was rejected in favour of the current architecture:
- **Gain trim + send assign switch** per channel (8 knobs + 8 switches)
- **Send master level** per send bus (2 knobs)
- **Total: 18 controls** instead of 24

The key insight is that the gain trim simultaneously sets the channel's contribution to both the main mix and the send buses — the send assign switch then routes that trimmed signal to the appropriate bus(es). The send master provides overall level control for each effect send. This is cleaner, fewer parts, and mechanically simpler to lay out on a panel.

### Send Assign Switch — C&K OS103011MS8QP1
A 3-position SP3T (Single Pole, 3 Throw) slide switch handles send assignment per channel. Requirements drove this selection:
- **3 positions needed:** Send 1 only / Send 2 only / Send 1+2
- **ON-ON-ON function:** all three positions are active states — no "off" position needed
- **Through-hole:** for mechanical robustness on a panel-mounted instrument
- **Small footprint:** 8 switches on one PCB, space is at a premium
- **Top-actuated:** switch protrudes vertically through a slot in the panel — the PCB sits behind the panel

The C&K OS103011MS8QP1 meets all criteria: SP3T, ON-ON-ON, through-hole vertical, 2mm pin pitch, 8.2mm × 7.5mm body, KiCad footprint available via SnapMagic, ~$0.50–$0.83 per unit.

**Wiring note:** The "Both" (Send 1+2) centre position connects the pole to a junction node that feeds both send summing amp inputs simultaneously. This must be handled explicitly in PCB routing.

---

## 7. 4-PCB Modular Architecture

### Why Four Separate Boards
A single-board design was considered and rejected. Four separate PCBs (Mixer A, Buffer B, Power Amp C, Power Distribution D) were chosen for:
- **Independent development:** each module can be designed, tested, and validated in isolation before integration
- **Ease of repair:** a failed module can be replaced without disturbing the rest of the system — critical for a small-batch instrument that will be used in performance contexts
- **Design clarity:** each PCB has a single, well-defined function with clear input/output boundaries
- **Risk management:** the high-power Module C (TPA3251, 32V, 70W) is physically separated from the sensitive audio signal path of Modules A and B

### Inter-board Connectivity
Modules connect via keyed headers carrying power rails and audio signals. Connector type and pinout are TBD — keying is essential to prevent miswiring during assembly and maintenance.

---

*This document should be updated whenever a significant design decision is made or reconsidered.*
