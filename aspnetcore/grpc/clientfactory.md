---
title: .NET Core での gRPC クライアントファクトリの統合
author: jamesnk
description: クライアントファクトリを使用して gRPC クライアントを作成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 11/12/2019
no-loc:
- SignalR
uid: grpc/clientfactory
ms.openlocfilehash: 3042bb61367f8b9a9f3142217ad329270ab2cca5
ms.sourcegitcommit: 3fc3020961e1289ee5bf5f3c365ce8304d8ebf19
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/12/2019
ms.locfileid: "73963679"
---
# <a name="grpc-client-factory-integration-in-net-core"></a>.NET Core での gRPC クライアントファクトリの統合

gRPC と `HttpClientFactory` の統合には、gRPC クライアントを一元的に作成する方法が用意されています。 [スタンドアロンの gRPC クライアントインスタンスを構成](xref:grpc/client)する代わりに使用することもできます。 ファクトリ統合は、 [gRPC .net. ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory) NuGet パッケージで使用できます。

ファクトリには、次のような利点があります。

* 論理 gRPC クライアントインスタンスを構成するための一元的な場所を提供します
* 基になる `HttpClientMessageHandler` の有効期間を管理します
* ASP.NET Core gRPC サービスでの期限とキャンセルの自動伝達

## <a name="register-grpc-clients"></a>GRPC クライアントの登録

gRPC クライアントを登録するには、`Startup.ConfigureServices`内で汎用 `AddGrpcClient` 拡張メソッドを使用し、gRPC で型指定されたクライアントクラスとサービスアドレスを指定します。

```csharp
services.AddGrpcClient<Greeter.GreeterClient>(o =>
{
    o.Address = new Uri("https://localhost:5001");
});
```

gRPC クライアントの種類は、依存関係の注入 (DI) と共に一時的に登録されます。 これで、クライアントを DI によって作成された型に直接挿入して使用できるようになりました。 ASP.NET Core MVC コントローラー、SignalR ハブ、および gRPC サービスは、gRPC クライアントが自動的に挿入される場所です。

```csharp
public class AggregatorService : Aggregator.AggregatorBase
{
    private readonly Greeter.GreeterClient _client;

    public AggregatorService(Greeter.GreeterClient client)
    {
        _client = client;
    }

    public override async Task SayHellos(HelloRequest request,
        IServerStreamWriter<HelloReply> responseStream, ServerCallContext context)
    {
        // Forward the call on to the greeter service
        using (var call = _client.SayHellos(request))
        {
            await foreach (var response in call.ResponseStream.ReadAllAsync())
            {
                await responseStream.WriteAsync(response);
            }
        }
    }
}
```

## <a name="configure-httpclient"></a>HttpClient の構成

`HttpClientFactory` は、gRPC クライアントによって使用される `HttpClient` を作成します。 標準 `HttpClientFactory` メソッドを使用して、発信要求ミドルウェアを追加したり、`HttpClient`の基になる `HttpClientHandler` を構成したりすることができます。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .ConfigurePrimaryHttpMessageHandler(() =>
    {
        var handler = new HttpClientHandler();
        handler.ClientCertificates.Add(LoadCertificate());
        return handler;
    });
```

詳細については、「 [IHttpClientFactory を使用して HTTP 要求を作成する](xref:fundamentals/http-requests)」を参照してください。

## <a name="configure-channel-and-interceptors"></a>チャネルとインターセプターの構成

gRPC 固有のメソッドは、次の目的で使用できます。

* gRPC クライアントの基になるチャネルを構成します。
* gRPC 呼び出しを行うときにクライアントが使用する `Interceptor` インスタンスを追加します。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .AddInterceptor(() => new LoggingInterceptor())
    .ConfigureChannel(o =>
    {
        o.Credentials = new CustomCredentials();
    });
```

## <a name="deadline-and-cancellation-propagation"></a>期限と取り消しの反映

gRPC サービスでファクトリによって作成された gRPC クライアントは、期限とキャンセルトークンを子呼び出しに自動的に反映するように `EnableCallContextPropagation()` を使用して構成できます。 `EnableCallContextPropagation()` の拡張メソッドは、 [AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore.Server.ClientFactory) NuGet パッケージで使用できますが、

呼び出しコンテキストの伝達は、現在の gRPC 要求コンテキストから期限とキャンセルトークンを読み取り、gRPC クライアントによって行われた発信呼び出しに自動的に伝達することによって機能します。 呼び出しコンテキストの伝達は、複雑で入れ子になった gRPC のシナリオが常に期限とキャンセルを伝達することを保証する優れた方法です。

```csharp
services
    .AddGrpcClient<Greeter.GreeterClient>(o =>
    {
        o.Address = new Uri("https://localhost:5001");
    })
    .EnableCallContextPropagation();
```

期限と RPC のキャンセルの詳細については、「 [rpc ライフサイクル](https://www.grpc.io/docs/guides/concepts/#rpc-life-cycle)」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/client>
* <xref:fundamentals/http-requests>
