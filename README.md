# 112Gbps Per Lane Optical Transceiver Analog Front End
## An Electronic-Photonic Co-Design (EPCD) 400G/800G PAM4 Optical Transceiver Architecture

> **Technology Node:** IHP SG13G2 (130nm SiGe BiCMOS)  
> **Target Tapeout:** OpenMPW Shuttle (October 2026)  
> **Status:** Phase 1.II - Schematic Capture & AC/Transient Validation (`Xschem` / `Qucs-S` and `ngspice`)  

## Project Scope and Context
Commercial 400G and 800G optical engines are massive multidisciplinary systems requiring dedicated ASIC DSP and Photonic layout teams. This repository does not claim to replace a commercial transceiver product. Instead this is an independent open source Silicon IP development project. The strict physical scope of Phase 1 is to architect, layout, and physically fabricate the standalone SiGe BiCMOS Analog Front End (TIA and Cascode Driver) via the IHP SG13G2 OpenMPW shuttle. The immediate objective is bare-die GSG RF characterization to validate the silicon topology and S-parameters, establishing a fully proven, physical IP foundation ready for future monolithic SG25H5EPIC scaling.

---

## 1. Abstract & Industry Motivation
As global datacenter interconnects scale beyond 3.2T and 51.2T aggregate switch capacities, the physical layer is bottlenecked by the analog boundaries between the digital domain and the optical domain. The industry standard for intra-datacenter connectivity (e.g., 400G-DR4 for 500m reach) relies on 56 GBaud PAM4 (Pulse Amplitude Modulation) over Single-Mode Fiber (SMF) using Intensity Modulation and Direct Detection (IM/DD). [^1]

This repository contains the system architecture and open-source EDA tapeout files for a complete Analog Front-End (AFE) comprising a 40 Gbps Transimpedance Amplifier (TIA) and a 30 GHz Mach-Zehnder Modulator (MZM) Driver. This project bridges the physical voltage gap between bleeding-edge 5nm DSPs and high-voltage Silicon Photonics.

---

## 2. The Data Journey: The Case for EPCD
To understand why this specific SiGe architecture is required, one must trace the complete lifecycle of the data through the network rack.

![ADC: Analog-to-digital converter, AFE: Analog Front-End, AGC: Automatic Gain Control, ASIC: Application-specific integrated circuit, CDR: Clock data recovery, DAC: Digital-to-analog converter, DSP: Digital signal processing, MZM: Mach–Zehnder modulator, PD: Photodiode, TIA: Transimpedance amplifier](./static/OC_system.png)



1. **The Switch ASIC (The Origin):** Digital routing logic (e.g., Broadcom Tomahawk) outputs highly parallel, low-speed digital data across the server motherboard. [^1][^2]
2. **The DSP & DAC (The 5nm Limit):** Inside the transceiver module, a 5nm CMOS DSP acts as the core of the SerDes (Serializer/Deserializer). On the Transmit (TX) path, it serializes the parallel data, applies Forward Error Correction (FEC) to embed parity bits, and mathematically pre-distorts the PAM4 levels to combat expected channel loss. A Phase-Locked Loop (PLL) synthesizes an ultra-high-frequency clock, driving the internal Digital-to-Analog Converter (DAC) to output the physical wave. However, to prevent dielectric breakdown, 5nm CMOS cannot safely output more than ~0.75V, requiring the signal to be handed off to a high-voltage Analog Front End (AFE). On the Receive (RX) path, the architecture reverses. A time-interleaved Analog-to-Digital Converter (ADC) digitizes the heavily degraded incoming wave. Because the data stream lacks a dedicated clock line, a Clock and Data Recovery (CDR) loop continuously analyzes the digitized bits, steering the ADC's sampling phase to stay perfectly aligned with the PAM4 eyes. Finally, the RX DSP mathematically equalizes the signal to restore the four voltage levels, and decodes the FEC to correct any physical transmission errors before passing the pristine data back to the ASIC. [^1][^2][^3]
3. **The EIC Driver (This Project):** A Silicon MZM requires a massive 3V swing to physically alter its refractive index. The 5nm DAC cannot drive it. The IHP 130nm SiGe Cascode Driver acts as the high-voltage translator, magnifying the 0.75V signal to 3V without sacrificing 40 GHz bandwidth.
4. **The PIC & Fiber Link:** The MZM chops a 1310nm Continuous Wave (CW) DFB laser into PAM4 pulses, sending them across 500m of SMF.
5. **The Rx Recovery:** At the destination, the light hits a Ge-on-Si p-i-n Photodiode, generating a microscopic current. The SiGe TIA (this project) converts this current back to voltage. An ADC digitizes it, and the receiving DSP cleans the signal before handing it to the destination Switch ASIC.

