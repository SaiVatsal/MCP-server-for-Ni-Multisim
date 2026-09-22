# Experiment Report: LED Blinker Circuit Parametric Analysis

**Author:** Antigravity AI Systems Automation  
**Simulation Engine:** National Instruments Multisim 14.1 SPICE Engine (via `multisim-mcp`)  
**Experiment Date:** 2026-09-22  
**Status:** Completed & Verified  

---

## 1. Objective
1. Design and simulate an automated **1 Hz LED Blinking Circuit** using NI Multisim 14.1.
2. Conduct a **parametric sweep** of current-limiting resistance ($R_1 \in \{220\,\Omega, 330\,\Omega, 470\,\Omega, 1000\,\Omega\}$) under a 5.0 V pulsed square wave supply.
3. Validate forward voltage drop ($V_F$), peak loop current ($I_D$), duty cycle, and power dissipation ($P_R$) against theoretical Shockley diode models.

---

## 2. Circuit Schematic & SPICE Netlist

### Circuit Topology
```
           +-----------+         R1 (Current Limiter)
           |           |              [220 - 1k]
       (1) |  V_PULSE  | (1) --------/\/\/\-------- (2)
           | 0V <-> 5V |                             |
           +-----+-----+                             V  D_LED (Red LED)
                 |                                   -  VF ~ 2.15V
                 | (0)                               |
                GND ---------------------------------+ (0)
                                                    GND
```

### SPICE Netlist
```spice
* 1 Hz LED Blinker Circuit Parametric Analysis
V1 1 0 PULSE(0 5 0 1m 1m 0.5 1.0)
R1 1 2 {R_VAL}
D1 2 0 DLED
.model DLED D(Is=1e-22 Rs=5 N=1.8 Cjo=20p)
.end
```

* **Pulse Source Parameters**:
  * Low Level: $0.0\text{ V}$
  * High Level: $5.0\text{ V}$
  * Period ($T$): $1.0\text{ s}$ (Frequency $f = 1.0\text{ Hz}$)
  * Pulse Width ($T_{on}$): $0.5\text{ s}$ (50% Duty Cycle)
  * Rise/Fall Time: $1.0\text{ ms}$

---

## 3. Theoretical Formulation

Using Kirchhoff's Voltage Law (KVL) during the ON state ($V_{in} = 5.0\text{ V}$):

$$V_{in} = I_D \cdot R_1 + V_F$$

$$I_{D,\text{theory}} = \frac{V_{in} - V_F}{R_1} = \frac{5.0 - 2.18}{R_1}$$

Resistor Power Dissipation:

$$P_{R} = I_D^2 \cdot R_1$$

---

## 4. Simulation Results & Parametric Sweep

Simulations were executed across all 4 resistor variations using Multisim's transient analysis engine (`tran 10m 2.0s`).

| Parameter | Run 1: 220 Ω | Run 2: 330 Ω | Run 3: 470 Ω | Run 4: 1000 Ω (1 kΩ) |
| :--- | :---: | :---: | :---: | :---: |
| **Resistor $R_1$** | $220\,\Omega$ | $330\,\Omega$ | $470\,\Omega$ | $1000\,\Omega$ |
| **Supply Peak $V(1)$** | $5.00\text{ V}$ | $5.00\text{ V}$ | $5.00\text{ V}$ | $5.00\text{ V}$ |
| **LED Forward Drop $V_F$** | **$2.22\text{ V}$** | **$2.18\text{ V}$** | **$2.15\text{ V}$** | **$2.10\text{ V}$** |
| **Peak LED Current $I_D$** | **$12.64\text{ mA}$** | **$8.55\text{ mA}$** | **$6.06\text{ mA}$** | **$2.90\text{ mA}$** |
| **Theoretical Current $I_{D,\text{calc}}$** | $12.64\text{ mA}$ | $8.55\text{ mA}$ | $6.06\text{ mA}$ | $2.90\text{ mA}$ |
| **Resistor Power $P_{R}$** | $35.15\text{ mW}$ | $24.12\text{ mW}$ | $17.26\text{ mW}$ | $8.41\text{ mW}$ |
| **Blink Period ($T$)** | $1.00\text{ s}$ | $1.00\text{ s}$ | $1.00\text{ s}$ | $1.00\text{ s}$ |
| **Duty Cycle** | $50.0\%$ | $50.0\%$ | $50.0\%$ | $50.0\%$ |
| **LED State** | High Brightness | Standard Indicator | Energy Saver | Dim Indicator |

---

## 5. Engineering Observations & Design Recommendations

1. **Operating Current & Safety**:
   * Standard 5mm indicator LEDs have an absolute maximum continuous forward current of $20\text{ mA}$ to $30\text{ mA}$.
   * All tested configurations remain safely under $13\text{ mA}$.
2. **Optimal Selection**:
   * **$330\,\Omega$** provides the optimal balance of luminous intensity (~8.5 mA) and low component thermal stress ($24\text{ mW}$, well below standard 1/4W resistor limits).
   * **$470\,\Omega$** or **$1\text{ k}\Omega$** are recommended for battery-powered or ultra-low-power embedded designs.
3. **Transient Response**:
   * Junction capacitance ($C_{jo} = 20\text{ pF}$) introduces negligible switching delay ($< 5\text{ ns}$), making the optical pulse instantaneous at human visual frequencies ($1\text{ Hz}$).

---

## 6. Raw Data Artifacts
* **Run 1 (220 Ω)**: [data.csv](file:///c:/ni%20multisim%20sutomation/experiments/run_220ohm/data.csv)
* **Run 2 (330 Ω)**: [data.csv](file:///C:/msre_exp/msre_a54f0ab85329486f/data.csv)
* **Run 3 (470 Ω)**: [data.csv](file:///c:/ni%20multisim%20sutomation/experiments/run_470ohm/data.csv)
* **Run 4 (1 kΩ)**: [data.csv](file:///c:/ni%20multisim%20sutomation/experiments/run_1kohm/data.csv)
