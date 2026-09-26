# NI Multisim 14.1 Automation with MCP

This project connects **NI Multisim 14.1** with an MCP server so an AI agent can work with Multisim directly.

The setup uses **multisim-mcp** and **Antigravity** to open circuits, edit components, run simulations, read measurements, perform parameter sweeps, and generate reports.

The main goal is to make working with Multisim easier by allowing many circuit tasks to be done using normal language instead of doing everything manually inside Multisim.

## What This Project Can Do

With the MCP setup you can:

* Open and inspect Multisim circuits
* Create and modify circuits
* Change component values
* Read circuit and net information
* Run DC operating point simulations
* Run AC frequency sweeps
* Run transient simulations
* Read voltage and current measurements
* Use virtual instruments
* Run parameter sweeps
* Export simulation data
* Generate experiment reports

The Multisim COM interface is used underneath the MCP server to communicate with Multisim.

## System Setup

The setup was tested on the following configuration:

| Component        | Version / Setup    |
| ---------------- | ------------------ |
| Operating System | Windows 11 64-bit  |
| NI Multisim      | 14.1 32-bit        |
| Python           | 3.14 32-bit        |
| MCP Server       | multisim-mcp 1.1.0 |
| AI Client        | Antigravity        |

The important part here is the **32-bit setup**. Multisim 14.1 uses 32-bit COM components, so the Python environment used to communicate with Multisim also needs to be 32-bit.

## Installation

### 1. Check the Python Installations

First check which Python versions are installed:

```powershell
py --list
```

Then check that the Python version you want to use is actually 32-bit:

```powershell
py -3.14-32 -c "import sys; print(sys.executable); print('Is 64-bit:', sys.maxsize > 2**32)"
```

The Python installation used in this setup was:

```text
C:\Program Files (x86)\Python314-32\python.exe
```

The important thing is that the output confirms that Python is running as 32-bit.

## 2. Install multisim-mcp

Install version 1.1.0 using the 32-bit Python installation:

```powershell
& "C:\Program Files (x86)\Python314-32\python.exe" -m pip install "multisim-mcp==1.1.0"
```

Because Python is installed under `Program Files (x86)`, Windows may not allow normal writing to that directory.

In that case, pip can place the user-level scripts under:

```text
C:\Users\<User>\AppData\Roaming\Python\Python314-32\Scripts
```

This is where the `multisim-mcp.exe` executable can be found.

## 3. Check the Multisim Connection

Before configuring Antigravity, it is a good idea to check whether the MCP setup can actually communicate with Multisim.

Run:

```powershell
& "$env:APPDATA\Python\Python314-32\Scripts\multisim-mcp.exe" --json doctor --connect
```

The diagnostic checks the Python architecture, Multisim COM registration, the Multisim application, and the simulation engine.

The verified setup showed:

```text
Python Architecture: 32-bit
COM Registration: Found
Live Connection: Connected
Simulation Engine: Ready
```

The Multisim COM object detected in the setup was:

```text
MultisimInterface.MultisimApp
```

with CLSID:

```text
{D9CBB7A1-6AD6-438B-AD1B-42FB2DB00CAE}
```

The Multisim installation was detected at:

```text
C:\Program Files (x86)\National Instruments\Circuit Design Suite 14.1\Multisim.exe
```

The diagnostic also reported:

```text
automation_ready: true
```

So the COM connection and simulation engine were working correctly.

## 4. Generate the MCP Configuration

You can generate the configuration using:

```powershell
& "$env:APPDATA\Python\Python314-32\Scripts\multisim-mcp.exe" config --client generic --python "C:\Program Files (x86)\Python314-32\python.exe"
```

The generated configuration looks like this:

```json
{
  "command": "C:\\Program Files (x86)\\Python314-32\\python.exe",
  "args": [
    "-m",
    "multisim_mcp.server"
  ]
}
```

This tells the MCP client to start the Multisim MCP server using the 32-bit Python installation.

## 5. Add It to Antigravity

The MCP configuration can be added to the Antigravity configuration file.

Possible configuration locations are:

```text
~/.gemini/config/mcp_config.json
```

