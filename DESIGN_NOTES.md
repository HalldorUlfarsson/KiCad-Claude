# Design Notes — Technical Rationale

**Project:** KiCad-Claude Audio Amplifier  
**Last Updated:** 2026-03-31

This document captures the extended technical reasoning behind key design decisions. The SYSTEM_SPEC.md records *what* was decided; this file records *why* in depth.

---

## 1. Input Buffer — NE5532 vs TL072

### The Core Question
Op-amp input stage topology — bipolar (NE5532) vs JFET (TL072) — has real consequences for a buffer circuit. The choice depends primarily on source impedance and noise requirements.

### Bipolar Input (NE5532)
Bipolar transistor input stages require a small bias current (~200nA for NE5532) to flow into the input pins. With a 100kΩ bias resistor this creates ~20mV DC offset — manageable with a coupling capacitor or bias compensation resistor.

The advantage is **voltage noise**: NE5532 achieves ~5nV/√Hz, which is excellent for audio. At line-level source impedances (typically <1kΩ), voltage noise dominates and bipolar wins clearly.

### JFET Input (TL072)
JFET inputs have bias currents in the picoamp range — effectively zero. This means virtually no DC offset from bias current and no loading of high-impedance sources. However, voltage noise is ~18nV/√Hz — roughly 3x worse than the NE5532.

The TL072 has lower **current noise**, which matters when source impedances are high (instrument pickups, piezo transducers, microphone capsules).

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
A dual rail supply (±V) is conventional for Class AB amplifiers because the output stage requires both positive and negative swing referenced to a centre ground. Class D does not — it operates from a single positive rail and the BTL output bridges the load between two half-bridges, creating a floating differential output.

Single rail simplifies everything: one transformer winding, one rectifier, one filter capacitor bank, one voltage to regulate.

### Linear Regulation for ±15V Buffer Supply
The 32V supply rail is stepped down to ±15V for the NE5532 input buffer using 7815/7915 linear regulators. SMPS regulators were considered and rejected:

- Linear regulators have inherently lower noise — important for the audio signal path
- 7815/7915 are among the most common components in electronics — universally available, well characterised, simple to apply
- The power dissipation is manageable: (32V - 15V) × I_buffer. Buffer current is typically <50mA, giving ~850mW dissipation per regulator — easily handled with a small heatsink

### Thermal Consideration
The 7815/7915 drop 17V at the buffer operating current. If buffer current rises (additional circuitry added in future), heatsinking requirements should be recalculated. This is flagged as an open question.

---

## 4. Frequency Range — 100Hz to 3kHz

The system is explicitly designed for a mid-low driver covering 100Hz–3kHz. This has several design implications:

- **No extended low frequency response needed** — no subwoofer-grade coupling capacitors required, smaller values acceptable
- **No extended high frequency response needed** — output filter can roll off above 3kHz without concern
- **Class D switching artefacts** (300–400kHz) are 100x above the operating ceiling — completely benign
- **Speaker protection** — a high-pass filter at the amplifier output below 100Hz protects the driver from out-of-band energy; this is flagged as a future consideration

---

*This document should be updated whenever a significant design decision is made or reconsidered.*
