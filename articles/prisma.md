---
title: "prisma migrationを途中から導入した話"
emoji: "🐡"
type: "tech"
topics: []
published: false
---

## 目的

以前は、ローカルで開発をしており、データベースを更新する際は、npx prism db push(prisma_shema の型を強制的に DB に反映させること)をしていた。しかし、デプロイ後、DB を npx primsa db push で更新するとデータが消える恐れがあった。そこでマイグレーションを導入することにした。今回は、ローカル環境とデプロイ環境を分けて説明をすることにした。

## マイグレーションとは

まるまる

## migration 導入　(ローカル環境)

- migraion の初期設定を行う。

```bash
# prsima migrationの初期設定
npx prisma migration init
```

- 以下のファイルが生成される

```bash
.
└─ prisma
   ├── migration
   |    └── 202401010000_init
   |      └── migration.sql
   └── schema.prisma
```

## migration 導入後(ローカル環境)

- prisma_schema のファイルを変更

- migration の実行

```bash
# migrationの作成
npx prisma migrate dev --name <マイグレーション名>
```

:::message

### おすすめ命名規則

#### migration の名前は[操作]_[粒度]_[テーブル名]

- 操作　= add(追加) delete(削除) edit(編集)
- 粒度 = table(テーブル) column(カラム)
- テーブル名
  :::

## migration 導入　(デプロイ環境)

- migration の初期設定を DB に反映

```bash
 npx prisma db execute --file ./prisma/migrations/20240713025925_init/migration.sql
```

## migration 　導入後 (デプロイ環境)

- 変更した migaration を DB に反映していく。

```bash
npx prisma migration deploy
```

## その他

migration deploy と migration dev の違い

```bash
# 差分の変更
npx prisma migration deplou

# 全てのmigrationを変更
npx prisma migration dev
```

## まとめ

migration を導入したおかげで DB の変更によるデータが消えることを防ぐことができた。

## 今後

現状では、migration を手動で行なっているが、今後は、CI/CD を導入して miration の自動化を行っていきたい。