or:

```text
~/.gemini/antigravity/mcp_config.json
```

Add the Multisim server under `mcpServers`:

```json
{
  "mcpServers": {
    "multisim": {
      "command": "C:\\Program Files (x86)\\Python314-32\\python.exe",
      "args": [
        "-m",
        "multisim_mcp.server"
      ]
    }
  }
}
```

After this is configured, Antigravity can start the MCP server when it needs to use the Multisim tools.

# How the Setup Works

You do not have to manually start the MCP server every time.

Once the server is configured in Antigravity:

1. Antigravity starts the MCP server when it needs it
2. The MCP server connects to Multisim
3. The AI agent can call the available Multisim tools
4. Multisim performs the requested operation
5. Results are returned to the AI agent

There is no need to keep a separate terminal window running `multisim-mcp serve`.

The configuration can also be available across different Antigravity workspaces when it is placed in the global configuration.

# Using Multisim with Natural Language

Once everything is configured, you can talk to the AI agent normally.

## Open and Inspect a Circuit

For example:

```text
Connect to Multisim and open the circuit file at C:\Circuits\AudioAmp.ms14
```

You can also ask:

```text
List all components, nets, and input/output pins in the active schematic.
```

To change component values:

```text
Change resistor R1 to 10k ohms and capacitor C1 to 100nF.
```

These commands can be translated into the corresponding Multisim operations through MCP.

## Run Simulations

You can ask for a DC operating point:

```text
Run a DC operating point analysis and show the nodal voltages.
```

For AC analysis:

```text
Run an AC frequency sweep from 10 Hz to 100 kHz on net Vout and report the -3dB cutoff frequency.
```

For transient analysis:

```text
Run a 10ms transient simulation with a 1us time step and report peak voltages.
```

You can also read virtual instrument measurements:

```text
Read the virtual multimeter value on net 3.
```

The MCP server provides tools for these simulation operations.

# Parameter Sweeps

The setup can also be used for testing different component values automatically.

For example:

```text
Run a parameter sweep for resistor R1 across [1k, 4.7k, 10k] and evaluate rise time.
```

You can also ask it to create a report:

```text
Export the formal experiment report with waveform CSV data.
```

This is useful when you want to compare several circuit configurations without changing each value manually.

# Verified Test Circuit

To test the complete setup, a simple **1 Hz blinking LED circuit** was simulated through Multisim.

The test was useful because it checked the whole process:

```text
AI Request
    ↓
MCP
    ↓
Multisim
    ↓
SPICE Simulation
    ↓
Measurements
    ↓
Results
```

## Circuit Netlist

The test circuit used the following SPICE netlist:

```spice
* LED Blinking Circuit (1 Hz)
V1 1 0 PULSE(0 5 0 1m 1m 0.5 1.0)
R1 1 2 330
D1 2 0 DLED
.model DLED D(Is=1e-22 Rs=5 N=1.8 Cjo=20p)
.end
```

The pulse source switches between 0 V and 5 V.

The resistor limits the LED current.

## Simulation Command

The transient simulation was run with:

```spice
tran 10m 2.0
```

The simulation produced the expected LED waveform.

The measured results were:

* Pulse voltage: 0 V to 5 V
* ON time: 500 ms
* OFF time: 500 ms
* LED anode voltage: about 2.18 V
* Peak LED current: about 8.55 mA
* Waveform data: saved to CSV

The complete test confirmed that Multisim was able to run the circuit and return simulation data through the automation setup.

# MCP Tools

The `multisim-mcp` setup provides around **55 tools** for working with Multisim.

The tools are grouped by purpose.

## Connection

```text
connect
disconnect
runtime_status
circuit_info
```

These tools are used to connect to Multisim and check its current state.

## Circuit Management

```text
new_circuit
open_circuit
save_circuit
get_circuit_image
```

These are used to create, open, save, and inspect circuits.

## Components and Netlists

```text
report_netlist
report_bom
enum_components
get_rlc_value
set_rlc_value
```

These tools make it possible to inspect components and change values programmatically.

## Simulation

