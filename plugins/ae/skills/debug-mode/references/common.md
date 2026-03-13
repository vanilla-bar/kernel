# 共通リファレンス

Debug Mode で使用する共通フォーマットと設定。

## ログフォーマット

NDJSON（1行1JSON）:

```json
{"h":"H1","l":"label","v":{"key":"value"},"ts":1702567890123}
```

| フィールド | 意味 |
|-----------|------|
| h | 仮説ID（H1, H2, ...） |
| l | ラベル（このログが何を表すか） |
| v | 値（JSONシリアライズ可能な任意のデータ） |
| ts | タイムスタンプ（ミリ秒） |

## Region構文

デバッグ計装を囲むリージョン構文:

| 言語 | 開始 | 終了 |
|------|------|------|
| JS/TS | `//#region debug:H1` | `//#endregion` |

## ログコレクターサーバー

ブラウザからログを収集するためのHTTPサーバー:

```bash
node -e "require('http').createServer((q,s)=>{s.setHeader('Access-Control-Allow-Origin','*');s.setHeader('Access-Control-Allow-Methods','POST,OPTIONS');s.setHeader('Access-Control-Allow-Headers','Content-Type');if(q.method==='OPTIONS'){s.writeHead(204).end();return}let b='';q.on('data',c=>b+=c);q.on('end',()=>{require('fs').appendFileSync('debug.log',b+'\n');s.writeHead(204).end()})}).listen(4567,()=>console.log('Collector: http://localhost:4567'))"
```

## ログ分析

```bash
cat debug.log | jq .                              # 全件表示
jq 'select(.h == "H1")' debug.log                 # 仮説でフィルタ
cat debug.log | jq -s 'group_by(.h)'              # 仮説ごとにグルーピング
tail -f debug.log | jq .                          # リアルタイム
```
