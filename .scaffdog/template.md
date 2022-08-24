---
name: 'template'
root: '.'
output: '../../'
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

# `{{ inputs.name | lower }}/documents/designs/design.md`

```md
```

# `{{ inputs.name | lower }}/documents/designs/images/.gitkeep`

```
```

# `{{ inputs.name | lower }}/kustomize/base/configmap.env`

```
LOG_LEVEL=debug
BROKER_ADDRESSES=kafka:9092
CONSUME_TOPIC=
PRODUCE_TOPIC=
CONSUMER_GROUP_ID=
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
```

# `{{ inputs.name | lower }}/kustomize/base/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ inputs.name | lower }}
spec:
  selector:
    matchLabels:
      app: {{ inputs.name | lower }}
  template:
    metadata:
      labels:
        app: {{ inputs.name | lower }}
    spec:
      shareProcessNamespace: true
      containers:
        - name: {{ inputs.name | lower }}
          image: localhost:5000/messaging/{{ inputs.name | lower }}:xxxxx
          imagePullPolicy: IfNotPresent
          resources:
            requests:
              memory: '100Mi'
              cpu: '100m'
            limits:
              memory: '200Mi'
              cpu: '200m'
          envFrom:
            - configMapRef:
                name: {{ inputs.name | lower }}
            - secretRef:
                name: {{ inputs.name | lower }}
```

# `{{ inputs.name | lower }}/kustomize/base/kustomization.yaml`

```yaml
namespace: canal-messaging
resources:
  - deployment.yaml
  - horizontalpodautoscaler.yaml
configMapGenerator:
  - name: {{ inputs.name | lower }}
    envs:
      - ./configmap.env
```

# `{{ inputs.name | lower }}/kustomize/base/secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ inputs.name | lower }}
data:
  # postgres
  POSTGRES_PASSWORD: cG9zdGdyZXM=
```

# `{{ inputs.name | lower }}/kustomize/base/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ inputs.name | lower }}
spec:
  selector:
    app: {{ inputs.name | lower }}
  type: ClusterIP
  ports:
  - port: 80
    name: http
    protocol: TCP
    targetPort: {{ inputs.name | lower }}-port
```

# `{{ inputs.name | lower }}/kustomize/overlays/local/configmap.env`

```
PORT=3000
DB_HOST=postgres.canal-gate.svc.cluster.local
DB_PORT=5432
DB_NAME=scorpio
DB_USER=postgres
DB_PASSWORD=postgres1234
```

# `{{ inputs.name | lower }}/kustomize/overlays/local/development.json`

```json
[
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/image",
    "value": "k3d-myregistry:15000/messaging/{{ inputs.name | lower }}:local-test"
  },
  {
    "op": "replace",
    "path": "/spec/template/spec/containers/0/imagePullPolicy",
    "value": "Always"
  }
]
```

# `{{ inputs.name | lower }}/kustomize/overlays/local/image.yaml`

```yaml
apiVersion: builtin
kind: ImageTagTransformer
metadata:
  name: {{ inputs.name | lower }}
imageTag:
  name: localhost:5000/messaging/{{ inputs.name | lower }}
  newName: 636082426924.dkr.ecr.ap-northeast-1.amazonaws.com/messaging/{{ inputs.name | lower }}
  newTag: 20220802-01
```

# `{{ inputs.name | lower }}/kustomize/overlays/local/kustomization.yaml`

```yaml
namespace: canal-gate
bases:
- ../../base
resources:
- postgres-storage.yaml
- postgres-secret.yaml
- postgres-deployment.yaml
- postgres-service.yaml
- ingress.yaml
configMapGenerator:
- name: {{ inputs.name }}
  behavior: replace
  envs:
  - ./configmap.env
```

# `{{ inputs.name | lower }}/kustomize/overlays/local/secret.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ inputs.name }}
type: Opaque
data:
  # password=postgres
  POSTGRES_PASSWORD: cG9zdGdyZXM=
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/iam/secretstore/.env`

```
AWS_REGION="ap-northeast-1"
POLICY_NAME="canal-prod-messaging-{{ inputs.name }}-policy"
ROLE_NAME="canal-prod-messaging-{{ inputs.name }}-role"
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/iam/secretstore/Earthfile`

```
# Earthfile
# create iam-role for application

