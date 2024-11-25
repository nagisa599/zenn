---
title: "Golangで研究室のwebサイトのRAGを作ってみた~ 夏の１日自由研究 ~"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["RAG", "Go", "openai", "AI", "chatBot"]
publication_name: nislab
published: true
---

## 私たちの研究室

https://nisk.doshisha.ac.jp/

## 目的

夏休みに時間があったため、今流行りの RAG を使って chatBot を作ってみることにした。

## RAG とは

#### Retrieval-Augment GEneration === 検索拡張性

- 情報検索と生成を組み合わせた自然言語処理（NLP）の手法である。。この手法では、まず関連情報を検索し、その検索結果を用いてテキストを生成します。これにより、より精度の高い、情報に基づいた内容の生成が可能になる。

#### メリット

- ファインチューニングとは違い、再学習の手間がない
- model の変化、進化を考えなくていい。(ファインチューニングだと再学習する手間がかかる)

:::message
ファインチューニングとは
機械学習および自然言語処理（NLP）における一般的な手法で、事前に大量のデータで学習されたモデル（事前学習済みモデル）を特定のタスクやデータセットに特化してさらに学習させる
:::

#### 利用場面

- 社内に溜まっているクローズドなデータに対して質問を返してくれる chatbot を作りたい

## システム概要

### 概要

![アーキテクチャ](/images/rag-study/rag-studey-artecutre.png)

### 大まかな流れ

1. 特定のデータベースに情報を検索
2. その検索結果を元に大規模言語モデル(LLM)にインプット
3. その検索情報をもとに質問に基づいて答えを返す。

### 知識データベースとは

機器固有の ｢ 知識 ｣ を体系化したデータの集合

- web サイトをスクレイミングしたデータ

### 検索エンジンとは

質問をもとに知識データベースから近い情報を取得する

- 今回は。gpt-3.5-turbo-instruct を用いて。質問とデータの cos 類似を求めて近いデータを取得する

:::message

### gpt-3.5-turbo-instruct

- 特にインストラクションに基づいて最適化された応答を生成するように設計されている。このモデルは、特定の指示に基づいた応答生成に特化しており、ユーザーからの明確な要求に対して効率的かつ正確に答えることができる
  :::
  :::message

### text-embedding-ada-002

- テキストデータを固定長のベクトル表現に変換するために設計されている。このエンベディングモデルは、さまざまな自然言語処理タスクでの使用を目的としており、テキストの意味的な内容を数値的な形式で捉えることができる。
  :::

## 技術選定

| 技術                 | 利用しているライブラリやツール等               |
| -------------------- | ---------------------------------------------- |
| 言語                 | go1.22.45                                      |
| ローカル環境構築     | Docker                                         |
| スクレイピング       | goquery                                        |
| Open AI 外部 package | sashabaranov                                   |
| Open AI モデル       | text-embedding-ada-002, gpt-3.5-turbo-instruct |

## 実際にコード

スクレイピングコード

```go:utilss/scriping.go
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

チャンク分割コード

```go:utils/chunk.go
package utils

func ChunkText(text string, chunkSize, overlap int) []string {
	var chunks []string
	runes := []rune(text) // マルチバイト文字を正しく扱うためにruneスライスに変換
	length := len(runes)

	for i := 0; i < length; i += chunkSize - overlap {
		end := i + chunkSize
		if end > length {
			end = length
		}
		chunks = append(chunks, string(runes[i:end]))
	}

	return chunks
}
```

検索エンジン

```go:utils/embedding.go
package utils


func ChunkText(text string, chunkSize, overlap int) []string {
	var chunks []string
	runes := []rune(text) // マルチバイト文字を正しく扱うためにruneスライスに変換
	length := len(runes)

	for i := 0; i < length; i += chunkSize - overlap {
		end := i + chunkSize
		if end > length {
			end = length
		}
		chunks = append(chunks, string(runes[i:end]))
	}

	return chunks
}
```

```go:cmd/main.go

package main

import (
	"context"
	"fmt"
	"log"
	"os"
	"sort"
	"strings"

	"github.com/nagisa599/nislab_chatBot/constants"
	"github.com/nagisa599/nislab_chatBot/utils"
	"github.com/sashabaranov/go-openai"
)

// チャンクに分割する関数
type ChunkSim struct {
	Index      int
	Similarity float64
}
func main() {
	client := openai.NewClient(os.Getenv("OPENAIAPIKEY"))
	question := "現在のB4のメンバー教えて？"
	fmt.Print("質問: ", question, "\n")
	chunkSize := 400
	overlap := 50
	consolidatedText, err := utils.FetchAndProcessMultipleURLs(constants.Urls)
	if err != nil {
		log.Fatal(err)
	}

	// テキストをチャンクに分割
	chunks := utils.ChunkText(consolidatedText, chunkSize, overlap)

	chunksVector ,err := utils.GetEmbedding(client, chunks)
	if err != nil {
		fmt.Println("error")
	}
	questionVector, err := utils.GetEmbedding(client, []string{question})
	if err != nil {
		fmt.Println("error")
	}
	if len(questionVector) == 0 || len(chunksVector) == 0 {
		fmt.Println("Error: chunks vector or question vector is empty.")
		return
	}
	var similarities []ChunkSim
	for i, vec := range chunksVector {
		similarity, err := utils.CosSimilarity(vec, questionVector[0])
		if err != nil {
			fmt.Printf("Error calculating similarity for chunk %d: %v\n", i, err)
			continue
		}
		similarities = append(similarities, ChunkSim{i, similarity})
	}

	sort.Slice(similarities, func(i, j int) bool {
		return similarities[i].Similarity > similarities[j].Similarity
	})
	prompt := fmt.Sprintf(`以下の質問に以下の情報をベースにして回答してください。
	[ユーザの情報]
	%s

	[情報]
	%s
	%s
	`, question, chunks[similarities[0].Index], chunks[similarities[1].Index])


	gptChatResponse, err := client.CreateCompletion(context.Background(), openai.CompletionRequest{
		Model:     "gpt-3.5-turbo-instruct", // GPT-3.5-turbo-instructモデルを指定
		Prompt:    prompt,
		MaxTokens: 300, // 応答の最大トークン数
	})
	if err != nil {
		fmt.Println("Error:", err)
		return
	}
	// レスポンスのテキストを取得し、改行をスペースに置換して余分な空白を削除
	responseText := gptChatResponse.Choices[0].Text
	responseText = strings.ReplaceAll(responseText, "\n", " ") // 改行をスペースに置き換え
	responseText = strings.Join(strings.Fields(responseText), " ") // 余分なスペースを削除

	fmt.Println("GPTの回答:", responseText)

}
```

## 結果

![result](/images/rag-study/result.png)

## まとめ

思ったよりも簡単に RAG を使った chatbot を作ることができた。会社内や学校内で知識データベースが溜まっている場合は、RAG を使って様々な問題解決をしてほしい。
