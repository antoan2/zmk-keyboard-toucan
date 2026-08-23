# ZMK config for beekeeb Toucan2 Keyboard

[The beekeeb Toucan2 Keyboard](https://beekeeb.com/introducing-toucan2/) is a wireless split 42-key column‑stagger keyboard that a display and a trackpad, with an aggressive stagger on the pinky columns.

# Customizations

- **Keymap**: [config/toucan.keymap](config/toucan.keymap)
- **General configs**: [boards/shields/toucan/toucan_left.conf](boards/shields/toucan/toucan_left.conf) and [boards/shields/toucan/toucan_right.conf](boards/shields/toucan/toucan_right.conf)
- **Swipe shortcuts**: the `swipe_button_mapper` node in [boards/shields/toucan/toucan.dtsi](boards/shields/toucan/toucan.dtsi)
- **Invert scroll / trackpad settings**: the `tps43_trackpad` node in [boards/shields/toucan/toucan_right.overlay](boards/shields/toucan/toucan_right.overlay)

# Local deviations from upstream

This fork differs from [beekeeb/zmk-keyboard-toucan2](https://github.com/beekeeb/zmk-keyboard-toucan2) in the
following ways. Re-check these whenever merging upstream, since upstream owns the files they live in.

### Touch layer is 6, not 4

`is_touching_processor` in [boards/shields/toucan/toucan.dtsi](boards/shields/toucan/toucan.dtsi) raises a layer
while a finger rests on the trackpad. Upstream binds this to `&mo 4`; here it is **`&mo 6`**.

Layer 4 in [config/toucan.keymap](config/toucan.keymap) is `SPE_L` (French accents), so upstream's default would
silently switch the keyboard into the accent layer on every trackpad touch. Layer 6 is `TOUCH_L`, a dedicated
mouse-click layer added for this purpose.

**This is a cross-file coupling with nothing to enforce it.** The layer index is a bare number in a shield
`.dtsi`; the layer it refers to is defined in the keymap. If you reorder or insert layers in
[config/toucan.keymap](config/toucan.keymap), you must update `&mo 6` in the `.dtsi` to match, or the trackpad
will start raising the wrong layer. Both sites carry a comment pointing at the other.

### Linux shortcut mode

Upstream ships macOS bindings with a `TOUCAN_WIN_MODE` toggle for Windows. This fork adds a third arm,
`TOUCAN_LINUX_MODE` (enabled near the top of the `.dtsi`), for the zoom and swipe chords. The macOS and Windows
arms are left intact so upstream merges stay clean. The swipe bindings assume GNOME/KDE defaults and are worth
tuning to your actual desktop environment.

### Keymap and tooling

The keymap is fully rewritten (6 base layers + `TOUCH_L`, home-row mods, combos, French-accent macros) and split
across `config/*.dtsi` includes. The `keymap-drawer/` diagrams and [scripts/draw.py](scripts/draw.py) are local
additions with no upstream counterpart. Upstream's own `config/toucan.keymap` is always resolved in favour of
this one during merges.

# License

The code in this repo is available under the MIT license.

The included shield nice_view_gem is modified from https://github.com/M165437/nice-view-gem licensed under the MIT License.

The linked trackpad module is based on https://github.com/geeksville/zmk_driver_azoteq

ZMK code snippets are taken from the ZMK documentation under the MIT license.

The embedded font QuinqueFive is designed by GGBotNet, licensed under under the SIL Open Font License, Version 1.1.
