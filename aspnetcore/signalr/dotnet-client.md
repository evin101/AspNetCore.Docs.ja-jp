---
title: ASP.NET Core SignalR .NET クライアント
author: bradygaster
description: ASP.NET Core SignalR .NET クライアントに関する情報
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 09/13/2019
uid: signalr/dotnet-client
ms.openlocfilehash: 4419799ef11469413f813843a9d02ac0223d30c6
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081290"
---
# <a name="aspnet-core-signalr-net-client"></a>ASP.NET Core SignalR .NET クライアント

ASP.NET Core SignalR .NET クライアントライブラリを使用すると、.NET アプリから SignalR hub と通信できます。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/dotnet-client/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

この記事のコードサンプルは、ASP.NET Core SignalR .NET クライアントを使用する WPF アプリです。

## <a name="install-the-signalr-net-client-package"></a>SignalR .NET クライアントパッケージをインストールする

.NET クライアントが SignalR hub に接続するには、 [Microsoft.AspNetCore.SignalR.Client](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client)パッケージが必要です。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

クライアントライブラリをインストールするには、**パッケージマネージャーコンソール**ウィンドウで次のコマンドを実行します。

```powershell
Install-Package Microsoft.AspNetCore.SignalR.Client
```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

クライアントライブラリをインストールするには、コマンドシェルで次のコマンドを実行します。

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

---

## <a name="connect-to-a-hub"></a>ハブへの接続します。

接続を確立するには`HubConnectionBuilder` 、を作成し、を呼び出し`Build`ます。 接続の構築中に、ハブの URL、プロトコル、トランスポートの種類、ログレベル、ヘッダー、およびその他のオプションを構成できます。 任意の`HubConnectionBuilder`メソッドをに`Build`挿入して、必要なオプションを構成します。 との`StartAsync`接続を開始します。

[!code-csharp[Build hub connection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_MainWindowClass&highlight=15-17,39)]

## <a name="handle-lost-connection"></a>失われた接続の処理

::: moniker range=">= aspnetcore-3.0"

### <a name="automatically-reconnect"></a>自動的に再接続する

は、の`WithAutomaticReconnect`メソッド<xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilder>を使用して自動的に再接続するように構成できます。<xref:Microsoft.AspNetCore.SignalR.Client.HubConnection> 既定では、自動的に再接続されません。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect()
    .Build();