FROM ubuntu:20.04
WORKDIR /earthly
COPY . .

RUN ln -snf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && echo Asia/Tokyo > /etc/timezone

RUN apt-get -y update && apt-get -y install \
  unzip \
  curl

create-application-iam:
  BUILD +attach-policy

delete-application-iam:
  BUILD +delete-iam-policy


aws-cli-settings:
  FROM +base
  ARG AWS_ACCESS_KEY
  ARG AWS_SECRET_KEY
  ARG AWS_REGION="$AWS_REGION"
  ARG VERSION=2.2.8
  RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64-"$VERSION".zip" -o "awscliv2.zip"
  RUN unzip awscliv2.zip
  RUN ./aws/install --update
  RUN mkdir -m 755 ~/.aws
  RUN touch ~/.aws/config
  RUN touch ~/.aws/credentials
  RUN chmod 600 ~/.aws/config
  RUN chmod 600 ~/.aws/credentials
  RUN echo '[default]' >> ~/.aws/credentials
  RUN echo "aws_access_key_id = "$AWS_ACCESS_KEY"" >> ~/.aws/credentials
  RUN echo "aws_secret_access_key = "$AWS_SECRET_KEY"" >> ~/.aws/credentials
  RUN echo '[default]' >> ~/.aws/config
  RUN echo "region = $AWS_REGION" >> ~/.aws/config
  RUN echo 'output = json' >> ~/.aws/config

create-iam-policy:
  FROM +aws-cli-settings
  ARG POLICY_NAME="$POLICY_NAME"
  RUN aws iam create-policy --policy-name "$POLICY_NAME" --policy-document file://iam-policy.json

delete-iam-policy:
  FROM +delete-iam-role
  ARG POLICY_NAME="$POLICY_NAME"
  RUN IAM_POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName==\`$POLICY_NAME\`].{ARN:Arn}" --output text) ; \
      aws iam delete-policy --policy-arn "$IAM_POLICY_ARN"

create-iam-role:
  ARG ROLE_NAME="$ROLE_NAME"
  FROM +create-iam-policy
  RUN aws iam create-role --role-name "$ROLE_NAME" --assume-role-policy-document file://role-trust-policy.json

delete-iam-role:
  FROM +detach-policy
  ARG ROLE_NAME="$ROLE_NAME"
  RUN aws iam delete-role --role-name "$ROLE_NAME"

attach-policy:
  FROM +create-iam-role
  ARG ROLE_NAME="$ROLE_NAME"
  ARG POLICY_NAME="$POLICY_NAME"
  RUN IAM_POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName==\`$POLICY_NAME\`].{ARN:Arn}" --output text) ; \
      aws iam attach-role-policy --role-name "$ROLE_NAME" --policy-arn "$IAM_POLICY_ARN"

detach-policy:
  FROM +aws-cli-settings
  ARG ROLE_NAME="$ROLE_NAME"
  ARG POLICY_NAME="$POLICY_NAME"
  RUN IAM_POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName==\`$POLICY_NAME\`].{ARN:Arn}" --output text) ; \
      aws iam detach-role-policy --role-name "$ROLE_NAME" --policy-arn "$IAM_POLICY_ARN"

update-policy:
  FROM +aws-cli-settings
  RUN IAM_POLICY_ARN=$(aws iam list-policies --query "Policies[?PolicyName==\`$POLICY_NAME\`].{ARN:Arn}" --output text) ; \
      aws iam create-policy-version --policy-arn "$IAM_POLICY_ARN" --policy-document file://iam-policy.json --set-as-default
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/iam/secretstore/iam-policy.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetResourcePolicy",
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "ssm:GetParameter",
        "secretsmanager:ListSecretVersionIds"
      ],
      "Resource": [
        "arn:aws:secretsmanager:*:636082426924:secret:*",
        "arn:aws:ssm:*:636082426924:parameter/*"
      ]
    }
  ]
}
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/iam/secretstore/role-trust-policy.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::636082426924:oidc-provider/oidc.eks.ap-northeast-1.amazonaws.com/id/0C730B1FC2A2E6FBFBB4028355C3D273"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.ap-northeast-1.amazonaws.com/id/0C730B1FC2A2E6FBFBB4028355C3D273:sub": "system:serviceaccount:canal-messaging:secretstore"
        }
      }
    }
  ]
}
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/configmap.env`

