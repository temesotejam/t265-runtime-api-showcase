# Research map

このプロジェクトを再開するとき、または他の人が同じ方向で調査するときに見るべきポイントをまとめます。

## まず見るべき順番

最初から実装に入るより、次の順に確認すると迷いにくいです。

1. USB上でT265がどう見えているか。
2. runtime mode なのか bootloader mode なのか。
3. runtime device の descriptor / interface / endpoint の構造。
4. streamを開始するために必要な設定の流れ。
5. 受信されるsampleの種類と頻度。
6. timestamp の単位・基準・ばらつき。
7. queueが詰まる条件。
8. 画像payloadのlifetime。
9. 長時間動作時のtimeout / backlog / drop。

## 観察する対象

| 対象 | 見る理由 |
|---|---|
| USB vendor/product ID | 状態切り分けの入口になる |
| descriptor | interface / endpoint 構造を確認する |
| runtime stream | pose / motion / image の流れを見る |
| timestamp | syncer の設計に必要 |
| callback頻度 | queue設計に必要 |
| queue depth | backlogやdropを検出するため |
| image payload | ownership / copy方針を決めるため |
| serial number | 複数台識別に必要 |
| udev permission | sudoなし運用に必要 |

## 重要な識別子

T265では、少なくとも次の状態を区別して考える必要があります。

| USB ID | ざっくりした意味 |
|---|---|
| `8087:0b37` | T265 runtime mode |
| `03e7:2150` | bootloader / Movidius mode として見える状態 |

runtime API 本体は `8087:0b37` を対象にするのが基本です。`03e7:2150` は runtime へ戻す前段として扱います。

## 調査で大事だった観点

### 1. 「動く」より「状態が説明できる」を優先する

一度動いただけでは不十分です。

- なぜ今runtimeとして見えているのか。
- どの状態ならAPI本体が使えるのか。
- timeoutが出たとき、それは致命的なのか、一時的なWARNなのか。

ここを説明できるようにすると、あとで壊れたときに戻れます。

### 2. 受信処理と処理処理を分ける

データを受け取る場所と、データを処理する場所を混ぜると、詰まりの原因が分かりにくくなります。

調査では、受信側は軽く、処理側は別にすることを意識しました。

### 3. sample種別を混ぜすぎない

pose、gyro、accel、fisheye metadata、image payload は性質が違います。

- 頻度が違う
- サイズが違う
- 必要な処理時間が違う
- 欲しい用途が違う

single queueに全部を詰めると、IMUなど高頻度sampleがqueueを圧迫しやすくなります。multi-queueに分けると、どこが詰まっているか観察しやすくなります。

### 4. timestampは「完全一致」ではなく「近さ」で見る

別種類のsampleを対応付けるとき、timestampが完全一致する前提にしない方がよいです。

実際には、thresholdを設けて「この範囲なら対応付ける」と考える方が扱いやすいです。さらに、accepted ratio や skipped count を統計として見ることで、syncerの品質を観察できます。

### 5. 公開前提でログを残しすぎない

USBログや個体シリアル入りのログは、公開には向きません。

公開用には、数値傾向や設計判断だけを残し、生ログはprivateに置く方が安全です。

## 調べるときのキーワード

このプロジェクトを自分で再調査するときは、以下のキーワードから入ると戻りやすいです。

- T265 runtime mode
- T265 bootloader mode
- `8087:0b37`
- `03e7:2150`
- libusb bulk transfer
- USB descriptor
- USB interface endpoint
- RealSense T265 fisheye timestamp
- T265 pose IMU timestamp
- producer consumer queue
- callback lightweight processing
- buffer ownership
- udev rule USB permission

より詳しい検索用メモは [`../notes/keywords_to_search.md`](../notes/keywords_to_search.md) に分けています。

