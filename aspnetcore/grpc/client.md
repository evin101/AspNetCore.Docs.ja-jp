---
title: .NET クライアントを使用して gRPC サービスを呼び出す
author: jamesnk
description: .NET gRPC クライアントを使用して gRPC サービスを呼び出す方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/21/2019
uid: grpc/client
ms.openlocfilehash: 56f79b303a8d53699e8eb6156d328c0da1259416
ms.sourcegitcommit: dc5b293e08336dc236de66ed1834f7ef78359531
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/16/2019
ms.locfileid: "71011144"
---
# <a name="call-grpc-services-with-the-net-client"></a>.NET クライアントを使用して gRPC サービスを呼び出す

.NET gRPC クライアントライブラリは、 [gRPC .net. クライアント](https://www.nuget.org/packages/Grpc.Net.Client)NuGet パッケージで入手できます。 このドキュメントでは、次の方法について説明します。

* gRPC サービスを呼び出すように gRPC クライアントを構成します。
* gRPC に対して、単項、サーバーストリーミング、クライアントストリーミング、および双方向のストリーミングメソッドを呼び出します。

## <a name="configure-grpc-client"></a>GRPC クライアントを構成する

gRPC クライアントは、 [  *\*プロトコル*ファイルから生成](xref:grpc/basics#generated-c-assets)される具象クライアントの種類です。 具象 gRPC クライアントには、  *\*proto*ファイルの gRPC サービスに変換するメソッドがあります。

gRPC クライアントは、チャネルから作成されます。 まずを使用`GrpcChannel.ForAddress`してチャネルを作成し、次にチャネルを使用して gRPC クライアントを作成します。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greet.GreeterClient(channel);
```

チャネルは、gRPC サービスへの長期的な接続を表します。 チャネルが作成されると、サービスの呼び出しに関連するオプションを使用して構成されます。 たとえば、呼び出しを`HttpClient`行うために使用される、送信メッセージと受信メッセージの最大サイズ、およびログ記録`GrpcChannelOptions`は、で`GrpcChannel.ForAddress`指定し、と共に使用できます。 オプションの完全な一覧については、「[クライアント構成オプション](xref:grpc/configuration#configure-client-options)」を参照してください。

チャネルの作成は高価な操作であり、gRPC 呼び出しのチャネルを再利用すると、パフォーマンスが向上します。 複数の具象 gRPC クライアントは、さまざまな種類のクライアントを含むチャネルから作成できます。 具象 gRPC クライアントの種類は軽量のオブジェクトであり、必要に応じて作成できます。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");

var greeterClient = new Greet.GreeterClient(channel);
var counterClient = new Count.CounterClient(channel);

// Use clients to call gRPC services
```

`GrpcChannel.ForAddress`gRPC クライアントを作成するための唯一のオプションではありません。 ASP.NET Core アプリから gRPC サービスを呼び出す場合は、 [gRPC クライアントファクトリの統合](xref:grpc/clientfactory)を検討してください。 gRPC との`HttpClientFactory`統合には、gRPC クライアントを作成するための一元化された代替手段が用意されています。

> [!NOTE]
> [.Net クライアントでセキュリティで保護されていない gRPC サービスを呼び出す](xref:grpc/troubleshoot#call-insecure-grpc-services-with-net-core-client)には、追加の構成が必要です。

## <a name="make-grpc-calls"></a>GRPC 呼び出しを行う

gRPC 呼び出しは、クライアントでメソッドを呼び出すことによって開始されます。 gRPC クライアントは、メッセージのシリアル化を処理し、適切なサービスに対する gRPC 呼び出しに対処します。

gRPC には、さまざまな種類のメソッドがあります。 クライアントを使用して gRPC 呼び出しを行う方法は、呼び出すメソッドの種類によって異なります。 gRPC のメソッドの種類は次のとおりです。

* 単項
* サーバーストリーミング
* クライアントストリーミング
* 双方向ストリーミング

### <a name="unary-call"></a>単項呼び出し

単項呼び出しは、クライアントからの要求メッセージの送信を開始します。 サービスの終了時に応答メッセージが返されます。

```csharp
var client = new Greet.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = "World" });

Console.WriteLine("Greeting: " + response.Message);
// Greeting: Hello World
```

*\*Proto*ファイルの各単項サービスメソッドは、メソッドを呼び出すための具象 gRPC クライアント型に対して、非同期メソッドとブロッキングメソッドの2つの .net メソッドを生成します。 たとえば、を呼び出す`GreeterClient` `SayHello`には、次の2つの方法があります。

* `GreeterClient.SayHelloAsync`-サービス`Greeter.SayHello`を非同期的に呼び出します。 待機することができます。
* `GreeterClient.SayHello`-サービス`Greeter.SayHello`を呼び出し、が完了するまでブロックします。 非同期コードでは使用しないでください。

### <a name="server-streaming-call"></a>サーバーストリーミング呼び出し

サーバーストリーミング呼び出しは、クライアントから要求メッセージを送信することで開始されます。 `ResponseStream.MoveNext()`サービスからストリーミングされたメッセージを読み取ります。 が返さ`ResponseStream.MoveNext()` `false`れると、サーバーストリーミングの呼び出しが完了します。

```csharp
var client = new Greet.GreeterClient(channel);
using (var call = client.SayHellos(new HelloRequest { Name = "World" }))
{
    while (await call.ResponseStream.MoveNext())
    {
        Console.WriteLine("Greeting: " + call.ResponseStream.Current.Message);
        // "Greeting: Hello World" is written multiple times
    }
}
```

8以降を使用C#している場合は`await foreach` 、構文を使用してメッセージを読み取ることができます。 拡張`IAsyncStreamReader<T>.ReadAllAsync()`メソッドは、応答ストリームからすべてのメッセージを読み取ります。

```csharp
var client = new Greet.GreeterClient(channel);
using (var call = client.SayHellos(new HelloRequest { Name = "World" }))
{
    await foreach (var response in call.ResponseStream.ReadAllAsync())
    {
        Console.WriteLine("Greeting: " + response.Message);
        // "Greeting: Hello World" is written multiple times
    }
}
```

### <a name="client-streaming-call"></a>クライアントストリーミング呼び出し

クライアントのストリーミング呼び出しは、クライアントがメッセージを送信する*ことなく*開始されます。 クライアントは、を使用して`RequestStream.WriteAsync`メッセージを送信することを選択できます。 クライアントがメッセージ`RequestStream.CompleteAsync`の送信を完了したら、サービスに通知するためにを呼び出す必要があります。 サービスが応答メッセージを返すと、呼び出しが完了します。

```csharp
var client = new Counter.CounterClient(channel);
using (var call = client.AccumulateCount())
{
    for (var i = 0; i < 3; i++)
    {
        await call.RequestStream.WriteAsync(new CounterRequest { Count = 1 });
    }
    await call.RequestStream.CompleteAsync();

    var response = await call;
    Console.WriteLine($"Count: {response.Count}");
    // Count: 3
}
```

### <a name="bi-directional-streaming-call"></a>双方向ストリーミング呼び出し

双方向のストリーミング呼び出しは、クライアントがメッセージを送信する*ことなく*開始されます。 クライアントは、を使用して`RequestStream.WriteAsync`メッセージを送信することを選択できます。 サービスからストリーミングされたメッセージに`ResponseStream.MoveNext()`は`ResponseStream.ReadAllAsync()`、またはを使用してアクセスできます。 にメッセージ`ResponseStream`がなくなった場合、双方向のストリーミング呼び出しは完了です。

```csharp
using (var call = client.Echo())
{
    Console.WriteLine("Starting background task to receive messages");
    var readTask = Task.Run(async () =>
    {
        await foreach (var response in call.ResponseStream.ReadAllAsync())
        {
            Console.WriteLine(response.Message);
            // Echo messages sent to the service
        }
    });

    Console.WriteLine("Starting to send messages");
    Console.WriteLine("Type a message to echo then press enter.");
    while (true)
    {
        var result = Console.ReadLine();
        if (string.IsNullOrEmpty(result))
        {
            break;
        }

        await call.RequestStream.WriteAsync(new EchoMessage { Message = result });
    }

    Console.WriteLine("Disconnecting");
    await call.RequestStream.CompleteAsync();
    await readTask;
}
```

双方向のストリーミング呼び出しでは、クライアントとサービスはいつでもメッセージを相互に送信できます。 双方向呼び出しとの対話に最適なクライアントロジックは、サービスロジックによって異なります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/clientfactory>
* <xref:grpc/basics>
