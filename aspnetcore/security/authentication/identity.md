---
title: ASP.NET Core での Id の概要
author: rick-anderson
description: ASP.NET Core アプリで Id を使用します。 パスワードの要件 (RequireDigit、RequiredLength、RequiredUniqueChars など) を設定する方法について説明します。
ms.author: riande
ms.date: 10/15/2019
uid: security/authentication/identity
ms.openlocfilehash: 8da13ca5f74a9c829eb8137d33af0684ff88266d
ms.sourcegitcommit: 07d98ada57f2a5f6d809d44bdad7a15013109549
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2019
ms.locfileid: "72333551"
---
# <a name="introduction-to-identity-on-aspnet-core"></a>ASP.NET Core での Id の概要

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core Id は、ユーザーインターフェイス (UI) のログイン機能をサポートするメンバーシップシステムです。 ユーザーは、Id に格納されているログイン情報を持つアカウントを作成することも、外部ログインプロバイダーを使用することもできます。 サポートされている外部ログインプロバイダーには、 [Facebook、Google、Microsoft アカウント、Twitter](xref:security/authentication/social/index)があります。

Id は、SQL Server データベースを使用して、ユーザー名、パスワード、およびプロファイルデータを格納するように構成できます。 別の永続ストアを使用することもできます (たとえば、Azure Table Storage)。

このトピックでは、Identity を使用してユーザーを登録、ログイン、ログアウトする方法について説明します。 Id を使用するアプリを作成する方法の詳細については、この記事の最後にある「次のステップ」を参照してください。

[!INCLUDE[](~/includes/IdentityServer4.md)]

サンプルコード ([ダウンロード方法)](xref:index#how-to-download-a-sample)を[表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/identity/sample)します。

<a name="adi"></a>

## <a name="create-a-web-app-with-authentication"></a>認証を使用して Web アプリを作成する

個々のユーザーアカウントを使用して ASP.NET Core Web アプリケーションプロジェクトを作成します。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* [ **File** > **New** > **Project**] を選択します。
* **[ASP.NET Core Web アプリケーション]** を選択します。 プロジェクトに**WebApp1**という名前を付け、プロジェクトのダウンロードと同じ名前空間にします。 **[OK]** をクリックします。
* ASP.NET Core **Web アプリケーション**を選択し、 **[認証の変更]** を選択します。
* **個々のユーザーアカウント**を選択し、[ **OK]** をクリックします。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

上記のコマンドは、SQLite を使用して Razor web アプリを作成します。 LocalDB を使用して web アプリを作成するには、次のコマンドを実行します。

```dotnetcli
dotnet new webapp --auth Individual -uld -o WebApp1
```

---

生成されたプロジェクトは、 [ASP.NET Core id](xref:security/authentication/identity)を[Razor クラスライブラリ](xref:razor-pages/ui-class)として提供します。 Identity Razor クラスライブラリは、`Identity` 領域を持つエンドポイントを公開します。 (例:

* /アカウント/ログイン
* /アカウント/ログアウト
* /アカウント/管理

### <a name="apply-migrations"></a>移行を適用する

移行を適用してデータベースを初期化します。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

パッケージマネージャーコンソール (PMC) で次のコマンドを実行します。

`PM> Update-Database`

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

SQLite を使用する場合、この手順で移行する必要はありません。 LocalDB の場合は、次のコマンドを実行します。

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a>テストレジスタとログイン

アプリを実行し、ユーザーを登録します。 画面のサイズによっては、[ナビゲーション] トグルボタンを選択して、**登録**リンクと**ログイン**リンクを表示する必要がある場合があります。

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-identity-services"></a>Id サービスを構成する

サービスは @no__t 0 で追加されます。 一般的なパターンは、すべての `Add{Service}` メソッドを呼び出した後、すべての `services.Configure{Service}` メソッドを呼び出すことです。

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configureservices&highlight=10-99)]

上の強調表示されたコードは、既定のオプション値を使用して Id を構成します。 サービスは、[依存関係の挿入](xref:fundamentals/dependency-injection)によってアプリで使用できるようになります。

Id は <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> を呼び出すことによって有効になります。 `UseAuthentication` は、認証[ミドルウェア](xref:fundamentals/middleware/index)を要求パイプラインに追加します。

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configure&highlight=19)]

