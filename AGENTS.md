# AGENTS.md — add zmk-studio-extras to a zmk-config

Goal: add combo / tap-dance / macro editing to a compatible keyboard using the chibiboard app. Do not promise compatibility or hardware verification before testing.
Before changing anything, back up the original firmware, complete config repository and runtime settings. Keep a reversible commit or file backup.
Rule: do not change the user's existing keymap bindings, layers, combos, or settings. Only add.

## Steps

1. Find the central (usually left) shield in `build.yaml`.
   Old board names do not build on this ZMK. Rename them in `build.yaml` first:
   `nice_nano_v2` → `nice_nano//zmk`, `seeeduino_xiao_ble` → `xiao_ble//zmk`
   (same hardware). Other boards: check `app/boards` of https://github.com/tibiboard/zmk/tree/studio-extras .
2. ZMK Studio
   - Add to `config/<keyboard>.conf` (create if missing):
     ```
     CONFIG_ZMK_STUDIO=y
     ```
     It also applies to the peripheral half. That is fine (tested); no need to split the file.
   - Keep Studio locking and an existing `&studio_unlock` key. If no unlock key exists, ask the owner where to put it; do not silently replace an existing binding or disable locking.
   - In `build.yaml`, add `snippet: studio-rpc-usb-uart` to the central side entry only.
   - The keyboard needs a physical layout. Check the shield/board for `zmk,physical-layout`.
     If missing, add one: https://zmk.dev/docs/development/hardware-integration/physical-layouts
3. `config/west.yml`: make the `zmk` project come from remote `tibiboard`
   (`url-base: https://github.com/tibiboard`), `revision: ba17c589301799dd1effd17b250e8049b32b6010` (verified studio-extras commit), `import: app/west.yml`.
   Keep the user's other projects/modules.
   `.github/workflows/build.yml` stays as is (`zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`
   reads this west.yml, so it fetches tibiboard's zmk).
4. Download `config/studio_extras_slots.dtsi` into the user's `config/`:
   https://raw.githubusercontent.com/tibiboard/zmk-studio-extras/main/config/studio_extras_slots.dtsi
   (If you can't open URLs, ask the user to download that file into `config/`.)
   Add `#include "studio_extras_slots.dtsi"` after the `#include <dt-bindings/zmk/keys.h>` line
   of the user's keymap. If the keymap already has a `combos { ... }` node, leave it: both merge.
5. Build (GitHub Actions or local `west build`) until it passes. Do not delete user content to
   make it pass.

## Known build issues

| Error | Fix |
|---|---|
| `key-positions` out of range | keyboard has < 2 keys: impossible; check the include is inside the right keymap |
| node label already defined (`spare_td1`, ...) | the user already had old spare slots; remove the old copy, keep one |
| board not found (`nice_nano_v2`, `seeeduino_xiao_ble`, ...) | rename as in step 1 |
| Studio tab missing in the app | snippet `studio-rpc-usb-uart` missing on the central side, or flashed the wrong half |

## Undo

Restore the original firmware and every changed config file from the backup (west.yml, build.yaml, .conf and .keymap, as applicable). Do not replace a previous custom fork or pinned revision with upstream main. Runtime settings may need separate restoration.

## Verification scope

Local IPPO central and Corne central builds and Corne central/peripheral GitHub Actions builds have passed. Hardware operation, persistence and compatibility with other models remain unverified. Instruct the owner to test combo/tap-dance/macro execution and clearing, power-cycle persistence, normal typing and split connectivity. Do not describe build success as hardware success.
