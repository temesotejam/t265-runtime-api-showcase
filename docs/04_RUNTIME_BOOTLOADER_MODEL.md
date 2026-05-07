# Runtime / bootloader model

T265を扱うときに最初に整理すべきなのは、USB上での見え方です。

## 2つの状態を分けて考える

T265は常に同じUSBデバイスとして見えるわけではありません。状態によって、OSから見えるIDや扱い方が変わります。

| 状態 | USB上の見え方 | 扱い |
|---|---|---|
| runtime mode | `8087:0b37` | pose / IMU / fisheye を扱う通常状態 |
| bootloader / Movidius mode | `03e7:2150` | runtimeへ戻すための前段 |

今回のAPI設計では、**通常利用の中心をruntime modeに置く**ことにしました。

bootloader mode を直接API本体の一部にしすぎると、firmwareの扱い、権利、公開可否、失敗時の復旧などが混ざります。そのため、bootloaderからruntimeへ戻す処理は補助機能として切り分ける方針がよいと判断しました。

## なぜこの切り分けが重要か

この切り分けをしないと、次の問題が起きます。

- runtime API を使いたいだけなのに firmware 依存が混ざる。
- 公開リポジトリに firmware blob を含めたくなる。
- 起動失敗なのか、runtime中の通信失敗なのか区別しにくい。
- テスト手順が複雑になる。

反対に、runtime API と boot helper を分けると、次のように整理できます。

| 層 | 役割 |
|---|---|
| boot helper | bootloader状態のT265をruntimeへ戻す |
| runtime core | runtime状態のT265を開いてstreamを扱う |
| reader / callback | data sampleを受け取る |
| latest / queue / syncer | 用途別にsampleを扱う |

## 公開時の考え方

公開版では、firmware blob は含めません。

理由は単純で、firmwareは自作コードではなく、再配布権限を明確にしにくいからです。

公開資料で書いてよいのは、次のレベルまでに留めるのが安全です。

- T265にはruntime modeとbootloader modeがある。
- runtime API本体はruntime modeを対象にする。
- bootloaderからruntimeへ戻すには、ローカルで適切なfirmwareが必要になる。
- firmwareそのものは公開リポジトリに含めない。

## 実装設計への影響

runtime / bootloader を分けたことで、API設計はかなりきれいになります。

- device enumeration は runtime device を中心にする。
- boot helper は通常ビルドから外す。
- firmwareがない状態でも、設計資料・runtime API・examplesの説明は成立する。
- private環境だけでfirmwareを扱い、public環境では存在しない前提にできる。

## 自分用メモ

この部分で一番大事なのは、T265を「カメラ」ではなく、まず「USB状態を持つデバイス」として見ることです。

動かないときに最初に見るべきなのはAPIではなく、USB上でどう見えているかです。

