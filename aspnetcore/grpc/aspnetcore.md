---
title: ASP.NET Core を使用した gRPC サービス
author: juntaoluo
description: ASP.NET Core で gRPC サービスを作成する際の基本的な概念について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/03/2019
uid: grpc/aspnetcore
ms.openlocfilehash: 2507ce6df05403cb19e8bfa2565d410d6140b144
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2019
ms.locfileid: "71925061"
---
# <a name="grpc-services-with-aspnet-core"></a>ASP.NET Core を使用した gRPC サービス

このドキュメントでは、ASP.NET Core を使用して gRPC サービスを開始する方法について説明します。

## <a name="prerequisites"></a>前提条件

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

## <a name="get-started-with-grpc-service-in-aspnet-core"></a>ASP.NET Core の gRPC サービスの概要

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/grpc/grpc-start/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

GRPC プロジェクトを作成する方法の詳細については、「 [grpc サービスの概要](xref:tutorials/grpc/grpc-start)」を参照してください。

# <a name="visual-studio-code--visual-studio-for-mactabvisual-studio-codevisual-studio-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

コマンド ラインから `dotnet new grpc -o GrpcGreeter` を実行します。

---

## <a name="add-grpc-services-to-an-aspnet-core-app"></a>ASP.NET Core アプリに gRPC サービスを追加する

gRPC には[Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore)パッケージが必要です。

### <a name="configure-grpc"></a>GRPC の構成

*Startup.cs* の場合:

* grpc は、 `AddGrpc`メソッドを使用して有効になっています。
* 各 grpc サービスは、メソッドを`MapGrpcService`使用してルーティングパイプラインに追加されます。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Startup.cs?name=snippet&highlight=7,24)]

ASP.NET Core ミドルウェアと features はルーティングパイプラインを共有するため、追加の要求ハンドラーを提供するようにアプリを構成できます。 MVC コントローラーなどの追加の要求ハンドラーは、構成されている gRPC サービスと並行して動作します。

### <a name="configure-kestrel"></a>Kestrel の構成

Kestrel gRPC エンドポイント:

* HTTP/2 が必要です。
* は、[トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246)を使用してセキュリティで保護する必要があります。

#### <a name="http2"></a>HTTP/2

gRPC には HTTP/2 が必要です。 gRPC for ASP.NET Core では、 [HttpRequest](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*)が`HTTP/2`検証されます。

Kestrel は、最新のオペレーティングシステムで[HTTP/2 をサポート](xref:fundamentals/servers/kestrel#http2-support)しています。 Kestrel エンドポイントは、既定で HTTP/1.1 接続と HTTP/2 接続をサポートするように構成されています。

#### <a name="tls"></a>TLS

GRPC に使用される kestrel エンドポイントは、TLS を使用してセキュリティで保護する必要があります。 開発では、TLS を使用してセキュリティで保護`https://localhost:5001`されたエンドポイントは、ASP.NET Core 開発証明書が存在するときに、に自動的に作成されます。 構成は必要ありません。 `https`プレフィックスは、kestrel エンドポイントが TLS を使用していることを確認します。

運用環境では、TLS を明示的に構成する必要があります。 次の*appsettings*の例では、TLS で保護された HTTP/2 エンドポイントが用意されています。

[!code-json[](~/grpc/aspnetcore/sample/appsettings.json?highlight=4)]

または、 *Program.cs*で Kestrel エンドポイントを構成することもできます。

[!code-csharp[](~/grpc/aspnetcore/sample/Program.cs?highlight=7&name=snippet)]

#### <a name="protocol-negotiation"></a>プロトコルネゴシエーション

TLS は、通信のセキュリティ保護以外にも使用されます。 エンドポイントで複数のプロトコルがサポートされている場合、TLS[アプリケーションレイヤープロトコルネゴシエーション (ALPN)](https://tools.ietf.org/html/rfc7301#section-3)ハンドシェイクを使用して、クライアントとサーバー間の接続プロトコルをネゴシエートします。 このネゴシエーションは、接続が HTTP/1.1 または HTTP/2 を使用するかどうかを判断します。

HTTP/2 エンドポイントが TLS を使用せずに構成されている場合は、エンドポイント`HttpProtocols.Http2`の[listenoptions](xref:fundamentals/servers/kestrel#listenoptionsprotocols)がに設定されている必要があります。 ネゴシエーションがないため、複数のプロトコル ( `HttpProtocols.Http1AndHttp2`たとえば、) を持つエンドポイントを TLS なしで使用することはできません。 セキュリティで保護されていないエンドポイントへのすべての接続は、既定で HTTP/1.1 に設定され、gRPC の呼び出しは失敗します。

Kestrel での HTTP/2 および TLS の有効化の詳細については、「 [kestrel エンドポイントの構成](xref:fundamentals/servers/kestrel#endpoint-configuration)」を参照してください。

> [!NOTE]
> macOS の場合、ASP.NET Core gRPC と TLS の組み合わせに対応していません。 macOS で gRPC サービスを正常に実行するには、追加の構成が必要です。 詳細については、[macOS で ASP.NET Core gRPC アプリを起動できない](xref:grpc/troubleshoot#unable-to-start-aspnet-core-grpc-app-on-macos)場合に関するページを参照してください。

## <a name="integration-with-aspnet-core-apis"></a>ASP.NET Core Api との統合

gRPC サービスには、[依存関係の挿入](xref:fundamentals/dependency-injection)(DI) や[ログ記録](xref:fundamentals/logging/index)などの ASP.NET Core 機能へのフルアクセスがあります。 たとえば、サービス実装は、コンストラクターを使用して DI コンテナーから logger サービスを解決できます。

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

既定では、gRPC サービス実装は、任意の有効期間 (シングルトン、スコープ、または一時的) を持つ他の DI サービスを解決できます。

### <a name="resolve-httpcontext-in-grpc-methods"></a>GRPC メソッドでの HttpContext の解決

GRPC API は、メソッド、ホスト、ヘッダー、トレーラーなど、一部の HTTP/2 メッセージデータへのアクセスを提供します。 アクセスは、各`ServerCallContext` grpc メソッドに渡される引数を使用して行われます。

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService.cs?highlight=3-4&name=snippet)]

`ServerCallContext`では、すべての ASP.NET `HttpContext` api でへのフルアクセスが提供されるわけではありません。 拡張`GetHttpContext`メソッドは、ASP.NET api の基`HttpContext`になる HTTP/2 メッセージを表すへのフルアクセスを提供します。

[!code-csharp[](~/grpc/aspnetcore/sample/GrcpService/GreeterService2.cs?highlight=6-7&name=snippet)]

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:fundamentals/servers/kestrel>
