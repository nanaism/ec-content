---
title: "MDXで構築するモダンなコンテンツサイト"
emoji: "✖️"
type: "tech"
topics: ["nextjs", "react", "markdown", "mdx"]
published: true
---

## はじめに：MDXとは何か？

MDXは「コンポーネント時代のためのマークダウン（Markdown for the component era）」をコンセプトにした、マークダウンファイルの内部でJSXコンポーネントを直接記述できるようにする技術です。 [5] これにより、静的な文章記述に優れたMarkdownの手軽さと、動的なUIを構築できるReactコンポーネントのインタラクティブ性をシームレスに両立できます。 [5]

例えば、以下のようにMarkdownの文章の中にReactコンポーネントを埋め込むことができます。

```mdx:example.mdx
# 私のブログへようこそ

これはMarkdownで書かれた文章です。

<MyButton>クリックしてください！</MyButton>

インタラクティブな要素も簡単に追加できます。
```

Next.jsとMDXを組み合わせることで、開発者はブログ記事、ドキュメント、ポートフォリオサイトなどを、コンテンツ管理のしやすさと高度な表現力を兼ね備えた形で効率的に構築できます。 [2] Next.jsのApp Routerでは、`.mdx`ファイルを置くだけで自動的にページとしてルーティングされ、サーバーコンポーネントとしてのレンダリングもサポートされています。 [2]

この記事では、Next.jsのApp Router環境でMDXを導入し、基本的な使い方から応用までをコード例と共に詳しく解説していきます。

## 1. 環境構築：Next.jsにMDXを導入する

まずは、Next.jsプロジェクトにMDXを導入するための環境を構築します。

### Step 1: Next.jsプロジェクトの作成

既にプロジェクトがある場合はこのステップをスキップしてください。ない場合は、以下のコマンドで新しいNext.jsプロジェクトを作成します。

```bash
npx create-next-app@latest my-mdx-blog
```

### Step 2: MDX関連パッケージのインストール

次に、MDXをNext.jsで利用するために必要なパッケージをインストールします。 [2]

```bash
npm install @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```

- `@next/mdx`: Next.jsでMDXを統合するための公式プラグインです。
- `@mdx-js/loader`: webpackローダーとしてMDXファイルを処理します。
- `@mdx-js/react`: MDXコンテンツをReactコンポーネントとしてレンダリングするためのコアライブラリです。
- `@types/mdx`: TypeScript環境での型定義を提供します。

### Step 3: next.config.mjs の設定

プロジェクトのルートにある`next.config.mjs`（なければ作成）を編集し、MDXプラグインを有効にします。これにより、`.mdx`や`.md`拡張子のファイルがページとして認識されるようになります。 [2]

```javascript:next.config.mjs
import createMDX from '@next/mdx'

/** @type {import('next').NextConfig} */
const nextConfig = {
  // .mdxファイルをページとして認識させる
  pageExtensions: ['js', 'jsx', 'ts', 'tsx', 'md', 'mdx'],
}

const withMDX = createMDX({
  // ここにremarkやrehypeのプラグインを追加できます
  options: {
    remarkPlugins: [],
    rehypePlugins: [],
  },
})

export default withMDX(nextConfig)
```

これで、Next.jsプロジェクトでMDXを使う準備が整いました。

## 2. 基本的な使い方：MDXでページを作成する

環境が整ったので、実際にMDXを使ってページを作成してみましょう。

### Step 1: インタラクティブなコンポーネントの作成

まず、MDXファイル内で使用する簡単なReactコンポーネントを作成します。`components`ディレクトリを作成し、その中に`Counter.tsx`というファイルを作成します。

```tsx:components/Counter.tsx
'use client'

import { useState } from 'react'

export function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div style={{ border: '1px solid #ccc', padding: '16px', borderRadius: '8px' }}>
      <p>現在のカウント: {count}</p>
      <button onClick={() => setCount(count + 1)}>カウントアップ</button>
    </div>
  )
}
```
`'use client'`ディレクティブは、このコンポーネントがクライアントサイドでインタラクティブに動作するために必要です。

### Step 2: MDXページの作成

`app`ディレクトリに`my-first-post`のような名前でフォルダを作成し、その中に`page.mdx`というファイルを作成します。

```mdx:app/my-first-post/page.mdx
import { Counter } from '@/components/Counter'

# MDXの最初の投稿

これはMarkdownで書かれた最初のセクションです。リストも簡単です。

- アイテム1
- アイテム2
- アイテム3

## Reactコンポーネントの埋め込み

そして、こちらがインポートしたReactコンポーネントです。
MDXの真価がここにあります。

<Counter />

このように、静的なドキュメントの中に動的なアプリケーションをシームレスに統合できます。
```

この状態で開発サーバーを起動（`npm run dev`）し、`/my-first-post`にアクセスすると、Markdownコンテンツとインタラクティブなカウンターコンポーネントが共に表示されることが確認できます。 [4]

