# API design philosophy

## API設計で一番意識したこと

一番意識したのは、**用途ごとに正しいデータの受け取り方が違う**という点です。

T265からは、pose、gyro、accel、fisheye metadata、image payload など複数種類のデータが来ます。これらをすべて同じ扱いにすると、アプリ側が複雑になります。

そこで、APIを用途別に分けました。

## latest state API

### 目的

ロボット制御やリアルタイム制御で、常に最新の状態を読みたい用途向けです。

### 考え方

古いsampleを全部処理するより、最新のposeやIMUを読み取ることを優先します。

制御周期がT265のsample頻度と完全に一致するとは限りません。そのため、アプリは必要なタイミングで最新値を読むだけにした方が扱いやすいです。

### 向いている用途

- ロボットの姿勢推定入力
- 制御ループ内の現在値取得
- 最新IMU値の確認
- 低遅延を優先する処理

### 注意点

- 過去sampleの完全な履歴は残りません。
- ログ保存や解析には向きません。
- sampleを取りこぼしたかどうかを詳しく見たい場合はqueue系を使います。

## queue API

### 目的

sampleを順番に処理したい用途向けです。

### 考え方

callbackで受け取ったsampleをqueueに入れ、アプリ側があとでpopして処理します。

これにより、受信処理と解析処理を分離できます。

### 向いている用途

- ログ保存
- 時系列解析
- sample順序の確認
- drop / backlog の観察

### 注意点

- queueが詰まる可能性があります。
- callback内で重い処理をすると受信自体が遅れます。
- queue depth と dropped count を見る必要があります。

## multi-queue API

### 目的

sample種別ごとに流量や処理負荷が違う問題を分けるためです。

### 考え方

single queueにすべてのsampleを入れると、高頻度のIMU sampleがqueueを圧迫することがあります。

multi-queueでは、motion系、fisheye metadata系、image系などを分けて扱います。

### 良かった点

- どの種類のsampleが詰まっているか観察しやすい。
- pose / IMU / fisheye metadata の扱いを分けやすい。
- ログ処理側の設計が明確になる。

## simple reader API

### 目的

最短で動作確認するための高レベルAPIです。

### 考え方

open、configure、start、reader開始のような初期化の流れをまとめ、まず動かすための入口にします。

### 向いている用途

- examples
- smoke test
- 初回動作確認
- API利用者の導入

### 注意点

simple reader は便利ですが、最終的なアプリ設計では所有権や停止順序を理解して使う必要があります。

## syncer

### 目的

fisheye frameset と pose / IMU sample を timestamp で対応付けるための補助機能です。

### 考え方

完全一致を期待せず、一定のthreshold内で近いsampleを採用します。

### 重要な指標

- accepted ratio
- skipped count
- timestamp delta
- threshold

### 注意点

syncerは便利ですが、thresholdの選び方によって結果が変わります。default値だけを盲信せず、実機ログで確認する必要があります。

## 複数台API

### 目的

T265を2台以上扱う場合に、USB indexではなく意味のある名前で開けるようにするためです。

### 考え方

- device listをrefreshする。
- serial numberを読む。
- role fileで `left` / `right` のような名前に紐づける。

### なぜ重要か

USB bus/addressや列挙indexは、抜き差しやハブ構成で変わる可能性があります。

一方、serial numberは個体識別に向いています。role fileを使うと、アプリ側では「leftを開く」という意味で扱えます。

## API分類まとめ

| API | 位置づけ | 用途 |
|---|---|---|
| latest state | stable core | 制御・現在値取得 |
| simple reader | stable entry | 最短起動・examples |
| callback registration | stable with rules | 受信入口。callback内は軽くする |
| multi-queue metadata | stable core | ログ・解析 |
| image payload queue | experimental useful | 画像本体。ownership注意 |
| syncer | experimental useful | timestamp対応付け |
| tools/probes | debug only | 実験・長時間確認 |

## 一言でまとめると

このAPI設計の中心は、**データの受け取り方を用途で分ける**ことです。

全部を1つの万能APIにせず、制御・ログ・同期・複数台運用を別々の入口にしたことで、使う側の見通しがよくなりました。