---

### 3. The EIC Driver and TIA

#### 3.1 MZM Driver

#### The Core Challenge
A state-of-the-art Electronic IC (EIC) driver must solve the fundamental mismatch between the physical limitations of deep-submicron digital logic and the optical physics of Silicon Photonics. 

#### The Optical Physics (Why the MZM needs $3V$)
Unlike Lithium Niobate ($LiNbO_3$) modulators that rely on the linear Pockels effect, Silicon Mach-Zehnder Modulators (MZMs) must rely on the Plasma Dispersion Effect. To alter the refractive index of the silicon waveguide and create a phase shift, we must physically inject or deplete massive amounts of free carriers (electrons and holes) in the PN junction. This is a weak effect in silicon. To achieve a full pi phase shift (characterized by the $V_\pi * L$ metric), a typical Si-MZM requires a massive differential voltage swing often $2V$ to $4V$ peak-to-peak to fully open the optical eye.[^1][^3]

#### The CMOS Deficit (Why the DSP cannot drive it)
The advanced $5nm$ CMOS nodes used for the DSP and DAC achieve their incredible speeds and density by shrinking the transistor oxide thickness to a few dozen atoms. Because the oxide is so thin, its dielectric breakdown voltage drops drastically. A $5nm$ DAC physically cannot output a swing larger than $0.75V$ without permanently destroying its own transistors. Therefore, the signal must leave the $5nm$ die and enter a dedicated, high-voltage Analog Front End (AFE).[^1][^2][^3]

#### The Technology Choice (Why IHP $130nm$ SiGe BiCMOS)
Analog design is governed by the Johnson Limit, a fundamental physical law stating that a transistor's breakdown voltage is inversely proportional to its maximum speed ($f_T$). Standard $130nm$ CMOS can survive $3V$, but it is far too slow for $112 Gbps$ ($56 GBaud$). By utilizing the IHP SG13G2 SiGe BiCMOS process, we leverage Heterojunction Bipolar Transistors (HBTs). The Germanium doping in the base of the HBT accelerates electron transit times, allowing the transistors to achieve staggering speeds ($f_T$ / $f_max$ > $300 GHz$) while still maintaining a safe breakdown voltage ($BV_CEO$) high enough to drive the photonics. [^4][^5]

#### The Topology (Why a Cascode?)
The EIC driver must provide a massive voltage gain (translating $0.75V$ to $3V$) without sacrificing the $40 GHz$ analog bandwidth required for $56 GBaud$ PAM4. A standard Common-Emitter amplifier could provide the gain, but the parasitic base-to-collector capacitance would be multiplied by the gain (the Miller Effect), creating a massive low-pass filter that destroys the high-frequency bandwidth. To solve this, this project utilizes a Cascode topology. The bottom Common-Emitter transistor provides the raw transconductance (gain), while the top Common-Base transistor buffers the output, effectively shielding the input from the voltage swing. This eliminates the Miller capacitance, preserving the $40 GHz$ bandwidth. Furthermore, the Cascode stacks the voltage across two devices, protecting both SiGe HBTs from exceeding their individual breakdown limits while delivering the full $3V$ swing to the MZM.[^2][^3]


