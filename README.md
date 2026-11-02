# zmk-studio-extras

Edit **combos / tap-dances / macros** of any ZMK keyboard from the [chibiboard app](https://tibiboard.net/).
どの ZMK キーボードでも、コンボ・タップダンス・マクロをちびボードアプリから編集できるようにします。

Your keymap, layers and settings stay as they are. Only empty slots are added.
今のキー配置・レイヤー・設定はそのまま。空き枠を足すだけです。

- AI を使う人 / With AI → [AI に頼む](#ai)
- 手で入れる人 / By hand → [1](#1-zmk-studio) → [2](#2-westyml) → [3](#3-keymap) → [4](#4-check)

---

## 1. ZMK Studio

ZMK Studio が使えるキーボードにする。/ Make your keyboard ZMK Studio-ready.
公式 / Official: https://zmk.dev/docs/features/studio

`config/<keyboard>.conf`
```ini
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
```

`build.yaml` (central / left side only)
```yaml
include:
  - board: nice_nano//zmk        # ← your board
    shield: corne_left           # ← your shield (central side)
    snippet: studio-rpc-usb-uart
```

Physical layout がない時 / If your keyboard has no physical layout:
https://zmk.dev/docs/development/hardware-integration/physical-layouts

## 2. west.yml

`config/west.yml` — `zmk` の取得先だけ変える。/ Only change where `zmk` comes from.
```yaml
manifest:
  remotes:
    - name: tibiboard
      url-base: https://github.com/tibiboard
  projects:
    - name: zmk
      remote: tibiboard
      revision: studio-extras
      import: app/west.yml
  self:
    path: config
```

## 3. keymap

[`config/studio_extras_slots.dtsi`](config/studio_extras_slots.dtsi) を `config/` に置く。/ Copy it into your `config/`.

`config/<keyboard>.keymap`
```c
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include "studio_extras_slots.dtsi"   // ← add this line
```

## 4. Check

1. Build → flash (central side).
2. [chibiboard app](https://tibiboard.net/) → connect → **Combos / Tap-Dances / Macros** tabs work.

元に戻す / Undo: `west.yml` を元に戻し、`#include` の行を消してビルド。/ Revert `west.yml`, remove the `#include`, rebuild.

---

## AI

Paste this to your AI (Claude, ChatGPT, Codex, ...) in your zmk-config repo.
AI に、zmk-config のリポジトリでこれを貼る。

```text
Read https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md
and add zmk-studio-extras to this zmk-config. Keep my keymap, layers and
settings unchanged. Make the build pass.
```

---

Tested: IPPO (30 keys, split, nice!nano), Corne (nice!nano). ZMK base: upstream `8feeb52e` (2026-03).
Not included: chibiboard vibration features. / 振動の機能は入っていません。
License: MIT (same as ZMK).
