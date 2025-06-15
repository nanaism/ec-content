---
title: "Stripe とは？"
emoji: "💳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "stripe", "payment"]
published: true
---

## はじめに

Webサイトやアプリケーションで商品を販売したり、有料サービスを提供したりする上で、**オンライン決済機能**は欠かせない要素です。しかし、「決済システムの開発」と聞くと、こんな不安を感じませんか？

*   クレジットカード情報の取り扱いが怖い…セキュリティはどうすれば？
*   カード会社との契約手続きが複雑で大変そう…
*   サブスクリプション（月額課金）みたいな複雑な処理を自分で作れるだろうか？

**Stripe** は、こうした決済に関するあらゆる課題を解決してくれる、非常に強力なオンライン決済プラットフォームです。

この記事では、Stripeがどのようなサービスで、なぜ世界中の開発者や企業に選ばれているのか、そしてNext.jsアプリケーションにどうやって組み込むのか、その基本的な流れを分かりやすく解説します。

## Stripe とは？

一言でいうと、Stripeは **「Webサイトやアプリに、オンライン決済機能を驚くほど簡単かつ安全に導入するためのサービス」** です。

開発者は、Stripeが提供するAPIやライブラリを利用することで、通常は非常に複雑な決済処理を、まるで外部のサービスを呼び出すかのように簡単に実装できます。

### Stripeが解決してくれること

#### 1. 多様な決済手段への対応
クレジットカードやデビットカードはもちろん、Apple Pay、Google Pay、さらには日本のユーザー向けにコンビニ決済や銀行振込といった、多岐にわたる決済方法に簡単対応できます。ユーザーに合わせた選択肢を用意することで、購入率の向上にも繋がります。

#### 2. サブスクリプション（定期課金）管理
SaaS（Software as a Service）などで一般的な、月額や年額での継続的な支払いを管理する「Stripe Billing」という機能があります。プランの作成、ユーザーの登録、支払いサイクルの管理、請求書の発行などを自動化でき、サブスクリプションビジネスの基盤を簡単に構築できます。

#### 3. 鉄壁のセキュリティ
オンライン決済で最も重要なのがセキュリティです。Stripeは、クレジットカード業界の国際的なセキュリティ基準である **PCI DSSに完全準拠** しています。
開発者は、ユーザーのクレジットカード番号といった機密情報を自身のサーバーで直接扱う必要がありません。これにより、情報漏洩のリスクを大幅に低減し、本来のアプリケーション開発に集中することができます。

#### 4. 開発者フレンドリーな設計
Stripeが絶大な支持を得ている理由の一つが、その **卓越した開発者体験（Developer Experience）** です。
*   非常に分かりやすく整理された公式ドキュメント
*   各プログラミング言語に対応した豊富なSDK（ライブラリ）
*   実際の決済を試せるテスト環境と、詳細なログが確認できるダッシュボード

これらが揃っているため、開発者はストレスなく、迅速に決済機能を実装できます。

## Next.js (App Router) での基本的な使い方

それでは、Next.js v15（App Router環境）でStripeを使った決済機能を実装する際の、基本的な流れを見ていきましょう。ここでは、ユーザーが商品を購入し、Stripeが提供する決済ページで支払いを行うシンプルなシナリオを例にします。

### ステップ1: パッケージのインストール

まず、プロジェクトにStripeの公式Node.jsライブラリをインストールします。

```bash
npm install stripe
```

### ステップ2: APIキーの設定

Stripeの機能を利用するにはAPIキーが必要です。Stripeのダッシュボードで**公開可能キー（Publishable Key）**と**シークレットキー（Secret Key）**を取得し、プロジェクトルートの`.env.local`ファイルに環境変数として設定します。

**シークレットキーは絶対に外部に漏らしてはいけない機密情報**なので、必ず環境変数で管理しましょう。

```text:.env.local
# クライアントサイドで使用するキー
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."
# サーバーサイドでのみ使用するキー
STRIPE_SECRET_KEY="sk_test_..."
```

### ステップ3: サーバーサイドで決済セッションを作成する

ユーザーが「購入ボタン」をクリックした際の処理を、Server ActionsやAPI Routeを使ってサーバーサイドで実装します。
ここでは、購入する商品情報（名前、価格など）を元に、Stripeに対して「これからこういう決済をします」という予約情報のようなものである **決済セッション（Checkout Session）** を作成します。

```typescript:app/actions.ts
"use server"

import Stripe from "stripe"
import { redirect } from "next/navigation"

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)

export async function createCheckoutSession() {
  const session = await stripe.checkout.sessions.create({
    payment_method_types: ["card"], // 決済方法
    line_items: [ // 購入する商品
      {
        price_data: {
          currency: "jpy",
          product_data: {
            name: "オリジナルTシャツ",
          },
          unit_amount: 2000, // 金額 (円単位)
        },
        quantity: 1,
      },
    ],
    mode: "payment", // 1回限りの支払い
    success_url: `${process.env.NEXT_PUBLIC_BASE_URL}/success`, // 決済成功時のリダイレクト先
    cancel_url: `${process.env.NEXT_PUBLIC_BASE_URL}/cancel`, // 決済キャンセル時のリダイレクト先
  })

  // 作成されたセッションのURLにリダイレクト
  if (session.url) {
    redirect(session.url)
  }
}
```

### ステップ4: クライアントサイドで決済処理を呼び出す

クライアントサイドのコンポーネントに「購入ボタン」を設置し、クリックされたら先ほど作成したServer Actionを呼び出します。これにより、ユーザーはStripeがホストする安全な決済ページにリダイレクトされます。

```tsx:app/page.tsx
import { createCheckoutSession } from "./actions"

export default function Home() {
  return (
    <form action={createCheckoutSession}>
      <button type="submit">
        2,000円で購入する
      </button>
    </form>
  )
}
```
この **Stripe Checkout** と呼ばれるページ上で、ユーザーはカード情報を入力し、決済を完了させます。自前で決済フォームを作る必要がないため、非常に簡単かつ安全です。

### ステップ5: Webhookで決済結果をハンドリングする

ユーザーが決済を完了またはキャンセルした後、Stripeはその結果を私たちのアプリケーションに通知してくれます。この通知を受け取る仕組みが **Webhook** です。

API Route（例: `/api/webhook`）を作成し、Stripeからのリクエストを待ち受けます。ここで決済完了の通知を受け取ったら、データベースの注文情報を更新したり、ユーザーに購入完了メールを送信したりといった、決済後の重要な処理を行います。

## おわりに

この記事では、オンライン決済プラットフォームStripeの概要と、Next.jsアプリケーションへの基本的な導入方法について解説しました。

StripeとNext.jsを組み合わせることで、開発者はセキュリティの複雑さに頭を悩ませることなく、Server Actionsなどの最新機能を活用して、堅牢でモダンな決済システムを迅速に構築できます。

今回紹介したStripe Checkoutはほんの一例です。自社サイトのデザインに合わせた決済フォームを埋め込める **Stripe Elements** や、より複雑なサブスクリプションを管理する **Stripe Billing** など、ビジネスの成長に合わせて活用できる機能が豊富に用意されています。

オンライン決済機能の実装を検討している方は、ぜひStripeの導入を検討してみてください。そのパワフルさと開発のしやすさに、きっと驚くはずです。