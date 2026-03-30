# 112Gbps Per Lane Optical Transceiver Analog Front End
## An Electronic-Photonic Co-Design (EPCD) 400G/800G PAM4 Optical Transceiver Architecture

> **Technology Node:** IHP SG13G2 (130nm SiGe BiCMOS)  
> **Target Tapeout:** OpenMPW Shuttle (October 2026)  
> **Status:** Phase 1.II - Schematic Capture & AC/Transient Validation (`Xschem` and `ngspice`)  

## Project Scope and Context
Commercial 400G and 800G optical engines are massive multidisciplinary systems requiring dedicated ASIC DSP and Photonic layout teams. This repository does not claim to replace a commercial transceiver product. Instead this is an independent open source Silicon IP development project. The strict physical scope of Phase 1 is to architect, layout, and physically fabricate the standalone SiGe BiCMOS Analog Front End (TIA and Cascode Driver) via the IHP SG13G2 OpenMPW shuttle. The immediate objective is bare-die GSG RF characterization to validate the silicon topology and S-parameters, establishing a fully proven, physical IP foundation ready for future monolithic SG25H5EPIC scaling.

---

## 1. Abstract & Industry Motivation
As global datacenter interconnects scale beyond 3.2T and 51.2T aggregate switch capacities, the physical layer is bottlenecked by the analog boundaries between the digital domain and the optical domain. The industry standard for intra-datacenter connectivity (e.g., 400G-DR4 for 500m reach) relies on 56 GBaud PAM4 (Pulse Amplitude Modulation) over Single-Mode Fiber (SMF) using Intensity Modulation and Direct Detection (IM/DD).

This repository contains the system architecture and open-source EDA tapeout files for a complete Analog Front-End (AFE) comprising a 40 Gbps Transimpedance Amplifier (TIA) and a 30 GHz Mach-Zehnder Modulator (MZM) Driver. This project bridges the physical voltage gap between bleeding-edge 5nm DSPs and high-voltage Silicon Photonics.

---

## 2. Target Specifications (Commercial 56 GBaud PAM4)
The AFE is engineered to strictly adhere to the boundary conditions of a 56 GBaud PAM4 signal chain, scaling to 400G via 4 parallel lanes ($4 \times 112$ Gbps) or 800G via 8 parallel lanes.

### Rx Path: Transimpedance Amplifier (TIA)
| Parameter | Target Spec | Physical Constraint Justification |
| :--- | :--- | :--- |
| **Analog Bandwidth** | > 40 GHz | Nyquist requirement for 56 GBaud symbol rate with margin. |
| **Transimpedance Gain** | 70 dBΩ | Recovers micro-ampere photodiode currents into analyzable voltages. |
| **Input Capacitance** | 20 fF | Assumed parasitic load of an Ge-on-Si Photodiode. |
| **Linearity (THD)** | < 3% | Critical for maintaining the 4 distinct optical eye levels of PAM4. |

### Tx Path: Modulator Driver (Differential Cascode)
| Parameter | Target Spec | Physical Constraint Justification |
| :--- | :--- | :--- |
| **Analog Bandwidth** | > 30 GHz | Matches 112 Gbps DSP/DAC output. |
| **Output Voltage Swing** | 2.5V - 3.0V (Diff) | Required to overcome the high $V_\pi$ of standard Silicon MZMs. |
| **Output Impedance** | 50 Ω | Perfect matching to the MZM Traveling Wave Electrode (TWE). |
| **Group Delay Variation**| < 20 ps | Ensures all frequency components of the PAM4 pulse arrive simultaneously. |

---

## 3. The Data Journey: The Case for EPCD
To understand why this specific SiGe architecture is required, one must trace the complete lifecycle of the data through the network rack.



1. **The Switch ASIC (The Origin):** Digital routing logic (e.g., Broadcom Tomahawk) outputs highly parallel, low-speed digital data across the server motherboard.
2. **The DSP & DAC (The 5nm Limit):** Inside the transceiver module, a 5nm CMOS DSP serializes the data, applies Forward Error Correction (FEC), and mathematically pre-distorts the PAM4 levels. The internal Digital-to-Analog Converter (DAC) outputs the physical wave. To prevent dielectric breakdown, 5nm CMOS cannot safely output more than ~0.75V.
3. **The EIC Driver (This Project):** A Silicon MZM requires a massive 3V swing to physically alter its refractive index. The 5nm DAC cannot drive it. The IHP 130nm SiGe Cascode Driver acts as the high-voltage translator, magnifying the 0.75V signal to 3V without sacrificing 40 GHz bandwidth.
4. **The PIC & Fiber Link:** The MZM chops a 1310nm Continuous Wave (CW) DFB laser into PAM4 pulses, sending them across 500m of SMF.
5. **The Rx Recovery:** At the destination, the light hits a Ge-on-Si p-i-n Photodiode, generating a microscopic current. The SiGe TIA (this project) converts this current back to voltage. An ADC digitizes it, and the receiving DSP cleans the signal before handing it to the destination Switch ASIC.

---

## 4. Technology Selection: IHP SG13G2 SiGe BiCMOS
This project relies entirely on the open-source **IHP SG13G2 130nm** process. While sub-micron CMOS dominates digital logic, it suffers from the Johnson Limit (the strict physical trade-off between transistor speed and breakdown voltage). 

