---
title: "WebSocketとは？"
emoji: "📡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "websocket", "socketio", "realtime"]
published: true
---

## はじめに

リアルタイムチャット、オンラインゲーム、共同編集ドキュメント、株価のライブ配信… このような「リアルタイム」な機能を持つWebアプリケーションを作ってみたいと思ったことはありませんか？

従来のWeb技術であるHTTP通信は、基本的に「クライアントが要求し、サーバーが応答する」という一方向のやり取りが基本でした。そのため、サーバー側で起きた変化を即座にクライアントに伝えるのが苦手です。

この課題を解決し、Web上でインタラクティブな体験を実現するための重要な技術が **WebSocket** です。この記事では、WebSocketがどのような技術で、なぜ必要なのか、そしてNext.jsアプリケーションでどうやって利用するのか、その基本を分かりやすく解説します。

## WebSocket とは？

WebSocketとは、一言でいうと **「Webブラウザ（クライアント）とサーバーの間で、双方向のリアルタイム通信を可能にするための技術仕様（プロトコル）」** です。

電話に例えると非常に分かりやすいです。

*   **従来のHTTP通信**: 「手紙のやり取り」
    *   用件があるたびに、クライアントが手紙（リクエスト）を書き、サーバーに送ります。
    *   サーバーはその手紙を読んで、返事（レスポンス）を書いて送り返します。
    *   サーバー側から自発的に手紙を送ることはできません。常にクライアントからの手紙を待つ必要があります。

*   **WebSocket**: 「電話回線」
    *   最初にクライアントがサーバーに電話をかけ、サーバーが応答すると、接続（電話回線）が確立されます。
    *   一度接続が確立されれば、その回線は開いたままになり、クライアントとサーバーのどちらからでも、いつでも好きなタイミングで会話（データを送受信）できます。

このように、WebSocketは一度接続を確立すると、その通信路（コネクション）を維持し続けます。これにより、HTTPのように毎回接続を確立し直す手間がなく、低遅延で効率的な双方向通信が実現できるのです。

### WebSocketの主なメリット

1.  **リアルタイム性**: サーバー側でイベントが発生した際、即座に全クライアントへデータを「プッシュ」できます。チャットの新着メッセージや、スポーツの試合速報などに最適です。
2.  **双方向通信**: サーバーとクライアントが対等な立場で、いつでもデータを送り合えます。
3.  **効率性**: 通信のたびに送信されるヘッダー情報がHTTPに比べて非常に小さく、通信のオーバーヘッド（無駄）が少ないため、ネットワークやサーバーへの負荷を軽減できます。

## Next.js で WebSocket を使用するには？

Next.js自体は、HTTPサーバーとしての機能が中心であり、WebSocketサーバーの機能を標準では内蔵していません。そのため、Next.jsアプリケーションでWebSocketを利用するには、少し工夫が必要です。

最も一般的で安定した方法は、**Next.jsのWebサーバーとは別に、専用のWebSocketサーバーを立てて連携させる**ことです。

ここでは、人気のライブラリである `Socket.IO` を使った基本的な実装手順の概要を見ていきましょう。`Socket.IO`は、WebSocketをより簡単に、そして安定して利用できるようにしたライブラリで、サーバーサイドとクライアントサイドの両方で利用できます。

### ステップ1: 必要なパッケージのインストール

WebSocketサーバー用に`socket.io`、クライアント用に`socket.io-client`をインストールします。

```bash
# サーバー用とクライアント用を両方インストール
npm install socket.io socket.io-client
```

### ステップ2: WebSocketサーバーの作成

Next.jsのプロジェクトとは別に、WebSocket通信を専門に扱うためのシンプルなNode.jsサーバーを立てます。（例: `websocket-server/index.js`）

```javascript:websocket-server/index.js
// ExpressなどのHTTPサーバーと連携させるのが一般的
const { createServer } = require("http");
const { Server } = require("socket.io");

const httpServer = createServer();
const io = new Server(httpServer, {
  cors: {
    origin: "http://localhost:3000", // Next.jsアプリのURL
    methods: ["GET", "POST"]
  }
});

io.on("connection", (socket) => {
  console.log("A user connected:", socket.id);

  // クライアントから'message'というイベント名でデータが送られてきた時の処理
  socket.on("message", (data) => {
    console.log("Message received:", data);
    // 全てのクライアントに'message'イベントをブロードキャストする
    io.emit("message", data);
  });

  socket.on("disconnect", () => {
    console.log("User disconnected:", socket.id);
  });
});

const PORT = 3001; // Next.jsアプリとは別のポートで起動
httpServer.listen(PORT, () => {
  console.log(`WebSocket server listening on port ${PORT}`);
});
```
このサーバーを `node websocket-server/index.js` のようにして起動しておきます。

### ステップ3: Next.jsのクライアントコンポーネントで接続する

Next.jsアプリケーション側（クライアントコンポーネント）で、先ほど立ち上げたWebSocketサーバーに接続し、データの送受信を行います。

```tsx:app/components/Chat.tsx
"use client"

import { useEffect, useState } from "react"
import io, { Socket } from "socket.io-client"

// サーバーに接続するためのSocketインスタンスを一度だけ作成
const socket: Socket = io("http://localhost:3001");

export default function Chat() {
  const [message, setMessage] = useState("");
  const [chat, setChat] = useState<string[]>([]);

  useEffect(() => {
    // サーバーから'message'イベントでデータを受信した時の処理
    socket.on("message", (data) => {
      setChat((prevChat) => [...prevChat, data]);
    });

    // コンポーネントがアンマウントされた時にリスナーをクリーンアップ
    return () => {
      socket.off("message");
    };
  }, []);

  const sendMessage = (e: React.FormEvent) => {
    e.preventDefault();
    if (message) {
      // サーバーに'message'イベントでデータを送信
      socket.emit("message", message);
      setMessage("");
    }
  };

  return (
    <div>
      <ul>
        {chat.map((msg, index) => (
          <li key={index}>{msg}</li>
        ))}
      </ul>
      <form onSubmit={sendMessage}>
        <input
          type="text"
          value={message}
          onChange={(e) => setMessage(e.target.value)}
        />
        <button type="submit">送信</button>
      </form>
    </div>
  );
}
```

### 注意点: サーバーレス環境での利用
VercelのようなサーバーレスプラットフォームにNext.jsアプリケーションをデプロイする場合、長時間接続を維持するWebSocketサーバーを動かすことは難しい、あるいは追加の構成が必要になります。
その場合は、**Pusher**や**Ably**といった、リアルタイム通信機能を提供する外部のSaaSを利用するのが現実的で簡単な解決策となります。

## おわりに

この記事では、WebSocketの基本的な概念と、Next.jsアプリケーションで利用するための一般的なアプローチについて解説しました。

WebSocketは、HTTPだけでは実現が難しかったリアルタイムでインタラクティブなWeb体験を可能にする強力な技術です。Next.jsで利用するには少し準備が必要ですが、`Socket.IO`のようなライブラリを使えば、その実装は決して難しいものではありません。

リアルタイムチャット、通知機能、ライブダッシュボードなど、WebSocketを使えばあなたのアプリケーションの可能性は大きく広がります。ぜひこの技術を活用して、よりダイナミックなWebアプリケーション開発に挑戦してみてください！