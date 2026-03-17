---
name: issue-open
description: GitHub Issueをサッと作成する。ユーザーが「issueに出して」「issueに上げて」「issue立てて」等と言った時にも使用する。
argument-hint: "[Issueの説明]"
---

# /issue-open - GitHub Issue作成

やることをIssueとして積む。

**引数:** `/issue-open <Issueの説明>`（省略時は聞く）

## 手順

### 1. 入力の受け取り

$ARGUMENTS またはユーザーの発言をそのまま使う。

入力がない場合のみ聞く：

```
Issueの内容を教えてください。
例: 「一覧画面でソートが効かない」「CSVエクスポート機能の追加」
```

入力が曖昧で「何をやるか」が読み取れない場合は、それだけを聞く。設計や実装方針には踏み込まない。

判断基準: 後から見て着手できる程度に「何を」が伝わるか。「どうやって」は不要。

### 2. Issue内容の生成

入力から以下を生成する：

- **タイトル**: 簡潔な要約（70文字以内）
- **ラベル**: `bug` / `enhancement` / `documentation` / `question` から選ぶ
- **本文**: ユーザーの入力を整理する程度。情報が少なければ短くてよい

### 3. 確認

AskUserQuestionで提示し、承認を得る：

```
**タイトル:** {TITLE}
**ラベル:** {LABELS}

{BODY}

この内容でIssueを作成してよいですか？
```

修正があれば反映して再確認。

### 4. 作成

```bash
gh issue create --title "{TITLE}" --body "$(cat <<'EOF'
{BODY}
EOF
)" --label "{LABELS}"
```

ラベルなしの場合は `--label` を省略する。

### 5. 完了

IssueのURLを表示する。
