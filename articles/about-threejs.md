---
title: "three.jsで創造する、次世代3D Webフロントエンド開発"
emoji: "🔺"
type: "tech"
topics: ["nextjs", "react",_ "threejs", "javascript", "frontend"]
published: true
---

## はじめに: なぜ今、Webで3Dなのか？

近年、Webサイトやアプリケーションにおける3D表現の需要が急速に高まっています。製品の3Dビューワー、インタラクティブなデータビジュアライゼーション、没入感のあるポートフォリオサイトなど、その活用範囲は多岐にわたります。これを技術的に可能にしているのが、ブラウザ上で3Dグラフィックスを扱うための標準APIである**WebGL**です。

しかし、WebGLを直接扱うのは複雑で多くの知識を要します。そこで登場するのが**three.js**です。three.jsは、WebGLをより簡単かつ直感的に利用できるようにしたJavaScriptライブラリであり、3Dコンテンツ制作のデファクトスタンダードとしての地位を確立しています。 

本記事では、このthree.jsを、現代のフロントエンド開発で絶大な人気を誇るReactフレームワーク**Next.js**と組み合わせて利用する方法を、基礎から実践まで徹底的に解説します。特に、現代の開発で主流となっているエコシステム、`@react-three/fiber`と`@react-three/drei`を活用し、宣言的かつコンポーネントベースで効率的に3Dコンテンツを構築する手法を学びます。 

### 本記事で目指すゴール

最終的には、Next.jsのページ内にインタラクティブな3Dオブジェクトを配置し、ライトやシャドウ、外部から読み込んだ3Dモデルを使ってリッチな表現ができるようになることを目指します。

## 第1章: three.jsのコアコンセプトを理解する

three.jsを効果的に使用するためには、まずその基本的な概念を理解する必要があります。  これらの概念は、現実世界の写真撮影や映画制作に例えると分かりやすいです。

### 3D空間を構成する三大要素

1.  **シーン (Scene):**
    これは、すべての3Dオブジェクト、ライト、カメラを配置する「舞台」や「仮想空間」そのものです。  `new THREE.Scene()` で作成します。

2.  **カメラ (Camera):**
    シーン内のどの部分を、どのように見るかを決定します。  主に2種類あります。
    *   `PerspectiveCamera`（透視投影カメラ）: 遠くのものが小さく見える、人間や写真の視点に近いカメラです。
    *   `OrthographicCamera`（平行投影カメラ）: 遠近感なく、オブジェクトを平行に描画します。設計図やアイソメトリックなゲームなどで使われます。

3.  **レンダラー (Renderer):**
    カメラで撮影したシーンの映像を、実際にブラウザの画面（HTMLの`<canvas>`要素）に描き出す「映写機」の役割を担います。  `WebGLRenderer` が最も一般的に使用されます。

### オブジェクトを形作る要素

シーンに配置するオブジェクト（例: 立方体、球体）は、主に以下の3つの要素で構成されます。

1.  **ジオメトリ (Geometry):**
    オブジェクトの「形状」を定義する頂点データの集まりです。  `BoxGeometry`（立方体）、`SphereGeometry`（球体）など、様々な基本形状が用意されています。

2.  **マテリアル (Material):**
    オブジェクトの「表面の質感」を定義します。色、テクスチャ、光の反射具合（金属質か、ざらざらしているか等）を設定します。  `MeshBasicMaterial`（光の影響を受けない）、`MeshStandardMaterial`（物理ベースのリアルな質感）などがあります。

3.  **メッシュ (Mesh):**
    ジオメトリ（形状）とマテリアル（質感）を組み合わせた最終的なオブジェクトです。  このメッシュをシーンに追加することで、3D空間にオブジェクトが配置されます。

これらをコードで表現すると、以下のようになります（これはthree.js単体での記述例です）。

```javascript
// 1. シーン、カメラ、レンダラーの準備
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);
document.body.appendChild(renderer.domElement);

// 2. ジオメトリとマテリアルの準備
const geometry = new THREE.BoxGeometry(1, 1, 1);
const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });

// 3. メッシュを作成し、シーンに追加
const cube = new THREE.Mesh(geometry, material);
scene.add(cube);

// 4. レンダリング
function animate() {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
}
animate();
```

## 第2章: 開発環境のセットアップ (Next.js + three.js)

ここからは、Next.jsプロジェクトでthree.jsを扱うためのモダンな開発環境を構築します。

### Next.jsプロジェクトの作成

まず、ターミナルで以下のコマンドを実行し、新しいNext.jsプロジェクトを作成します。 

```bash
npx create-next-app@latest next-three-app
```

いくつかの質問が表示されるので、好みに合わせて設定してください（本記事ではApp Routerの使用を前提とします）。 

### 必要なライブラリのインストール

次に、プロジェクトディレクトリに移動し、three.js関連のライブラリをインストールします。

```bash
cd next-three-app
npm install three @react-three/fiber @react-three/drei
```

ここでインストールしたライブラリの役割は非常に重要です。

