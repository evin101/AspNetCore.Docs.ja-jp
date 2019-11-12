---
title: GRPC での認証と承認 (ASP.NET Core)
author: jamesnk
description: GRPC で認証と承認を使用して ASP.NET Core する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/13/2019
uid: grpc/authn-and-authz
ms.openlocfilehash: e8dd384ec43a66e56891925dcaa529085fa200c7
ms.sourcegitcommit: 6d26ab647ede4f8e57465e29b03be5cb130fc872
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/07/2019
ms.locfileid: "71999862"
---
# <a name="authentication-and-authorization-in-grpc-for-aspnet-core"></a>GRPC での認証と承認 (ASP.NET Core)

[James Newton-King](https://twitter.com/jamesnk)別

[サンプルコードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/grpc/authn-and-authz/sample/)[する (ダウンロードする方法)](xref:index#how-to-download-a-sample)

## <a name="authenticate-users-calling-a-grpc-service"></a>GRPC サービスを呼び出しているユーザーを認証する

gRPC を[ASP.NET Core 認証](xref:security/authentication/identity)と共に使用すると、ユーザーを各呼び出しに関連付けることができます。

GRPC と ASP.NET Core 認証を使用する @no__t 0 の例を次に示します。

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();
    
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        routes.MapGrpcService<GreeterService>();
    });
}
```

> [!NOTE]
> ASP.NET Core 認証ミドルウェアを登録する順序は重要です。 常に `UseAuthentication` を呼び出し、`UseRouting` と `UseEndpoints` の後に-1 を @no__t します。

呼び出し時にアプリが使用する認証メカニズムを構成する必要があります。 認証の構成は @no__t 0 で追加されます。アプリが使用する認証メカニズムによって異なります。 ASP.NET Core アプリをセキュリティで保護する方法の例については、「[認証のサンプル](xref:security/authentication/samples)」を参照してください。

認証がセットアップされると、ユーザーには `ServerCallContext` を使用して gRPC サービスメソッドでアクセスできるようになります。

```csharp
public override Task<BuyTicketsResponse> BuyTickets(
    BuyTicketsRequest request, ServerCallContext context)
{
    var user = context.GetHttpContext().User;

    // ... access data from ClaimsPrincipal ...
}

