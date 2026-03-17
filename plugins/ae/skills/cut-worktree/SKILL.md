---
name: cut-worktree
description: /cut-worktree - featureブランチ作成（worktree）+ ドラフトPR作成。複数指定でまとめて作成可能。
disable-model-invocation: true
argument-hint: "[#issue番号やテキスト（複数可）]"
---

# /cut-worktree - featureブランチ作成（worktree）+ ドラフトPR作成

GitHubのデフォルトブランチを起点にfeatureブランチをworktreeとして作成し、ドラフトPRまで一気に作成する。
メインの作業ディレクトリを切り替えずに並行作業したい場合に使う。複数指定でまとめて作成可能。

**引数の例:**
- `/cut-worktree #123` — 1つ
- `/cut-worktree #123, #124` — 複数（カンマ区切り）
- `/cut-worktree ログイン画面の修正` — 1つ
- `/cut-worktree #123, ログイン画面の修正` — Issue + テキスト混在
- `/cut-worktree ログイン修正, CSV追加` — テキスト複数
- 省略時は対話で聞く

## 手順

### 1. 引数のパース

$ARGUMENTS をカンマ（`,`）で分割し、**作成項目リスト**を作る：

**パース規則:**
- カンマ（`,`）で区切って項目に分割する（1つだけならカンマなしでOK）
- 各項目の前後の空白をトリムする
- `#` + 数字のパターン → Issue番号
- それ以外 → テキスト（そのままブランチの説明として使用）
- 引数なしの場合は対話で聞く

**パース例:**

| 入力 | 作成項目リスト |
|------|---------------|
| `#123` | [Issue #123] |
| `#123, #124` | [Issue #123, Issue #124] |
| `ログイン画面の修正` | [テキスト: ログイン画面の修正] |
| `ログイン修正, CSV追加` | [テキスト: ログイン修正, テキスト: CSV追加] |
| `#123, ログイン画面の修正` | [Issue #123, テキスト: ログイン画面の修正] |
| `#123, ログイン修正, #124` | [Issue #123, テキスト: ログイン修正, Issue #124] |

### 2. Issue情報の取得

作成項目リストにIssue番号が含まれる場合、各Issueの情報を取得する：

```bash
gh issue view {番号} --json title,labels,body
```

Issueのタイトルをブランチの説明・PRタイトルとして使用する。ラベル情報はプレフィックス判定に活用する（`bug` → `fix/` 等）。

### 3. ブランチ名の自動生成

各項目について、以下のルールに従ってブランチ名を生成する：

**プレフィックス判定:**
- 新機能の追加 → `feature/`
- バグ修正 → `fix/`
- リファクタリング → `refactor/`
- ドキュメント → `docs/`
- テスト → `test/`
- その他（設定変更、CI等） → `chore/`

**Issue番号指定時の命名規則:**
- ブランチ名にIssue番号を含める（例: `feature/123-user-list`）
- ラベルからプレフィックスを判定する（`bug` → `fix/`、`enhancement` → `feature/` 等）

**命名規則:**
- 小文字のみ使用
- 単語はハイフン（`-`）で区切る
- 簡潔で分かりやすい英語名
- 日本語は英語に変換

### 4. ブランチ名・PR先の確認

デフォルトブランチを取得し、AskUserQuestionツールで**全項目のブランチ名とPRのbase先をまとめて**確認する：

```bash
DEFAULT_BRANCH=$(gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name')
```

**1項目の場合:**
```
以下の内容でworktreeを作成します：

  ブランチ名: {BRANCH_NAME}
  作成先: ../worktrees/{BRANCH_NAME}
  起点/PR先:  {DEFAULT_BRANCH}

（問題なければそのまま進めます。変更したい場合は指示してください）
```

**2項目以上の場合:**
```
以下のworktreeをまとめて作成します（起点/PR先: {DEFAULT_BRANCH}）：

  1. feature/123-user-list   ← Issue #123: ユーザー一覧画面の追加
     作成先: ../worktrees/feature/123-user-list

  2. feature/login-fix       ← ログイン画面の修正
     作成先: ../worktrees/feature/login-fix

（問題なければそのまま進めます。変更したい場合は番号と新しい名前を指定してください）
```

### 5. worktree作成 + 空コミット + push + ドラフトPR

承認後、各項目に対して以下を順番に実行する：

```bash
git fetch origin  # 初回のみ

# --- 各項目について繰り返し ---

# worktree作成
git worktree add -b {BRANCH_NAME} ../worktrees/{BRANCH_NAME} origin/${DEFAULT_BRANCH}

# .env.localコピー
if [ -f .env.local ]; then
  cp .env.local ../worktrees/{BRANCH_NAME}/.env.local
fi

# 空コミット + push
cd ../worktrees/{BRANCH_NAME}
git commit --allow-empty --no-verify -m "chore: start work on {説明}"
git push -u origin {BRANCH_NAME}

# ドラフトPR作成
gh pr create --draft --base ${DEFAULT_BRANCH} --title "{タイトル}" --body "$(cat <<'EOF'
{本文}
EOF
)"
```

**PR本文の決定:**
- Issue由来の項目: `Closes #{番号}`
- テキスト由来の項目: `（作業中）`

### 6. 完了メッセージ

`pwd` で取得した絶対パスをもとに、cdコマンドを絶対パスで案内する。

**1項目の場合:**
```
✅ Worktree `{BRANCH_NAME}` を作成し、ドラフトPRを作成しました（origin/${DEFAULT_BRANCH}起点）

  PR: {PR_URL}

新しいターミナルで以下を実行してください：
  cd {WORKTREE_ABSOLUTE_PATH}
```

**2項目以上の場合:**
```
✅ {N}個のworktreeを作成しました（origin/${DEFAULT_BRANCH}起点）

  1. feature/123-user-list  PR: {PR_URL_1}
     cd {WORKTREE_1_ABSOLUTE_PATH}

  2. feature/login-fix      PR: {PR_URL_2}
     cd {WORKTREE_2_ABSOLUTE_PATH}
```

## 注意事項

このスキルの役割はworktreeの作成とドラフトPR作成まで。完了メッセージを表示したら**そこで終了する**。
ブランチ作成後に自動で実装作業やファイル編集を開始してはならない。次の作業はユーザーの指示を待つこと。
