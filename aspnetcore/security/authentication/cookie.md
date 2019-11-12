---
title: ASP.NET Core Id を指定せずに cookie 認証を使用する
author: rick-anderson
description: ASP.NET Core Id を使用せずに cookie 認証を使用する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 08/20/2019
uid: security/authentication/cookie
ms.openlocfilehash: 288fa4317801544bf0d689280c56836431017c89
ms.sourcegitcommit: 9e85c2562df5e108d7933635c830297f484bb775
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/04/2019
ms.locfileid: "73462931"
---
# <a name="use-cookie-authentication-without-aspnet-core-identity"></a>ASP.NET Core Id を指定せずに cookie 認証を使用する

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)と[Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core Id は、ログインを作成および管理するための完全な機能を備えた完全な認証プロバイダーです。 ただし、ASP.NET Core Id のない cookie ベースの認証プロバイダーを使用することもできます。 詳細については、「<xref:security/authentication/identity>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでのデモンストレーションのために、架空のユーザーであるマリア Rodriguez のユーザーアカウントは、アプリにハードコードされています。 **電子メール**アドレス `maria.rodriguez@contoso.com` とパスワードを使用して、ユーザーにサインインします。 ユーザーは、 *Pages/Account/Login. cshtml. .cs*ファイルの `AuthenticateUser` メソッドで認証されます。 実際の例では、ユーザーはデータベースに対して認証されます。

## <a name="configuration"></a>構成

アプリで[AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)が使用されていない場合は、 [Microsoft.AspNetCore.Authentication.Cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/)パッケージのプロジェクトファイルにパッケージ参照を作成します。

`Startup.ConfigureServices` メソッドで、<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> と <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> のメソッドを使用して認証ミドルウェアサービスを作成します。

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet1)]

`AddAuthentication` に渡された <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> は、アプリの既定の認証スキームを設定します。 `AuthenticationScheme` は、cookie 認証の複数のインスタンスがあり、[特定のスキームで承認](xref:security/authorization/limitingidentitybyscheme)する場合に便利です。 `AuthenticationScheme` を[Cookieauthenticationdefaults に設定します。 AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme)は、スキームに対して "cookie" の値を提供します。 スキームを区別する任意の文字列値を指定できます。

アプリの認証方式は、アプリの cookie 認証スキームとは異なります。 <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>に cookie 認証スキームが指定されていない場合、`CookieAuthenticationDefaults.AuthenticationScheme` ("Cookie") が使用されます。

認証 cookie の <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> プロパティは、既定では `true` に設定されています。 サイトビジターがデータコレクションに同意していない場合は、認証 cookie が許可されます。 詳細については、「<xref:security/gdpr#essential-cookies>」を参照してください。

`Startup.Configure`で `UseAuthentication` と `UseAuthorization` を呼び出して `HttpContext.User` プロパティを設定し、要求の承認ミドルウェアを実行します。 `UseEndpoints`を呼び出す前に、`UseAuthentication` および `UseAuthorization` メソッドを呼び出します。

