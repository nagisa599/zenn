---
title: "Nextjs(App Router)✖️Nest(graphql)✖️Auth0を用いた認証基盤の作成　〜設計編〜"
emoji: "🦁"
type: "tech"
topics:
  - "graphql"
  - "nextjs"
  - "nestjs"
  - "auth0"
  - "approuter"
published: true
published_at: "2024-04-30 19:13"
---

# 目的

NextJs(フロントエンド）と Auth0 を用いた singOut と singIn の実装は、多くの記事で挙げられているため実装は、簡単であった。しかし自分で作成した api(バックエンド）と auth0 の連携は、記事があまりなく、１から設計、実装をすることに決めた。今回は、設計について説明したいと思う。

# 前提知識（cookie や jwt がわかる人は、飛ばしてください）

https://zenn.dev/naginagi124/articles/82b450b9c34b77

# 認証の流れ（イメージ）

![](https://storage.googleapis.com/zenn-user-upload/b9d30bf84141-20240430.png)

# 実際の設計

今回は、以下の構成で実装を行なった。

- NextJs App Router (フロントエンド）
- Nest Graphql (バックエンド）

Nextjs の client 側で acceeToken を持たない理由としては以下の記事を参考にしました。
https://zenn.dev/mh4gf/articles/urql-client-working-with-credential-in-nextjs
実践編でも説明しますが、app router では next-http-proxy-middleware と対応していなかったため api route に独自の proxy を作成しました。

![](https://storage.googleapis.com/zenn-user-upload/93d48977a46c-20240430.png)

# 参考記事

auth0 を使ったアーキテクチャの様々な種類が載っています。（多分、公式）
https://auth0.com/blog/ultimate-guide-nextjs-authentication-auth0/

# 実践編へ

実践編は、　言語 typescriot でフレームワークは NextJs App Router (フロントエンド）
、Nest Graphql (バックエンド）を使用しています。最新の技術を用いた実装になっていますので記事が出たらぜひよんでみてください
