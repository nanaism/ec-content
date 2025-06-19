---
title: "コンテンツ販売アプリを作るガイド"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "stripe", "nextauth"]
published: true
price: 100
---

## はじめに

「自分の知識や経験を、デジタルコンテンツとして販売してみたい」
「noteやZennのような、洗練されたコンテンツ販売プラットフォームを自分の手で作り上げてみたい」

もしあなたがReactとNext.jsの基礎を学び終え、次なるステップとして「実践的で価値のあるポートフォリオを作りたい」と考えているなら、この記事はまさにそのための羅針盤となるでしょう。

このガイドでは、単なる技術のチュートリアルに留まりません。**デジタルコンテンツ（記事や電子書籍）を販売するECサイトをゼロから構築する**という、一つのプロダクト開発を丸ごと体験していただきます。

完成するアプリケーションは、以下のような機能を持ちます。

*   **ユーザー認証**: ソーシャルログイン（GitHub）による簡単で安全な会員登録・ログイン機能。
*   **コンテンツ管理**: エンジニアにお馴染みのMarkdown形式でコンテンツを執筆・管理。
*   **無料・有料設定**: 記事や本ごとに、無料公開または有料販売を設定可能。
*   **オンライン決済**: Stripeと連携し、クレジットカードによる安全な決済フローを実現。
*   **購入済みコンテンツの閲覧**: 一度購入したコンテンツは、いつでも自由に閲覧できる。


この旅路を終える頃には、あなたは以下の強力な技術スタックを自在に操り、複雑なWebアプリケーションを設計・実装する確かな自信を手にしているはずです。

## このガイドで深く学べること

本ガイドの核心となるテーマは、モダンなWeb開発における最重要トピックである**「認証」**と**「決済」**です。

### 1. Auth.js (v5) を用いた認証機能の実装
なぜ認証は自前で実装すべきではないのか？ `Auth.js` を使うことで、どれだけ安全かつ迅速に認証機能を組み込めるのかを学びます。ソーシャルログインの仕組みから、サーバーコンポーネント/クライアントコンポーネントでのセッション管理、そしてアクセスコントロールまで、実践的なノウハウを徹底解説します。

### 2. Stripeを使用した商品の購入（決済）フロー
オンライン決済の複雑さを、Stripeがいかにシンプルにしてくれるかを体験します。安全な決済ページ（Stripe Checkout）へのリダイレクト、購入処理の裏側で行われるWebhook通信の仕組み、そして購入履歴をデータベースで管理する方法まで、ECサイトの根幹をなす決済システムの全体像を掴みます。

### 使用する技術スタック（なぜこれを選ぶのか？）

私たちは、ただ流行りの技術を使うのではなく、それぞれの技術がもたらす価値を理解した上で、最適な選択を行います。

*   **Next.js 15 (App Router)**
    *   サーバーコンポーネントをフル活用し、パフォーマンスと開発者体験を極限まで高めます。Server Actionsを用いて、クライアントとサーバーの連携をシームレスに記述します。
*   **Auth.js v5**
    *   認証という複雑でセキュリティリスクの高い領域を、信頼できるオープンソースライブラリに任せることで、開発者はプロダクトのコアな価値創造に集中できます。
*   **Stripe**
    *   世界標準の決済プラットフォーム。開発者フレンドリーなAPIと鉄壁のセキュリティで、決済機能の実装を驚くほど容易にします。PCI DSS準拠の責任をStripeにオフロードできます。
*   **Prisma**
    *   TypeScriptとの相性が抜群のORM（Object-Relational Mapper）。直感的なスキーマ定義でデータベースを操作し、型安全な開発を実現します。購入履歴などの重要なデータを永続化します。
*   **GitHub API によるコンテンツ取得**
    *   CMSを導入せず、エンジニアが慣れ親しんだGitHubリポジトリでコンテンツを管理します。`git`によるバージョン管理の恩恵を受けながら、Markdownで快適に執筆できるフローを構築します。
