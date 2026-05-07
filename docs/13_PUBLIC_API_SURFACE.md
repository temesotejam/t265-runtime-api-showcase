# 公開用API surface

このページは、公開リポジトリで見せるAPI構成を説明するためのものです。

ここでは設計上のカテゴリだけを示します。関数本体、ヘッダ全文、構造体定義、firmware helper、USBプロトコルの詳細は公開しません。

## Device lifecycle

T265を扱うAPIでは、deviceを直接いきなり読むのではなく、lifecycleを明確に分ける必要があります。

| 段階 | 目的 | 注意点 |
|---|---|---|
| Context | USBやdevice管理の親オブジェクト | 最後に破棄される所有者として扱う |
| Device enumeration | 接続中deviceの一覧取得 | indexは安定識別子ではない |
| Open | 対象deviceを開く | serial / role / bus-address など選択方法を分ける |
| Start | runtime streamやreaderを開始する | callbackやqueueを開始前に準備する |
| Stop | readerとproducerを止める | callbackが走っている状態でbufferを破棄しない |
| Close | device handleを閉じる | queueやowned bufferの扱いを先に決める |

## Device selection

複数台のT265を扱う場合、USBの列挙順に依存すると運用が不安定になります。

| 選択方法 | 位置づけ | 向いている用途 |
|---|---|---|
| First available device | 最短のsmoke test向け | 1台だけ接続した検証 |
| Serial-based selection | 個体識別の基本 | 同じ機種を複数台つなぐ運用 |
| Role-based selection | アプリ側の意味付け | left / right / front / rear などの固定配置 |

公開資料では、実機のserial numberやrole fileの内容は公開しません。role方式は、serialを直接アプリに散らさず、意味のある名前に束ねるための設計として説明しています。

## Reader model

Readerは、データをどのような形でアプリに渡すかを決める層です。

| Reader | 考え方 | 主な用途 |
|---|---|---|
| Simple reader | 初回動作確認用の薄い入口 | examples、smoke test |
| Callback reader | 受信時に短い処理を呼ぶ | copy、enqueue、状態更新 |
| Latest-state reader | 最新sampleだけを保持する | 制御ループ、低遅延の現在値取得 |
| Queue reader | sampleを順番に蓄積する | ログ、解析、順序確認 |

これらは優劣ではなく用途の違いです。制御用途ではlatest-stateが扱いやすく、ログ用途ではqueueが向いています。

## Data categories

T265 runtime APIで扱うデータは、種類ごとに流量・サイズ・lifetimeが違います。

| 種類 | 例 | 設計上の注意 |
|---|---|---|
| Pose / tracking state | position、orientation、tracking confidence | 最新値取得と履歴保存の用途を分ける |
| Motion / IMU-like data | gyro、accel相当のsample | 高頻度sampleがqueueを圧迫しないよう分ける |
| Image payload | fisheye image payload | viewかowned bufferかを明確にする |
| Metadata / timestamp | frame metadata、device timestamp | 同期と遅延評価に使う |

## API style comparison

| API style | 使いどころ | 弱点 |
|---|---|---|
| Latest state | control loopで常に最新値を読む | 完全な履歴保存には向かない |
| Queue | loggingやsequential processing | consumerが遅いとbacklogが増える |
| Multi-queue | 複数device、複数sample種別の観察 | queue設計と統計の確認が必要 |
| Syncer | timestampが近いsampleを対応付ける | threshold tuningが必要 |

## 概念的なAPI名

次の名前は、実装関数の公開ではありません。API surfaceを説明するための概念名です。

公開repoでは、関数本体、ヘッダ全文、構造体定義、低レベルI/Oの詳細は公開しません。この表は、利用者から見た責務の分割を説明するためのものです。

| 領域 | 概念的なAPI名 | 目的 |
|---|---|---|
| Context | `context_create` | API contextを作成する |
| Context | `context_destroy` | context単位のresourceを解放する |
| Enumeration | `enumerate_devices` | runtime modeのT265 deviceを探す |
| Device open | `open_first_available` | 最初に見つかったdeviceを開く |
| Device open | `open_by_serial` | serial numberでdeviceを開く |
| Device open | `open_by_role` | local role mappingを通じてdeviceを開く |
| Runtime | `device_start` | runtime data readingを開始する |
| Runtime | `device_stop` | runtime readerの停止を要求する |
| Latest state | `latest_get_snapshot` | 観測済みの最新状態を読む |
| Queue | `queue_pop` | sampleを時系列で読む |
| Multi-device | `multi_queue_poll` | 複数deviceのsampleをpollする |
| Syncer | `syncer_match_nearest` | timestamp threshold内のsampleを対応付ける |
| Cleanup | `device_close` | deviceを閉じてresourceを解放する |

## 意図的に公開しないもの

この公開資料では、次のものはAPI surfaceとして扱いません。

| 非公開 | 理由 |
|---|---|
| Function bodies | 実装コード公開ではないため |
| Full header contents | APIカテゴリの説明に留めるため |
| Firmware helper details | 再配布や誤用につながる可能性があるため |
| Raw USB protocol dump | 実機情報や低レベル詳細を含む可能性があるため |
| Device-specific serials | 個体情報のため |
| Private logs | 環境依存情報が混ざるため |
