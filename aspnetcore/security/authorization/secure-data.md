---
title: 承認によって保護されたユーザー データと ASP.NET Core アプリを作成します。
author: rick-anderson
description: 承認によって保護されたユーザー データと Razor ページ アプリを作成する方法について説明します。 HTTPS、認証、セキュリティ、ASP.NET Core Identity が含まれています。
ms.author: riande
ms.date: 12/18/2018
ms.custom: mvc, seodec18
uid: security/authorization/secure-data
ms.openlocfilehash: 65c72d4dd457f85451796c5713bedebafec7a7de
ms.sourcegitcommit: 8157e5a351f49aeef3769f7d38b787b4386aad5f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/20/2019
ms.locfileid: "74239834"
---
# <a name="create-an-aspnet-core-app-with-user-data-protected-by-authorization"></a>承認によって保護されたユーザー データと ASP.NET Core アプリを作成します。

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Joe Audette](https://twitter.com/joeaudette)

::: moniker range="<= aspnetcore-1.1"

ASP.NET Core MVC バージョンについては、[この PDF](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_1-16-18.pdf)を参照してください。 このチュートリアルの ASP.NET Core 1.1 バージョンは、[この](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data)フォルダーにあります。 1\.1 ASP.NET Core サンプルは、「」[のサンプルに](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/final2)含まれています。

::: moniker-end

::: moniker range="= aspnetcore-2.0"

[次の pdf](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_July16_18.pdf)を参照

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

このチュートリアルでは、承認によって保護されたユーザー データと ASP.NET Core web アプリを作成する方法を示します。 認証済み (登録済み) のユーザーの連絡先の一覧を表示を作成します。 これには 3 つのセキュリティ グループがあります。

* **登録されているユーザー**は、すべての承認済みデータを表示したり、自分のデータを編集または削除したりできます。
* **管理者**は、連絡先データを承認または拒否できます。 承認されたメンバーのみがユーザーに表示されます。
* **管理者**は、任意のデータを承認/拒否したり、編集/削除したりできます。

このドキュメントの画像は、最新のテンプレートと完全には一致しません。

次の図では、user Rick (`rick@example.com`) がサインインしています。 Rick は、承認された連絡先を表示して**編集**/**削除**/、連絡先の新しいリンクを**作成**することのみができます。 Rick によって作成された最後のレコードにのみ、**編集**および**削除**のリンクが表示されます。 他のユーザーは、管理者は、状態を"Approved"に変わるまで、最後のレコードを表示されません。

![サインインして Rick を示すスクリーン ショット](secure-data/_static/rick.png)

次の図では、`manager@contoso.com` がサインインし、マネージャーのロールに含まれています。

![サインイン manager@contoso.com を示すスクリーンショット](secure-data/_static/manager1.png)

次の図は、連絡先の詳細ビューをマネージャーには。

![連絡先のマネージャーの表示](secure-data/_static/manager.png)

**[承認]** ボタンと **[却下]** ボタンは、マネージャーと管理者に対してのみ表示されます。

次の図では、`admin@contoso.com` がサインインし、管理者のロールに含まれています。

![サインイン admin@contoso.com を示すスクリーンショット](secure-data/_static/admin.png)

管理者は、すべての特権を持ちます。 彼女は、読み取り/編集/削除のいずれかの連絡先をことができ、連絡先の状態を変更します。

アプリは、次の `Contact` モデルを[スキャフォールディング](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model)することによって作成されました。

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

サンプルには、次の承認ハンドラーが含まれています。

* `ContactIsOwnerAuthorizationHandler`: ユーザーがデータを編集できるようになります。
* `ContactManagerAuthorizationHandler`: 管理者が連絡先を承認または拒否できるようにします。
* `ContactAdministratorsAuthorizationHandler`: 管理者は、連絡先を承認または拒否したり、連絡先を編集または削除したりできます。

## <a name="prerequisites"></a>前提条件

このチュートリアルは、高度なです。 理解しておく必要があります。

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [認証](xref:security/authentication/identity)
* [アカウントの確認とパスワードの回復](xref:security/authentication/accconfirm)
* [承認](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>Starter および完成したアプリ

完成し[た](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples)アプリを[ダウンロード](xref:index#how-to-download-a-sample)します。 完成したアプリを[テスト](#test-the-completed-app)して、セキュリティ機能に慣れるようにします。

### <a name="the-starter-app"></a>スターター アプリ

[スターター](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/)アプリを[ダウンロード](xref:index#how-to-download-a-sample)します。

アプリを実行し、 **[Contactmanager]** リンクをタップして、連絡先を作成、編集、および削除できることを確認します。

## <a name="secure-user-data"></a>セキュリティで保護されたユーザー データ

次のセクションでは、セキュリティで保護されたユーザー データ アプリを作成するすべての主要な手順があります。 完成したプロジェクトを参照すると便利です。

### <a name="tie-the-contact-data-to-the-user"></a>ユーザーに連絡先データを関連付ける

ASP.NET [id](xref:security/authentication/identity)ユーザー id を使用すると、ユーザーがデータを編集できるようになりますが、他のユーザーデータは編集できません。 `OwnerID` と `ContactStatus` を `Contact` モデルに追加します。

[!code-csharp[](secure-data/samples/final3/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` は、 [id](xref:security/authentication/identity)データベースの `AspNetUser` テーブルのユーザー ID です。 [`Status`] フィールドによって、連絡先が一般ユーザーに表示されるかどうかが決まります。

新しい移行を作成し、データベースを更新します。

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a>役割サービスの Id を追加します。

役割サービスを追加するには、 [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1)を追加します。

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet2&highlight=9)]

### <a name="require-authenticated-users"></a>認証されたユーザーが必要です。

ユーザー認証を必要とする既定の認証ポリシーを設定します。

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet&highlight=15-99)] 

 Razor ページ、コントローラー、またはアクションメソッドレベルで、`[AllowAnonymous]` 属性を使用して認証をオプトアウトできます。 ユーザー認証を必要とする既定の認証ポリシーの設定は、新しく追加された Razor ページとコント ローラーを保護します。 既定で認証が必要になるのは、新しいコントローラーに依存していて、Razor Pages `[Authorize]` 属性を含めるよりも安全です。

匿名ユーザーが登録前にサイトに関する情報を取得できるように、 [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute)をインデックスおよびプライバシーページに追加します。

[!code-csharp[](secure-data/samples/final3/Pages/Index.cshtml.cs?highlight=1,7)]

### <a name="configure-the-test-account"></a>テスト アカウントを構成します。

`SeedData` クラスは、管理者とマネージャーの2つのアカウントを作成します。 これらのアカウントのパスワードを設定するには、[シークレットマネージャーツール](xref:security/app-secrets)を使用します。 プロジェクトディレクトリ ( *Program.cs*を含むディレクトリ) のパスワードを設定します。

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

強力なパスワードが指定されていない場合、`SeedData.Initialize` が呼び出されると例外がスローされます。

テストパスワードを使用するように `Main` を更新します。

[!code-csharp[](secure-data/samples/final3/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>テスト アカウントを作成し、連絡先の更新

`SeedData` クラスの `Initialize` メソッドを更新して、テストアカウントを作成します。

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet_Initialize)]

管理者のユーザー ID と `ContactStatus` を連絡先に追加します。 「送信済み」と「拒否」の 1 つの連絡先を作成します。 すべての連絡先に、ユーザー ID と状態を追加します。 1 人だけが表示されます。

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>所有者、マネージャー、および管理者の承認ハンドラーを作成します。

*承認*フォルダーに `ContactIsOwnerAuthorizationHandler` クラスを作成します。 `ContactIsOwnerAuthorizationHandler` は、リソースに対して動作しているユーザーがリソースを所有していることを確認します。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler` は[コンテキストを呼び出します。](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_)現在の認証済みユーザーが連絡先の所有者である場合は成功します。 承認ハンドラー通常。

* 要件が満たされた場合に `context.Succeed` を返します。
* 要件を満たしていない場合は `Task.CompletedTask` を返します。 `Task.CompletedTask` は成功でも失敗でも&mdash;他の承認ハンドラーの実行を許可します。

明示的に失敗する必要がある場合は、コンテキストを返し[ます。失敗](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail)。

アプリは、独自のデータを編集/削除/作成所有者が連絡先にできます。 `ContactIsOwnerAuthorizationHandler` は、要件パラメーターで渡された操作を確認する必要はありません。

### <a name="create-a-manager-authorization-handler"></a>マネージャーの承認ハンドラーを作成します。

*承認*フォルダーに `ContactManagerAuthorizationHandler` クラスを作成します。 `ContactManagerAuthorizationHandler` は、リソースに対して動作しているユーザーがマネージャーであることを確認します。 管理者だけでは、承認したり、(新規または変更された) コンテンツの変更を拒否することができます。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>管理者の承認ハンドラーを作成します。

*承認*フォルダーに `ContactAdministratorsAuthorizationHandler` クラスを作成します。 `ContactAdministratorsAuthorizationHandler` は、リソースに対して動作しているユーザーが管理者であることを確認します。 管理者は、すべての操作を実行できます。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>承認ハンドラーを登録します。

Entity Framework Core を使用するサービスは、 [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions)を使用して[依存関係の挿入](xref:fundamentals/dependency-injection)に登録する必要があります。 `ContactIsOwnerAuthorizationHandler` は Entity Framework Core 上に構築された ASP.NET Core [id](xref:security/authentication/identity)を使用します。 サービスコレクションにハンドラーを登録して、[依存関係の挿入](xref:fundamentals/dependency-injection)によって `ContactsController` で使用できるようにします。 `ConfigureServices`の末尾に次のコードを追加します。

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet_defaultPolicy&highlight=23-99)]

`ContactAdministratorsAuthorizationHandler` と `ContactManagerAuthorizationHandler` はシングルトンとして追加されます。 EF を使用せず、必要なすべての情報が `HandleRequirementAsync` メソッドの `Context` パラメーターに含まれているため、これらはシングルトンです。

## <a name="support-authorization"></a>承認をサポートします。

このセクションでは、Razor ページを更新し、操作の要件クラスを追加します。

### <a name="review-the-contact-operations-requirements-class"></a>連絡先の操作の要件のクラスを確認してください。

`ContactOperations` クラスを確認します。 このクラスには、要件が含まれています。 アプリがサポートされます。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a>連絡先の Razor ページの基本クラスを作成します。

連絡先の Razor ページで使用されるサービスを含む基本クラスを作成します。 基底クラスでは、1 つの場所に、初期化コードが配置します。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/DI_BasePageModel.cs)]

上のコードでは以下の操作が行われます。

* 承認ハンドラーにアクセスするための `IAuthorizationService` サービスを追加します。
* Id `UserManager` サービスを追加します。
* `ApplicationDbContext` を追加します。

### <a name="update-the-createmodel"></a>CreateModel の更新します。

Create page model コンストラクターを更新して、`DI_BasePageModel` 基底クラスを使用するようにします。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

`CreateModel.OnPostAsync` メソッドを次のように更新します。

* `Contact` モデルにユーザー ID を追加します。
* ユーザーが連絡先を作成するためのアクセス許可を確認する認証ハンドラーを呼び出します。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>更新プログラム、IndexModel

`OnGetAsync` メソッドを更新して、承認された連絡先だけが一般ユーザーに表示されるようにします。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>更新プログラム、EditModel

ユーザーが連絡先を所有していることを確認するための承認ハンドラーを追加します。 リソース承認が検証されているため、`[Authorize]` 属性では不十分です。 アプリは、属性が評価されたときに、リソースへのアクセスにすることがありません。 リソース ベースの承認は、命令型である必要があります。 ページ モデルに読み込んで、または読み込みハンドラー自体内で、アプリが、リソースへのアクセスを持つ、チェックを実行する必要があります。 リソース キーを渡すことで、リソースを頻繁にアクセスするとします。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>更新プログラム、DeleteModel

承認ハンドラーを使用して、ユーザーが連絡先に削除するアクセス許可を持ってことを確認する delete ページ モデルを更新します。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>承認サービスをビューに挿入します。

現時点では、UI の表示では、編集し、ユーザーが変更できない連絡先へのリンクを削除します。

*ページ/_ViewImports*ファイルに承認サービスを挿入して、すべてのビューで使用できるようにします。

[!code-cshtml[](secure-data/samples/final3/Pages/_ViewImports.cshtml?highlight=6-99)]

上記のマークアップでは、いくつかの `using` ステートメントが追加されます。

*Pages/Contacts/Index. cshtml*の**編集**と**削除**のリンクを更新して、適切なアクセス許可を持つユーザーにのみ表示されるようにします。

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> 変更データへのアクセス許可がないユーザーからのリンクを非表示と、アプリをセキュリティで保護しません。 リンクを非表示アプリよりユーザー フレンドリな唯一の有効なリンクを表示することで。 ユーザーは、編集を呼び出すし、所有していないデータの操作を削除するには、生成された Url を切断できます。 Razor ページまたはコント ローラーは、データを保護するアクセス チェックを適用する必要があります。

### <a name="update-details"></a>更新プログラムの詳細

マネージャーが承認または連絡先を拒否するために、詳細ビューを更新します。

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Details.cshtml?name=snippet)]

