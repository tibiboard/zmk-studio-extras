# AGENTS.md — add zmk-studio-extras to a zmk-config

Goal: the user's keyboard can edit combos / tap-dances / macros from the chibiboard app.
Rule: do not change the user's existing keymap bindings, layers, combos, or settings. Only add.

## Steps

1. Find the central (usually left) shield in `build.yaml`.
2. ZMK Studio
   - Add to `config/<keyboard>.conf` (create if missing):
     ```
     CONFIG_ZMK_STUDIO=y
     CONFIG_ZMK_STUDIO_LOCKING=n
     ```
   - In `build.yaml`, add `snippet: studio-rpc-usb-uart` to the central side entry only.
   - The keyboard needs a physical layout. Check the shield/board for `zmk,physical-layout`.
     If missing, add one: https://zmk.dev/docs/development/hardware-integration/physical-layouts
3. `config/west.yml`: make the `zmk` project come from remote `tibiboard`
   (`url-base: https://github.com/tibiboard`), `revision: studio-extras`, `import: app/west.yml`.
   Keep the user's other projects/modules.
4. Download `config/studio_extras_slots.dtsi` from this repo into the user's `config/`.
   Add `#include "studio_extras_slots.dtsi"` after the `#include <dt-bindings/zmk/keys.h>` line
   of the user's keymap.
5. Build (GitHub Actions or local `west build`) until it passes. Do not delete user content to
   make it pass.

## Known build issues

| Error | Fix |
|---|---|
| `key-positions` out of range | keyboard has < 2 keys: impossible; check the include is inside the right keymap |
| node label already defined (`spare_td1`, ...) | the user already had old spare slots; remove the old copy, keep one |
| board not found (`nice_nano_v2`) | new ZMK board names: `nice_nano//zmk`, `xiao_ble//zmk` |
| Studio tab missing in the app | snippet `studio-rpc-usb-uart` missing on the central side, or flashed the wrong half |

## Undo

Restore `config/west.yml` (zmk from `zmkfirmware`, `revision: main`) and remove the `#include` line.
