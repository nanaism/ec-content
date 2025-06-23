---
title: "Next.js 15 と Auth.js v5, Stripeで作る！モダンなデジタルコンテンツ販売サイト構築ガイド"
emoji: "🛒"
type: "tech"
topics: ["nextjs", "react", "authjs", "stripe", "prisma", "typescript"]
published: true
---

## はじめに

こんにちは！フロントエンドエンジニア2年目のogaです。
ReactとNext.jsの基本をなんとなく理解して、「次は何を作ろうかな…」「ただのTodoアプリじゃ物足りないな…」と悩んでいた時期が、皆さんにもありませんでしたか？ 

そんな時、ふと思ったんです。

「自分の知識や経験を、デジタルコンテンツとして販売してみたい」
「Zennやnoteみたいな、洗練されたプラットフォームを自分の手で作れたら、最高のポートフォリオになるんじゃないか？」

もしあなたが僕と同じように、学んだ知識を活かして**実践的で価値のあるポートフォリオ**を作りたいと考えているなら、この記事はまさにそのための羅針盤になるはずです。

この記事では、単なる技術のチュートリアルで終わりません。**デジタルコンテンツ（技術記事や電子書籍）を販売するECサイトをゼロから構築する**という、一つのプロダクト開発を丸ごと体験していただきます。エンジニアとして、認証や決済といった、サービス開発の根幹に関わる機能を実装する経験は、間違いなくあなたの市場価値を押し上げてくれるはずです。

### 完成するアプリケーションのイメージ

このガイドを最後まで進めると、以下のような機能を持つアプリケーションが完成します。

*   **ユーザー認証**: ソーシャルログイン（GitHub）による簡単で安全な会員登録・ログイン機能。
*   **コンテンツ管理**: エンジニアに馴染み深いMarkdown形式でコンテンツを執筆・管理。
*   **無料・有料設定**: 記事や本ごとに、無料公開または有料販売を設定可能。
*   **オンライン決済**: Stripeと連携し、クレジットカードによる安全な決済フローを実現。
*   **購入済みコンテンツの閲覧**: 一度購入したコンテンツは、いつでも自由に閲覧できる。

この旅路を終える頃には、あなたは以下の強力な技術スタックを自在に操り、複雑なWebアプリケーションを設計・実装する確かな自信を手にしているはずです。僕自身、この開発を通してサーバーコンポーネントやServer Actionsの本当の魅力を理解できたと感じています。

## このガイドで深く学べること

本ガイドの核心となるテーマは、モダンなWeb開発における最重要トピックである**「認証」**と**「決済」**です。この二つは、どんなWebサービスを作る上でも避けては通れない道ですよね。

### 1. Auth.js (v5) を用いた認証機能の実装
「認証は自前で実装するな」という言葉をよく聞きますが、それはなぜでしょうか？ Auth.jsを使うことで、どれだけ安全かつ迅速に認証機能を組み込めるのかを学びます。 v5から`@auth/core`をベースとするようになり、より標準に準拠した設計になりました。 ソーシャルログインの仕組みから、サーバーコンポーネント/クライアントコンポーネントでのセッション管理、そしてコンテンツへのアクセスコントロールまで、僕が実際にハマったポイントも交えながら実践的なノウハウを徹底解説します。

### 2. Stripeを使用した商品の購入（決済）フロー
オンライン決済の複雑さを、Stripeがいかにシンプルにしてくれるかを体験します。安全な決済ページ（Stripe Checkout）へのリダイレクト、購入処理の裏側で行われるWebhook通信の仕組み、そして購入履歴をデータベースで管理する方法まで、ECサイトの根幹をなす決済システムの全体像を掴みます。 特にWebhookの取り扱いは重要で、署名検証を必ず行うことで、Stripeからのリクエストであることを保証する必要があります。

### 使用する技術スタック（なぜこれを選ぶのか？）

私たちは、ただ流行りの技術を使うのではなく、それぞれの技術がもたらす価値を理解した上で、最適な選択を行います。2年目のエンジニアとして、技術選定の理由を自分の言葉で説明できることは、とても重要だと感じています。

*   **Next.js 15 (App Router)**
    *   React 19をサポートし、パフォーマンスと開発者体験がさらに向上しました。 特に、キャッシュの挙動が変更され、デフォルトではキャッシュされなくなったため、より直感的な制御が可能です。 `next dev --turbo` が安定版になり、開発サーバーの起動が劇的に速くなったのも嬉しいポイントです。 この記事では、サーバーコンポーネントとServer Actionsをフル活用し、クライアントとサーバーの連携をシームレスに記述していきます。
*   **Auth.js v5**
    *   認証という複雑でセキュリティリスクの高い領域を、信頼できるオープンソースライブラリに任せることで、私たちはプロダクトのコアな価値創造に集中できます。v5では設定ファイルが`auth.ts`に集約され、`handlers`や`auth`をエクスポートする形になり、コードの見通しが良くなりました。