```

パラメーターを指定し`WithAutomaticReconnect()`ない場合、は、再接続を試行する前に、それぞれ0、2、10、および30秒間待機するようにクライアントを構成します。これにより、4回の試行が失敗すると停止します。

再接続の試行を開始する`HubConnection`前に、は`HubConnectionState.Reconnecting`状態に遷移し`Reconnecting` 、イベントを発生させます。  これにより、接続が失われたことをユーザーに警告し、UI 要素を無効にすることができます。 非対話型アプリでは、メッセージのキューまたは削除を開始できます。

```csharp
connection.Reconnecting += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Reconnecting);

    // Notify users the connection was lost and the client is reconnecting.
    // Start queuing or dropping messages.

    return Task.CompletedTask;
};
```

最初の4回の試行でクライアントが正常に再`HubConnection`接続した場合、 `Connected`は状態に戻り`Reconnected` 、イベントを発生させます。 これにより、接続が再確立されたことをユーザーに通知し、キューに置かれたすべてのメッセージをデキューすることができます。

接続はサーバーにまったく新しいものであるため、 `ConnectionId` `Reconnected`イベントハンドラーに新しいが提供されます。

> [!WARNING]
> が`Reconnected` [ネゴシエーションをスキップ](xref:signalr/configuration#configure-client-options)するように構成され`HubConnection`ている場合、イベントハンドラーの`connectionId`パラメーターは null になります。

```csharp
connection.Reconnected += connectionId =>
{
    Debug.Assert(connection.State == HubConnectionState.Connected);

    // Notify users the connection was reestablished.
    // Start dequeuing messages queued while reconnecting if any.

    return Task.CompletedTask;
};
```

`WithAutomaticReconnect()`最初の開始`HubConnection`エラーを再試行するようにを構成しません。そのため、開始エラーは手動で処理する必要があります。

```csharp
public static async Task<bool> ConnectWithRetryAsync(HubConnection connection, CancellationToken token)
{
    // Keep trying to until we can start or the token is canceled.
    while (true)
    {
        try
        {
            await connection.StartAsync(token);
            Debug.Assert(connection.State == HubConnectionState.Connected);
            return true;
        }
        catch when (token.IsCancellationRequested)
        {
            return false;
        }
        catch
        {
            // Failed to connect, trying again in 5000 ms.
            Debug.Assert(connection.State == HubConnectionState.Disconnected);
            await Task.Delay(5000);
        }
    }
}
```

最初の4回の試行でクライアントが正常に再接続`HubConnection`されない場合`Disconnected` 、は状態に<xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed>遷移し、イベントを発生させます。 これにより、接続を手動で再起動したり、接続が完全に失われたことをユーザーに通知したりすることができます。

```csharp
connection.Closed += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Disconnected);

    // Notify users the connection has been closed or manually try to restart the connection.

    return Task.CompletedTask;
};
```

再接続のタイミングを切断または変更する前に、カスタムの再接続試行`WithAutomaticReconnect`回数を構成するために、では、各再接続の試行を開始するまでの待機時間 (ミリ秒) を表す数値の配列を受け取ります。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.Zero, TimeSpan.FromSeconds(10) })
    .Build();

    // .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(30) }) yields the default behavior.
```

前の例では`HubConnection` 、接続が失われた直後に再接続を開始するようにを構成しています。 これは、既定の構成にも当てはまります。

最初の再接続の試行が失敗した場合、2回目の再接続試行も、既定の構成のように2秒間待機するのではなく、直ちに開始されます。

2回目の再接続の試行が失敗した場合、3回目の再接続は10秒後に開始され、既定の構成のようになります。

その後、3回目の再接続の試行が失敗した後に停止することで、カスタム動作が既定の動作から再び逸脱します。 既定の構成では、もう1回は30秒後に再接続が試行されます。

タイミングと自動再接続試行回数をさらに細かく制御する場合は、 `WithAutomaticReconnect` `IRetryPolicy`インターフェイスを実装するオブジェクトを受け取ります。このインターフェイスには`NextRetryDelay`、という名前の1つのメソッドがあります。

`NextRetryDelay`型`RetryContext`の1つの引数を受け取ります。 `RetryReason` `ElapsedTime` `long` `PreviousRetryCount` `TimeSpan` には`Exception` 、、、およびの3つのプロパティがあります。`RetryContext` 最初の再接続を試行する`PreviousRetryCount`前`ElapsedTime`に、との`RetryReason`両方がゼロになり、は接続が失われる原因となった例外になります。 再試行が再試行されるたび`PreviousRetryCount`に、が`ElapsedTime` 1 ずつインクリメントされ、これまでに再接続に費やされた時間が反映さ`RetryReason`れます。また、は、最後の再接続の試行が失敗した原因となった例外になります。

`NextRetryDelay`次の再接続が試行される前に待機する時間を表す TimeSpan `null`を返す`HubConnection`か、またはの再接続を停止する必要があります。

```csharp
public class RandomRetryPolicy : IRetryPolicy
{
    private readonly Random _random = new Random();

    public TimeSpan? NextRetryDelay(RetryContext retryContext)
    {
        // If we've been reconnecting for less than 60 seconds so far,
        // wait between 0 and 10 seconds before the next reconnect attempt.
        if (retryContext.ElapsedTime < TimeSpan.FromSeconds(60))
        {
            return TimeSpan.FromSeconds(_random.Next() * 10);
        }
        else
        {
            // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
            return null;
        }
    }
}
```

```csharp
HubConnection connection = new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chatHub"))
    .WithAutomaticReconnect(new RandomRetryPolicy())
    .Build();
```

