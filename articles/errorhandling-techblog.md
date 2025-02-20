---
title: "エラーステータスには 2 種類ある!?"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 背景

# 前提

- REST API
- Graphql

# エラーステータスは 2 種類ある!?

一般的に言うと status と言ったら基本的に header の status のことを指します。
しかし、あるサービスには header の status 以外に body にも status が含まれるていることがあります。
以下の理由があります

1. 複数のエラーが返ってきた場合
2. header の status はフレームワークに依存しがち
3. graphql の場合は、すべて 200 を返す思想

# ベストプラクティス

REST API の場合
body で code と message で持つエラーオブジェクトを配列で返す
