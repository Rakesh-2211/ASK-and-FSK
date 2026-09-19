# ASK and BFSK Digital Communication Simulation Using Python

## 📌 Overview

This project demonstrates the simulation and analysis of **Amplitude Shift Keying (ASK)** and **Binary Frequency Shift Keying (BFSK)** digital communication systems using Python.

The simulation covers the complete communication process:

**Binary Data → Modulation → AWGN Channel → Detection → BER Analysis → Visualization**

The project also demonstrates both **coherent and noncoherent BFSK detection** and validates the **orthogonality of BFSK basis signals** mathematically.

---

## 🎯 Objectives

The main objectives of this experiment are:

* Generate a random binary data sequence.
* Implement ASK modulation.
* Implement BFSK modulation.
* Add Additive White Gaussian Noise (AWGN).
* Implement coherent BFSK detection using correlation.
* Implement noncoherent BFSK detection using I/Q energy detection.
* Verify BFSK signal orthogonality.
* Calculate BER for different Eb/N0 values.
* Analyze the frequency spectrum using Welch's method.
* Visualize transmitted waveforms and receiver decision statistics.

---

## 🛠️ Technologies and Libraries

### Programming Language

* **Python 3.x**

### Python Libraries

| Library      | Purpose                                                        |
| ------------ | -------------------------------------------------------------- |
| `NumPy`      | Numerical calculations, arrays, random bits, signal generation |
| `Matplotlib` | Plotting and visualization                                     |
| `SciPy`      | Power Spectral Density estimation using Welch's method         |

### Required Imports

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.signal import welch
```

---

## 📐 System Parameters

The main simulation parameters are:

| Parameter                |   Value | Description                          |
| ------------------------ | ------: | ------------------------------------ |
| Number of bits           |  10,000 | Number of random binary bits         |
| Bit duration, `Tb`       |     1 s | Duration of each bit                 |
| Sampling frequency, `fs` |  100 Hz | Samples per second                   |
| ASK carrier              |    5 Hz | ASK carrier frequency                |
| BFSK frequency for `0`   |    5 Hz | First BFSK tone                      |
| BFSK frequency for `1`   |   10 Hz | Second BFSK tone                     |
| Visualization SNR        |    8 dB | SNR used for noisy waveform analysis |
| Eb/N0 range              | 0–10 dB | BER simulation range                 |
| Eb/N0 step               |    2 dB | Simulation increment                 |

---

# 🔄 System Block Diagram

```text
                 TRANSMITTER
                     │
                     ▼
              Random Binary Data
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
       ASK Modulation       BFSK Modulation
          │                     │
          └──────────┬──────────┘
                     │
                     ▼
                AWGN Channel
                     │
                     ▼
                  RECEIVER
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
     Coherent BFSK       Noncoherent BFSK
       Detector             Detector
          │                     │
          └──────────┬──────────┘
                     ▼
                 Bit Decisions
                     │
                     ▼
                BER Analysis
```

---

# 1️⃣ Binary Data Generation

A random binary sequence is generated using:

```python
bits = np.random.randint(0, 2, N_bits)
```

This generates a sequence containing zeros and ones.

Since the simulation uses 10,000 bits, the resulting sequence contains 10,000 randomly generated binary symbols.

The bits are then upsampled:

```python
bits_upsampled = np.repeat(bits, fs)
```

Since the sampling frequency is 100 Hz and the bit duration is 1 second, every bit is represented by 100 samples.

---

# 2️⃣ ASK Modulation

ASK stands for **Amplitude Shift Keying**.

In this simulation, a 5 Hz cosine carrier is used:

```python
carrier_ask = np.cos(2 * np.pi * fc_ask * t)
```

The ASK signal is generated as:

```python
ask_signal = bits_upsampled * carrier_ask
```

Therefore:

* Bit `1` → Carrier is transmitted.
* Bit `0` → Carrier amplitude becomes zero.

Conceptually:

```text
Bit 1 → cos(2πfc t)

Bit 0 → 0
```

This demonstrates binary ASK, where the information is represented through changes in carrier amplitude.

---

# 3️⃣ BFSK Modulation

BFSK stands for **Binary Frequency Shift Keying**.

Two carrier frequencies are used:

```python
f1_fsk = 5.0
f2_fsk = 10.0
```

The two carriers are:

```python
carrier_0 = np.cos(2 * np.pi * f1_fsk * t)
carrier_1 = np.cos(2 * np.pi * f2_fsk * t)
```

The BFSK signal is generated using:

```python
bfsk_signal = np.where(
    bits_upsampled == 0,
    carrier_0,
    carrier_1
)
```

Therefore:

```text
Bit 0 → 5 Hz carrier