```text
run_dc_operating_point
run_ac_sweep
run_ac_single_frequency
run_transient
stop_simulation
run_spice_netlist
```

These cover the main simulation types used in the project.

## Virtual Instruments

```text
read_virtual_multimeter
analyze_bode_response
analyze_logic_signals
```

These tools are useful for reading measurements and analyzing simulation results.

## Experiments and Reports

```text
run_circuit_experiment
run_verified_circuit_experiment
plan_experiment_sweep
run_experiment_sweep
generate_report
export_formal_experiment_report
```

These tools are useful when running complete experiments and generating results automatically.

# Analog Laboratory Experiments

The project also contains a set of analog electronics experiments.

Each experiment has:

* A native `.ms14` Multisim schematic
* A Markdown experiment report
* Simulation results
* Component values
* Theoretical calculations
* Verification results

The project currently documents the following experiments.

## 1. Active First-Order Low-Pass Filter

Files:

```text
experiments/RC_Low_Pass_Filter.ms14
experiments/RC_FILTER_EXPERIMENT_REPORT.md
```

Main components:

```text
R1 = 1 kΩ
R2 = 1 kΩ
C1 = 100 nF
Op-Amp = OPAMP_5T_VIRTUAL
Supply = ±12 V
```

The theoretical cutoff frequency is:

```text
fc = 1 / (2πRC)
```

For the given values:

```text
fc ≈ 1591.55 Hz
```

So the expected cutoff frequency is approximately:

```text
1.59 kHz
```

The passband gain is:

```text
Av0 = -R2 / R1
```

which gives:

```text
Av0 = -1
```

or approximately:

```text
0 dB
```

The Multisim circuit and report are included in the project.

## 2. Variable Regulated DC Power Supply

Files:

```text
experiments/variable_power_supply_bridge_rectifier.ms14
experiments/BRIDGE_RECTIFIER_EXPERIMENT_REPORT.md
```

Main components include:

```text
10:1 Step-Down Transformer
3N246 Full-Wave Diode Bridge
2200 µF Smoothing Capacitor
LM317K Voltage Regulator
10 kΩ Variable Potentiometer
33 Ω Load Resistor
```

The expected output range is approximately:

```text
1.75 V to 13 V
```

The circuit includes the transformer, bridge rectifier, filter capacitor, adjustable regulator, and load.

## 3. RC Phase Shift Audio Oscillator

Files:

```text
experiments/rc_phase_shift_oscillator.ms14
experiments/RC_PHASE_SHIFT_OSCILLATOR_EXPERIMENT_REPORT.md
```

The circuit uses:

```text
LM741 Op-Amp
±15 V supply
3-stage RC network
C1 = C2 = C3 = 0.01 µF
R1 = R2 = R3 = 1.5 kΩ
R4 = 15 kΩ
R5 = 1 MΩ potentiometer
```

The theoretical oscillation frequency is:

```text
f0 = 1 / (2πRC√6)
```

For the given component values:

```text
f0 ≈ 4331.8 Hz
```

or approximately:

```text
4.33 kHz
```

The required loop gain is approximately:

```text
|Av| ≥ 29
```

The project documentation gives a feedback resistance requirement of approximately:

```text
R5 ≥ 435 kΩ
```

for the stated condition.

## 4. Inverting Op-Amp

Files:

```text
experiments/opamp_inverting_amplifier.ms14
experiments/OPAMP_INVERTING_AMPLIFIER_EXPERIMENT_REPORT.md
```

Main components:

```text
LM741
±12 V supply
R1 = 10 kΩ
R2 = 100 kΩ
Input = 2 Vpk at 1 kHz
```

The theoretical gain is:

```text
Av = -R2 / R1
```

Therefore:

```text
Av = -10
```

This corresponds to approximately:

```text
+20 dB
```

The output also has a 180° phase reversal because the amplifier is configured as an inverting amplifier.

The negative sign in the gain represents this phase inversion.

## 5. BJT Common-Emitter Amplifier

The project also includes a BJT common-emitter amplifier experiment.

File:

```text
experiments/bjt_ce_amplifier.ms14
```

Report:

