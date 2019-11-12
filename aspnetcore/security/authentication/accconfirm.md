---
title: ASP.NET Core でのアカウントの確認とパスワードの回復
author: rick-anderson
description: 電子メールの確認とパスワードのリセットを使用して ASP.NET Core アプリを構築する方法について説明します。
ms.author: riande
ms.date: 03/11/2019
uid: security/authentication/accconfirm
ms.openlocfilehash: 8a515990be584aa1233fc3bf77811ae3784d9b1c
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081558"
---
# <a name="account-confirmation-and-password-recovery-in-aspnet-core"></a>ASP.NET Core でのアカウントの確認とパスワードの回復

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、 [Ponant](https://github.com/Ponant)、 [Joe Audette](https://twitter.com/joeaudette)

このチュートリアルでは、電子メールの確認とパスワードのリセットを使用して ASP.NET Core アプリを構築する方法について説明します。 このチュートリアルは最初のトピックでは**ありません**。 理解しておく必要があります。

* [ASP.NET Core](xref:tutorials/razor-pages/razor-pages-start)
* [認証](xref:security/authentication/identity)
* [Entity Framework Core](xref:data/ef-mvc/intro)

<!-- see C:/Dropbox/wrk/Code/SendGridConsole/Program.cs -->

::: moniker range="<= aspnetcore-2.0"

ASP.NET Core 1.1 バージョンについては、[この PDF ファイル](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_1-16-18.pdf)を参照してください。

::: moniker-end

::: moniker range="> aspnetcore-2.2"

## <a name="prerequisites"></a>必須コンポーネント

[.NET Core 3.0 SDK 以降](https://dotnet.microsoft.com/download/dotnet-core/3.0)

## <a name="create-and-test-a-web-app-with-authentication"></a>認証を使用した web アプリの作成とテスト

認証を使用して web アプリを作成するには、次のコマンドを実行します。

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet run
```

アプリを実行し、 **[登録]** リンクを選択して、ユーザーを登録します。 登録されると、電子メールの確認`/Identity/Account/RegisterConfirmation`をシミュレートするためのリンクを含む [to] ページにリダイレクトされます。

* `Click here to confirm your account`リンクを選択します。
* **ログイン**リンクを選択し、同じ資格情報でサインインします。
* リンクを選択すると、 `/Identity/Account/Manage/PersonalData`ページにリダイレクトされます。 `Hello YourEmail@provider.com!`
* 左側の **[Personal data]** タブを選択し、 **[削除]** を選択します。

### <a name="configure-an-email-provider"></a>電子メールプロバイダーを構成する

このチュートリアルでは、 [Sendgrid](https://sendgrid.com)を使用して電子メールを送信します。 電子メールを送信するには、SendGrid アカウントとキーが必要です。 他の電子メールプロバイダーを使用することもできます。 SendGrid または別の電子メールサービスを使用して電子メールを送信することをお勧めします。 SMTP のセキュリティを保護し、正しく設定することは困難です。

セキュリティで保護された電子メールキーを取得するクラスを作成します。 このサンプルでは、*サービス/認証の Enderoptions を作成します。*

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>SendGrid ユーザーシークレットの構成

`SendGridUser` と`SendGridKey`を、[シークレットマネージャーツール](xref:security/app-secrets)を使用して設定します。 例:

```dotnetcli
dotnet user-secrets set SendGridUser RickAndMSFT
dotnet user-secrets set SendGridKey <key>

Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

Windows では、シークレットマネージャーはキーと値のペアを*シークレットの json*ファイル`%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>`に格納します。

*シークレットの json*ファイルの内容は暗号化されていません。 次のマークアップは、*シークレットの json*ファイルを示しています。 `SendGridKey`値は削除されています。

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

詳細については、「パターンと[構成](xref:fundamentals/configuration/index)の[オプション](xref:fundamentals/configuration/options)」を参照してください。

### <a name="install-sendgrid"></a>SendGrid のインストール

このチュートリアルでは、 [Sendgrid](https://sendgrid.com/)を使用して電子メール通知を追加する方法について説明しますが、SMTP などのメカニズムを使用して電子メールを送信することもできます。

NuGet パッケージ`SendGrid`をインストールします。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

パッケージマネージャーコンソールで、次のコマンドを入力します。

```powershell
Install-Package SendGrid
```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コンソールで、次のコマンドを入力します。

```dotnetcli
dotnet add package SendGrid
```

---

無料の sendgrid アカウントを登録するには、「 [SendGrid を無料で開始](https://sendgrid.com/free/)する」を参照してください。

### <a name="implement-iemailsender"></a>IEmailSender を実装する

を実装`IEmailSender`するには、次のようなコードを使用して*サービス/emailsender*を作成します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>電子メールをサポートするスタートアップの構成

`ConfigureServices` *Startup.cs*ファイルのメソッドに次のコードを追加します。

* を`EmailSender`一時サービスとして追加します。
* 構成インスタンス`AuthMessageSenderOptions`を登録します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/Startup.cs?name=snippet1&highlight=11-15)]

## <a name="register-confirm-email-and-reset-password"></a>登録、電子メールの確認、およびパスワードのリセット

Web アプリを実行し、アカウントの確認とパスワードの回復フローをテストします。

* アプリを実行し、新しいユーザーの登録
* アカウントの確認リンクについては、電子メールを確認してください。 電子メールを受信しない場合は、「[デバッグ電子メール](#debug)」を参照してください。
* リンクをクリックして、電子メールを確認します。
* 電子メールとパスワードを使用してサインインします。
* サインアウトします。

### <a name="test-password-reset"></a>パスワードリセットのテスト

* サインインしている場合は、 **[ログアウト]** を選択します。
* **[ログイン]** リンクを選択し、 **[パスワードを忘れ]** た場合 リンクを選択します。
* アカウントの登録に使用した電子メールを入力します。
* パスワードをリセットするためのリンクを含む電子メールが送信されます。 メールを確認し、リンクをクリックしてパスワードをリセットします。 パスワードが正常にリセットされたら、電子メールと新しいパスワードでサインインできます。

## <a name="change-email-and-activity-timeout"></a>電子メールとアクティビティのタイムアウトを変更する

既定の非アクティブタイムアウトは14日です。 次のコードは、非アクティブタイムアウトを5日間に設定します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>すべてのデータ保護トークンの lifespans を変更する

次のコードでは、すべてのデータ保護トークンのタイムアウト期間を3時間に変更します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAllTokens.cs?name=snippet1&highlight=11-12)]

組み込みの Id ユーザートークン (「 [AspNetCore/src/Identity/Extensions. Core/src/TokenOptions .cs](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) 」を参照) には、 [1 日のタイムアウト](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)があります。

### <a name="change-the-email-token-lifespan"></a>電子メールトークンの有効期間を変更する

[Id ユーザートークン](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs)の既定のトークン有効期間は[1 日](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)です。 このセクションでは、電子メールトークンの有効期間を変更する方法について説明します。

カスタム[DataProtectorTokenProvider\<tuser >](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1)およびを追加<xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

カスタムプロバイダーをサービスコンテナーに追加します。

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupEmail.cs?name=snippet1&highlight=10-16)]