Bit 1 → 10 Hz carrier
```

Unlike ASK, the amplitude remains approximately constant while the carrier frequency changes.

---

# 4️⃣ AWGN Channel

The transmitted signal is passed through an **Additive White Gaussian Noise (AWGN)** channel.

The function used is:

```python
def add_awgn(signal, snr_db):
    snr_linear = 10**(snr_db / 10.0)
    signal_power = np.mean(signal**2)
    noise_power = signal_power / snr_linear
    noise = np.sqrt(noise_power) * np.random.randn(len(signal))
    return signal + noise
```

The noise power is calculated based on the specified SNR.

For visualization, an SNR of:

```python
snr_test = 8
```

is used.

The noisy signals are then:

```python
ask_rx = add_awgn(ask_signal, snr_test)
bfsk_rx = add_awgn(bfsk_signal, snr_test)
```

---

# 5️⃣ Coherent BFSK Detection

The coherent detector uses **correlation** with the two reference carriers.

For each bit interval, the received signal is correlated with:

* 5 Hz reference carrier
* 10 Hz reference carrier

The correlation outputs are calculated using:

```python
corr_0[i] = np.sum(
    bfsk_rx[start:end] * carrier_0[start:end]
)

corr_1[i] = np.sum(
    bfsk_rx[start:end] * carrier_1[start:end]
)
```

The decision rule is:

```python
bfsk_coherent_bits = (corr_1 > corr_0).astype(int)
```

Therefore:

```text
corr_1 > corr_0 → Bit 1

corr_1 ≤ corr_0 → Bit 0
```

---

# 6️⃣ Noncoherent BFSK Detection

The noncoherent detector uses **I/Q components** to estimate the energy associated with each BFSK frequency.

For the 5 Hz frequency:

```python
I0 = np.sum(segment * np.cos(2 * np.pi * f1_fsk * t_bit))
Q0 = np.sum(segment * np.sin(2 * np.pi * f1_fsk * t_bit))

energy_0[i] = I0**2 + Q0**2
```

Similarly, the energy at 10 Hz is calculated.

```python
I1 = np.sum(segment * np.cos(2 * np.pi * f2_fsk * t_bit))
Q1 = np.sum(segment * np.sin(2 * np.pi * f2_fsk * t_bit))

energy_1[i] = I1**2 + Q1**2
```

The decision rule is:

```python
bfsk_noncoherent_bits = (energy_1 > energy_0).astype(int)
```

Therefore:

```text
Energy at 10 Hz > Energy at 5 Hz → Bit 1

Energy at 10 Hz ≤ Energy at 5 Hz → Bit 0
```

---

# 7️⃣ BFSK Orthogonality Validation

An important part of the experiment is verifying that the two BFSK basis signals are orthogonal over one bit interval.

The two basis functions are:

```python
phi1 = np.cos(2 * np.pi * f1_fsk * t_bit)

phi2 = np.cos(2 * np.pi * f2_fsk * t_bit)
```

Their inner product is calculated using numerical integration:

```python
inner_product = np.trapz(
    phi1 * phi2,
    dx=1/fs
)
```

If the result is approximately zero:

```text
Inner Product ≈ 0
```

the two basis signals are considered orthogonal over the selected interval.

The program prints the validation result:

```text
Mandatory Validation: Inner product of BFSK basis signals over Tb = ...
-> The basis signals are orthogonal.
```

---

# 8️⃣ BER Analysis

BER stands for **Bit Error Rate**.

The simulation evaluates coherent BFSK performance for several Eb/N0 values:

```python
ebno_dbs = np.arange(0, 12, 2)
```

This produces:

```text
0, 2, 4, 6, 8, 10 dB
```

For each Eb/N0 value:

1. AWGN is added.
2. The received signal is reshaped into individual bit intervals.
3. Coherent correlation is performed.
4. Received bits are estimated.
5. Estimated bits are compared with the original bits.
6. BER is calculated.

The BER calculation is:

```python
ber_coherent.append(
    np.mean(bits_coh != bits)
)
```

The general relationship expected is:

```text
Higher Eb/N0
      ↓
Lower relative noise
      ↓
Fewer bit errors
      ↓
