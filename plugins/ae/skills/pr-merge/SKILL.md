---
name: pr-merge
description: /pr-merge - PRマージ
disable-model-invocation: true
argument-hint: "[PR番号]"
---

# /pr-merge - PRマージ

指定されたPull Requestをマージする。

**引数:** `/pr-merge [PR番号]`（省略時は現在のブランチのPR）

## 手順

### 1. 対象PRの特定

$ARGUMENTS でPR番号が指定されている場合はそのPRを対象にする。
指定がない場合は、現在のブランチに関連するPRを対象にする。

```bash
gh pr view {PR_NUMBER} --json number,title,state,mergeable,url,headRefName,baseRefName
```

- PRが存在しない場合はその旨を伝えて終了
- マージ不可（コンフリクト等）の場合は、**自動解消せず**以下を報告して終了する：
  - コンフリクトが発生しているファイル
  - 何と何が競合しているか（どのブランチのどの変更同士か）
  - 解消の方針案（ユーザーが判断できる情報）

### 2. マージ内容の表示と確認

AskUserQuestionツールを使い、ユーザーに確認する：

```
以下のPRをマージします：

  #{NUMBER} {TITLE}
  {HEAD_REF} → {BASE_REF}
  {URL}

マージ方式: merge commit
マージ後にリモートブランチを削除します。

よろしいですか？
```

### 3. マージ実行

ユーザーの承認後：

```bash
gh pr merge {PR_NUMBER} --merge --delete-branch
```

### 4. ローカルの整理

```bash
git checkout dev
git pull origin dev
```

### 5. 完了報告

マージ完了を伝える。
