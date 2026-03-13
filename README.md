# kernel

Claude Code 向けのプラグインマーケットプレイス。

## インストール

### CLI

```bash
# 1. マーケットプレイスを追加
/plugin marketplace add vanilla-bar/kernel

# 2. プラグインをインストール
/plugin install ae@kernel
```

### settings.json

`~/.claude/settings.json`（個人用）または `.claude/settings.json`（プロジェクト用）に追記：

```json
{
  "extraKnownMarketplaces": {
    "kernel": {
      "source": {
        "source": "github",
        "repo": "vanilla-bar/kernel"
      }
    }
  },
  "enabledPlugins": {
    "ae@kernel": true
  }
}
```

## プラグイン一覧

| プラグイン | 説明 |
|-----------|------|
| [ae](plugins/ae/) | Git/GitHub ワークフロー & 開発支援スキル集（15スキル） |

## ライセンス

[MIT](LICENSE)
