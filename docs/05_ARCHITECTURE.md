# Architecture

## 全体方針

設計の中心は、低レベルUSB処理と利用者向けAPIを分けることです。

低レベルでは、libusbを使ってT265 runtime deviceと通信します。しかし、利用者にそのままUSB転送やprotocol detailを触らせると、アプリ側が複雑になります。

そのため、段階的に抽象化します。

```text
T265 device
  ↓
Linux USB stack
  ↓
libusb layer
  ↓
runtime core
  ↓
reader / callback
  ↓
latest state / queue / multi-queue / syncer
  ↓
application
```

## レイヤごとの責務

| Layer | 役割 | 公開度 |
|---|---|---|
| USB / libusb | device open, transfer, descriptor確認 | 内部実装 |
| runtime core | stream設定、開始停止、sample受信 | 内部中心 |
| reader | 受信処理を継続して回す | API境界 |
| latest state | 最新pose/IMUを保持 | 利用者向け |
| queue | sampleを順番に渡す | 利用者向け |
| multi-queue | sample種別ごとに分離 | 利用者向け |
| syncer | timestampで近いsampleを対応付け | 実験的利用者向け |
| tools / probes | 検証・長時間確認 | 開発者向け |

## なぜこの構成にしたか

### 1. USB処理をアプリから隠すため

アプリ側が欲しいのは、多くの場合「poseが読めること」や「sampleを受け取れること」です。USB endpointやtransferの詳細ではありません。

低レベル処理を隠すことで、アプリ側は用途に集中できます。

### 2. 制御とログを分けるため

latest state と queue を分けることで、制御用途とログ用途を両立できます。

- latest state: 古いsampleを捨ててもよい。現在値が重要。
- queue: sampleを順番に処理したい。取りこぼしやbacklogを観察したい。

### 3. 複数台に拡張しやすくするため

T265を1台だけ扱うなら、最初に見つかったdeviceを開くだけでも動きます。

しかし複数台では、indexが変わる可能性があります。そのため、contextを作り、device listを更新し、serialやroleで開く構成にした方が安全です。

### 4. toolsとpublic APIを分けるため

実験用probeや長時間テストは重要ですが、利用者向けAPIではありません。

この2つを混ぜると、何が安定APIで何が検証用なのか分かりにくくなります。

## 推奨構成

最終的な推奨は、次の2本柱です。

| 用途 | 推奨 |
|---|---|
| 制御 | latest state API |
| ログ・解析 | callback -> multi-queue API |

単純な動作確認には simple reader を使い、timestamp対応付けが必要なときだけ syncer を使います。

## 図

Mermaid形式の図を `diagrams/architecture.mmd` に置いています。GitHub上ではMermaidが表示できる環境なら図として見られます。

