# SPEC - 技術仕様・要件定義

## 1. 成果物と実行条件

- 配布成果物はルートの `debris_flow_dam_simulator.html` 一つ。
- HTML、CSS、JavaScript、Workerソース、参照プリセット、自己試験を内包する。
- 外部フレームワーク、CDN、外部フォント、必須のサーバーAPIを使用しない。
- `file://`で標準1D、CSV、自己試験が動作する。国土地理院DEMのみオンライン補助とする。
- 非圧縮700KiB以下を目標とし、超過時は機能削減ではなく重複・データ表現・遅延初期化を見直す。

## 2. UIコンポーネント

- `AppShell`, `CommandBar`, `ScenarioEditor`, `LongitudinalViewport`, `Terrain2DViewport`
- `HydrographPanel`, `MassBalancePanel`, `Timeline`, `DamEditor`, `MaterialEditor`
- `ObservationEditor`, `CsvIO`, `ValidationPanel`, `UncertaintyPanel`
- デスクトップでは設定・可視化・診断の3領域、狭幅ではタブ切替とする。
- 動的文字列へ`innerHTML`を使用せず、Canvas描画と`textContent`を基本とする。
- 色だけに依存しない凡例、キーボード操作、十分なコントラスト、reduced motionを実装する。

## 3. 信頼境界と検証

- UI入力から直接計算を開始せず、`parse -> normalize -> validate -> freeze -> dispatch`を必須順序とする。
- CSV parserはRFC 4180、UTF-8、最大ファイルサイズ、最大行数、最大列長、有限数、既知section、既知unitを検査する。
- Workerはmain threadの検証済み主張を信用せず、schema version、配列長、有限性、単調性、範囲、推定メモリ、推定ステップを再検査する。
- Worker応答もrequest ID、message type、配列長、有限性、物理範囲を検査してから表示する。
- GSI URLは固定allowlistと既知のdataset IDだけを許可し、取得枚数、範囲、timeout、欠測率を制限する。
- Worker数は最大4、数値配列推定384MiB以下、総ステップ数は有限上限とし、開始前に見積りを表示する。
- エラー時は最後の検証済み状態を保持し、部分適用や暗黙の既定値置換を行わない。

## 4. CSV公開インターフェース

- ケースCSV列: `schema_version,section,id,parent_id,time_s,x_m,y_m,z_m,quantity,value,unit,text`
- section: `meta,solver,material,grain,cell_1d,inflow,structure,observer,grid_2d,boundary,uncertainty,provenance`
- 出力: `summary.csv`, `timeseries.csv`, `spatial.csv`, `mass_balance.csv`, `ensemble.csv`
- UTF-8 BOM、CRLF、RFC 4180 quoting、数値の`.`小数点、SI単位を使用する。
- ケースのCSV export -> importで、正規化済みケースの設定ハッシュが一致すること。
- 可搬ファイルとしてJSON、`.kanako`、独自バイナリを出力しない。

## 5. Workerプロトコル

- request: `init,run,pause,resume,cancel,validate,sweep`
- response: `ready,progress,frame,checkpoint,complete,error`
- すべてに`protocolVersion`, `requestId`, `type`を持たせる。
- 数値計算はWorker内の`Float64Array`を正とし、表示用スナップショットだけを転送する。
- 進捗通知は最大10Hz、描画は最大30fps。計算時間刻みと描画間隔を分離する。

## 6. 標準1Dモデル

- 保存変数は水・混合物の質量、運動量、粒径別土砂質量、河床高。
- 有限体積法、hydrostatic reconstruction、HLLC系流束、MUSCL、制限関数、SSP-RK2を採用する。
- well-balanced、正水深、濡れ乾き、CFL適応時間刻み、半陰的Manning摩擦を実装する。
- 2～8粒径、粒径別侵食・堆積、活性層交換、分級を質量保存付きで扱う。
- 平衡濃度型とダム上流低流速域向け降伏応力型の堆積を選択可能にする。
- 物理係数、単位、適用範囲、出典をUIで確認可能にする。

## 7. 砂防ダム

- 同一河道へ複数配置可能とする。
- 不透過型: 越流堰、堆砂容量、越流後の流束。
- スリット型: 越流・オリフィス・急縮損失。
- 格子型: 粒径別通過、捕捉、Seed固定の確率的閉塞、せん断による再移動。
- 構造物境界でも水量・運動量・粒径別土砂の収支を記録する。

## 8. 1D-2D結合

- 2Dは構造格子の有限体積法とし、1D出口と2D流入口で保存的な数値流束を一度だけ評価・共有する。
- 水、運動量、粒径別土砂を共通時間管理下で交換し、見た目だけの接続を禁止する。
- 2Dでもwell-balanced、正値性、濡れ乾き、河床変動、複数粒径を維持する。
- GSI DEM1A -> DEM5A -> DEM5B -> DEM5C -> DEM10Bの順で取得し、ローカル座標へ変換する。
- dataset ID、zoom、tile座標、取得日時、欠測補間、座標原点、出典をprovenanceへ保存する。
- 通信失敗時は`grid_2d` CSV入力へ明示的にフォールバックし、無断で粗いDEMへ置換しない。

## 9. 高度二相流と不確実性

- 固相率、固相・液相速度、間隙水圧、ダイレイタンシー、侵食取り込みを持つ深さ平均モデルとする。
- 公開D-Claw理論を参考にするが、互換を称さず「実験」と表示する。
- 一因子変化、Latin Hypercube、Morrisスクリーニングを実装する。
- 乱数Seedを必須とし、同一ケース・同一Seedで結果が再現すること。

## 10. 数値検証と合格基準

- 静水保持、不規則河床、乾床ダムブレーク、定常等流、移動汀線を自己試験に含める。
- 粒径別土砂保存、河床変動、ダム越流・スリット・格子閉塞、1D-2D境界、二相流を個別試験する。
- 閉領域相対保存誤差: 1D `<= 1e-8`、2Dと結合 `<= 1e-6`。
- 全ステップで水深、固相率、濃度を有限・非負・物理上限内に保つ。
- 滑らかな製造解のL1収束次数を1.7以上とする。
- KANAko参照ケースはピーク流量・到達時刻5%以内、最大河床変化10%以内を比較目標とする。
- 差が閾値を超える場合は結果合わせをせず、モデル差と未証明事項を表示する。

## 11. 完了条件

- `.spec/TODO.md` の全項目に直接証拠があり、チェック済みである。
- 独立監査タスクの重大指摘を解消または明示的な未対応理由付きで仕様へ反映している。
- Chrome、Edge、FirefoxでCSV往復、実行、停止、失敗復旧、狭幅表示を確認している。
- 開発用サーバーや不要プロセスが残っていない。
- `main`と`origin/main`が一致し、作業ツリーがcleanである。
