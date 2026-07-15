# TODO - 実装・検証チェックリスト

## 0. 初期構築・文脈保持

- [x] 実装計画、主要機能、CSV限定、段階統合をユーザーと確定する。
- [x] 新規プロジェクト名、GitHub URL、`main`ブランチを確定する。
- [x] AGENTS、MEMORY、HANDOFF、PLAN、SPEC、TODO、KNOWLEDGEの初期内容を作成する。
- [ ] Git初期化、初回commit、`origin/main` pushを完了する。
- [ ] 独立監査タスクの初回検証マトリクスを受領し、KNOWLEDGEへ反映する。

## 1. 単一HTML基盤

- [ ] セマンティックなAppShell、3領域レイアウト、狭幅タブを実装する。
- [ ] CSS tokensと再利用可能なフォーム・ボタン・パネル・警告・表を実装する。
- [ ] AppStore、イベント配送、状態遷移、エラー境界を実装する。
- [ ] Workerソースを単一HTML内へ埋め込み、versioned protocolを実装する。
- [ ] Workerのpause/resume/cancelと古いrequest応答破棄を実装する。
- [ ] Canvas 2Dの縦断図・ハイドログラフ・収支図を実装する。
- [ ] 自己試験ランナーと検証結果パネルを実装する。

## 2. 入力不信・CSV

- [ ] RFC 4180 CSV parserを状態機械として実装する。
- [ ] CSV serializer、UTF-8 BOM、CRLF、式注入対策を実装する。
- [ ] schema/section/unit/ID/単調性/有限数/範囲/サイズ制限を検証する。
- [ ] UIとは独立したnormalize/validate/freeze層を実装する。
- [ ] Worker内の二重検証と推定メモリ・推定ステップ上限を実装する。
- [ ] Worker応答のprotocol/request/type/配列/物理範囲を検証する。
- [ ] ケースCSV round-tripと設定ハッシュ一致試験を実装する。
- [ ] malformed/巨大/重複/未知単位/NaN/Infinity/CSV injectionを拒否する試験を実装する。

## 3. 高精度1D

- [ ] Float64Arrayベースの1D stateと境界条件を実装する。
- [ ] hydrostatic reconstructionとwell-balanced sourceを実装する。
- [ ] HLLC系流束、MUSCL、制限関数、SSP-RK2を実装する。
- [ ] 正水深・濡れ乾き・CFL適応時間刻みを実装する。
- [ ] Manning摩擦と境界流入ハイドログラフを実装する。
- [ ] 2～8粒径の質量保存・侵食・堆積・活性層・分級を実装する。
- [ ] 平衡濃度型と降伏応力型堆積を実装する。
- [ ] 水・運動量・粒径別土砂・河床の収支診断を実装する。
- [ ] KANAko既定条件相当の参照プリセットを実装する。

## 4. 砂防ダム

- [ ] 複数ダムの配置・編集・検証を実装する。
- [ ] 不透過型の越流・堆砂を保存的に実装する。
- [ ] スリット型の越流・オリフィス・急縮損失を実装する。
- [ ] 格子型の粒径別通過・捕捉・Seed固定閉塞・再移動を実装する。
- [ ] 構造物単体と複数連続配置の収支試験を実装する。

## 5. 1D-2D結合・地形

- [ ] 2D構造格子state、流束、source、濡れ乾き、河床変動を実装する。
- [ ] 1D-2D共有界面の保存的流束交換を実装する。
- [ ] WebGL2描画とCanvas 2D fallbackを実装する。
- [ ] GSI tile座標、DEM RGB decode、dataset fallback、欠測検査を実装する。
- [ ] URL allowlist、tile上限、timeout、Abort、provenance、出典表示を実装する。
- [ ] ローカル`grid_2d` CSV地形とオンライン失敗時の明示fallbackを実装する。
- [ ] 1D-2Dの水・運動量・粒径別土砂の保存試験を実装する。

## 6. 高度二相流・感度分析

- [ ] 固相率、二速度、間隙水圧、ダイレイタンシーを持つ高度stateを実装する。
- [ ] 侵食取り込みと正値・上限制約を実装する。
- [ ] 高度モードの実験表示、出典、適用限界を実装する。
- [ ] Seed付きPRNGと再現試験を実装する。
- [ ] 一因子変化、Latin Hypercube、Morris法を実装する。
- [ ] Worker pool上限4、計算量見積り、cancel、部分失敗報告を実装する。
- [ ] ensemble CSVと感度・不確実性可視化を実装する。

## 7. 数値・UI・性能検証

- [ ] 静水保持と不規則河床well-balanced試験を合格させる。
- [ ] 乾床ダムブレーク、定常等流、移動汀線試験を合格させる。
- [ ] 1D閉領域の相対保存誤差`<=1e-8`を証明する。
- [ ] 2D・結合の相対保存誤差`<=1e-6`を証明する。
- [ ] 滑らかな製造解のL1収束次数1.7以上を証明する。
- [ ] 粒径別土砂、河床、各ダム、二相流の個別ベンチマークを合格させる。
- [ ] KANAko参照ケースの差分と理由を記録する。
- [ ] Chrome・Edge・Firefoxの機能・狭幅・キーボード・reduced motionを確認する。
- [ ] 初期表示、計算中fps、Worker通知、メモリ、ファイルサイズを計測する。
- [ ] 非圧縮700KiB目標を確認し、超過時は重複とデータ表現を最適化する。
- [ ] 開発サーバー・補助プロセスが残っていないことを確認する。

## 8. 独立監査・最終監査

- [ ] 別タスクへ完成HTMLと検証証拠の再監査を依頼する。
- [ ] 独立監査の重大・高優先度指摘を解消する。
- [ ] SPECの全要件とTODOを証拠付きで一件ずつ再監査する。
- [ ] KNOWLEDGE、MEMORY、HANDOFFを最終状態へ更新する。
- [ ] `main`を`origin/main`へpushし、cleanを確認する。
