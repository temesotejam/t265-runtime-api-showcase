# T265 Runtime API — 設計資料ショーケース

Intel RealSense T265 を Linux 上で扱うために、どのように調査し、どのように API と runtime 構造を考え、どこを重要視して実装したかをまとめた公開用ドキュメントです。

このリポジトリは **コード公開ではなく、設計資料・技術調査メモ・成果説明用のショーケース** です。実装コード全体、firmware、実機固有設定、生ログは含めていません。

## 範囲

このリポジトリは、`librealsense` の完全な代替実装を公開するものではありません。

非公開実装では、Intel RealSense T265 の runtime mode におけるアクセス方法、API設計、queue処理、同期、検証方法を調査しました。この公開リポジトリでは、実装コードではなく設計思想・調査観点・検証方針のみを公開しています。

firmware loader、完全なdriver、production-ready SDKは含みません。

## この資料の目的

この資料は、次の2つを同時に満たすために作っています。

1. 自分があとで見返したときに、「何を考えて作ったのか」を思い出せること。
2. 他の人が見たときに、「どこを調べれば同じ方向に進めるのか」が分かること。

コードそのものよりも、次のような判断の記録を重視しています。

- なぜ `librealsense` だけに寄せず、`libusb` ベースの runtime API を考えたのか。
- T265 の USB 上の状態をどう見たのか。
- 制御用途とログ用途で API を分けた理由。
- `latest state`、`queue`、`multi-queue`、`syncer` をどう使い分けたのか。
- どこが詰まりやすく、どこを先に検証すべきか。
- 公開すると危ないものをどう切り分けたか。
- 実装コードを出さずに、どのようなAPI契約・検証観点を示せるか。

## 先に結論

今回の設計で一番重要だったのは、**T265 を「データを出すUSBデバイス」として見ること**、そして API を用途別に分けることでした。

| 用途 | 考え方 |
|---|---|
| ロボット制御 | 古いデータを全部処理するより、常に最新状態を読む |
| ログ保存 | 取りこぼしや順序を観察できる queue に流す |
| 複数台運用 | index ではなく serial / role で識別する |
| 画像処理 | 画像本体の lifetime / ownership を明確にする |
| timestamp同期 | 完璧な同期より、閾値と統計で観察可能にする |
| callback | 重い処理を避け、copy/enqueue 程度に抑える |
| shutdown | reader、producer、queue、buffer、device、context の順序を決める |
| 公開 | firmware・実機設定・生ログを出さず、設計だけを公開する |

## この公開版に含めているもの

- 設計思想
- 調査観点
- アーキテクチャ説明
- API分類の考え方
- API契約の考え方
- queue / threading / syncer の設計メモ
- 匿名化した実機検証サマリと見るべきポイント
- timestamp / 座標系の注意点
- 公開・非公開の境界線
- 将来の改善案
- Mermaid形式の構成図

## この公開版に含めていないもの

- 実装ソースコード
- ヘッダファイル本文
- 関数実装や構造体定義の全文
- firmware binary
- firmware を抽出・再配布する手順
- 実機シリアル番号
- 実機固有の role 設定
- USB生ダンプ
- ビルド済みバイナリ
- 個体環境に依存するログ一式
- private repository のURL

## 推奨の読み方

はじめて見る人は、次の順番がおすすめです。

1. [`docs/01_PROJECT_STORY.md`](docs/01_PROJECT_STORY.md)
2. [`docs/02_PROBLEM_AND_GOALS.md`](docs/02_PROBLEM_AND_GOALS.md)
3. [`docs/05_ARCHITECTURE.md`](docs/05_ARCHITECTURE.md)
4. [`docs/06_API_DESIGN_PHILOSOPHY.md`](docs/06_API_DESIGN_PHILOSOPHY.md)
5. [`docs/13_PUBLIC_API_SURFACE.md`](docs/13_PUBLIC_API_SURFACE.md)
6. [`docs/15_API_CONTRACTS.md`](docs/15_API_CONTRACTS.md)
7. [`docs/16_COORDINATE_AND_TIMESTAMP_NOTES.md`](docs/16_COORDINATE_AND_TIMESTAMP_NOTES.md)
8. [`docs/10_PUBLICATION_BOUNDARY.md`](docs/10_PUBLICATION_BOUNDARY.md)

自分が再開発・再確認するときは、次の2つから読むと早いです。

- [`docs/03_RESEARCH_MAP.md`](docs/03_RESEARCH_MAP.md)
- [`docs/APPENDIX_CHECKLISTS.md`](docs/APPENDIX_CHECKLISTS.md)

## リポジトリ構成

```text
T265_API_SHOWCASE_DOCS_PUBLIC/
├── README.md
├── NOTICE.md
├── LICENSE.md
├── docs/
│   ├── 00_INDEX.md
│   ├── 01_PROJECT_STORY.md
│   ├── 02_PROBLEM_AND_GOALS.md
│   ├── 03_RESEARCH_MAP.md
│   ├── 04_RUNTIME_BOOTLOADER_MODEL.md
│   ├── 05_ARCHITECTURE.md
│   ├── 06_API_DESIGN_PHILOSOPHY.md
│   ├── 07_THREADING_QUEUE_SYNC.md
│   ├── 08_IMAGE_HANDLING_NOTES.md
│   ├── 09_VERIFICATION_AND_TEST_STRATEGY.md
│   ├── 10_PUBLICATION_BOUNDARY.md
│   ├── 11_LESSONS_LEARNED.md
│   ├── 12_FUTURE_WORK.md
│   ├── 13_PUBLIC_API_SURFACE.md
│   ├── 14_VERIFICATION_SUMMARY.md
│   ├── 15_API_CONTRACTS.md
│   ├── 16_COORDINATE_AND_TIMESTAMP_NOTES.md
│   ├── 17_DESIGN_DECISIONS.md
│   └── APPENDIX_CHECKLISTS.md
├── diagrams/
│   ├── architecture.mmd
│   ├── api_choices.mmd
│   ├── data_flow.mmd
│   └── usb_state_flow.mmd
└── notes/
    ├── interview_pitch.md
    └── keywords_to_search.md
```

## 状態

これは公開用に整理したドキュメントパッケージです。検証には非公開実装を使いましたが、実装詳細、firmware handling、実機固有設定、private logを含むため、この公開リポジトリの外に置いています。

このリポジトリは独立した非公式資料です。Intel / RealSense の公式プロジェクトではなく、設計メモと検証方針として読んでください。

## ライセンス

このリポジトリ内の文章・図は `LICENSE.md` の条件で公開します。実装コードや firmware に対する権利は、この公開資料には含まれません。
