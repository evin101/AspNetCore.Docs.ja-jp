---
title: ClaimsPrincipal からの移行
author: mjrousos
description: ASP.NET Core で現在認証されているユーザーの id と要求を取得するために、ClaimsPrincipal から移行する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 03/26/2019
uid: migration/claimsprincipal-current
ms.openlocfilehash: f7472f5b851d3869da3d26b881e276ce4ca004fb
ms.sourcegitcommit: 231780c8d7848943e5e9fd55e93f437f7e5a371d
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2019
ms.locfileid: "74115975"
---
# <a name="migrate-from-claimsprincipalcurrent"></a>ClaimsPrincipal からの移行

ASP.NET 4.x プロジェクトでは、 [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal.current)を使用して、現在認証されているユーザーの id と要求を取得するのが一般的でした。 ASP.NET Core では、このプロパティは現在設定されていません。 別の方法で現在認証されているユーザーの id を取得するには、それに依存していたコードを更新する必要があります。

## <a name="context-specific-data-instead-of-static-data"></a>静的なデータではなく、コンテキスト固有のデータ

ASP.NET Core を使用する場合、`ClaimsPrincipal.Current` と `Thread.CurrentPrincipal` の両方の値が設定されていません。 これらのプロパティはどちらも静的な状態を表しているので、通常は回避 ASP.NET Core ます。 代わりに、ASP.NET Core のアーキテクチャは、コンテキスト固有のサービスコレクション ([依存関係挿入](xref:fundamentals/dependency-injection)(DI) モデルを使用) から依存関係 (現在のユーザーの id など) を取得することです。 さらに、スレッドが静的である `Thread.CurrentPrincipal`、一部の非同期シナリオでは変更が保持されない可能性があります (と `ClaimsPrincipal.Current` 既定で `Thread.CurrentPrincipal` を呼び出すだけです)。

スレッドの静的メンバーが非同期のシナリオで発生する可能性のある問題の種類を理解するには、次のコードスニペットを考えてみます。

```csharp
// Create a ClaimsPrincipal and set Thread.CurrentPrincipal
var identity = new ClaimsIdentity();
identity.AddClaim(new Claim(ClaimTypes.Name, "User1"));
Thread.CurrentPrincipal = new ClaimsPrincipal(identity);

// Check the current user
Console.WriteLine($"Current user: {Thread.CurrentPrincipal?.Identity.Name}");

// For the method to complete asynchronously
await Task.Yield();

// Check the current user after
Console.WriteLine($"Current user: {Thread.CurrentPrincipal?.Identity.Name}");
```

前のサンプルコードでは、非同期呼び出しを待機する前と後に、`Thread.CurrentPrincipal` を設定し、その値をチェックします。 `Thread.CurrentPrincipal` は、それが設定されている*スレッド*に固有であり、メソッドは await の後に別のスレッドで実行を再開する可能性があります。 したがって、最初にチェックされたときは `Thread.CurrentPrincipal` が存在しますが、`await Task.Yield()`の呼び出しの後は null になります。

テスト id は簡単に挿入できるため、アプリの DI サービスコレクションから現在のユーザーの id を取得することも、よりテストが容易になります。

## <a name="retrieve-the-current-user-in-an-aspnet-core-app"></a>ASP.NET Core アプリで現在のユーザーを取得する

`ClaimsPrincipal.Current`の代わりに ASP.NET Core で現在認証されているユーザーの `ClaimsPrincipal` を取得するには、いくつかのオプションがあります。

* **コントローラーの基本ユーザー**。 MVC コントローラーは、[ユーザー](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.user)プロパティを使用して、現在の認証済みユーザーにアクセスできます。
* **HttpContext. ユーザー**。 現在の `HttpContext` (ミドルウェアなど) にアクセスできるコンポーネントは、現在のユーザーの `ClaimsPrincipal` を[HttpContext.User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user)から取得できます。
* **呼び出し元から渡さ**れました。 現在の `HttpContext` にアクセスできないライブラリは、多くの場合コントローラーまたはミドルウェアコンポーネントから呼び出され、現在のユーザーの id を引数として渡すことができます。
* **IHttpContextAccessor**。 ASP.NET Core に移行されるプロジェクトが大きすぎて、現在のユーザーの id を必要なすべての場所に簡単に渡すことができない場合があります。 このような場合は、回避策として[IHttpContextAccessor](/dotnet/api/microsoft.aspnetcore.http.ihttpcontextaccessor)を使用できます。 `IHttpContextAccessor` は現在の `HttpContext` にアクセスできます (存在する場合)。 DI が使用されている場合は、「<xref:fundamentals/httpcontext>」を参照してください。 ASP.NET Core の DI ドリブンアーキテクチャで動作するようにまだ更新されていないコードで現在のユーザーの id を取得するための短期的なソリューションは次のようになります。

  * `Startup.ConfigureServices`で[AddHttpContextAccessor](https://github.com/aspnet/Hosting/issues/793)を呼び出して、`IHttpContextAccessor` DI コンテナーで使用できるようにします。
  * 起動時に `IHttpContextAccessor` のインスタンスを取得し、静的変数に格納します。 インスタンスは、以前に静的プロパティから現在のユーザーを取得していたコードで使用できるようになります。
  * `HttpContextAccessor.HttpContext?.User`を使用して、現在のユーザーの `ClaimsPrincipal` を取得します。 このコードが HTTP 要求のコンテキストの外部で使用されている場合、`HttpContext` は null になります。

最後のオプションでは、静的変数に格納されている `IHttpContextAccessor` インスタンスを使用して、ASP.NET Core の原則 (静的な依存関係に挿入された依存関係を優先) とは逆になります。 代わりに、最終的に依存関係の挿入から `IHttpContextAccessor` インスタンスを取得することを計画します。 ただし、`ClaimsPrincipal.Current`を以前に使用していた大規模な既存の ASP.NET アプリを移行する場合、静的ヘルパーは便利なブリッジになることがあります。
