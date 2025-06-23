---
title: "Next.js + Prisma入門：型安全なWebアプリ開発の始め方"
emoji: "❄️"
type: "tech"
topics: ["nextjs", "prisma", "typescript", "orm"]
published: true
---

## はじめに：ORMとは何か？

現代のWebアプリケーション開発において、データベースとの連携は不可欠です。その際、多くの開発者が「ORM」という技術を利用します。

ORMは **Object-Relational Mapping（オブジェクト関係マッピング）** の略で、一言で言うと「**プログラミング言語のオブジェクトと、リレーショナルデータベースのテーブルを対応付ける（マッピングする）技術**」のことです。

通常、データベースを操作するにはSQL（Structured Query Language）を記述する必要があります。

```sql
SELECT id, name, email FROM "User" WHERE id = 1;
```

ORMを使うと、上記のようなSQL文を直接書く代わりに、使い慣れたプログラミング言語（この場合はTypeScript/JavaScript）のコードでデータベースを操作できます。

```typescript
const user = await prisma.user.findUnique({
  where: { id: 1 },
});
```

### ORMを利用するメリット

*   **生産性の向上**: SQLを直接記述する必要がなくなり、より直感的かつ迅速にデータベース操作のコードを記述できます。
*   **安全性の向上**: ORMがSQLを自動生成するため、SQLインジェクションのような脆弱性のリスクを低減できます。特に、後述するPrismaは「型安全性」に優れており、コンパイル時にエラーを発見しやすくなります。
*   **コードの可読性と保守性の向上**: SQLがコードから分離され、アプリケーションのロジックがより明確になります。
*   **データベースの抽象化**: 使用するデータベース（PostgreSQL, MySQLなど）が変更になっても、アプリケーション側のコードを大幅に変更する必要が少なくなります。

一方で、複雑なクエリではパフォーマンスが低下する可能性がある、ORMの学習コストがかかるといったデメリットも存在しますが、多くのアプリケーションではメリットが大きく上回ります。

## Prisma入門：次世代のORM


Prismaは、Node.jsとTypeScript向けの「次世代ORM」と呼ばれています。他のORMと比較して、特に**型安全性**と**開発者体験（DX）**に重点を置いているのが大きな特徴です。

Prismaは、単一のライブラリではなく、主に3つのツールで構成されています。

1.  **Prisma Client**: 型安全なデータベースクライアントです。データベースのスキーマから自動生成され、補完が効くメソッドで直感的にクエリを記述できます。
2.  **Prisma Migrate**: 宣言的なマイグレーションツールです。SQLを書かずに`schema.prisma`というファイルでデータモデルを定義するだけで、データベースのテーブル構造を管理・変更できます。
3.  **Prisma Studio**: データベースのデータを視覚的に閲覧・編集できるGUIツールです。開発中にデータの状態を確認するのに非常に便利です。

これらのツールが連携し、データベースに関する開発体験を飛躍的に向上させます。

## 開発環境の準備：Next.js + Prisma

それでは、実際にNext.jsプロジェクトにPrismaを導入していきましょう。

### 1. Next.jsプロジェクトのセットアップ

まず、`create-next-app`を使ってTypeScriptベースのNext.jsプロジェクトを作成します。

```bash
npx create-next-app@latest nextjs-prisma-app --typescript
cd nextjs-prisma-app
```

### 2. Prisma CLIのインストール

次に、Prismaのコマンドラインツール（CLI）を開発者依存関係としてプロジェクトにインストールします。

```bash
npm install prisma --save-dev
```

### 3. Prismaの初期設定

以下のコマンドを実行して、Prismaの初期設定を行います。

```bash
npx prisma init
```

このコマンドを実行すると、プロジェクトのルートに以下のものが作成されます。

*   **`prisma` ディレクトリ**: この中に `schema.prisma` ファイルが生成されます。
*   **`.env` ファイル**: データベースの接続情報などを格納する環境変数ファイルです。

`schema.prisma`はPrismaの設定とデータモデルを定義する中心的なファイルで、`.env`はパスワードなどの機密情報をコードから分離するために使われます。`.gitignore`に`.env`が追加されていることも確認しておきましょう。

## データモデルの設計：`schema.prisma`

`prisma/schema.prisma` ファイルが、Prismaの心臓部です。ここでデータベース接続やデータモデルを定義します。

初期状態の `schema.prisma` はこのようになっています。

