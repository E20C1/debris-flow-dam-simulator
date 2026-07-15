# KNOWLEDGE - ドメイン知識・調査結果

## 既知の参照資料

- KANAko公式ページ・詳細マニュアル・2007年開発論文を1Dモデルとダムモデルの公開根拠にする。
- 2008年・2014年のダム上流堆積研究を、低流速域の降伏応力型堆積の根拠にする。
- KANAko 2D論文を、渓流1Dと扇状地2Dの結合方式の比較対象にする。
- USGS D-Claw公開理論を高度二相流の参考にするが、互換性を主張しない。
- 国土地理院の標高タイル詳細仕様と利用規約をDEM decode・provenance・出典の正とする。

## 初期技術判断

- 単一HTMLでもWorkerをBlobとして生成すればUIと計算を分離できる。
- SharedArrayBufferはCOOP/COEPを必要とするため必須にせず、Transferable ArrayBufferを使う。
- 主要計算はCanvas/WebGL描画から独立させ、表示API非対応でも数値結果を得られるようにする。
- 完全クライアントアプリにサーバー権威はない。したがって、UI入力を正規化・凍結し、Workerで再検証し、main側で応答も検証する多層境界を採用する。
- KANAkoとの数値差を埋める特殊処理は禁止し、旧モデルと現代化モデルの差として説明する。

## 未証明事項

- HLLC系流束と土石流構成則の組合せで、すべての湿潤・乾燥・高濃度ケースが安定するか。
- 1D-2D界面で粒径別土砂と河床変動まで所定の保存誤差に収まるか。
- 高度二相流を単一HTMLの性能・容量目標内で実用的に計算できるか。
- KANAko参照ケースで比較目標を満たすか。
- これらは実装後の直接ベンチマークなしに完了扱いしない。

## 2026-07-15 独立監査タスクの結論

- 別Codexタスク `019f6606-edf6-7fc0-8581-3697d75940a6` は、一次資料に基づく検証マトリクス作成という目標を完了した。
- 現時点の実装完了判定は、HTML・試験証拠が存在しないため不合格。これは想定どおりの初期状態であり、実装後に同タスクへ再監査を依頼する。
- P0指摘1: 標準1Dの「水・混合物質量」は二重計上の危険がある。単一混合速度の`h,hu,hC_k`を保存し、水体積を`h-ΣhC_k`から導出する仕様へ修正した。
- P0指摘2: D-Clawは固液二速度を独立に解かない。高度モードをD-Claw型の単一混合速度・五方程式系へ統一した。
- P1指摘: 384MiBをアプリ全体ピーク上限と明記し、CSV、格子、frame、step、tile、ensembleの具体的上限をSPECへ追加した。
- KANAko一致は解析解・実験一致の代用にしない。KANAko、解析解、独立実験を別々に評価する。
- 静水保持は完全湿潤・孤立乾床・複数乾湿境界、ダムブレークは4格子、結合は逆流・斜交流入・非一致幅まで検証対象とする。
- CSVは`=,+,-,@,tab,CR,LF`始まりのtext fieldを安全化し、GSI欠測RGB `(128,0,0)`を標高0として扱わない。

## 2026-07-16 基盤・CSV・標準1D初回実装

- 単一HTMLは初回統合時点で139,940 byte。外部必須依存なしで、700KiB目標に十分な余裕がある。
- Chromium実ブラウザでAppShell、Canvas、Blob Worker、protocol 1、CSV境界を確認した。自己試験はRFC 4180往復、式注入、NaN/Infinity、未知section/unit、必須不足、32KiB field、10MiB file、古い応答、非有限bufferを含む17/17合格。
- pauseは9,100 stepで停止状態を保持し、resume後23,600 stepでcancelを受理した。古いrun応答はactive request IDで破棄する。
- ダム捕捉は移動土砂と構造物貯留を二重計上せず、上流の混合物体積・固相・運動量から等値交換する。
- RK2で河床割合を直接平均すると粒径別固相が保存されない。粒径別河床固相体積を平均してから割合を再導出することで、参照開境界ケースの土砂相対誤差を`1.52e-3`から`3.85e-8`へ改善した。
- 同じ参照ケースの1200 s計算は24,001 step、243 frame、最大CFL 0.383。水相対誤差`1.30e-6`、土砂相対誤差`3.85e-8`。開境界かつ期待残量が小さい終端なので、閉領域合格基準の証拠には使用しない。

## 2026-07-16 1D-2D結合・GSI地形

- 2D構造格子は`h,qx,qy,hC_k`と粒径別河床量をFloat64Arrayで保持し、閉外周・濡れ乾き・摩擦・河床交換をSSP-RK2で更新する。
- 1D末端と2D流入口は界面流束をstepごとに一度だけ評価し、異なる面幅を考慮して符号反転した同一体積・x運動量・粒径別固相を両側へ適用する。順流・逆流・斜交流入・非一致幅試験に加え、界面単独の保存誤差は`1.11e-16`だった。
- 参照ケースを24×18格子へ結合した1200 s実行は24,001 stepで完了し、最大CFL `0.354`、水相対誤差`4.38e-7`、土砂相対誤差`3.49e-8`で結合基準`1e-6`を満たした。
- 固相濃度の不変領域を越える輸送丸めは、超過固相を粒径比どおり河床へ移し、混合物深も同量減らす保存的射影で処理する。水量と粒径別全固相の単体保存誤差は0だった。
- GSI標高タイルは固定HTTPS host・dataset allowlist・1 tile上限・Abort timeoutを使う。公式RGB仕様の0 m、欠測`(128,0,0)`、負標高を試験し、東京駅地点でDEM1A z17の256×256タイルを実ブラウザ取得、欠測率0%と出典表示を確認した。
- 2D地形を含むケースCSVはenabled、格子形状、全標高値、正規化hashがround-trip後に一致した。オンライン取得失敗時は現在地形を変更せず、ローカル`grid_2d` CSVを案内する。

