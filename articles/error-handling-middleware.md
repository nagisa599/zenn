---
title: "忘れっぽい僕には、middlewareでエラーハンドリング"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["エラーハンドリング", "middleware", "セキュリティ", "エラー", "golang"]
published: true
---

## 背景

最近のセキュリティ診断で、エラーをそのままハンドリングせずに返してしまうという問題が指摘された 😱。 これをきっかけに`なぜエラーハンドリング`がセキュリティと関係しているのかを調べることにした。

## なぜエラーハンドリングが必要なのか

エラーハンドリングは主に以下の２点で非常に重要である

### 1. バックエンドのシステム構成の漏洩防止

バックエンドのシステム構成（ディレクトリーやファイルなど）が外部に漏れるリスクがある。例えば、エラーハンドリングを忘れてレスポンスのボディにスタックトレースを含めてしまうと、システムの構成情報が露呈してしまう。

#### 例:

エラーハンドリングを忘れた結果、レスポンスボディにスタックトレースが含まれてしまったケース。
![エラーハンドリング忘れ例](/images/errorhandling/image1.png)

### 2. クロスサイトスクリプティング (XSS) の防止

クロスサイトスクリプティング（XSS）は、悪意のあるスクリプトが信頼されたウェブサイトに注入されるタイプのインジェクション攻撃だる。入力値を適切にサニタイズせずにエラーメッセージとしてそのまま返してしまうと、攻撃のリスクが生じる

#### 例:

ユーザー入力値をそのままエラーメッセージとして返してしまったケース。
![XSS攻撃例](/images/errorhandling/image3.png)

# **なぜ Middleware を使うのか?**

一言で言うと,**「必ず通るから」**です。エラーが発生した時に必ず Middleware を通るようにして、Response に返すエラーのハンドリングを行います。
そうすることで、エラーを返す際も Middleware が整形してくれます。

Go Echo ライブラリでは、公式にエラーハンドリングの Middleware が用意されている(さすが 👏)
詳細はこちら：[Echo Error Handling Documentation](https://echo.labstack.com/docs/error-handling)
https://echo.labstack.com/docs/error-handling

## 実装例

カスタムエラーを実装しており、指定したエラー以外の場合は、500 の Exception エラーを返すようにしています。

```go
func CustomHTTPErrorHandler(err error, c echo.Context) {
	if me, ok := err.(*Error.CustomerError); ok {
		// カスタムエラーの場合
        // response
		c.JSON(me.Code, echo.Map{
			"code":    me.Code,
			"message": me.Msg,
		})
	} else {
		// その他のエラーの場合、Exception Errorを返す
		c.JSON(http.StatusInternalServerError, map[string]string{
            "code": 500
			"message": "Exception Error",
		})
	}
}
```

# 結論

これらの例からも分かるように、適切なエラーハンドリングはセキュリティの観点からも極めて重要ある。システムの安全性を確保するためにも、常にエラーハンドリングの実装は必要となる。忘れても大丈夫なのように、`必ずエラーハンドリングはmiddlewareを通す`ことをお勧めする!

参考文献
https://zenn.dev/genda_jp/articles/5ec82c7d8d51ae
