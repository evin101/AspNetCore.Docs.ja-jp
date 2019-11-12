---
title: ASP.NET Core での WebSocket のサポート
author: rick-anderson
description: ASP.NET Core で WebSocket の使用を開始する方法を説明します。
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/10/2019
uid: fundamentals/websockets
ms.openlocfilehash: 098e6826d6f7114baceb9578dc6d9883eb83f0aa
ms.sourcegitcommit: a166291c6708f5949c417874108332856b53b6a9
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/18/2019
ms.locfileid: "72589703"
---
# <a name="websockets-support-in-aspnet-core"></a>ASP.NET Core での WebSocket のサポート

作成者: [Tom Dykstra](https://github.com/tdykstra) および [Andrew Stanton-Nurse](https://github.com/anurse)

この記事では、ASP.NET Core で WebSocket の使用を開始する方法について説明します。 [WebSocket](https://wikipedia.org/wiki/WebSocket) ([RFC 6455](https://tools.ietf.org/html/rfc6455)) は、TCP 接続を使用した双方向の永続的通信チャネルを有効にするプロトコルです。 このプロトコルは、チャット、ダッシュボード、ゲーム アプリなど、高速かつリアルタイムのコミュニケーションを活用するアプリで使用されます。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/websockets/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。 [実行方法](#sample-app)。

## <a name="signalr"></a>SignalR

[ASP.NET Core SignalR](xref:signalr/introduction) は、アプリへのリアルタイム Web 機能の追加を簡単にするライブラリです。 可能なかぎり、WebSocket が使用されます。

ほとんどのアプリケーションでは、生の WebSocket よりも SignalR が推奨されます。 SignalR には、WebSocket を使用できない環境の場合にトランスポートのフォールバックが用意されています。 シンプルなリモート プロシージャ呼び出しアプリ モデルも用意されています。 また、ほとんどのシナリオで、SignalR には生の WebSocket を使用した場合と比較してパフォーマンス上の大きなデメリットがありません。

## <a name="prerequisites"></a>必須コンポーネント

* ASP.NET Core 1.1 以降
* ASP.NET Core をサポートする任意の OS:
  
  * Windows 7 / Windows Server 2008 以降
  * Linux
  * macOS
  
* アプリが IIS を含む Windows 上で実行されている場合:

  * Windows 8 / Windows Server 2012 以降
  * IIS 8 / IIS 8 Express
  * WebSockets を有効にする必要があります (「[IIS/IIS Express のサポート](#iisiis-express-support)」セクションを参照してください)。
  
* アプリが [HTTP.sys](xref:fundamentals/servers/httpsys) で実行されている場合:

  * Windows 8 / Windows Server 2012 以降

* サポートされているブラウザーについては、 https://caniuse.com/#feat=websockets を参照してください。

::: moniker range="< aspnetcore-2.1"

## <a name="nuget-package"></a>NuGet パッケージ

[Microsoft.AspNetCore.WebSockets](https://www.nuget.org/packages/Microsoft.AspNetCore.WebSockets/) パッケージをインストールします。

::: moniker-end

## <a name="configure-the-middleware"></a>ミドルウェアの構成


`Startup` クラスの `Configure` メソッドに、WebSocket ミドルウェアを追加します。

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSockets)]

::: moniker range="< aspnetcore-2.2"

次の設定を構成できます。

* `KeepAliveInterval`: プロキシの接続の維持を保証する、クライアントに "ping" フレームを送信する頻度。 既定値は 2 分です。
* `ReceiveBufferSize`: データの受信に使用されるバッファーのサイズ。 これは、上級ユーザーが、データのサイズに応じたパフォーマンス調整のために変更する必要がある場合があります。 既定値は 4 KB です。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

次の設定を構成できます。

* `KeepAliveInterval`: プロキシの接続の維持を保証する、クライアントに "ping" フレームを送信する頻度。 既定値は 2 分です。
* <xref:Microsoft.AspNetCore.Builder.WebSocketOptions.ReceiveBufferSize>: データの受信に使用されるバッファーのサイズ。 これは、上級ユーザーが、データのサイズに応じたパフォーマンス調整のために変更する必要がある場合があります。 既定値は 4 KB です。
* `AllowedOrigins` - WebSocket 要求で許可される配信元ヘッダー値の一覧。 既定では、すべての配信元が許可されています。 詳細については、下記の "WebSocket の配信元の制限" を参照してください。

::: moniker-end

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptions)]

## <a name="accept-websocket-requests"></a>WebSocket の要求の受け入れ

以降の要求ライフサイクルのどこかで (たとえば、以降の `Configure` メソッドまたはアクション メソッド)、それが WebSocket 要求であるかを確認し、WebSocket 要求を受け入れます。

次の例は、以降の `Configure` メソッドから抜粋したものです。

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=AcceptWebSocket&highlight=7)]

