---
name: pr-ready
description: /pr-ready - ドラフトPRを更新してレビュー待ちに変更
disable-model-invocation: true
argument-hint: "[PR番号]"
---

# /pr-ready - ドラフトPR → レビュー待ち

ドラフトPRのタイトル・本文をコミット履歴から更新し、レビュー待ちに変更する。

**引数:** `/pr-ready [PR番号]`（省略時は現在のブランチのPR）

## 手順

### 1. 対象PRの特定

$ARGUMENTS でPR番号が指定されている場合はそのPRを対象にする。
指定がない場合は、現在のブランチに関連するPRを対象にする。

```bash
gh pr view {PR_NUMBER} --json number,title,body,isDraft,baseRefName
```

ドラフトでない場合はその旨を伝えて終了する。

### 2. 未コミットの変更確認

```bash
git status
```

未コミットの変更がある場合は警告し、先に `/commit` を案内する。

### 3. Spec整合性チェック

`git diff --name-only origin/{BASE_BRANCH}...HEAD` で変更ファイルを取得し、`src/features/{name}/` パターンから影響する機能名を抽出する。

対応する `docs/specs/{機能名}.md` が存在する機能について:

1. specとコードの両方を読み、以下の観点で比較する:
   - SPEC-IDテーブルの各項目に対応する実装があるか
   - コードにspec未記載の実装がないか

2. 整合性レポートを出力する:

```
## Spec整合性チェック

### {機能名} (docs/specs/{機能名}.md)
- ✅ SPEC-001: {説明} — 一致
- ⚠️ SPEC-003: {説明} — 差分あり（spec: ..., 実装: ...）
- 🆕 {実装の説明} — spec未記載
```

`stable` 項目に差分がある場合は特に警告する。

3. 差分がある場合、AskUserQuestionで対応を確認する:
   - specを更新する → 更新してコミット（変更した項目の安定度は `draft` に戻す）
   - コード側を修正する → 修正してコミット
   - 無視してPR作成を続行する

4. `draft` 項目がすべてコードと一致している場合、安定度を `stable` に昇格するか確認する。

**specが存在しない機能がある場合:**

```
📋 以下の機能にspecがありません:
- {機能名}（docs/specs/{機能名}.md）

specのドラフトを作成しますか？（スキップも可）
```

ユーザーが希望した場合、`docs/specs/_template.md` ベースでドラフトを生成し、コミットする。

### 4. ベースブランチのバックマージ

```bash
git fetch origin
git merge origin/{BASE_BRANCH}
```

コンフリクトが発生した場合は、**自動解消せず**以下を報告する：
- コンフリクトが発生しているファイル一覧
- 各ファイルの競合内容の概要
- 解消の方針案（ユーザーが判断できる情報）

コンフリクトの解消はユーザーの指示を待つこと。解消後に再度 `/pr-ready` を案内する。

### 5. Push

```bash
git push
```

### 6. PR内容の自動生成

コミット履歴からPRタイトルとサマリーを自動生成する：

```bash
git log origin/{BASE_BRANCH}..HEAD --oneline
```

- **タイトル**: 変更の要約（70文字以内）
- **本文**: PRテンプレートに従って生成する。既存の本文に `Closes #xxx` がある場合はそれを保持する

**テンプレートの読み込み:**

プロジェクト内の `.agents/pr-template.md` を探す。

- **存在する場合:** その内容をテンプレートとして使う
- **存在しない場合:** スキル内蔵の [references/pr-template.md](references/pr-template.md) をデフォルトとして使い、PR作成を進める。完了報告（手順8）で以下を追記する：

```
💡 プロジェクト固有のPRテンプレートを設定できます。
`.agents/pr-template.md` を作成すると、次回以降そちらが優先されます。
デフォルトのテンプレートをコピーして編集しますか？
```

ユーザーが希望した場合、デフォルトの内容を `.agents/pr-template.md` にコピーして編集を促す。

### 7. PR内容の確認とチェック項目の選択

AskUserQuestionツールを使い、PR内容の確認と確認事項の選択を**1回の質問で**行う：

```
## PR内容プレビュー

**タイトル:** {TITLE}
**ブランチ:** {CURRENT_BRANCH} → {BASE_BRANCH}

{BODY（「確認したこと」セクションは仮で全て未チェック）}

---

確認済みの項目を番号で選択してください（複数可、例: 1,2,3）：
  1. lint通過（npm run lint）
  2. ビルド通過（npm run build）
  3. 画面で動作確認済み
  4. ドキュメントリント通過（npm run lint:docs）

該当なしの場合は「なし」と入力してください。
PR内容の修正があれば、その旨を記載してください。
```

ユーザーの回答に基づいて：
- 選択された項目を `- [x]`、未選択を `- [ ]` として反映する
- 「なし」の場合は全て `- [ ]` のまま
- PR内容の修正指示があれば反映して再度確認する

### 8. PRを更新してレビュー待ちに変更

```bash
gh pr edit {PR_NUMBER} --title "{TITLE}" --body "$(cat <<'EOF'
{BODY}
EOF
)"
gh pr ready {PR_NUMBER}
```

### 9. 完了報告

PRのURLを表示する。
