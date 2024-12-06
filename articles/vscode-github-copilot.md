---
title: "予測変換しか使っていないcopilotから脱却したい大学生"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github", "github copilot", "copilot", "vscode", "AI"]
publication_name: nislab
published: true
---

# 私たちの研究室(Nislab)

https://nisk.doshisha.ac.jp/

# アドベントカレンダー 6 日目~

https://nislab-advent-calendar-2024-12.vercel.app

# 背景

chatGPT や claude AI、github copiloit など AI を駆使した開発が一般的になってきた。自分は`エラーコードをのっけて直すだけ(chatgpt,cloude AI)`や`予測変換(copilot)`を利用して、コードを生成するだけになっている。明らかに AI を使いこなせていないため、今回は copilot を使って、使えそうな機能をまとめてみた。

# github copilot とは

GitHub Copilot は、ユーザーのコーディング作業を強力にサポートしてくれる AI ペアプログラマーで、コードの自動補完、関数やクラスの生成、自然言語からのコード生成など、様々な方法で開発を効率化してくれる。

## 学生は無料で利用可能!!

https://docs.github.com/ja/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/managing-your-copilot-subscription/getting-free-access-to-copilot-as-a-student-teacher-or-maintainer

# 主要機能 3 選

**1. 予測変換**

**2. chat 機能**

**3. チャット画面からのアクション**

## 1 予測変換

1. コードを書いている途中で予測変換が出る
2. `tab` を押すとコードが生成、`esc`を押すと予測変換が破棄される
3. カーソルを合わせ、矢印キーで他の候補があれば表示することが可能になる
   ![](/images/resarch-lab-reason/image2.png)

## 2 chat 機能

### 2.1 chat

1. `command + shift + P `でコマンドパレットを開き、chat と書いて github copilot chat を開く (`command + shift + I` )
2. ファイルを指定して chat をすることができる
   ![](/images/resarch-lab-reason/image5.png)

### 2.2 インライン chat

1. 行を選択して、`command + I `を押す
   ![](/images/resarch-lab-reason/image8.png)

### 2.3. edit モード

1. edit モードにすると生成したものが自動的にファイルを変更してくれる
   ![](/images/resarch-lab-reason/image6.png)

## 3. チャット画面からのアクション

- @workspace
  - @workspace/fix コードの修正
  - @workspace/test テストの作成
  - @workspace /setupTests テストの 1 から実装
  - @workspacefixTestFailure - 失敗したテストの修正を提案
  - @workspace/doc ドキュメントの作成
  - @workspace/new フレームワークを作成
  - @workspace/explain コードの説明
- @vscode - VS Code に関する質問
  - @vscode/search - ワークスペース検索のクエリ パラメーターを生成
  - @vscode/runCommand - Search for and execute a command in VS Code
  - @vscode/startDebugging - VS Code で起動構成を生成してデバッグを開始 (試験段階)
- @terminal - ターミナルで何かを行う方法を確認
  - /explain - ターミナルで何か説明(ターミナルのエラー出る時に使えそう)

![](/images/resarch-lab-reason/image11.png)

## まとめ

個人的に以下の 3 つは、実際に使いこなせたら工数の削減につながるのではないかなと思った

- **@workspace/test によるテストの作成**
- **@workspace/new による簡単な api のフレームワーク**
- **@terminal/explain のエラーの説明**
