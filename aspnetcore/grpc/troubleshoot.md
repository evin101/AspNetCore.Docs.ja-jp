---
title: .NET Core での gRPC のトラブルシューティング
author: jamesnk
description: .NET Core で gRPC を使用しているときのエラーのトラブルシューティングを行います。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 10/16/2019
uid: grpc/troubleshoot
ms.openlocfilehash: c501cda14f3bac9297695ece59cbc4634e4b7895
ms.sourcegitcommit: e71b6a85b0e94a600af607107e298f932924c849
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/17/2019
ms.locfileid: "72519056"
---
# <a name="troubleshoot-grpc-on-net-core"></a>.NET Core での gRPC のトラブルシューティング

[James のニュートン-キング](https://twitter.com/jamesnk)別

このドキュメントでは、.NET で gRPC アプリを開発するときによく発生する問題について説明します。

## <a name="mismatch-between-client-and-service-ssltls-configuration"></a>クライアントとサービスの SSL/TLS の構成が一致しません

GRPC テンプレートとサンプルでは、[トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246)を使用して、既定で grpc サービスをセキュリティで保護しています。 gRPC クライアントはセキュリティで保護された接続を使用して、セキュリティで保護された gRPC サービスを正常に呼び出す必要があります。

アプリの起動時に書き込まれたログで、ASP.NET Core gRPC サービスが TLS を使用していることを確認できます。 サービスは、HTTPS エンドポイントでリッスンします。

```
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

.NET Core クライアントは、サーバーアドレスに `https` を使用して、セキュリティで保護された接続で呼び出しを行う必要があります。

```csharp
static async Task Main(string[] args)
{
    // The port number(5001) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("https://localhost:5001");
    var client = new Greet.GreeterClient(channel);
}
```

すべての gRPC クライアント実装は TLS をサポートしています。 他の言語の gRPC クライアントでは、通常、`SslCredentials` で構成されたチャネルが必要です。 `SslCredentials` は、クライアントが使用する証明書を指定します。この証明書は、セキュリティで保護されていない資格情報の代わりに使用する必要があります。 異なる gRPC クライアント実装で TLS を使用するように構成する例については、「 [Grpc 認証](https://www.grpc.io/docs/guides/auth/)」を参照してください。

## <a name="call-a-grpc-service-with-an-untrustedinvalid-certificate"></a>信頼されていない/無効な証明書を使用して gRPC サービスを呼び出す

.NET gRPC クライアントでは、サービスに信頼された証明書が必要です。 信頼された証明書を使用せずに gRPC サービスを呼び出すと、次のエラーメッセージが返されます。

> ハンドルされていない例外です。 System.net.http.httprequestexception: SSL 接続を確立できませんでした。内部例外を参照してください。
> ---> を実行しています。 AuthenticationException: 検証プロシージャによると、リモート証明書が無効です。

このエラーは、アプリをローカルでテストしていて、ASP.NET Core HTTPS 開発証明書が信頼されていない場合に表示されることがあります。 この問題を解決する手順については、「[Windows と macOS で ASP.NET Core HTTPS 開発証明書を信頼します](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)」を参照してください。

別のコンピューターで gRPC サービスを呼び出していて、その証明書を信頼できない場合、gRPC クライアントは無効な証明書を無視するように構成できます。 次のコードでは、 [Httpclienthandler. ServerCertificateCustomValidationCallback](/dotnet/api/system.net.http.httpclienthandler.servercertificatecustomvalidationcallback)を使用して、信頼された証明書がない呼び出しを許可します。

```csharp
var httpClientHandler = new HttpClientHandler();
// Return `true` to allow certificates that are untrusted/invalid
httpClientHandler.ServerCertificateCustomValidationCallback = 
    HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;
var httpClient = new HttpClient(httpClientHandler);

var channel = GrpcChannel.ForAddress("https://localhost:5001",
    new GrpcChannelOptions { HttpClient = httpClient });
var client = new Greet.GreeterClient(channel);
```

> [!WARNING]
> 信頼されていない証明書は、アプリの開発中にのみ使用してください。 実稼働アプリケーションでは、常に有効な証明書を使用する必要があります。

## <a name="call-insecure-grpc-services-with-net-core-client"></a>.NET Core クライアントを使用してセキュリティで保護される gRPC サービスを呼び出す

.NET Core クライアントでセキュリティで保護されていない gRPC サービスを呼び出すには、追加の構成が必要です。 GRPC クライアントは、`System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` スイッチを `true` に設定し、サーバーアドレスに `http` を使用する必要があります。

```csharp
// This switch must be set before creating the GrpcChannel/HttpClient.
AppContext.SetSwitch(
    "System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);

// The port number(5000) must match the port of the gRPC server.
var channel = GrpcChannel.ForAddress("http://localhost:5000");
var client = new Greet.GreeterClient(channel);
```

## <a name="unable-to-start-aspnet-core-grpc-app-on-macos"></a>MacOS で gRPC アプリを開始できません ASP.NET Core

Kestrel では、macOS での TLS と Windows 7 などの古いバージョンの HTTP/2 はサポートされていません。 ASP.NET Core gRPC テンプレートとサンプルでは、既定で TLS が使用されます。 GRPC サーバーを起動しようとすると、次のエラーメッセージが表示されます。

> IPv4 ループバックインターフェイスの https://localhost:5001 にバインドできません。 ALPN のサポートがないため、macOS で ' HTTP/2 over TLS はサポートされていません。 '。

この問題を回避するには、TLS を使用*せず*に HTTP/2 を使用するように Kestrel と grpc クライアントを構成します。 これは開発時にのみ実行してください。 TLS を使用しないと、gRPC メッセージが暗号化なしで送信されます。

Kestrel では、 *Program.cs*で TLS を使用せずに HTTP/2 エンドポイントを構成する必要があります。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                // Setup a HTTP/2 endpoint without TLS.
                options.ListenLocalhost(5000, o => o.Protocols = 
                    HttpProtocols.Http2);
            });
            webBuilder.UseStartup<Startup>();
        });
```

HTTP/2 エンドポイントが TLS を使用せずに構成されている場合は、エンドポイントの[Listenoptions](xref:fundamentals/servers/kestrel#listenoptionsprotocols)が `HttpProtocols.Http2` に設定されている必要があります。 HTTP/2 のネゴシエートに TLS が必要であるため、`HttpProtocols.Http1AndHttp2` は使用できません。 TLS を使用しない場合、エンドポイントへのすべての接続は既定の HTTP/1.1 に設定され、gRPC の呼び出しは失敗します。

GRPC クライアントは、TLS を使用しないように構成する必要もあります。 詳細については、「 [.Net Core クライアントを使用した安全でない gRPC サービスの呼び出し](#call-insecure-grpc-services-with-net-core-client)」を参照してください。

> [!WARNING]
> TLS を使用しない HTTP/2 は、アプリの開発中にのみ使用してください。 運用アプリでは、常にトランスポートセキュリティを使用する必要があります。 詳細については、「 [gRPC のセキュリティに関する考慮事項 ASP.NET Core](xref:grpc/security#transport-security)」を参照してください。

## <a name="grpc-c-assets-are-not-code-generated-from-proto-files"></a>gRPC C#資産は、proto ファイルから生成されたコードではありません

具象クライアントとサービス基底クラスの gRPC コード生成には、protobuf ファイルとツールをプロジェクトから参照する必要があります。 次のものを含める必要があります。

* `<Protobuf>` 項目グループで使用する*プロトコルファイル。* [インポートさ*れたプロトコル*ファイル](https://developers.google.com/protocol-buffers/docs/proto3#importing-definitions)は、プロジェクトによって参照される必要があります。
* GRPC ツールパッケージ[Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/)に対するパッケージリファレンス。

GRPC C#アセットの生成の詳細については、<xref:grpc/basics> を参照してください。

既定では、@no__t 0 の参照によって、具象クライアントとサービス基本クラスが生成されます。 参照要素の @no__t 0 属性を使用して、資産のC#生成を制限できます。 有効な `GrpcServices` オプションは次のとおりです。

* `Both` (存在しない場合は既定)
* `Server`
* `Client`
* `None`

GRPC サービスをホストしている ASP.NET Core web アプリには、生成されたサービス基本クラスのみが必要です。

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
</ItemGroup>
```

GRPC 呼び出しを行う gRPC クライアントアプリでは、具象クライアントのみが生成されます。

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
</ItemGroup>
```

## <a name="wpf-projects-unable-to-generate-grpc-c-assets-from-proto-files"></a>WPF プロジェクトは、proto ファイルからC# grpc アセットを生成できません

WPF プロジェクトには、gRPC コード生成が正常に動作しないという[既知の問題](https://github.com/dotnet/wpf/issues/810)があります。 @No__t-0 と*プロトコル*ファイルを参照することによって WPF プロジェクトで生成された grpc の種類では、次のようなコンパイルエラーが発生します。

> エラー CS0246: 型または名前空間の名前 ' MyGrpcServices ' が見つかりませんでした。 using ディレクティブまたはアセンブリ参照が指定されていることを確認してください。

この問題を回避するには、次の方法があります。

1. 新しい .NET Core クラスライブラリプロジェクトを作成します。
2. 新しいプロジェクトで、 [ C# *\** ファイルからのコード生成](xref:grpc/basics#generated-c-assets)を有効にするための参照を追加します。
    * [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) パッケージにパッケージ参照を追加します。
    * `<Protobuf>` 項目グループに *\*.proto* ファイルを追加します。
3. WPF アプリケーションで、新しいプロジェクトへの参照を追加します。

WPF アプリケーションでは、新しいクラスライブラリプロジェクトから、gRPC によって生成された型を使用できます。

[!INCLUDE[](~/includes/gRPCazure.md)]