Lower BER
```

---

# 📊 Generated Visualizations

The program produces five main plots.

## 1. Passband Waveforms

The first plot displays:

* ASK waveform
* BFSK waveform

for the first 10 bits.

It demonstrates the difference between amplitude-based and frequency-based modulation.

---

## 2. Power Spectral Density

The second plot shows the frequency-domain representation of the ASK and BFSK signals.

Welch's method is used:

```python
f_ask, Pxx_ask = welch(
    ask_signal,
    fs,
    nperseg=1024
)
```

and:

```python
f_fsk, Pxx_fsk = welch(
    bfsk_signal,
    fs,
    nperseg=1024
)
```

This allows the frequency content of the two modulation schemes to be analyzed.

---

## 3. Coherent Correlator Outputs

The third plot displays the correlation outputs:

```text
X-axis → Correlator 0 / 5 Hz

Y-axis → Correlator 1 / 10 Hz
```

The scatter plot shows how the received symbols are separated according to the transmitted bit.

---

## 4. Noncoherent Decision Statistic

The fourth plot shows the histogram of:

```text
Energy 1 − Energy 0
```

Two distributions are displayed:

* Transmitted bit 0
* Transmitted bit 1

The histogram illustrates the effect of noise on the noncoherent decision statistic.

---

## 5. BER Curve

The final graph shows:

```text
X-axis → Eb/N0 (dB)

Y-axis → BER
```

The BER axis uses a logarithmic scale.

The simulation demonstrates the relationship between signal-to-noise conditions and the probability of bit detection errors.

---

# 📁 Project Structure

A simple project structure can be:

```text
ASK-BFSK-Simulation/
│
├── ask_bfsk_simulation.py
│
├── README.md
│
├── requirements.txt
│
└── results/
    ├── passband_waveforms.png
    ├── power_spectral_density.png
    ├── correlator_outputs.png
    ├── decision_histogram.png
    └── ber_curve.png
```

---

# ⚙️ Installation

## 1. Install Python

Install **Python 3.x** on your computer.

## 2. Install Required Libraries

Open Command Prompt or Terminal and run:

```bash
pip install numpy matplotlib scipy
```

Alternatively, create a `requirements.txt` file:

```text
numpy
matplotlib
scipy
```

Then install using:

```bash
pip install -r requirements.txt
```

---

# ▶️ How to Run

Clone or download this repository.

Open the project directory in your terminal:

```bash
cd ASK-BFSK-Simulation
```

Run the Python program:

```bash
python ask_bfsk_simulation.py
```

The program will:

1. Generate random binary data.
2. Generate ASK and BFSK signals.
3. Add AWGN.
4. Perform BFSK detection.
5. Validate orthogonality.
6. Calculate BER.
7. Display the simulation graphs.

---

# 📚 Concepts Demonstrated

This project covers several important **Digital Communication** concepts:

* Binary data generation
* Sampling
* ASK modulation
* BFSK modulation
* Passband signaling
* AWGN channel
* Coherent detection
* Noncoherent detection
* Correlator receiver
* I/Q detection
* Signal energy
* Orthogonal basis functions
* Inner product
* Power Spectral Density
* Welch's method
* Eb/N0
* Bit Error Rate
* Digital communication system analysis

---

# ⚠️ Important Note

The variable used for the BER sweep is named `ebno_dbs`, but the `add_awgn()` function internally interprets its input as an **SNR value** and derives noise power from the total signal power.

Therefore, the BER section is best described as a **simulation parameter sweep labeled Eb/N0**, rather than a rigorously energy-normalized theoretical Eb/N0 simulation.

For a more rigorous communication-theory implementation, the signal energy per bit and noise spectral density can be explicitly normalized.

---

# 🚀 Possible Improvements

This project can be extended by adding:

* Theoretical BFSK BER curve.
* Theoretical ASK BER curve.
* Simulated ASK BER.
* Noncoherent BFSK BER curve.
* Comparison between coherent and noncoherent BFSK.
* Multiple SNR/Eb/N0 ranges.
* Larger numbers of transmitted bits.
* Eye diagrams.
* Constellation-style visualizations.
* Frequency-domain comparison for different tone spacings.
* GUI-based modulation simulator.
* Real-time signal visualization.
* Comparison of ASK, FSK, PSK, and QAM.
* Monte Carlo performance analysis.

---

# 🎓 Educational Purpose

This project is intended for **learning and academic experimentation in Digital Communication and Signal Processing**.

It provides a practical way to understand how mathematical concepts such as modulation, orthogonality, correlation, noise, spectral analysis, and BER are implemented using Python.

---

## 👨‍💻 Author

**Rakesh Karmakar**

Electronics and Communication Engineering

---

## ⭐ If You Found This Project Useful

If this project helped you understand ASK, BFSK, AWGN, coherent detection, noncoherent detection, or BER analysis, consider giving the repository a ⭐.

More communication-system simulations and ECE projects can be adde
