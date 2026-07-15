# TODO - 実装・検証チェックリスト

## 0. 初期構築・文脈保持

- [x] 実装計画、主要機能、CSV限定、段階統合をユーザーと確定する。
- [x] 新規プロジェクト名、GitHub URL、`main`ブランチを確定する。
- [x] AGENTS、MEMORY、HANDOFF、PLAN、SPEC、TODO、KNOWLEDGEの初期内容を作成する。
- [x] Git初期化、初回commit、`origin/main` pushを完了する。
- [x] 独立監査タスクの初回検証マトリクスを受領し、KNOWLEDGEとSPECへ反映する。

## 1. 単一HTML基盤

- [x] セマンティックなAppShell、3領域レイアウト、狭幅タブを実装する。
- [x] CSS tokensと再利用可能なフォーム・ボタン・パネル・警告・表を実装する。
- [x] AppStore、イベント配送、状態遷移、エラー境界を実装する。
- [x] Workerソースを単一HTML内へ埋め込み、versioned protocolを実装する。
- [x] Workerのpause/resume/cancelと古いrequest応答破棄を実装する。
- [x] Canvas 2Dの縦断図・ハイドログラフ・収支図を実装する。
- [x] 自己試験ランナーと検証結果パネルを実装する。

## 2. 入力不信・CSV

- [x] RFC 4180 CSV parserを状態機械として実装する。
- [x] CSV serializer、UTF-8 BOM、CRLF、式注入対策を実装する。
- [x] schema/section/unit/ID/単調性/有限数/範囲/サイズ制限を検証する。
- [x] UIとは独立したnormalize/validate/freeze層を実装する。
- [x] Worker内の二重検証と推定メモリ・推定ステップ上限を実装する。
- [x] Worker応答のprotocol/request/type/配列/物理範囲を検証する。
- [x] ケースCSV round-tripと設定ハッシュ一致試験を実装する。
- [x] malformed/巨大/重複/未知単位/NaN/Infinity/CSV injectionを拒否する試験を実装する。
- [x] `summary.csv`、`timeseries.csv`、`spatial.csv`、`mass_balance.csv`を共通の安全なCSV serializerで出力する。

## 3. 高精度1D

- [x] `h,hu,hC_k`を保存し水体積を導出するFloat64Arrayベースの1D stateと境界条件を実装する。
- [x] hydrostatic reconstructionとwell-balanced sourceを実装する。
- [x] positivity-preserving HLL流束、MUSCL、制限関数、SSP-RK2を実装する。
- [x] 正水深・濡れ乾き・CFL適応時間刻みを実装する。
- [x] Manning摩擦と境界流入ハイドログラフを実装する。
- [x] 2～8粒径の質量保存・侵食・堆積・活性層・分級を実装する。
- [x] 平衡濃度型と降伏応力型堆積を実装する。
- [x] 水・運動量・粒径別土砂・河床の収支診断を実装する。
- [x] KANAko既定条件相当の参照プリセットを実装する。

## 4. 砂防ダム

- [x] 複数ダムの配置・編集・検証を実装する。
- [x] 不透過型の越流・堆砂を保存的に実装する。
- [x] スリット型の越流・オリフィス・急縮損失を実装する。
- [x] 格子型の粒径別通過・捕捉・Seed固定閉塞・再移動を実装する。
- [x] 構造物単体と複数連続配置の収支試験を実装する。

## 5. 1D-2D結合・地形

- [x] 2D構造格子state、流束、source、濡れ乾き、河床変動を実装する。
- [x] 1D-2D共有界面の保存的流束交換を実装する。
- [x] WebGL2描画とCanvas 2D fallbackを実装する。
- [x] GSI tile座標、DEM RGB decode、dataset fallback、欠測検査を実装する。
- [x] URL allowlist、tile上限、timeout、Abort、provenance、出典表示を実装する。
- [x] ローカル`grid_2d` CSV地形とオンライン失敗時の明示fallbackを実装する。
- [x] 1D-2Dの水・運動量・粒径別土砂の保存試験を実装する。

## 6. 高度二相流・感度分析

- [x] D-Claw型単一混合速度、固相率、間隙水圧、ダイレイタンシーを持つ五方程式高度stateを実装する。
- [x] 侵食取り込みと正値・上限制約を実装する。
- [x] 高度モードの実験表示、出典、適用限界を実装する。
- [x] Seed付きPRNGと再現試験を実装する。
- [x] 一因子変化、Latin Hypercube、Morris法を実装する。
- [x] Worker pool上限4、計算量見積り、cancel、部分失敗報告を実装する。
- [x] ensemble CSVと感度・不確実性可視化を実装する。

## 7. 数値・UI・性能検証

- [x] 静水保持と不規則河床well-balanced試験を合格させる。
- [x] 乾床ダムブレーク、定常等流、移動汀線試験を合格させる。
- [x] 1D閉領域の相対保存誤差`<=1e-8`を証明する。
- [x] 2D・結合の相対保存誤差`<=1e-6`を証明する。
- [x] MUSCL界面再構成次数1.7以上と線形浅水定在波の3格子収束を別々に証明する。
- [x] 2D不規則地形静水と横断一様Ritter解析解の直接ベンチマークを合格させる。
- [x] 複数乾燥島、ダム限流の極端条件、LHS設計分位の倍増収束を試験する。
- [x] 粒径別土砂、河床、各ダム、二相流の個別ベンチマークを合格させる。
- [x] KANAko参照ケースの差分と理由を記録する。
- [x] Chrome・Edge・Firefoxの機能・狭幅・キーボード・reduced motionを確認する。
- [x] 初期表示、計算中fps、Worker通知、メモリ、ファイルサイズを計測する。
- [x] 非圧縮700KiB目標を確認し、超過時は重複とデータ表現を最適化する。
- [x] 開発サーバー・補助プロセスが残っていないことを確認する。

## 8. 独立監査・最終監査

- [x] 別タスクへ完成HTMLと検証証拠の再監査を依頼する。
- [x] 独立監査の重大・高優先度指摘を解消する。
- [x] SPECの全要件とTODOを証拠付きで一件ずつ再監査する。
- [x] KNOWLEDGE、MEMORY、HANDOFFを最終状態へ更新する。
- [x] `main`を`origin/main`へpushし、cleanを確認する。
