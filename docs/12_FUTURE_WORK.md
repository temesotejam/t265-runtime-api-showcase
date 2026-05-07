# Future work

今後このプロジェクトを続けるなら、次の方向が考えられます。

## 1. Public向け最小実装の再設計

現在の全実装はprivateに置く方針ですが、将来的に本当に公開するなら、最小実装を別に作るのがよいです。

条件:

- firmwareを含めない。
- 実機シリアルを含めない。
- 生ダンプを含めない。
- 低レベル詳細を出しすぎない。
- 依存関係を明確にする。
- ライセンス境界を確認する。

## 2. API安定度の分類

各APIを次のように分類すると、利用者に伝わりやすくなります。

| 分類 | 意味 |
|---|---|
| stable | 通常利用推奨 |
| experimental | 実用可能だが変更の可能性あり |
| internal | 利用者向けではない |
| probe | 検証用 |
| private | 公開しない |

## 3. long-run検証の自動化

手動確認だけではなく、長時間確認の結果を一定フォーマットで出すと良いです。

見る指標:

- runtime device count
- sample count
- queue dropped
- backlog max
- timeout count
- memory usage
- accepted ratio
- final status

## 4. image workerの安定化

画像保存や画像処理は、worker threadとして独立させるのがよいです。

今後見るべき点:

- save latency
- queue depth
- dropped count
- disk I/Oの影響
- owned imageのcopyコスト
- shutdown順序

## 5. syncerの評価を増やす

timestamp thresholdをいくつか変えて、用途ごとの推奨値を整理すると良いです。

見るべき点:

- strict thresholdでのdrop
- relaxed thresholdでのdelta増加
- pose / gyro / accel のそれぞれのズレ
- 長時間runでの安定性

## 6. D435など他センサとの接続設計

将来的に、T265 + D435 のような構成へ拡張する場合、次の点を考える必要があります。

- timestamp基準の違い
- 外部同期の有無
- frame alignment
- calibration file
- poseとdepth/imageの対応付け
- ROS / non-ROS の境界

## 7. ドキュメントだけの公開を育てる

この公開版はコードを出さない資料ですが、それでも育てる価値があります。

追加すると良いもの:

- 実機構成図
- 失敗パターン集
- FAQ
- API設計の変遷
- 「最初に見るべき3項目」
- 面接用1ページ要約

## 最後に

今すぐ全部を公開する必要はありません。

Privateで実装を育てつつ、Publicでは設計思想と調査観点を磨く。この2段構成が、このプロジェクトには一番合っています。

