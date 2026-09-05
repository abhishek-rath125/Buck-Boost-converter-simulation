# Changelog

All notable changes to this project, in the order they actually happened.

## [v1] — `ltspice/v1_sepic_fast_switching/`
### Changed
- Power stage rebuilt as a two-inductor, SEPIC-style topology (non-inverting), replacing an earlier single-inductor inverting buck-boost stage.
- Timer timing capacitors (C1, C4) reduced 10x (10nF → 1nF) to raise switching frequency (~13kHz → ~131kHz) and cut inductor ripple current.
- Small coupling-cap damping resistors added (R11 = R12 = 5Ω).
### Known issues
- Both NE555s and MOSFET gate drive still share the 24V power rail — exceeds a real NE555's rated Vcc and approaches the IRFZ44N's Vgs max.
- Duty-cycle resistor ratios still at their original, extreme split (2k/8k and 8k/2k).

## [Draft1] — `ltspice/draft1_555_vcc_isolation_test/`
### Added
- Standalone single-555 testbench, powered from a dedicated 12V rail, isolated from the power stage entirely.
### Verified
- 555 diode-steered duty-cycle timing is unaffected by the Vcc change (confirmed: duty cycle depends only on R-ratio and RC, not supply voltage).

## [v3] — `ltspice/v3_final/` (final)
### Added
- Dedicated 12V rail (`V3`), feeding only the two NE555s' Vcc pins — repurposed from a previously orphaned, unused −24V source.
- Coupling-cap damping resistors increased to 22Ω (R11/R12).
- Output-capacitor ESR modeled (Rser = 0.1Ω on C6/C8).
### Changed
- Duty-cycle resistor ratios moderated (2k/8k → 3.3k/6.7k; 8k/2k → 6k/4k), reducing the required boost gain from an ideal 4.5x down to ~1.75x.
- Inductors increased (100µH/220µH → 1mH/10mH) to keep ripple current low at the original (slower, ~13kHz) switching frequency.
### Known limitation
- Did not carry over v1's faster switching (timer caps left at 10nF) — see `docs/04-v3-final-fixes.md` for the resulting "v4" recommendation.
