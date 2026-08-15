# 5-Stage-CMOS-Ring-Oscillator-Cadence
Design and simulation of a 5-stage CMOS ring oscillator using Cadence Virtuoso, including schematic, transient analysis, frequency and propagation delay analysis.

# 5-Stage Ring Oscillator (Cadence Virtuoso / Spectre, gpdk090)

## Overview

This project implements and characterizes a **5-stage CMOS ring oscillator** designed and simulated in Cadence Virtuoso ADE using the **GPDK090 (90 nm)** process design kit. The circuit consists of five cascaded CMOS inverter stages connected in a feedback loop, with an additional variable load capacitor at each stage output to study the effect of load capacitance on oscillation frequency.

## Circuit Description

- **Topology:** 5 CMOS inverters (PM0–PM4, NM0–NM4) connected in a ring, with the output of the last stage fed back to the input of the first stage.
- **Supply:** `VDD = 1 V` (via ideal DC source `V0`, `vdc = 1`)
- **Transistor sizing (identical for all stages):**
  | Device | Type | W | L | Multiplier |
  |--------|------|-----|------|------|
  | PM0–PM4 | `gpdk090_pmos1v` | 240 nm | 100 nm | 1 |
  | NM0–NM4 | `gpdk090_nmos1v` | 120 nm | 100 nm | 1 |
- **Load capacitors:** Each inverter output (except node `out`, which is externally probed after stage 5) drives a capacitor `C0–C4` with value `c = c_var`, used to model interconnect/fan-out loading and to sweep the oscillation frequency.
- **Number of stages:** 5 (odd number of inversions ⇒ ensures sustained oscillation, as required for a ring oscillator).
- **Output node:** `out`, taken at the output of the fifth inverter stage.

An odd number of inverting stages in a closed loop provides a net phase shift of 180°, plus loop gain greater than unity at DC, satisfying the Barkhausen criterion and producing self-sustained oscillation without any external clock or excitation.

## Simulation Setup

- **Analysis type:** Transient (`tran`)
- **Time range:** 0 to 5 µs
- **Design variable:** `c_var` (load capacitance per stage), swept in later analysis
- **Base case:** `c_var = 1 pF`
- **Output expressions monitored:**
  - `frequency(VT("/out"))` – measured oscillation frequency
  - `out` – transient voltage waveform (all values saved)

## Results

### 1. Transient Output Waveform (c_var = 1 pF)

The output node `out` produces a clean, periodic square-like waveform swinging rail-to-rail between 0 V and 1 V, confirming stable self-sustained oscillation.

- **Measured frequency:** ≈ **8.97 MHz** (`frequency(VT("/out")) ≈ 8.971 MHz`)
- **Corresponding period:** T ≈ 1 / 8.97 MHz ≈ **111.5 ns**
- **Per-stage propagation delay:** For an N-stage ring oscillator, `f = 1 / (2 × N × t_p)`. With N = 5 and f ≈ 8.97 MHz:

  t_p ≈ 1 / (2 × 5 × 8.97 MHz) ≈ **11.15 ns per stage**

### 2. Parametric Sweep — Frequency vs. Load Capacitance

`c_var` was swept logarithmically from **1 pF to 100 pF** (points at 1.0, 1.668, 2.783, 4.642, 7.743, 12.92, 21.54, 35.94, 59.95, 100.0 pF), and the transient response / measured frequency were recorded for each case.

**Observations:**
- As `c_var` increases, the rise/fall time of each stage increases (visible as the transient edges become progressively slower/more sloped for larger capacitance traces).
- The oscillation frequency **decreases monotonically** with increasing load capacitance, following the expected inverse relationship (`f ∝ 1/C`) since stage delay `t_p` scales approximately linearly with the load capacitance being charged/discharged by the finite drive strength of each inverter.
- Frequency drops steeply for small capacitances (1 pF → ~9 MHz, down to ~3.3 MHz by 2.78 pF) and flattens out at higher capacitances (approaching ~0.3 MHz near 100 pF), consistent with the `1/C` curve shape.

| c_var | Approx. Frequency |
|-------|-------------------|
| 1.0 pF | ~8.97 MHz |
| 1.67 pF | ~5.4 MHz |
| 2.78 pF | ~3.3 MHz |
| 4.64 pF | ~2.0 MHz |
| 7.74 pF | ~1.2 MHz |
| 12.9 pF | ~0.75 MHz |
| 21.5 pF | ~0.5 MHz |
| 35.9 pF | ~0.35 MHz |
| 59.95 pF | ~0.3 MHz |
| 100 pF | ~0.28 MHz |

*(Values read approximately from the frequency-vs-c_var plot; refer to simulation raw data for exact numbers.)*

## Key Takeaways

1. The 5-stage inverter ring, with an odd number of stages, successfully self-oscillates with no external excitation — validating the Barkhausen stability/oscillation criterion.
2. The base-case oscillation frequency at `c_var = 1 pF` is approximately **8.97 MHz**, giving a per-stage delay of about **11 ns**.
3. Oscillation frequency is inversely related to per-stage load capacitance, making the ring oscillator's frequency tunable by varying `c_var` — a useful property for **voltage/capacitance-controlled oscillator (VCO/CCO)** applications, delay-line calibration, and process-monitoring test structures.
4. As expected, the frequency-vs-capacitance curve is hyperbolic (`f ∝ 1/C`), with diminishing sensitivity at higher capacitance values.

## Files in this Repository

- `ring_osc_schematic.png` – Cadence schematic of the 5-stage ring oscillator
- `ring_osc_graph.png` – Transient output waveform at `c_var = 1 pF`
- `ring_osc_frequency.png` – ADE simulation setup showing measured frequency output
- `ring_osc_graph_parametric_.png` – Transient waveforms overlaid for the parametric `c_var` sweep
- `ring_osc_frequency_parametric_.png` – Frequency vs. `c_var` plot from the parametric sweep

## Tools Used

- Cadence Virtuoso ADE (Analog Design Environment)
- Spectre simulator
- GPDK090 (90 nm) generic process design kit