詳細のページ モデルを更新します。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>追加またはロールにユーザーを削除します。

次の情報については、[この問題](https://github.com/aspnet/AspNetCore.Docs/issues/8502)を参照してください。

* ユーザーから権限を削除しています。 たとえば、チャットアプリでユーザーのミュートを行います。
* ユーザーに特権を追加します。

<a name="challenge"></a>

## <a name="differences-between-challenge-and-forbid"></a>チャレンジと禁止の違い

このアプリでは、認証された[ユーザーを要求](#require-authenticated-users)する既定のポリシーを設定します。 次のコードでは、匿名ユーザーを許可します。 匿名ユーザーは、チャレンジと禁止の違いを示すことができます。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details2.cshtml.cs?name=snippet)]

上のコードでは以下の操作が行われます。

* ユーザーが認証**されていない**場合は、`ChallengeResult` が返されます。 `ChallengeResult` が返されると、ユーザーはサインインページにリダイレクトされます。
* ユーザーが認証されているが承認されていない場合、`ForbidResult` が返されます。 `ForbidResult` が返されると、ユーザーは [アクセス拒否] ページにリダイレクトされます。

## <a name="test-the-completed-app"></a>完成したアプリをテストします。

シードされたユーザーアカウントのパスワードをまだ設定していない場合は、[シークレットマネージャーツール](xref:security/app-secrets#secret-manager)を使用してパスワードを設定します。

* 強力なパスワードを選択します。 8 を使用して、または詳細文字と少なくとも 1 つの大文字の文字、番号、およびシンボル。 たとえば、`Passw0rd!` は強力なパスワードの要件を満たしています。
* プロジェクトのフォルダーから次のコマンドを実行します。 `<PW>` はパスワードです。

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

場合は、アプリでは、連絡先があります。

* `Contact` テーブル内のすべてのレコードを削除します。
* データベースをシードするアプリを再起動します。

完成したアプリをテストする簡単な方法では、次の 3 つの異なるブラウザー (または incognito/inprivate ブラウズ セッション) を起動します。 1つのブラウザーで、新しいユーザーを登録します (たとえば、`test@example.com`)。 ブラウザーごとに別のユーザーでサインインします。 次の操作を確認します。

* 登録済みユーザーには、すべての承認済みの連絡先データを表示できます。
* 登録済みユーザーは編集/削除が自分のデータ。
* 管理者では、連絡先データの承認または却下をします。 `Details` ビューには、 **[承認]** ボタンと **[却下]** ボタンが表示されます。
* 管理者承認または却下して編集/削除のすべてのデータ。

| ユーザー                | アプリによってシード処理 | オプション                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | いいえ                | 独自のデータを編集/削除します。                |
| manager@contoso.com | はい               | 承認または却下と編集/削除は、データを所有します。 |
| admin@contoso.com   | はい               | 承認または却下し、すべてのデータの編集/削除します。 |

管理者のブラウザーで連絡先を作成します。 削除の URL をコピーし、管理者の連絡先から管理者を編集します。 テスト ユーザーのブラウザー テスト ユーザーがこれらの操作を実行できないことを確認するには、これらのリンクを貼り付けます。

## <a name="create-the-starter-app"></a>スターター アプリを作成します。

* 「ContactManager」という名前の Razor ページ アプリの作成
  * **個々のユーザーアカウント**を使用してアプリを作成します。
  * ファイル名「ContactManager」名前空間サンプルで使用する名前空間が一致するようにします。
  * `-uld` は SQLite ではなく LocalDB を指定します。

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* *モデル/Contact を追加します。*

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* `Contact` モデルをスキャフォールディングします。
* 最初の移行を作成し、データベースを更新します。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
dotnet ef database drop -f
dotnet ef migrations add initial
dotnet ef database update
```

`dotnet aspnet-codegenerator razorpage` コマンドでバグが発生した場合は、 [GitHub の問題](https://github.com/aspnet/Scaffolding/issues/984)を参照してください。

* *Pages/Shared/_Layout cshtml*ファイルで**contactmanager**アンカーを更新します。

 ```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Contacts/Index">ContactManager</a>
  ```

* 作成、編集、および連絡先を削除して、アプリをテストします。

### <a name="seed-the-database"></a>データベースのシード

[SeedData](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/starter3/Data/SeedData.cs)クラスを*Data*フォルダーに追加します。

[!code-csharp[](secure-data/samples/starter3/Data/SeedData.cs)]

`Main`から `SeedData.Initialize` を呼び出します。

[!code-csharp[](secure-data/samples/starter3/Program.cs)]

アプリが、データベースをシード処理をテストします。 DB の連絡先に行がある場合は、seed メソッドは実行されません。

::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

このチュートリアルでは、承認によって保護されたユーザー データと ASP.NET Core web アプリを作成する方法を示します。 認証済み (登録済み) のユーザーの連絡先の一覧を表示を作成します。 これには 3 つのセキュリティ グループがあります。

* **登録されているユーザー**は、すべての承認済みデータを表示したり、自分のデータを編集または削除したりできます。
* **管理者**は、連絡先データを承認または拒否できます。 承認されたメンバーのみがユーザーに表示されます。
* **管理者**は、任意のデータを承認/拒否したり、編集/削除したりできます。

次の図では、user Rick (`rick@example.com`) がサインインしています。 Rick は、承認された連絡先を表示して**編集**/**削除**/、連絡先の新しいリンクを**作成**することのみができます。 Rick によって作成された最後のレコードにのみ、**編集**および**削除**のリンクが表示されます。 他のユーザーは、管理者は、状態を"Approved"に変わるまで、最後のレコードを表示されません。

![サインインして Rick を示すスクリーン ショット](secure-data/_static/rick.png)

次の図では、`manager@contoso.com` がサインインし、マネージャーのロールに含まれています。

![サインイン manager@contoso.com を示すスクリーンショット](secure-data/_static/manager1.png)

次の図は、連絡先の詳細ビューをマネージャーには。

![連絡先のマネージャーの表示](secure-data/_static/manager.png)

**[承認]** ボタンと **[却下]** ボタンは、マネージャーと管理者に対してのみ表示されます。

次の図では、`admin@contoso.com` がサインインし、管理者のロールに含まれています。

![サインイン admin@contoso.com を示すスクリーンショット](secure-data/_static/admin.png)

管理者は、すべての特権を持ちます。 彼女は、読み取り/編集/削除のいずれかの連絡先をことができ、連絡先の状態を変更します。

アプリは、次の `Contact` モデルを[スキャフォールディング](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model)することによって作成されました。

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

サンプルには、次の承認ハンドラーが含まれています。

* `ContactIsOwnerAuthorizationHandler`: ユーザーがデータを編集できるようになります。
* `ContactManagerAuthorizationHandler`: 管理者が連絡先を承認または拒否できるようにします。
* `ContactAdministratorsAuthorizationHandler`: 管理者は、連絡先を承認または拒否したり、連絡先を編集または削除したりできます。

## <a name="prerequisites"></a>前提条件

このチュートリアルは、高度なです。 理解しておく必要があります。

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [認証](xref:security/authentication/identity)
* [アカウントの確認とパスワードの回復](xref:security/authentication/accconfirm)
* [承認](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>Starter および完成したアプリ

完成し[た](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples)アプリを[ダウンロード](xref:index#how-to-download-a-sample)します。 完成したアプリを[テスト](#test-the-completed-app)して、セキュリティ機能に慣れるようにします。

### <a name="the-starter-app"></a>スターター アプリ

[スターター](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/)アプリを[ダウンロード](xref:index#how-to-download-a-sample)します。

アプリを実行し、 **[Contactmanager]** リンクをタップして、連絡先を作成、編集、および削除できることを確認します。

## <a name="secure-user-data"></a>セキュリティで保護されたユーザー データ

次のセクションでは、セキュリティで保護されたユーザー データ アプリを作成するすべての主要な手順があります。 完成したプロジェクトを参照すると便利です。

### <a name="tie-the-contact-data-to-the-user"></a>ユーザーに連絡先データを関連付ける

ASP.NET [id](xref:security/authentication/identity)ユーザー id を使用すると、ユーザーがデータを編集できるようになりますが、他のユーザーデータは編集できません。 `OwnerID` と `ContactStatus` を `Contact` モデルに追加します。

[!code-csharp[](secure-data/samples/final2.1/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` は、 [id](xref:security/authentication/identity)データベースの `AspNetUser` テーブルのユーザー ID です。 [`Status`] フィールドによって、連絡先が一般ユーザーに表示されるかどうかが決まります。

新しい移行を作成し、データベースを更新します。

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-identity"></a>役割サービスの Id を追加します。

役割サービスを追加するには、 [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1)を追加します。

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet2&highlight=12)]

### <a name="require-authenticated-users"></a>認証されたユーザーが必要です。

ユーザー認証を必要とする既定の認証ポリシーを設定します。

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet&highlight=17-99)] 

 Razor ページ、コントローラー、またはアクションメソッドレベルで、`[AllowAnonymous]` 属性を使用して認証をオプトアウトできます。 ユーザー認証を必要とする既定の認証ポリシーの設定は、新しく追加された Razor ページとコント ローラーを保護します。 既定で認証が必要になるのは、新しいコントローラーに依存していて、Razor Pages `[Authorize]` 属性を含めるよりも安全です。

