# 検証サマリ

このページは、公開可能な範囲に絞った匿名化済みの検証サマリです。

詳細な数値ログ、USB生ログ、serial number、firmwareファイル名、firmware hash、private file pathは公開しません。数値を公開できない項目は、無理に作らず `<not published>` または `kept private` としています。

## テスト環境

| 項目 | 公開サマリ |
|---|---|
| OS | Ubuntu 24.04 LTS |
| Kernel | Linux 6.8.0-31-generic x86_64 |
| Compiler | GCC 13.2.0 |
| Build tool | GNU Make 4.3 |
| libusb version | `<public environmentでは取得不可>` |
| Device count | private validationでは2台を使用 |
| Device-specific serial numbers | redacted |
| Firmware binary | not published |
| Raw USB dumps | not published |

## 公開可能な結果サマリ

この表は、公開しても安全な粒度まで丸めた結果サマリです。詳細なcount、raw log、実機固有情報は公開しません。

| 指標 | 公開結果 |
|---|---|
| Test duration | 300 seconds |
| Device count | 2 devices |
| Runtime-mode observation | completed |
| Runtime start | success |
| Continuous read loop | completed |
| Timeout behavior | monitored; detailed count kept private |
| Dropped samples | monitored; detailed count kept private |
| Max queue backlog | monitored; exact value kept private |
| Two-device run | privately validated |
| Shutdown result | clean stop confirmed |
| Firmware binary | not published |
| Serial numbers | redacted |
| Raw USB logs | not published |

## Runtime observation

| 項目 | 公開サマリ |
|---|---|
| Runtime mode observation | private validationで確認 |
| Bootloader/runtime distinction | 検証checklistの一部として確認 |
| Start behavior | サマリのみ公開。詳細logはkept private |
| Stop behavior | サマリのみ公開。詳細logはkept private |
| Shutdown behavior | サマリのみ公開。詳細logはkept private |
| Test duration | private long-run validationで300 seconds |
| Sample count | `<not published>` |
| Timeout count | `<not published>` |
| Dropped count | `<not published>` |
| Max queue backlog | `<not published>` |

## Queue / callback observation

| 項目 | 公開サマリ |
|---|---|
| Latest-state path | private implementationで低遅延read pathとして検証 |
| Queue path | private implementationで時系列処理pathとして検証 |
| Multi-queue path | 検証時にdevice/sample categoryの分離に使用 |
| Callback rule | callbackは短くし、重い処理はcallback pathの外へ移動 |
| Backlog metrics | private validationで確認 |
| Drop metrics | private validationで確認 |
| Full logs | not published |

## Multi-device observation

| 項目 | 公開サマリ |
|---|---|
| Device selection | serial-based selectionはprivately validated。role-based mappingはdesign documented |
| Role mapping | design documented。実際のmappingはnot published |
| Two-device run | privately validated |
| Per-device identity | serial numbers redacted |
| Per-device logs | not published |

## 複数台検証の公開境界

この表では、実際に検証したもの、設計として公開しているもの、公開しないものを分けています。

| 項目 | 公開状態 |
|---|---|
| Multiple runtime devices detected | privately validated |
| Two-device long-run read | privately validated |
| Serial-based selection | privately validated; actual serials redacted |
| Role-based mapping | design documented; private configuration not published |
| Published role configuration | example only |
| Device-specific serial numbers | not published |
| Raw per-device logs | not published |

## Shutdown observation

| 項目 | 公開サマリ |
|---|---|
| Reader stop | private validation checklistに含めて確認 |
| Producer/callback stop | private validation checklistに含めて確認 |
| Queue drain/discard | shutdown時の明示的な判断として扱う |
| Buffer release | ownership contractの一部として扱う |
| Device close | private validation checklistに含めて確認 |
| Context destroy | private validation checklistに含めて確認 |

## 公開しないもの

| 項目 | 公開サマリ |
|---|---|
| Device-specific serial numbers | redacted |
| Firmware binary | not published |
| Firmware hash | not published |
| Firmware file name/path | not published |
| Raw USB dumps | not published |
| Full private logs | not published |
| Implementation source code | not published |
| Header contents | not published |
| Private repository URL | not published |

## 解釈

この公開リポジトリでは、何を確認したか、どのような検証構造にしたかを示します。非公開実装を再現することや、実機固有情報・再配布上注意が必要な情報を含むraw evidenceを公開することは目的にしていません。

公開できる重要な成果は、検証戦略そのものです。runtime mode、複数台選択、latest-state read、queue behavior、callback pressure、drop/backlog metrics、timestamp behavior、shutdown orderを、最初から検証対象として扱いました。
