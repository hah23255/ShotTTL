# [完了] ShotTTL バグ・セキュリティ・品質監査 2026-07-10

> 最終更新: 2026-07-10

## context配分

| 章 | 種別 | 概要 |
|---|---|---|
| C1 | fix | 監査前提・前回 finding 再検証・新規走査 |
| C2 | fix | 確定 high/medium の最小修正（Win/Unix） |
| C3 | fix | 検証・再調査・結果報告 |

## 作業目的

ソースコード全体（主に `scripts/`）のバグ・セキュリティ・品質監査。現行機能を壊さない範囲で確定 finding を修正。コミット・ビルドなし。

## 対象範囲

- `scripts/windows/shotttl.ps1`
- `scripts/windows/run-hidden.vbs`
- `scripts/unix/shotttl.sh`
- README 安全記述

## 除外

- 撮影/GUI/exe/常駐
- settings.json ローダー新規実装
- allow-list のみへの仕様刷新

## DB を使わない前提

永続 DB なし。状態は対象ファイル mtime・ログ・CLI 引数のみ。

## 禁止事項

- 停止禁止 / DB 前提禁止 / 抜本改修禁止 / ビルド・commit 禁止
- 既定 Trash を Delete にしない、rm フォールバックを入れない

## 確認済みルール

1. Windows `Get-ChildItem -Recurse` は PS 5.1 / 7.6 とも既定で junction を辿らない（WIN-001 旧主張は却下）。防御的 BFS は追加済み。
2. `Add-Type Microsoft.VisualBasic` は当該環境 PS7 で成功。欠落環境向けに Shell.Application フォールバックを追加。
3. Unix 中間 symlink は allow/deny すり抜け（U-1）→ `path_has_symlink_component` + HOME 脱出検出で封鎖。
4. 安全モデルは危険プレフィックス拒否 + スクショ allow 例外 + ホーム外明示 target 許可。
5. Linux trash backend 連鎖・stderr→ログは前回修正を維持。

## 実施した修正

詳細は `docs/local/report_bug_security_quality_audit_2026-07-10.md`。

- Win: ログ日跨ぎ/共有、BFS reparse 非侵入、Trash フォールバック
- Unix: 中間 symlink、mv -n 検証、TOCTOU 再検査、ログ、文言パリティ
- README EN/JA Safety 更新

## 検証

- Parse / bash -n OK
- PS5/PS7 機能テスト OK
- Git Bash 機能テスト OK（実 symlink は環境上 SKIP）

## 残課題（進言）

- SEC-001 allow-list 仕様刷新
- DEP-005 VBScript 廃止移行
- DEP-004 CI
- M1–M4 共有定数

## 完了条件

- [x] plan / report 作成
- [x] 調査・敵対的検証
- [x] high 確定の最小修正
- [x] 検証実行
- [x] commit / build 未実施