By utilizing Silicon-Germanium Heterojunction Bipolar Transistors (HBTs):
* **Transit Frequency ($f_T / f_{max}$):** Exceeds 300 GHz, providing the massive bandwidth headroom required for the Cherry-Hooper and Cascode topologies.
* **Breakdown Voltage ($BV_{CEO}$):** Allows the Tx Driver to safely deliver 3V peak-to-peak swings that would physically destroy a 5nm FinFET.



*Note: For this Phase 1.II SG13G2 simulation to perfectly prepare the architecture for Phase 2 scaling the ultra low 20 fF parasitic capacitance of a monolithic Ge p i n Photodiode is modeled directly in ngspice replacing traditional high capacitance flip chip models.*

---

### 5. Project Roadmap and Execution Phases

This project follows a strict commercial silicon de-risking strategy. The architecture is split into a physical bare-die electrical validation phase, followed by a monolithic scaling phase.

#### Phase 1: IHP SG13G2 Electrical IP Validation [ACTIVE]
The strict physical scope of Phase 1 is to architect, layout, and physically fabricate the standalone SiGe BiCMOS Analog Front End (TIA and Cascode Driver) via the IHP SG13G2 OpenMPW shuttle. The immediate objective is bare-die GSG RF characterization to validate the silicon topology and S-parameters.

* **Phase 1.I: Architecture Definition and Market Scoping [COMPLETED]**
  Execution of deep literature reviews, boundary condition definitions, and commercial technology gap analysis between 5nm DSP limitations and 56 GBaud PAM4 optical requirements.

* **Phase 1.II: Schematic Capture and AC/Transient Simulation [CURRENT]**
  Active drafting of both the receiver TIA and the transmitter driver topologies in Xschem. Executing rigorous ngspice AC sweeps, transient eye-diagram simulations, and noise analysis to secure the 40 GHz bandwidth and 70 dBΩ gain targets for the Rx path, alongside the 30 GHz bandwidth and massive 3V differential voltage swing required for the Tx path.

* **Phase 1.III: Physical Layout and LVS/DRC Verification [UPCOMING]**
  Translating schematics into physical GDS geometry utilizing KLayout. Running exhaustive Design Rule Checks (DRC) and Layout Versus Schematic (LVS) verification to ensure foundry compliance before submission.

* **Phase 1.IV: Post-Layout Parasitic Extraction (PEX) & EM Simulation [UPCOMING]**
  Extracting the physical RLCK parasitics from the completed GDS layout. Running rigorous electromagnetic (EM) simulations on the high-speed RF routing and GSG pad structures to verify 50Ω transmission line impedance matching. Re-simulating the active core with the extracted netlist to guarantee the 40 GHz bandwidth targets survive the physical metallization.

* **Phase 1.V: Tapeout Submission and Foundry Fabrication [TARGET: OCT 2026]**
  Final GDS freeze and submission to the IHP SG13G2 OpenMPW shuttle. The silicon will be physically poured and etched at the IHP foundry.

* **Phase 1.VI: Bare-Die Measurement and RF Characterization [POST-FAB]**
  Mounting the returned raw silicon die on an RF probe station. Utilizing high-frequency GSG (Ground-Signal-Ground) micro-probes and a Vector Network Analyzer (VNA) to extract physical S-parameters and validate the silicon against the ngspice simulation models.


#### Phase 2: IHP SG25H5EPIC Monolithic Integration [ROADMAP]
Once the core SiGe BiCMOS amplifier topologies are physically proven in Phase 1, the IP will be ported to the premium IHP SG25H5EPIC monolithic node. This phase eliminates the parasitic inductance of 2.5D flip-chip micro-bumps by integrating zero-pad-capacitance Germanium photodiodes and Silicon Mach-Zehnder Modulators directly into the same SOI substrate as the 130nm RFIC drivers, unlocking true Electronic-Photonic Co-Design (EPCD) for next-generation Terabit architectures.

## 6. Future Outlook & Scaling
While Silicon Photonics (SiPh) and 2.5D integration represent the current high-volume datacenter standard, this architecture is designed with forward-compatibility for next-generation boundary conditions:

* **Monolithic EPIC (e.g., IHP SG25H5EPIC):** Future iterations of this architecture will evaluate fully monolithic Electronic-Photonic Integrated Circuits, where the SiGe BiCMOS and silicon waveguides are fabricated on the exact same SOI substrate and unlocking 224 Gbps/lane physical limits.
* **Thin-Film Lithium Niobate (TFLN):** TFLN is rapidly emerging due to its ultra-low $V_\pi$ and massive bandwidth. While TFLN can theoretically be driven by weaker CMOS drivers, maintaining this driver's robust 3V capability allows optical engineers to drastically shrink the physical length ($L$) of the TFLN modulator (since $L = V_\pi L / V_{swing}$), saving premium wafer real estate and lowering optical insertion loss.


---

## 7. Open-Source EDA Toolchain
This project bypasses proprietary commercial NDA firewalls by utilizing a fully open-source silicon compiler toolchain:
* **Schematic Capture:** `Xschem`
* **Simulation Engine:** `ngspice` (AC, Transient, and Noise sweeps)
* **Physical Layout:** `KLayout`
* **Verification (DRC/LVS):** `Magic` / `Netgen`
