---
name: prune
description: /prune - 不要ブランチ削除
disable-model-invocation: true
---

# /prune - 不要ブランチ削除

マージ済み・リモート削除済みのローカルブランチおよびリモートブランチを整理する。

## 手順

### 1. リモート情報の更新

```bash
git fetch --prune
```

### 2. 削除候補の検出

**ローカルブランチ** — 以下の条件に該当するものを検出する：
- devにマージ済みのブランチ
- リモートが削除済み（gone）のブランチ

**リモートブランチ** — 以下の条件に該当するものを検出する：
- PRがマージ済み（closed/merged）でリモートに残っているブランチ

```bash
gh pr list --state merged --json headRefName --limit 100
git branch -r
```

**絶対に削除しないブランチ:** `master`、`stage`、`dev`

### 3. 削除候補の表示

AskUserQuestionツールを使い、削除候補をユーザーに提示して確認する：

```
## 削除候補のブランチ

### ローカル
| ブランチ名 | 状態 |
|-----------|------|
| feature/xxx | マージ済み |
| fix/yyy | リモート削除済み |

### リモート
| ブランチ名 | 状態 |
|-----------|------|
| origin/feature/zzz | PRマージ済み |

上記のブランチを削除してよいですか？
```

削除候補がない場合はその旨を伝えて終了する。

### 4. 削除実行

ユーザーの承認後、各ブランチを削除する：

**ローカル:**
```bash
git branch -d {BRANCH_NAME}
```

`-d` で削除できない場合（未マージ）はスキップし、ユーザーに報告する。

**リモート:**
```bash
git push origin --delete {BRANCH_NAME}
```

### 5. 完了報告

削除結果を報告する。