*   **Stripe**
    *   世界標準の決済プラットフォーム。開発者フレンドリーなAPIと鉄壁のセキュリティで、決済機能の実装を驚くほど容易にします。PCI DSS準拠の責任をStripeにオフロードできるのは、個人開発者にとって計り知れないメリットです。
*   **Prisma**
    *   TypeScriptとの相性が抜群のORM（Object-Relational Mapper）。直感的なスキーマ定義でデータベースを操作し、型安全な開発を実現します。購入履歴などの重要なデータを永続化するために採用します。
*   **GitHub API によるコンテンツ取得**
    *   本格的なCMSを導入せず、エンジニアが慣れ親しんだGitHubリポジトリでコンテンツを管理します。`git`によるバージョン管理の恩恵を受けながら、Markdownで快適に執筆できるフローは、技術ブログとの相性も抜群です。
*   **TypeScript**
    *   言わずと知れた、現代のWeb開発における必須言語。型による静的解析が、実行時エラーを未然に防ぎ、コードの保守性を飛躍的に向上させます。もうJavaScriptだけの世界には戻れません…！
*   **Tailwind CSS & shadcn/ui**
    *   ユーティリティファーストのCSSフレームワークと、革新的なUIコンポーネント構築ツール。デザインの自由度と開発スピードを両立し、「自分のもの」として所有・管理できるUIを構築できるのが魅力です。

さあ、準備はいいですか？
ここから、本格的なWebアプリケーション開発の世界へ深くダイブしていきましょう。

---

### 第1章: プロジェクトの設計と環境構築

何事も最初が肝心です。まずは、これから作るアプリケーションの全体像を把握し、開発を始めるための土台を固めていきましょう。

#### 1-1. アーキテクチャとユーザーフロー

ユーザーがサイトを訪れてからコンテンツを購入するまで、データはどのように流れるのでしょうか？以下の図（Mermaid記法）で全体像を掴みましょう。

```mermaid
sequenceDiagram
    participant User as ユーザー
    participant App as Next.jsアプリ
    participant DB as Prisma (DB)
    participant GitHub as GitHub API
    participant Stripe

    User->>App: GitHubでログイン
    App->>DB: 認証情報を保存 (Auth.js)
    DB-->>App: ログイン成功
    App-->>User: ログイン状態を表示

    User->>App: コンテンツ一覧/詳細を閲覧
    App->>GitHub: Markdownコンテンツを取得
    GitHub-->>App: コンテンツデータを返す
    App-->>User: ページを表示

    User->>App: 購入ボタンをクリック (Server Action)
    App->>Stripe: 決済セッションを作成
    Stripe-->>App: 決済ページのURLを返す
    App-->>User: Stripe決済ページへリダイレクト

    User->>Stripe: クレジットカード情報入力・決済
    Stripe-->>User: 決済完了画面

    Stripe-)App: Webhookで決済完了を通知
    App->>App: 署名を検証しリクエストの正当性を確認
    App->>DB: 購入履歴を保存 (どのユーザーが、どの商品か)
    DB-->>App: 保存成功
    App-)Stripe: 200 OKレスポンスを返す

    User->>App: 購入済みコンテンツにアクセス
    App->>DB: ユーザーの購入履歴を確認
    DB-->>App: 購入済み情報を返す
    App->>GitHub: コンテンツを取得
    GitHub-->>App: コンテンツデータを返す
    App-->>User: コンテンツの全文を表示
```

#### 1-2. 開発環境のセットアップ

`create-next-app`から始め、必要なライブラリをインストールし、各種設定ファイルを準備します。

```bash
# Next.jsプロジェクトの作成 (最新版を使います)
npx create-next-app@latest nextjs-contents-market --typescript --tailwind --eslint

# プロジェクトディレクトリへ移動
cd nextjs-contents-market

# 必要なライブラリのインストール
# Auth.jsとPrismaアダプタ、Prisma本体、Stripe
npm install next-auth@beta @auth/prisma-adapter prisma stripe

# 開発環境用のライブラリ
npm install -D @types/node ts-node

# Prismaの初期化
npx prisma init

# shadcn/uiのセットアップ (これがあるとUI実装が爆速になります)
npx shadcn-ui@latest init
```

このセクションでは、`tailwind.config.ts`の調整、`components.json`の確認、そして最も重要な`prisma/schema.prisma`の設計を丁寧に行います。`User`, `Account`, `Session`はAuth.jsに必須のモデルです。 これに加えて、私たちのアプリケーションの核となる`Purchase`（購入履歴）、`Post`（記事）、`Book`（本）といったモデルを定義し、リレーションを組んでいきます。

---

### 第2章: Auth.jsによる認証基盤の構築

