# 1U Rotary Encoder Switch — KiCad Library

*[日本語版はこちら](#日本語)*

KiCad symbol and footprint for a low-profile rotary encoder with a push
switch whose mounting dimensions are compatible with Cherry MX style
keyswitches widely used in mechanical keyboards.

The footprint can be placed directly on top of a Cherry MX switch
footprint (hotswap-compatible), allowing you to choose either a
keyswitch or a rotary encoder at the same position on the same PCB.

![Encoder, knob and silicone spacer](Resources/encoder_switch_parts.jpg)

![OLSK60 keyboard using this footprint](Resources/olsk60.jpeg)

## Target component

This library is designed for the rotary encoder ("encoder key switch")
manufactured by Skyloong, commonly available on AliExpress and similar
platforms.

- Reference: [Skyloong_Components — encoder_key_switch](https://github.com/JZ-Skyloong/Skyloong_Components/tree/main)
- Datasheet: [encoder_key_switch_V1_05092023.pdf](https://github.com/JZ-Skyloong/Skyloong_Components/blob/main/encoder_key_switch/encoder_key_switch_V1_05092023.pdf)

The footprint was created independently based on actual measurements of
the part and is not affiliated with or endorsed by Skyloong.

## Contents

| File | Description |
|---|---|
| `1U_Rotary_Encoder_Switch.kicad_sym` | Symbol library (schematic symbol) |
| `1U_Rotary_Encoder_Switch.pretty/1U_Rotary_Encoder_Switch.kicad_mod` | Footprint library |

Requires KiCad 8.0 or later.

## Installation

1. Clone or download this repository.
2. **Symbol**: *Preferences → Manage Symbol Libraries…* → add
   `1U_Rotary_Encoder_Switch.kicad_sym`.
3. **Footprint**: *Preferences → Manage Footprint Libraries…* → add the
   `1U_Rotary_Encoder_Switch.pretty` folder.

With the default library nickname (`1U_Rotary_Encoder_Switch`), the
symbol's Footprint field resolves automatically.

## Pin mapping

| Pad | Symbol pin | Function |
|---|---|---|
| 1 | B | Encoder channel B |
| 2 | A | Encoder channel A |
| 3 | E | Push switch contact |
| 4 | C | Common |

As drawn in the symbol, the push switch closes between **C** and **E**;
the quadrature outputs are **A**/**B** referenced to the common **C**.
See the datasheet for electrical details and timing.

## Design notes

- The center hole (4.1 mm non-plated) accepts the encoder shaft; the
  four circular SMD pads solder to the encoder legs on the front side.
- The `Dwgs.User` layer shows the 1U key area (19.05 mm square) and the
  14 mm body corner marks for alignment with an MX switch footprint.
- Silkscreen is intentionally empty so the footprint can be stacked on
  top of an existing MX (hotswap) footprint without overlapping marks.
- For PCB manufacturing, an ENIG (Electroless Nickel / Immersion Gold)
  surface finish is recommended.

## Firmware example (QMK)

The push switch of this encoder shares its common terminal **C** with
the rotary contacts, and QMK's encoder driver requires C to be wired to
ground. As a result the push switch **cannot** be wired into the key
matrix as an ordinary diode + row × column intersection — one side of
the switch is always GND.

A zero-custom-code pattern that works around this (proven on the OLSK60
keyboard shown above) is to detect **E** as a direct active-low input,
but fold it into the normal matrix scan as a *dedicated row*. The
official QMK documentation covers encoder rotation only and does not
show a recipe for this case; the alternatives — `DIRECT_PINS` (which is
all-or-nothing for the whole board) or custom `matrix_scan` code — are
both more invasive.

### Wiring

| Encoder pad | Connect to |
|---|---|
| 2 (A) / 1 (B) | MCU pins → `pin_a` / `pin_b` (swap the two to flip direction) |
| 4 (C) | GND |
| 3 (E) | one **dedicated row input pin** of the matrix |

### keyboard.json

```jsonc
{
    "encoder": {
        "enabled": true,
        "rotary": [
            {"pin_a": "GP15", "pin_b": "GP14", "resolution": 4}
        ]
    },
    "matrix_pins": {
        "rows": ["GP8", "GP9", "GP10", "GP11", "GP12", "GP23"],
        "cols": ["GP0", "GP1", "..."]
    },
    "layouts": {
        "LAYOUT": {
            "layout": [
                {"matrix": [0, 0], "x": 0, "y": 0},
                {"matrix": [5, 0], "x": 15.5, "y": 4}
            ]
        }
    }
}
```

The last row (`GP23` here) exists *only* for the encoder push switch.
Rotation is assigned with a standard `encoder_map` in the keymap:

```c
#if defined(ENCODER_MAP_ENABLE)
const uint16_t PROGMEM encoder_map[][NUM_ENCODERS][NUM_DIRECTIONS] = {
    [0] = {ENCODER_CCW_CW(KC_VOLD, KC_VOLU)},
};
#endif
```

### How it works

Because the far side of the switch is ground (through C), pressing it
pulls the dedicated row low during **every** column strobe. Define
exactly one layout position on that row — the column index is arbitrary
and no extra column pin is needed; every other position on the row has
no keycode (`KC_NO`) and is ignored. The push switch then behaves as a
completely normal key: debounced by the stock matrix code and
remappable in VIA / Remap / Vial, without a single line of custom C.

### Trade-offs

- Keep the dedicated row exclusive. Any other key wired to it would
  register together with the encoder push — use one row per direct
  switch, at a cost of one GPIO each.
- Matrix testers will show the whole row active while pressed
  (cosmetic; the keymap only fires the one defined position).
- The extra row slightly grows the dynamic keymap storage
  (`columns × layers × 2` bytes).

## License

Distributed under the
[CERN Open Hardware Licence Version 2 — Permissive](LICENSE)
(CERN-OHL-P-2.0). Provided as-is, without warranty of any kind; verify
the footprint against your actual parts before ordering PCBs.

---

## 日本語

メカニカルキーボードで広く使われる Cherry MX 互換スイッチと取り付け寸法に
互換性のある、プッシュスイッチ付きロータリーエンコーダ用の KiCad シンボル・
フットプリントです。

Cherry MX スイッチのフットプリント(ホットスワップ互換)の上に重ねて配置
できるため、同じ基板の同じ位置でキースイッチとロータリーエンコーダを選択
できます。

### 対象部品

Skyloong 製ロータリーエンコーダ(encoder key switch)向けです。AliExpress
などで入手できます。

- 参考: [Skyloong_Components — encoder_key_switch](https://github.com/JZ-Skyloong/Skyloong_Components/tree/main)
- データシート: [encoder_key_switch_V1_05092023.pdf](https://github.com/JZ-Skyloong/Skyloong_Components/blob/main/encoder_key_switch/encoder_key_switch_V1_05092023.pdf)

本フットプリントは実測に基づいて独自に作成したもので、Skyloong とは
無関係です(非公式)。

### 収録内容と導入方法

KiCad 8.0 以降に対応しています。

1. このリポジトリをクローンまたはダウンロードします。
2. **シンボル**: 「設定 → シンボルライブラリを管理…」で
   `1U_Rotary_Encoder_Switch.kicad_sym` を追加します。
3. **フットプリント**: 「設定 → フットプリントライブラリを管理…」で
   `1U_Rotary_Encoder_Switch.pretty` フォルダを追加します。

既定のライブラリニックネーム(`1U_Rotary_Encoder_Switch`)のままなら、
シンボルのフットプリント指定は自動で解決されます。

### ピン対応

| パッド | シンボルピン | 機能 |
|---|---|---|
| 1 | B | エンコーダ B 相 |
| 2 | A | エンコーダ A 相 |
| 3 | E | プッシュスイッチ接点 |
| 4 | C | コモン |

シンボルの結線どおり、プッシュスイッチは **C**–**E** 間で閉じます。
A/B 相はコモン **C** 基準の直交(クアドラチャ)出力です。電気的な詳細は
データシートを参照してください。

### 設計メモ

- 中央の穴(4.1 mm 非メッキ)にエンコーダのシャフトが入り、4つの円形
  SMD パッドに表面側で足をはんだ付けします。
- `Dwgs.User` レイヤーに 1U キー領域(19.05 mm 角)と 14 mm 本体のコーナー
  マークを描いてあり、MX スイッチフットプリントとの位置合わせに使えます。
- MX(ホットスワップ)フットプリントに重ねて使う前提のため、シルクは
  意図的に空にしています。
- 基板製造時は ENIG(無電解ニッケル/金フラッシュ)表面処理を推奨します。

### ファームウェア実装例(QMK)

このエンコーダの押し込みスイッチはコモン端子 **C** を回転接点と共有して
おり、QMK のエンコーダドライバは C を GND に接続することを要求します。
そのため押し込みスイッチを通常の「ダイオード+行×列の交点」としてキー
マトリクスに組み込むことはできません(スイッチの片側が常に GND のため)。

これを回避するカスタムコード不要のパターン(上の写真の OLSK60 で実証済み)
が、「**E** をアクティブ Low のダイレクト入力として検出しつつ、**専用行**
としてマトリクススキャンに取り込む」方法です。QMK 公式ドキュメントは回転
のみを扱っており、このケースのレシピはありません。代替手段の `DIRECT_PINS`
(基板全体が対象になる)やカスタム `matrix_scan` コードは、いずれもこの
パターンより大掛かりになります。

#### 配線

| エンコーダパッド | 接続先 |
|---|---|
| 2 (A) / 1 (B) | MCU ピン → `pin_a` / `pin_b`(入れ替えると回転方向が反転) |
| 4 (C) | GND |
| 3 (E) | マトリクスの**専用行の入力ピン** |

設定は英語セクションの `keyboard.json` / `encoder_map` の例をそのまま
使えます。最後の行(例では `GP23`)は押し込みスイッチ専用です。

#### 動作原理

スイッチの反対側が(C 経由で)GND のため、押すと専用行はどの列のスト
ローブ中でも Low になります。その行にレイアウト位置を**1つだけ**定義して
ください(列番号は任意で、列ピンの追加も不要です)。行内の他の位置は
キーコード無し(`KC_NO`)として無視されます。これにより押し込みスイッチは
完全に通常のキーとして振る舞い、標準のマトリクスコードでデバウンスされ、
VIA / Remap / Vial で再割り当てできます。カスタム C コードは1行も不要です。

#### トレードオフ

- 専用行は専有を保つこと。同じ行に他のキーを配線すると押し込みと同時に
  反応します。ダイレクトスイッチ1個につき1行(=GPIO 1本)使います。
- マトリクステスターでは押下中に行全体が点灯します(表示上のみ。キーマップ
  が発火するのは定義した1位置だけです)。
- 行が増える分、ダイナミックキーマップ領域がわずかに増えます
  (列数 × レイヤー数 × 2 バイト)。

### ライセンス

[CERN Open Hardware Licence Version 2 — Permissive](LICENSE)
(CERN-OHL-P-2.0)で配布します。無保証です。基板発注前に実部品との
整合を必ず確認してください。