## 2026-07-16 実験的五方程式・感度分析

- USGS D-Claw理論 `https://claw.code-pages.usgs.gov/dclaw/src/theory.html` とGeorge & Iverson (2014) `https://pubs.usgs.gov/publication/70170254` を再確認した。実装は`h,hu,hv,hm,p_b`、単一混合速度、固相率、間隙水圧、ダイレイタンシー、排水、侵食取り込みを持つ縮約実験モデルであり、USGS実装互換ではない。
- 固相率0で高度流束と標準HLLCの質量・運動量流束差は0。濃厚状態の正値・上限、間隙水圧、同一Seed完全一致、侵食取り込みの試験は水・全固相保存誤差`2.31e-16`以下で合格した。
- UI実行では実験警告と適用限界を表示したまま、参照ケース1200 s・24,001 stepを完走した。最大CFL`0.376`、内部sourceを含む水収支誤差`4.28e-6`、固相収支誤差`2.28e-8`だった。
- 一因子変化は基準点から一軸のみ、LHSは各パラメータの全層を一度ずつ、Morrisはtrajectory内で一軸ずつ変化させる。全手法はSeed固定で同じsampling designを再生成する。
- Worker poolはrequest IDを`workerIndex:requestId`名前空間で管理し、最大4かつ総推定ピーク384MiB以内に制限する。LHS 4 caseは4/4、Morris 5 caseは5/5成功した。
- 8 case実行中のcancelでは実行中4件と未開始4件を区別して保持した。各caseの失敗も他caseを止めずCSVのstatus/errorへ残す。
- `ensemble.csv`はUTF-8 BOM、CRLF、式注入対策済みserializerを使用し、パラメータJSON、収支、CFL、step、ピーク流量、Morris素効果を出力する。Canvasは同じ結果モデルのピーク流量を描画する。

## 2026-07-16 数値・ブラウザ・性能監査

- Chromium実ブラウザの統合自己試験は46/46合格。不規則河床静水誤差`2.61e-16`、4格子乾床ダムブレーク最大保存誤差`3.55e-16`、移動汀線保存誤差0かつ1 cellのwet mask変化、MUSCL製造解L1次数`2.03`を確認した。
- 結合参照ケースは水`4.38e-7`・土砂`3.49e-8`で`1e-6`基準内。高度モデル単体は水・固相`2.31e-16`以下。各粒径・活性層・3形式ダム・連続ダムも`1e-8`基準内だった。
- KANAko 1.44公式配布ページ `https://sabo-int.org/new-technology/kanako_en/kanako_jp/` と公開論文を確認したが、現在のクリーンルーム参照条件に対応するcanonical出力時系列は公開されていない。したがってピーク・到達・河床差分はN/Aと記録し、物理検証合格には使わない。差の要因はKanakoのstaggered有限差分に対する本実装のHLLC有限体積、分級、活性層、急縮損失のモデル差であり、結果合わせをしない。
- Chrome・Edge・FirefoxはいずれもWorker起動、自己試験46/46、390px幅overflow 0、3モバイルタブ切替、キーボードEnter、reduced-motion `1µs`を確認した。Edge・Firefoxは停止後に24,001 stepを再完走し、console error/warningは0だった。
- `file:`直接起動はChrome headlessで`data-js=enabled`、Blob Worker `ready / protocol 1`、初期化ログあり、利用不可stateなしを確認した。
- Chrome計測はDOMContentLoaded `38.9ms`、load `39.9ms`、Worker readyまで`61ms`。参照計算は`11.63s`、rAF平均`6.50ms`、最大`12.7ms`、243 Worker frame、main heap peak`7,845,689 byte`。同時4 Worker見積りは`39.5MiB`で384MiB内。
- 初期ネットワークはHTML documentと同一HTMLから生成したBlob Workerの2 requestのみで、外部必須resource・依存chainはない。GSIはユーザー操作時だけ取得する。
- a11y検査は可視interactive 54要素すべてに名前あり、重複IDなし、h1→h2順序、focus-visibleあり。最小文字色`--faint`を`#788c86`へ上げ、panel背景との比を`4.70:1`にした。
- Chrome DevTools MCPが未設定のため`web-perf`スキル固有のLCP/CLS traceは未実施。Playwright Performance/Network/DOM APIを代替証拠にし、未実施点を隠さない。
- HTMLは結果CSV追加後も251,644 byteで、非圧縮700KiB上限の35.1%。外部framework・bundler・画像・fontはない。
- 5 frame参照計算から`summary.csv` 1行、`timeseries.csv` 5行、`spatial.csv` 155行、`mass_balance.csv` 5行を出力し、全てUTF-8 BOM・CRLFを確認した。`ensemble.csv`を含む全出力は250,000行上限と式注入対策を共有する。