[!code-csharp[](cookie/samples/3.x/CookieSample/Startup.cs?name=snippet2)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> クラスは、認証プロバイダーオプションを構成するために使用されます。

`Startup.ConfigureServices` 方法で認証を行うために、サービス構成の `CookieAuthenticationOptions` を設定します。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>クッキーポリシーミドルウェア

Cookie[ポリシーミドルウェア](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware)を使用すると、cookie ポリシー機能が有効になります。 アプリ処理パイプラインへのミドルウェアの追加は順序に依存しており、パイプラインに登録されている下流コンポーネントにのみ影響&mdash;ます。

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Cookie を追加または削除したときに cookie 処理のグローバル特性を制御し、クッキー処理ハンドラーにフックするには、クッキーポリシーミドルウェアに提供されている <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> を使用します。

既定の <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> 値は、OAuth2 認証を許可する `SameSiteMode.Lax` です。 `SameSiteMode.Strict`の同じサイトポリシーを厳密に適用するには、`MinimumSameSitePolicy`を設定します。 この設定は、OAuth2 およびその他のクロスオリジン認証スキームを分割しますが、クロスオリジン要求処理に依存しない他の種類のアプリの cookie セキュリティのレベルを昇格させます。

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

`MinimumSameSitePolicy` の Cookie ポリシーのミドルウェア設定は、次の表に従って `CookieAuthenticationOptions` 設定の `Cookie.SameSite` の設定に影響を与える可能性があります。

| MinimumSameSitePolicy | SameSite | 結果の SameSite 設定 |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode なし     | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode 厳密でない      | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode 厳密でない<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode Strict   | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode Strict<br>SameSiteMode Strict<br>SameSiteMode Strict |

## <a name="create-an-authentication-cookie"></a>認証 cookie を作成する

ユーザー情報を保持する cookie を作成するには、<xref:System.Security.Claims.ClaimsPrincipal>を構築します。 ユーザー情報がシリアル化され、cookie に格納されます。 

必要な <xref:System.Security.Claims.Claim>s を持つ <xref:System.Security.Claims.ClaimsIdentity> を作成し、<xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> を呼び出してユーザーにサインインします。

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync` は、暗号化されたクッキーを作成し、現在の応答に追加します。 `AuthenticationScheme` が指定されていない場合は、既定のスキームが使用されます。

暗号化には ASP.NET Core の[データ保護](xref:security/data-protection/using-data-protection)システムが使用されます。 複数のコンピューターでホストされているアプリの場合、アプリ間で負荷を分散する場合、または web ファームを使用する場合は、同じキーリングとアプリ識別子を使用するように[データ保護を構成](xref:security/data-protection/configuration/overview)します。

## <a name="sign-out"></a>サインアウト

現在のユーザーをサインアウトして cookie を削除するには、<xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>を呼び出します。

[!code-csharp[](cookie/samples/3.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

`CookieAuthenticationDefaults.AuthenticationScheme` (または "Cookie") がスキームとして使用されていない場合 (例: "ContosoCookie")、認証プロバイダーを構成するときに使用するスキームを指定します。 それ以外の場合は、既定のスキームが使用されます。

## <a name="react-to-back-end-changes"></a>バックエンドの変更に対処する

Cookie が作成されると、cookie は id の単一のソースになります。 バックエンドシステムでユーザーアカウントが無効になっている場合は、次のようになります。

* アプリの cookie 認証システムは、認証 cookie に基づいて要求を処理し続けます。
* 認証 cookie が有効である限り、ユーザーはアプリにサインインしたままになります。

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> イベントを使用して、cookie id の検証をインターセプトおよびオーバーライドできます。 すべての要求で cookie を検証すると、アプリにアクセスするユーザーが失効するリスクが軽減されます。

Cookie を検証する方法の1つは、ユーザーデータベースがいつ変更されたかを追跡することです。 ユーザーの cookie が発行されてからデータベースが変更されていない場合、cookie が有効である場合、ユーザーを再認証する必要はありません。 サンプルアプリでは、データベースは `IUserRepository` に実装され、`LastChanged` 値を格納します。 データベースでユーザーが更新されると、`LastChanged` の値が現在の時刻に設定されます。

`LastChanged` 値に基づいてデータベースが変更されたときに cookie を無効にするには、データベースの現在の `LastChanged` 値を含む `LastChanged` 要求を使用して cookie を作成します。

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

`ValidatePrincipal` イベントのオーバーライドを実装するには、<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>から派生したクラスに次のシグネチャを持つメソッドを記述します。

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

`CookieAuthenticationEvents`の実装例を次に示します。

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Cookie サービスの登録時に、`Startup.ConfigureServices` メソッドでイベントインスタンスを登録します。 `CustomCookieAuthenticationEvents` クラスの[スコープ付きサービス登録](xref:fundamentals/dependency-injection#service-lifetimes)を指定します。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

ユーザーの名前が&mdash;更新された場合に、セキュリティに影響を与えないようにするという状況を考えてみます。 ユーザープリンシパルを非破壊更新する場合は、`context.ReplacePrincipal` を呼び出し、`context.ShouldRenew` プロパティを `true`に設定します。

> [!WARNING]
> ここで説明する方法は、すべての要求でトリガーされます。 すべての要求に対して認証 cookie を検証すると、アプリのパフォーマンスが大幅に低下する可能性があります。

## <a name="persistent-cookies"></a>永続的な cookie

Cookie がブラウザーセッション間で保持されるようにすることもできます。 この永続化を有効にする必要があるのは、サインイン時に [忘れずに保存] チェックボックスをオンにして、明示的なユーザーの同意を有効にすることだけです。 

次のコードスニペットは、ブラウザーのクロージャを介して存続する id とそれに対応する cookie を作成します。 以前に構成したスライド式有効期限の設定はすべて受け入れられます。 ブラウザーを閉じている間に cookie の有効期限が切れると、ブラウザーは再起動後に cookie をクリアします。

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>で `true` に <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> を設定します。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>クッキーの絶対有効期限

絶対有効期限は <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>で設定できます。 永続的なクッキーを作成するには、`IsPersistent` も設定する必要があります。 それ以外の場合、cookie はセッションベースの有効期間で作成され、保持する認証チケットの前または後に有効期限が切れる可能性があります。 `ExpiresUtc` が設定されている場合は <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>の <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> オプションの値が上書きされます (設定されている場合)。

次のコードスニペットでは、20分間継続する id とそれに対応する cookie を作成します。 これにより、以前に構成したスライド式有効期限の設定は無視されます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core Id は、ログインを作成および管理するための完全な機能を備えた完全な認証プロバイダーです。 ただし、ASP.NET Core Id のない cookie ベースの認証認証プロバイダーを使用できます。 詳細については、「<xref:security/authentication/identity>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/cookie/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

サンプルアプリでのデモンストレーションのために、架空のユーザーであるマリア Rodriguez のユーザーアカウントは、アプリにハードコードされています。 **電子メール**アドレス `maria.rodriguez@contoso.com` とパスワードを使用して、ユーザーにサインインします。 ユーザーは、 *Pages/Account/Login. cshtml. .cs*ファイルの `AuthenticateUser` メソッドで認証されます。 実際の例では、ユーザーはデータベースに対して認証されます。

## <a name="configuration"></a>構成

アプリで[AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)が使用されていない場合は、 [Microsoft.AspNetCore.Authentication.Cookies](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Cookies/)パッケージのプロジェクトファイルにパッケージ参照を作成します。

`Startup.ConfigureServices` メソッドで、<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> および <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> メソッドを使用して認証ミドルウェアサービスを作成します。

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet1)]

`AddAuthentication` に渡された <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme> は、アプリの既定の認証スキームを設定します。 `AuthenticationScheme` は、cookie 認証の複数のインスタンスがあり、[特定のスキームで承認](xref:security/authorization/limitingidentitybyscheme)する場合に便利です。 `AuthenticationScheme` を[Cookieauthenticationdefaults に設定します。 AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme)は、スキームに対して "cookie" の値を提供します。 スキームを区別する任意の文字列値を指定できます。

アプリの認証方式は、アプリの cookie 認証スキームとは異なります。 <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*>に cookie 認証スキームが指定されていない場合、`CookieAuthenticationDefaults.AuthenticationScheme` ("Cookie") が使用されます。

認証 cookie の <xref:Microsoft.AspNetCore.Http.CookieBuilder.IsEssential> プロパティは、既定では `true` に設定されています。 サイトビジターがデータコレクションに同意していない場合は、認証 cookie が許可されます。 詳細については、「<xref:security/gdpr#essential-cookies>」を参照してください。

`Startup.Configure` メソッドで、`UseAuthentication` メソッドを呼び出して、`HttpContext.User` プロパティを設定する認証ミドルウェアを呼び出します。 `UseMvcWithDefaultRoute` または `UseMvc`を呼び出す前に、`UseAuthentication` メソッドを呼び出します。

[!code-csharp[](cookie/samples/2.x/CookieSample/Startup.cs?name=snippet2)]

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions> クラスは、認証プロバイダーオプションを構成するために使用されます。

`Startup.ConfigureServices` 方法で認証を行うために、サービス構成の `CookieAuthenticationOptions` を設定します。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        ...
    });
```

## <a name="cookie-policy-middleware"></a>クッキーポリシーミドルウェア

Cookie[ポリシーミドルウェア](xref:Microsoft.AspNetCore.CookiePolicy.CookiePolicyMiddleware)を使用すると、cookie ポリシー機能が有効になります。 アプリ処理パイプラインへのミドルウェアの追加は順序に依存しており、パイプラインに登録されている下流コンポーネントにのみ影響&mdash;ます。

```csharp
app.UseCookiePolicy(cookiePolicyOptions);
```

Cookie を追加または削除したときに cookie 処理のグローバル特性を制御し、クッキー処理ハンドラーにフックするには、クッキーポリシーミドルウェアに提供されている <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions> を使用します。

既定の <xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy> 値は、OAuth2 認証を許可する `SameSiteMode.Lax` です。 `SameSiteMode.Strict`の同じサイトポリシーを厳密に適用するには、`MinimumSameSitePolicy`を設定します。 この設定は、OAuth2 およびその他のクロスオリジン認証スキームを分割しますが、クロスオリジン要求処理に依存しない他の種類のアプリの cookie セキュリティのレベルを昇格させます。

```csharp
var cookiePolicyOptions = new CookiePolicyOptions
{
    MinimumSameSitePolicy = SameSiteMode.Strict,
};
```

`MinimumSameSitePolicy` の Cookie ポリシーのミドルウェア設定は、次の表に従って `CookieAuthenticationOptions` 設定の `Cookie.SameSite` の設定に影響を与える可能性があります。

| MinimumSameSitePolicy | SameSite | 結果の SameSite 設定 |
| --------------------- | --------------- | --------------------------------- |
| SameSiteMode なし     | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode 厳密でない      | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode 厳密でない<br>SameSiteMode 厳密でない<br>SameSiteMode Strict |
| SameSiteMode Strict   | SameSiteMode なし<br>SameSiteMode 厳密でない<br>SameSiteMode Strict | SameSiteMode Strict<br>SameSiteMode Strict<br>SameSiteMode Strict |

## <a name="create-an-authentication-cookie"></a>認証 cookie を作成する

ユーザー情報を保持する cookie を作成するには、<xref:System.Security.Claims.ClaimsPrincipal>を構築します。 ユーザー情報がシリアル化され、cookie に格納されます。 

必要な <xref:System.Security.Claims.Claim>s を持つ <xref:System.Security.Claims.ClaimsIdentity> を作成し、<xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*> を呼び出してユーザーにサインインします。

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet1)]