#### 3.2 TIA

#### The Core Challenge
While the TX Driver is a battle of brute-force high voltage, the RX Transimpedance Amplifier (TIA) is a battle of extreme sensitivity. The TIA must translate microscopic fluctuations in optical power into a clean, macroscopic electrical voltage for the $5nm$ ADC, all without adding enough thermal noise to destroy the $112 Gbps$ PAM4 eyes.

#### The Optical Physics (Why the PD current is so weak)
Silicon is transparent to standard $1550nm$ telecom light, so Electronic-Photonic ICs use Epitaxial Germanium Photodiodes (Ge-PD) to absorb the photons. The physics of this absorption is governed by the Responsivity metric ($R$, in $A/W$). By the time the optical signal survives the fiber loss, coupling loss, and waveguide attenuation, the received optical power is often in the microwatt range (e.g., $-10 dBm$). Multiply this by a typical responsivity of $0.8 A/W$, and the Ge-PD outputs a pathetic, noisy signal in the low microamps. [^1][^2][^3]

#### The Bandwidth/Noise Tradeoff (Why not just use a resistor?)
To convert that microamp current into a voltage for the ADC, we need a transimpedance gain ($Z_T$). We cannot just dump the current across a simple $50 \Omega$ resistor. First, the resulting voltage would be too small (microvolts). Second, the intrinsic parasitic capacitance of the Ge-PD ($C_{pd}$, typically $15fF$ to $30fF$) combined with a high-value resistor creates an RC low-pass filter that completely destroys the $40 GHz$ bandwidth required for $56 GBaud$ operation. Active amplification is mandatory. [^2][^3]

#### The Technology Choice (Why IHP $130nm$ SiGe BiCMOS)
In a TIA, the Signal-to-Noise Ratio (SNR) of the entire receiver is dominated by the very first transistor the photodiode touches. The total Input-Referred Noise Current ($I_{in,noise}$) dictates the ultimate sensitivity of the link. IHP's SiGe Heterojunction Bipolar Transistors (HBTs) have a massive advantage over deep-submicron CMOS here. Because HBTs provide exponentially higher transconductance ($g_m$) for a given bias current, and possess extremely low base-resistance, SiGe can achieve an ultra-low noise figure at extreme high frequencies that pure CMOS simply cannot match.[^4][^5]

#### The Topology (Why a Modified Cherry-Hooper?)
To break the brutal RC bandwidth limit imposed by the photodiode without destroying the signal-to-noise ratio, this project utilizes a Modified Cherry-Hooper topology. It combines the best of two legendary architectures. The first stage is a classic Shunt-Feedback amplifier; by wrapping a feedback resistor ($R_F$) around an inverting core, it mathematically divides the input impedance by the loop gain. This safely isolates the photodiode capacitance ($C_{pd}$) from the dominant pole while maintaining an ultra-low thermal noise floor. However, a pure shunt-feedback cannot provide enough voltage gain at $40 GHz$ without sacrificing bandwidth. Therefore, the signal is immediately fed into a Cherry-Hooper post-amplifier. By intentionally driving a high-impedance transconductance stage into an ultra-low-impedance transimpedance stage, this cascade effectively eliminates the intermediate RC time constants, pushing the bandwidth poles out to massive frequencies. Finally, to squeeze out the ultimate $112 Gbps$ performance, the design incorporates on-chip Inductive Peaking utilizing spiral inductors in the IHP metal stack to resonate out residual parasitic capacitances, keeping the PAM4 eyes wide open for the ADC. [^2][^3][^4]

## 4. Target Specifications (Commercial 56 GBaud PAM4)
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


## 5. Technology Selection: IHP SG13G2 SiGe BiCMOS
This project relies entirely on the open-source IHP SG13G2 130nm process. While sub-micron CMOS dominates digital logic, it suffers from the Johnson Limit (the strict physical trade-off between transistor speed and breakdown voltage). 

