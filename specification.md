# scaffdog 仕様書

## 目的

新規アプリケーション開発時に、他のアプリから、コピペせずとも、ファイルを生成し開発がスタートできる。

## 実行

### scaffdog を実行

`/cube/tools/template-nodejs`

```sh
npm run scaffdog
```

### 階層を選択

`/cube/tools/template-nodejs`

```sh
? Please select the output destination directory. (Use arrow keys or type to search)
  .
❯ ../../  => こちらを選択
```

`cube/` 配下に生成し、 `mv` コマンドで移動します

### アプリケーション名を入力

`/cube/tools/template-nodejs`

```sh
? Please enter a app name. appName
```

### 階層を移動

`/cube`

```sh
mv ./appName ./services/messaging
```

## 生成されるファイル

- `appName/documents/designs/specification.md`
- `appName/src/app.ts`
- `appName/src/drivers/logger.ts`
- `appName/src/usecase/index.ts`
- `appName/.dockerignore`
- `appName/.eslintrc.js`
- `appName/.prettierc`
- `appName/.gitignore`
- `appName/.prettierignore`
- `appName/docker-compose.yml`
- `appName/Dockerfile`
- `appName/tsconfig.eslint.json`
- `appName/README.md`
