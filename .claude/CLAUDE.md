# Endgame Trackball ZMK Config

## Key Position Reference

- **Always** reference `KEY_POSITIONS.svg` when the user gives binding slot numbers.
- Canonical layout: **12 slots per layer** — positions 0-7 are physical buttons, 8-9 are left encoder (CW/CCW), 10-11 are right encoder (CW/CCW).
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

## Button Layout (physical, top-right CCW per buttons.dtsi)

```
[pos 0]  [pos 1]
[pos 2]  [pos 3]
[pos 4]  [pos 5]
[pos 6]  [pos 7]

ENC0 CW=pos 8   ENC0 CCW=pos 9
ENC1 CW=pos 10  ENC1 CCW=pos 11
```

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