`SignInAsync` は、暗号化されたクッキーを作成し、現在の応答に追加します。 `AuthenticationScheme` が指定されていない場合は、既定のスキームが使用されます。

暗号化には ASP.NET Core の[データ保護](xref:security/data-protection/using-data-protection)システムが使用されます。 複数のコンピューターでホストされているアプリの場合、アプリ間で負荷を分散する場合、または web ファームを使用する場合は、同じキーリングとアプリ識別子を使用するように[データ保護を構成](xref:security/data-protection/configuration/overview)します。

## <a name="sign-out"></a>サインアウト

現在のユーザーをサインアウトして cookie を削除するには、<xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>を呼び出します。

[!code-csharp[](cookie/samples/2.x/CookieSample/Pages/Account/Login.cshtml.cs?name=snippet2)]

`CookieAuthenticationDefaults.AuthenticationScheme` (または "Cookie") がスキームとして使用されていない場合 (例: "ContosoCookie")、認証プロバイダーを構成するときに使用するスキームを指定します。 それ以外の場合は、既定のスキームが使用されます。

## <a name="react-to-back-end-changes"></a>バックエンドの変更に対処する

Cookie が作成されると、cookie は id の単一のソースになります。 バックエンドシステムでユーザーアカウントが無効になっている場合は、次のようになります。

