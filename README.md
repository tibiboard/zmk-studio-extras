# zmk-studio-extras

ZMKキーボードで、[ちびボードアプリ](https://tibiboard.net/shop/使い方ガイド#app-downloads)を使うための導入キットです。お使いのキーボード用ファームウェアに、次の2つを組み込みます。

1. **ZMK Studioへの対応**
2. **ちびボードアプリ用の拡張機能**（コンボ・タップダンス・マクロの設定変更）

ZMK Studioに対応済みの場合は、2の拡張機能を追加します。機種に合わせたファームウェアの作成・書き込みが必要です。既存のキー配置・レイヤー・コンボなどを変更するための手順ではありません。振動機能は含みません。

This kit prepares keyboard-specific firmware for the tibiboard app: **(1) ZMK Studio support** and **(2) the app's combo / tap-dance / macro editing extensions**. If Studio is already supported, add the extensions. Preserve the existing keymap, layers and combos. Vibration is not included.

先に現在のファーム（左右があれば両方）、zmk-config一式、アプリで変更した設定の控えを保存してください。ソースとアプリで変更した設定が同じとは限りません。
Back up your current firmware for both halves, complete zmk-config and any runtime settings before migrating. Your source keymap may differ from settings edited in an app.

## 導入方法を選ぶ

次のどちらか一方で進めてください。行う変更は同じです。

- **[AIに頼む](#ai-setup)**：AIに設定ファイルの修正とビルドを依頼します。
- **[自分で修正する](#manual-setup)**：変更する場所と理由を確認しながら、自分で設定ファイルを修正します。

Choose either **With AI** or **Manual setup**. Both methods prepare the same firmware.

---

<a id="ai-setup"></a>

## AIに頼む / With AI

キーボードの設定ファイルを扱えるAI（Claude Code・Codexなど）に、次の文章をそのまま渡してください。

```text
私のZMKキーボードで、ちびボードアプリを使えるようにしてください。
https://github.com/tibiboard/zmk-studio-extras/blob/main/AGENTS.md
の手順に従い、ZMK Studioへの対応と、コンボ・タップダンス・マクロを
アプリで編集するための拡張機能を追加してください。
今のキー配置・レイヤー・設定は維持し、変更前に戻せる控えを残してください。
必要な設定ファイルが分からなければ、私に確認してください。
ビルドを確認し、書き込むファイルと書き込み方を案内してください。
```

---

<a id="manual-setup"></a>

## 自分で修正する / Manual setup

**1はZMK Studioに対応させる作業、2・3はちびボードアプリ用の機能を追加する作業です。最後に4でファームウェアを作成して書き込みます。**

ZMK Studioに対応済みの場合は、1の設定がそろっていることを確認してから、2へ進みます。既存のキー配置・レイヤー・コンボなどの設定は維持してください。

Step 1 prepares ZMK Studio support; steps 2 and 3 add the tibiboard app extensions. Step 4 builds and flashes the firmware. If Studio already works, check the existing settings in step 1, then proceed to step 2. Preserve existing bindings and settings.

### 1. ZMK Studioに対応させる / Enable ZMK Studio

アプリがキーボードと通信できるように、Studio機能・USB通信・解錠キー・キーの並びの定義を設定します。

Enable Studio and its USB transport, keep an unlock key, and provide a physical layout.
公式 / Official: https://zmk.dev/docs/features/studio

親機側のビルドに、以下の例のように `CONFIG_ZMK_STUDIO=y` とUSB通信用の `snippet` を追加します。すでに設定されている場合は維持してください。

Enable Studio and USB transport on the central build, as shown below. Preserve existing working settings.

既存のStudio解錠キー（`&studio_unlock`）を維持します。未配置なら、どのキーに置くかを決めて追加してください。ロックを無効にする設定は必須ではありません。
Keep an existing `&studio_unlock` key, or explicitly choose a key for it if missing. Disabling Studio locking is not required.

`build.yaml` — **以下はnice!nanoを使うCorneの例です。** `board` と `shield` は、自分のキーボードの組み合わせを使ってください。Studio設定と `snippet` は親機側（central）へ追加します。

**Example: Corne with nice!nano.** Keep the board/shield pair for your hardware. Add Studio settings and the USB snippet to the central build.
```yaml
include:
  - board: nice_nano//zmk        # ← 自分のボード / your board
    shield: corne_left           # ← 自分のシールド / your shield
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y
  - board: nice_nano//zmk
    shield: corne_right
```

**XIAO BLE（Seeed Studio XIAO nRF52840）の場合**

ボード名は `xiao_ble//zmk` です。上のCorne用の例をそのまま使うのではなく、今使っているXIAO対応キーボードの `shield`・配線定義を維持してください。nice!nanoとXIAOでは端子の対応が違います。

For XIAO BLE, use `xiao_ble//zmk` with your existing XIAO-compatible shield and pin definitions. Do not copy the nice!nano/Corne combination: they use different interconnects.

この導入キットが使うZMKでの旧名からの変更（同じハードウェアの場合のみ）：

- nice!nano v2：`nice_nano_v2` → `nice_nano//zmk`
- XIAO BLE：`seeeduino_xiao_ble` → `xiao_ble//zmk`

[公式の対応ハードウェア一覧](https://zmk.dev/docs/hardware)

キーの並びの定義(physical layout)がない時 / If your keyboard has no physical layout:
https://zmk.dev/docs/hardware-integration/physical-layouts

### 2. ちびボードアプリ用のZMKに切り替える / Use the extended ZMK

これにより、ちびボードアプリでマクロ・タップダンス・コンボの設定を変更できるようになります。

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

1. [ちびボードアプリ](https://tibiboard.net/shop/使い方ガイド#app-downloads) でつなぐ → **コンボ・タップダンス・マクロ** のタブが使える。/ Connect in the [tibiboard app](https://tibiboard.net/shop/使い方ガイド#app-downloads) → **Combos / Tap-Dances / Macros** tabs work.

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
振動の機能は入っていません。/ tibiboard vibration features are not included.
License: MIT (ZMK と同じ / same as ZMK).
