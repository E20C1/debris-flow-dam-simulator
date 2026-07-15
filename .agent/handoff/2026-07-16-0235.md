# HANDOFF - 2026-07-16 01:28

## 現在の目標

- 承認済みSPECの全機能を`debris_flow_dam_simulator.html`単一ファイルへ実装し、TODOの直接証拠、独立再監査、`origin/main` pushまで完遂する。

## 完了済み

- [x] 初期構築、GitHub main push、独立監査マトリクス反映。
- [x] AppShell、Blob Worker、厳格CSV、pause/resume/cancel、自己試験基盤。
- [x] 標準1D、粒径別河床交換、活性層、保存的濃度上限射影、3種ダム。
- [x] 2D構造格子、保存的1D–2D共有界面、WebGL2/Canvas 2D描画。
- [x] GSI DEM固定allowlist取得、欠測decode、provenance、ローカル`grid_2d` CSV。
- [x] 実験的`h,hu,hv,hm,p_b`五方程式、ダイレイタンシー、間隙水圧、侵食取り込み。
- [x] 一因子・LHS・Morris、最大4 Worker、cancel、部分結果、ensemble CSV/Canvas。
- [x] 46自己試験、3ブラウザ、狭幅、file直接起動、性能・a11y監査、4結果CSV。

## 直前の検証証拠

- Nodeで3 script blockの構文検査合格。HTML 251,644 byte（700KiBの35.1%）。
- Chrome・Edge・Firefoxで自己試験46/46、狭幅overflow 0、reduced motion、キーボード、停止後再実行、console error 0。
- 標準/結合/高度はいずれも参照1200 s・約24,001 stepを完走。結合水誤差`4.38e-7`、土砂`3.49e-8`。
- LHS 4/4、Morris 5/5、cancel 8/8状態保持。全5種の結果CSVをBOM/CRLF付きで実機保存した。
- Chrome boot 61ms、main heap peak 7.85MB、計算中rAF最大12.7ms、243 frame。a11y可視interactive 54要素は全て名前あり。

## 次にやること

1. checkpoint commit/push後、独立監査タスクへ完成HTMLと全証拠を送り再監査を依頼する。
2. 重大・高優先度指摘を修正し、SPEC/TODOを一件ずつ再監査する。
3. MEMORY/HANDOFF/KNOWLEDGEを最終更新し、main push・cleanを確認する。
4. 全Playwright sessionとHTTP server PIDを停止し、TODO最後の後片付けを確認する。

## 注意点

- 高度モデルはUSGS D-Claw互換を主張せず実験表示と適用限界を維持する。
- 保存的濃度射影は水量と粒径別固相を保持し、超過固相を河床へ移す。無言切捨てに戻さない。
- 検証用HTTPサーバーは127.0.0.1:8765、listen PID 34948で一時稼働中。Playwright sessionはdfds/edge/firefox/filetest/audit。最終完了前に全停止確認する。
- 独立監査タスクIDは`019f6606-edf6-7fc0-8581-3697d75940a6`。