## 3. コンポーネントのカスタマイズ：デザインを統一する

MDXの強力な機能の一つに、Markdownが変換する標準的なHTML要素（`<h1>`, `<p>`, `<a>`など）を、自作のReactコンポーネントに差し替える機能があります。これにより、サイト全体で一貫したデザインシステムを適用できます。

この差し替えは、`mdx-components.tsx`という規約ファイルを作成することで実現します。 [2]

### Step 1: `mdx-components.tsx`の作成

プロジェクトのルート（`app`ディレクトリと同じ階層）に`mdx-components.tsx`ファイルを作成します。

```tsx:mdx-components.tsx
import type { MDXComponents } from 'mdx/types'
import { ReactNode } from 'react'

export function useMDXComponents(components: MDXComponents): MDXComponents {
  return {
    // h1タグをカスタマイズ
    h1: ({ children }) => (
      <h1 style={{ fontSize: '2.5rem', color: '#1a202c', borderBottom: '2px solid #718096' }}>
        {children}
      </h1>
    ),
    // pタグをカスタマイズ
    p: ({ children }) => <p style={{ fontSize: '1.125rem', lineHeight: 1.6 }}>{children}</p>,
    // aタグをカスタマイズ
    a: ({ href, children }) => (
      <a href={href} style={{ color: '#3182ce' }} target="_blank" rel="noopener noreferrer">
        {children}
      </a>
    ),
    // 既存のコンポーネントもマージする
    ...components,
  }
}
```

### Step 2: ルートレイアウトでの適用

App Routerを使用している場合、Next.jsは自動的に`mdx-components.tsx`ファイルを認識し、MDXコンテンツに適用します。`@next/mdx`がこのファイルを自動的に使用するため、特別な設定は不要です。 [2]

この設定により、プロジェクト内のすべての`.mdx`ファイルで`<h1>`や`<p>`が自動的にスタイル付けされたカスタムコンポーネントに置き換わり、デザインの一貫性が保たれます。

## 4. メタデータの扱い：Frontmatterを活用する

ブログ記事にはタイトル、公開日、概要といったメタデータが不可欠です。MDXでは、`export`構文を使ってこれらの情報を簡単に定義・利用できます。

### MDXファイル内でメタデータを定義

`page.mdx`ファイルの先頭で、`metadata`オブジェクトをエクスポートします。これはNext.jsの`metadata`規約と統合されます。

```mdx:app/my-first-post/page.mdx
import { Counter } from '@/components/Counter'

export const metadata = {
  title: 'MDXの最初の投稿',
  description: 'Next.jsとMDXの基本的な使い方を学びます。',
  date: '2025-06-23',
}

# {metadata.title}

*公開日: {metadata.date}*

これはMarkdownで書かれた最初のセクションです。リストも簡単です。

{/* ...以降のコンテンツ... */}
```

このように定義したメタデータは、Next.jsによって自動的にページの`<head>`タグ内に`<title>`や`<meta name="description">`として設定されます。また、`{metadata.title}`のように本文中で変数を展開して利用することも可能です。

## 5. 応用編：より高度な機能の実装

基本的な使い方に慣れたら、プラグインを使ってさらにMDXを強化しましょう。`remark`（MarkdownのASTを操作）と`rehype`（HTMLのASTを操作）のプラグインエコシステムを利用できます。

### シンタックスハイライトの導入

コードブロックを美しく見せるために、`rehype-pretty-code`を導入します。 [3]

**Step 1: パッケージのインストール**
```bash
npm install rehype-pretty-code shikiji
```

**Step 2: `next.config.mjs`の更新**
設定ファイルに`rehype-pretty-code`を追加します。

```javascript:next.config.mjs
import createMDX from '@next/mdx'
import rehypePrettyCode from 'rehype-pretty-code'

/** @type {import('rehype-pretty-code').Options} */
const options = {
  theme: 'one-dark-pro', // テーマを選択
  onVisitLine(node) {
    // 空行はハイライトしないようにする
    if (node.children.length === 0) {
      node.children = [{ type: 'text', value: ' ' }]
    }
  },
  onVisitHighlightedLine(node) {
    // 強調表示された行にクラスを追加
    node.properties.className.push('line--highlighted')
  },
  onVisitHighlightedWord(node) {
    // 強調表示された単語にクラスを追加
    node.properties.className = ['word--highlighted']
  },
}

/** @type {import('next').NextConfig} */
const nextConfig = {
  pageExtensions: ['js', 'jsx', 'ts', 'tsx', 'md', 'mdx'],
}

const withMDX = createMDX({
  options: {
    remarkPlugins: [],
    rehypePlugins: [[rehypePrettyCode, options]],
  },
})

export default withMDX(nextConfig)
```

これで、MDXファイル内のコードブロックが自動的にシンタックスハイライトされるようになります。
