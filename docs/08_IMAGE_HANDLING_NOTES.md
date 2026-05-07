# Image handling notes

## なぜ画像周りが難しいか

T265のfisheye imageを扱うとき、poseやIMUより注意が必要です。

理由は、image payload が大きく、bufferのlifetimeや所有権が問題になりやすいからです。

metadataだけなら小さな構造体として扱いやすいですが、画像本体は次の点を考える必要があります。

- bufferは誰が所有しているのか。
- callback終了後も参照してよいのか。
- deep copyが必要か。
- queueに積むのはviewかowned imageか。
- 保存処理はどのthreadで行うのか。
- 画像保存が受信処理を詰まらせないか。

## image view と owned image

設計上は、画像を次の2種類に分けて考えると整理しやすいです。

| 種類 | 考え方 |
|---|---|
| image view | 元bufferを参照する軽量な見方 |
| owned image | deep copyして所有する画像 |

image viewは軽いですが、元bufferのlifetimeに依存します。

owned imageは安全に保持できますが、copyコストとメモリ使用量が増えます。

## callback内で画像保存しない

画像保存は重い処理です。

callback内で保存すると、受信処理が詰まる原因になります。保存したい場合は、次のように分ける方が安全です。

1. callbackでは必要最小限の情報をqueueに逃がす。
2. 必要ならowned imageとしてcopyする。
3. save workerが別threadで保存する。

## image queueを使うときの観察ポイント

画像queueを使う場合、次の指標を見る必要があります。

- queue depth
- dropped image count
- saved frame count
- save latency
- backlog
- memory usage
- timestamp continuity

画像はサイズが大きいため、poseやIMUと同じ感覚でqueueに入れるとすぐ詰まる可能性があります。

## metadata と payload を分ける

fisheye metadata と image payload は分けて扱った方がよいです。

metadataは軽く、timestamp対応付けにも使いやすいです。

payloadは重く、copyや保存の方針を明確にしないと危険です。

| 種類 | 扱い |
|---|---|
| fisheye metadata | queue / syncerで扱いやすい |
| image payload | ownershipとworker threadを意識する |

## 公開資料での扱い

公開版では、画像処理の実装コードや保存workerの中身は出さず、設計方針だけを説明するのが安全です。

特に、USBから取った生データや画像dumpを公開する必要はありません。

## 自分用メモ

画像周りで一番大切なのは、**参照しているだけなのか、所有しているのかを曖昧にしない**ことです。

曖昧なままqueueに入れると、あとで壊れた画像、use-after-free、謎のクラッシュにつながります。

