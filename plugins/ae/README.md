# ae

Git/GitHub ワークフロー & 開発支援スキルプラグイン。

## 前提条件

- [GitHub CLI (`gh`)](https://cli.github.com/) — Issue/PR 操作系スキルで使用

## スキル一覧

### Git 操作

| スキル | コマンド | 説明 |
|--------|---------|------|
| [commit](skills/commit/SKILL.md) | `/commit` | 変更内容を分析し、意味的なまとまりで分割コミット |
| [cut](skills/cut/SKILL.md) | `/cut` | feature ブランチ作成 + ドラフト PR 作成 |
| [cut-worktree](skills/cut-worktree/SKILL.md) | `/cut-worktree` | worktree で feature ブランチ作成 + ドラフト PR 作成。複数同時作成可 |
| [prune](skills/prune/SKILL.md) | `/prune` | マージ済み・リモート削除済みの不要ブランチを整理 |

### PR 操作

| スキル | コマンド | 説明 |
|--------|---------|------|
| [pr-list](skills/pr-list/SKILL.md) | `/pr-list` | PR 一覧を表示 |
| [pr-ready](skills/pr-ready/SKILL.md) | `/pr-ready` | ドラフト PR のタイトル・本文をコミット履歴から更新し、レビュー待ちに変更 |
| [pr-review](skills/pr-review/SKILL.md) | `/pr-review` | PR の差分を観点ベースでコードレビュー |
| [pr-merge](skills/pr-merge/SKILL.md) | `/pr-merge` | PR をマージしてローカルを整理 |

### Issue 操作

| スキル | コマンド | 説明 |
|--------|---------|------|
| [issue-list](skills/issue-list/SKILL.md) | `/issue-list` | GitHub Issue 一覧を表示 |
| [issue-open](skills/issue-open/SKILL.md) | `/issue-open` | Issue をサッと作成 |

### レビュー・調査

| スキル | コマンド | 説明 |
|--------|---------|------|
| [think](skills/think/SKILL.md) | `/think` | 要件ヒアリング + 批判的コード調査。制約や前提の誤りを洗い出す |
| [self-review](skills/self-review/SKILL.md) | `/self-review` | サブエージェントによるセルフレビュー。仕様書・計画書・コード差分を独立視点でチェック |

### デバッグ

| スキル | コマンド | 説明 |
|--------|---------|------|
| [debug-mode](skills/debug-mode/SKILL.md) | `/debug-mode` | ログ計装による構造化デバッグ。推測ではなく実行時証拠で原因を特定 |

### 引き継ぎ

| スキル | コマンド | 説明 |
|--------|---------|------|
| [handover](skills/handover/SKILL.md) | `/handover` | 次のセッション向けの引き継ぎ書を生成 |
| [takeover](skills/takeover/SKILL.md) | `/takeover` | 引き継ぎ書を読み込んで作業を再開 |
