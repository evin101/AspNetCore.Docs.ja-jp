---
title: .NET での gRPC でのログ記録と診断
author: jamesnk
description: .NET で gRPC アプリから診断を収集する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 09/23/2019
uid: grpc/diagnostics
ms.openlocfilehash: 7194e91b40a08c4a7ee619b8f207900af2683aa1
ms.sourcegitcommit: fae6f0e253f9d62d8f39de5884d2ba2b4b2a6050
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/25/2019
ms.locfileid: "71250734"
---
# <a name="logging-and-diagnostics-in-grpc-on-net"></a>.NET での gRPC でのログ記録と診断

[James のニュートン-キング](https://twitter.com/jamesnk)別

この記事では、問題のトラブルシューティングに役立つ gRPC アプリから診断情報を収集するためのガイダンスを提供します。

## <a name="grpc-services-logging"></a>gRPC サービスのログ記録

> [!WARNING]
> サーバー側のログには、アプリからの機密情報が含まれている場合があります。 運用アプリから GitHub などのパブリックフォーラムに未加工のログを投稿**しない**でください。

gRPC サービスは ASP.NET Core でホストされるため、ASP.NET Core ログシステムを使用します。 既定の構成では、gRPC はほとんど情報をログに記録しませんが、これは構成できます。 ASP.NET Core ログの構成の詳細については、 [ASP.NET Core のログ記録](xref:fundamentals/logging/index#configuration)に関するドキュメントを参照してください。

gRPC は、カテゴリの`Grpc`下にログを追加します。 gRPC から詳細なログを有効にする`Grpc`には、 `Debug`の`LogLevel` `Logging`サブセクションに次の項目を追加して、 *appsettings*ファイルのレベルにプレフィックスを構成します。

[!code-json[](diagnostics/sample/logging-config.json?highlight=7)]

これは、 *Startup.cs*で次のよう`ConfigureLogging`に構成することもできます。

[!code-csharp[](diagnostics/sample/logging-config-code.cs?highlight=5)]

JSON ベースの構成を使用していない場合は、構成システムで次の構成値を設定します。

* `Logging:LogLevel:Grpc` = `Debug`

構成システムのドキュメントを参照して、入れ子になった構成値を指定する方法を確認してください。 たとえば、環境変数を使用する場合、 `_`の代わり`:`に2文字が使用されます`Logging__LogLevel__Grpc`(たとえば、)。

アプリのより詳細`Debug`な診断情報を収集する場合は、レベルを使用することをお勧めします。 レベル`Trace`は非常に低いレベルの診断を生成するため、アプリの問題を診断するためにはほとんど必要ありません。

### <a name="sample-logging-output"></a>サンプルのログ記録の出力

gRPC サービスの`Debug`レベルでのコンソール出力の例を次に示します。

```console
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
dbug: Grpc.AspNetCore.Server.ServerCallHandler[1]
      Reading message.
info: GrpcService.GreeterService[0]
      Hello World
dbug: Grpc.AspNetCore.Server.ServerCallHandler[6]
      Sending message.
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 1.4113ms 200 application/grpc
```

## <a name="access-server-side-logs"></a>サーバー側のログへのアクセス

サーバー側のログにアクセスする方法は、を実行している環境によって異なります。

### <a name="as-a-console-app"></a>コンソールアプリとして

コンソールアプリでを実行している場合は、既定で[コンソール logger](xref:fundamentals/logging/index#console-provider)が有効になっている必要があります。 gRPC ログがコンソールに表示されます。

### <a name="other-environments"></a>その他の環境

アプリが別の環境 (Docker、Kubernetes、または Windows サービスなど) に配置されている<xref:fundamentals/logging/index>場合は、「」を参照して、環境に適したログプロバイダーを構成する方法の詳細を確認してください。

## <a name="grpc-client-logging"></a>gRPC クライアントのログ記録

> [!WARNING]
> クライアント側のログには、アプリからの機密情報が含まれている場合があります。 運用アプリから GitHub などのパブリックフォーラムに未加工のログを投稿**しない**でください。

.Net クライアントからログを取得するには、クライアントの`GrpcChannelOptions.LoggerFactory`チャネルの作成時にプロパティを設定します。 ASP.NET Core アプリから gRPC サービスを呼び出す場合は、logger ファクトリを依存関係の挿入 (DI) から解決できます。

[!code-csharp[](diagnostics/sample/net-client-dependency-injection.cs?highlight=7,16)]

クライアントのログ記録を有効にする別の方法として、 [gRPC クライアントファクトリ](xref:grpc/clientfactory)を使用してクライアントを作成する方法があります。 クライアントファクトリに登録され、DI から解決される gRPC クライアントは、アプリの構成済みログを自動的に使用します。

アプリが DI を使用していない場合は、 `ILoggerFactory` [server.loggerfactory](xref:Microsoft.Extensions.Logging.LoggerFactory.Create*)を使用して新しいインスタンスを作成できます。 このメソッドにアクセスするには、アプリに[Microsoft 拡張子のログ](https://www.nuget.org/packages/microsoft.extensions.logging/)パッケージを追加します。

[!code-csharp[](diagnostics/sample/net-client-loggerfactory-create.cs?highlight=1,8)]

### <a name="sample-logging-output"></a>サンプルのログ記録の出力

gRPC クライアントの`Debug`レベルでのコンソール出力の例を次に示します。

```console
dbug: Grpc.Net.Client.Internal.GrpcCall[1]
      Starting gRPC call. Method type: 'Unary', URI: 'https://localhost:5001/Greet.Greeter/SayHello'.
dbug: Grpc.Net.Client.Internal.GrpcCall[6]
      Sending message.
dbug: Grpc.Net.Client.Internal.GrpcCall[1]
      Reading message.
dbug: Grpc.Net.Client.Internal.GrpcCall[4]
      Finished gRPC call.
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/logging/index>
* <xref:grpc/configuration>
* <xref:grpc/clientfactory>