WebSocket 要求はどの URL からも受け取る場合がありますが、このサンプル コードでは `/ws` の要求のみを受け取ります。

WebSocket を使用するとき、接続中、ミドルウェア パイプラインの実行を維持する**必要があります**。 ミドルウェア パイプラインの修了後に WebSocket メッセージを送信するか、受信する場合、次のような例外を受け取ることがあります。

```
System.Net.WebSockets.WebSocketException (0x80004005): The remote party closed the WebSocket connection without completing the close handshake. ---> System.ObjectDisposedException: Cannot write to the response body, the response has completed.
Object name: 'HttpResponseStream'.
```

バックグラウンド サービスを利用してデータを WebSocket に書き込む場合、ミドルウェア パイプラインの実行を維持します。 これは <xref:System.Threading.Tasks.TaskCompletionSource%601> を使用して行います。 `TaskCompletionSource` をバックグラウンド サービスに渡し、WebSocket が終わったとき、それに <xref:System.Threading.Tasks.TaskCompletionSource%601.TrySetResult%2A> を呼び出させます。 次の例に示すように、要求中に <xref:System.Threading.Tasks.TaskCompletionSource%601.Task> プロパティの `await` を行います。

```csharp
app.Use(async (context, next) => {
    var socket = await context.WebSockets.AcceptWebSocketAsync();
    var socketFinishedTcs = new TaskCompletionSource<object>();

    BackgroundSocketProcessor.AddSocket(socket, socketFinishedTcs); 

    await socketFinishedTcs.Task;
});
```
WebSocket の終了例外は、アクション メソッドから早く戻りすぎた場合にも発生する可能性があります。 アクション メソッドでソケットを受け入れる場合は、そのソケットを使用するコードが完了するまで待ち、アクション メソッドから戻ってください。

重大なスレッドの問題を引き起こす可能性があるので、ソケットの完了を待つために `Task.Wait()`、`Task.Result`、または同様のブロック呼び出しを使用しないでください。 常に `await` を使用します。

## <a name="send-and-receive-messages"></a>メッセージの送受信

`AcceptWebSocketAsync` メソッドは、TCP 接続を WebSocket 接続にアップグレードし、[WebSocket](/dotnet/core/api/system.net.websockets.websocket) オブジェクトを提供します。 メッセージの送受信に、`WebSocket` オブジェクトを使用します。

前に示した、WebSocket 要求を受け入れるコードが、`WebSocket` オブジェクトを `Echo` メソッドに渡します。 このコードは、メッセージを受信し、同じメッセージをすぐに送信します。 クライアントが接続を閉じるまで、メッセージがループで送受信されます。

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=Echo)]

このループを開始する前に、WebSocket 接続を受け入れた場合、ミドルウェア パイプラインは終了します。 ソケットを閉じると、パイプラインはアンワインドされます。 つまり、WebSocket を受け入れると、要求はパイプラインでの先への移動を中止します。 ループを終了し、ソケットを閉じた場合、要求はパイプラインのバックアップを続けます。

::: moniker range=">= aspnetcore-2.2"

## <a name="handle-client-disconnects"></a>クライアントの切断の処理

接続の損失によってクライアントが切断されても、サーバーに自動的に通知されるわけではありません。 サーバーが切断メッセージを受信するのは、クライアントがそれを送信した場合のみです。インターネット接続が失われた場合、これを実行することはできません。 これが発生した場合に何らかのアクションを実行したい場合は、特定の時間枠内でクライアントからの受信を待つタイムアウトを設定します。

クライアントが常にメッセージを送信するとは限らず、その接続がアイドル状態になっただけでタイムアウトしたくない場合は、X 秒ごとに ping メッセージを送信するタイマーをクライアントに使用させます。 サーバー上では、前のものから 2\*X 秒以内にメッセージが到着しなかった場合に、接続を終了してクライアントが切断されたことをレポートします。 予想される 2 倍の期間を待機することで、ping メッセージを遅らせる可能性のあるネットワークの遅延のために余分な時間を残します。

## <a name="websocket-origin-restriction"></a>WebSocket の配信元の制限

