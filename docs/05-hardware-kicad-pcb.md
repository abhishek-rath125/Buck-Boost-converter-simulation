# Hardware Implementation & KiCad PCB Design

This document details the transition from the LTspice simulations (`ltspice/`) to a physical, manufactured hardware board (`hardware/`) designed in **KiCad 10**.

---

## 📸 Hardware Previews

### 1. Assembled Board 3D Render
![KiCad 3D PCB Render](hardware/buckboostv1_pcb_3d.png)

*Figure 1: 3D render of the assembled converter PCB showing the electrolytic capacitor bank (left), TO-220/TO-218 IRF3205 power MOSFET on top, NE555P timer DIP-8 IC, trimmer potentiometer RV1, steering diodes, and dual power inductors.*

---

### 2. KiCad Schematic Diagram
![KiCad Schematic](hardware/buckboostv1_schematic.png)

*Figure 2: Complete KiCad schematic (`buckboostv1.kicad_sch`) showing the NE555 PWM generator with potentiometer-controlled duty cycle steering, driving the IRF3205 MOSFET gate, into the dual-inductor SEPIC power stage.*

---

### 3. 2D PCB Layout & Copper Routing
![KiCad PCB Layout](hardware/buckboostv1_pcb_layout.png)

*Figure 3: 2-layer routed board layout (`buckboostv1.kicad_pcb`). Red indicates Front Copper (F.Cu) tracks and blue indicates Back Copper (B.Cu) ground return and power links.*

---

## 🛠️ Hardware Evolution: From LTspice to PCB

In the LTspice simulation suite (`ltspice/v3_final/`), two fixed resistor dividers set the duty cycle for the two channels ($39.1\%$ for buck mode and $63.6\%$ for boost mode). For the hardware demonstrator, we wanted a single physical board capable of **continuous real-time tuning**:

```mermaid
flowchart TD
    VCC["VCC (+12V / +24V Rail)"] --> R1["Pull-Up Resistor R1"]
    R1 --> DISCH["555 DISCH (Pin 7)"]
    DISCH --> RV1["Potentiometer RV1 (Wiper)"]
    RV1 -->|Charge Path| D1["Diode D1 (Forward)"]
    RV1 -->|Discharge Path| D2["Diode D2 (Reverse)"]
    D1 --> C5["Timing Cap C5"]
    D2 --> C5
    C5 --> TRIG["555 TRIG/THRES (Pins 2 & 6)"]
    TRIG --> 555["NE555 Timer"]
    555 -->|Pin 3 (PWM Out)| R2["Gate Resistor R2"]
    R2 --> Q1["Gate of IRF3205 MOSFET"]
```

### Why a Single Potentiometer RV1?
By connecting the two sides of potentiometer `RV1` to steering diodes `D1` and `D2` with their anodes/cathodes reversed:
- Rotating `RV1` towards `D1` increases charging resistance relative to discharge resistance $\rightarrow$ **Duty cycle $D > 50\%$ (Boost Mode)**.
- Rotating `RV1` towards `D2` decreases charging resistance relative to discharge resistance $\rightarrow$ **Duty cycle $D < 50\%$ (Buck Mode)**.
- When `RV1` is centered, $D \approx 50\%$ where $V_{out} \approx V_{in}$.

This makes the PCB an ideal educational demonstrator: turning a single knob immediately shifts the converter from step-down operation to step-up operation.

---

## ⚡ Power Stage Components & Selection

1. **Power MOSFET (`Q1` - IRF3205):**
   - $V_{DS} = 55\,\text{V}$, $I_D = 110\,\text{A}$, $R_{DS(on)} = 8.0\,\text{m}\Omega$.
   - Ultra-low on-resistance minimizes conduction losses ($I_{rms}^2 R_{DS(on)}$).
   - Packaged in TO-218 / TO-220 with direct mounting hole for a clip-on aluminum heat sink.

2. **Dual Inductors (`L3` & `L2`):**
   - Dual-inductor SEPIC configuration prevents DC path from input to output when switch is OFF.
   - Vertically mounted Fastron power chokes with heavy copper wire loops to handle high peak currents without saturation.

3. **Capacitor Array (`C2`, `C3`, `C4`, `C5`):**
   - High-grade radial electrolytic capacitors (10mm diameter, 5mm lead pitch).
   - `C3` serves as the AC coupling capacitor transferring stored energy from `L3` to the output stage.
   - `C4` provides low-ripple output voltage filtering.

4. **Freewheeling Diode (`D3` - BAT54C / Schottky):**
   - Ultra-fast switching response ensures minimal reverse recovery current spikes into `Q1` during turn-on.

---

## 🔍 PCB Layout & Routing Considerations

- **Gate Drive Trace Length:** Traces from Pin 3 of `U1` through resistor `R2` to the Gate of `Q1` are kept short to minimize parasitic inductance and prevent gate oscillation/ringing.
- **High-Current Loop Area:** The pulsating current loops (MOSFET drain $\leftrightarrow$ coupling cap $\leftrightarrow$ freewheeling diode) are routed with wide copper traces to reduce radiated EMI and inductive voltage spikes.
- **Thermal Relief:** Large heatsink clearance is provided at the board top so that `Q1` can be run under continuous heavy loads.
