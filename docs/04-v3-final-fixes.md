# `ltspice/v3_final/buckboostv3.asc`

## What's different from v1

| | v1 | v3 |
|---|---|---|
| 555 Vcc | Shared 24V power rail | **Dedicated 12V rail (`V3`)**, feeding only U1 and U2 |
| MOSFET gate drive | ~22–23V swing | ~10–11V swing — plenty to fully enhance an IRFZ44N (datasheet RDS(on) is spec'd at Vgs=10V), and clear of its ±20V absolute max |
| Duty cycle (U1 / U2) | 27.3% / 81.8% (extreme) | 39.1% / 63.6% (moderate) |
| Timer caps | 1nF (fast, ~131kHz) | 10nF (slower, ~13.1kHz) — **not** carried over from v1 |
| Inductors | L1/L3 = 100µH, L2/L4 = 220µH | L1/L3 = **1mH**, L2/L4 = **10mH** — bigger, compensating for the lower switching frequency |
| Coupling-cap damping | R11/R12 = 5Ω | R11/R12 = **22Ω** |
| Output-cap ESR | not modeled | modeled, **Rser = 0.1Ω** |

## The Vcc fix, mechanically

The old, unused −24V source (previously wired to a dead net called `Vref`) got repurposed: its value changed to **+12V** and its net renamed to `V12`, which now feeds only the Vcc pull-up resistors of U1 and U2 (R1, R6). The 24V rail (`V1`) still feeds everything else — L1/L3, M1/M2, the rest of the power stage — completely untouched. Since 555 timing is Vcc-independent, this is a change with zero effect on duty cycle or frequency, and a real effect on whether the chips survive.

## Why the ripple current is still fine, despite the slower switching

v3 didn't adopt v1's 10x-faster timer caps, but it compensates with much bigger inductors:

| Channel | Duty | Ton | Inductor ripple (ΔI = Vin·Ton/L) |
|---|---|---|---|
| U1 (39.1%) | 29.80 µs | 1mH | **~0.72 A** |
| U2 (63.6%) | 48.51 µs | 1mH | **~1.16 A** |

Comparable ripple to v1's, arrived at through the opposite lever — bigger L instead of higher frequency.

## What's *not* fixed yet (a "v4" candidate)

v1 and v3 each solved a different half of the original problem, and neither file has both fixes at once:

- v1: fast switching + aggressive duty cycle, but still on the shared 24V rail (correct in sim, not breadboard-safe).
- v3: safe supply split + moderate duty, but at the original slower switching frequency (works, but a less dramatic demonstration of the boost gain, since 63.6% duty gives an ideal gain of only `0.636/0.364 ≈ 1.75x`, versus v1's 82%-duty ideal gain of `4.5x`).

The natural next step is a v4 that takes v3's dedicated-12V-rail + damping + ESR modeling and combines it with v1's smaller timer caps (faster switching) *while keeping v3's moderate duty cycle* — getting the more dramatic boost ratio without reintroducing the extreme-duty ripple problem from the very first version of this project.

## Practical breadboard note

Even with the damping resistors, a real (non-PCB) build of this stage will have more parasitic inductance in the switch-node loop than the simulation accounts for — keep the MOSFET–diode–inductor loop wiring as short as possible, and bring the supply up through a current-limited bench PSU the first time.