アプリケーションの全ての機能の土台となる、ユーザー認証システムを構築します。v5になって、設定がよりシンプルになりました。

#### 2-1. GitHub OAuth Appの作成と環境変数設定

まずはGitHubにアクセスし、開発者設定から新しいOAuth Appを作成します。ここで取得した`Client ID`と`Client Secret`は、プロジェクトのルートに作成する`.env.local`ファイルに保存します。Auth.js v5からは、環境変数を`AUTH_`プレフィックスで命名すると自動で読み込んでくれるので便利です。

```.env.local
# .env.local

# GitHub Provider
AUTH_GITHUB_ID="gho_..."
AUTH_GITHUB_SECRET="..."

# Auth.js Core
AUTH_SECRET="openssl rand -base64 32 で生成したランダムな文字列"
AUTH_URL="http://localhost:3000"

# Database
DATABASE_URL="postgresql://user:password@host:port/database?sslmode=require"
```

#### 2-2. 認証設定ファイルとAPIルートの作成

`auth.ts`というファイルをプロジェクトのルート（または`src/`）に作成し、Auth.jsの中核となる設定を記述します。`PrismaAdapter`を使い、認証情報をデータベースと連携させます。

```typescript:auth.ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";
import { PrismaAdapter } from "@auth/prisma-adapter";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  providers: [GitHub], // 環境変数からIDとSecretが自動で読み込まれる
  session: {
    strategy: "jwt", // JWTを使うのがモダンなスタイルです
  },
  callbacks: {
    // セッション情報にユーザーIDを追加する
    session({ session, token }) {
      if (token.sub && session.user) {
        session.user.id = token.sub;
      }
      return session;
    },
  },
});
```
次に、この設定を読み込むためのAPIルートを作成します。
```typescript:app/api/auth/[...nextauth]/route.ts
import { handlers } from "@/auth"; // auth.tsからインポート
export const { GET, POST } = handlers;
```
v5ではこれだけでAPIルートの完成です。とてもシンプルになりましたね。

#### 2-3. ヘッダーにログイン/ログアウト機能を実装

サーバーコンポーネントとクライアントコンポーネントでセッション情報の取得方法が異なります。

*   **サーバーコンポーネント**: `auth()` ヘルパー関数を使います。
*   **クライアントコンポーネント**: 従来通り `useSession` フックを使いますが、Layoutで `<SessionProvider>` でラップする必要があります。

`shadcn/ui`の`Avatar`や`DropdownMenu`を活用し、ユーザーの認証状態に応じて表示が変わるモダンなUIのヘッダーをサクッと構築しましょう。

---

### 第3章: Stripeによる決済システムの統合

アプリケーションのマネタイズの心臓部、決済機能を実装します。初めて触ると難しそうに感じますが、Stripeのドキュメントとライブラリは本当に優秀です。

#### 3-1. Stripeアカウントと商品の設定

Stripeダッシュボードでアカウントを有効化し、テスト用のAPIキーを取得して`.env.local`に設定します。また、販売するコンテンツに対応する「商品（Product）」と「価格（Price）」をStripe上で登録し、その`price_...`から始まる価格IDを控えておきます。これが商品とStripeを結びつける鍵になります。

#### 3-2. 購入フローの実装 (Server Actions)

ユーザーが購入ボタンをクリックした際の処理を、Server Actionsで実装します。このActionは、ログイン状態を確認し、Stripeの決済セッションを作成後、ユーザーを決済ページへリダイレクトさせます。APIルートを別途作成する必要がないのがServer Actionsの強力な点です。

