# HANDOFF - 2026-07-16 00:52

## 現在の目標

- 承認済みSPECの全機能を`debris_flow_dam_simulator.html`単一ファイルへ実装し、TODOの直接証拠、独立再監査、`origin/main` pushまで完遂する。

## 完了済み

- [x] 初期構築、GitHub main push、独立監査マトリクス反映。
- [x] AppShell、Blob Worker、厳格CSV、pause/resume/cancel、自己試験基盤。
- [x] 標準1D、粒径別河床交換、活性層、保存的濃度上限射影、3種ダム。
- [x] 2D構造格子、保存的1D–2D共有界面、WebGL2/Canvas 2D描画。
- [x] GSI DEM固定allowlist取得、欠測decode、provenance、ローカル`grid_2d` CSV。

## 直前の検証証拠

- Nodeで3 script blockの構文検査合格。HTMLは約192 KiBで700 KiB目標内。
- Chromiumで自己試験37/37合格。共有界面単体の水・x運動量・粒径別保存誤差`1.11e-16`。
- 結合参照ケース: 1200 s、24,001 step、5 frame、最大CFL `0.354`、水誤差`4.38e-7`、土砂誤差`3.49e-8`。
- GSI DEM1A z17の256×256タイルを東京駅地点で取得し、欠測率0%、固定HTTPS host、出典表示を確認。

## 次にやること

1. D-Claw型単一混合速度の実験的五方程式モデルを実装・検証する。
2. 一因子・LHS・Morris、上限4 Worker pool、cancel、部分失敗、ensemble CSV/可視化を実装する。
3. 移動汀線・4格子・複数ブラウザ・狭幅・性能・file実行を検証する。
4. 独立監査タスクへ完成HTMLと証拠を送り、P0/P1を解消して全TODOを再監査する。

## 注意点

- 高度モデルは互換性を主張せず実験表示と適用限界を維持する。
- 保存的濃度射影は水量と粒径別固相を保持し、超過固相を河床へ移す。無言切捨てに戻さない。
- 検証用HTTPサーバーは127.0.0.1:8765、listen PID 34948で一時稼働中。最終完了前にPlaywright sessionとサーバーを停止確認する。
- 独立監査タスクIDは`019f6606-edf6-7fc0-8581-3697d75940a6`。