CORS で提供される保護は、WebSocket には適用されません。 ブラウザーでは以下を実行**しません**。

* CORS の事前要求を実行する。
* WebSocket 要求を行うときに `Access-Control` ヘッダーに指定された制限を考慮する。

ただし、WebSocket 要求を発行するときにはブラウザーから `Origin` ヘッダーが送信されます。 予期した配信元からの WebSocket のみが許可されるように、アプリケーションでこれらのヘッダーが検証されるように構成する必要があります。

"https://server.com" でサーバーを、"https://client.com" でクライアントをホスティングしている場合は、検証のために "https://client.com" を WebSocket の `AllowedOrigins` 一覧に追加します。

[!code-csharp[](websockets/samples/2.x/WebSocketsSample/Startup.cs?name=UseWebSocketsOptionsAO&highlight=6-7)]

> [!NOTE]
> `Origin` ヘッダーは、クライアントによって制御され、`Referer` のように偽装することができます。 これらのヘッダーを認証メカニズムとして使用**しないでください**。

::: moniker-end

## <a name="iisiis-express-support"></a>IIS/IIS Express のサポート

IIS/IIS Express 8 以降を含む、Windows Server 2012 以降および Windows 8 以降では、WebSocket プロトコルをサポートします。

> [!NOTE]
> IIS Express を使用する場合、WebSockets は常に有効になります。

### <a name="enabling-websockets-on-iis"></a>IIS での WebSockets の有効化

Windows Server 2012 以降で WebSocket プロトコルのサポートを有効にするには

> [!NOTE]
> これらの手順は、IIS Express を使用する場合は必要ありません

1. **[管理]** メニューから**役割と機能の追加**ウィザードを使用するか、**サーバー マネージャー**にあるリンクを使用します。
1. **[役割ベースまたは機能ベースのインストール]** を選択します。 **[次へ]** を選択します。
1. 適切なサーバーを選択します (既定では、ローカル サーバーが選択されます)。 **[次へ]** を選択します。
1. **[役割]** ツリーで **[Web サーバー (IIS)]** を展開し、 **[Web サーバー]** 、 **[アプリケーション開発]** の順に展開します。
1. **[WebSocket プロトコル]** を選択します。 **[次へ]** を選択します。
1. 追加機能が不要な場合は、 **[次へ]** を選択します。
1. **[インストール]** を選択します。
1. インストールが完了したら、 **[閉じる]** を選択してウィザードを終了します。

Windows 8 以降で WebSocket プロトコルのサポートを有効にするには

> [!NOTE]
> これらの手順は、IIS Express を使用する場合は必要ありません

1. **[コントロール パネル]**  >  **[プログラム]**  >  **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。
1. 次のノード: **[インターネット インフォメーション サービス]**  >  **[World Wide Web サービス]**  >  **[アプリケーション開発機能]** を開きます。
1. **[WebSocket プロトコル]** 機能を選択します。 **[OK]** を選択します。

### <a name="disable-websocket-when-using-socketio-on-nodejs"></a>Node.js で socket.io を使用する場合に WebSocket を無効にする

[Node.js](https://nodejs.org/) の [socket.io](https://socket.io/) で WebSocket サポートを使用する場合、`webSocket` 要素を使用する既定の WebSocket モジュールを *web.config* または *applicationHost.config* で無効にします。この手順が実行されない場合、IIS WebSocket モジュールは Node.js とアプリ以外の WebSocket コミュニケーションを処理しようとします。

```xml
<system.webServer>
  <webSocket enabled="false" />
</system.webServer>
```

## <a name="sample-app"></a>サンプル アプリ

この記事に添えられている[サンプル アプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/websockets/samples)は、エコー アプリです。 これには、WebSocket 接続を作成する Web ページがあり、サーバーが受け取るすべてのメッセージをクライアントに再送信します。 コマンド プロンプトからアプリを実行し (IIS Express を使用した Visual Studio からは実行するように設定されていません)、 http://localhost:5000 に移動します。 Web ページの左上に、接続の状態が示されます。

![Web ページの初期状態](websockets/_static/start.png)

**[接続]** を選択し、表示されている URL に WebSocket 要求を送信します。 テスト メッセージを入力し、 **[送信]** を選択します。 完了したら、 **[Close Socket]\(ソケットを閉じる\)** を選択します。 **[Communication Log]\(コミュニケーション ログ\)** セクションに、発生した各オープン、送信、クローズのアクションが表示されます。

![Web ページの初期状態](websockets/_static/end.png)

