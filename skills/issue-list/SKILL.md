---
name: issue-list
description: /issue-list - GitHub Issue一覧表示
disable-model-invocation: true
argument-hint: "[open|closed|all]"
---

# /issue-list - GitHub Issue一覧表示

GitHub Issueの一覧を表示する。

**引数:** `/issue-list [open|closed|all]`（省略時は `open`）

## 手順

### 1. 引数の確認

$ARGUMENTS でstateフィルタを指定できる：
- `open`（デフォルト）: オープン中のIssue
- `closed`: クローズ済みのIssue
- `all`: すべてのIssue

### 2. Issue一覧の取得・表示

```bash
gh issue list --state {STATE}
```

結果をテーブル形式で見やすく表示する。
