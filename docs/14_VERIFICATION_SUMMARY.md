# Verification summary

このページは、公開可能な範囲に絞った匿名化済みの検証サマリです。

詳細な数値ログ、USB生ログ、serial number、firmwareファイル名、firmware hash、private file pathは公開しません。数値を公開できない項目は、無理に作らず `<not published>` または `kept in private repository` としています。

## Test environment

| Item | Public summary |
|---|---|
| OS | Ubuntu 24.04 LTS |
| Kernel | Linux 6.8.0-31-generic x86_64 |
| Compiler | GCC 13.2.0 |
| Build tool | GNU Make 4.3 |
| libusb version | `<not available in public environment>` |
| Device count | 2 devices were used during private validation |
| Device-specific serial numbers | Redacted |
| Firmware binary | Not published |
| Raw USB dumps | Not published |

## Runtime observation

| Item | Public summary |
|---|---|
| Runtime mode observation | Checked in private validation |
| Bootloader/runtime distinction | Used as part of the validation checklist |
| Start behavior | Summarized only; detailed logs kept private |
| Stop behavior | Summarized only; detailed logs kept private |
| Shutdown behavior | Summarized only; detailed logs kept private |
| Test duration | 300 seconds in private long-run validation |
| Sample count | `<not published>` |
| Timeout count | `<not published>` |
| Dropped count | `<not published>` |
| Max queue backlog | `<not published>` |

## Queue / callback observation

| Item | Public summary |
|---|---|
| Latest-state path | Validated as the low-latency read path in private implementation |
| Queue path | Validated as the sequential processing path in private implementation |
| Multi-queue path | Used to separate device/sample categories during validation |
| Callback rule | Callback kept short; heavy processing moved outside the callback path |
| Backlog metrics | Checked in private validation |
| Drop metrics | Checked in private validation |
| Full logs | Not published |

## Multi-device observation

| Item | Public summary |
|---|---|
| Device selection | Serial-based and role-based selection were considered |
| Role mapping | Concept documented; actual mapping not published |
| Two-device run | Validated privately |
| Per-device identity | Serial numbers redacted |
| Per-device logs | Not published |

## Shutdown observation

| Item | Public summary |
|---|---|
| Reader stop | Included in private validation checklist |
| Producer/callback stop | Included in private validation checklist |
| Queue drain/discard | Treated as an explicit shutdown decision |
| Buffer release | Treated as part of ownership contract |
| Device close | Included in private validation checklist |
| Context destroy | Included in private validation checklist |

## What is not published

| Item | Public summary |
|---|---|
| Device-specific serial numbers | Redacted |
| Firmware binary | Not published |
| Firmware hash | Not published |
| Firmware file name/path | Not published |
| Raw USB dumps | Not published |
| Full private logs | Not published |
| Implementation source code | Not published |
| Header contents | Not published |
| Private repository URL | Not published |

## Interpretation

The public repository shows what was checked and how the validation was structured. It does not attempt to reproduce the private implementation or publish raw evidence that may contain device-specific or redistribution-sensitive information.

The important public result is the verification strategy: runtime mode, multi-device selection, latest-state reads, queue behavior, callback pressure, drop/backlog metrics, timestamp behavior, and shutdown order were treated as first-class validation targets.