匿名ユーザーが登録前にサイトに関する情報を取得できるように、 [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute)を Index、About、および Contact の各ページに追加します。

[!code-csharp[](secure-data/samples/final2.1/Pages/Index.cshtml.cs?highlight=1,6)]

### <a name="configure-the-test-account"></a>テスト アカウントを構成します。

`SeedData` クラスは、管理者とマネージャーの2つのアカウントを作成します。 これらのアカウントのパスワードを設定するには、[シークレットマネージャーツール](xref:security/app-secrets)を使用します。 プロジェクトディレクトリ ( *Program.cs*を含むディレクトリ) のパスワードを設定します。

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

強力なパスワードが指定されていない場合、`SeedData.Initialize` が呼び出されると例外がスローされます。

テストパスワードを使用するように `Main` を更新します。

[!code-csharp[](secure-data/samples/final2.1/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>テスト アカウントを作成し、連絡先の更新

`SeedData` クラスの `Initialize` メソッドを更新して、テストアカウントを作成します。

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet_Initialize)]

管理者のユーザー ID と `ContactStatus` を連絡先に追加します。 「送信済み」と「拒否」の 1 つの連絡先を作成します。 すべての連絡先に、ユーザー ID と状態を追加します。 1 人だけが表示されます。

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>所有者、マネージャー、および管理者の承認ハンドラーを作成します。

