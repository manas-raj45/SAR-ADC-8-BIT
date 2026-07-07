# 8-Bit SAR ADC Design in 90nm CMOS

## Overview
Welcome to my custom 8-bit Successive Approximation Register (SAR) ADC project! I designed this mixed-signal front-end entirely from the transistor level up using Cadence Virtuoso (gpdk090 process). 

My goal with this project wasn't just to get a working simulation, but to deeply understand and neutralize the real-world physical non-idealities of silicon—like charge injection, kickback noise, and common-mode collapse. The result is a highly efficient, low-power ADC optimized for edge devices and hardware accelerator interfaces.

## Key Performance Specs
* **Resolution:** 8 Bits (6.9 ENOB Nominal)
* **Sampling Rate:** 2 MS/s
* **Average Power Consumption:** 40.98 µW
* **Walden Figure of Merit (FoM):** 171.6 fJ/conv-step
* **Supply Voltage:** 1.8V

---

## The Math Behind the Metrics

### 1. Coherent Input Frequency Calculation
To get an accurate FFT spectrum and prevent spectral leakage during testing, I calculated a strict coherent input frequency using the formula: 
`fin = (M / N) * fs`

* **fs (Sampling Frequency):** 2 MHz
* **N (Total Samples):** 512
* **M (Prime Number of Cycles):** 11

**Calculation:**
`fin = (11 / 512) * 2,000,000 = 42,968.75 Hz`
By driving the analog input at exactly 42.97 kHz, the FFT captures perfect, non-overlapping bins.

### 2. Power & Figure of Merit (FoM)
The Walden FoM measures how much energy the ADC burns to resolve a single digital bit. A lower number means a more efficient design. 
`FoM = Average Power / (fs * 2^ENOB)`

* **Average Power:** 40.98 uW
* **fs:** 2 MHz
* **ENOB:** 6.9

**Calculation:**
`FoM = 40.98uW / (2,000,000 * 119.43)`
`FoM = 171.6 fJ/conv-step`
Breaking below the 200 fJ barrier proves the CDAC and comparator sizing is tight and highly power-efficient.

---

## Architecture & Design Choices
I built this architecture specifically to tackle real-world silicon limitations:
* **Bootstrapped Switch:** Used for the sample-and-hold circuit to maintain a constant Vgs. This eliminates signal-dependent ON-resistance and minimizes charge injection.
* **CDAC Array:** A single-ended Binary-Weighted Charge Redistribution DAC, physically balanced against a 2.56pF sampling capacitor to absorb kickback noise.
* **StrongARM Latch:** An NMOS-input comparator optimized for a 50ns clock constraint, featuring a custom-sized regenerative core to rapidly resolve microscopic 7mV LSBs.
* **Digital Control:** A custom Verilog-A Finite State Machine (FSM) driving the binary search algorithm.

---

## The Debugging Journey & Physical Solutions
This project was a massive exercise in transistor-level troubleshooting. Here is how I pushed the design from an initial 3.3 ENOB up to its theoretical limit.

### Overcoming Common-Mode Collapse
**The Problem Statement:** Initial simulations capped the ADC at ~6.1 ENOB. The digital tracking suffered severe distortion specifically at the lowest input voltages. I diagnosed that the NMOS input differential pair of the StrongARM latch was starving for current and physically shutting off when the analog signal dropped below the ~400mV threshold voltage.
**The Debugging Solution:** I applied a strict 1.0V DC offset with a 0.6V amplitude to the input signal. By shifting the swing from 0.4V to 1.6V, I guaranteed the comparator remained actively biased throughout the entire conversion phase. This instantly cured the SR Latch metastability and restored the digital staircase.

### Neutralizing Kickback Noise
**The Problem Statement:** Early iterations suffered from massive, destructive voltage spikes on the analog input pins the moment the StrongARM latch evaluated.
**The Debugging Solution:** I mathematically matched the total capacitive weight of the CDAC array to the C0 sample-and-hold capacitor. This provided a perfectly balanced, heavy electrical load to absorb the comparator kickback without distorting the sensitive sampled voltage.

### PVT Corner Robustness
To prove this design could survive commercial operating conditions, I subjected it to strict Process, Voltage, and Temperature (PVT) corner analysis:
* **Worst-Case Corner (85°C, 1.62V Supply):** Maintained **6.41 ENOB**. The ADC successfully survived high thermal noise and starved overdrive voltages without stalling.
* **Best-Case Corner (-40°C, 1.98V Supply):** Peaked at **7.1 ENOB**. 

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
