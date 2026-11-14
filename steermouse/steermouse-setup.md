---
summary: The SteerMouse setup on the Mac that makes the Endgame's left-right scroll follow the ball, with the steps to check it and put it back.
read_when:
  - left-right (horizontal) drag scroll is jumpy, too far, too slow or missing on macOS
  - SteerMouse was reinstalled, reset or updated, or the trackball is used on a new Mac
  - changing SteerMouse wheel settings for the Endgame
---

# SteerMouse setup for the Endgame (macOS)

This setup lives on the Mac, not in the trackball. The firmware and the saved
trackball settings have no part in it. It was found and measured on 2026-11-14
with SteerMouse 5.8.2; the full story is in
`personal-marshmellow-ui-notes.md`, section "SteerMouse left-right scroll".

## What it does

SteerMouse reads an unknown device's left-right scroll as a tilt switch (a wheel
that you push sideways). A switch is on or off, so SteerMouse drops the
trackball's real left-right amounts and sends its own fixed steps on a timer.
The result is a pan that runs too far at slow speed and jumps, worst on a design
board (Brilliant, Pencil).

The file `Endgame Trackball.xml` in this folder tells SteerMouse that the
Endgame's left-right scroll is a rolling wheel (`hScrollOp` = `Roll`). SteerMouse
then offers Roll Left / Roll Right, and with Linear it passes one step per
trackball report, the same rule as up-down.

## Signs that the setup is lost

- Left-right drag scroll jumps or runs far while up-down is normal.
- In SteerMouse, the Endgame's Wheel tab shows Tilt Left / Tilt Right and no
  Roll Left / Roll Right.

## Check

1. The file must exist and match the copy here:
   `diff "steermouse/Endgame Trackball.xml" "$HOME/Library/Application Support/SteerMouse & CursorSense/Device Definitions/Endgame Trackball.xml"`
2. In SteerMouse, select the Endgame entry that is green (one entry per
   connection: "Endgame" on the cable, "Endgame BT" on Bluetooth) and open the
   Wheel tab. Expected rows:

   | Row | Setting |
   | --- | --- |
   | Roll Up | Scroll, Linear |
   | Roll Down | Scroll, Linear |
   | Roll Left | Scroll, Linear |
   | Roll Right | Scroll, Linear |

   Tilt rows must not be shown for the Endgame.

## Put it back

1. Copy the file:
   `mkdir -p "$HOME/Library/Application Support/SteerMouse & CursorSense/Device Definitions" && cp "steermouse/Endgame Trackball.xml" "$HOME/Library/Application Support/SteerMouse & CursorSense/Device Definitions/"`
2. Quit SteerMouse (Cmd+Q).
3. Restart its background helper, which reads device files only when it starts:
   `launchctl kickstart -k gui/$(id -u)/jp.plentycom.boa.SteerMouse`
   (a logout and login does the same).
4. Open SteerMouse, select the green Endgame entry, open the Wheel tab and set
   the four rows as in the table. Do this once on the cable and once on
   Bluetooth; SteerMouse keeps a separate entry for each.

## Facts to know before changing anything

- Device IDs in the file: vendor 0x0011 / product 0x0006 is the Endgame on the
  USB cable. Vendor 0x1D50 / product 0x615E is the Endgame on Bluetooth, and
  every ZMK Bluetooth device reports that same ID (also the Corne Min keyboard),
  so the file applies to them too.
- Do not use Tilt Left / Tilt Right for the Endgame. "Scroll" there gives the
  timed steps at any Interval, and "No Action" blocks left-right scroll.
- A SteerMouse update replaces `/Applications/SteerMouse.app` only and should
  keep the file and the settings (not tested). SteerMouse's own Uninstall, a
  deleted Library folder or a new Mac lose them.
- The key `hScrollOp` (values `None`, `Tilt`, `Roll`) is not in SteerMouse's
  manual. It comes from SteerMouse's own device files in
  `/Applications/SteerMouse.app/Contents/Resources/Device Definitions/`. If a
  later version ignores the file, look there for the current format (search for
  a device with a rolling side wheel, such as "Logitech MX Master.xml").
- To read SteerMouse's saved settings without its window: the store
  `$HOME/Library/Application Support/SteerMouse & CursorSense/Device.smsetting`
  is a Core Data file whose keyed archive (binary plist) starts at byte 0x40.
  Read it only; change settings in the app.
- To measure what apps really receive, use a scroll event tap placed with tail
  append. A tap placed with head insert sees the events before SteerMouse
  changes them.

## Test status

- Cable: verified 2026-11-14. Apps received only the trackball's own events,
  each a 10-point step in both directions; 0.62 points per unit of ball movement
  left-right against 0.64 up-down.
- Bluetooth: the ID was added to the file on 2026-11-14. Not yet loaded or
  tested on that day.
