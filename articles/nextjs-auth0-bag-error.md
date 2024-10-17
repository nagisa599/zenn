---
title: "Nextjs(appRouter)でnextjs-auth0ライブラリーを使ったログイン後の画面遷移でつこずった話"
emoji: "💢"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# 前提

- auth0 を認証基盤に使っており、ログイン後のページ遷移を行うことにした。
- フロントエンドは、Nextjs(appruter)を利用している。

# 詰まったところ

- 公式ドキュメントを見る限りでは、handleLogin の Optional として returnTo が記載されていた。しかし、正しく profile に飛ばされず/(ルートパス)に飛ばされてしまう
- [公式ドキュメント該当箇所 URL](https://auth0.github.io/nextjs-auth0/interfaces/handlers_login.LoginOptions.html)
  ![https://auth0.github.io/nextjs-auth0/interfaces/handlers_login.LoginOptions.html](/images/nextjs-auth0-bag-error/document-auth0.png)

```ts:src/app/api/auth/[auth0]/route.ts
import { handleAuth, handleLogin } from "@auth0/nextjs-auth0";

export const GET = handleAuth({
  login: handleLogin({
    authorizationParams: {
      audience: "https:xxxxxxxx",
      // scope: "openid profile email", // 例として scope を追加
    },
    returnTo: "/profile",
  }),
});
```

# 解決方法

- middlreware に returnTo の設定を書く。

```ts:middleware.ts
import { withMiddlewareAuthRequired } from '@auth0/nextjs-auth0/edge'

export default withMiddlewareAuthRequired({
  returnTo: '/profile',
})

export const config = {
  matcher: ['/:path*'], // すべてのパスとサブパスにマッチします
}
```

#　原因

1. /api/auth/login?returnTo=/profile の api を叩くことでログイン後のページを設定できる。そのため/api/auth/login の中に returnTo の実装を書くのはおかしかった(だったら handleLogin の option パラメータに returnTo 用意しとくなよ 💢)
2. pageRouter では、有効な可能性もある。
3. ドキュメントの approuter の仕様が明らかに少ない

# まとめ

最近は、approuter のドキュメントや記事が増えてきた。しかし、今回のように approuter で実装はできるが、ドキュメントが無いためかなり実装の道のり苦労する。(結構こういったライブラリーあるよね)
ライブラリーの選定の 1 つに app Router のドキュメントの豊富さも観点に入れてもいいかもしれない。
