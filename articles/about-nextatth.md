---
title: "NextAuth.js とは？"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "nextauth"]
published: true
---

## NextAuth.js とは？

NextAuth.js は、Next.js アプリケーションに認証機能を簡単に実装できるオープンソースのライブラリです。

v4 までは、NextAuth.js と呼ばれていましたが、
v5 からは、Auth.js という名称に変更されています。

OAuth、JWT、データベース連携など複数の認証方法をサポートし、セキュリティのベストプラクティスに従いながら、開発者が複雑な認証ロジックを記述することなく、Google、GitHub、Twitter などの多数のプロバイダーを使った認証システムを迅速に構築できるようにするツールです。

## Next.js v15 で使用するには？

Next.js v15 で NextAuth.js を使用する大まかな手順は、

1. 最新の `next-auth` パッケージをインストール
2. App Router の場合は `/app/api/auth/[...nextauth]/route.js` にルートハンドラーを作成して認証プロバイダーを設定
3. `SessionProvider` を使用してアプリケーション全体で認証状態を共有するようにセットアップ
4. `useSession` フックや `getServerSession` 関数を使って認証状態にアクセスする

## おわりに

Claude 3.7 Sonnet で生成した、勉強会用のダミーデータです。
