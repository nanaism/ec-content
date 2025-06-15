---
title: "NextAuth.js とは？"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "nextauth", "authjs"]
published: true
---

## はじめに

この記事では、Next.jsでWebアプリケーションを開発する際に、多くの開発者が利用する「認証」機能について、それを驚くほど簡単に実装できるライブラリ **NextAuth.js（現: Auth.js）** をご紹介します。

「ログイン機能って、自分で作るのはセキュリティとか色々考えなきゃいけなくて大変そう…」と感じている方も多いのではないでしょうか。NextAuth.jsは、そんな悩みを解決してくれる、非常に強力で開発者に優しいツールです。

この記事を読めば、NextAuth.jsがどのようなもので、なぜ便利なのか、そしてNext.jsアプリケーションにどうやって導入するのか、基本的な流れを理解することができます。

## NextAuth.js (Auth.js) とは？

一言でいうと、NextAuth.jsは **「Next.jsアプリケーションの『ログイン機能』を、安全かつ迅速に構築するためのオープンソースライブラリ」** です。

従来、認証機能を自前で実装するには、以下のような多くの課題がありました。

*   パスワードの安全なハッシュ化と保存
*   セッション管理とCookieのセキュアな取り扱い
*   CSRF（クロスサイトリクエストフォージェリ）などの脆弱性対策
*   Google、GitHub、Twitterなど、外部サービスとの連携（OAuth認証）

NextAuth.jsは、これらの複雑で専門知識が求められる処理をすべて裏側で担当してくれます。開発者は、難しいセキュリティの知識に頭を悩ませることなく、数行の設定コードを書くだけで、堅牢な認証システムをアプリケーションに組み込むことができるのです。

### NextAuth.jsの主な特徴

#### 1. 豊富な認証プロバイダー
Google、GitHub、Twitter、Facebookといった主要なSNSアカウントを利用した「ソーシャルログイン」を簡単に実装できます。また、メールアドレスとパスワードによる古典的な認証（Credentials認証）や、メールアドレスに送られるリンクをクリックしてログインする「パスワードレス認証」にも対応しています。

#### 2. 高いカスタマイズ性
認証プロセス中に独自の処理を挟むことができる「コールバック関数」が用意されています。これにより、セッションに含めるユーザー情報を変更したり、特定のユーザーのログインを拒否したりと、アプリケーションの要件に合わせた柔軟なカスタマイズが可能です。

#### 3. データベースとの連携
`Prisma`や`Drizzle ORM`などのデータベースアダプタを通じて、ユーザーアカウントやセッション情報をデータベースに永続化できます。これにより、本格的なWebサービスに必要なユーザー管理基盤を構築できます。

#### 4. Next.jsとの高い親和性
サーバーコンポーネントやクライアントコンポーネント、APIルート（Route Handlers）など、Next.jsの機能を最大限に活かせるように設計されており、シームレスな開発体験を提供します。

### 名称変更について：NextAuth.jsからAuth.jsへ
v4までは「NextAuth.js」という名称で知られていましたが、v5からは **「Auth.js」** という名称に変更されました。これは、ライブラリがNext.jsだけでなく、SvelteKitやExpressなど、より多くのフレームワークで利用できる汎用的な認証ソリューションへと進化したことを意味します。
もちろん、Next.jsとの親和性の高さは変わらず、今でもNext.js開発における認証ライブラリの第一選択肢であり続けています。

## Next.js (App Router) での基本的な使い方

それでは、実際にNext.js v15（App Router環境）でNextAuth.jsを使用する際の、大まかな手順を見ていきましょう。ここでは、Googleアカウントでログインする機能を実装する場合を例に説明します。

### ステップ1: パッケージのインストール

まず、プロジェクトに`next-auth`パッケージをインストールします。

```bash
npm install next-auth
```

### ステップ2: APIルートの作成と認証プロバイダーの設定

次に、認証関連のリクエストを処理するためのAPIルートを作成します。App Routerでは、`app/api/auth/[...nextauth]/route.ts` というファイルを作成するのが一般的です。

