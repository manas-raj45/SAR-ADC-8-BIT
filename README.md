# 8-Bit SAR ADC Design in 90nm CMOS

## Overview
This repository contains the transistor-level design and verification of an 8-bit Successive Approximation Register (SAR) Analog-to-Digital Converter (ADC). Designed in Cadence Virtuoso using the gpdk090 (90nm) process, the project focuses on addressing non-ideal silicon effects such as charge injection, kickback noise, and common-mode variations to achieve a power-efficient mixed-signal front-end.

## Key Performance Specs
* **Resolution:** 8 Bits (6.9 ENOB Nominal)
* **Sampling Rate:** 2 MS/s
* **Average Power Consumption:** 40.98 µW
* **Walden Figure of Merit (FoM):** 171.6 fJ/conv-step
* **Supply Voltage:** 1.8V

---

## Design Calculations

### 1. Coherent Input Frequency
To ensure an accurate FFT spectrum and prevent spectral leakage, the coherent input frequency was calculated using:
`fin = (M / N) * fs`

* **fs (Sampling Frequency):** 2 MHz
* **N (Total Samples):** 512
* **M (Prime Number of Cycles):** 11

**Calculation:**
`fin = (11 / 512) * 2,000,000 = 42,968.75 Hz`
Driving the analog input at exactly 42.97 kHz ensures the FFT captures non-overlapping bins.

### 2. Power & Figure of Merit (FoM)
The Walden FoM measures the energy consumed per conversion step.
`FoM = Average Power / (fs * 2^ENOB)`

* **Average Power:** 40.98 µW
* **fs:** 2 MHz
* **ENOB:** 6.9

**Calculation:**
`FoM = 40.98µW / (2,000,000 * 119.43) = 171.6 fJ/conv-step`
A FoM of 171.6 fJ/conv-step demonstrates an efficient CDAC and comparator sizing strategy.

---

## Architecture Details
Key design implementations include:
* **Bootstrapped Switch:** Used in the sample-and-hold circuit to maintain a constant Vgs. This minimizes signal-dependent ON-resistance and charge injection.
* **CDAC Array:** A single-ended Binary-Weighted Charge Redistribution DAC, scaled and balanced against a 2.56pF sampling capacitor to absorb kickback noise.
* **StrongARM Latch:** An NMOS-input comparator optimized for a 50ns clock constraint, with a custom-sized regenerative core for accurate resolution of 7mV LSBs.
* **Digital Control:** A Verilog-A Finite State Machine (FSM) implementing the binary search algorithm.

---

## Design Challenges & Solutions

### Common-Mode Collapse
* **Issue:** Initial simulations yielded ~6.1 ENOB with distortion at lower input voltages. The NMOS input differential pair of the StrongARM latch entered the subthreshold region when the analog signal dropped below ~400mV.
* **Solution:** Applied a 1.0V DC offset with a 0.6V amplitude to the input signal (0.4V to 1.6V swing). This ensured the comparator remained actively biased throughout the conversion phase, preventing SR Latch metastability and restoring digital tracking.

### Kickback Noise Mitigation
* **Issue:** Evaluation of the StrongARM latch caused significant kickback noise on the analog input pins.
* **Solution:** Sized the total capacitive weight of the CDAC array to match the C0 sample-and-hold capacitor. This balanced the load, absorbing the comparator kickback without altering the sampled voltage.

### PVT Corner Analysis
The design was verified across Process, Voltage, and Temperature (PVT) corners to confirm robustness:
* **Worst-Case Corner (85°C, 1.62V Supply):** Maintained **6.41 ENOB**.
* **Best-Case Corner (-40°C, 1.98V Supply):** Achieved **7.1 ENOB**.

---

## Circuit Schematics & Layout

### Top-Level Mixed-Signal Testbench
<img width="821" height="673" alt="toptb" src="https://github.com/user-attachments/assets/484de561-64b9-4d95-be4f-be12e3f2c834" />

### Transistor-Level StrongARM Latch
<img width="1316" height="672" alt="strong arm final" src="https://github.com/user-attachments/assets/3a80bb1f-8c4d-4bb8-b0bc-44c5a392dce0" />

### Bootstrapped Switch with Charge Pump
<img width="1026" height="681" alt="boot switch" src="https://github.com/user-attachments/assets/6cde0c54-fe9d-47cf-8792-73a875a8c1bf" />

### Binary-Weighted CDAC Array
<img width="649" height="271" alt="cdac" src="https://github.com/user-attachments/assets/7f7a404e-625e-42eb-a8e8-104a0fef9fef" />

---

## Performance Proof

### Final FFT Spectrum (6.9 ENOB)
<img width="868" height="674" alt="spectrum" src="https://github.com/user-attachments/assets/2a9a217d-77ce-4efe-b040-5662384da678" />
<img width="846" height="334" alt="eob_final" src="https://github.com/user-attachments/assets/9de9f690-0bb1-49a9-b56d-6df97cc0410c" />
<img width="916" height="651" alt="fft_vout" src="https://github.com/user-attachments/assets/11217ec6-95dc-4761-8f2d-5f3431af133b" />

### Transient Waveform Tracking
<img width="1716" height="696" alt="vout vs v sample" src="https://github.com/user-attachments/assets/e962600d-b406-46fb-9fd7-53670d559014" />
<img width="1436" height="667" alt="vout vs input" src="https://github.com/user-attachments/assets/ab498930-1557-47cc-80cc-18232b1fb77e" />
<img width="1615" height="663" alt="output" src="https://github.com/user-attachments/assets/3db34eb0-94b0-4c57-81f8-41bac2a918a0" />

### PVT Corner Analysis
<img width="929" height="566" alt="pvt" src="https://github.com/user-attachments/assets/6f5f42c8-592d-4770-9f8a-4bc8cf973e06" />
<img width="1680" height="626" alt="pvt fft" src="https://github.com/user-attachments/assets/c3043235-78da-4474-ade4-094c5a36b9ce" />
