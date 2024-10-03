---
title: "研究室のwebサイトでRAGを作ってみた~ 夏の１日自由研究 ~"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## 目的

夏休みに時間があったため、今流行りの RAG を使って chatBot を作ってみることにした。s

## RAG とは

## システム概要

### gpt-3.5-turbo-instruct

- 特にインストラクションに基づいて最適化された応答を生成するように設計されている。このモデルは、特定の指示に基づいた応答生成に特化しており、ユーザーからの明確な要求に対して効率的かつ正確に答えることができる

### text-embedding-ada-002

- テキストデータを固定長のベクトル表現に変換するために設計されている。このエンベディングモデルは、さまざまな自然言語処理タスクでの使用を目的としており、テキストの意味的な内容を数値的な形式で捉えることができる。 ##　　技術選定

| 技術                 | 利用しているライブラリやツール等               |
| -------------------- | ---------------------------------------------- |
| 言語                 | go1.22.45                                      |
| ローカル環境構築     | Docker                                         |
| スクレイピング       | goquery                                        |
| Open AI 外部 package | sashabaranov                                   |
| Open AI モデル       | text-embedding-ada-002, gpt-3.5-turbo-instruct |

## 実際にコード

スクレイピングコード

```go
package utils

import (
	"fmt"
	"net/http"
	"strings"
	"sync"

	"github.com/PuerkitoBio/goquery"
)

// 指定されたURLからHTMLを取得し、divタグ内のテキストを整形して返す関数
func FetchAndProcessURL(url string) (string, error) {
	res, err := http.Get(url)
	if err != nil {
		return "", fmt.Errorf("HTTP request failed: %w", err)
	}
	defer res.Body.Close()

	if res.StatusCode != 200 {
		return "", fmt.Errorf("status code error: %d %s", res.StatusCode, res.Status)
	}

	doc, err := goquery.NewDocumentFromReader(res.Body)
	if err != nil {
		return "", fmt.Errorf("error parsing HTML: %w", err)
	}

	var divTexts []string
	doc.Find("div").Each(func(i int, s *goquery.Selection) {
		text := strings.Join(strings.Fields(s.Text()), " ")
		if text != "" {
			divTexts = append(divTexts, text)
		}
	})

	consolidatedText := strings.Join(divTexts, " ")
	return consolidatedText, nil
}

// 複数のURLからテキストをフェッチして連結する関数
func FetchAndProcessMultipleURLs(urls []string) (string, error) {
	var wg sync.WaitGroup
	results := make([]string, len(urls))
	errors := make([]error, len(urls))

	for i, url := range urls {
		wg.Add(1)
		go func(i int, url string) {
			defer wg.Done()
			result, err := FetchAndProcessURL(url)
			if err != nil {
				errors[i] = err
				return
			}
			results[i] = result
		}(i, url)
	}

	wg.Wait()

	// エラーをチェックし、最初のエラーを報告
	for _, err := range errors {
		if err != nil {
			return "", err
		}
	}

	// 全ての結果を連結
	finalText := strings.Join(results, " ")
	return finalText, nil
}


```

## まとめ
