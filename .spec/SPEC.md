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
- Worker数は最大4とする。384MiBは各Workerではなく、全Worker、main thread、転送中コピー、履歴、DEM decode、Canvas/WebGLを含むアプリ全体ピーク上限とする。
- 具体的な入力上限は、CSV 10MiB、250,000行、1 field 32KiB、1D 10,000セル、2D 262,144セル、8粒径、2,000出力frame、20,000,000総step、64 DEM tile、512 ensemble caseとする。さらに384MiB見積りを超える入力は個別上限内でも拒否する。
- エラー時は最後の検証済み状態を保持し、部分適用や暗黙の既定値置換を行わない。

## 4. CSV公開インターフェース

- ケースCSV列: `schema_version,section,id,parent_id,time_s,x_m,y_m,z_m,quantity,value,unit,text`
- section: `meta,solver,material,grain,cell_1d,inflow,structure,observer,grid_2d,boundary,uncertainty,provenance`
- 出力: `summary.csv`, `timeseries.csv`, `spatial.csv`, `mass_balance.csv`, `ensemble.csv`
- UTF-8 BOM、CRLF、RFC 4180 quoting、数値の`.`小数点、SI単位を使用する。
- ケースのCSV export -> importで、正規化済みケースの設定ハッシュが一致すること。
- 可搬ファイルとしてJSON、`.kanako`、独自バイナリを出力しない。

## 5. Workerプロトコル

- request: `init,run,pause,resume,cancel,validate`
- response: `ready,progress,frame,checkpoint,complete,error`
- request/response対応を固定し、`init -> ready`、`validate -> complete(tests必須)`、`run -> progress/frame/checkpoint/complete(summary必須)`、制御要求 -> 同名acknowledgement とする。不一致はcallback前に拒否する。
- すべてに`protocolVersion`, `requestId`, `type`を持たせる。
- 数値計算はWorker内の`Float64Array`を正とし、表示用スナップショットだけを転送する。
- 進捗通知は最大10Hzとし、計算時間刻みと表示用出力間隔を分離する。

## 6. 標準1Dモデル

- 標準1Dは単一混合速度の深さ平均モデルとし、単位幅あたりの保存変数を混合物体積深`h`、混合物運動量`hu`、粒径別移動固相体積`hC_k`とする。水と混合物を別々の独立質量として二重計上しない。
- 移動水体積は`h_w = h - Σ(hC_k)`、混合物密度は`rho_m = rho_f(1-ΣC_k)+rho_sΣC_k`から導出する。`0 <= ΣC_k <= C_max < 1`を全stepで保証する。
- 河床粒径別固相体積は`(1-porosity)(z_b-z_fixed)F_k`として移動固相と別に保存し、`ΣF_k=1`を保証する。侵食・堆積は移動固相と活性層の等値逆符号交換とする。
- 原始変数は`h,u,C_k,z_b,F_k`、運動量流束は`hu^2 + g h^2/2`を基礎とする。適用範囲は単一混合速度近似が成立する石礫型・未発達土石流および掃流状態とし、相間すべりが支配的なケースは高度モードへ分離する。
- 有限体積法、hydrostatic reconstruction、乾床希薄波速を考慮したpositivity-preserving HLL流束、MUSCL、制限関数、SSP-RK2を採用する。
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

- 高度モードはD-Claw型の単一混合速度・五方程式系へ統一し、固相・液相の独立速度を持つ二速度モデルとは混成しない。
- 保存・発展量は混合物質量、2方向混合物運動量、固相体積、基底間隙水圧とし、ダイレイタンシー、液化・排水、侵食取り込みを結合する。
- 固相率ゼロで標準浅水流へ退化すること、相間すべりを直接解かない適用限界をUIへ明示する。
- 公開D-Claw理論を参考にするが、USGS実装との互換を称さず「実験・未検証」を既定表示とする。
- 一因子変化、Latin Hypercube、Morrisスクリーニングを実装する。
- 乱数Seedを必須とし、同一ケース・同一Seedで結果が再現すること。

## 10. 数値検証と合格基準

- 静水保持は完全湿潤と複数乾燥島、乾床ダムブレークはRitter解析解との4格子比較、定常等流はManning正常水深・流量と追加stepの正規化時間残差、移動汀線はwet mask変化と保存を自己試験に含める。Manning時間残差は2刻みで評価し、最大`2e-6/s`、刻み半減差5%以下とする。
- 粒径別土砂保存、河床変動、ダム越流・スリット・格子閉塞、1D-2D境界、二相流を個別試験する。
- 閉領域相対保存誤差: 1D `<= 1e-8`、2Dと結合 `<= 1e-6`。
- 全ステップで水深、固相率、濃度を有限・非負・物理上限内に保つ。
- MUSCL界面再構成のL1収束次数を1.7以上、線形浅水定在波のPDE解は3格子で次数1.5以上かつ正規化L1 1%以下とする。この2試験を同一の「製造解」と呼ばない。
- 2D単体は不規則地形静水と横断一様Ritter解析解を直接比較し、1D–2D結合は順流・逆流・斜交流入と共有界面の直接保存を試験する。
- ダム限流は構造物なし、堰公式、高天端遮断の極端条件を式と直接比較する。
- LHSは各入力軸の層化、Seed再現、64→128 caseの設計分位収束を検査する。校正済み確率分布ではないため、出力分位を確率予測とは称さない。
- KANAko参照ケースは対応するcanonical出力が入手できた場合にピーク流量・到達時刻5%以内、最大河床変化10%以内を比較目標とする。現状はN/Aとし、KANAko一致を物理妥当性の合格条件にしない。
- 高度モードは公開実験との校正・検証が未完了のため「実験・未検証」の範囲でのみリリースし、現地予測の合格ゲートには使用しない。
- 差が閾値を超える場合は結果合わせをせず、モデル差と未証明事項を表示する。

## 11. 完了条件

- `.spec/TODO.md` の全項目に直接証拠があり、チェック済みである。
- 独立監査タスクの重大指摘を解消または明示的な未対応理由付きで仕様へ反映している。
- Chrome、Edge、FirefoxでCSV往復、実行、停止、失敗復旧、狭幅表示を確認している。
- 開発用サーバーや不要プロセスが残っていない。
- `main`と`origin/main`が一致し、作業ツリーがcleanである。