```prisma:prisma/schema.prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

*   `generator client`: Prisma Clientを生成するための設定です。
*   `datasource db`: 接続するデータベースの種類と接続URLを指定します。デフォルトはPostgreSQLですが、MySQLやSQLiteなども利用可能です。`url`は`.env`ファイル内の`DATABASE_URL`を参照するように設定されています。

### `.env`ファイルの設定

今回はローカル開発で手軽に試せるSQLiteを使用してみましょう。`.env`ファイルを以下のように編集します。

```env:.env
# Environment variables declared in this file are automatically made available to Prisma.
# See the documentation for more detail: https://pris.ly/d/prisma-schema#accessing-environment-variables-from-the-schema

# Prisma supports the native connection string format for PostgreSQL, MySQL, SQLite, SQL Server, MongoDB and CockroachDB.
# See the documentation for all the connection string options: https://pris.ly/d/connection-strings

DATABASE_URL="file:./dev.db"
```

そして、`schema.prisma`の`datasource`ブロックをSQLite用に変更します。

```prisma:prisma/schema.prisma
datasource db {
  provider = "sqlite" // "postgresql" から "sqlite" へ変更
  url      = env("DATABASE_URL")
}
```

### モデルの定義

`schema.prisma`に`model`ブロックを追加して、アプリケーションで使うデータモデルを定義します。ここでは、ブログアプリケーションを想定して「ユーザー」と「投稿」のモデルを定義してみましょう。

```prisma:prisma/schema.prisma
// (generatorとdatasourceの設定は省略)

model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  posts Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

*   `model User`と`model Post`が、それぞれデータベースの`User`テーブルと`Post`テーブルに対応します。
*   `id Int @id @default(autoincrement())`: `id`という名前の整数（`Int`）型のフィールドを定義し、`@id`で主キーに指定、`@default(autoincrement())`で自動採番されるように設定しています。
*   `String`, `Boolean`: フィールドの型を定義します。`?`を付けると、そのフィールドがオプショナル（`NULL`を許容）になります。
*   `@unique`: そのフィールドの値がテーブル内で一意であることを保証する制約です。
*   `@relation`: モデル間のリレーションを定義します。`Post`モデルの`author`フィールドは`User`モデルと関連付いており、`authorId`が外部キーとして機能します。
*   `posts Post[]`: `User`モデル側から、そのユーザーが持つ`Post`のリストにアクセスできるように定義しています。

## データベースの構築：Prisma Migrate

スキーマの定義が完了したら、`prisma migrate`コマンドを使って、この定義を実際のデータベースに反映させます。

```bash
npx prisma migrate dev --name init
```

*   `prisma migrate dev`: 開発用のマイグレーションコマンドです。
*   `--name init`: このマイグレーションに`init`という名前を付けています。

このコマンドを実行すると、Prismaは以下の処理を自動で行います。

1.  `prisma/migrations`ディレクトリに、スキーマの変更内容を記録したSQLファイル（マイグレーションファイル）を作成します。
2.  そのSQLファイルを実行して、データベース（今回は`dev.db`というSQLiteファイル）に`User`テーブルと`Post`テーブルを作成します。
3.  Prisma Clientを最新のスキーマに合わせて再生成します。

これで、定義したモデル通りのデータベースが構築されました。

## データの操作：Prisma ClientのCRUD処理

データベースとやり取りするための`Prisma Client`が生成されたので、これを使ってデータの操作（CRUD: Create, Read, Update, Delete）をしてみましょう。

### Prisma Clientのインスタンス化（Next.jsでのベストプラクティス）

Next.jsの開発サーバーはホットリロード（コード変更時に自動でリロード）機能を持っています。その際、何も対策をしないとリロードのたびにPrisma Clientの新しいインスタンスが大量に生成され、データベース接続を使い果たしてしまう可能性があります。

これを防ぐため、グローバルオブジェクトにPrisma Clientのインスタンスを保存し、シングルトン（常に単一のインスタンス）として利用するのがベストプラクティスとされています。

プロジェクトのルートに`lib/prisma.ts`というファイルを作成し、以下のコードを記述します。

