# Single-Stage Buck / Boost Demonstrator (555-timer PWM)

**Author:** Abhishek — B.Tech, Electrical & Electronics Engineering, NIT Goa

## What this is

A LTspice project demonstrating **buck-mode (step-down) and boost-mode (step-up) behaviour from a single power-stage topology**, purely by changing the PWM duty cycle. Two identical 555-timer astable oscillators drive two copies of the power stage:

- One timer runs at duty cycle **< 50%** → the stage behaves like a buck converter (output magnitude below Vin).
- The other runs at duty cycle **> 50%** → the same stage behaves like a boost converter (output magnitude above Vin).

This repo isn't just the final schematic — it's the actual debugging trail: three real problems found along the way, and how each was diagnosed and fixed (or is still being fixed).

## Repo structure

```
ltspice/
├── v1_sepic_fast_switching/     Two-inductor (SEPIC-style) power stage, 555 caps
│                                shrunk 10x to raise switching frequency.
│                                Still shares the 24V rail between the 555s
│                                and the power MOSFETs.
├── draft1_555_vcc_isolation_test/
│                                Standalone single-555 testbench, run at 12V
│                                Vcc in isolation, before touching the full
│                                power stage — proof-of-concept for the fix
│                                that made it into v3.
└── v3_final/                    Full converter with a dedicated 12V rail for
                                 both 555s (separate from the 24V power rail),
                                 moderated duty cycles, coupling-cap damping
                                 resistors, and modeled output-cap ESR.
docs/
├── 01-topology-and-goal.md
├── 02-v1-sepic-fast-switching.md
├── 03-draft1-vcc-isolation-test.md
└── 04-v3-final-fixes.md
```

## The short version

| | v1 (this repo) | Draft1 | v3 (final) |
|---|---|---|---|
| Power stage | SEPIC-style, 2 inductors | *(none — timer only)* | SEPIC-style, 2 inductors |
| Timer caps → switching freq | 1nF → **~131 kHz** | 10nF → ~9.3kHz (isolated test) | 10nF → ~13.1kHz |
| Duty cycle (2 channels) | 27.3% / **81.8%** (extreme) | 29.5% (single channel) | 39.1% / 63.6% (moderate) |
| Inductor ripple current | ~0.5A / ~1.5A | n/a | ~0.7A / ~1.2A |
| 555 Vcc | Shared 24V rail ⚠️ | **Dedicated 12V** ✅ | **Dedicated 12V** ✅ |
| Coupling-cap damping | none | n/a | 22Ω (R11/R12) |
| Output-cap ESR modeled | no | n/a | yes (Rser=0.1Ω) |

**Bottom line:** v1 solved the ripple-current problem by raising switching frequency instead of moderating the duty cycle, which is arguably the more elegant fix — but it still runs both NE555s and the MOSFET gate drive off the full 24V rail, which is fine in simulation and not fine on a real breadboard (a real NE555 tops out around 16–18V Vcc). Draft1 is where that specific fix got proven out on its own before being folded into v3. v3 is the version that's actually safe to build, but it kept the original 10nF timing caps rather than adopting v1's frequency increase — see [`docs/04-v3-final-fixes.md`](docs/04-v3-final-fixes.md) for why a "v4" combining both would be the logical next step.

## Simulation results (v3, final)

![v3 buck/boost simulation result](docs)

With `Vref` (V1, the main rail) at **24V**, the two channels of `v3_final/buckboostv3.asc` settle at:

| Channel | Duty cycle | Output | vs. Vin (24V) |
|---|---|---|---|
| Buck (U1) | 39.1% | **13.50 V** | below — step-down confirmed |
| Boost (U2) | 63.6% | **29.43 V** | above — step-up confirmed |

This is the result the whole project was aiming at: the same power-stage topology, one at a duty cycle below 50% and one above it, producing an output below and above Vin respectively. Note both outputs are read as negative in the trace (SEPIC-family stages carry a polarity dependent on where the diode and coupling cap are referenced) — the demonstration is about the *magnitude* relative to Vin, which is exactly what the duty cycle is controlling here.

## Opening these files

Free LTspice (Windows/macOS, and runs fine under Wine on Linux) — Analog Devices' download page has the current installer. Open any `.asc` file directly; LTspice will pull in the standard `NE555`, `nmos`, `schottky`, etc. symbols from its own library automatically.

## License

MIT — see [LICENSE](LICENSE).
