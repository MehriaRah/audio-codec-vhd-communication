# Synchronous Serial Communication with an Audio Codec

## 📌 Project Overview
This repository contains a digital design project implemented in VHDL that establishes **Synchronous Serial Communication** between an FPGA block and an external audio codec[cite: 648, 1102]. The core objective was designing and implementing a behavioral model for a **Dual-Channel Parallel-to-Serial Converter (`PAR_TO_SER_LR`)** [cite: 1106] [cite_start]that handles 16-bit audio samples for both left and right audio channels using a shared internal shift register architecture[cite: 1110, 1124].

The design operates within a larger audio system loopback, receiving continuous analog signals via `LINE IN`, digitizing and serializing them through the FPGA processing matrix, and outputting them back via `LINE OUT`[cite: 653, 680].

## 🛠️ Hardware & Tools
* **Target Hardware Platform:** Modsys 2.0 Development Board [cite: 1104]
* **Main Clock Core:** On-board Oscillator ($f_{SYSCLK} = 100 \text{ MHz}$) [cite: 1104]
* **Design Tools:** AMD Xilinx Vivado (2025.1) 
* **Simulation Environment:** ModelSim / Vivado Behavioral Simulator 
* **Hardware Diagnostics:** Digital Oscilloscope (Rohde & Schwarz RTB2004) [cite: 656, 670]

---

## 📐 System Specifications & Calculations
### Sampling Frequency Optimization
Due to the system clock frequency ($f_{SYSCLK} = 100 \text{ MHz}$) not dividing evenly into an exact audio industry-standard $48 \text{ kHz}$ clock cycle by powers of two, an optimal target ratio $f_{CLK}/f_{SYSCLK} = 8$ was established[cite: 1104, 1106].
* **Calculated Sampling Frequency ($\hat{f}_{LRC}$):** $48.828125 \text{ kHz}$ [cite: 1106]
* **Relative Deviation Error ($\epsilon_{rel}$):** $-1.73\%$ [cite: 1106]

---

## 💻 VHDL Architecture Implementation
[cite_start]The parallel-to-serial converter uses a single internal shift register matrix to systematically stream interleaved multi-channel data[cite: 1110, 1124]. [cite_start]The design maps four main control behaviors inside a combinational/sequential routing block[cite: 1120, 1121]:

1.  **Synchronous System Reset:** Initializes the register structure safely to a baseline state `0x0000` on active-low assertion[cite: 1121, 1130].
2. **Left Channel Priority Loading:** Captures incoming 16-bit left channel parallel audio word data into the shift matrix when `SAVE_R_LOAD_L` asserts high[cite: 1121, 1124].
3. ]**Right Channel Loading:** Captures incoming 16-bit right channel parallel audio word data when `SAVE_L_LOAD_R` asserts high[cite: 1121, 1124].
4. **MSB-First Bit Serial Shifting:** Shifts internal bits leftward progressively on active `SHIFT_OUT` intervals, driving serialized output data line-by-line (`SER_OUT <= SHIFT_REG(W-1)`)[cite: 1121, 1124].

---

## 🧪 Simulation & Test Cases
Design compliance was thoroughly verified using functional test scripts across three critical milestones[cite: 1111]:

* **Test Case 1: Power-On Reset & Boot-up State Verification**
    * Assures the internal register defaults safely to zero and stays stable while the system undergoes hardware reset states[cite: 1124, 1129].
* **Test Case 2: Priority Parallel Bus Loading**
    * Asserts deterministic hex audio sample arrays (e.g., `0xFE00`) across parallel structures and verifies accurate single-cycle latches into internal components[cite: 1124, 1131].
* **Test Case 3: MSB-First Output Serialization**
    * Tracks long-duration timing intervals over $100 \ \mu\text{s}$ to confirm reliable bit-stream shifting synchronous with the Bit Clock (BCK)[cite: 1124].

---

## 📊 Empirical Laboratory Diagnostics
Following successful hardware validation on the Modsys development kit, diagnostic parameters were measured under specific signaling test workloads[cite: 1104]:

### 1. High-Frequency System Loop Back Characteristics
* Tested inputs with $1 \text{ V}$ peak-to-peak sinusoidal traces[cite: 653].
* **Observations:** The data pipeline tracks signals with phase shifts due to pipeline delay buffers[cite: 662]. At frequencies ascending past $\approx 24 \text{ kHz}$, structural low-pass anti-aliasing constraints inside the hardware codec prompt signal attenuation[cite: 663].

### 2. Time-Delay Frame Mapping
* **Digital Loop Mapping:** Monitored parallel streaming paths mapping serial inputs `DIN` alongside serialized outputs `DOUT`[cite: 665]. Results validated that `DOUT` operates as a time-shifted structural mirror of `DIN` across sequential Left-Right Clock (LRC) frame boundaries[cite: 669, 672].
* [cite_start]**Digital Precision Readouts Example:** * *Left Channel Loop Array:* `0x2720` (Decimal: `10,016`) [cite: 669]
    * [cite_start]*Right Channel Loop Array:* `0xD48D` (Decimal: `-54,413` signed conversion format) [cite: 669]

### 3. Critical Timing Metrics (Datasheet Compliance)
To ensure reliable, glitch-free serial communication, physical timing intervals were verified against the audio codec hardware datasheet parameters[cite: 673, 674]:
* **Bit Clock (BCK) Cycle Time ($t_{BCY}$):** Measured at **$635\text{ ns}$** (Spec requirement: $\ge 300\text{ ns}$ minimum)[cite: 674, 679].
***Data Output Delay ($t_{BDO}$):** Measured at **$24\text{ ns}$** from falling edge (Spec requirement: $\le 40\text{ ns}$ maximum tolerance)[cite: 674, 679].
* **Status:** **Pass**. All parameters comply with the physical operational envelopes[cite: 679].

---

## 🚀 How to Run the Project
1. Clone this repository to your local machine:
   ```bash
   git clone [https://github.com/YOUR_USERNAME/audio-codec-vhd-communication.git](https://github.com/YOUR_USERNAME/audio-codec-vhd-communication.git)