*   **`@react-three/fiber` (R3F):**
    three.jsのためのReactレンダラーです。  R3Fを使用すると、three.jsのオブジェクト（シーン、メッシュ、ライト等）を、ReactコンポーネントのようにJSXで宣言的に記述できます。 [1, 2] これにより、Reactの状態管理やフック、コンポーネントの再利用性といったエコシステムの恩恵を最大限に活用できます。 [2, 5]

*   **`@react-three/drei`:**
    R3Fのためのヘルパーライブラリです。カメラコントロール（`OrbitControls`）、便利なジオメトリ、画像や3Dモデルのローダーなどを、使いやすいコンポーネントやフックとして提供します。  これにより、定型的なコードの記述量を大幅に削減できます。

### クライアントコンポーネントの準備

three.jsはブラウザのAPI（WebGL）に依存するため、クライアントサイドで実行される必要があります。Next.jsのApp Routerでは、デフォルトでコンポーネントはサーバーコンポーネントとして扱われるため、3Dシーンを描画するコンポーネントには`'use client'`ディレクティブをファイルの先頭に記述する必要があります。 [2, 3]

また、3Dコンポーネントは初期バンドルサイズを削減するために、`next/dynamic`を使って動的にインポートするのが一般的です。

## 第3章: 最初の3DシーンをNext.js上に描画する

環境が整ったので、実際にNext.js上に3Dオブジェクトを表示してみましょう。

まず、`components`フォルダを作成し、その中に`Scene.js`というファイルを作成します。

**`components/Scene.js`**
```javascript
'use client';

import { Canvas } from '@react-three/fiber';
import { OrbitControls } from '@react-three/drei';

// 再利用可能なBoxコンポーネント
const Box = () => {
  return (
    <mesh>
      <boxGeometry args={} />
      <meshStandardMaterial color="orange" />
    </mesh>
  );
};

// 3Dシーン全体を管理するコンポーネント
const Scene = () => {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Canvas>
        {/* 環境光: シーン全体を均等に照らす */}
        <ambientLight intensity={0.5} />
        {/* 平行光: 特定の方向から照らす。太陽光のようなイメージ */}
        <directionalLight position={} intensity={1} />
        
        <Box />

        {/* カメラコントロール: マウスで視点を操作できるようにする */}
        <OrbitControls />
      </Canvas>
    </div>
  );
};

export default Scene;
```

次に、この`Scene`コンポーネントをホームページ (`app/page.js`) で動的に読み込みます。

**`app/page.js`**
```javascript
import dynamic from 'next/dynamic';

// 'ssr: false' を指定して、サーバーサイドではレンダリングしないようにする
const Scene = dynamic(() => import('@/components/Scene'), { ssr: false });

export default function Home() {
  return (
    <main>
      <h1>My 3D App with Next.js</h1>
      <Scene />
    </main>
  );
}```

これで開発サーバーを起動 (`npm run dev`) すると、オレンジ色の立方体が表示され、マウスのドラッグで視点をグリグリと動かせるはずです。

R3Fでは、`<Canvas>`コンポーネントがthree.jsの`Scene`と`Renderer`の役割を自動的に担ってくれます。  また、three.jsのクラス名（`BoxGeometry`, `MeshStandardMaterial`など）をキャメルケース（`boxGeometry`, `meshStandardMaterial`）にしたコンポーネントとしてJSX内に記述できるのが特徴です。  `args`プロパティには、コンストラクタの引数を配列で渡します。 

## 第4章: アニメーションとユーザーインタラクション

静的なオブジェクトだけでは物足りないので、動きとインタラクションを追加しましょう。

### `useFrame`フックによるアニメーション

R3Fが提供する`useFrame`フックを使うと、毎フレーム実行される処理を記述できます。これはthree.jsの`requestAnimationFrame`ループに相当します。

先ほどの`Box`コンポーネントを修正して、回転するアニメーションを追加してみましょう。

**`components/Scene.js` (Boxコンポーネントを修正)**
```javascript
// ... (imports)
import { useRef } from 'react';
import { useFrame } from '@react-three/fiber';

const Box = () => {
  // useRefでmeshオブジェクトへの参照を保持
  const meshRef = useRef();

  // useFrameで毎フレーム実行される処理を定義
  useFrame((state, delta) => {
    // deltaは前フレームからの経過時間(秒)
    if (meshRef.current) {
      meshRef.current.rotation.x += delta;
      meshRef.current.rotation.y += delta;
    }
  });

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={} />
      <meshStandardMaterial color="orange" />
    </mesh>
  );
};

// ... (Sceneコンポーネントは変更なし)
```
`useRef`フックで`mesh`要素への参照を作成し、`useFrame`の中でその`rotation`プロパティを毎フレーム更新することで、スムーズなアニメーションが実現できます。

### オブジェクトへのマウスイベント

R3Fでは、`onClick`, `onPointerOver` (ホバー開始), `onPointerOut` (ホバー終了) といった、使い慣れたReactのイベントハンドラを3Dオブジェクトに追加できます。

ホバーした時に色が変わるインタラクションを追加してみましょう。

