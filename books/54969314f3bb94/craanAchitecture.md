---
title: "クリーンアーキテクチャーとは"
free: true
---

## クリーンアーキテクチャとは

##　 infrustructure/router

## infrustructure/database_handler

型の変換の処理を行う。
行うことは以下の２つである

- router から送れれた来た型を domain のインスタンスにして usecase にわたす処理
- usecase から渡され来たものを api の型に変換して router にわたす処理　(今回は grpc の proto で設定した型に変換を行なっている)

## internal/domain

-

## internal/usecase

## internal/repository

-　 DB にアクセスする際の処理を書く。
