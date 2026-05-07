# Threading, queue, and sync

## なぜthreadingが重要か

T265からのsample受信は、アプリ側の処理とは独立して継続します。

アプリ側が重い処理をしている間に受信処理が止まると、sampleの遅延やdropにつながります。そのため、受信処理と処理処理を分ける必要があります。

## 基本方針

基本方針は次の通りです。

1. reader thread がT265からsampleを受ける。
2. callbackは短時間で終わる処理だけを行う。
3. 必要に応じて latest state を更新する。
4. 必要に応じて queue に sample を積む。
5. 重い保存・解析・画像処理は別threadまたはpop側で行う。

## callbackでやってよいこと

callback内では、軽い処理だけにします。

やってよいこと:

- sample種別を判定する。
- latest state を更新する。
- queue に push する。
- 統計カウンタを増やす。

避けること:

- 画像ファイル保存
- 長いprintf出力
- 重い画像変換
- ネットワーク送信
- blocking I/O
- mutexを長時間保持する処理

## queue設計

queueは、受信側と処理側を分けるために使います。

| 項目 | 見る理由 |
|---|---|
| depth | queue容量が足りているか |
| remaining | backlogが増えていないか |
| dropped | 処理が追いついていないか |
| sample kind | どの種類が詰まるか |

## single queue と multi-queue

### single queue

single queueは単純です。最初の検証には向いています。

ただし、sample種別ごとの流量差が大きい場合、高頻度sampleがqueueを圧迫しやすくなります。

### multi-queue

multi-queueでは、sample種別ごとにqueueを分けます。

これにより、次のような見方ができます。

- motion queueだけ詰まっているのか。
- fisheye metadata queueだけ遅れているのか。
- image payloadが重すぎるのか。
- pop側の処理が遅いのか。

実用ではmulti-queueの方が観察しやすいです。

## syncerの考え方

syncerは、fisheye frameset と pose / gyro / accel を timestamp で対応付けるためのものです。

ここで重要なのは、timestampの完全一致を期待しないことです。

代わりに、thresholdを設けます。

```text
fisheye timestamp
  ± threshold
nearest pose / gyro / accel
```

## thresholdの見方

thresholdは広すぎても狭すぎても問題があります。

| threshold | 起きやすいこと |
|---|---|
| 狭い | 対応付けられないsampleが増える |
| 広い | 時間的に遠いsampleを採用する可能性が増える |

そのため、thresholdは固定値として決め打ちするだけでなく、統計を見て調整します。

見るべき指標:

- accepted ratio
- skipped out of threshold
- timestamp delta max / average
- timeoutの有無
- 長時間runでの安定性

## 実機で重要だったこと

実機では、短時間でPASSしても、長時間でbacklogやtimeoutが出ることがあります。

そのため、次の2種類の確認が必要です。

1. smoke test: すぐ動くか確認する。
2. long-run test: 数分以上動かして詰まりを見る。

特に画像payloadや保存処理を入れる場合、受信threadから切り離すことが重要です。

## 自分用メモ

この部分の設計で一番大事なのは、**callbackを処理場所にしない**ことです。

callbackは「受け取って逃がす場所」。処理はqueueの外でやる。この方針を守ると、後から問題を切り分けやすくなります。

