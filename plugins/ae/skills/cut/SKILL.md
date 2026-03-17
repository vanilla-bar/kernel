---
name: cut
description: featureブランチを作成しドラフトPRまで一気に作成する。ユーザーが「ブランチ切って」「ブランチ作って」「作業始めて」等と言った時にも使用する。
argument-hint: "[やりたいことの説明 or #issue番号]"
---

# /cut - featureブランチ作成 + ドラフトPR作成

GitHubのデフォルトブランチを起点にfeatureブランチを作成し、ドラフトPRまで一気に作成する。

**引数:** `/cut <やりたいことの説明>` または `/cut #123`（Issue番号指定）（省略時は対話で聞く）

## 手順

### 1. 引数の確認

$ARGUMENTS を確認し、以下のいずれかで処理を分岐する：

**A) Issue番号の場合**（`#123` や `123` のような数字のみのパターン）:

```bash
gh issue view {番号} --json title,labels,body
```

を実行し、Issueのタイトルをブランチの説明・PRタイトルとして使用する。ラベル情報はプレフィックス判定に活用する（`bug` → `fix/` 等）。

**B) テキストの場合**: そのままブランチの説明・PRタイトルとして使用する。

**C) 引数なしの場合**: 以下のメッセージでユーザーに尋ねる：

```
やりたいことや実装したい機能を教えてください。
例: 「ユーザー一覧画面の追加」「ログイン画面のバグ修正」「#123」（Issue番号）
```

### 2. ブランチ名の自動生成

ブランチの説明（Issueタイトルまたはユーザー入力）から、以下のルールに従ってブランチ名を生成する：

**プレフィックス判定:**
- 新機能の追加 → `feature/`（例: 「ユーザー一覧画面の追加」→ `feature/user-list`）
- バグ修正 → `fix/`（例: 「ログイン画面のバグ修正」→ `fix/login-bug`）
- リファクタリング → `refactor/`（例: 「認証処理のリファクタリング」→ `refactor/auth`）
- ドキュメント → `docs/`（例: 「READMEの更新」→ `docs/readme-update`）
- テスト → `test/`（例: 「ユニットテストの追加」→ `test/unit-tests`）
- その他（設定変更、CI等） → `chore/`

**Issue番号指定時の命名規則:**
- ブランチ名にIssue番号を含める（例: `feature/123-user-list`）
- ラベルからプレフィックスを判定する（`bug` → `fix/`、`enhancement` → `feature/` 等）

**命名規則:**
- 小文字のみ使用
- 単語はハイフン（`-`）で区切る
- 簡潔で分かりやすい英語名
- 日本語は英語に変換

### 3. ブランチ名・PR先の確認

デフォルトブランチを取得し、AskUserQuestionツールでブランチ名とPRのbase先をまとめてユーザーに提示する：

```bash
DEFAULT_BRANCH=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')
```

```
以下の内容で作成します：

  ブランチ名: {BRANCH_NAME}
  起点/PR先:  {DEFAULT_BRANCH}

（問題なければそのまま進めます。変更したい場合は指示してください）
```

### 4. ブランチ作成 + 空コミット + push

ユーザーの承認後、以下を実行する：

```bash
git fetch origin
git checkout -b {BRANCH_NAME} origin/${DEFAULT_BRANCH}
git commit --allow-empty --no-verify -m "chore: start work on {説明}"
git push -u origin {BRANCH_NAME}
```

### 5. ドラフトPR作成

**Issue番号指定時:**

```bash
gh pr create --draft --base ${DEFAULT_BRANCH} --title "{Issueタイトル}" --body "$(cat <<'EOF'
Closes #{番号}
EOF
)"
```

**テキスト指定時:**

```bash
gh pr create --draft --base ${DEFAULT_BRANCH} --title "{説明テキスト}" --body "$(cat <<'EOF'
（作業中）
EOF
)"
```

### 6. 完了メッセージ

```
✅ ブランチ `{BRANCH_NAME}` を作成し、ドラフトPRを作成しました（origin/${DEFAULT_BRANCH}起点）
   PR: {PR_URL}
```

## 注意事項

このスキルの役割はブランチの作成とドラフトPR作成まで。完了メッセージを表示したら**そこで終了する**。
ブランチ作成後に自動で実装作業やファイル編集を開始してはならない。次の作業はユーザーの指示を待つこと。