```typescript:lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// グローバルオブジェクトにPrismaClientのインスタンスを保持するための型拡張
const globalForPrisma = global as unknown as {
  prisma: PrismaClient | undefined;
};

// prismaインスタンスをエクスポート
// production環境では常に新しいインスタンスを生成
// development環境ではグローバルオブジェクトに保存されたインスタンスを再利用
export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: ['query'], // 実行されるクエリをコンソールに出力する
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

これ以降、データベースを操作する際は、この`lib/prisma.ts`から`prisma`をインポートして使います。

### Next.js API Routesでの実装例

Next.jsのAPI Routesを使って、ユーザーを作成し、一覧を取得するAPIを作成してみましょう。

#### ユーザー作成API (`/api/users/create`)

`pages/api/users/create.ts`

```typescript:pages/api/users/create.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { prisma } from '../../../lib/prisma';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method Not Allowed' });
  }

  try {
    const { email, name } = req.body;
    const newUser = await prisma.user.create({
      data: {
        email,
        name,
      },
    });
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).json({ message: 'Something went wrong' });
  }
}
```

#### ユーザー一覧取得API (`/api/users`)

`pages/api/users/index.ts`

```typescript:pages/api/users/index.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { prisma } from '../../../lib/prisma';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    // findManyで全ユーザーを取得し、そのユーザーが持つ投稿（posts）も一緒に取得する
    const users = await prisma.user.findMany({
      include: {
        posts: true,
      },
    });
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ message: 'Something went wrong' });
  }
}
```

*   `prisma.user.create()`: `user`テーブルに新しいレコードを作成します。
*   `prisma.user.findMany()`: `user`テーブルから複数のレコードを取得します。
*   `include: { posts: true }`: `findMany`や`findUnique`などのクエリで、リレーション関係にある`Post`のデータも同時に取得するためのオプションです。

このように、Prisma ClientはTypeScriptの型定義に基づいており、`prisma.user.`まで入力すると`create`, `findMany`, `update`などの利用可能なメソッドがエディタ上で補完されます。これにより、タイプミスを防ぎ、非常に快適にコーディングを進めることができます。

## 開発を加速する：Prisma Studio

Prismaには、`Prisma Studio`という非常に強力なGUIツールが同梱されています。
ターミナルで以下のコマンドを実行してください。

```bash
npx prisma studio
```

ブラウザで`http://localhost:5555`が開き、下のような画面が表示されます。

![Prisma Studio](https://www.prisma.io/images/docs/studio/studio-all-models-dark.png)

この画面から、

*   `User`や`Post`モデル（テーブル）のデータを一覧表示
*   GUI操作でのレコードの追加、編集、削除
*   フィルタリングやソート

などが直感的に行えます。APIを実装しながら、データが正しく登録・更新されているかをリアルタイムで確認できるため、開発効率が劇的に向上します。

## 実践編：Next.jsページとの連携

最後に、作成したAPIをNext.jsのページコンポーネントから利用し、画面にデータを表示してみましょう。

ここでは、サーバーサイドでデータを取得する`getServerSideProps`を使います。

`pages/users.tsx`というファイルを作成します。

```tsx:pages/users.tsx
import type { GetServerSideProps, NextPage } from 'next';
import { prisma } from '../lib/prisma';
import { User, Post } from '@prisma/client';

// サーバーから渡されるpropsの型定義
// UserモデルにPostの配列が含まれる形にする
type UsersPageProps = {
  users: (User & {
    posts: Post[];
  })[];
};

// ページコンポーネント
const UsersPage: NextPage<UsersPageProps> = ({ users }) => {
  return (
    <div>
      <h1>Users</h1>
      {users.map((user) => (
        <div key={user.id} style={{ border: '1px solid #ccc', margin: '10px', padding: '10px' }}>
          <h2>{user.name} ({user.email})</h2>
          <h3>Posts:</h3>
          <ul>
            {user.posts.length > 0 ? (
              user.posts.map((post) => (
                <li key={post.id}>
                  <h4>{post.title}</h4>
                  <p>{post.content}</p>
                </li>
              ))
            ) : (
              <p>No posts yet.</p>
            )}
          </ul>
        </div>
      ))}
    </div>
  );
};

export default UsersPage;

// サーバーサイドで実行される関数
export const getServerSideProps: GetServerSideProps = async () => {
  // DBから直接データを取得する
  const users = await prisma.user.findMany({
    include: {
      posts: true, // ユーザーに関連する投稿も取得
    },
  });

  return {
    props: {
      users,
    },
  };
};
```

このコードでは、`getServerSideProps`内でPrisma Clientを直接使用して全ユーザーとその投稿を取得し、ページのpropsとして渡しています。クライアントサイドでは、渡された`users`配列を`map`で展開して表示しています。

`getServerSideProps`はページのリクエストごとにサーバー上で実行されるため、常に最新のデータベースの状態がページに反映されます。

