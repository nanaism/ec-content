---
title: "Framer Motionとは？Next.jsでリッチなアニメーションを驚くほど簡単に。"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "react", "animation", "framermotion"]
published: true
---

## はじめに

モダンなWebサイトやアプリケーションにおいて、**アニメーション**は単なる飾りではありません。ユーザーの操作に対するフィードバックを分かりやすく示したり、ページの遷移をスムーズに見せたり、ブランドの世界観を表現したりと、ユーザー体験（UX）を向上させるための非常に重要な要素です。

しかし、いざアニメーションを実装しようとすると、

*   CSSの `@keyframes` や `transition` は記述が複雑で、細かい制御が難しい…
*   JavaScriptでアニメーションを自作するのは、パフォーマンスも考慮せねばならず大変…
*   Reactのコンポーネントのライフサイクルとアニメーションをどう連携させればいいか分からない…

といった壁にぶつかることがよくあります。

そんな悩みを解決してくれるのが、今回ご紹介する **Framer Motion** です。このライブラリを使えば、まるで魔法のように、簡単かつ直感的にリッチなアニメーションをReactアプリケーションに組み込むことができます。

## Framer Motion とは？

Framer Motionは、一言でいうと **「Reactアプリケーションのための、本番環境で使える強力なアニメーションライブラリ」** です。

最大の特徴は、**宣言的な記法**にあります。
これは「どうやって動かすか（How）」を細かく記述するのではなく、「どういう状態になってほしいか（What）」を記述するだけで、Framer Motionがその間の滑らかなアニメーションを自動的に生成してくれる、という考え方です。

例えば、「この箱を右に100px動かしたい」という場合、複雑な計算やフレームごとの処理を書く必要はありません。ただ「`x`座標が`100`になってほしい」と指定するだけでいいのです。

### Framer Motionの主なメリット

#### 1. 驚くほどシンプルな記述
HTMLタグ（`div`, `h1`, `button`など）の前に `motion.` を付けるだけで、その要素がアニメーション可能になります。そして、`animate`や`transition`といったシンプルなプロパティを指定するだけで、リッチな動きを実装できます。

#### 2. Reactとの高い親和性
Reactのコンポーネント指向の考え方と非常に相性が良く、コンポーネントの状態（state）の変化に応じてアニメーションを簡単にトリガーできます。例えば、「ボタンがクリックされたら、メニューがスライドして表示される」といったインタラクションを直感的に記述できます。

#### 3. 物理ベースのリアルな動き
単純な直線的な動きだけでなく、「バネ（spring）」のような物理法則に基づいたリアルで心地よいアニメーションを簡単に作れます。これにより、アプリケーションがより生き生きとした印象になります。

#### 4. 豊富なアニメーション機能
*   **ジェスチャーアニメーション**: ホバー、タップ、ドラッグ、フォーカスといったユーザーの操作に応じたアニメーション。
*   **レイアウトアニメーション**: 要素のサイズや位置が変わった際に、自動で滑らかにアニメーションさせます。
*   **登場・退場アニメーション (`AnimatePresence`)**: Reactコンポーネントがマウント・アンマウントされる際の複雑なアニメーションを驚くほど簡単に実装できます。

## Next.js (App Router) での基本的な使い方

それでは、Next.js v15（App Router環境）でFramer Motionを使い、簡単なアニメーションを実装する手順を見ていきましょう。

### ステップ1: パッケージのインストール

まず、プロジェクトに`framer-motion`をインストールします。

```bash
npm install framer-motion
```

### ステップ2: Client Componentで`motion`コンポーネントを使用する

Framer Motionはユーザーのブラウザ上で動作するアニメーションライブラリなので、**Client Component**内で使用する必要があります。ファイルの先頭に`"use client"`を記述しましょう。

基本的な使い方は、アニメーションさせたいHTMLタグの前に`motion.`を付けるだけです。例えば、`<div>`をアニメーションさせたい場合は`<motion.div>`を使用します。

### ステップ3: 簡単なアニメーションを実装する

`initial`、`animate`、`transition`プロパティを使って、コンポーネントが表示された時に動くアニメーションを作成してみましょう。

*   `initial`: アニメーションの開始状態
*   `animate`: アニメーションの終了状態
*   `transition`: アニメーションの挙動（時間、遅延、イージングなど）

```tsx:app/components/AnimatedBox.tsx
"use client"

import { motion } from "framer-motion"

export default function AnimatedBox() {
  return (
    <motion.div
      className="w-24 h-24 bg-blue-500 rounded-lg"
      // 初期状態: 透明で、左に100pxずれている
      initial={{ opacity: 0, x: -100 }}
      // アニメーション後の状態: 不透明で、元の位置に戻る
      animate={{ opacity: 1, x: 0 }}
      // アニメーションの挙動: 0.5秒かけて実行
      transition={{ duration: 0.5 }}
    />
  )
}
```
このコンポーネントをページに配置するだけで、左からスライドインしながら表示される青い箱が完成します。

### ステップ4: ジェスチャーアニメーションを追加する

Framer Motionの真骨頂は、インタラクティブなアニメーションです。`whileHover`や`whileTap`プロパティを追加するだけで、マウスホバー時やクリック時のアニメーションを簡単に実装できます。

```tsx:app/components/InteractiveButton.tsx
"use client"

import { motion } from "framer-motion"

export default function InteractiveButton() {
  return (
    <motion.button
      className="px-6 py-3 font-bold text-white bg-green-500 rounded-full"
      // マウスホバー時のアニメーション
      whileHover={{ scale: 1.1, rotate: 5 }}
      // クリック（プレス）時のアニメーション
      whileTap={{ scale: 0.9, rotate: -5 }}
    >
      Click Me!
    </motion.button>
  )
}
```
このボタンは、マウスを乗せると少し大きくなって傾き、クリックすると少し小さくなって反対に傾きます。たった2行の追加で、これだけリッチなフィードバックが実現できるのです。

