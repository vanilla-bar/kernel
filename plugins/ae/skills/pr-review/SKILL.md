---
name: pr-review
description: PRの差分を直接レビューする。観点に基づいてコードレビューし、問題点・改善案を報告する。
disable-model-invocation: true
context: fork
agent: general-purpose
argument-hint: "[PR番号]"
---

# PRレビュー

あなたは独立したレビュアーとして、PRの差分をレビューする。

## 1. 対象PRの特定

$ARGUMENTS でPR番号が指定されている場合はそのPRを対象にする。
指定がない場合は、現在のブランチに関連するPRを対象にする。

```bash
gh pr view {PR_NUMBER} --json number,title,body,baseRefName,headRefName,files
```

PRが見つからない場合はその旨を報告して終了する。

## 2. PR情報の収集

以下を取得する:

```bash
# PRの差分
gh pr diff {PR_NUMBER}

# PRの概要（タイトル・本文・コメント）
gh pr view {PR_NUMBER} --json title,body,comments

# コミット履歴
gh pr view {PR_NUMBER} --json commits
```

## 3. レビュー観点の読み込み

プロジェクト内の `.agents/pr-review-perspectives.md` を探す。

**ファイルが存在する場合:** その内容を読み込み、レビュー観点として適用する。

**ファイルが存在しない場合:** スキル内蔵の [references/perspectives.md](references/perspectives.md) をデフォルトとして使う。

## 4. 関連コードの読み込み

差分に含まれるファイルについて、レビューに必要な周辺コンテキストをReadツールで読み込む:

- 変更されたファイルの全体（差分だけでは判断できない場合）
- 変更が参照している型定義・スキーマ定義
- 関連する既存仕様（`docs/specs/` 配下）
- プロジェクトルール（`.claude/rules/` 配下で該当するもの）

## 5. レビューの実施

観点テーブルの各項目について、PRの差分と関連コードを照合しながらレビューする。

以下のフォーマットで結果を出力する:

```markdown
## PRレビュー結果

### 問題点
重要度順に列挙。各項目に以下を含める:
- **ファイル:行番号** — 該当箇所
- **何が問題か**
- **なぜ問題か**（どこと不整合か、何が漏れているか）
- **修正案**

### 改善提案
必須ではないが改善すると良い点。

### 確認事項
レビューだけでは判断できず、PR作成者に確認が必要な点。

### 総評
変更全体の評価を1-2文で。
```

問題点がない場合は「問題点なし」と明記し、総評のみ出力する。

## 原則

- **PRの内容を変更しない。** Read Only
- **差分だけで判断しない。** 関連コードを読み込んで文脈を理解した上でレビューする
