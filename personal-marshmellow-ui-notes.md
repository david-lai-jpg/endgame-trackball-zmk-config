---
summary: Personal pointer and scroll settings saved on the trackball through the Marshmellow UI shell, with the reasons behind them.
read_when:
  - restoring or preserving personal pointer acceleration settings
  - tuning drag-scroll or twist-scroll feel, or restoring scroll settings
  - changing the HID scroll setup or using the trackball on Windows
---

# Personal Marshmellow UI Notes

Settings live in the trackball's own memory, not in this repo. Flashing firmware keeps them (the firmware ends before the settings area at 0x6c000), but a settings reset loses them; send the commands below in the Marshmellow UI terminal to restore.

## Pointer

No acceleration curve, by choice (David, 2026-11-07): the cursor moves at a fixed 25% sensitivity. Do not load a pointer curve.

## Scroll (2026-11-07)

Used mostly on macOS (with SteerMouse wheel mode "Linear" for the Endgame), sometimes on Windows. Default OS mode is macOS: `bistable set 1`, `rtcfg set bst/default 1`. Switch the trackball to Windows mode (slot 0) on Windows: hold the Device-layer button (pos 7) and press pos 4 (OS toggle, flashes cyan), or `bistable set 0`. The chosen mode is saved and survives restarts.

- macOS mode: `rtcfg set bst/twist/s1_div 1`, `rtcfg set bst/dragscroll/s1_div 1` (no divide before the curve)
- Windows mode: `rtcfg set bst/twist/s0_div 3`, `rtcfg set bst/dragscroll/s0_div 3` (Windows scrolls 3 lines per unit, macOS 1)
- `curve set scroll 0 0 100 9 10 9 10 9 100 9 6000 40 2000 9 4000 40` (twist: 0.09x up to step ~8, smooth rise to 0.40x at step 60)
- `curve set dragscroll 0 0 100 6 10 6 10 6 100 6 6000 21 2000 6 4000 21` (drag: 0.06x, smooth rise to 0.21x at step 60)
- `p2sm sens twist set 150` (15%)
- Twist detection: `rtcfg set p2sm/twist_dy_mag_mul 3`, `rtcfg set p2sm/twist_dy_mag_div 2` (start needs twist > 1.5x roll, default 2x), `rtcfg set p2sm/twist_act_ms 8` (default 25), `rtcfg set p2sm/twist_deb 15` (default 35), `rtcfg set p2sm/twist_hyst_mul 1` + `rtcfg set p2sm/twist_hyst_div 1` (continuation needs twist > roll, default 7/6), `rtcfg set p2sm/twist_ttl 120` (default 65). Tighten these first if twist scroll starts by accident.

Why the curves look like this: the curve's speed input is the per-event scroll step after the OS-slot divider. With divider 10 that step was a whole number of 1 to 5, so any speed-up curve jumped (the "Aggressive" preset went from 0.29x at step 1 to 1.06x at step 4, about 14 times the output for a little more force). With divider 1 the step runs from 1 (slow) to about 110 (hardest twist), measured with `curve monitor on --abs`, and the curve does the dividing with y below 1.0, so the speed-up is smooth. Set the slow speed with the curve's base y and the flick distance with its end y. The Windows divider sits before the curve, so on Windows the speed-up starts at 3 times the speed and flicks go about half as far as on macOS (accepted, 2026-11-07).

## Firmware scroll settings (2026-11-07)

- `CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=n` in `config/efogtech_trackball_0.conf`. Measured with a macOS scroll-event tap: while the HID Resolution Multiplier is declared, macOS turns every wheel report into one 1-line step whatever its value, so fast scrolls and flicks were capped. Without it macOS keeps the value (events up to 36 lines seen). On macOS 1 unit = 1 line = 10 px. Cost: Windows loses sub-notch (high-resolution) scrolling; 1 unit = 1 notch there.
- `sync-scroll-report-ms` stays `<8>` in `boards/arm/efogtech_trackball_0/pointer.dtsi`. A 4 ms build (2026-11-07) made cursor, drag and twist feel floaty and jumpy; the user kept the 8 ms build.
- `twist-interference-thres = <0>` in the same file. The default 200 drops a twist step whose roll part is over 200 counts. It was not the cause of dropped fast twists (the HID multiplier was); it stays because the relative twist-vs-roll check still guards.
- With SteerMouse off, macOS's own scroll acceleration stacks on the curves and small twists jump; keep SteerMouse "Linear" for the Endgame.
