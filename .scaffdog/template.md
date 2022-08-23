---
name: 'template'
root: '.'
output: '**/*'
ignore: []
questions:
  name: 'Please enter a app name.'
---

# `{{ inputs.name | lower }}/documents/designs/specification.md`

```md
# 仕様

## 目的

## 方針

## 機能
```

# `{{ inputs.name | lower }}/src/app.ts`

```ts
```

# `{{ inputs.name | lower }}/src/drivers/logger.ts`

```ts
```

# `{{ inputs.name | lower }}/src/usecase/index.ts`

```ts
```

# `{{ inputs.name | lower }}/.dockerignore`

```
documents/
kustomize/
node_modules/
```

# `{{ inputs.name | lower }}/.eslintrc.js`

```js
module.exports = {
  root: true,
  env: {
    es6: true,
    node: true,
  },
  parser: '@typescript-eslint/parser',
  parserOptions: {
    sourceType: 'module',
    ecmaVersion: 2015,
    tsconfigRootDir: __dirname,
    project: ['./tsconfig.eslint.json'],
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking',
    'prettier',
  ],
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
  },
};
```

# `{{ inputs.name | lower }}/.prettierc`

```json
{
  "semi": true,
  "arrowParens": "always",
  "bracketSpacing": true,
  "singleQuote": true,
  "endOfLine": "auto"
}
```

# `{{ inputs.name | lower }}/.gitignore`

```
/node_modules/
/dist/
.vscode
report
coverage
```

# `{{ inputs.name | lower }}/.prettierignore`

```
/dist
documents
node_modules
package.json
package-lock.json
tsconfig.json
tsconfig.eslint.json
```

# `{{ inputs.name | lower }}/docker-compose.yml`

```yml
```

# `{{ inputs.name | lower }}/Dockerfile`

```
FROM node:16-alpine
```

# `{{ inputs.name | lower }}/tsconfig.eslint.json`

```json
{
  "extends": "./tsconfig.json",
  "include": [
    "src/**/*.ts",
    "integration-test/**/*.ts",
    ".eslintrc.js"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

# `{{ inputs.name | lower }}/README.md`

```md
# {{ inputs.name | lower }}
```
