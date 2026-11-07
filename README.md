# zmk-studio-extras

対応するZMKキーボードに、[ちびボードアプリ](https://tibiboard.net/shop/使い方ガイド#app-downloads)でコンボ・タップダンス・マクロを編集する機能を追加する導入キットです。
Add **combo / tap-dance / macro editing** to compatible ZMK keyboards for the [chibiboard app](https://tibiboard.net/shop/使い方ガイド#app-downloads).

既存のキー配置を引き継ぐ前提で、専用のZMKと編集用の空き枠を導入します。機種に応じたビルドが必要です。
The kit uses an extended ZMK and empty editing slots while preserving your source keymap. A keyboard-specific build is required.

標準のZMKファームのままでは、この追加編集は使えません。ファームに定義済みのマクロ等をキーへ割り当てる操作と、中身の編集は別です。振動機能は含みません。
These editors require the extended firmware. Assigning an existing behavior is different from editing its contents. Vibration is not included.

先に現在のファーム（左右があれば両方）、zmk-config一式、アプリで変更した設定の控えを保存してください。ソースとアプリで変更した設定が同じとは限りません。
Back up your current firmware for both halves, complete zmk-config and any runtime settings before migrating. Your source keymap may differ from settings edited in an app.

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

URL を開けない AI には、[AGENTS.md](AGENTS.md) の中身を貼り、[studio_extras_slots.dtsi](config/studio_extras_slots.dtsi) を自分で `config/` に置く。
If your AI can't open URLs, paste [AGENTS.md](AGENTS.md) and put [studio_extras_slots.dtsi](config/studio_extras_slots.dtsi) into `config/` yourself.

### 3. ファームを作って焼く / Build and flash

```bash
git push
```

GitHub の **Actions** → 最新のビルド → **Artifacts** から `.uf2` をダウンロード → 分割式は左右それぞれに対応するファームを書き込む → [4. 確かめる](#4-確かめる--check)
GitHub **Actions** → latest run → **Artifacts** → download `.uf2` → flash each half with its matching firmware → [4. Check](#4-確かめる--check)

---

## 1. ZMK Studio

ZMK Studio が使えるキーボードにする。/ Make your keyboard ZMK Studio-ready.
公式 / Official: https://zmk.dev/docs/features/studio

`config/<keyboard>.conf`
```ini
CONFIG_ZMK_STUDIO=y
```

既存のStudio解錠キー（`&studio_unlock`）を維持します。未配置なら、どのキーに置くかを決めて追加してください。ロックを無効にする設定は必須ではありません。
Keep an existing `&studio_unlock` key, or explicitly choose a key for it if missing. Disabling Studio locking is not required.

`build.yaml`(`snippet` は left/right の名前ではなく central 側だけ / `snippet` on the central side only)
```yaml
include:
  - board: nice_nano//zmk        # ← 自分のボード / your board
    shield: corne_left           # ← 自分のシールド / your shield
    snippet: studio-rpc-usb-uart
  - board: nice_nano//zmk
    shield: corne_right
```

古いボード名は書き換える(同じ基板)/ Rename old board names (same hardware):
`nice_nano_v2` → `nice_nano//zmk`、`seeeduino_xiao_ble` → `xiao_ble//zmk`

キーの並びの定義(physical layout)がない時 / If your keyboard has no physical layout:
https://zmk.dev/docs/development/hardware-integration/physical-layouts

## 2. west.yml

`config/west.yml` — 自分の他のプロジェクト・モジュールは残し、`zmk` の取得先を変更します。下は該当部分の例です。
Keep your other projects/modules. The example below shows the ZMK portion; do not overwrite unrelated entries.
```yaml
manifest:
  remotes:
    - name: tibiboard
      url-base: https://github.com/tibiboard
  projects:
    - name: zmk
      remote: tibiboard
      revision: ba17c589301799dd1effd17b250e8049b32b6010 # verified studio-extras revision
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
今ある `combos` はそのままで大丈夫。/ Existing `combos` can stay as they are.

## 4. 確かめる / Check

1. ビルド → 対応するファームを書き込む。分割式では左右のファームを取り違えないでください。/ Build → flash matching firmware for each half of a split keyboard.
2. [ちびボードアプリ](https://tibiboard.net/) でつなぐ → **コンボ・タップダンス・マクロ** のタブが使える。/ Connect in the [chibiboard app](https://tibiboard.net/) → **Combos / Tap-Dances / Macros** tabs work.

3. コンボ・タップダンス・マクロを1つずつ作り、発動・変更・解除と、電源を入れ直しても設定が残るかを実機で確認します。通常の入力と左右接続も確認します。
Check each editor, execution, clearing and persistence after a power cycle, as well as normal typing and split connectivity.

元に戻す: 保存した元のファームを正しい側へ書き戻します。ソースから作り直すなら、`west.yml`だけでなく`.conf`・`.keymap`・`build.yaml`など変更したファイルを元の控えへ戻します。アプリ内の設定は別途復元が必要な場合があります。
Undo: flash the backed-up original firmware to the correct half. When rebuilding from source, restore all changed files, including `.conf`, `.keymap` and `build.yaml`, from the backup. Runtime settings may need separate restoration.

---

確認範囲（2026-11-07）：IPPO左・Corne左のローカルビルド、Corne左右のGitHub Actionsビルドは成功。実機での入力・各編集・再起動後の保存は未確認です。全機種の互換性を保証するものではありません。
Verified scope: local IPPO central and Corne central builds; Corne central/peripheral builds in GitHub Actions. Hardware operation and settings persistence have not been verified. Compatibility with every ZMK keyboard is not guaranteed.
基準のZMKは本家 `8feeb52e`（2026-03）、この案内は拡張コミット `ba17c589` を固定参照します。
ZMK base: upstream `8feeb52e` (2026-03); this guide pins extras commit `ba17c589`.
振動の機能は入っていません。/ chibiboard vibration features are not included.
License: MIT (ZMK と同じ / same as ZMK).
