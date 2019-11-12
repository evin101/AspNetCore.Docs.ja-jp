---
title: ASP.NET Core での Facebook 外部ログインのセットアップ
author: rick-anderson
description: Facebook アカウントのユーザー認証を既存の ASP.NET Core アプリに統合する方法を示すコード例を紹介したチュートリアルです。
ms.author: riande
ms.custom: seoapril2019, mvc, seodec18
ms.date: 03/04/2019
uid: security/authentication/facebook-logins
ms.openlocfilehash: f7b21de7e5fe9d77804588280c3d8be9df8afee5
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71082547"
---
# <a name="facebook-external-login-setup-in-aspnet-core"></a>ASP.NET Core での Facebook 外部ログインのセットアップ

作成者: [Valeriy Novytskyy](https://github.com/01binary)、[Rick Anderson](https://twitter.com/RickAndMSFT)

このチュートリアルでは、[前のページ](xref:security/authentication/social/index)で作成したサンプル ASP.NET Core 2.0 プロジェクトを使用して、ユーザーが Facebook アカウントでサインインできるようにする方法を示します。 まず Facebook アプリケーションの ID の作成を[公式手順](https://developers.facebook.com)します。

## <a name="create-the-app-in-facebook"></a>Facebook で、アプリを作成します。

* 移動し、 [Facebook Developers アプリ](https://developers.facebook.com/apps/)ページし、サインインします。 Facebook アカウントがない場合は、使用、 **Facebook にサインアップ**を作成するログイン ページのリンク。

* タップして、**新しいアプリの追加**新しいアプリ ID を作成する右上隅にあるボタンをクリックします。

   ![Microsoft Edge で開発者ポータルの Facebook を開く](index/_static/FBMyApps.png)

* フォームに入力し、タップ、 **Create App ID**ボタンをクリックします。

  ![アプリ ID の新しいフォームを作成します。](index/_static/FBNewAppId.png)

* **製品を選択**] ページで [**セットアップ**上、 **Facebook ログイン**カード。

  ![製品のセットアップ ページ](index/_static/FBProductSetup.png)

* **クイック スタート**使用してウィザードを起動**プラットフォームを選択して**最初のページとして。 ここでは、ウィザードをクリックしてバイパス、**設定**左側のメニューのリンク。

  ![スキップのクイック スタート](index/_static/FBSkipQuickStart.png)

* 表示されます、**クライアント OAuth 設定**ページ。

  ![クライアントの OAuth の設定 ページ](index/_static/FBOAuthSetup.png)

* 開発 URI を入力と */signin-facebook*に追加されます、**有効な OAuth リダイレクト Uri**フィールド (例: `https://localhost:44320/signin-facebook`)。 このチュートリアルの後半で構成されている Facebook 認証の要求では自動的に処理 */signin-facebook* OAuth フローを実装するルート。

> [!NOTE]
> URI */signin-facebook* Facebook の認証プロバイダーの既定のコールバックとして設定されます。 既定のコールバック URI を変更するには、継承を使用して、Facebook の認証ミドルウェアを構成するときに[RemoteAuthenticationOptions.CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath)のプロパティ、 [FacebookOptions](/dotnet/api/microsoft.aspnetcore.authentication.facebook.facebookoptions)クラス。

* クリックして**変更を保存**します。

* クリックして**設定** > **基本的な**左側のナビゲーション リンク。

  このページで、メモしてをおきます、`App ID`と`App Secret`します。 次のセクションでは、両方に、ASP.NET Core アプリケーションを追加します。

* サイトをデプロイするときに再アクセスする必要があります、 **Facebook ログイン**セットアップ ページと、新しいパブリック URI を登録します。

## <a name="store-facebook-app-id-and-app-secret"></a>Facebook アプリケーションの ID とアプリ シークレットを格納します。

Facebook などの機密設定をリンク`App ID`と`App Secret`アプリケーションの構成を使用して、 [Secret Manager](xref:security/app-secrets)します。 このチュートリアルの目的で、名前トークン`Authentication:Facebook:AppId`と`Authentication:Facebook:AppSecret`します。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

安全に保存するには、次のコマンドを実行`App ID`と`App Secret`シークレット マネージャーを使用します。

```dotnetcli
dotnet user-secrets set Authentication:Facebook:AppId <app-id>
dotnet user-secrets set Authentication:Facebook:AppSecret <app-secret>
```

## <a name="configure-facebook-authentication"></a>Facebook 認証を構成します。

::: moniker range=">= aspnetcore-2.0"

Facebook のサービスを追加、`ConfigureServices`メソッドで、 *Startup.cs*ファイル。

```csharp
services.AddDefaultIdentity<IdentityUser>()
        .AddDefaultUI(UIFramework.Bootstrap4)
        .AddEntityFrameworkStores<ApplicationDbContext>();

services.AddAuthentication().AddFacebook(facebookOptions =>
{
    facebookOptions.AppId = Configuration["Authentication:Facebook:AppId"];
    facebookOptions.AppSecret = Configuration["Authentication:Facebook:AppSecret"];
});
```

[!INCLUDE [default settings configuration](includes/default-settings.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

::: moniker-end

::: moniker range="< aspnetcore-2.0"

インストール、 [Microsoft.AspNetCore.Authentication.Facebook](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Facebook)パッケージ。

* Visual Studio 2017 では、このパッケージをインストールするには、クリックし、プロジェクトを右クリックし**NuGet パッケージの管理**します。
* で .NET Core CLI をインストールするには、プロジェクト ディレクトリで、次を実行します。

   `dotnet add package Microsoft.AspNetCore.Authentication.Facebook`

Facebook ミドルウェアを追加、`Configure`メソッド*Startup.cs*ファイル。

```csharp
app.UseFacebookAuthentication(new FacebookOptions()
{
    AppId = Configuration["Authentication:Facebook:AppId"],
    AppSecret = Configuration["Authentication:Facebook:AppSecret"]
});
```

::: moniker-end

参照してください、 [FacebookOptions](/dotnet/api/microsoft.aspnetcore.builder.facebookoptions) Facebook 認証でサポートされる構成オプションの詳細について、API リファレンス。 構成オプションを使用できます。

* ユーザーに関するさまざまな情報を要求します。
* ログイン エクスペリエンスをカスタマイズするクエリ文字列引数を追加します。

## <a name="sign-in-with-facebook"></a>Facebook のサインイン

アプリケーションを実行し、をクリックして**ログイン**します。 Facebook でサインインするオプションが表示されます。

![Web アプリケーション:認証されていないユーザー](index/_static/DoneFacebook.png)

クリックすると**Facebook**認証に Facebook にリダイレクトされます。

![Facebook 認証ページ](index/_static/FBLogin.png)

Facebook の認証は、既定では、パブリック プロファイルと電子メール アドレスを要求します。

![Facebook 認証ページの同意画面](index/_static/FBLoginDone.png)

Facebook の資格情報を入力すると、電子メールを設定するサイトにリダイレクトされます。

Facebook の資格情報を使用してログインしました。

![Web アプリケーション:認証されたユーザー](index/_static/Done.png)

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

## <a name="troubleshooting"></a>トラブルシューティング

* **ASP.NET Core 2.x のみ:** で`services.AddIdentity` *を呼び出すことによって id が構成されていない場合、認証を試みると ArgumentException が発生します。 `ConfigureServices`' SignInScheme ' オプションを指定*する必要があります。 このチュートリアルで使用するプロジェクト テンプレートによりこれが行われるようになります。
* 取得する場合は、初期移行を適用することで、サイト データベースが作成されていない*要求の処理中にデータベース操作が失敗しました*エラー。 タップ**適用移行**データベースを作成し、エラーを引き続き更新します。

## <a name="next-steps"></a>次の手順

* Facebook の高度な認証シナリオ用に、 [Microsoft.AspNetCore.Authentication.Facebook](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Facebook) NuGet パッケージをプロジェクトに追加します。 このパッケージは、Facebook 外部ログイン機能をアプリに統合するためには必要ありません。 

* この記事では、Facebook を認証する方法を示しました。 記載されているその他のプロバイダーで認証する同様のアプローチを利用できる、[前のページ](xref:security/authentication/social/index)します。

* リセットする必要があります、web サイトを Azure web アプリを発行すると、 `AppSecret` Facebook 開発者ポータルでします。

* 設定、`Authentication:Facebook:AppId`と`Authentication:Facebook:AppSecret`として、Azure portal でアプリケーションの設定。 構成システムは、環境変数からキーの読み取りを設定します。
