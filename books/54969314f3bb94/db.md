---
title: "データベース"
free: true
---

## MYSQL のコンテナを立ち上げる

```yaml
services:
  db:
    image: mysql:8.0 # imageのコンテナ名を利用、ネット上にあるimageを使う。
    container_name: mysql-db # コンテナ名を指定
    environment: # 環境変数を指定
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: "mysql_template" # データベース名
      MYSQL_USER: test-user
      MYSQL_PASSWORD: test-pass
      TZ: "Asia/Tokyo" # タイムゾーンを指定
    ports:
      - 3306:3306 # ポートを指定
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"] # ヘルスチェックを指定
      interval: 10s # チェック間隔
      timeout: 5s # タイムアウト
      retries: 3 # リトライ回数
  backend:
    build: .
    container_name: grahql-backend
    env_file:
      - .env
    command: sh -c "sh ./build/init.sh"
    volumes:
      - ../:/go/src/
    # 以下の依存関係を追加
    depends_on:
      db: # dbが起動してからbackendを起動
        condition: service_healthy
    ports:
      - 3001:3001
```

```bash
cd ./build && docker compose up
```

## MYSQL に接続を行う。

## backend サーバから migration を行う。

```env
DB_USER=root
DB_PASSWORD=
DB_HOST=host.docker.internal
DB_PORT=3306
DB_NAME=mysql_template
ENV=development
PORT=3001
```

##