このファイルに、使用したい認証プロバイダー（今回はGoogle）を設定します。

```typescript:app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth"
import GoogleProvider from "next-auth/providers/google"

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
})

export { handler as GET, handler as POST }
```

`clientId`と`clientSecret`は、Google Cloud Platformで取得した認証情報を、プロジェクトルートの`.env.local`ファイルに環境変数として設定します。また、セッションを暗号化するための`AUTH_SECRET`も必ず設定してください。

```text:.env.local
GOOGLE_CLIENT_ID="Your Google Client ID"
GOOGLE_CLIENT_SECRET="Your Google Client Secret"
AUTH_SECRET="A secret string for signing cookies, generated from `openssl rand -hex 32`"
```

### ステップ3: `SessionProvider`でアプリケーションをラップする

クライアントコンポーネントでセッション情報（ログイン状態など）を受け取るために、アプリケーションのルートを`SessionProvider`で囲みます。`SessionProvider`はClient Component内で使用する必要があるため、別ファイルに切り出すのがおすすめです。

```tsx:app/providers.tsx
"use client"

import { SessionProvider } from "next-auth/react"

export function Providers({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>
}
```

そして、ルートレイアウト（`app/layout.tsx`）でこの`Providers`コンポーネントを使用します。

```tsx:app/layout.tsx
import { Providers } from "./providers"

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="ja">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

### ステップ4: 認証状態へのアクセス

これで準備は完了です。コンポーネントから認証状態にアクセスしてみましょう。

#### クライアントコンポーネントでのアクセス (`useSession`フック)
`useSession`フックを使うと、ログインしているユーザー情報や認証状態（ローディング中、認証済み、未認証）を簡単に取得できます。

```tsx:app/components/LoginButton.tsx
"use client"

import { useSession, signIn, signOut } from "next-auth/react"

export default function LoginButton() {
  const { data: session, status } = useSession()

  if (status === "loading") {
    return <p>Loading...</p>
  }

  if (session) {
    return (
      <>
        こんにちは, {session.user?.name}さん! <br />
        <button onClick={() => signOut()}>サインアウト</button>
      </>
    )
  }

  return (
    <>
      あなたはログインしていません <br />
      <button onClick={() => signIn("google")}>Googleでサインイン</button>
    </>
  )
}
```

#### サーバーコンポーネントでのアクセス (`getServerSession`関数)
サーバーコンポーネントやAPIルートでは、`getServerSession`関数を使ってセッション情報を取得します。これにより、ページがレンダリングされる前にサーバー側で認証状態を確認できます。

```tsx:app/dashboard/page.tsx
import { getServerSession } from "next-auth/next"
// ステップ2で作成したroute.tsから、設定オブジェクトをインポートできるようにしておく必要があります
// (例: `export const authOptions = { ... }`)
import { authOptions } from "@/app/api/auth/[...nextauth]/route"
import { redirect } from "next/navigation"

export default async function DashboardPage() {
  const session = await getServerSession(authOptions)

  // セッションがない場合はトップページにリダイレクト
  if (!session) {
    redirect("/")
  }

  return (
    <div>
      <h1>ようこそ、ダッシュボードへ</h1>
      <p>あなたのメールアドレス: {session.user?.email}</p>
    </div>
  )
}
```

## おわりに

この記事では、NextAuth.js（Auth.js）の基本的な概念と、Next.jsアプリケーションへの導入手順を解説しました。NextAuth.jsが、複雑な認証処理を抽象化し、開発者が本来のアプリケーション開発に集中できるようにしてくれる、非常に強力なツールであることがお分かりいただけたかと思います。

今回ご紹介したのは、ほんの入り口に過ぎません。公式ドキュメントを参照すれば、

*   コールバックを使ったセッション内容のカスタマイズ
*   データベースアダプタを利用したユーザー情報の永続化
*   ユーザーの役割（Role）に基づいたアクセス制御

など、より高度で実践的な機能について学ぶことができます。

ぜひ、ご自身のNext.jsプロジェクトにNextAuth.jsを導入して、安全で快適な認証機能を実装してみてください！