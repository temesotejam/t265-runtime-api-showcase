# API契約

このページは、実装コードではなくAPI利用者が守るべき契約をまとめたものです。

T265のようなdevice APIでは、「値が取れる」だけでは足りません。古いsampleの扱い、queue overflow、callback、buffer lifetime、shutdown順序を先に決めておく必要があります。

## Latest state API

Latest state APIは、常に最新値を読みたい用途のためのAPIです。

| 契約 | 考え方 |
|---|---|
| 用途 | control loop、現在位置確認、低遅延の状態取得 |
| 履歴 | 古いsampleの完全な履歴保存には向かない |
| 読み取り | 読んだ時点のsnapshotとして扱う |
| 更新 | producer側は新しいsampleで現在値を更新する |
| thread-safety | 読み手が途中更新中の値を見ないように、snapshot境界を設計する |
| dropの見方 | latest APIでは「上書き」が基本なので、履歴dropとは別に考える |

制御ループでは、未処理sampleを全部消化するより、現在の状態を安定して読めることが重要です。一方で、後から解析する用途ではqueue APIを使います。

## Queue API

Queue APIは、sampleを時系列で処理したい用途のためのAPIです。

| 契約 | 考え方 |
|---|---|
| 用途 | logging、解析、順序確認、timestamp評価 |
| producer | callbackやUSB/event threadからsampleをenqueueする |
| consumer | アプリ側のworkerがpopして処理する |
| 観察値 | dropped count、queue depth、max backlog、timeoutを観察する |
| bounded queue | 無制限queueにせず、上限を決める |

### Queue full時の方針

Queueがfullになったときの方針は、用途によって変わります。

| 方針 | 利点 | リスク |
|---|---|---|
| Drop oldest | 最新寄りのデータを残せる | 古い履歴が欠ける |
| Drop newest | 既存の順序を守りやすい | 最新sampleを失う |
| Block producer | dropを避けられる | USB/event threadやcallbackを詰まらせる危険がある |

このプロジェクトでは、producerを長く止めない方針を優先しました。特にdevice受信側やcallback側をblockすると、遅延が連鎖して原因を追いにくくなります。

実装ごとの最終方針はqueueの用途で選びますが、どの方針でも dropped count と backlog を見えるようにすることが重要です。

## Callback rule

Callbackは受信の入口なので、重い処理を置く場所ではありません。

| ルール | 理由 |
|---|---|
| callbackは短くする | 受信側の遅延を増やさないため |
| disk I/Oを避ける | 保存待ちが受信処理に影響するため |
| 画像保存を直接しない | image payloadは大きく、backlogを作りやすいため |
| 同期処理を抱え込まない | threshold探索や複数queue照合はworkerに逃がすため |
| copy/enqueue程度に抑える | callbackの責務を安定させるため |

Callbackでできるだけ早くsampleを軽い形にして、重い処理はworker threadで行う方が観察しやすくなります。

## Image ownership

Image payloadはmetadataより大きく、lifetimeの扱いを間違えると不安定になります。

| 用語 | 意味 |
|---|---|
| View | callback中だけ有効な参照として扱う |
| Owned buffer | callback後も使えるようにコピーしたbuffer |
| Deep copy | payload本体を別bufferへコピーすること |
| Release responsibility | owned bufferを誰が解放するかの責任 |

Callback後もimageを使う場合は、viewのまま持ち回らずowned bufferとして扱います。Queueへ積む場合も、consumerが読む時点でpayloadが有効であることを保証する必要があります。

## Shutdown order

停止処理は、起動処理よりも曖昧になりやすい部分です。

推奨順序:

1. reader stopを要求する
2. USB/event threadを止める
3. producer callbackを止める
4. queueをdrainまたはdiscardする
5. image bufferを解放する
6. deviceを閉じる
7. contextを破棄する

重要なのは、callbackがまだ動く可能性がある状態でqueueやbufferを破棄しないことです。停止時にqueueをdrainするかdiscardするかは用途で決めますが、どちらを選んだかをAPI契約として明確にします。

## Error handling

| Error | 公開用の契約 |
|---|---|
| Timeout | 一時的なtimeoutと停止を区別する |
| Device disconnect | reader active中でも安全に停止へ進める |
| Queue overflow | drop/backlogを観察可能にする |
| Timestamp anomaly | monotonicity、delta、out-of-orderを記録対象にする |
| Invalid frame | sample種別ごとに破棄またはWARN扱いを決める |
| Shutdown while reader active | stop requestを先に出し、callback停止後にresourceを解放する |

エラーは「返す」だけでなく、後から原因を見られるように分類しておくことが大切です。
