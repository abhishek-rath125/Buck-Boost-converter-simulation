# Goal and shared topology

## The idea

Rather than building a separate buck converter and a separate boost converter, the goal was to show that **one power stage can do both**, just by changing the duty cycle of the PWM driving its switch. Two independent 555-timer channels, each driving an identical copy of the power stage, let both regimes run side-by-side in one simulation:

- Channel 1 (U1): duty cycle **below 50%** → buck-like (|Vout| < Vin)
- Channel 2 (U2): duty cycle **above 50%** → boost-like (|Vout| > Vin)

## How the duty cycle is set

Both 555s use the classic "diode-steered astable" trick instead of a plain RC astable, because a plain 555 astable can never get below ~50% duty cycle:

```mermaid
flowchart LR
    Vcc -->|R_pullup| N[DISCH node]
    N -->|D_charge + R_charge| Cap[Timing cap]
    Cap -->|D_discharge + R_discharge| N
    Cap --- GND
```

- **Charge phase** (555 output HIGH): current flows `Vcc → R_pullup → R_charge → D_charge → cap`.
- **Discharge phase** (555 output LOW): the cap dumps back through `D_discharge → R_discharge` into the DISCH pin, which the 555 pulls to ~0V internally.

Because the two paths use separate diodes, the charge and discharge resistors are independent, so:

```
Duty  ≈ (R_pullup + R_charge) / (R_pullup + R_charge + R_discharge)
Freq  ≈ 1 / [0.693 × C × (R_pullup + R_charge + R_discharge)]
```

Swap which resistor is bigger, and the duty cycle flips from "mostly buck" to "mostly boost" — that's the entire mechanism behind having two different-looking resistor pairs around the two 555s in every file in this repo.

## Why a plain buck-boost went wrong the first time (context)

An earlier iteration (not included in this repo — see the chat history this project came from) used a **single-inductor inverting buck-boost** stage: switch in series from Vin, inductor shunted to ground, diode in series to the output. That topology inverts polarity (output is always negative for a positive Vin) and is very sensitive to switching frequency — too low a frequency relative to the inductor value meant the "boost" channel needed a ~22A inductor ripple current that the 13µs off-time simply couldn't discharge, so the output collapsed to a few volts instead of boosting above Vin.

The fix that stuck was moving to a **two-inductor (SEPIC-style) stage** — non-inverting, same ideal gain formula (`Vout/Vin = D/(1-D)`), and, as it turns out, far more forgiving of the switching-frequency/inductor-value combination. That's the stage all three files in this repo actually use. The rest of this project's story (documented in the following files) is about getting the *supporting* details — switching frequency, duty-cycle aggressiveness, and 555/MOSFET supply voltage — right.
