<div align="center">

<img src="https://img.shields.io/badge/Analog%20BMS-Intelligent%20Li--ion%20%26%20LiPo%20Battery%20Management-0B3D91?style=for-the-badge&logoColor=white" alt="Analog BMS Banner"/>

<br/>

<img src="https://img.shields.io/badge/Hardware-Analog%20Circuitry-0B3D91?style=flat-square"/>
<img src="https://img.shields.io/badge/SPICE-Simulation-0B3D91?style=flat-square"/>
<img src="https://img.shields.io/badge/KiCad-EDA-0B3D91?style=flat-square&logo=kicad&logoColor=white"/>
<img src="https://img.shields.io/badge/Li--ion%20%2F%20LiPo-Battery%20Protection-0B3D91?style=flat-square"/>

<br/>

[![GitHub](https://img.shields.io/badge/GitHub-Stanislus29-0B3D91?style=flat-square\&logo=github\&logoColor=white)](https://github.com/Stanislus29)
[![Repository](https://img.shields.io/badge/Repository-analog--bms-0B3D91?style=flat-square\&logo=github\&logoColor=white)](https://github.com/Stanislus29/analog-bms)

<br/>

*An analog battery management system for electrical and thermal protection of lithium-ion and lithium-polymer batteries.*

</div>

---

## Overview

This project is the design and construction of an **analog Battery Management System (BMS)** for lithium-ion and lithium-polymer batteries.

The system is built entirely around discrete analog and mixed-signal components rather than a microcontroller-based monitoring system. Its purpose is to provide independent protection against electrical and thermal conditions that can lead to unsafe battery operation.

The architecture is divided into independent protection and power-management subcircuits:

* Overcharge protection
* Thermal protection
* CC-CV charging
* Over-discharge protection
* Power-path management

Each subsystem is derived from its underlying electrical behaviour and can be analysed independently before being integrated into the complete system.

The design was developed through mathematical analysis, SPICE simulation, schematic design, physical construction, and experimental validation.

---

## System Architecture

```text
                         ┌──────────────────────┐
                         │   External Supply    │
                         │       USB-C          │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    Power Path        │
                         │    Management        │
                         └──────────┬───────────┘
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                     ▼                             ▼
              ┌─────────────┐               ┌─────────────┐
              │ Overcharge  │               │  Thermal    │
              │ Protection  │               │ Protection  │
              └──────┬──────┘               └──────┬──────┘
                     │                             │
                     └──────────────┬──────────────┘
                                    ▼
                              ┌─────────────┐
                              │     NOR     │
                              │    Logic    │
                              └──────┬──────┘
                                     │
                                     ▼
                              ┌─────────────┐
                              │    CC-CV    │
                              │   Charger   │
                              └──────┬──────┘
                                     │
                                     ▼
                                ┌─────────┐
                                │ Battery │
                                └────┬────┘
                                     │
                                     ▼
                              ┌─────────────┐
                              │ Over-discharge│
                              │  Protection │
                              └─────────────┘
```

The charging path is enabled only when the battery satisfies the relevant voltage and temperature conditions.

---

## Protection Subsystems

<details>
<summary><b>Overcharge Protection</b></summary>
<br/>

The overcharge protection circuit monitors the battery voltage and prevents charging when the voltage reaches the configured upper threshold.

The implementation uses an **MCP6561-OT comparator** configured as an inverting Schmitt trigger. The design target is approximately **4.2 V** for a single lithium cell, with hysteresis incorporated to prevent unwanted switching around the threshold.

The simulated transition occurred at approximately **4.32 V**, giving a deviation of approximately **0.12 V** from the theoretical design value. The physical charging tests demonstrated the expected charging inhibit and enable behaviour.

</details>

<details>
<summary><b>Thermal Protection</b></summary>
<br/>

Battery temperature is monitored using a **10 kΩ NTC thermistor**.

The protection circuit is configured around a target thermal cutoff of approximately **45 °C**. When the temperature condition exceeds the configured threshold, the charging path is disabled.

The thermal protection stage also incorporates hysteresis to prevent repeated switching around the cutoff temperature.

Due to the safety implications of intentionally heating a lithium battery, physical thermal testing was not performed. The thermal protection behaviour was therefore evaluated through circuit simulation.

</details>

<details>
<summary><b>Charging Control</b></summary>
<br/>

The overcharge and thermal protection outputs are combined using a **74AHC1G02 NOR gate**.

Charging is enabled only when both protection conditions indicate that charging is safe:

```text
OC_OUT = LOW
TH_OUT = LOW

        ↓

NOR = HIGH

        ↓

Charging Enabled
```

If either an overcharge or thermal fault is detected, the NOR output disables the charging path.

An LED indicator is also incorporated into the logic stage to provide a visual indication of the charging state.

</details>

<details>
<summary><b>CC-CV Charging</b></summary>
<br/>

The charging subsystem implements a constant-current / constant-voltage charging profile.

A **TLV76701DRVx LDO** is used to regulate the charging voltage, while a sense resistor establishes the current-limiting behaviour.

The design targets approximately **4.2 V** for a single lithium cell.

A **SS14 Schottky diode** is included to prevent unwanted reverse current or backfeeding between the charging and battery paths.

Simulation produced an output voltage of approximately **4.18 V**, with an initial simulated charging current of approximately **972 mA** at a battery voltage of 3.0 V.

</details>

<details>
<summary><b>Over-discharge Protection</b></summary>
<br/>

The over-discharge stage prevents the battery from being driven below its configured minimum operating voltage.

A **BSS215P P-channel MOSFET** is used as the switching element, with an auxiliary 3 V coin-cell reference providing the gate control.

The design uses a target cutoff of approximately **3.0 V**.

</details>

<details>
<summary><b>Power-Path Management</b></summary>
<br/>

The power-path stage manages the relationship between the external supply and the battery output.

Two **SS14 Schottky diodes** provide isolation between the relevant supply paths while allowing the system to select the available power source.

The output is provided through the `BOUT` node, with a bleed resistor included in the power-path circuitry.

</details>

---

## Design Methodology

The BMS was developed from the circuit level upward.

Rather than treating the system as a black-box BMS, each protection function was first analysed according to its electrical behaviour and then implemented as an independent circuit.

The main mathematical and electrical foundations include:

* Voltage-divider analysis
* Comparator transfer characteristics
* Schmitt-trigger hysteresis
* MOSFET switching behaviour
* LDO regulation
* Current-sense relationships
* Thermistor temperature dependence
* Boolean logic for protection-state combination

The resulting subsystems were simulated independently before being integrated into the complete architecture.

---

## Simulation

The design was validated using SPICE-based transient simulations.

The simulations were configured with a transient analysis of:

```text
.tran 5u 80m
```

with a maximum timestep of:

```text
.options maxstep=100n
```

The Gear integration method was used alongside tighter convergence parameters for the circuit simulations.

Simulation was used to examine:

* Protection threshold behaviour
* Comparator switching
* Thermal cutoff behaviour
* Charging current
* CC-CV voltage regulation
* Power-path behaviour
* Transient response

The repository contains the relevant simulation projects and circuit files.

---

## Experimental Validation

Following simulation, the circuit was physically constructed and tested.

The experimental work focused on verifying that the physical implementation followed the behaviour predicted by the circuit analysis and SPICE models.

### Overcharge Protection

The theoretical cutoff was designed around **4.2 V**.

The simulated transition occurred around **4.32 V**. Physical bench-supply testing demonstrated the expected charging inhibit and enable behaviour.

### CC-CV Charging

The simulated LDO output was approximately **4.18 V**, with an initial current of approximately **972 mA** at a 3.0 V battery voltage.

Physical testing similarly demonstrated the transition from constant-current charging towards constant-voltage regulation.

### Thermal Protection

The theoretical thermal cutoff was approximately **45 °C**.

Simulation demonstrated the expected protection transition. Physical thermal testing was not performed because intentionally heating a lithium battery introduces unnecessary safety risk.

---

## Main Components

| Component              | Function                                                  |
| :--------------------- | :-------------------------------------------------------- |
| **MCP6561-OT**         | Voltage and thermal comparator                            |
| **TLV76701DRVx**       | CC-CV charging regulator                                  |
| **74AHC1G02**          | NOR logic for protection-state control                    |
| **BSS215P**            | Over-discharge MOSFET switching                           |
| **10 kΩ NTC**          | Temperature sensing                                       |
| **SS14**               | Schottky isolation / reverse-current protection           |
| **Passive components** | Voltage sensing, hysteresis, current limiting and biasing |

---

## Hardware

The physical BMS implementation is based on the same architecture validated through simulation.

The circuit was constructed using the selected analog components and tested against the expected protection and charging behaviour.

The repository contains the hardware design and simulation artefacts used throughout the development process.

---

## Design Scope

The current implementation focuses on the core electrical and thermal protection functions of a lithium battery management system.

### Included

* Analog overcharge protection
* Thermal protection
* CC-CV charging
* Over-discharge protection
* USB-C input power
* USB-A output power path
* Adjustable protection thresholds
* Analog protection-state logic
* SPICE simulation
* Physical prototype construction and testing

### Not Included

The current design does not implement:

* Cell balancing
* State-of-charge estimation
* Digital battery communication
* Battery monitoring software
* Production-grade battery certification
* Large-scale field deployment

The architecture is therefore best treated as an engineering prototype and research/design implementation rather than a certified production BMS.

---

## Repository Structure

```text
analog-bms/
├── schematics/              # KiCad schematics and hardware design
├── simulations/             # Main SPICE simulation files
├── sim-LDO-SUBCKT/          # LDO subsystem simulations
├── sim-OC-SUBCKT/           # Overcharge protection simulations
├── sim-PP-MGT/              # Power-path management simulations
├── sim-lm311/               # Comparator-related simulations
├── scripts/                 # Supporting scripts
└── .gitignore
```

---

## Tools

The project uses a combination of electronic design and circuit-simulation tools:

| Tool          | Purpose                             |
| :------------ | :---------------------------------- |
| **KiCad 7.x** | Schematic and PCB/electronic design |
| **LTspice**   | Circuit simulation                  |
| **PSPICE**    | Circuit simulation and analysis     |
| **ngspice**   | SPICE-based circuit simulation      |

---

## Future Development

Potential extensions to the current architecture include:

* Multi-cell battery support
* Cell balancing
* State-of-charge estimation
* Battery state monitoring
* Digital communication interfaces
* Microcontroller-assisted monitoring
* More comprehensive thermal validation
* PCB optimisation
* Extended fault-condition testing
* Further threshold and component-tolerance analysis

---

## Project Documentation

The associated project documentation provides the mathematical derivations, circuit analysis, simulation methodology, implementation details, and experimental results behind the design.

The repository is intended to preserve the engineering artefacts of the project, including its schematics and simulation work.

---

## Collaboration

This project was developed collaboratively.

<div align="center">

<a href = "https://github.com/Stanislus29">
<img src="https://img.shields.io/badge/Somtochukwu%20Stanislus%20Emeka--Onwuneme-Project%20Owner%20%26%20Maintainer-0B3D91?style=for-the-badge&logo=github&logoColor=white"/>
</a>

<br/>

<a href = "https://github.com/kayeh23">
<img src="https://img.shields.io/badge/Aseda%20Ayeh--Bampoe-Hardware%20Collaborator-0B3D91?style=for-the-badge&logo=github&logoColor=white"/>
</a>

<br/>

<img src="https://img.shields.io/badge/Paul%20Eshoiza%20Awolu-Collaborator%20%26%20Project%20Report%20Author-0B3D91?style=for-the-badge&logoColor=white"/>

</div>

---

## Safety Notice

This project involves lithium battery charging and protection circuitry.

The design is an engineering and research prototype and should **not** be treated as a certified battery-safety device or production-ready BMS without additional validation, qualification, protection analysis, and compliance testing.

Lithium batteries can present significant electrical, thermal, and fire hazards when improperly charged, discharged, damaged, or handled.

---

## License

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

<div align="center">

**Somtochukwu Stanislus Emeka-Onwuneme**

### Connect with me

<a href="https://twitter.com/vzyengineer">
  <img src="https://img.shields.io/badge/X-000000?style=flat&logo=x&logoColor=white" />
</a><a href="https://linkedin.com/in/emekasomto3">
  <img src="https://img.shields.io/badge/LinkedIn-0077B5?style=flat&logo=linkedin&logoColor=white" />
</a>

<br/><br/>

<a href="https://github.com/Stanislus29">
  <img src="https://img.shields.io/badge/GitHub-Stanislus29-0B3D91?style=flat-square&logo=github&logoColor=white" />
</a>

</div>