*承認*フォルダーを作成し、その中に `ContactIsOwnerAuthorizationHandler` クラスを作成します。 `ContactIsOwnerAuthorizationHandler` は、リソースに対して動作しているユーザーがリソースを所有していることを確認します。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler` は[コンテキストを呼び出します。](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_)現在の認証済みユーザーが連絡先の所有者である場合は成功します。 承認ハンドラー通常。

* 要件が満たされた場合に `context.Succeed` を返します。
* 要件を満たしていない場合は `Task.CompletedTask` を返します。 `Task.CompletedTask` は成功でも失敗でも&mdash;他の承認ハンドラーの実行を許可します。

明示的に失敗する必要がある場合は、コンテキストを返し[ます。失敗](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail)。

アプリは、独自のデータを編集/削除/作成所有者が連絡先にできます。 `ContactIsOwnerAuthorizationHandler` は、要件パラメーターで渡された操作を確認する必要はありません。

### <a name="create-a-manager-authorization-handler"></a>マネージャーの承認ハンドラーを作成します。

*承認*フォルダーに `ContactManagerAuthorizationHandler` クラスを作成します。 `ContactManagerAuthorizationHandler` は、リソースに対して動作しているユーザーがマネージャーであることを確認します。 管理者だけでは、承認したり、(新規または変更された) コンテンツの変更を拒否することができます。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>管理者の承認ハンドラーを作成します。

*承認*フォルダーに `ContactAdministratorsAuthorizationHandler` クラスを作成します。 `ContactAdministratorsAuthorizationHandler` は、リソースに対して動作しているユーザーが管理者であることを確認します。 管理者は、すべての操作を実行できます。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>承認ハンドラーを登録します。

Entity Framework Core を使用するサービスは、 [AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions)を使用して[依存関係の挿入](xref:fundamentals/dependency-injection)に登録する必要があります。 `ContactIsOwnerAuthorizationHandler` は Entity Framework Core 上に構築された ASP.NET Core [id](xref:security/authentication/identity)を使用します。 サービスコレクションにハンドラーを登録して、[依存関係の挿入](xref:fundamentals/dependency-injection)によって `ContactsController` で使用できるようにします。 `ConfigureServices`の末尾に次のコードを追加します。

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet_defaultPolicy&highlight=27-99)]

`ContactAdministratorsAuthorizationHandler` と `ContactManagerAuthorizationHandler` はシングルトンとして追加されます。 EF を使用せず、必要なすべての情報が `HandleRequirementAsync` メソッドの `Context` パラメーターに含まれているため、これらはシングルトンです。

## <a name="support-authorization"></a>承認をサポートします。

このセクションでは、Razor ページを更新し、操作の要件クラスを追加します。

### <a name="review-the-contact-operations-requirements-class"></a>連絡先の操作の要件のクラスを確認してください。

`ContactOperations` クラスを確認します。 このクラスには、要件が含まれています。 アプリがサポートされます。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-razor-pages"></a>連絡先の Razor ページの基本クラスを作成します。

連絡先の Razor ページで使用されるサービスを含む基本クラスを作成します。 基底クラスでは、1 つの場所に、初期化コードが配置します。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/DI_BasePageModel.cs)]

上のコードでは以下の操作が行われます。

* 承認ハンドラーにアクセスするための `IAuthorizationService` サービスを追加します。
* Id `UserManager` サービスを追加します。
* `ApplicationDbContext` を追加します。

### <a name="update-the-createmodel"></a>CreateModel の更新します。

Create page model コンストラクターを更新して、`DI_BasePageModel` 基底クラスを使用するようにします。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

`CreateModel.OnPostAsync` メソッドを次のように更新します。

* `Contact` モデルにユーザー ID を追加します。
* ユーザーが連絡先を作成するためのアクセス許可を確認する認証ハンドラーを呼び出します。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>更新プログラム、IndexModel

`OnGetAsync` メソッドを更新して、承認された連絡先だけが一般ユーザーに表示されるようにします。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>更新プログラム、EditModel

ユーザーが連絡先を所有していることを確認するための承認ハンドラーを追加します。 リソース承認が検証されているため、`[Authorize]` 属性では不十分です。 アプリは、属性が評価されたときに、リソースへのアクセスにすることがありません。 リソース ベースの承認は、命令型である必要があります。 ページ モデルに読み込んで、または読み込みハンドラー自体内で、アプリが、リソースへのアクセスを持つ、チェックを実行する必要があります。 リソース キーを渡すことで、リソースを頻繁にアクセスするとします。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>更新プログラム、DeleteModel

承認ハンドラーを使用して、ユーザーが連絡先に削除するアクセス許可を持ってことを確認する delete ページ モデルを更新します。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>承認サービスをビューに挿入します。

現時点では、UI の表示では、編集し、ユーザーが変更できない連絡先へのリンクを削除します。

すべてのビューで使用できるように、 *views/_ViewImports cshtml*ファイルに承認サービスを挿入します。

[!code-cshtml[](secure-data/samples/final2.1/Pages/_ViewImports.cshtml?highlight=6-99)]

上記のマークアップでは、いくつかの `using` ステートメントが追加されます。

*Pages/Contacts/Index. cshtml*の**編集**と**削除**のリンクを更新して、適切なアクセス許可を持つユーザーにのみ表示されるようにします。

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> 変更データへのアクセス許可がないユーザーからのリンクを非表示と、アプリをセキュリティで保護しません。 リンクを非表示アプリよりユーザー フレンドリな唯一の有効なリンクを表示することで。 ユーザーは、編集を呼び出すし、所有していないデータの操作を削除するには、生成された Url を切断できます。 Razor ページまたはコント ローラーは、データを保護するアクセス チェックを適用する必要があります。

### <a name="update-details"></a>更新プログラムの詳細

マネージャーが承認または連絡先を拒否するために、詳細ビューを更新します。

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml?name=snippet)]

詳細のページ モデルを更新します。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>追加またはロールにユーザーを削除します。

次の情報については、[この問題](https://github.com/aspnet/AspNetCore.Docs/issues/8502)を参照してください。

* ユーザーから権限を削除しています。 たとえば、チャットアプリでユーザーのミュートを行います。
* ユーザーに特権を追加します。

## <a name="test-the-completed-app"></a>完成したアプリをテストします。

シードされたユーザーアカウントのパスワードをまだ設定していない場合は、[シークレットマネージャーツール](xref:security/app-secrets#secret-manager)を使用してパスワードを設定します。

* 強力なパスワードを選択します。 8 を使用して、または詳細文字と少なくとも 1 つの大文字の文字、番号、およびシンボル。 たとえば、`Passw0rd!` は強力なパスワードの要件を満たしています。
* プロジェクトのフォルダーから次のコマンドを実行します。 `<PW>` はパスワードです。

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

* データベースを削除して更新する

  ```dotnetcli
  dotnet ef database drop -f
  dotnet ef database update  
  ```

* データベースをシードするアプリを再起動します。

完成したアプリをテストする簡単な方法では、次の 3 つの異なるブラウザー (または incognito/inprivate ブラウズ セッション) を起動します。 1つのブラウザーで、新しいユーザーを登録します (たとえば、`test@example.com`)。 ブラウザーごとに別のユーザーでサインインします。 次の操作を確認します。

* 登録済みユーザーには、すべての承認済みの連絡先データを表示できます。
* 登録済みユーザーは編集/削除が自分のデータ。
* 管理者では、連絡先データの承認または却下をします。 `Details` ビューには、 **[承認]** ボタンと **[却下]** ボタンが表示されます。
* 管理者承認または却下して編集/削除のすべてのデータ。

| ユーザー                | アプリによってシード処理 | オプション                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | いいえ                | 独自のデータを編集/削除します。                |
| manager@contoso.com | はい               | 承認または却下と編集/削除は、データを所有します。 |
| admin@contoso.com   | はい               | 承認または却下し、すべてのデータの編集/削除します。 |

管理者のブラウザーで連絡先を作成します。 削除の URL をコピーし、管理者の連絡先から管理者を編集します。 テスト ユーザーのブラウザー テスト ユーザーがこれらの操作を実行できないことを確認するには、これらのリンクを貼り付けます。

## <a name="create-the-starter-app"></a>スターター アプリを作成します。

* 「ContactManager」という名前の Razor ページ アプリの作成
  * **個々のユーザーアカウント**を使用してアプリを作成します。
  * ファイル名「ContactManager」名前空間サンプルで使用する名前空間が一致するようにします。
  * `-uld` は SQLite ではなく LocalDB を指定します。

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* *モデル/Contact を追加します。*

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* `Contact` モデルをスキャフォールディングします。
* 最初の移行を作成し、データベースを更新します。

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
  dotnet ef database drop -f
  dotnet ef migrations add initial
  dotnet ef database update
  ```

* *Pages/_Layout cshtml*ファイルで**contactmanager**アンカーを更新します。

  ```cshtml
  <a asp-page="/Contacts/Index" class="navbar-brand">ContactManager</a>
  ```

* 作成、編集、および連絡先を削除して、アプリをテストします。

### <a name="seed-the-database"></a>データベースのシード

[SeedData](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/starter2.1/Data/SeedData.cs)クラスを*Data*フォルダーに追加します。

`Main`から `SeedData.Initialize` を呼び出します。

[!code-csharp[](secure-data/samples/starter2.1/Program.cs?name=snippet)]

アプリが、データベースをシード処理をテストします。 DB の連絡先に行がある場合は、seed メソッドは実行されません。

::: moniker-end

<a name="secure-data-add-resources-label"></a>

### <a name="additional-resources"></a>その他のリソース

* [Azure App Service で .NET Core と SQL Database web アプリを構築する](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [ASP.NET Core の承認ラボ](https://github.com/blowdart/AspNetAuthorizationWorkshop)。 このラボでは、このチュートリアルで導入されたセキュリティ機能の詳細に移動します。
* <xref:security/authorization/introduction>
* [カスタム ポリシー ベースの承認](xref:security/authorization/policies)
