---
title: "shadcn/ui とは？次世代のUIコンポーネント管理術を徹底解説！"
emoji: "🧩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "react", "tailwindcss", "radixui", "shadcnui"]
published: true
---

## はじめに

ReactやNext.jsでアプリケーションを開発する際、ボタンやダイアログ、フォームといったUIコンポーネントを効率的に構築するために、多くの開発者がUIライブラリを利用します。MUI (Material-UI) や Chakra UI などが有名ですね。

これらのライブラリは非常に高機能で便利ですが、一方でこんな悩みを感じたことはありませんか？

*   ライブラリのデザインが強すぎて、自分たちのプロダクトのデザインに合わせるのが大変…
*   細かい部分をカスタマイズしたいだけなのに、複雑なAPIを調べたり、`!important`でスタイルを上書きしたりする必要がある…
*   使わないコンポーネントまでバンドルされて、アプリケーションが重くなってしまう…
*   ライブラリのバージョンアップに追従するのが大変…

**shadcn/ui** は、こうした従来のUIライブラリが抱える課題を、まったく新しいアプローチで解決する、今最も注目されているツールキットです。

この記事では、`shadcn/ui`がなぜ「単なるUIライブラリではない」のか、その革新的なコンセプトと使い方を分かりやすく解説します。

## shadcn/ui とは？

`shadcn/ui`を理解するための最も重要なポイントは、これです。

**`shadcn/ui`は、NPMでインストールする「ライブラリ」ではありません。**

一言でいうと、`shadcn/ui`は **「美しくデザインされ、アクセシビリティにも配慮されたUIコンポーネントのソースコードを、あなたのプロジェクトに直接コピーするためのツール」** です。

従来のUIライブラリが、アプリケーションの外部依存（`node_modules`）として存在する「ブラックボックス」だったのに対し、`shadcn/ui`はコンポーネントのソースコードそのものをあなたのプロジェクト内に提供します。これにより、あなたはコンポーネントの完全な**所有権**を持つことになります。

### shadcn/uiの革新的な特徴

#### 1. あなたのコードになる
CLIコマンドを使ってコンポーネントを追加すると、そのコンポーネントの`.tsx`ファイルがあなたのプロジェクト内の`components/ui`ディレクトリに直接生成されます。これはもうライブラリのコードではなく、**あなた自身のコード**です。そのため、好きなように構造を変えたり、スタイルを調整したり、機能を追加・削除したりと、自由自在にカスタマイズできます。

#### 2. 依存関係からの解放
コンポーネントはあなたのプロジェクトの一部なので、`shadcn/ui`自体のバージョンアップに振り回されることはありません。ライブラリの破壊的変更に怯える必要はなく、自分たちのペースでコンポーネントを管理・進化させることができます。

#### 3. Tailwind CSSによる高いデザイン自由度
`shadcn/ui`のコンポーネントは、**Tailwind CSS** をベースにスタイリングされています。これは、色、フォント、余白、角丸といったデザインの基本要素（デザイントークン）が、すべてあなたのプロジェクトの`tailwind.config.js`ファイルに依存することを意味します。
この設定ファイルを変更するだけで、すべてのコンポーネントのデザインを、あなたのプロダクトのブランドに合わせて一括で変更できます。

#### 4. Radix UIによる堅牢な土台
コンポーネントの見た目（スタイル）はTailwind CSSが担当しますが、その裏側の複雑な振る舞いやアクセシビリティ（キーボード操作やスクリーンリーダー対応など）は、ヘッドレスUIライブラリの **Radix UI** が担当しています。これにより、私たちは難しいロジックやアクセシビリティについて深く悩むことなく、高品質なコンポーネントの恩恵を受けることができます。

## Next.js (App Router) での基本的な使い方

`shadcn/ui`のコンセプトが分かったところで、実際にNext.jsプロジェクトで使う際の基本的な流れを見ていきましょう。

### ステップ1: 初期セットアップ (init)

まず、CLIを使ってあなたのプロジェクトに`shadcn/ui`をセットアップします。

```bash
npx shadcn-ui@latest init
```

このコマンドを実行すると、いくつかの質問（TypeScriptの使用、スタイルの種類、`tailwind.config.js`のパスなど）をされます。これに答えると、必要な設定やユーティリティファイル、そして設定ファイルである`components.json`が自動で生成されます。

### ステップ2: 必要なコンポーネントを追加 (add)

次に、使いたいコンポーネントを個別に追加します。例えば、`Button`コンポーネントと`Dialog`コンポーネントが必要な場合は、以下のコマンドを実行します。

```bash
npx shadcn-ui@latest add button dialog
```

実行後、あなたのプロジェクトの`components/ui`ディレクトリに`button.tsx`と`dialog.tsx`というファイルが作成されているはずです。

### ステップ3: コンポーネントを使用する

あとは、自作のコンポーネントをインポートするのと全く同じように、追加したコンポーネントを使います。

```tsx:app/page.tsx
import { Button } from "@/components/ui/button"
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog"

export default function Home() {
  return (
    <div className="p-10">
      <Dialog>
        <DialogTrigger asChild>
          <Button variant="outline">ダイアログを開く</Button>
        </DialogTrigger>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>shadcn/uiへようこそ！</DialogTitle>
            <DialogDescription>
              これはあなたのプロジェクトにコピーされたコンポーネントです。
              自由に編集できます！
            </DialogDescription>
          </DialogHeader>
        </DialogContent>
      </Dialog>
    </div>
  )
}
```

### ステップ4: コンポーネントをカスタマイズする

`shadcn/ui`の真価はここからです。例えば、「ボタンの角丸をもっと大きくしたい」と思ったら、ライブラリのドキュメントを探す必要はありません。
**`components/ui/button.tsx`ファイルを直接開いて編集すれば良いのです。**

```tsx:components/ui/button.tsx
// ...省略
const buttonVariants = cva(
  "inline-flex items-center justify-center whitespace-nowrap text-sm font-medium ring-offset-background transition-colors ...",
  {
    variants: {
      // ...省略
    },
    defaultVariants: {
      // ...省略
    },
  }
)
// 例えば、ここに `rounded-full` を追加すれば、すべてのボタンが丸くなります。
// Или, "rounded-md" を "rounded-xl" に変更するだけで、角丸の大きさを変えられます。
```
このように、コンポーネントのソースコードを直接コントロールできるのが、`shadcn/ui`の最大の強みです。

