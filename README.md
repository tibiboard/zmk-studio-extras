# zmk-studio-extras

ZMKキーボードで、[ちびボードアプリ](https://tibiboard.net/shop/使い方ガイド#app-downloads)を使うための導入キットです。お使いのキーボード用ファームウェアに、次の2つを組み込みます。

1. **ZMK Studioへの対応**
2. **ちびボードアプリ用の拡張機能**（コンボ・タップダンス・マクロの設定変更）

ZMK Studioに対応済みの場合は、2の拡張機能を追加します。機種に合わせたファームウェアの作成・書き込みが必要です。既存のキー配置・レイヤー・コンボなどを変更するための手順ではありません。振動機能は含みません。

This kit prepares keyboard-specific firmware for the chibiboard app: **(1) ZMK Studio support** and **(2) the app's combo / tap-dance / macro editing extensions**. If Studio is already supported, add the extensions. Preserve the existing keymap, layers and combos. Vibration is not included.

先に現在のファーム（左右があれば両方）、zmk-config一式、アプリで変更した設定の控えを保存してください。ソースとアプリで変更した設定が同じとは限りません。
Back up your current firmware for both halves, complete zmk-config and any runtime settings before migrating. Your source keymap may differ from settings edited in an app.

## 導入方法を選ぶ

次のどちらか一方で進めてください。行う変更は同じです。

- **[AIに頼む](#ai-setup)**：AIに設定ファイルの修正とビルドを依頼します。
- **[自分で修正する](#manual-setup)**：変更する場所と理由を確認しながら、自分で設定ファイルを修正します。

Choose either **With AI** or **Manual setup**. Both methods prepare the same firmware.

---

<a id="ai-setup"></a>

## 方法1：AIに頼む / With AI

この方法では、ZMK Studioへの対応と、ちびボードアプリ用の拡張機能の追加を、まとめてAIに依頼します。以下の3つを進めてください。「自分で修正する」の手順を別に行う必要はありません。

Ask the AI to prepare both Studio support and the app extensions. Follow the three steps below; you do not also need to follow the manual setup.

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

### 2. 修正内容をAIに依頼する / Ask the AI to make the changes

```text
https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md を読んで、
この zmk-config をZMK Studioに対応させ、ちびボードアプリ用の拡張機能も追加してください。
ZMK Studioに対応済みの部分は維持してください。
今のキー配置・レイヤー・設定は変えないでください。ビルドが通るまで確かめてください。
```

```text
Read https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md
and add zmk-studio-extras to this zmk-config. Keep my keymap, layers and
settings unchanged. Make the build pass.
```

URL を開けない AI には、[AGENTS.md](AGENTS.md) の中身を貼り、[studio_extras_slots.dtsi](config/studio_extras_slots.dtsi) を自分で `config/` に置く。
If your AI can't open URLs, paste [AGENTS.md](AGENTS.md) and put [studio_extras_slots.dtsi](config/studio_extras_slots.dtsi) into `config/` yourself.

### 3. ファームウェアを作成して書き込む / Build and flash

```bash
git push
```

GitHub の **Actions** → 最新のビルド → **Artifacts** から `.uf2` をダウンロード → 分割式は左右それぞれに対応するファームを書き込む → [導入後の動作確認](#verify)
GitHub **Actions** → latest run → **Artifacts** → download `.uf2` → flash each half with its matching firmware → [Check after installation](#verify)

---

<a id="manual-setup"></a>

## 方法2：自分で修正する / Manual setup

**1はZMK Studioに対応させる作業、2・3はちびボードアプリ用の機能を追加する作業です。最後に4でファームウェアを作成して書き込みます。**

ZMK Studioに対応済みの場合は、1の設定がそろっていることを確認してから、2へ進みます。既存のキー配置・レイヤー・コンボなどの設定は維持してください。

Step 1 prepares ZMK Studio support; steps 2 and 3 add the chibiboard app extensions. Step 4 builds and flashes the firmware. If Studio already works, check the existing settings in step 1, then proceed to step 2. Preserve existing bindings and settings.

### 1. ZMK Studioに対応させる / Enable ZMK Studio

アプリがキーボードと通信できるように、Studio機能・USB通信・解錠キー・キーの並びの定義を設定します。

Enable Studio and its USB transport, keep an unlock key, and provide a physical layout.
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

### 2. ちびボードアプリ用のZMKに切り替える / Use the extended ZMK

**ここからが、ちびボードアプリ用の変更です。** ファームウェアを作る際のZMKの取得先を、コンボ・タップダンス・マクロの編集機能を加えた版に変更します。

**The app-specific changes start here.** Select the ZMK version that includes the combo, tap-dance and macro editors.

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

### 3. アプリから編集するための枠を追加する / Add editing slots

キー配置ファイルに、アプリがコンボ・タップダンス・マクロの設定を書き込める空き枠を追加します。今あるキーの位置や、設定済みのコンボを移動する作業ではありません。

Add empty slots for app-based combo, tap-dance and macro settings. Keep your existing key positions and combos.

[`config/studio_extras_slots.dtsi`](config/studio_extras_slots.dtsi) を自分の `config/` に置く。/ Copy it into your `config/`.

`config/<keyboard>.keymap`
```c
#include <behaviors.dtsi>
#include <dt-bindings/zmk/keys.h>
#include "studio_extras_slots.dtsi"   // ← この行を足す / add this line
```
今ある `combos` はそのままで大丈夫。/ Existing `combos` can stay as they are.

### 4. ファームウェアを作成して書き込む / Build and flash

変更した設定を保存し、GitHub Actionsまたはローカル環境でビルドします。GitHub Actionsの場合は、最新のビルドの **Artifacts** からファームウェアをダウンロードしてください。

分割式では、左右それぞれに対応するファームウェアを書き込みます。

Save your changes, build with GitHub Actions or locally, then flash each half with its matching firmware. For GitHub Actions, download the firmware from the latest build's **Artifacts**.

---

<a id="verify"></a>

## 導入後の動作確認（どちらの方法でも行います） / Check after either method

1. [ちびボードアプリ](https://tibiboard.net/) でつなぐ → **コンボ・タップダンス・マクロ** のタブが使える。/ Connect in the [chibiboard app](https://tibiboard.net/) → **Combos / Tap-Dances / Macros** tabs work.

2. コンボ・タップダンス・マクロを1つずつ作り、発動・変更・解除と、電源を入れ直しても設定が残るかを実機で確認します。通常の入力と左右接続も確認します。
Check each editor, execution, clearing and persistence after a power cycle, as well as normal typing and split connectivity.

### 元に戻す / Undo

保存した元のファームを正しい側へ書き戻します。ソースから作り直すなら、`west.yml`だけでなく`.conf`・`.keymap`・`build.yaml`など変更したファイルを元の控えへ戻します。アプリ内の設定は別途復元が必要な場合があります。
Undo: flash the backed-up original firmware to the correct half. When rebuilding from source, restore all changed files, including `.conf`, `.keymap` and `build.yaml`, from the backup. Runtime settings may need separate restoration.

---

確認範囲（2026-11-07）：IPPO左・Corne左のローカルビルド、Corne左右のGitHub Actionsビルドは成功。実機での入力・各編集・再起動後の保存は未確認です。全機種の互換性を保証するものではありません。
Verified scope: local IPPO central and Corne central builds; Corne central/peripheral builds in GitHub Actions. Hardware operation and settings persistence have not been verified. Compatibility with every ZMK keyboard is not guaranteed.
基準のZMKは本家 `8feeb52e`（2026-03）、この案内は拡張コミット `ba17c589` を固定参照します。
ZMK base: upstream `8feeb52e` (2026-03); this guide pins extras commit `ba17c589`.
振動の機能は入っていません。/ chibiboard vibration features are not included.
License: MIT (ZMK と同じ / same as ZMK).