または、[手動で再接続](#manually-reconnect)する方法で示すように、手動でクライアントを再接続するコードを記述することもできます。

::: moniker-end

### <a name="manually-reconnect"></a>手動で再接続する

::: moniker range="< aspnetcore-3.0"

> [!WARNING]
> 3\.0 より前では、SignalR 用の .NET クライアントは自動的に再接続しません。 クライアントを手動で再接続するコードを記述する必要があります。

::: moniker-end

失われ<xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed>た接続に応答するには、イベントを使用します。 たとえば、再接続を自動化することができます。

イベント`Closed`には`Task`、を返すデリゲートが必要です。これにより、を使用`async void`せずに非同期コードを実行できます。 同期的に実行される`Closed`イベントハンドラーでデリゲートシグネチャを満たすには、次を返し`Task.CompletedTask`ます。

```csharp
connection.Closed += (error) => {
    // Do your close logic.
    return Task.CompletedTask;
};
```

非同期サポートの主な理由は、接続を再起動できるようにするためです。 接続の開始は、非同期アクションです。

接続を再起動するハンドラーで、次の例に示すように、サーバーの過負荷を防ぐために、ランダムな遅延を待機することを検討してください。`Closed`

[!code-csharp[Use Closed event handler to automate reconnection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ClosedRestart)]

## <a name="call-hub-methods-from-client"></a>クライアントからのハブ メソッドの呼び出し

`InvokeAsync`ハブでメソッドを呼び出します。 ハブメソッドの名前と、ハブメソッドで定義されている`InvokeAsync`すべての引数をに渡します。 SignalR は非同期であるため`async` 、 `await`呼び出しを行うときにとを使用します。

[!code-csharp[InvokeAsync method](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_InvokeAsync)]

この`InvokeAsync`メソッドは、 `Task`サーバーメソッドから制御が戻ったときに完了するを返します。 戻り値 (存在する場合) は、 `Task`の結果として提供されます。 サーバー上のメソッドによってスローされた例外`Task`が発生すると、エラーが発生します。 構文`await`を使用して、サーバーメソッドが`try...catch`完了するのを待機し、構文を使用してエラーを処理します。

メソッド`SendAsync`は、メッセージ`Task`がサーバーに送信されたときに完了するを返します。 サーバーメソッドが完了するまで待機`Task`しないため、戻り値は提供されません。 メッセージの送信中にクライアントでスローされた例外は`Task`、エラーを生成します。 および`await`構文`try...catch`を使用して、送信エラーを処理します。

> [!NOTE]
> *サーバーレスモード*で Azure SignalR サービスを使用している場合は、クライアントからハブメソッドを呼び出すことはできません。 詳細については、 [SignalR サービスのドキュメント](/azure/azure-signalr/signalr-concept-serverless-development-config)を参照してください。

## <a name="call-client-methods-from-hub"></a>ハブからのクライアント メソッドを呼び出す

ハブがビルド後、接続`connection.On`を開始する前にを使用して呼び出すメソッドを定義します。

[!code-csharp[Define client methods](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ConnectionOn)]

の`connection.On`前のコードは、サーバー側のコードが`SendAsync`メソッドを使用して呼び出したときに実行されます。

[!code-csharp[Call client method](dotnet-client/sample/signalrchat/hubs/chathub.cs?name=snippet_SendMessage)]

## <a name="error-handling-and-logging"></a>エラー処理とログ記録

Try-catch ステートメントを使用してエラーを処理します。 `Exception`オブジェクトを調べて、エラーが発生した後に実行する適切なアクションを決定します。

[!code-csharp[Logging](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ErrorHandling)]

## <a name="additional-resources"></a>その他の技術情報

* [ハブ](xref:signalr/hubs)
* [JavaScript クライアント](xref:signalr/javascript-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [Azure SignalR Service サーバーレスドキュメント](/azure/azure-signalr/signalr-concept-serverless-development-config)
