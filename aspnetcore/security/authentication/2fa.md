---
title: ASP.NET Core での SMS で 2 要素認証
author: rick-anderson
description: ASP.NET Core アプリで 2 要素認証 (2 fa) を設定する方法について説明します。
monikerRange: < aspnetcore-2.0
ms.author: riande
ms.date: 09/22/2018
ms.custom: mvc, seodec18
uid: security/authentication/2fa
ms.openlocfilehash: 68219579be9b7a7b25da6e348054e1ff2015cf5f
ms.sourcegitcommit: e54672f5c493258dc449fac5b98faf47eb123b28
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/24/2019
ms.locfileid: "71248388"
---
# <a name="two-factor-authentication-with-sms-in-aspnet-core"></a>ASP.NET Core での SMS で 2 要素認証

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)と[Swiss-Devs](https://github.com/Swiss-Devs)

>[!WARNING]
> 時間ベース ワンタイム パスワード アルゴリズム (TOTP) を使用して 2 要素認証 (2 fa) authenticator アプリは、業界の 2 fa のアプローチをお勧めします。 2 fa を SMS 2 fa を TOTP を使用しています。 詳細については、次を参照してください。 [TOTP 認証アプリが ASP.NET Core での QR コードを有効にする生成](xref:security/authentication/identity-enable-qrcodes)ASP.NET Core 2.0 以降。

このチュートリアルでは、SMS を使用して 2 要素認証 (2 fa) を設定する方法を示します。 手順が示されて[twilio](https://www.twilio.com/)と[ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/)、他の SMS プロバイダーを使用することができます。 完了したことをお勧めします。[アカウントの確認とパスワードの回復](xref:security/authentication/accconfirm)このチュートリアルを開始する前にします。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/2fa/sample/Web2FA)します。 [ダウンロードする方法](xref:index#how-to-download-a-sample)します。

## <a name="create-a-new-aspnet-core-project"></a>新しい ASP.NET Core プロジェクトを作成する

という名前の新しい ASP.NET Core web アプリ作成`Web2FA`個々 のユーザー アカウントを使用します。 「」の<xref:security/enforcing-ssl>手順に従って、HTTPS を設定し、要求します。

### <a name="create-an-sms-account"></a>SMS アカウントを作成します。

例については、SMS アカウントの作成[twilio](https://www.twilio.com/)または[ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/)します。 認証資格情報を記録します (twilio: accountSid および authToken の場合は、ASPSMS の場合:ユーザーキーとパスワード) を入力します。

#### <a name="figuring-out-sms-provider-credentials"></a>SMS プロバイダーの資格情報を見極める

**Twilio**

Twilio アカウントの [ダッシュボード] タブで、**アカウント SID**と**認証トークン**をコピーします。

**ASPSMS:**

アカウントの設定から**ユーザーキー**に移動し、**パスワード**と共にコピーします。

キー内 secret manager ツールを使用してこれらの値を後で保存`SMSAccountIdentification`と`SMSAccountPassword`します。

#### <a name="specifying-senderid--originator"></a>SenderID を指定する/発信元

**Twilio**[数値] タブで、Twilio**電話番号**をコピーします。

**ASPSMS:** [発信元のロック解除] メニュー内で、1つ以上の発信者をロック解除するか、英数字の発信者を選択します (すべてのネットワークでサポートされていません)。

キー内のシークレット マネージャー ツールを使用してこの値を後で保存`SMSAccountFrom`します。

### <a name="provide-credentials-for-the-sms-service"></a>SMS サービスの資格情報を提供します。

使用して、[オプション パターン](xref:fundamentals/configuration/options)ユーザー アカウントとキーの設定にアクセスします。

* セキュリティで保護された SMS キーを取得するためのクラスを作成します。 このサンプルで、`SMSoptions`でクラスを作成、 *Services/SMSoptions.cs*ファイル。

[!code-csharp[](2fa/sample/Web2FA/Services/SMSoptions.cs)]

設定、 `SMSAccountIdentification`、`SMSAccountPassword`と`SMSAccountFrom`で、 [secret manager ツール](xref:security/app-secrets)します。 例:

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```

* SMS プロバイダーの NuGet パッケージを追加します。 パッケージ マネージャー コンソール (PMC) を実行します。

**Twilio**

`Install-Package Twilio`

**ASPSMS:**

`Install-Package ASPSMS`

* 内のコードを追加、 *Services/MessageServices.cs* SMS が有効にするファイル。 Twilio または ASPSMS セクションのいずれかを使用します。

**Twilio**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

**ASPSMS:**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a>使用するスタートアップを構成します。 `SMSoptions`

追加`SMSoptions`でサービス コンテナーに、`ConfigureServices`メソッドで、 *Startup.cs*:

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a>2 要素認証を有効にします。

*Views/Manage/Index. cshtml* Razor ビューファイルを開き、コメント文字を削除します (マークアップがコメントアウトされることはありません)。

## <a name="log-in-with-two-factor-authentication"></a>2 要素認証をログインします。

* アプリを実行し、新しいユーザーの登録

![Web アプリケーションの登録が Microsoft Edge でビューを開く](2fa/_static/login2fa1.png)

* アクティブ化するユーザー名をタップして、`Index`管理コント ローラー アクション メソッド。 電話番号の順にタップ**追加**リンク。

![ビューの管理 -"の追加 リンクをタップします。](2fa/_static/login2fa2.png)

* 確認コードを受信してタップ、する電話番号を追加**確認コードを送信**します。

![ページの 電話番号を追加します。](2fa/_static/login2fa3.png)

* 確認コードを含むテキスト メッセージが表示されます。 入力し、タップ**送信**

![ページの 電話番号を確認します。](2fa/_static/login2fa4.png)

テキスト メッセージが表示されない場合は、twilio ログ ページを参照してください。

* 電話番号が正常に追加された、管理ビューを示しています。

![ビュー - 電話番号が正常に追加の管理します。](2fa/_static/login2fa5.png)

* タップ**を有効にする**2 要素認証を有効にします。

![ビューの管理 - 2 要素認証を有効にします。](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a>2 要素認証のテスト

* ログオフします。

* ログイン。

* 認証の 2 番目の要素を提供する必要があるために、ユーザー アカウントは、2 要素認証を有効になります。 このチュートリアルでは、電話番号の検証を有効にします。 組み込みのテンプレートでは、2 番目の要素として電子メールを設定することもできます。 QR コードなど、認証の他の 2 番目の要素を設定することができます。 タップ**送信**します。

![ビューの確認コードを送信します。](2fa/_static/login2fa7.png)

* SMS メッセージを取得するコードを入力します。

* クリックすると、**このブラウザーを記憶する** チェック ボックスの除外 2 fa を使用して、同じデバイスとブラウザーを使用するときにログオンする必要がなくなります。 2 fa を有効にし、**このブラウザーを記憶する**は 2 fa の強力な保護実現デバイスへのアクセスがあるない限り、アカウントにアクセスしようとした悪意のあるユーザーから。 これは、定期的に使用するプライベートあらゆるデバイスで行うことができます。 設定して**このブラウザーを記憶する**2 fa の追加のセキュリティを定期的に使用しないデバイスから取得した、独自のデバイスで 2 fa を経由する必要があるの利便性を取得します。

![ビューを確認します。](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a>ブルート フォース攻撃から保護するためのアカウントのロックアウト

2 fa では、アカウントのロックアウトをお勧めします。 ユーザーがローカル アカウントまたはソーシャル アカウントでサインインすると、2 fa に失敗した場合は各が格納されます。 失敗したアクセス試行の最大回数に達した場合、ユーザーはロックアウトされます (既定:5分間のロックアウトは5分間のアクセス試行に失敗しました)。 成功した認証は失敗したアクセス試行数がリセットされ、時計をリセットします。 失敗したアクセス試行の最大値とロックアウト時刻で設定できる[MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts)と[DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan)します。 次で 10 アクセス試行の失敗後 10 分間のアカウントのロックアウトが構成されます。

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)]

確認します[PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync)設定`lockoutOnFailure`に`true`:

```csharp
var result = await _signInManager.PasswordSignInAsync(
                 Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
```
