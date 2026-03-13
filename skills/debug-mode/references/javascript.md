# JavaScript / TypeScript

Debug Mode の計装パターン。

## HTTP方式（ブラウザ / React）

コレクターサーバーの起動が必要。ブラウザから自動でログが収集される。

### ワンライナー

```typescript
//#region debug:H1
fetch('http://localhost:4567',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify({h:'H1',l:'label',v:{key:value},ts:Date.now()})}).catch(()=>{});
//#endregion
```

### 展開形

```typescript
//#region debug:H1
fetch('http://localhost:4567', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    h: 'H1',
    l: 'user_state',
    v: { userId, cart, timestamp: new Date().toISOString() },
    ts: Date.now()
  })
}).catch(() => {});
//#endregion
```

## console.log方式（Lambda / サーバーサイド）

CloudWatchに出力される。ユーザーがログをコピーして `debug.log` に貼れば、同じ `jq` コマンドで分析できる。

### ワンライナー

```typescript
//#region debug:H1
console.log(JSON.stringify({h:'H1',l:'label',v:{key:value},ts:Date.now()}));
//#endregion
```

### 展開形

```typescript
//#region debug:H1
console.log(JSON.stringify({
  h: 'H1',
  l: 'lambda_state',
  v: { event, context: context.functionName },
  ts: Date.now()
}));
//#endregion
```
