# `ltspice/v1_sepic_fast_switching/buckboostv1.asc`

## What changed here

- Power stage: SEPIC-style, two inductors per channel — `L1 (100µ) → switch → C3 (4.7µ coupling) → L2 (220µ, shunt to GND) → D4 (schottky) → C6 → R5`. Same for the second channel (L3/C7/D6/L4/C8/R10).
- Small damping resistors (**R11 = R12 = 5Ω**) at the coupling-cap junction.
- Timer caps **C1 = C4 = 1nF** — 10x smaller than a straightforward first pass would use.
- Duty-cycle resistors are the original, aggressive split: **R2=2k / R3=8k** (U1) and **R7=8k / R8=2k** (U2).
- Both 555s' Vcc, and both MOSFETs' gate drive, still come from the same **24V** rail as the power stage.

## Why the small timer caps matter

Shrinking C1/C4 from 10nF to 1nF doesn't touch the duty cycle (duty depends only on the resistor ratio), but it raises the switching frequency roughly 10x:

| Channel | Duty | Switching freq | Ton | Inductor ripple (ΔI = Vin·Ton/L) |
|---|---|---|---|---|
| U1 (buck-mode, D≈27%) | 27.3% | ~131 kHz | 2.08 µs | **~0.50 A** |
| U2 (boost-mode, D≈82%) | 81.8% | ~131 kHz | 6.24 µs | **~1.50 A** |

Compare that to the very first attempt at this project (a single-inductor inverting buck-boost at ~13 kHz with 68µH), where the boost channel needed ~22A of ripple current in the same 13µs off-time — an obviously unworkable combination that buried almost all the delivered power in parasitic resistance instead of the output. Raising the switching frequency by 10x here cuts the required ripple by roughly the same factor, and combined with a much larger inductor (100µH vs 68µH) the boost channel should now come reasonably close to its ideal SEPIC gain (`D/(1-D) ≈ 4.5x` at 82% duty).

## The problem that's still open in this file

Both NE555s and the gate-drive signal for M1/M2 are still riding the full **24V** rail:

- A real NE555's absolute maximum Vcc is typically **16–18V**. Running it at 24V works fine in a SPICE simulation (the model doesn't enforce datasheet limits) but would damage a real chip.
- The IRFZ44N's gate is rated **±20V Vgs absolute max**. A 555 output swinging close to 22–23V into the gate is uncomfortably close to that limit.

This is purely a "safe to actually build" issue, not a "does the simulation work" issue — which is exactly why it's easy to miss if you're only watching waveforms. See `03-draft1-vcc-isolation-test.md` for how this got fixed, and `v3_final/` for where the fix landed.
