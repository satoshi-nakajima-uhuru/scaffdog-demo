# scaffdog 仕様書

## 目的

新規アプリケーション開発時に、他のアプリから、コピペせずとも、ファイルを生成し開発がスタートできる。

## 実行

```sh
npx scaffdog generate ファイル名
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