```text
experiments/BJT_CE_AMPLIFIER_EXPERIMENT_REPORT.md
```

The documented circuit uses a:

```text
2N2222 transistor
```

The reported voltage gain is approximately:

```text
Av ≈ -18.2
```

The experiment also includes DC bias validation.

# Experiment Summary

| Experiment                | Multisim File                                 | Report                                           | Main Result          |
| ------------------------- | --------------------------------------------- | ------------------------------------------------ | -------------------- |
| Active Low-Pass Filter    | `RC_Low_Pass_Filter.ms14`                     | `RC_FILTER_EXPERIMENT_REPORT.md`                 | Cutoff ≈ 1.59 kHz    |
| Variable DC Power Supply  | `variable_power_supply_bridge_rectifier.ms14` | `BRIDGE_RECTIFIER_EXPERIMENT_REPORT.md`          | 1.75 V to 13 V       |
| RC Phase Shift Oscillator | `rc_phase_shift_oscillator.ms14`              | `RC_PHASE_SHIFT_OSCILLATOR_EXPERIMENT_REPORT.md` | Frequency ≈ 4.33 kHz |
| Inverting Op-Amp          | `opamp_inverting_amplifier.ms14`              | `OPAMP_INVERTING_AMPLIFIER_EXPERIMENT_REPORT.md` | Gain = -10           |
| BJT CE Amplifier          | `bjt_ce_amplifier.ms14`                       | `BJT_CE_AMPLIFIER_EXPERIMENT_REPORT.md`          | Gain ≈ -18.2         |

The project keeps the native Multisim `.ms14` files and Markdown reports. It does not keep separate `.cir` files.

# Project Structure

```text
ni-multisim-automation/
│
├── README.md
│
├── assets/
│   ├── rc_active_low_pass_filter_schematic.png
│   ├── rc_active_low_pass_filter.png
│   ├── variable_power_supply_bridge_rectifier.png
│   ├── rc_phase_shift_oscillator.png
│   └── opamp_inverting_amplifier.png
│
├── experiments/
│   ├── RC_Low_Pass_Filter.ms14
│   ├── RC_FILTER_EXPERIMENT_REPORT.md
│   │
│   ├── variable_power_supply_bridge_rectifier.ms14
│   ├── BRIDGE_RECTIFIER_EXPERIMENT_REPORT.md
│   │
│   ├── rc_phase_shift_oscillator.ms14
│   ├── RC_PHASE_SHIFT_OSCILLATOR_EXPERIMENT_REPORT.md
│   │
│   ├── opamp_inverting_amplifier.ms14
│   ├── OPAMP_INVERTING_AMPLIFIER_EXPERIMENT_REPORT.md
│   │
│   ├── bjt_ce_amplifier.ms14
│   └── BJT_CE_AMPLIFIER_EXPERIMENT_REPORT.md
│
└── ANALOG_LAB_EXPERIMENTS_MANUAL.md
```

# GitHub Setup

If you want to upload the project to GitHub, first open PowerShell in the project directory:

```powershell
cd "c:\ni multisim sutomation"
```

Initialize Git:

```powershell
git init
```

Add the README and gitignore:

```powershell
git add README.md .gitignore
```

Create the first commit:

```powershell
git commit -m "Initial commit: NI Multisim 14.1 MCP automation setup and documentation"
```

Then set the main branch:

```powershell
git branch -M main
```

Add your GitHub repository:

```powershell
git remote add origin https://github.com/<YOUR-USERNAME>/<YOUR-REPO-NAME>.git
```

Finally push the project:

```powershell
git push -u origin main
```

Replace the username and repository name with your actual GitHub details.

# Final Notes

This project is mainly about connecting **NI Multisim 14.1 with an AI agent through MCP**.

The useful part is that circuit work that normally requires a lot of manual interaction can be controlled through commands such as:

```text
Open this circuit
Change R1 to 10k
Run an AC sweep
Measure Vout
Find the cutoff frequency
Run the experiment
Generate the report
```

The current setup has already been tested with Multisim's COM interface and a working transient LED simulation. It also includes several analog laboratory circuits that can be opened and simulated directly in Multisim.
