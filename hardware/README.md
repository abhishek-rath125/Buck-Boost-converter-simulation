# Hardware & PCB Implementation (KiCad 10)

This directory contains the complete hardware design, schematic, 2-layer PCB layout, Bill of Materials (BOM), and manufacturing Gerber files for the **Single-Stage Buck / Boost Demonstrator**.

---

## 📐 Circuit Architecture

The hardware translates the dual-mode SEPIC / buck-boost topology into a physical board powered by a single supply and tuned in real time via an on-board trimmer potentiometer:

1. **Continuous Duty-Cycle PWM Generator (NE555P):**
   - Configured as an astable multivibrator with diode steering (`D1`, `D2`).
   - Single trimmer potentiometer (`RV1`) adjusts the ratio between charge resistance and discharge resistance without changing total cycle period drastically.
   - Turning `RV1` allows seamless on-the-fly transition between **Buck mode ($D < 50\%$)** and **Boost mode ($D > 50\%$)**.
   - Output pin 3 drives the gate of power MOSFET `Q1` through gate damping resistor `R2`.

2. **SEPIC Power Stage:**
   - **Switch (`Q1`):** IRF3205 N-Channel MOSFET in a TO-220/TO-218 vertical package mounted with heatsink clearance for thermal dissipation.
   - **Inductors (`L3`, `L2`):** Dual high-current inductors (input choke and output inductor) supporting energy storage in both CCM and DCM modes.
   - **Coupling Capacitor (`C3`):** High-capacitance electrolytic block blocking DC while passing switching ripple.
   - **Freewheeling Diode (`D3`):** Fast switching Schottky diode (`BAT54C` / TO-126 package) for low forward drop and minimal reverse recovery losses.
   - **Output Filter (`C4`, `R4`):** Low-ESR electrolytic capacitor bank with dummy load resistor `R4`.

---

## 📦 Bill of Materials (BOM)

| Reference | Value / Part | Package / Footprint | Description / Purpose |
|---|---|---|---|
| **U1** | NE555P | DIP-8 (`W7.62mm`) | Precision Timer IC (PWM oscillator) |
| **Q1** | IRF3205 | TO-218-3 / TO-220 Vertical | 55V, 110A, 8mΩ N-Channel Power MOSFET |
| **RV1** | R_Potentiometer | ACP CA9-V10 Vertical | Trimmer potentiometer for duty cycle tuning |
| **D1, D2** | Fast Switching Diode | DO-35 (SOD27) / Axial | Steering diodes for independent charge/discharge |
| **D3** | BAT54C / Schottky | TO-126-3 Vertical | High-speed freewheeling rectifier |
| **L2, L3** | Inductors (Power Chokes) | Axial Fastron MECC (`P7.62mm`) | Dual energy storage inductors |
| **C2** | Filter Cap | Radial D10mm, P5.0mm | 555 Pin 5 (CONT) noise decoupling |
| **C3** | Coupling Cap | Radial D10mm, P5.0mm | AC energy transfer capacitor |
| **C4** | Output Cap | Radial D10mm, P5.0mm | DC output voltage smoothing filter |
| **C5** | Timing Cap | Radial D10mm, P5.0mm | 555 astable timing capacitor |
| **R1** | Pull-up Resistor | Axial DIN0204 (`P5.08mm`) | Discharge pullup from Vcc to Pin 7 |
| **R2** | Gate Resistor | Axial DIN0204 (`P5.08mm`) | MOSFET gate damping resistor (suppresses ringing) |
| **R3** | Damping Resistor | Axial DIN0204 (`P5.08mm`) | Coupling-loop damping resistor |
| **R4** | Load Resistor | Axial DIN0204 (`P5.08mm`) | Output test load |
| **J1** | Conn_01x02_Pin | JWT A3963 2-pin (`P3.96mm`) | DC power input connector |

---

## 🗂️ File Directory

- [`buckboostv1.kicad_pro`](buckboostv1.kicad_pro) — KiCad 10 project definition file.
- [`buckboostv1.kicad_sch`](buckboostv1.kicad_sch) — Complete hierarchical schematic with netlabels and title block.
- [`buckboostv1.kicad_pcb`](buckboostv1.kicad_pcb) — 2-layer routed PCB layout with power planes.
- [`schematic.pdf`](schematic.pdf) — High-resolution vector PDF export of the schematic.
- [`buckboostv1.svg`](buckboostv1.svg) — Scalable vector graphics export of the schematic.
- [`bom.csv`](bom.csv) — Comma-separated Bill of Materials export.
- [`gerbers/`](gerbers/) — Production-ready Gerber and Excellon drill files.

---

## 🏭 PCB Fabrication Details

The files in [`gerbers/`](gerbers/) are directly compatible with standard PCB fabricators (JLCPCB, PCBWay, OSH Park):

| Extension | Layer Description | Fabricator Layer Name |
|---|---|---|
| `.gtl` | Front / Top Copper | Top Layer |
| `.gbl` | Back / Bottom Copper | Bottom Layer |
| `.gts` | Front / Top Solder Mask | Top Solder Mask |
| `.gbs` | Back / Bottom Solder Mask | Bottom Solder Mask |
| `.gto` | Front / Top Silkscreen | Top Overlay / Silkscreen |
| `.gbo` | Back / Bottom Silkscreen | Bottom Overlay / Silkscreen |
| `.gm1` | Board Outline / Edge Cuts | Mechanical 1 / Edge Cuts |
| `.drl` | Excellon NC Drill File | Plated / Non-Plated Through Holes |

### Recommended Fab Specifications:
- **Board Thickness:** 1.6 mm
- **Copper Weight:** 1 oz (35 µm) or 2 oz (for high current demonstrator runs)
- **Minimum Trace Width / Spacing:** 0.25 mm / 0.25 mm
- **Surface Finish:** HASL (with lead) or ENIG
- **Solder Mask Color:** Green / Matte Black
