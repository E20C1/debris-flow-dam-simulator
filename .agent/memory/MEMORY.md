# MEMORY

## プロジェクト概要

- `debris_flow_dam_simulator.html`だけで動く、土石流・砂防ダム統合シミュレーター。
- 標準1D、多粒径河床変動、3形式の複数砂防ダム、保存的1D–2D結合、実験的五方程式高度モード、一因子・LHS・Morris感度分析を統合済み。
- 外部必須依存はなく、`file://`でBlob Workerを起動できる。GSI DEMだけ任意のオンライン補助。
- 可搬入出力はRFC 4180 CSVに統一し、ケース設定hashとSeedを再現性の基準にする。

## 恒久的な設計判断

- 数値stateはWorker内`Float64Array`を正とし、main/UIとはversioned messageで分離する。
- 入力は`parse -> normalize -> validate -> freeze -> dispatch`、Worker内再検証、main側応答検証の三層境界を維持する。
- 標準1Dは`h,hu,hC_k`を保存し、水体積を`h-ΣhC_k`から導出する。水と混合物を二重保存しない。
- 乾床希薄波速を含むpositivity-preserving HLL、hydrostatic reconstruction、MUSCL、SSP-RK2を使用する。
- 384MiBは2D全frame履歴、全Worker、転送、描画を含むアプリ全体ピーク上限とする。
- KANAkoとの差を特殊処理で合わせない。canonical出力不在はN/Aとし、物理検証合格には使わない。

## 完了時の検証基線

- 独立監査タスク`019f6606-edf6-7fc0-8581-3697d75940a6`はcommit `ed1cad9`を再々監査し、P0/P1/P2なし・技術的リリース可と判定。
- Chrome・Edge・FirefoxとChrome `file://`で57/57、fail 0、console error/warning 0。Worker数値試験32/32。
- request/response交差型4件を全拒否し、正常4件を全受理。pause/resume/cancel実動作も確認。
- Ritter 4格子L1 `1.293e-3`・次数`0.980`、線形浅水PDE次数`1.623`、2D Ritter L1 `1.076e-2`。
- Manning正常水深・流量誤差`3.216e-8`、時間残差`1.56e-6/s`、dt半減差`0.09%`。
- 標準・結合・高度は各1200秒・24,001 step完走。HTMLは279,089 byteで700KiB以下。

## 明示する利用制約

- 高度モードは縮約実験モデルで、USGS D-Claw互換ではなく、現地・防災判断には使わない。
- GSI DEMは河道横断や構造物を保証しない。現地利用には測量・係数校正・観測比較が必要。
- KANAko canonical出力、公開実験による高度モデル校正、実地形での1D–2D校正は未実施。

## 運用

- 仕様の正は`.spec/SPEC.md`、完了証拠は`.spec/TODO.md`、詳細数値は`.spec/KNOWLEDGE.md`。
- 完了時点で監査用HTTPサーバー、Playwright session、Astro/Gradle副作用は残していない。
