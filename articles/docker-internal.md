---
title: "localhostとinternal"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 目的

開発環境で開発をしている際に backend のコンテナから frontend のコンテナに接続する際に一部で以下のエラーが発生した。この原因を調べたとところ興味深い事実が判明したので共有をしたい。

```env
DB_USER=root
DB_PASSWORD=
DB_HOST=host.docker.internal
DB_PORT=3306
DB_NAME=mysql_template
ENV=development
PORT=3001
```

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
    depends_on:
      db: # dbが起動してからbackendを起動
        condition: service_healthy
    ports:
      - 3001:3001
```

```go

func NewDatabaseHandler() *DatabaseHandler {
	 dsn := fmt.Sprintf("%s:%s@tcp(%s:%s)/%s?parseTime=true&loc=Local&charset=utf8",
        os.Getenv("DB_USER"),
        os.Getenv("DB_PASSWORD"),
        os.Getenv("DB_HOST"),
        os.Getenv("DB_PORT"),
        os.Getenv("DB_NAME"))
	db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatal("Failed to connect to database",err)
	}
	if os.Getenv("ENV") == "development" {
		db.Logger = db.Logger.LogMode(logger.Info)
	}

	return &DatabaseHandler{DB: db}
}

```

```bash
grahql-backend  | [error] failed to initialize database, got error Error 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

##　　解決策

環境変数の DB_HOTS を host.docker.internal から localhost に変更をする。

## 原因