### <a name="resend-email-confirmation"></a>電子メールの再送信の確認

[こちらの GitHub のイシュー](https://github.com/aspnet/AspNetCore/issues/5410)を参照してください。

<a name="debug"></a>

### <a name="debug-email"></a>デバッグ用電子メール

電子メールが機能しない場合:

* にブレークポイント`EmailSender.Execute`を設定し`SendGridClient.SendEmailAsync`て、が呼び出されることを確認します。
* 同様のコード`EmailSender.Execute`を使用して[電子メールを送信するコンソールアプリ](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html)を作成します。
* [[電子メール活動](https://sendgrid.com/docs/User_Guide/email_activity.html)] ページを確認します。
* 迷惑メールフォルダーを確認します。
* 別の電子メールプロバイダー (Microsoft、Yahoo、Gmail など) で別の電子メールエイリアスを試してください。
* 別の電子メールアカウントに送信してみてください。

**セキュリティのベストプラクティス**として、テストと開発では運用シークレットを使用し**ないこと**をお勧めします。 アプリを Azure に発行する場合は、Azure Web アプリポータルで SendGrid シークレットをアプリケーション設定として設定します。 構成システムは、環境変数からキーの読み取りを設定します。

## <a name="combine-social-and-local-login-accounts"></a>ソーシャルおよびローカルログインアカウントの結合

このセクションを完了するには、まず外部認証プロバイダーを有効にする必要があります。 「 [Facebook、Google、および外部プロバイダーの認証](xref:security/authentication/social/index)」を参照してください。

電子メールリンクをクリックして、ローカルアカウントとソーシャルアカウントを組み合わせることができます。 次のシーケンスでは、RickAndMSFT@gmail.com"" は最初にローカルログインとして作成されますが、最初にアカウントをソーシャルログインとして作成してから、ローカルログインを追加することができます。

![Web アプリケーション: RickAndMSFT@gmail.comユーザー認証済み](accconfirm/_static/rick.png)

**[管理]** リンクをクリックします。 このアカウントに関連付けられている0外部 (ソーシャルログイン) に注意してください。

![ビューの管理](accconfirm/_static/manage.png)

別のログインサービスへのリンクをクリックし、アプリの要求を受け入れます。 次の図では、Facebook が外部認証プロバイダーです。

![外部ログインビューを管理する Facebook の一覧表示](accconfirm/_static/fb.png)

2つのアカウントが結合されています。 いずれかのアカウントでサインインできます。 ソーシャルログイン認証サービスがダウンした場合、またはソーシャルアカウントへのアクセスが失われた可能性がある場合は、ユーザーにローカルアカウントを追加することができます。

## <a name="enable-account-confirmation-after-a-site-has-users"></a>サイトにユーザーがいる場合のアカウントの確認を有効にする

ユーザーがサイトでアカウントの確認を有効にすると、すべての既存のユーザーがロックアウトされます。 アカウントが確認されていないため、既存のユーザーがロックアウトされています。 既存のユーザーロックアウトを回避するには、次のいずれかの方法を使用します。

* データベースを更新して、すべての既存のユーザーを確認済みとしてマークします。
* 既存のユーザーを確認します。 たとえば、確認リンクを含む電子メールをバッチ送信します。

::: moniker-end

::: moniker range="> aspnetcore-2.0 < aspnetcore-3.0"

## <a name="prerequisites"></a>必須コンポーネント

[.NET Core 2.2 SDK 以降](https://www.microsoft.com/net/download/all)

## <a name="create-a-web--app-and-scaffold-identity"></a>Web アプリとスキャフォールディング Id を作成する

認証を使用して web アプリを作成するには、次のコマンドを実行します。

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator identity -dc WebPWrecover.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.ConfirmEmail"
dotnet ef database drop -f
dotnet ef database update
dotnet run

```

## <a name="test-new-user-registration"></a>新しいユーザー登録をテストする

アプリを実行し、 **[登録]** リンクを選択して、ユーザーを登録します。 この時点で、電子メールの検証は、 [[EmailAddress]](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute)属性でのみ行うことができます。 登録を送信すると、アプリにログインします。 このチュートリアルの後半では、電子メールが検証されるまで新しいユーザーがサインインできないように、コードが更新されています。

[!INCLUDE[](~/includes/view-identity-db.md)]

テーブルの`EmailConfirmed`フィールドがである`False`ことに注意してください。

アプリが確認の電子メールを送信するときに、次の手順でもう一度このメールを使用することもできます。 行を右クリックし、 **[削除]** をクリックします。 電子メールエイリアスを削除すると、次の手順が簡単になります。

<a name="prevent-login-at-registration"></a>

## <a name="require-email-confirmation"></a>電子メールの確認が必要

新しいユーザー登録の電子メールを確認することをお勧めします。 電子メールの確認では、他のユーザーの権限を借用していないこと (つまり、他のユーザーの電子メールに登録されていないこと) を確認できます。 ディスカッションフォーラムを使用していて、"yli@example.com" を ""nolivetto@contoso.comとして登録できないようにしたいとします。 電子メールを確認しnolivetto@contoso.comない場合、"" はアプリから不要な電子メールを受信する可能性があります。 ユーザーが誤って "ylo@example.com" として登録され、"yli" のスペルミスに気付かないとします。 アプリには正しい電子メールがないため、パスワードの回復を使用できません。 電子メールの確認では、ボットからの保護が制限されています。 電子メールの確認では、多くの電子メールアカウントを持つ悪意のあるユーザーからの保護は提供されません。

通常は、確認済みの電子メールを送信する前に、新しいユーザーが web サイトにデータを投稿できないようにします。

確認`Startup.ConfigureServices`済みの電子メールを要求するように更新します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=8-11)]

`config.SignIn.RequireConfirmedEmail = true;`登録されたユーザーが電子メールを確認するまでログインできないようにします。

### <a name="configure-email-provider"></a>電子メールプロバイダーの構成

このチュートリアルでは、 [Sendgrid](https://sendgrid.com)を使用して電子メールを送信します。 電子メールを送信するには、SendGrid アカウントとキーが必要です。 他の電子メールプロバイダーを使用することもできます。 ASP.NET Core 2.x には、 `System.Net.Mail`アプリから電子メールを送信できるようにするが含まれています。 SendGrid または別の電子メールサービスを使用して電子メールを送信することをお勧めします。 SMTP のセキュリティを保護し、正しく設定することは困難です。

セキュリティで保護された電子メールキーを取得するクラスを作成します。 このサンプルでは、*サービス/認証の Enderoptions を作成します。*

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>SendGrid ユーザーシークレットの構成

`SendGridUser` と`SendGridKey`を、[シークレットマネージャーツール](xref:security/app-secrets)を使用して設定します。 例:

```console
C:/WebAppl>dotnet user-secrets set SendGridUser RickAndMSFT
info: Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

Windows では、シークレットマネージャーはキーと値のペアを*シークレットの json*ファイル`%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>`に格納します。

*シークレットの json*ファイルの内容は暗号化されていません。 次のマークアップは、*シークレットの json*ファイルを示しています。 `SendGridKey`値は削除されています。

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

詳細については、「パターンと[構成](xref:fundamentals/configuration/index)の[オプション](xref:fundamentals/configuration/options)」を参照してください。

### <a name="install-sendgrid"></a>SendGrid のインストール

このチュートリアルでは、 [Sendgrid](https://sendgrid.com/)を使用して電子メール通知を追加する方法について説明しますが、SMTP などのメカニズムを使用して電子メールを送信することもできます。

NuGet パッケージ`SendGrid`をインストールします。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

パッケージマネージャーコンソールで、次のコマンドを入力します。

```powershell
Install-Package SendGrid
```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

コンソールで、次のコマンドを入力します。

```dotnetcli
dotnet add package SendGrid
```

---

無料の sendgrid アカウントを登録するには、「 [SendGrid を無料で開始](https://sendgrid.com/free/)する」を参照してください。

### <a name="implement-iemailsender"></a>IEmailSender を実装する

を実装`IEmailSender`するには、次のようなコードを使用して*サービス/emailsender*を作成します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>電子メールをサポートするスタートアップの構成

`ConfigureServices` *Startup.cs*ファイルのメソッドに次のコードを追加します。

* を`EmailSender`一時サービスとして追加します。
* 構成インスタンス`AuthMessageSenderOptions`を登録します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=15-99)]

## <a name="enable-account-confirmation-and-password-recovery"></a>アカウントの確認とパスワードの回復を有効にする

このテンプレートには、アカウントの確認とパスワードの回復のためのコードが含まれています。 Areas/ `OnPostAsync` *Identity/Pages/Account/Register. cshtml. .cs*でメソッドを見つけます。

新しく登録されたユーザーが自動的にサインインしないようにするには、次の行をコメントアウトします。

```csharp
await _signInManager.SignInAsync(user, isPersistent: false);
```

完全なメソッドは、変更された行が強調表示された状態で表示されます。

[!code-csharp[](accconfirm/sample/WebPWrecover22/Areas/Identity/Pages/Account/Register.cshtml.cs?highlight=22&name=snippet_Register)]

## <a name="register-confirm-email-and-reset-password"></a>登録、電子メールの確認、およびパスワードのリセット

Web アプリを実行し、アカウントの確認とパスワードの回復フローをテストします。

* アプリを実行し、新しいユーザーの登録
* アカウントの確認リンクについては、電子メールを確認してください。 電子メールを受信しない場合は、「[デバッグ電子メール](#debug)」を参照してください。
* リンクをクリックして、電子メールを確認します。
* 電子メールとパスワードを使用してサインインします。
* サインアウトします。

### <a name="view-the-manage-page"></a>[管理] ページを表示する

ブラウザーでユーザー名を選択します![: ユーザー名が表示されたブラウザーウィンドウ](accconfirm/_static/un.png)

管理 ページが、**プロファイル** タブが選択された状態で表示されます。 電子メールには、電子メールが確認されたことを示すチェックボックス**が表示さ**れます。

### <a name="test-password-reset"></a>パスワードリセットのテスト

* サインインしている場合は、 **[ログアウト]** を選択します。
* **[ログイン]** リンクを選択し、 **[パスワードを忘れ]** た場合 リンクを選択します。
* アカウントの登録に使用した電子メールを入力します。
* パスワードをリセットするためのリンクを含む電子メールが送信されます。 メールを確認し、リンクをクリックしてパスワードをリセットします。 パスワードが正常にリセットされたら、電子メールと新しいパスワードでサインインできます。

## <a name="change-email-and-activity-timeout"></a>電子メールとアクティビティのタイムアウトを変更する

既定の非アクティブタイムアウトは14日です。 次のコードは、非アクティブタイムアウトを5日間に設定します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>すべてのデータ保護トークンの lifespans を変更する

次のコードでは、すべてのデータ保護トークンのタイムアウト期間を3時間に変更します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAllTokens.cs?name=snippet1&highlight=15-16)]

組み込みの Id ユーザートークン (「 [AspNetCore/src/Identity/Extensions. Core/src/TokenOptions .cs](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) 」を参照) には、 [1 日のタイムアウト](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)があります。

### <a name="change-the-email-token-lifespan"></a>電子メールトークンの有効期間を変更する

[Id ユーザートークン](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs)の既定のトークン有効期間は[1 日](https://github.com/aspnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)です。 このセクションでは、電子メールトークンの有効期間を変更する方法について説明します。

カスタム[DataProtectorTokenProvider\<tuser >](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1)およびを追加<xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions>します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

カスタムプロバイダーをサービスコンテナーに追加します。

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupEmail.cs?name=snippet1&highlight=10-13,18)]

### <a name="resend-email-confirmation"></a>電子メールの再送信の確認

[こちらの GitHub のイシュー](https://github.com/aspnet/AspNetCore/issues/5410)を参照してください。

<a name="debug"></a>

### <a name="debug-email"></a>デバッグ用電子メール

電子メールが機能しない場合:

* にブレークポイント`EmailSender.Execute`を設定し`SendGridClient.SendEmailAsync`て、が呼び出されることを確認します。
* 同様のコード`EmailSender.Execute`を使用して[電子メールを送信するコンソールアプリ](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html)を作成します。
* [[電子メール活動](https://sendgrid.com/docs/User_Guide/email_activity.html)] ページを確認します。
* 迷惑メールフォルダーを確認します。
* 別の電子メールプロバイダー (Microsoft、Yahoo、Gmail など) で別の電子メールエイリアスを試してください。
* 別の電子メールアカウントに送信してみてください。

**セキュリティのベストプラクティス**として、テストと開発では運用シークレットを使用し**ないこと**をお勧めします。 アプリを Azure に発行する場合は、Azure Web アプリポータルで SendGrid シークレットをアプリケーション設定として設定できます。 構成システムは、環境変数からキーの読み取りを設定します。

## <a name="combine-social-and-local-login-accounts"></a>ソーシャルおよびローカルログインアカウントの結合

このセクションを完了するには、まず外部認証プロバイダーを有効にする必要があります。 「 [Facebook、Google、および外部プロバイダーの認証](xref:security/authentication/social/index)」を参照してください。

電子メールリンクをクリックして、ローカルアカウントとソーシャルアカウントを組み合わせることができます。 次のシーケンスでは、RickAndMSFT@gmail.com"" は最初にローカルログインとして作成されますが、最初にアカウントをソーシャルログインとして作成してから、ローカルログインを追加することができます。

![Web アプリケーション: RickAndMSFT@gmail.comユーザー認証済み](accconfirm/_static/rick.png)

**[管理]** リンクをクリックします。 このアカウントに関連付けられている0外部 (ソーシャルログイン) に注意してください。

![ビューの管理](accconfirm/_static/manage.png)

別のログインサービスへのリンクをクリックし、アプリの要求を受け入れます。 次の図では、Facebook が外部認証プロバイダーです。

![外部ログインビューを管理する Facebook の一覧表示](accconfirm/_static/fb.png)

2つのアカウントが結合されています。 いずれかのアカウントでサインインできます。 ソーシャルログイン認証サービスがダウンした場合、またはソーシャルアカウントへのアクセスが失われた可能性がある場合は、ユーザーにローカルアカウントを追加することができます。

## <a name="enable-account-confirmation-after-a-site-has-users"></a>サイトにユーザーがいる場合のアカウントの確認を有効にする

ユーザーがサイトでアカウントの確認を有効にすると、すべての既存のユーザーがロックアウトされます。 アカウントが確認されていないため、既存のユーザーがロックアウトされています。 既存のユーザーロックアウトを回避するには、次のいずれかの方法を使用します。

* データベースを更新して、すべての既存のユーザーを確認済みとしてマークします。
* 既存のユーザーを確認します。 たとえば、確認リンクを含む電子メールをバッチ送信します。

::: moniker-end