```
LOG_LEVEL=debug
BROKER_ADDRESSES=b-3.canal-prod-msk-cl.axdx7h.c4.kafka.ap-northeast-1.amazonaws.com:9092,b-1.canal-prod-msk-cl.axdx7h.c4.kafka.ap-northeast-1.amazonaws.com:9092,b-2.canal-prod-msk-cl.axdx7h.c4.kafka.ap-northeast-1.amazonaws.com:9092
CONSUME_TOPIC=canal.message.origin.v1
PRODUCE_TOPIC=canal.message.parsed.v1
CONSUMER_GROUP_ID=canal.messaging.{{ inputs.name | lower }}
POSTGRES_HOST=canal-prod-postgres-custer.cluster-cgceiv43vnkv.ap-northeast-1.rds.amazonaws.com
POSTGRES_PORT=5432
POSTGRES_DB=canalcore
POSTGRES_USER=canal
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/externalsecret.yaml`

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  # ExternalSecretの名前。アプリ名を指定する。
  name: {{ inputs.name | lower }}
spec:
  refreshInterval: 1h
  secretStoreRef:
    # secretstore.yamlで指定したSecretStoreのnameを指定する。基本は変更しない。
    name: secretstore
    kind: SecretStore
  target:
    # secretの名前。
    # アプリ側で指定されているsecretの名前があればそちらに合わせる。ここに付けた名前とアプリ側の指定が一致していれば良い。
    name: {{ inputs.name | lower }}
    creationPolicy: Owner
  data:
  # secretのキー名
  - secretKey: POSTGRES_PASSWORD
    remoteRef:
      # secretのキーに入る値。パラメータストアやsecrets managerに保存しているキー名を指定する。
      key: production/postgres/canal/password

```

# `{{ inputs.name | lower }}/kustomize/overlays/production/image.yaml`

```yaml
apiVersion: builtin
kind: ImageTagTransformer
metadata:
  name: {{ inputs.name | lower }}
imageTag:
  name: localhost:5000/messaging/{{ inputs.name | lower }}
  newName: 636082426924.dkr.ecr.ap-northeast-1.amazonaws.com/messaging/{{ inputs.name | lower }}
  newTag: 20220802-01
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/kustomization.yaml`

```yaml
namespace: canal-messaging
bases:
  - ../../base
resources:
  - ./secretstore.yaml
  - ./externalsecret.yaml
transformers:
  - ./image.yaml
configMapGenerator:
  - name: {{ inputs.name | lower }}
    behavior: replace
    envs:
      - ./configmap.env
```

# `{{ inputs.name | lower }}/kustomize/overlays/production/secretstore.yaml`

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: secretstore
spec:
  # 機密情報を保管しているproviderに対する設定を記載する
  provider:
    aws:
      # `ParameterStore` or `SecretsManager` が指定可能
      service: SecretsManager
      # パラメータを保存しているリージョン名
      region: ap-northeast-1
      # AWSに対する認証を行うためのリソースを指定する
      auth:
        jwt:
          serviceAccountRef:
            name: secretstore
---
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    # role/の後は[.envで指定したROLE_NAME]を指定する。
    eks.amazonaws.com/role-arn: arn:aws:iam::636082426924:role/canal-prod-messaging-{{ inputs.name | lower }}-role
  name: secretstore
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

# `{{ inputs.name | lower }}/docker-compose.yaml`

```yaml
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
