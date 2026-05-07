# Design decisions

このページは、後から見返したときに「なぜそうしたのか」を思い出すための判断記録です。

## Why latest and queue are separate

Latest stateとqueueは、同じsampleを扱っていても目的が違います。

Latest stateは、control loopが「いまの値」を読むための入口です。古いsampleを全部処理することより、低遅延で最新snapshotを読むことを優先します。

Queueは、loggingや解析のための入口です。sample順序、drop、backlog、timestamp deltaを観察するため、履歴を持つことに意味があります。

この2つを1つの万能APIにすると、制御用途では古いsample処理が邪魔になり、ログ用途ではdropや順序が見えにくくなります。

## Why callbacks should stay light

Callbackは受信経路の一部です。ここにdisk I/O、画像保存、同期処理、重い解析を置くと、受信そのものが遅れます。

そのため、callbackはcopy/enqueueやlatest snapshot更新程度に抑え、重い処理はworker側へ逃がす方針にしました。これにより、遅延の原因をcallback、queue、consumerのどこにあるか分けて観察できます。

## Why multi-queue matters

Pose、motion、image payload、metadataは流量もサイズも違います。

すべてを1つのqueueに入れると、高頻度sampleや大きなpayloadが他のsampleを圧迫する可能性があります。Multi-queueにすると、どの種類が詰まっているか、どのconsumerが遅れているかを見やすくなります。

複数台のT265を扱う場合も、deviceごと、sample種別ごとに観察できることが重要です。

## Why syncer uses a threshold

複数streamのtimestampが常に完全一致するとは限りません。

Syncerは完全一致を前提にせず、threshold内で近いsampleを採用する設計にしました。その代わり、accepted ratio、skipped count、timestamp deltaを観察して、thresholdが用途に合っているかを確認します。

## Why serial and role are private

Serial numberは複数台運用には重要ですが、公開資料に載せる必要はありません。

Public docsでは、serial-based selectionやrole-based selectionの考え方だけを説明します。実際のserial number、role mapping、device固有設定はprivate boundaryの外に置きます。

## Why firmware is not published

Firmware binaryやfirmwareに由来するファイルは、再配布権限や第三者権利の境界が不明確になりやすい領域です。

この公開repoでは、runtime/bootloaderの概念や検証観点だけを説明し、firmware binary、firmware hash、firmware file path、firmware helperの詳細は公開しません。

## Why this is a docs-only showcase

このプロジェクトで見せたい成果は、低レベル実装の全文ではなく、設計判断と検証の考え方です。

Docs-onlyにすることで、firmware、serial number、raw USB dump、private log、実装コードを公開せずに、次の点を説明できます。

- T265をruntime modeで扱うときに何を調べるべきか
- latest / queue / multi-queue / syncerをなぜ分けたか
- callback、ownership、shutdownをなぜ契約として扱うか
- 実機検証でどの指標を見るべきか
