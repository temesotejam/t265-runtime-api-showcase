# Documentation index

この資料集は、T265 runtime API を作ったときの「思考の跡」を残すためのものです。

| File | 内容 |
|---|---|
| `01_PROJECT_STORY.md` | 作った背景、方針、全体の物語 |
| `02_PROBLEM_AND_GOALS.md` | 解こうとした問題と設計目標 |
| `03_RESEARCH_MAP.md` | 調査するときに見るべきポイント |
| `04_RUNTIME_BOOTLOADER_MODEL.md` | T265のUSB状態モデル |
| `05_ARCHITECTURE.md` | レイヤ構成と責務分離 |
| `06_API_DESIGN_PHILOSOPHY.md` | API設計思想 |
| `07_THREADING_QUEUE_SYNC.md` | thread / queue / syncer の考え方 |
| `08_IMAGE_HANDLING_NOTES.md` | fisheye image と ownership の注意点 |
| `09_VERIFICATION_AND_TEST_STRATEGY.md` | 検証戦略と見るべき指標 |
| `10_PUBLICATION_BOUNDARY.md` | 公開してよいもの・避けるもの |
| `11_LESSONS_LEARNED.md` | 学んだこと、重要だった判断 |
| `12_FUTURE_WORK.md` | 今後やるなら何を足すか |
| `13_PUBLIC_API_SURFACE.md` | コードを出さずに示すAPI構成 |
| `14_VERIFICATION_SUMMARY.md` | 匿名化済み検証サマリ |
| `15_API_CONTRACTS.md` | latest / queue / callback / ownership / shutdown の契約 |
| `16_COORDINATE_AND_TIMESTAMP_NOTES.md` | timestamp、syncer、座標系の注意点 |
| `17_DESIGN_DECISIONS.md` | なぜその設計にしたかの判断記録 |
| `APPENDIX_CHECKLISTS.md` | 再開時・公開前・検証時チェックリスト |

## この資料で扱う範囲

扱うもの:

- 設計判断
- 調査観点
- runtime API の分割方針
- API利用時の契約と注意点
- 実機検証の見方
- 匿名化済み検証サマリ
- timestamp / coordinate frame の確認観点
- 公開時の安全境界

扱わないもの:

- 実装コード
- firmware
- リバースエンジニアリングの生データ
- 個体シリアル番号
- 実行バイナリ
