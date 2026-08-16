# zmk-config-kb02 — keyboard#02 検証用 ZMK 設定（雛形）

infused-kim の [サンプル zmk-config](https://github.com/infused-kim/kb_zmk_ps2_mouse_trackpoint_driver-zmk_config)
（`corne_tp`）を正として、keyboard#02 のベンチ／split 検証用に書き直したもの。2026-08-17 作成。

## 使い方（GitHub Actions でビルド）

1. GitHub に空リポジトリ `zmk-config-kb02` を作り、このフォルダの中身をそのまま push
   （`.github/workflows/build.yml` を含む）。
2. Actions タブでビルドが走る → Artifacts から `.uf2` を取得。
3. **最初のビルド＝検証 A**。赤なら `config/west.yml` の ZMK 参照先（main ↔ infused-kim fork）を切り替える。

## シールド一覧

| シールド | ボード | 用途 |
|---|---|---|
| `kb02bench_xiao` | `seeeduino_xiao_ble` | **単体（非 split）**：XIAO Plus ＋ SK8707-01 ＋ 2×2 仮マトリクス。TP を USB で確認（検証 B/D） |
| `kb02bench_sm` | `nice_nano_v2` | 単体：SuperMini ＋ SK8707-01。**BLE 無効で USB のみ**（技適回避）。XIAO 到着前の事前ベンチ |
| `kb02test_left` / `kb02test_right` | `seeeduino_xiao_ble` | **split**：右＝central＋TP、左＝peripheral、各 2×2（検証 C/E/F） |
| `settings_reset` | 両ボード | ペアリング情報の消去 |

## ピン（生 `&gpioX N` 指定。ボード差し替えは数字だけ）

| 信号 | XIAO Plus（ベンチ／split） | SuperMini（ベンチ） |
|---|---|---|
| TP DATA (SDA, UART RX) | P1.12 (D7)　※最終は P0.15 (D11) | P0.17 (D2) |
| TP CLK (SCL) | P1.11 (D6)　※最終は P0.19 (D12) | P0.20 (D3) |
| TP RST | P0.28 (D2) | P1.06 (D9) |
| ROW0 / ROW1 | P0.02 (D0) / P0.03 (D1) | P1.00 (D6) / P0.11 (D7) |
| COL0 / COL1 | P0.04 (D4) / P0.05 (D5) | P1.13 (D15) / P1.15 (D18) |
| UART 未露出ピン TX/RX | P0.11 / P0.12（要確認：XIAO 回路図で未使用か。Sense 版で使う P0.27/P0.16/P0.07/P1.00 は避けた） | P0.27 / P0.28（サンプルどおり） |

XIAO で最終割当（P0.15/P0.19）を試すときは `kb02bench_xiao.overlay` の `MOUSE_PS2_PIN_*` を書き換えるだけ。

## 重要な注意

- ドライバは `zmk,input-listener-ps2` と `behaviors/mouse_keys.dtsi` を前提にしている（＝infused-kim の ZMK fork）。
  `config/west.yml` は **まず fork（既知で動く）** を指す。ZMK main で通るかは 2 回目に試す（west.yml のコメント参照）。
- keymap は `#define HAS_MOUSE_TP` を外せば TP 無しでもビルドできる構造（サンプル踏襲）。
- 割り込み優先度の上書き（`config/include/tp_irq_priorities.dtsi`）は nRF52840 共通なので XIAO でもそのまま。