* アプリの cookie 認証システムは、認証 cookie に基づいて要求を処理し続けます。
* 認証 cookie が有効である限り、ユーザーはアプリにサインインしたままになります。

<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents.ValidatePrincipal*> イベントを使用して、cookie id の検証をインターセプトおよびオーバーライドできます。 すべての要求で cookie を検証すると、アプリにアクセスするユーザーが失効するリスクが軽減されます。

Cookie を検証する方法の1つは、ユーザーデータベースがいつ変更されたかを追跡することです。 ユーザーの cookie が発行されてからデータベースが変更されていない場合、cookie が有効である場合、ユーザーを再認証する必要はありません。 サンプルアプリでは、データベースは `IUserRepository` に実装され、`LastChanged` 値を格納します。 データベースでユーザーが更新されると、`LastChanged` の値が現在の時刻に設定されます。

`LastChanged` 値に基づいてデータベースが変更されたときに cookie を無効にするには、データベースの現在の `LastChanged` 値を含む `LastChanged` 要求を使用して cookie を作成します。

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, user.Email),
    new Claim("LastChanged", {Database Value})
};

var claimsIdentity = new ClaimsIdentity(
    claims, 
    CookieAuthenticationDefaults.AuthenticationScheme);

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme, 
    new ClaimsPrincipal(claimsIdentity));
