---
title: ASP.NET Core SignalR でストリーミングを使用する
author: bradygaster
description: クライアントとサーバーの間でデータをストリーミングする方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 06/05/2019
uid: signalr/streaming
ms.openlocfilehash: d520c8eec3e777acb9604bdcb5969268deabf8da
ms.sourcegitcommit: d34b2627a69bc8940b76a949de830335db9701d3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/23/2019
ms.locfileid: "71186929"
---
# <a name="use-streaming-in-aspnet-core-signalr"></a>ASP.NET Core SignalR でストリーミングを使用する

作成者: [Brennan Conroy](https://github.com/BrennanConroy)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core SignalR は、クライアントからサーバー、およびサーバーからクライアントへのストリーミングをサポートしています。 これは、時間の経過と共にデータのフラグメントが到着するシナリオに役立ちます。 ストリーミング時には、すべてのデータが使用可能になるのを待機するのではなく、使用可能になるとすぐに各フラグメントがクライアントまたはサーバーに送信されます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core SignalR は、サーバーメソッドのストリーミング戻り値をサポートしています。 これは、時間の経過と共にデータのフラグメントが到着するシナリオに役立ちます。 戻り値がクライアントにストリーミングされると、すべてのデータが使用可能になるまで待機するのではなく、使用可能になるとすぐに各フラグメントがクライアントに送信されます。

::: moniker-end

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/live/aspnetcore/signalr/streaming/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="set-up-a-hub-for-streaming"></a>ストリーミング用のハブを設定する

::: moniker range=">= aspnetcore-3.0"

ハブ<xref:System.Collections.Generic.IAsyncEnumerable`1>メソッドは<xref:System.Threading.Channels.ChannelReader%601> `Task<ChannelReader<T>>`、、、、またはを返すと、自動的にストリーミングハブメソッドになります。 `Task<IAsyncEnumerable<T>>`

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ハブメソッドは、 <xref:System.Threading.Channels.ChannelReader%601> `Task<ChannelReader<T>>`またはを返すと、自動的にストリーミングハブメソッドになります。

::: moniker-end

### <a name="server-to-client-streaming"></a>サーバーからクライアントへのストリーミング

::: moniker range=">= aspnetcore-3.0"

ストリーミングハブのメソッドは`IAsyncEnumerable<T>` 、に加え`ChannelReader<T>`てを返すことができます。 を返す`IAsyncEnumerable<T>`最も簡単な方法は、次の例に示すように、ハブメソッドを非同期反復子メソッドにすることです。 ハブの非同期反復子メソッドは`CancellationToken` 、クライアントがストリームからアンサブスクライブしたときにトリガーされるパラメーターを受け取ることができます。 非同期反復子メソッドは、チャネルに共通する問題を回避し`ChannelReader` <xref:System.Threading.Channels.ChannelWriter`1>ます。たとえば、を完了しないうちに十分な初期状態を返さない場合や、メソッドを終了する場合などです。

[!INCLUDE[](~/includes/csharp-8-required.md)]

[!code-csharp[Streaming hub async iterator method](streaming/samples/3.0/Hubs/AsyncEnumerableHub.cs?name=snippet_AsyncIterator)]

::: moniker-end

次のサンプルは、チャネルを使用してクライアントにデータをストリーミングする方法の基本を示しています。 オブジェクトがに<xref:System.Threading.Channels.ChannelWriter%601>書き込まれるたびに、オブジェクトは直ちにクライアントに送信されます。 最後に、は、 `ChannelWriter`ストリームが閉じられていることをクライアントに通知するために完了します。

> [!NOTE]
> バックグラウンドスレッド`ChannelReader`でに書き込み、できるだけ早くを返します。 `ChannelWriter<T>` が返されるまで、 `ChannelReader`他のハブ呼び出しはブロックされます。
>
> でロジックをラップ`try ... catch`します。 ハブメソッドの呼び出し`catch`が正しく完了`catch`していることを確認するには、のとの外側にあるを完了します。`Channel`

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Streaming hub method](streaming/samples/3.0/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[Streaming hub method](streaming/samples/2.2/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[Streaming hub method](streaming/samples/2.1/Hubs/StreamHub.cs?name=snippet1)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

サーバーからクライアントへのストリーミングハブメソッドは、クライアント`CancellationToken`がストリームからアンサブスクライブしたときにトリガーされるパラメーターを受け取ることができます。 ストリームの終了前にクライアントが切断された場合は、このトークンを使用してサーバー操作を停止し、リソースを解放します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>クライアントとサーバー間のストリーミング

ハブメソッドは、型または<xref:System.Threading.Channels.ChannelReader%601> <xref:System.Collections.Generic.IAsyncEnumerable%601>型の1つ以上のオブジェクトを受け入れると、クライアントとサーバー間のストリーミングハブメソッドに自動的になります。 次のサンプルは、クライアントから送信されたストリーミングデータの読み取りの基本を示しています。 クライアントがに<xref:System.Threading.Channels.ChannelWriter%601>書き込むたびに、データはハブメソッドの読み取り`ChannelReader`元のサーバー上のに書き込まれます。

[!code-csharp[Streaming upload hub method](streaming/samples/3.0/Hubs/StreamHub.cs?name=snippet2)]

メソッド<xref:System.Collections.Generic.IAsyncEnumerable%601>のバージョンは次のとおりです。

[!INCLUDE[](~/includes/csharp-8-required.md)]

```csharp
public async Task UploadStream(IAsyncEnumerable<string> stream)
{
    await foreach (var item in stream)
    {
        Console.WriteLine(item);
    }
}
```

::: moniker-end

## <a name="net-client"></a>.NET クライアント

### <a name="server-to-client-streaming"></a>サーバーからクライアントへのストリーミング


::: moniker range=">= aspnetcore-3.0"

の`StreamAsync` `StreamAsChannelAsync`メソッドとメソッドは、サーバーからクライアントへのストリーミングメソッドを呼び出すために使用されます。 `HubConnection` ハブメソッドで定義されているハブメソッド名と引数`StreamAsync`を`StreamAsChannelAsync`またはに渡します。 `StreamAsync<T>` および`StreamAsChannelAsync<T>`のジェネリックパラメーターは、ストリーミングメソッドによって返されるオブジェクトの型を指定します。 または`IAsyncEnumerable<T>` `ChannelReader<T>`型のオブジェクトがストリーム呼び出しから返され、クライアントのストリームを表します。

を`StreamAsync` 返す`IAsyncEnumerable<int>`例を次に示します。

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var stream = await hubConnection.StreamAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

await foreach (var count in stream)
{
    Console.WriteLine($"{count}");
}

Console.WriteLine("Streaming completed");
```

を返す`StreamAsChannelAsync` `ChannelReader<int>`対応する例を次に示します。

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var channel = await hubConnection.StreamAsChannelAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

`StreamAsChannelAsync` の`HubConnection`メソッドは、サーバーからクライアントへのストリーミングメソッドを呼び出すために使用されます。 ハブメソッドで定義されているハブメソッド名と引数`StreamAsChannelAsync`をに渡します。 の`StreamAsChannelAsync<T>`ジェネリックパラメーターは、ストリーミングメソッドによって返されるオブジェクトの型を指定します。 `ChannelReader<T>`はストリーム呼び出しから返され、クライアントのストリームを表します。

```csharp
// Call "Cancel" on this CancellationTokenSource to send a cancellation message to
// the server, which will trigger the corresponding token in the hub method.
var cancellationTokenSource = new CancellationTokenSource();
var channel = await hubConnection.StreamAsChannelAsync<int>(
    "Counter", 10, 500, cancellationTokenSource.Token);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range="= aspnetcore-2.1"

`StreamAsChannelAsync` の`HubConnection`メソッドは、サーバーからクライアントへのストリーミングメソッドを呼び出すために使用されます。 ハブメソッドで定義されているハブメソッド名と引数`StreamAsChannelAsync`をに渡します。 の`StreamAsChannelAsync<T>`ジェネリックパラメーターは、ストリーミングメソッドによって返されるオブジェクトの型を指定します。 `ChannelReader<T>`はストリーム呼び出しから返され、クライアントのストリームを表します。

```csharp
var channel = await hubConnection
    .StreamAsChannelAsync<int>("Counter", 10, 500, CancellationToken.None);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>クライアントとサーバー間のストリーミング

.NET クライアントからクライアントとサーバー間のストリーミングハブメソッドを呼び出すには、2つの方法があります。 呼び出されたハブメソッドに`IAsyncEnumerable<T>`応じて`ChannelReader` 、またはを引数`InvokeAsync`とし`StreamAsChannelAsync`て、、またはに`SendAsync`渡すことができます。

オブジェクト`IAsyncEnumerable`または`ChannelWriter`オブジェクトにデータが書き込まれるたびに、サーバーのハブメソッドは、クライアントからのデータを使用して新しい項目を受け取ります。

`IAsyncEnumerable`オブジェクトを使用している場合、ストリームは、ストリーム項目を返すメソッドが終了した後に終了します。

[!INCLUDE[](~/includes/csharp-8-required.md)]

```csharp
async IAsyncEnumerable<string> clientStreamData()
{
    for (var i = 0; i < 5; i++)
    {
        var data = await FetchSomeData();
        yield return data;
    }
    //After the for loop has completed and the local function exits the stream completion will be sent.
}

await connection.SendAsync("UploadStream", clientStreamData());
```

または、を使用し`ChannelWriter`ている場合は、次`channel.Writer.Complete()`のようにチャネルを完成させます。

```csharp
var channel = Channel.CreateBounded<string>(10);
await connection.SendAsync("UploadStream", channel.Reader);
await channel.Writer.WriteAsync("some data");
await channel.Writer.WriteAsync("some more data");
channel.Writer.Complete();
```

::: moniker-end

## <a name="javascript-client"></a>JavaScript クライアント

### <a name="server-to-client-streaming"></a>サーバーからクライアントへのストリーミング

JavaScript クライアントは、を使用して`connection.stream`ハブでサーバーからクライアントへのストリーミングメソッドを呼び出します。 `stream`メソッドは 2 つの引数を受け取ります。

* ハブ メソッドの名前。 次の例では、ハブメソッド名は`Counter`です。
* ハブメソッドで定義されている引数。 次の例では、引数は、受信するストリーム項目数とストリーム項目間の遅延をカウントします。

`connection.stream`メソッドを`subscribe`含むを返します。 `IStreamResult` `IStreamSubscriber`を`next`に`subscribe`渡し、、 `stream` 、およびの`complete`各コールバックを設定して、呼び出しから通知を受信します。 `error`

::: moniker range=">= aspnetcore-2.2"

[!code-javascript[Streaming javascript](streaming/samples/2.2/wwwroot/js/stream.js?range=19-36)]

クライアントからストリームを終了するには、 `dispose` `subscribe`メソッドから返され`ISubscription`たに対してメソッドを呼び出します。 このメソッドを呼び出すと、 `CancellationToken`ハブメソッドのパラメーターがキャンセルされます (指定した場合)。

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-javascript[Streaming javascript](streaming/samples/2.1/wwwroot/js/stream.js?range=19-36)]

クライアントからストリームを終了するには、 `dispose` `subscribe`メソッドから返され`ISubscription`たに対してメソッドを呼び出します。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

### <a name="client-to-server-streaming"></a>クライアントとサーバー間のストリーミング

JavaScript クライアントは、呼び出されたハブメソッドに応じて、を`Subject`引数として、、または`send` `invoke` `stream`に渡すことによって、ハブでクライアントからサーバーへのストリーミングメソッドを呼び出します。 は、の`Subject`ように見えるクラスです。 `Subject` たとえば、RxJS では、そのライブラリの[サブジェクト](https://rxjs-dev.firebaseapp.com/api/index/class/Subject)クラスを使用できます。

[!code-javascript[Upload javascript](streaming/samples/3.0/wwwroot/js/stream.js?range=41-51)]

項目`subject.next(item)`を使用してを呼び出すと、項目がストリームに書き込まれ、ハブメソッドはサーバー上の項目を受け取ります。

ストリームを終了するには`subject.complete()`、を呼び出します。

## <a name="java-client"></a>Java クライアント

### <a name="server-to-client-streaming"></a>サーバーからクライアントへのストリーミング

SignalR Java クライアントは、 `stream`メソッドを使用してストリーミングメソッドを呼び出します。 `stream`3つ以上の引数を受け取ります。

* ストリーム項目の予期される型。
* ハブ メソッドの名前。
* ハブメソッドで定義されている引数。

```java
hubConnection.stream(String.class, "ExampleStreamingHubMethod", "Arg1")
    .subscribe(
        (item) -> {/* Define your onNext handler here. */ },
        (error) -> {/* Define your onError handler here. */},
        () -> {/* Define your onCompleted handler here. */});
```

`stream` の`HubConnection`メソッドは、ストリーム項目の型の観測可能なを返します。 観測可能な型`subscribe`のメソッドは`onNext`、 `onError` 、 `onCompleted`およびハンドラーが定義されている場所です。

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [ハブ](xref:signalr/hubs)
* [.NET クライアント](xref:signalr/dotnet-client)
* [JavaScript クライアント](xref:signalr/javascript-client)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