*   **TypeScript**
    *   言わずと知れた、大規模アプリケーション開発の必須言語。型による静的解析が、バグを未然に防ぎ、コードの保守性を飛躍的に向上させます。
*   **Tailwind CSS & shadcn/ui**
    *   ユーティリティファーストのCSSフレームワークと、革新的なUIコンポーネント構築ツール。デザインの自由度と開発スピードを両立し、「自分のもの」として所有・管理できるUIを構築します。

さあ、準備はいいですか？
ここから、本格的なWebアプリケーション開発の世界へ深くダイブしていきましょう。

---

### 第1章: プロジェクトの設計と環境構築

最初に、これから作るアプリケーションの全体像を把握し、開発を始めるための土台を固めます。

#### 1-1. アーキテクチャとユーザーフロー

ユーザーがサイトを訪れてからコンテンツを購入するまで、データはどのように流れるのでしょうか？以下の図で全体像を掴みましょう。

1.  **ユーザー登録/ログイン**: ユーザーはGitHubアカウントでサイトにログインします（Auth.js）。認証情報はDBに保存されます（Prisma）。
2.  **コンテンツ閲覧**: コンテンツ一覧や詳細ページが表示されます。コンテンツデータはGitHubリポジトリから取得されます（GitHub API）。
3.  **購入処理**: ユーザーが購入ボタンをクリックすると、Server Actionが発火し、Stripeの決済セッションを作成します。
4.  **決済**: ユーザーはStripeの安全な決済ページにリダイレクトされ、支払いを完了します。
5.  **購入完了通知**: StripeはWebhookを介して、私たちのサーバーに決済完了を通知します。
6.  **購入履歴の保存**: Webhookを受け取ったサーバーは、「どのユーザーがどのコンテンツを購入したか」という情報をDBに記録します（Prisma）。
7.  **購入済みコンテンツの表示**: ユーザーが再度コンテンツページにアクセスすると、サーバーはDBの購入履歴を確認し、購入済みであればコンテンツの全文を表示します。

#### 1-2. 開発環境のセットアップ

`create-next-app`から始め、必要なライブラリをインストールし、各種設定ファイルを準備します。

```bash
# Next.jsプロジェクトの作成
npx create-next-app@latest nextjs-contents-market --typescript --tailwind --eslint

# 必要なライブラリのインストール
npm install next-auth @auth/prisma-adapter prisma stripe
npm install -D @types/node ts-node

# Prismaの初期化
npx prisma init

# shadcn/uiのセットアップ
npx shadcn-ui@latest init
```

このセクションでは、`tailwind.config.ts`の調整、`components.json`の確認、そして最も重要な`schema.prisma`の設計と定義を丁寧に行います。User, Account, Session, Purchase, Post, Bookといったモデルを定義し、リレーションを組んでいきます。

---

### 第2章: Auth.jsによる認証基盤の構築

アプリケーションの全ての機能の土台となる、ユーザー認証システムを構築します。

#### 2-1. GitHub OAuth Appの作成と環境変数設定

GitHubでOAuthアプリケーションを登録し、取得した`Client ID`と`Client Secret`を`.env.local`に設定します。同時に、`AUTH_SECRET`やデータベース接続情報も設定します。

#### 2-2. 認証APIルートの作成

`app/api/auth/[...nextauth]/route.ts`に、Auth.jsの中核となる設定を記述します。`PrismaAdapter`を使い、認証情報をデータベースと連携させます。

```typescript:app/api/auth/[...nextauth]/authOptions.ts
import { NextAuthOptions } from "next-auth";
import GitHubProvider from "next-auth/providers/github";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { prisma } from "@/lib/prisma";

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  providers: [
    GitHubProvider({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
  ],
  // ... callbacksやpagesの設定
};
```

#### 2-3. ヘッダーにログイン/ログアウト機能を実装

