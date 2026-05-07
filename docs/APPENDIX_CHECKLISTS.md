# Appendix: checklists

## 再開時チェックリスト

久しぶりにこのプロジェクトを再開するときは、次の順で見ると戻りやすいです。

- [ ] T265がUSB上でruntime modeとして見えているか。
- [ ] bootloader modeとして見えていないか。
- [ ] udev permissionは問題ないか。
- [ ] 1台だけでlatest stateが取れるか。
- [ ] queueにsampleが入るか。
- [ ] queue droppedが増えていないか。
- [ ] 複数台の場合、serialが読めるか。
- [ ] role fileのleft/rightが正しいか。
- [ ] syncerのthresholdは用途に合っているか。
- [ ] image payloadを保持する場合、owned copyになっているか。

## 設計確認チェックリスト

- [ ] callback内で重い処理をしていないか。
- [ ] latest state と queue を混ぜていないか。
- [ ] sample種別ごとに詰まりを観察できるか。
- [ ] queue full時の扱いが決まっているか。
- [ ] queue emptyを正常状態として扱えているか。
- [ ] shutdown順序が明確か。
- [ ] image view / owned image の区別が明確か。
- [ ] timestamp thresholdの根拠があるか。

## 長時間確認チェックリスト

- [ ] run durationを決めたか。
- [ ] sample countを記録しているか。
- [ ] timeout countを見ているか。
- [ ] queue droppedを見ているか。
- [ ] backlog maxを見ているか。
- [ ] 最終USB状態を確認したか。
- [ ] WARNとFAILの基準を分けたか。

## 公開前チェックリスト

- [ ] firmware binaryを含めていないか。
- [ ] `.bin`, `.fw`, `.mvcmd`, `.o` が入っていないか。
- [ ] 実機シリアル番号が入っていないか。
- [ ] USB生ダンプが入っていないか。
- [ ] 個体依存ログが入っていないか。
- [ ] ビルド済みバイナリが入っていないか。
- [ ] READMEに非公式プロジェクトであることを書いたか。
- [ ] 実装コード全体を含めない方針を書いたか。
- [ ] 公開する目的が「設計思想の共有」になっているか。

## 面接・説明前チェックリスト

- [ ] 何を作ったかを一言で説明できるか。
- [ ] なぜ公式SDKだけではなく下位層を見たのか説明できるか。
- [ ] latest state と queue を分けた理由を説明できるか。
- [ ] callbackを軽くする理由を説明できるか。
- [ ] multi-queueの利点を説明できるか。
- [ ] firmwareを公開しない理由を説明できるか。
- [ ] private repoとpublic docs repoを分けた理由を説明できるか。

