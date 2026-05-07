# Coordinate and timestamp notes

T265のようなtracking deviceでは、sampleを取るだけでなく、timestampとcoordinate frameの扱いが重要になります。

このページでは、互換性や座標定義を断言するのではなく、実装・検証時に確認すべき点を整理します。

## Timestamp

Timestampは、単に表示用の値ではなく、同期・遅延・drop検出の基準になります。

| Checkpoint | 見る理由 |
|---|---|
| Timestamp unit | 秒、ミリ秒、マイクロ秒などの扱いを誤ると同期が崩れる |
| Device time vs host time | device側時刻とhost受信時刻は同じ意味ではない |
| Monotonicity | sample順序や停止を検出する |
| Wraparound possibility | 長時間運用で値が戻る可能性を考慮する |
| Out-of-order sample | queueや複数streamで順序が前後する可能性を見る |
| Sync threshold | どのdeltaまで同一時刻近傍として扱うかを決める |
| Accepted / skipped / delta metrics | syncerの状態を数値で観察する |

Queue delayとtimestamp delayも分けて考えます。Queue delayはconsumerが遅れている可能性を示し、timestamp delayはdevice sample時刻とhost処理時刻の差を示します。

## Syncer

Syncerは、完全一致するtimestampを探すものではなく、threshold内で近いsampleを採用する補助層として考えます。

| Metric | 意味 |
|---|---|
| Accepted ratio | threshold内で対応付けできた割合 |
| Skipped count | 対応付けに使わなかったsample数 |
| Timestamp delta | 採用したsample同士の時刻差 |
| Threshold | 許容する時刻差 |

Thresholdが狭すぎるとaccepted ratioが下がり、広すぎると対応付けの意味が弱くなります。Default値を固定の正解として扱わず、用途と実機ログで調整する前提にします。

## Coordinate frame

Poseをそのまま制御に使う前に、どのframeの値かを確認する必要があります。

| Frame / concept | 確認すべき点 |
|---|---|
| Tracking frame | 原点、軸方向、初期化時の向き |
| Camera body frame | device本体の向きとセンサー中心 |
| Robot base frame | ロボットの制御系が期待するframe |
| Mounting direction | 上下、前後、左右の取り付け向き |
| Center of tracking | poseがどの点の動きとして扱われるか |
| ROS integration | ROSへ渡す前のframe変換と命名 |
| Handedness | 右手系/左手系の前提 |

この公開資料では、特定のrobotやROS構成との互換性を断言しません。重要なのは、T265から得たposeを制御入力にする前に、mounting direction、axis convention、unit、originを明示的に確認することです。

## Practical checklist

| Item | Public note |
|---|---|
| Unit | position、velocity、timestampの単位を確認する |
| Axis | X/Y/Zの意味をdevice配置ごとに確認する |
| Origin | 起動時原点か、外部基準かを確認する |
| Transform | robot base frameへの変換を明示する |
| Delay | device timestampとhost処理時刻を分けて見る |
| Sync | accepted ratioとtimestamp deltaを記録する |