`useSession`, `signIn`, `signOut`フックを使い、ユーザーの認証状態に応じて表示が変わるヘッダーコンポーネントを作成します。`shadcn/ui`の`Avatar`や`DropdownMenu`を活用し、モダンなUIを構築します。

---

### 第3章: Stripeによる決済システムの統合

アプリケーションのマネタイズの心臓部である、決済機能を実装します。

#### 3-1. Stripeアカウントと商品の設定

Stripeダッシュボードでアカウントを有効化し、テスト用のAPIキーを取得します。また、販売するコンテンツに対応する「商品（Product）」と「価格（Price）」をStripe上に登録し、そのIDを控えておきます。

#### 3-2. 購入フローの実装 (Server Actions)

ユーザーが購入ボタンをクリックした際の処理を、Server Actionsで実装します。このActionは、ログイン状態を確認し、Stripeの決済セッションを作成後、ユーザーを決済ページへリダイレクトさせます。

```typescript:app/actions/stripe.ts
"use server";
// ... imports

export async function createCheckoutSession(userId: string, priceId: string) {
  const session = await stripe.checkout.sessions.create({
    // ... 設定
    metadata: {
      userId,
      priceId,
    },
  });
  redirect(session.url!);
}
```
`metadata`にユーザーIDと商品IDを詰めることが、後のWebhook処理で非常に重要になります。

#### 3-3. Webhookエンドポイントの構築

Stripeからの非同期な通知を受け取るためのAPIルートを作成します。署名を検証してリクエストが本物かを確認し、`checkout.session.completed`イベントに応じて、データベースに購入履歴を書き込みます。

```typescript:app/api/webhooks/stripe/route.ts
// ... imports

export async function POST(req: Request) {
  // ... 署名検証のロジック

  if (event.type === "checkout.session.completed") {
    const session = event.data.object;
    const { userId, priceId } = session.metadata!;
    
    // DBに購入履歴を保存
    await prisma.purchase.create({
      data: {
        userId,
        priceId, // 本来はコンテンツIDと紐付ける
      },
    });
  }
  return new Response("Webhook received", { status: 200 });
}
```

---

### 第4章: コンテンツ表示と購入制御の実装

認証と決済、二つのシステムを連携させ、コンテンツへのアクセスを制御します。これこそが、このアプリケーションのコアロジックです。

#### 4-1. GitHubからのコンテンツ取得ロジック

`lib`ディレクトリに、GitHub APIを叩いてリポジトリからMarkdownファイルを取得し、パースする関数を作成します。`gray-matter`を使ってfrontmatter（メタデータ）を読み込み、コンテンツが有料か、Stripeのどの`priceId`に対応するかを判断します。

#### 4-2. コンテンツ詳細ページの構築

動的ルート (`app/posts/[slug]/page.tsx`など) を使って、コンテンツ詳細ページを作成します。このページはサーバーコンポーネントとして実装します。

#### 4-3. アクセス制御ロジックの核心

ページのサーバーコンポーネント内で、以下の処理を順番に行います。

1.  URLの`slug`から、表示すべきコンテンツをGitHubから取得。
2.  `authOptions`を使って、現在のユーザーセッションを取得。
3.  コンテンツが有料の場合、DBにアクセスし、現在のユーザーがこのコンテンツを購入済みかを確認。
    ```typescript
    const isPurchased = await prisma.purchase.findFirst({
      where: { userId: session.user.id, contentId: post.id },
    });
    ```
4.  これらの情報（`isLoggedIn`, `isPurchased`, `isPaidContent`）を元に、条件分岐で表示するコンポーネントを切り替えます。
    *   購入済み or 無料コンテンツ → 全文表示コンポーネント
    *   未購入の有料コンテンツ → 一部表示＋購入ボタンコンポーネント
    *   未ログインの有料コンテンツ → ログイン促進＋購入ボタンコンポーネント

このロジックを実装することで、noteやZennのようなセキュアなコンテンツ販売が実現できます。