```

`ValidatePrincipal` イベントのオーバーライドを実装するには、<xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationEvents>から派生したクラスに次のシグネチャを持つメソッドを記述します。

```csharp
ValidatePrincipal(CookieValidatePrincipalContext)
```

`CookieAuthenticationEvents`の実装例を次に示します。

```csharp
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;

public class CustomCookieAuthenticationEvents : CookieAuthenticationEvents
{
    private readonly IUserRepository _userRepository;

    public CustomCookieAuthenticationEvents(IUserRepository userRepository)
    {
        // Get the database from registered DI services.
        _userRepository = userRepository;
    }

    public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
    {
        var userPrincipal = context.Principal;

        // Look for the LastChanged claim.
        var lastChanged = (from c in userPrincipal.Claims
                           where c.Type == "LastChanged"
                           select c.Value).FirstOrDefault();

        if (string.IsNullOrEmpty(lastChanged) ||
            !_userRepository.ValidateLastChanged(lastChanged))
        {
            context.RejectPrincipal();

            await context.HttpContext.SignOutAsync(
                CookieAuthenticationDefaults.AuthenticationScheme);
        }
    }
}
```

Cookie サービスの登録時に、`Startup.ConfigureServices` メソッドでイベントインスタンスを登録します。 `CustomCookieAuthenticationEvents` クラスの[スコープ付きサービス登録](xref:fundamentals/dependency-injection#service-lifetimes)を指定します。

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.EventsType = typeof(CustomCookieAuthenticationEvents);
    });

services.AddScoped<CustomCookieAuthenticationEvents>();
```

ユーザーの名前が&mdash;更新された場合に、セキュリティに影響を与えないようにするという状況を考えてみます。 ユーザープリンシパルを非破壊更新する場合は、`context.ReplacePrincipal` を呼び出し、`context.ShouldRenew` プロパティを `true`に設定します。

> [!WARNING]
> ここで説明する方法は、すべての要求でトリガーされます。 すべての要求に対して認証 cookie を検証すると、アプリのパフォーマンスが大幅に低下する可能性があります。

## <a name="persistent-cookies"></a>永続的な cookie

Cookie がブラウザーセッション間で保持されるようにすることもできます。 この永続化を有効にする必要があるのは、サインイン時に [忘れずに保存] チェックボックスをオンにして、明示的なユーザーの同意を有効にすることだけです。 

次のコードスニペットは、ブラウザーのクロージャを介して存続する id とそれに対応する cookie を作成します。 以前に構成したスライド式有効期限の設定はすべて受け入れられます。 ブラウザーを閉じている間に cookie の有効期限が切れると、ブラウザーは再起動後に cookie をクリアします。

<xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties>で `true` に <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.IsPersistent> を設定します。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true
    });
```

## <a name="absolute-cookie-expiration"></a>クッキーの絶対有効期限

絶対有効期限は <xref:Microsoft.AspNetCore.Authentication.AuthenticationProperties.ExpiresUtc>で設定できます。 永続的なクッキーを作成するには、`IsPersistent` も設定する必要があります。 それ以外の場合、cookie はセッションベースの有効期間で作成され、保持する認証チケットの前または後に有効期限が切れる可能性があります。 `ExpiresUtc` が設定されている場合は <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions>の <xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.ExpireTimeSpan> オプションの値が上書きされます (設定されている場合)。

次のコードスニペットでは、20分間継続する id とそれに対応する cookie を作成します。 これにより、以前に構成したスライド式有効期限の設定は無視されます。

```csharp
// using Microsoft.AspNetCore.Authentication;

await HttpContext.SignInAsync(
    CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(claimsIdentity),
    new AuthenticationProperties
    {
        IsPersistent = true,
        ExpiresUtc = DateTime.UtcNow.AddMinutes(20)
    });
```

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authorization/limitingidentitybyscheme>
* <xref:security/authorization/claims>
* [ポリシーベースのロールチェック](xref:security/authorization/roles#policy-based-role-checks)
* <xref:host-and-deploy/web-farm>
