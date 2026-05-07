# Verification and test strategy

## 検証の考え方

T265 runtime APIの検証では、「一度動いた」だけでは足りません。

見るべきなのは、次の4つです。

1. 起動できるか。
2. sampleが継続して取れるか。
3. queueやcallbackが詰まらないか。
4. 長時間でdrop / timeout / backlogが増えないか。

## 検証を段階に分ける

### Phase 1: USB状態確認

最初に見るのはUSB状態です。

- runtime modeとして見えているか。
- bootloader modeとして見えていないか。
- permissionで失敗していないか。
- 複数台が期待通り見えているか。

### Phase 2: device open / stream確認

次に、runtime deviceを開けるか確認します。

ここでは、まだ複雑なqueueやsyncerを使わず、最小限のopen/start/stopを確認します。

### Phase 3: latest state確認

pose / gyro / accel などの最新値が更新されているかを確認します。

見るべき点:

- timestampが進むか。
- 値が更新されるか。
- 一定時間止まらないか。

### Phase 4: queue確認

sampleをqueueに入れてpopできるかを確認します。

見るべき点:

- queueにsampleが入るか。
- popできるか。
- queue emptyを正常状態として扱えているか。
- droppedが増えていないか。

### Phase 5: multi-queue確認

motion / fisheye metadata / image payload などを分けて扱えるか確認します。

見るべき点:

- どのqueueが詰まりやすいか。
- 高頻度sampleが他を圧迫していないか。
- backlogが増え続けていないか。

### Phase 6: syncer確認

timestampで近いsampleを対応付けられるか確認します。

見るべき点:

- accepted ratio
- skipped count
- timestamp delta
- thresholdごとの差
- 長時間での安定性

### Phase 7: long-run確認

最後に、長時間動作で問題が出ないか確認します。

短時間では見えない問題:

- queueのじわじわした増加
- たまに出るtimeout
- 保存処理による詰まり
- メモリ使用量増加
- USB抜き差し後の復帰問題

## PASS / WARN / FAILの考え方

T265やUSB周りでは、一時的なtimeoutが必ずしも即FAILとは限りません。

大事なのは、最終状態と継続性を見ることです。

| 状態 | 判断 |
|---|---|
| sampleが継続し、最終USB状態もruntime | PASSまたはWARN |
| 一時timeoutがあるが復帰している | WARN候補 |
| sampleが止まる | FAIL |
| queue backlogが増え続ける | FAILまたは設計見直し |
| droppedが増え続ける | FAILまたはqueue/処理負荷見直し |
| runtime deviceが消える | FAIL |

## ログで見るべき項目

- runtime device count
- serial / role対応
- pose timestamp
- gyro / accel timestamp
- fisheye metadata timestamp
- queue depth
- dropped count
- backlog
- accepted ratio
- skipped count
- timeout count
- run duration

## 公開資料でのログ扱い

公開資料には、生ログをそのまま載せない方が安全です。

代わりに、次のようにまとめます。

- 何秒確認したか。
- どのAPI pathを確認したか。
- PASS / WARN / FAIL の判定基準。
- どの指標を見たか。
- 実機固有情報を除いた結果概要。

## 自分用メモ

検証で一番大事なのは、**詰まりを観察できる設計になっているか**です。

動かない原因より、じわじわ遅れる原因の方が見つけにくいです。queue depth、dropped、backlogは必ず見る。

