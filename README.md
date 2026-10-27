# ZMK config for beekeeb Toucan2 Keyboard

[The beekeeb Toucan2 Keyboard](https://beekeeb.com/introducing-toucan2/) is a wireless split 42-key column‑stagger keyboard that a display and a trackpad, with an aggressive stagger on the pinky columns.

# Customizations

- **Keymap**: [config/toucan.keymap](config/toucan.keymap)
- **General configs**: [config/toucan_left.conf](config/toucan_left.conf) and [config/toucan_right.conf](config/toucan_right.conf)
- **Swipe, zoom and touch-layer bindings**: the `&swipe_button_mapper`, `&zip_zoom_mapper` and `&is_touching_processor` overrides in [config/toucan.keymap](config/toucan.keymap)
- **Trackpad settings**: the `&tps43_trackpad` override in [config/toucan_right.keymap](config/toucan_right.keymap), on top of upstream's node in [boards/shields/toucan/toucan_right.overlay](boards/shields/toucan/toucan_right.overlay)

# Local deviations from upstream

This fork differs from [beekeeb/zmk-keyboard-toucan2](https://github.com/beekeeb/zmk-keyboard-toucan2) in the
following ways. `boards/shields/toucan/` stays byte-identical to upstream so merges never conflict: every local
change is a devicetree override in `config/`.

### Touch layer is `TOUCH_L`, not 4

`is_touching_processor` raises a layer while a finger rests on the trackpad. Upstream binds it to `&mo 4`, which
is `SPE_L` (French accents) here, so every trackpad touch would switch to the accent layer.
[config/toucan.keymap](config/toucan.keymap) overrides it to `&mo TOUCH_L`. The symbolic name follows any layer
reordering.

### Linux shortcuts

Upstream ships macOS bindings with a `TOUCAN_WIN_MODE` toggle for Windows. This fork overrides
`&zip_zoom_mapper` and `&swipe_button_mapper` in [config/toucan.keymap](config/toucan.keymap) with GNOME
chords.

### Trackpad wake-up

Upstream sets `report-rate-lp2 = <640>`: after ~30 s without a touch, the pad samples every 640 ms, so the
cursor takes up to ~1 s to move on the next touch.
[config/toucan_right.keymap](config/toucan_right.keymap) restores the chip default (160 ms).

### Keymap and tooling

The keymap is fully rewritten (5 typing layers, `TOUCH_L`, two pointer-speed layers, home-row mods, combos,
French-accent macros) and split across `config/*.dtsi` includes. The `keymap-drawer/` diagrams and
[scripts/draw.py](scripts/draw.py) are local additions with no upstream counterpart. Upstream's own
`config/toucan.keymap` is always resolved in favour of this one during merges.

# License

The code in this repo is available under the MIT license.

The included shield nice_view_gem is modified from https://github.com/M165437/nice-view-gem licensed under the MIT License.

The linked trackpad module is based on https://github.com/geeksville/zmk_driver_azoteq

ZMK code snippets are taken from the ZMK documentation under the MIT license.

The embedded font QuinqueFive is designed by GGBotNet, licensed under under the SIL Open Font License, Version 1.1.