テンプレートで生成されたアプリは、[承認](xref:security/authorization/secure-data)を使用しません。 アプリが承認を追加するのと同じ順序で追加されるように、`app.UseAuthorization` が含まれています。 `UseRouting`、`UseAuthentication`、`UseAuthorization`、`UseEndpoints` は、前のコードに示されている順序で呼び出す必要があります。

@No__t-0 および `Startup` の詳細については、「<xref:Microsoft.AspNetCore.Identity.IdentityOptions>」と「[アプリケーションの起動](xref:fundamentals/startup)」を参照してください。

## <a name="scaffold-register-login-and-logout"></a>スキャフォールディング Register、Login、および LogOut

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Register、Login、および LogOut の各ファイルを追加します。 スキャフォールディング id に従って、このセクションに示されているコードを生成するための[承認命令を含む Razor プロジェクトに](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization)従います。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

**WebApp1**という名前のプロジェクトを作成した場合は、次のコマンドを実行します。 それ以外の場合は、`ApplicationDbContext` に正しい名前空間を使用します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

PowerShell では、コマンドの区切り記号としてセミコロンを使用します。 PowerShell を使用する場合は、前の例に示したように、ファイルリスト内のセミコロンをエスケープするか、ファイルリストを二重引用符で囲みます。

スキャフォールディング Id の詳細については、「[スキャフォールディング identity to a Razor project with authorization](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization)」を参照してください。

---

### <a name="examine-register"></a>レジスタの確認

ユーザーが **[登録]** リンクをクリックすると、`RegisterModel.OnPostAsync` アクションが呼び出されます。 ユーザーは、`_userManager` オブジェクトの[Createasync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_)によって作成されます。 `_userManager` は依存関係の挿入によって提供されます):

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=9)]

ユーザーが正常に作成された場合、ユーザーは `_signInManager.SignInAsync` への呼び出しによってログインされます。

登録時にすぐにログインできないようにする手順については、「[アカウントの確認](xref:security/authentication/accconfirm#prevent-login-at-registration)」をご覧ください。

### <a name="log-in"></a>ログイン

ログインフォームは次の場合に表示されます。

* **ログイン**リンクが選択されています。
* ユーザーがアクセスを承認されていない、**または**システムによって認証されていない制限付きページにアクセスしようとしています。

ログインページのフォームが送信されると、`OnPostAsync` アクションが呼び出されます。 `PasswordSignInAsync` は、(依存関係の挿入によって提供される) `_signInManager` オブジェクトで呼び出されます。

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

Base `Controller` クラスは、コントローラーメソッドからアクセスできる @no__t 1 プロパティを公開します。 たとえば、@no__t 0 を列挙し、承認の決定を行うことができます。 詳細については、「<xref:security/authorization/introduction>」を参照してください。

### <a name="log-out"></a>ログアウト

**Log out**リンクは、`LogoutModel.OnPost` アクションを呼び出します。 

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Logout.cshtml.cs?highlight=36)]

前のコードでは、ブラウザーが新しい要求を実行し、ユーザーの id が更新されるように、コード `return RedirectToPage();` はリダイレクトである必要があります。

[Signoutasync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync)者は、cookie に格納されているユーザーの要求をクリアします。

Post は、 *Pages/Shared/Loginpartial*に指定されています。

[!code-csharp[](identity/sample/WebApp3/Pages/Shared/_LoginPartial.cshtml?highlight=15)]

## <a name="test-identity"></a>テスト Id

既定の web プロジェクトテンプレートでは、ホームページへの匿名アクセスが許可されます。 Id をテストするには[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute)を追加します。

[!code-csharp[](identity/sample/WebApp3/Pages/Privacy.cshtml.cs?highlight=7)]

サインインしている場合は、サインアウトします。アプリを実行し、 **[プライバシー]** リンクを選択します。 ログインページにリダイレクトされます。

### <a name="explore-identity"></a>Id の探索

Id の詳細については、以下を参照してください。

