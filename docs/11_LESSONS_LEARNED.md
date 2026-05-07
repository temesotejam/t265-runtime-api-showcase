# Lessons learned

## 1. 最初にUSB状態を見る

動かないときに、いきなりAPIやコードを見るより、まずUSB上の状態を確認する方が早いです。

runtime modeなのか、bootloader modeなのかで、必要な対応がまったく変わります。

## 2. 制御用途とログ用途を混ぜない

latest state API と queue API を分けたのは良い判断でした。

制御では最新値が重要です。ログでは履歴が重要です。この2つを同じAPIで扱うと、どちらかが使いにくくなります。

## 3. callbackを重くしない

callback内で保存や解析を始めると、受信処理が詰まります。

callbackは軽く、処理はqueueの外で行う。この方針はかなり重要です。

## 4. single queueは分かりやすいが、実用ではmulti-queueが見やすい

single queueは最初の確認には便利です。

しかし、sample種別ごとに頻度やサイズが違うため、詰まりの原因を分けるにはmulti-queueの方が向いています。

## 5. 画像は所有権を曖昧にしない

画像payloadは、poseやIMUより壊れやすいポイントです。

viewなのか、owned copyなのか、callback後も保持してよいのかを明確にする必要があります。

## 6. timestamp同期は統計で見る

syncerでは、完全一致ではなく近さで対応付けます。

thresholdを決めるだけでなく、accepted ratio、skipped count、deltaを見ることが大切です。

## 7. 複数台ではserial / roleが重要

index指定は簡単ですが、抜き差しで変わる可能性があります。

複数T265を扱うなら、serial numberで識別し、アプリ側ではrole名で扱う方が良いです。

## 8. long-runをしないと分からないことがある

短時間のPASSでは、queueのじわじわした詰まりや、まれなtimeoutを見落とします。

最低限のsmoke testと、少し長めのrunを分けるべきです。

## 9. 公開しない判断も成果

firmware、実機シリアル、USB生ダンプを出さない判断は重要です。

安全に見せるために、設計資料として公開する形にしたのは良い落としどころです。

## 10. 実装コードより、判断の記録があとで効く

時間が経つと、コードの細部より「なぜこの設計にしたのか」を忘れます。

この資料のように、判断理由を残しておくと再開しやすいです。

