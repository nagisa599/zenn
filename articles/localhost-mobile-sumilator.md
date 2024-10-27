---
title: "まだブラウザの検証ツールに頼ってる？モバイル表示検証テクニック"
emoji: "📱"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "mobile", "tool", "開発ツール", "便利"]
published: true
---

# 目的

- モバイルの画面表示をブラウザの検証ツールでサイズを毎回調整するのは、とても面倒.
- PC 画面とモバイル画面を並べて開発をすることが難しい。
- PC 画面とスマホでは操作感が違う

# 提案

#### 1. 自分のスマホ(実機)で表示　(おすすめ度：⭐️⭐️⭐️⭐️⭐️)

![実機](/images/localhost-mobile-sumilator/IMG_4799.jpg)

#### 2. simulator で表示(おすすめ度：⭐️⭐️⭐️)

![simulator](/images/localhost-mobile-sumilator/sumilator.png)

#### 3. ミラーリングで表示　(おすすめ度:⭐️)

![ミラーリング](/images/localhost-mobile-sumilator/mirror.png)

## 1. 自分のスマホで表示

#### step1

自分の IP アドレス(vvv.xxx.yyy.zzz)を調べる。

```bash
ipconfig getifaddr en0
```

#### step2

app のサーバを起動

```bash
  ▲ Next.js 15.0.1
  - Local:        http://localhost:3000
```

####　 step3
スマホからでは localhost は使えないので vvv.xxx.yyy.zzz に変換する
"http://vvv.xxx.yyy.zzz:3000"

## simulator で表示

#### step 1

app のサーバを起動

```bash
  ▲ Next.js 15.0.1
  - Local:        http://localhost:3000
```

#### step2

localhost:3000 の URL でアクセス

## ミラーリングで表示

自分のスマホで表示(実機)の場合と一緒

# まとめ

実機で実装するのが操作感が掴めて、やりやすく PC の作業スペースが維持できるのでプログラミングしやすい。
Ipad やタブレッドなどの特殊のサイズの場合は、実機で持っていない場合もあるので、simulator で確認するのもあり