* [完全な id UI ソースの作成](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* 各ページのソースを確認し、デバッガーをステップ実行します。

## <a name="identity-components"></a>Id コンポーネント

すべての Id に依存する NuGet パッケージは、 [ASP.NET Core 共有フレームワーク](xref:aspnetcore-3.0#use-the-aspnet-core-shared-framework)に含まれています。

Id のプライマリパッケージは[Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)です。 このパッケージには ASP.NET Core Id のインターフェイスのコアセットが含まれており、`Microsoft.AspNetCore.Identity.EntityFrameworkCore` によって含まれています。

## <a name="migrating-to-aspnet-core-identity"></a>ASP.NET Core Id への移行

既存の Id ストアを移行する方法の詳細とガイダンスについては、「[認証と id の移行](xref:migration/identity)」を参照してください。

## <a name="setting-password-strength"></a>パスワードの強度を設定する

パスワードの最小要件を設定するサンプルについては、「[構成](#pw)」を参照してください。

## <a name="adddefaultidentity-and-addidentity"></a>AddDefaultIdentity と AddIdentity

<xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> は ASP.NET Core 2.1 で導入されました。 @No__t-0 の呼び出しは、次の呼び出しに似ています。

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

詳細については、「 [AddDefaultIdentity source](https://github.com/aspnet/AspNetCore/blob/release/3.0/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) 」を参照してください。

## <a name="next-steps"></a>次のステップ

* [Identity の構成](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core Id は、ASP.NET Core アプリにログイン機能を追加するメンバーシップシステムです。 ユーザーは、Id に格納されているログイン情報を持つアカウントを作成することも、外部ログインプロバイダーを使用することもできます。 サポートされている外部ログインプロバイダーには、 [Facebook、Google、Microsoft アカウント、Twitter](xref:security/authentication/social/index)があります。

Id は、SQL Server データベースを使用して、ユーザー名、パスワード、およびプロファイルデータを格納するように構成できます。 別の永続ストアを使用することもできます (たとえば、Azure Table Storage)。

サンプルコード ([ダウンロード方法)](xref:index#how-to-download-a-sample)を[表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/identity/sample/src/ASPNETCore-IdentityDemoComplete/)します。

このトピックでは、Identity を使用してユーザーを登録、ログイン、ログアウトする方法について説明します。 Id を使用するアプリを作成する方法の詳細については、この記事の最後にある「次のステップ」を参照してください。

<a name="adi"></a>

## <a name="adddefaultidentity-and-addidentity"></a>AddDefaultIdentity と AddIdentity

<xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> は ASP.NET Core 2.1 で導入されました。 @No__t-0 の呼び出しは、次の呼び出しに似ています。

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

詳細については、「 [AddDefaultIdentity source](https://github.com/aspnet/AspNetCore/blob/release/3.0/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) 」を参照してください。

## <a name="create-a-web-app-with-authentication"></a>認証を使用して Web アプリを作成する

個々のユーザーアカウントを使用して ASP.NET Core Web アプリケーションプロジェクトを作成します。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* [ **File** > **New** > **Project**] を選択します。
* **[ASP.NET Core Web アプリケーション]** を選択します。 プロジェクトに**WebApp1**という名前を付け、プロジェクトのダウンロードと同じ名前空間にします。 **[OK]** をクリックします。
* ASP.NET Core **Web アプリケーション**を選択し、 **[認証の変更]** を選択します。
* **個々のユーザーアカウント**を選択し、[ **OK]** をクリックします。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

---

生成されたプロジェクトは、 [ASP.NET Core id](xref:security/authentication/identity)を[Razor クラスライブラリ](xref:razor-pages/ui-class)として提供します。 Identity Razor クラスライブラリは、`Identity` 領域を持つエンドポイントを公開します。 (例:

* /アカウント/ログイン
* /アカウント/ログアウト
* /アカウント/管理

### <a name="apply-migrations"></a>移行を適用する

移行を適用してデータベースを初期化します。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

パッケージマネージャーコンソール (PMC) で次のコマンドを実行します。

```PM> Update-Database```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a>テストレジスタとログイン

アプリを実行し、ユーザーを登録します。 画面のサイズによっては、[ナビゲーション] トグルボタンを選択して、**登録**リンクと**ログイン**リンクを表示する必要がある場合があります。

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-identity-services"></a>Id サービスを構成する

サービスは @no__t 0 で追加されます。 一般的なパターンは、すべての `Add{Service}` メソッドを呼び出した後、すべての `services.Configure{Service}` メソッドを呼び出すことです。

[!code-csharp[](identity/sample/WebApp1/Startup.cs?name=snippet_configureservices)]

前のコードでは、既定のオプション値を使用して Id を構成しています。 サービスは、[依存関係の挿入](xref:fundamentals/dependency-injection)によってアプリで使用できるようになります。

Id は、 [Useauthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_)を呼び出すことによって有効になります。 `UseAuthentication` は、認証[ミドルウェア](xref:fundamentals/middleware/index)を要求パイプラインに追加します。

[!code-csharp[](identity/sample/WebApp1/Startup.cs?name=snippet_configure&highlight=18)]

詳細については、「[クラス](/dotnet/api/microsoft.aspnetcore.identity.identityoptions)と[アプリケーションの起動](xref:fundamentals/startup)」を参照してください。

## <a name="scaffold-register-login-and-logout"></a>スキャフォールディング Register、Login、および LogOut

スキャフォールディング id に従って、このセクションに示されているコードを生成するための[承認命令を含む Razor プロジェクトに](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization)従います。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Register、Login、および LogOut の各ファイルを追加します。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

**WebApp1**という名前のプロジェクトを作成した場合は、次のコマンドを実行します。 それ以外の場合は、`ApplicationDbContext` に正しい名前空間を使用します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

PowerShell では、コマンドの区切り記号としてセミコロンを使用します。 PowerShell を使用する場合は、前の例に示したように、ファイルリスト内のセミコロンをエスケープするか、ファイルリストを二重引用符で囲みます。

---

### <a name="examine-register"></a>レジスタの確認

ユーザーが **[登録]** リンクをクリックすると、`RegisterModel.OnPostAsync` アクションが呼び出されます。 ユーザーは、`_userManager` オブジェクトの[Createasync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_)によって作成されます。 `_userManager` は依存関係の挿入によって提供されます):

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=7)]

ユーザーが正常に作成された場合、ユーザーは `_signInManager.SignInAsync` への呼び出しによってログインされます。

**注:** 登録時にすぐにログインできないようにする手順については、「[アカウントの確認](xref:security/authentication/accconfirm#prevent-login-at-registration)」をご覧ください。

### <a name="log-in"></a>ログイン

ログインフォームは次の場合に表示されます。

* **ログイン**リンクが選択されています。
* ユーザーがアクセスを承認されていない、**または**システムによって認証されていない制限付きページにアクセスしようとしています。

ログインページのフォームが送信されると、`OnPostAsync` アクションが呼び出されます。 `PasswordSignInAsync` は、(依存関係の挿入によって提供される) `_signInManager` オブジェクトで呼び出されます。

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

Base `Controller` クラスは、コントローラーメソッドからアクセスできる `User` プロパティを公開します。 たとえば、@no__t 0 を列挙し、承認の決定を行うことができます。 詳細については、「<xref:security/authorization/introduction>」を参照してください。

### <a name="log-out"></a>ログアウト

**Log out**リンクは、`LogoutModel.OnPost` アクションを呼び出します。 

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Logout.cshtml.cs)]

[Signoutasync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync)者は、cookie に格納されているユーザーの要求をクリアします。

Post は、 *Pages/Shared/Loginpartial*に指定されています。

[!code-csharp[](identity/sample/WebApp1/Pages/Shared/_LoginPartial.cshtml?highlight=16)]

## <a name="test-identity"></a>テスト Id

既定の web プロジェクトテンプレートでは、ホームページへの匿名アクセスが許可されます。 Id をテストするには、プライバシーページに[`[Authorize]`](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute)を追加します。

[!code-csharp[](identity/sample/WebApp1/Pages/Privacy.cshtml.cs?highlight=7)]

サインインしている場合は、サインアウトします。アプリを実行し、 **[プライバシー]** リンクを選択します。 ログインページにリダイレクトされます。

### <a name="explore-identity"></a>Id の探索

Id の詳細については、以下を参照してください。

* [完全な id UI ソースの作成](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* 各ページのソースを確認し、デバッガーをステップ実行します。

## <a name="identity-components"></a>Id コンポーネント

すべての Id 依存 NuGet パッケージは、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)に含まれています。

Id のプライマリパッケージは[Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)です。 このパッケージには ASP.NET Core Id のインターフェイスのコアセットが含まれており、`Microsoft.AspNetCore.Identity.EntityFrameworkCore` によって含まれています。

## <a name="migrating-to-aspnet-core-identity"></a>ASP.NET Core Id への移行

既存の Id ストアを移行する方法の詳細とガイダンスについては、「[認証と id の移行](xref:migration/identity)」を参照してください。

## <a name="setting-password-strength"></a>パスワードの強度を設定する

パスワードの最小要件を設定するサンプルについては、「[構成](#pw)」を参照してください。

## <a name="next-steps"></a>次のステップ

* [Identity の構成](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end