```typescript:app/actions/stripe.ts
"use server";

import { auth } from "@/auth";
import { stripe } from "@/lib/stripe";
import { redirect } from "next/navigation";

// 引数として、どの商品(価格ID)を購入するかを受け取る
export async function createCheckoutSession(priceId: string) {
  const session = await auth();
  if (!session?.user?.id) {
    throw new Error("User not authenticated");
  }

  const checkoutSession = await stripe.checkout.sessions.create({
    mode: "payment",
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.AUTH_URL}/payment/success`,
    cancel_url: `${process.env.AUTH_URL}/payment/cancel`,
    // Webhookでユーザー情報と商品情報を特定するためにmetadataに含める
    metadata: {
      userId: session.user.id,
      priceId: priceId,
    },
  });

  if (!checkoutSession.url) {
    throw new Error("Could not create checkout session");
  }

  redirect(checkoutSession.url);
}
````metadata`に`userId`と`priceId`を詰めることが、後のWebhook処理で非常に重要になります。

#### 3-3. Webhookエンドポイントの構築

Stripeからの非同期な通知を受け取るためのAPIルートを作成します。Stripeからのリクエストが本物かを確認するために、必ず署名を検証します。 この検証を怠ると、悪意のある第三者が購入完了を偽装できてしまうため、絶対に必要です。

`checkout.session.completed`イベントを受け取ったら、`metadata`からユーザーIDと商品IDを取り出し、データベースに購入履歴を書き込みます。

```typescript:app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from "next/server";
import { stripe } from "@/lib/stripe";
import { prisma } from "@/lib/prisma";
import Stripe from "stripe";

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(req: NextRequest) {
  const buf = await req.text();
  const sig = req.headers.get("stripe-signature")!;

  let event: Stripe.Event;

  try {
    event = stripe.webhooks.constructEvent(buf, sig, webhookSecret);
  } catch (err) {
    console.error("Webhook signature verification failed.", err);
    return NextResponse.json({ error: "Webhook error" }, { status: 400 });
  }

  if (event.type === "checkout.session.completed") {
    const session = event.data.object as Stripe.Checkout.Session;
    const { userId, priceId } = session.metadata!;

    // DBに購入履歴を保存
    // TODO: priceIdから実際のコンテンツIDへのマッピングが必要
    await prisma.purchase.create({
      data: {
        userId: userId,
        stripePriceId: priceId,
        // contentId: "..."
      },
    });
  }

  return NextResponse.json({ received: true });
}
```
Stripeのベストプラクティスとして、Webhookハンドラは迅速に`200`レスポンスを返すことが推奨されています。 重い処理は非同期で行うのが理想です。

---

### 第4章: コンテンツ表示と購入制御の実装

認証と決済、二つのシステムを連携させ、コンテンツへのアクセスを制御します。これこそが、このアプリケーションのコアロジックであり、腕の見せ所です！

#### 4-1. GitHubからのコンテンツ取得ロジック

`lib`ディレクトリなどに、GitHub APIを叩いてリポジトリからMarkdownファイルを取得し、パースする関数を作成します。`gray-matter`というライブラリを使うと、Markdownファイルの先頭に書かれたfrontmatter（YAML形式のメタデータ）と本文を簡単に分離できます。

```markdown:example-post.md
---
title: "初めてのServer Actions"
priceId: "price_xxxxxxxxxxxxxx" # この記事が有料の場合、Stripeの価格IDを指定
---

## ここから本文が始まります

Server Actionsは...
```この`priceId`の有無で、コンテンツが無料か有料かを判断します。

#### 4-2. コンテンツ詳細ページの構築

Next.jsの動的ルート (`app/posts/[slug]/page.tsx`など) を使って、コンテンツ詳細ページを作成します。このページはサーバーコンポーネントとして実装することで、サーバーサイドで全てのロジックを完結させることができます。

#### 4-3. アクセス制御ロジックの核心

ページのサーバーコンポーネント内で、以下の処理を順番に行います。このロジックを考えるのが、一番楽しいところかもしれません。

1.  URLの`slug`から、表示すべきコンテンツをGitHubから取得します。
2.  `auth()`ヘルパー関数を使って、現在のユーザーセッションを取得します。（未ログインなら`null`が返ります）
3.  取得したコンテンツのfrontmatterに`priceId`があるか（＝有料コンテンツか）を確認します。
4.  コンテンツが有料の場合、DBにアクセスし、現在のユーザーIDとコンテンツID（または`priceId`）で`Purchase`テーブルを検索し、購入済みかを確認します。

```typescript:app/posts/[slug]/page.tsx
import { auth } from "@/auth";
import { getPostBySlug } from "@/lib/github";
import { prisma } from "@/lib/prisma";
import { MarkdownContent } from "@/components/markdown-content";
import { PurchaseButton } from "@/components/purchase-button";

export default async function PostPage({ params }: { params: { slug: string } }) {
  const { slug } = params;
  const session = await auth(); // 1. 認証情報を取得

  const post = await getPostBySlug(slug); // 2. コンテンツを取得
  const isPaidContent = !!post.frontmatter.priceId; // 3. 有料か判定

  let isPurchased = false;
  if (isPaidContent && session?.user?.id) {
    const purchase = await prisma.purchase.findFirst({ // 4. 購入履歴を確認
      where: {
        userId: session.user.id,
        stripePriceId: post.frontmatter.priceId,
      },
    });
    isPurchased = !!purchase;
  }

  const canViewContent = !isPaidContent || isPurchased;

  return (
    <article>
      <h1>{post.frontmatter.title}</h1>
      {canViewContent ? (
        <MarkdownContent content={post.content} />
      ) : (
        <div>
          <p>この記事は有料です。続きを読むには購入が必要です。</p>
          {/* 購入ボタンはServer Actionを呼び出すクライアントコンポーネント */}
          <PurchaseButton priceId={post.frontmatter.priceId} />
        </div>
      )}
    </article>
  );
}
```
この条件分岐ロジックを実装することで、noteやZennのようなセキュアなコンテンツ販売が実現できるわけです。
