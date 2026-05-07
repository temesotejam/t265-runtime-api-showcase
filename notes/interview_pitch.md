# Interview pitch

## 30秒説明

Intel RealSense T265を、公式SDKに強く依存しすぎずLinux上で扱うための実験的なruntime APIを設計・実装しました。USB上でのruntime / bootloader状態を切り分け、pose・IMU・fisheye系データを用途別に扱えるように、制御向けのlatest state APIとログ・解析向けのqueue / multi-queue APIを分けて設計しました。firmwareや実機固有情報を含むため、実装全体はprivateに置き、公開版では設計思想と検証観点だけをまとめています。

## 1分説明

T265をロボット制御や実験で使うとき、単にサンプルを動かすだけではなく、現在値を低遅延で読みたい場面と、sampleを順番にログ保存・解析したい場面が分かれます。そこで、T265をUSB runtime deviceとして観察し、libusbベースで扱うruntime層を作り、その上にlatest state API、queue API、multi-queue API、syncerを分けて設計しました。

特に意識したのは、callback内で重い処理をしないこと、sample種別ごとに詰まりを観察できること、複数台利用ではserial / roleで識別すること、画像payloadではownershipを曖昧にしないことです。

公開にあたっては、firmware blobや実機シリアル番号を含めないよう、全コード公開ではなく設計資料として切り分けました。

## 技術的に強調するポイント

- Linux USB / libusb の層からデバイス状態を見た。
- T265のruntime / bootloader状態を切り分けた。
- APIを用途別に分けた。
- latest state と queue の違いを設計に反映した。
- callbackを軽くして、処理をqueueやworker側に逃がす構成にした。
- 複数台運用でserial / roleの考え方を入れた。
- 画像payloadのownershipを意識した。
- 公開できる成果と公開しないファイルを切り分けた。

## 聞かれたときの答え方

### なぜコードを公開していないのですか？

firmware handling、実機固有のシリアル設定、USB調査ログなどが含まれるため、全コードはprivateで管理しています。公開版では、権利や個体情報に配慮して、設計思想・構成・検証観点だけを公開しています。

### 何が一番難しかったですか？

T265を単なるカメラとしてではなく、USB状態を持つruntime deviceとして整理することです。特に、sample種別ごとの流量差、queueの詰まり、timestamp対応付け、画像payloadのownershipを分けて考える必要がありました。

### 何が一番工夫した点ですか？

制御用途とログ用途を分けた点です。制御では最新値を読むlatest state API、ログや解析ではsampleを順番に扱うmulti-queue APIを使うようにしました。

