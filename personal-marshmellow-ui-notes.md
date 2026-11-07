---
summary: Personal pointer and scroll settings saved on the trackball through the Marshmellow UI shell, with the reasons behind them.
read_when:
  - restoring or preserving personal pointer acceleration settings
  - tuning drag-scroll or twist-scroll feel, or restoring scroll settings
---

# Personal Marshmellow UI Notes

Settings live in the trackball's own memory, not in this repo. Flashing firmware does not restore them; send the commands in the Marshmellow UI terminal.

## Pointer

No acceleration curve, by choice (David, 2026-11-07): the cursor moves at a fixed 25% sensitivity. Do not load a pointer curve.

## Scroll (2026-11-07, tuning in progress)

Used on macOS in OS slot 0 (Windows/Linux mode), with SteerMouse wheel mode "Linear" for the Endgame (macOS scroll acceleration off), so all scroll acceleration comes from these curves.

- `rtcfg set bst/twist/s0_div 1` and `rtcfg set bst/dragscroll/s0_div 1` (no divide before the curve)
- `curve set scroll 0 0 100 9 10 9 10 9 100 9 6000 40 2000 9 4000 40` (twist: 0.09x up to step ~8, smooth rise to 0.40x at step 60)
- `curve set dragscroll 0 0 100 6 10 6 10 6 100 6 6000 21 2000 6 4000 21` (drag: 0.06x, smooth rise to 0.21x at step 60)
- `p2sm sens twist set 150` (15%)
- Twist detection: `rtcfg set p2sm/twist_dy_mag_mul 3`, `rtcfg set p2sm/twist_dy_mag_div 2` (start needs twist > 1.5x roll, default 2x), `rtcfg set p2sm/twist_act_ms 15` (default 25), `rtcfg set p2sm/twist_deb 15` (default 35).

Why: the curve's speed input is the per-event scroll step after the OS-slot divider. With divider 10 that step was a whole number of 1 to 5, so any speed-up curve jumped (the "Aggressive" preset went from 0.29x at step 1 to 1.06x at step 4, about 14 times the output for a little more force). With divider 1 the step runs from 1 (slow) to about 76 (hardest flick) (measured with `curve monitor on --abs`), and the curve does the dividing with y below 1.0, so the speed-up is smooth. macOS ignores the HID high-resolution wheel multiplier: each unit is a full wheel notch. Set the slow speed with the curve's base y, and the flick distance with its end y.

Firmware: `twist-interference-thres = <0>` in `boards/arm/efogtech_trackball_0/pointer.dtsi` (2026-11-07). The default 200 dropped a whole twist step when its roll part was over 200 counts per ~9 ms step, so fast, big twists did not register.
