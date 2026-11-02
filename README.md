# zmk-studio-extras

どの ZMK キーボードでも、コンボ・タップダンス・マクロを [ちびボードアプリ](https://tibiboard.net/) から編集できるようにします。
Edit **combos / tap-dances / macros** of any ZMK keyboard from the [chibiboard app](https://tibiboard.net/).

今のキー配置・レイヤー・設定はそのまま。空き枠を足すだけです。
Your keymap, layers and settings stay as they are. Only empty slots are added.

- AI に頼む / With AI → [AI に頼む](#ai-に頼む--with-ai)
- 手で入れる / By hand → [1](#1-zmk-studio) → [2](#2-westyml) → [3](#3-keymap) → [4](#4-確かめる--check)

---

## AI に頼む / With AI

### 1. AI で zmk-config を開く / Open your zmk-config in an AI tool

自分のキーボードの zmk-config リポジトリ(GitHub)を、手元に置いて AI で開く。
Clone your zmk-config repo and open it in an AI coding tool.

```bash
git clone https://github.com/<あなた / you>/zmk-config
cd zmk-config
```

| AI | 開き方 / How to open |
|---|---|
| Claude Code | `claude` |
| Codex | `codex` |
| Cursor など / etc. | フォルダを開く / Open the folder |

### 2. これを貼る / Paste this

```text
https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md を読んで、
この zmk-config に zmk-studio-extras を入れてください。
今のキー配置・レイヤー・設定は変えないでください。ビルドが通るまで確かめてください。
```

```text
Read https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md
and add zmk-studio-extras to this zmk-config. Keep my keymap, layers and
settings unchanged. Make the build pass.
```

URL を開けない AI には、[AGENTS.md](AGENTS.md) の中身をそのまま貼る。
If your AI can't open URLs, paste the contents of [AGENTS.md](AGENTS.md) instead.

### 3. ファームを作って焼く / Build and flash

```bash
git push
```

GitHub の **Actions** → 最新のビルド → **Artifacts** から `.uf2` をダウンロード → 左手(central)に焼く → [4. 確かめる](#4-確かめる--check)
GitHub **Actions** → latest run → **Artifacts** → download `.uf2` → flash the central (left) half → [4. Check](#4-確かめる--check)

---

## 1. ZMK Studio

ZMK Studio が使えるキーボードにする。/ Make your keyboard ZMK Studio-ready.
公式 / Official: https://zmk.dev/docs/features/studio

`config/<keyboard>.conf`
```ini
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
```

`build.yaml`(左手 central だけ / central side only)
```yaml
include:
  - board: nice_nano//zmk        # ← 自分のボード / your board
    shield: corne_left           # ← 自分のシールド / your shield
    snippet: studio-rpc-usb-uart
```

キーの並びの定義(physical layout)がない時 / If your keyboard has no physical layout:
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

[`config/studio_extras_slots.dtsi`](config/studio_extras_slots.dtsi) を自分の `config/` に置く。/ Copy it into your `config/`.

`config/<keyboard>.keymap`
```c
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include "studio_extras_slots.dtsi"   // ← この行を足す / add this line
```

## 4. 確かめる / Check

1. ビルド → 左手(central)に焼く。/ Build → flash the central half.
2. [ちびボードアプリ](https://tibiboard.net/) でつなぐ → **コンボ・タップダンス・マクロ** のタブが使える。/ Connect in the [chibiboard app](https://tibiboard.net/) → **Combos / Tap-Dances / Macros** tabs work.

元に戻す: `west.yml` を元に戻し、`#include` の行を消してビルド。
Undo: revert `west.yml`, remove the `#include` line, rebuild.

---

試したキーボード: IPPO(30キー・左右分割・nice!nano)、Corne(nice!nano)。元の ZMK: 本家 `8feeb52e`(2026-03)。
Tested: IPPO (30 keys, split, nice!nano), Corne (nice!nano). ZMK base: upstream `8feeb52e` (2026-03).
振動の機能は入っていません。/ chibiboard vibration features are not included.
License: MIT (ZMK と同じ / same as ZMK).
