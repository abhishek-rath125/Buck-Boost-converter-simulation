# `ltspice/draft1_555_vcc_isolation_test/Draft1.asc`

## What this file is

A **standalone single-555 testbench** — just one NE555, its diode-steered duty-cycle network (D1/D2, R2=3.6k, R3=11k), and the timing caps. No power stage, no MOSFET, nothing else. It exists to answer one narrow question in isolation, before wiring it into the full converter:

> *If the 555 is powered from its own 12V rail instead of the 24V power rail, does the duty-cycle-control network still behave the way it's supposed to?*

## Why isolate it

The 555's charge/discharge timing depends only on the resistor ratio and R×C — it does **not** depend on Vcc, because both switching thresholds (⅓ Vcc and ⅔ Vcc) scale with the supply and cancel out of the timing equations. That's the theory. Testing it standalone, with a dedicated `V1 = 12V` source and nothing else attached, confirms it in practice before spending time re-wiring the full two-channel converter:

- Duty cycle here: **(1k + 3.6k) / (1k + 3.6k + 11k) ≈ 29.5%**
- Switching frequency (with the 10nF cap used here): **~9.3 kHz**

## The `R4 = 10MΩ` on the output

There's a 10MΩ resistor in series between the raw `pwm` node and a separate `pwmout` flag. At 10MΩ this carries essentially no current — its job is to let the output waveform be probed/tapped without loading the 555's output stage, the SPICE equivalent of a high-impedance scope probe. It's not a component that belongs in the final gate-drive path (a 10MΩ series resistor would starve a real MOSFET gate of any useful drive current) — it's scaffolding for this specific test, dropped once the concept moved into the full circuit.

## Outcome

This confirmed the 12V-Vcc approach was safe to fold into the full converter without touching the duty-cycle math — which is exactly what `v3_final/buckboostv3.asc` does for both channels.