By utilizing Silicon-Germanium Heterojunction Bipolar Transistors (HBTs):
* **Transit Frequency ($f_T / f_{max}$):** Exceeds 300 GHz, providing the massive bandwidth headroom required for the Cherry-Hooper and Cascode topologies.
* **Breakdown Voltage ($BV_{CEO}$):** Allows the Tx Driver to safely deliver 3V peak-to-peak swings that would physically destroy a 5nm FinFET.



*Note: For this Phase 1.II SG13G2 simulation to perfectly prepare the architecture for Phase 2 scaling the ultra low 20 fF parasitic capacitance of a monolithic Ge p i n Photodiode is modeled directly in ngspice replacing traditional high capacitance flip chip models.*

---

### 6. Project Roadmap and Execution Phases

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

## 7. Future Outlook & Scaling
While Silicon Photonics (SiPh) and 2.5D integration represent the current high-volume datacenter standard, this architecture is designed with forward-compatibility for next-generation boundary conditions:

* **Monolithic EPIC (e.g., IHP SG25H5EPIC):** Future iterations of this architecture will evaluate fully monolithic Electronic-Photonic Integrated Circuits, where the SiGe BiCMOS and silicon waveguides are fabricated on the exact same SOI substrate and unlocking 224 Gbps/lane physical limits.
* **Thin-Film Lithium Niobate (TFLN):** TFLN is rapidly emerging due to its ultra-low $V_\pi$ and massive bandwidth. While TFLN can theoretically be driven by weaker CMOS drivers, maintaining this driver's robust 3V capability allows optical engineers to drastically shrink the physical length ($L$) of the TFLN modulator (since $L = V_\pi L / V_{swing}$), saving premium wafer real estate and lowering optical insertion loss.


---

## 8. Open-Source EDA Toolchain
This project bypasses proprietary commercial NDA firewalls by utilizing a fully open-source silicon compiler toolchain:
* **Schematic Capture:** `Xschem` / `Qucs-S`
* **Simulation Engine:** `ngspice` (AC, Transient, and Noise sweeps)
* **Physical Layout:** `KLayout` /`Magic` 
* **Verification (DRC/LVS):** `KLayout` /`Magic`


## 9. References
[^1]: Agrell, E., Karlsson, M., Poletti, F., Namiki, S., Chen, X. (Vivian), Rusch, L. A., Puttnam, B., Bayvel, P., Schmalen, L., Tao, Z., Kschischang, F. R., Alvarado, A., Mukherjee, B., Casellas, R., Zhou, X., van Veen, D., Mohs, G., Wong, E., Mecozzi, A., … Uysal, M. (2024). Roadmap on optical communications. Journal of Optics, 26(9), 093001. https://doi.org/10.1088/2040-8986/ad261f
[^2]: Sackinger, E. (2005). Broadband circuits for Optical Fiber Communication (1). Wiley-Interscience.   
[^3]: Razavi, B. (2012). Design of integrated circuits for Optical Communications (2nd edition). Wiley.  
[^4]: IHP GmbH - Leibniz Institute for High Performance Microelectronics, 130nm BiCMOS open source PDK, dedicated for analog, mixed signal and RF design, commit 2dcbf07. GitHub. Accessed: Mar. 30, 2026. Online. Available: https://github.com/IHP-GmbH/IHP-Open-PDK
[^5]: Herman, K., Herfurth, N., Henkes, T., Andreev, S., Scholz, R., Müller, M., Krattenmacher, M., Pretl, H., & Grabinski, W. (2024). On the versatility of the IHP Bicmos Open Source and manufacturable PDK: A step towards the future where anybody can design and build a chip. IEEE Solid-State Circuits Magazine, 16(2), 30–38. https://doi.org/10.1109/mssc.2024.3372907 