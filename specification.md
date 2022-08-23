# scaffdog 仕様書

## 目的

新規アプリケーション開発時に、他のアプリから、コピペせずとも、ファイルを生成し開発がスタートできる。

## 実行

```sh
npx scaffdog generate ファイル名
```

## 生成されるファイル

- `app name /documents/designs/specification.md`
- `app name /src/app.ts`
- `app name /src/drivers/logger.ts`
- `app name /src/usecase/index.ts`
- `app name /.dockerignore`
- `app name /.eslintrc.js`
- `app name /.prettierc`
- `app name /.gitignore`
- `app name /.prettierignore`
- `app name /docker-compose.yml`
- `app name /Dockerfile`
- `app name /tsconfig.eslint.json`
- `app name /README.md`
