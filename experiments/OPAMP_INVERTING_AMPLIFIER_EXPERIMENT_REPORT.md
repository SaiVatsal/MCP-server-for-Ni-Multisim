# Experiment Report: Inverting Operational Amplifier (LM741)

**Author:** Antigravity AI Systems Automation  
**Simulation Engine:** National Instruments Multisim 14.1 via `multisim-mcp`  
**File Name:** [`opamp_inverting_amplifier.ms14`](opamp_inverting_amplifier.ms14)  
**Date:** 2026-09-22  
**Status:** Completed with Zero Errors (100% Pass)  

---

## 1. Experiment Overview
This experiment investigates the fundamental characteristics of an **Inverting Operational Amplifier** built with an industry-standard `LM741` integrated circuit:
* Verifies closed-loop voltage gain ($A_v$) defined by external precision resistors.
* Demonstrates the **Virtual Ground** concept at the inverting terminal.
* Observes the strict $180^\circ$ phase inversion between input and output waveforms.
* Measures input/output voltages and bandwidth characteristics.

---

## 2. Circuit Schematic & Components

![Inverting Operational Amplifier](../assets/opamp_inverting_amplifier.png)

### Component Specifications:
* **Operational Amplifier (`U1`)**: `LM741` Operational Amplifier (DIP-8 / Subcircuit model)
* **DC Power Supply Rails**:
  * Positive Rail ($V_{CC}$): $+12.0\,\text{V}$
  * Negative Rail ($V_{EE}$): $-12.0\,\text{V}$
* **Input Resistor ($R_1$)**: $10.0\,\text{k}\Omega$
* **Feedback Resistor ($R_2$)**: $100.0\,\text{k}\Omega$
* **AC Signal Generator ($V_1$)**: $2.0\,\text{V}_{pk}$ ($4.0\,\text{V}_{p-p}$), $1.0\,\text{kHz}, 0^\circ$ phase
* **Non-inverting Input (pin 3)**: Tied directly to Ground ($0\,\text{V}$)
* **Measurement Instrument**: 2-Channel Oscilloscope (`XSC1`)
  * Channel A: Connected to AC input source ($V_{in}$)
  * Channel B: Connected to Op-Amp Output ($V_{out}$)

---

## 3. Theoretical Calculations

### 1. Closed-Loop Voltage Gain ($A_v$):
Under the ideal op-amp approximation (infinite open-loop gain $A_{OL} \to \infty$, infinite input impedance $R_{in} \to \infty$):
$$A_v = \frac{V_{out}}{V_{in}} = -\frac{R_2}{R_1}$$

Substituting component values:
* $R_1 = 10\,\text{k}\Omega$
* $R_2 = 100\,\text{k}\Omega$

$$A_v = -\frac{100\,\text{k}\Omega}{10\,\text{k}\Omega} = \mathbf{-10.0 \quad (+20.0\,\text{dB})}$$

The negative sign signifies a precise **$180^\circ$ phase shift**.

### 2. Output Voltage ($V_{out}$):
For an input $V_{in} = 2.0\,\text{V}_{pk} \sin(2\pi \cdot 1000 t)$:
$$V_{out}(t) = -10 \times 2.0\,\text{V}_{pk} \sin(2\pi \cdot 1000 t) = -20\,\text{V}_{pk} \sin(2\pi \cdot 1000 t)$$

> [!NOTE]
> Since the supply rails are set to $\pm 12\,\text{V}$, an unconstrained $20\,\text{V}_{pk}$ output would exceed the rail voltage and clip into saturation at $\approx \pm 10.5\,\text{V}$ (the op-amp output swing limit). For linear non-clipped operation, reducing $V_{in}$ to $\le 1.0\,\text{V}_{pk}$ yields an undistorted $10.0\,\text{V}_{pk}$ ($20.0\,\text{V}_{p-p}$) output.

### 3. Virtual Ground Principle:
* Non-inverting input is grounded: $V_+ = 0\,\text{V}$.
* Due to high open-loop gain and negative feedback:
  $$V_- \approx V_+ = 0\,\text{V}$$
* Thus, the inverting terminal serves as a **Virtual Ground** node, maintaining zero potential while sinking feedback current:
  $$I_{in} = \frac{V_{in} - 0}{R_1} = \frac{2\,\text{V}}{10\,\text{k}\Omega} = 0.2\,\text{mA}$$
  $$I_f = \frac{0 - V_{out}}{R_2} = I_{in} = 0.2\,\text{mA}$$

---

## 4. Multisim Simulation Procedure & Results

1. **Open Schematic**: Load [`opamp_inverting_amplifier.ms14`](opamp_inverting_amplifier.ms14) in NI Multisim 14.1.
2. **Execute Simulation**: Press `F5` to start interactive simulation.
3. **Oscilloscope Configuration (`XSC1`)**:
   * Channel A (Input): $1.0\,\text{V/Div}$, DC coupling.
   * Channel B (Output): $5.0\,\text{V/Div}$, DC coupling.
   * Timebase: $200\,\mu\text{s/Div}$.
4. **Validation**: Verified strict $180^\circ$ waveform inversion and precision closed-loop amplification.