```

### <a name="bearer-token-authentication"></a>ベアラートークン認証

クライアントは、認証用のアクセストークンを提供できます。 サーバーはトークンを検証し、それを使用してユーザーを識別します。

サーバーでは、ベアラートークン認証は[JWT ベアラーミドルウェア](/dotnet/api/microsoft.extensions.dependencyinjection.jwtbearerextensions.addjwtbearer)を使用して構成されます。

.NET gRPC クライアントでは、トークンをヘッダーとして呼び出しと共に送信できます。

```csharp
public bool DoAuthenticatedCall(
    Ticketer.TicketerClient client, string token)
{
    var headers = new Metadata();
    headers.Add("Authorization", $"Bearer {token}");

    var request = new BuyTicketsRequest { Count = 1 };
    var response = await client.BuyTicketsAsync(request, headers);

    return response.Success;
}
```

チャネルで `ChannelCredentials` を構成することは、gRPC 呼び出しを使用してトークンをサービスに送信する別の方法です。 資格情報は gRPC 呼び出しが行われるたびに実行されます。これにより、トークンを自分で渡すためにコードを複数の場所に記述する必要がなくなります。

次の例の資格情報は、すべての gRPC 呼び出しを使用してトークンを送信するようにチャネルを構成します。

```csharp
private static GrpcChannel CreateAuthenticatedChannel(string address)
{
    var credentials = CallCredentials.FromInterceptor((context, metadata) =>
    {
        if (!string.IsNullOrEmpty(_token))
        {
            metadata.Add("Authorization", $"Bearer {_token}");
        }
        return Task.CompletedTask;
    });

    // SslCredentials is used here because this channel is using TLS.
    // Channels that aren't using TLS should use ChannelCredentials.Insecure instead.
    var channel = GrpcChannel.ForAddress(address, new GrpcChannelOptions
    {
        Credentials = ChannelCredentials.Create(new SslCredentials(), credentials)
    });
    return channel;
}
```

### <a name="client-certificate-authentication"></a>クライアント証明書の認証

クライアントは、認証用にクライアント証明書を提供することもできます。 [証明書の認証](https://tools.ietf.org/html/rfc5246#section-7.4.4)は、TLS レベルで実行されますが、その前に ASP.NET Core になります。 要求が ASP.NET Core になると、[クライアント証明書の認証パッケージ](xref:security/authentication/certauth)によって、証明書を `ClaimsPrincipal` に解決できるようになります。

> [!NOTE]
> クライアント証明書を受け入れるようにホストを構成する必要があります。 Kestrel、IIS、および Azure でのクライアント証明書の受け入れの詳細については、「[証明書を要求するようにホストを構成する](xref:security/authentication/certauth#configure-your-host-to-require-certificates)」を参照してください。

.NET gRPC クライアントでは、クライアント証明書が `HttpClientHandler` に追加され、次に gRPC クライアントの作成に使用されます。

```csharp
public Ticketer.TicketerClient CreateClientWithCert(
    string baseAddress,
    X509Certificate2 certificate)
{
    // Add client cert to the handler
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(certificate);

    // Create the gRPC channel
    var channel = GrpcChannel.ForAddress(baseAddress, new GrpcChannelOptions
    {
        HttpClient = new HttpClient(handler)
    });

    return new Ticketer.TicketerClient(channel);
}
```

### <a name="other-authentication-mechanisms"></a>その他の認証メカニズム

GRPC では、ASP.NET Core サポートされている多くの認証メカニズムが使用できます。

* Azure Active Directory
* クライアント証明書
* IdentityServer
* JWT トークン
* OAuth 2.0
* OpenID Connect
* WS-Federation

サーバーでの認証の構成の詳細については、「 [ASP.NET Core 認証](xref:security/authentication/identity)」を参照してください。

認証を使用するように gRPC クライアントを構成することは、使用している認証メカニズムによって異なります。 前のベアラートークンとクライアント証明書の例では、grpc 呼び出しを使用して認証メタデータを送信するように gRPC クライアントを構成するいくつかの方法を示しています。

* 厳密に型指定された gRPC クライアントは @no__t 0 を内部的に使用します。 認証は[`HttpClientHandler`](/dotnet/api/system.net.http.httpclienthandler)で構成することも、`HttpClient` にカスタム[の @no__t](/dotnet/api/system.net.http.httpmessagehandler)インスタンスを追加することによって構成することもできます。
* 各 gRPC 呼び出しには、省略可能な @no__t 0 引数があります。 カスタムヘッダーは、オプションの headers コレクションを使用して送信できます。

> [!NOTE]
> Windows 認証 (NTLM/Kerberos/Negotiate) を gRPC で使用することはできません。 gRPC には HTTP/2 が必要ですが、HTTP/2 は Windows 認証をサポートしていません。

## <a name="authorize-users-to-access-services-and-service-methods"></a>サービスとサービスメソッドへのアクセスをユーザーに承認する

既定では、サービス内のすべてのメソッドは、認証されていないユーザーが呼び出すことができます。 認証を要求するには、 [[承認]](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)属性をサービスに適用します。

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
}
```

@No__t-0 属性のコンストラクター引数とプロパティを使用して、特定の[承認ポリシー](xref:security/authorization/policies)に一致するユーザーのみにアクセスを制限できます。 たとえば、`MyAuthorizationPolicy` というカスタム承認ポリシーがある場合は、次のコードを使用して、そのポリシーに一致するユーザーだけがサービスにアクセスできることを確認します。

```csharp
[Authorize("MyAuthorizationPolicy")]
public class TicketerService : Ticketer.TicketerBase
{
}
```

個々のサービスメソッドには、@no__t 0 属性も適用できます。 現在のユーザーが、メソッドとクラスの**両方**に適用されているポリシーと一致しない場合、エラーが呼び出し元に返されます。

```csharp
[Authorize]
public class TicketerService : Ticketer.TicketerBase
{
    public override Task<AvailableTicketsResponse> GetAvailableTickets(
        Empty request, ServerCallContext context)
    {
        // ... buy tickets for the current user ...
    }

    [Authorize("Administrators")]
    public override Task<BuyTicketsResponse> RefundTickets(
        BuyTicketsRequest request, ServerCallContext context)
    {
        // ... refund tickets (something only Administrators can do) ..
    }
}
```

## <a name="additional-resources"></a>その他の技術情報

* [ASP.NET Core でのベアラートークン認証](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [ASP.NET Core でクライアント証明書認証を構成する](xref:security/authentication/certauth)
