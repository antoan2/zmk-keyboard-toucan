# CLAUDE.md

ZMK config for a beekeeb Toucan2 (split 42 keys + TPS43 trackpad), ZMK `v0.3`.
Branch for the Toucan2 port: `toucan2-migration` (PR #2). Upstream remote: `toucan2`.

## Build and validation

- No local ZMK toolchain (`west` is not installed). Firmware builds only in GitHub Actions
  (`.github/workflows/build.yml`). Validate a keymap change with `gh run list --branch <branch>`
  then `gh run watch <id> --exit-status`.
- Keymap diagrams are validated locally (see below).

## Push workflow: the drawer bot commits on the branch

`.github/workflows/keymap-drawer.yml` runs on every push that touches `config/*.keymap`,
`config/*.dtsi`, `scripts/draw.py` or `keymap-drawer/config.yaml`. It commits
`[Draw] <last commit message>` back to the same branch.

- Run `git fetch` and `git pull --rebase --autostash` before every push, or the push is rejected.
- Do not hand-edit `keymap-drawer/toucan.yaml` or `keymap-drawer/toucan.svg`: they are generated.

## Keymap drawer

- Run: `uv run --with keymap-drawer --with pyyaml python scripts/draw.py`.
  It overwrites `keymap-drawer/toucan.yaml` and `.svg`. To test without touching the
  working tree, run it on a copy of `config/`, `scripts/`, `keymap-drawer/`.
- Labels: `keymap-drawer/config.yaml` → `parse_config.raw_binding_map`. A macro without an entry
  is drawn with its raw name (e.g. `&sym_arrow`).
- Combo placement: `COMBOS_BY_DISPLAY_LAYER` in `scripts/draw.py` assigns each combo, by its
  drawn label, to a virtual layer (`Combos/Symbols`, `Combos/Edit`). A combo whose label is not
  listed gets no layer, so it is drawn on every combo diagram. The script prints these under
  `[WARN] ... combos left on base layer`.
- Combo position tweaks (`align`, `offset`, `draw_separate`): `parse_config.zmk_combos` in
  `keymap-drawer/config.yaml`, keyed by `combo_<name>`.
- 5 columns: this unit has no switches on the outer columns. `draw.py` drops key positions
  0 11 12 23 24 35 (`DROPPED_KEY_POSITIONS`), renumbers combo positions, and draws on
  `keymap-drawer/toucan_5col.json` (`config/toucan.json` minus those keys). The firmware keeps
  the 42-key matrix, so keymap positions are unchanged.
- `draw.py` does not pass `keymap parse -b <previous yaml>`: the rebase copied stale labels
  from old runs and fails once the drawn layout has fewer keys than the keymap.

## Adding a combo

1. `config/toucan.keymap`: `#define COMBO_KEYS_<NAME> <pos> <pos> // "<keys>" -> "<output>"`.
2. `config/combos.dtsi`: `COMBO(<name>, <binding>, COMBO_KEYS_<NAME>, BASE_L, <timeout>)`
   inside `#ifdef COMBO_KEYS_<NAME>`.
3. Macro binding: define it in `config/macros.dtsi`, map its label in `raw_binding_map`.
4. Add the drawn label to `COMBOS_BY_DISPLAY_LAYER` in `scripts/draw.py`.

## Keymap facts

- Key positions: grid comment at the top of `config/toucan.keymap`. Thumbs: left 36 37 38,
  right 39 40 41 (39 is the innermost right).
- The space thumb (39) is `&space NAV_L SPACE`, a hold-tap: holding activates `NAV_L` and never
  sends a held space. Push-to-talk tools that need a held space use `&kt SPACE` on 41
  (press once to hold, press again to release). ESC lives on the `wer` combo.
- Never edit `boards/shields/toucan/`: it is upstream's file. Override nodes from
  `config/toucan.keymap` (both halves) or `config/toucan_right.keymap` (right half only,
  e.g. `&tps43_trackpad`, which does not exist on the left half; it `#include`s
  `toucan.keymap`).
- Devicetree order on a build: `config/<shield>.overlay` → shield overlays in `boards/` →
  keymap. A `config/*.overlay` cannot reference labels from the shield ("undefined node
  label"); only the keymap can. ZMK picks `config/toucan_right.keymap` over
  `config/toucan.keymap` for the right half.

## Trackpad power (TPS43, driver `beekeeb/zmk_driver_azoteq`)

- Chip modes: active → idle-touch → idle (10 s) → LP1 (20 s, 80 ms) → LP2. The LP2 report rate
  is the wake-up latency after ~30 s without a touch. Upstream sets 640 ms; the local overlay
  sets 160 ms (`config/toucan_right.keymap`).
- ZMK counts key presses and input events (trackpad included) as activity. Each half enters
  deep sleep after `CONFIG_ZMK_IDLE_SLEEP_TIMEOUT` (1 h) of no activity, never while on USB power.
  The driver suspends the pad on `ZMK_ACTIVITY_SLEEP`, and on `IDLE` only with `idle-sleep`.