**`components/Scene.js` (Boxコンポーネントをさらに修正)**
```javascript
// ... (imports)
import { useRef, useState } from 'react';
import { useFrame } from '@react-three/fiber';

const Box = (props) => {
  const meshRef = useRef();
  const [hovered, setHover] = useState(false);
  const [active, setActive] = useState(false);

  useFrame((state, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.x += delta;
    }
  });

  return (
    <mesh
      {...props}
      ref={meshRef}
      scale={active ? 1.5 : 1}
      onClick={(event) => setActive(!active)}
      onPointerOver={(event) => setHover(true)}
      onPointerOut={(event) => setHover(false)}>
      <boxGeometry args={} />
      <meshStandardMaterial color={hovered ? 'hotpink' : 'orange'} />
    </mesh>
  );
};
```
`useState`を使ってホバー状態(`hovered`)とクリック状態(`active`)を管理し、それに応じてマテリアルの色やメッシュのスケールを動的に変更しています。このようにReactのstateと3D表現を直感的に結びつけられるのがR3Fの大きな強みです。

## 第5章: シーンを豊かにするライティングとシャドウ

リアルな3Dシーンには、光と影の表現が不可欠です。

*   **ライトの種類:** 
    *   `ambientLight`: 環境光。影を作らず、シーン全体を底上げするように照らします。
    *   `directionalLight`: 平行光。一方向から照らす強い光で、太陽光線のように振る舞い、影を落とすことができます。
    *   `pointLight`: 点光源。電球のように、光源から全方向に光を放ちます。
*   **シャドウ（影）:**
    影を生成するには、以下の設定が必要です。
    1.  レンダラーで影を有効にする (`<Canvas shadows>`)。
    2.  光を落とすライトに`castShadow`プロパティを設定する。
    3.  影を落とすオブジェクトに`castShadow`プロパティを設定する。
    4.  影を受けるオブジェクト（地面など）に`receiveShadow`プロパティを設定する。

地面を追加して、立方体の影が落ちるようにしてみましょう。

**`components/Scene.js` (Sceneコンポーネントを修正)**
```javascript
// ... (Boxコンポーネントは前の章のもの)

const Scene = () => {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      {/* shadowsプロパティで影を有効化 */}
      <Canvas shadows>
        <ambientLight intensity={0.5} />
        {/* castShadowプロパティでこのライトが影を落とすように設定 */}
        <directionalLight position={} intensity={1.5} castShadow />

        {/* castShadowでこのBoxが影を落とすように設定 */}
        <Box position={} castShadow />

        {/* 地面を作成 */}
        {/* receiveShadowでこの平面が影を受けるように設定 */}
        <mesh rotation={[-Math.PI / 2, 0, 0]} receiveShadow>
          <planeGeometry args={} />
          <meshStandardMaterial color="lightgray" />
        </mesh>
        
        <OrbitControls />
      </Canvas>
    </div>
  );
};
```
`Canvas`に`shadows`プロパティを追加し、各要素に`castShadow`と`receiveShadow`を設定するだけで、リアルな影が描画されます。

## 第6章: 外部3Dモデル(GLTF)のインポート

自分でジオメトリを組み合わせるだけでなく、BlenderなどのDCCツールで作成された3Dモデルを読み込むこともできます。Webで標準的に使われるフォーマットは**GLTF** (.gltf, .glb) です。

`@react-three/drei`の`useGLTF`フックを使えば、GLTFモデルの読み込みが非常に簡単になります。 

1.  Next.jsプロジェクトの`public`フォルダに、利用したい`.glb`ファイルを配置します（例: `model.glb`）。
2.  `gltf-pipeline`を使って、モデルをJSXコンポーネントに変換しておくと便利です。
    ```bash
    npx gltfjsx public/model.glb -o components/Model.js
    ```
    これにより、モデルの構造が予め解析されたコンポーネントが生成され、扱いやすくなります。

生成された`Model.js`をシーンに組み込んでみましょう。

**`components/Model.js` (gltfjsxで生成されたファイルの例)**
```javascript
/*
Auto-generated by: https://github.com/pmndrs/gltfjsx
*/
import React, { useRef } from 'react';
import { useGLTF } from '@react-three/drei';

export function Model(props) {
  const { nodes, materials } = useGLTF('/model.glb');
  return (
    <group {...props} dispose={null}>
      {/* モデルの構造に合わせてメッシュがここに記述される */}
      <mesh geometry={nodes.YourMesh.geometry} material={materials.YourMaterial} />
    </group>
  );
}

// プリロードしておくと表示がスムーズになる
useGLTF.preload('/model.glb');
```

**`components/Scene.js` (Sceneコンポーネントで読み込み)**
```javascript
// ... (imports)
import { Model } from './Model'; // 生成したモデルコンポーネントをインポート

const Scene = () => {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Canvas shadows>
        {/* ...ライトや他の設定... */}
        
        {/* モデルをシーンに追加 */}
        <Model position={[0, 0.5, 0]} />

        {/* ...地面やOrbitControls... */}
      </Canvas>
    </div>
  );
};
```
このように、複雑なモデルも一つのコンポーネントとして簡単に扱うことができます。 

