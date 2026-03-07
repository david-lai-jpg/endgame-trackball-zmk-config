# Endgame Trackball ZMK Config

## Key Position Reference

- **Always** reference `KEY_POSITIONS.svg` when the user gives binding slot numbers.
- Canonical layout: **12 slots per layer** — positions 0-7 are physical buttons, 8-11 are encoder actions (see wiring below).
- **Always** reference `keymap-drawer/efogtech_trackball_0.svg` to see current keymap bindings visually.

## Keymap Drawer (MANDATORY)

**Before every commit that touches the keymap, you MUST regenerate the keymap drawings locally:**

```sh
keymap -c keymap_drawer.config.yaml parse -z config/efogtech_trackball_0.keymap > keymap-drawer/efogtech_trackball_0.yaml
keymap -c keymap_drawer.config.yaml draw -j config/efogtech_trackball_0.json keymap-drawer/efogtech_trackball_0.yaml > keymap-drawer/efogtech_trackball_0.svg
```

Add any new custom bindings to `keymap_drawer.config.yaml` → `raw_binding_map` so they render with proper labels/icons.

**Never commit and push keymap changes without rerendering first.**

The GitHub Actions workflow (`.github/workflows/draw-keymap.yml`) also runs this automatically on push — but local regeneration is required before committing so the SVG is always in sync.

## Layer Architecture

| Index | Name | Activation | Purpose |
|-------|------|------------|---------|
| 0 | `LAYER_DEFAULT` | Boot | Main layer: hold-taps, click, copy/paste |
| 1 | `LAYER_EXTRAS` | Hold pos 6 (MB4) | Copy/cut/paste/undo macros |
| 2 | `LAYER_DEVICE` | Hold pos 7 (MB5) | BT, RGB, Studio unlock, soft-off |
| 3 | `LAYER_SCROLL` | Hold pos 1 or pos 3 | Trackball → scroll wheel |
| 4 | `LAYER_SNIPE` | Hold pos 0 | Trackball → precision snipe |
| 5 | `LAYER_USER` | — | User-defined |

## Button Layout (corrected via kscan0 GPIO remapping in efogtech_trackball_0.dts)

```
Physical grid (left/right pairs, top to bottom):

  [pos 0: TL outer]   [pos 1: TR outer]
  [pos 2: ML outer]   [pos 3: MR outer]
  [pos 4: ML inner]   [pos 5: MR inner]
  [pos 6: BL inner]   [pos 7: BR inner]
```

**⚠️ Do NOT trust the "top-right then CCW" comment in `buttons.dtsi`.** The `kscan0` input-gpios
ordering in the DTS file remaps the raw GPIO order. The mapping above is the final resolved order
used in the keymap.

## Encoder Wiring (from efogtech_trackball_0.dts line 15)

```c
#define DECLARE_ENCODERS sensor-bindings = <&rotenc_follower 10 8 &rotenc_follower 9 11>
// sensors order: <&left_encoder &right_encoder>
```

| Encoder | Action | Position | Default Binding |
|---------|--------|----------|-----------------|
| Left    | Scroll left (CCW)  | pos 8  | `C_VOL_UP` (Volume Up) |
| Left    | Scroll right (CW)  | pos 10 | `C_VOL_DN` (Volume Down) |
| Right   | Scroll left (CCW)  | pos 11 | `LC(LS(TAB))` (Prev Tab) |
| Right   | Scroll right (CW)  | pos 9  | `LC(TAB)` (Next Tab) |

**⚠️ The `rotenc_follower` macro arguments are `<CW CCW>`, so `<&rotenc_follower 10 8>` means
left encoder CW → pos 10, CCW → pos 8.**

## Trackball Modes

- **Default**: pointer with accel, rotate, BLE rate limit
- **`LAYER_SCROLL`**: XY → scroll (1/3 scale, Y-inverted, clamped)
- **`LAYER_SNIPE`**: precision cursor (1/4 scale)

Sensitivity adjustable at runtime: `scrlsens P2SM_INC/DEC 1` (scroll speed), `sens P2SM_INC/DEC 1` (snipe speed).

## Custom Behaviors

| Behavior | Type | Description |
|----------|------|-------------|
| `cdch` | hold-tap | tap → `cdc` (copy/paste dance), hold → `&mo LAYER_SCROLL` |
| `cdc` | tap-dance | tap → copy (`K_COPY`), double-tap → paste (`K_PASTE`) |
| `rgb_tog` | macro | `RGB_TOG` + `EP_TOG` together |
| `rgb_off` | macro | `EP_OFF` + `RGB_OFF` together |

## Adding New Bindings

When adding a custom behavior binding to the keymap, ALWAYS add a corresponding entry to `keymap_drawer.config.yaml` → `parse_config.raw_binding_map` with `tap`/`hold`/`shifted` labels and/or MDI icons. Otherwise it renders as raw text.
