---
title: バックグラウンドサービスでのホスト ASP.NET Core SignalR
author: bradygaster
description: .NET Core BackgroundService クラスから SignalR クライアントにメッセージを送信する方法について説明します。
monikerRange: '>= aspnetcore-2.2'
ms.author: bradyg
ms.custom: mvc
ms.date: 02/04/2019
uid: signalr/background-services
ms.openlocfilehash: 23a53f33a03ce3b76cf6846c3f214a6cad055209
ms.sourcegitcommit: f65d8765e4b7c894481db9b37aa6969abc625a48
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/06/2019
ms.locfileid: "70773931"
---
# <a name="host-aspnet-core-signalr-in-background-services"></a>バックグラウンドサービスでのホスト ASP.NET Core SignalR

作成者: [Brady](https://twitter.com/bradygaster)

この記事では、次のガイダンスを提供します。

* ASP.NET Core でホストされたバックグラウンドワーカープロセスを使用して SignalR Hub をホストする。
* .NET Core [Backgroundservice](xref:Microsoft.Extensions.Hosting.BackgroundService)内から接続されたクライアントにメッセージを送信する。

[サンプルコードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/sample/)する[(ダウンロード方法)](xref:index#how-to-download-a-sample)

## <a name="wire-up-signalr-during-startup"></a>スタートアップ時の SignalR の接続

::: moniker range=">= aspnetcore-3.0"

バックグラウンドワーカープロセスのコンテキストで ASP.NET Core SignalR ハブをホストすることは、ASP.NET Core web アプリでハブをホストすることと同じです。 メソッドで、を呼び`services.AddSignalR`出すと、SignalR をサポートするために必要なサービスが ASP.NET Core 依存関係挿入 (DI) レイヤーに追加されます。 `Startup.ConfigureServices` で`Startup.Configure`は、 `UseEndpoints`メソッドがコールバックで呼び出され、ASP.NET Core 要求パイプラインでハブエンドポイントを接続します`MapHub` 。

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR();
        services.AddHostedService<Worker>();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHub<ClockHub>("/hubs/clock");
        });
    }
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

バックグラウンドワーカープロセスのコンテキストで ASP.NET Core SignalR ハブをホストすることは、ASP.NET Core web アプリでハブをホストすることと同じです。 メソッドで、を呼び`services.AddSignalR`出すと、SignalR をサポートするために必要なサービスが ASP.NET Core 依存関係挿入 (DI) レイヤーに追加されます。 `Startup.ConfigureServices` で`Startup.Configure`は、 `UseSignalR`メソッドを呼び出して、ASP.NET Core 要求パイプラインでハブエンドポイントを接続します。

[!code-csharp[Startup](background-service/sample/Server/Startup.cs?name=Startup)]

::: moniker-end

前の例では、 `ClockHub`クラスは、 `Hub<T>`厳密に型指定されたハブを作成するクラスを実装しています。 は`ClockHub` 、エンドポイント`/hubs/clock`での`Startup`要求に応答するようにクラスで構成されています。

厳密に型指定されたハブの詳細については、「 [Use hub in ASP.NET Core SignalR」](xref:signalr/hubs#strongly-typed-hubs)を参照してください。

> [!NOTE]
> この機能は[\<ハブ t >](xref:Microsoft.AspNetCore.SignalR.Hub`1)クラスに限定されません。 また、 [Dynamichub](xref:Microsoft.AspNetCore.SignalR.DynamicHub)など、[ハブ](xref:Microsoft.AspNetCore.SignalR.Hub)から継承されるすべてのクラスも機能します。

[!code-csharp[Startup](background-service/sample/Server/ClockHub.cs?name=ClockHub)]

厳密に型指定`ClockHub` `IClock`されたによって使用されるインターフェイスは、インターフェイスです。

[!code-csharp[Startup](background-service/sample/HubServiceInterfaces/IClock.cs?name=IClock)]

## <a name="call-a-signalr-hub-from-a-background-service"></a>バックグラウンドサービスから SignalR Hub を呼び出す

起動時`Worker`に、クラス`BackgroundService`はを使用して`AddHostedService`接続されます。

```csharp
services.AddHostedService<Worker>();
```

SignalR は`Startup`フェーズ中にも接続されるため、各ハブは ASP.NET Core の HTTP 要求パイプラインの個々のエンドポイントにアタッチされます。各ハブは`IHubContext<T>` 、サーバー上のによって表されます。 ASP.NET Core の DI 機能を使用して、 `BackgroundService`クラス、MVC コントローラークラス、Razor ページモデルなどのホスト層によってインスタンス化された他のクラスは、構築中にのインスタンスを受け入れることによって、サーバー側ハブへの`IHubContext<ClockHub, IClock>`参照を取得できます。

[!code-csharp[Startup](background-service/sample/Server/Worker.cs?name=Worker)]

バックグラウンドサービスで`ClockHub`メソッドが繰り返し呼び出されると、サーバーの現在の日付と時刻が、を使用して接続されたクライアントに送信`ExecuteAsync`されます。

## <a name="react-to-signalr-events-with-background-services"></a>バックグラウンドサービスを使用した SignalR イベントへの対応

SignalR 用の JavaScript クライアントを使用するシングルページアプリの場合と同様に、.net デスクトップアプリでを<xref:signalr/dotnet-client>使用`BackgroundService`して`IHostedService`を使用する場合は、またはの実装を使用して SignalR hub に接続し、イベントに応答することもできます。

クラス`ClockHubClient`は、インターフェイスと`IClock` `IHostedService`インターフェイスの両方を実装します。 これにより、の実行中`Startup`に接続して、サーバーからハブイベントに応答できるようになります。

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

初期化`ClockHubClient`中に、は`HubConnection`のインスタンスを作成し`IClock.ShowTime` 、ハブの`ShowTime`イベントのハンドラーとしてメソッドを使用します。

[!code-csharp[The ClockHubClient constructor](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

実装では、 `HubConnection`が非同期的に開始されます。 `IHostedService.StartAsync`

[!code-csharp[StartAsync method](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

メソッドの`IHostedService.StopAsync`実行中`HubConnection` 、は非同期的に破棄されます。

[!code-csharp[StopAsync method](background-service/sample/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

## <a name="additional-resources"></a>その他の技術情報

* [開始するには](xref:tutorials/signalr)
* [ハブ](xref:signalr/hubs)
* [Azure に発行する](xref:signalr/publish-to-azure-web-app)
* [厳密に型指定されたハブ](xref:signalr/hubs#strongly-typed-hubs)
