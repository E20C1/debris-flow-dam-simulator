# HANDOFF - 2026-07-16 00:12

## 現在の目標

- 承認済みSPECの全機能を`debris_flow_dam_simulator.html`単一ファイルへ実装し、TODOの直接証拠、独立再監査、`origin/main` pushまで完遂する。

## 完了済み

- [x] 初期構築、GitHub main push、独立監査マトリクス反映。
- [x] AppShell、CSS tokens、Store、状態遷移、Canvas、Blob Worker protocol 1。
- [x] pause/resume/cancel、active request IDによる古い応答破棄。
- [x] RFC 4180 CSV、BOM/CRLF、式注入対策、厳格な必須項目・単位・上限・有限性検証。
- [x] `h,hu,hC_k`標準1D、HLLC系流束、MUSCL、hydrostatic reconstruction、SSP-RK2、CFL、摩擦、粒径別河床交換、3種ダムの初期カーネル。

## 直前の検証証拠

- Nodeで3 script blockの構文検査合格。HTML 139,940 byte。
- Chromiumで自己試験17/17合格。
- 参照ケース: 1200 s、24,001 step、243 frame、最大CFL 0.383、水誤差`1.30e-6`、土砂誤差`3.85e-8`。
- 長時間制御試験: 9,100 stepでpause、resume後23,600 stepでcancel。停止時の水誤差`1.48e-13`、土砂誤差`1.29e-12`。

## 次にやること

1. 1Dの閉領域・静水・乾床・等流・収束・構造物単体ベンチマークをWorker自己試験へ追加し、未証明TODOだけを判定する。
2. 2D構造格子、単一評価の1D–2D共有界面、2D frame転送・描画を実装する。
3. GSI/ローカル地形、WebGL2 fallback、高度五方程式、感度分析・ensemble CSVを実装する。
4. Chrome/Edge/Firefox、狭幅、性能、ファイルI/Oを監査し、別タスクへ完成HTMLを再監査依頼する。

## 注意点

- 参照ケースは開境界なので、閉領域`<=1e-8`の合格証拠として使わない。
- 濃度超過は無言縮小せず、侵食容量制限後も超過する場合は計算停止する。
- 検証用HTTPサーバーは127.0.0.1:8765で一時稼働中。作業完了前にPlaywright sessionとPID 2064/実listen PID 34948を停止確認する。
- 独立監査タスクIDは`019f6606-edf6-7fc0-8581-3697d75940a6`。
