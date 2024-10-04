---
title: "backendサーバ"
free: true
---

## backend の image を作成

```dockerfile
FROM golang:1.22.4

# ログに出力する時間をJSTにするため、タイムゾーンを設定
ENV TZ /usr/share/zoneinfo/Asia/Tokyo

ENV ROOT=/go/src/
WORKDIR ${ROOT}

COPY . .
EXPOSE 3001

# migration用にgooseをインストール
RUN go install github.com/pressly/goose/v3/cmd/goose@v3.21.1

# Airをインストールし、コンテナ起動時に実行する
RUN go install github.com/air-verse/air@v1.52.3
CMD ["air", "-c", "./build/.air.toml"]
```

```toml
root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  args_bin = []
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ./cmd"
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  post_cmd = []
  pre_cmd = []
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = false

[proxy]
  app_port = 0
  enabled = false
  proxy_port = 0

[screen]
  clear_on_rebuild = false
  keep_scroll = true

```

```bash
cd ./build && docker compose build --no-cache
```

## image からコンテナを立ち上げる

```bash
cd ./build && docker compose up
```
