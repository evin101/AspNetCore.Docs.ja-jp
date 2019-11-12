---
title: ASP.NET Core での統合テスト
author: guardrex
description: 統合テストによってデータベース、ファイル システム、ネットワークなどのインフラストラクチャ レベルで、アプリのコンポーネントがどのように正しく機能するようになるかを説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/28/2019
uid: test/integration-tests
ms.openlocfilehash: 33f3e29bc649fa65efdff0c47e54a83662005577
ms.sourcegitcommit: de0fc77487a4d342bcc30965ec5c142d10d22c03
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2019
ms.locfileid: "73143363"
---
# <a name="integration-tests-in-aspnet-core"></a>ASP.NET Core での統合テスト

作成者: [Luke Latham](https://github.com/guardrex)、 [Javier Calvarro Nelson](https://github.com/javiercn)、[Steve Smith](https://ardalis.com/)、 [Jos van der Til](https://jvandertil.nl)

::: moniker range=">= aspnetcore-3.0"

統合テストでは、アプリケーションのコンポーネントが、データベース、ファイルシステム、ネットワークなど、アプリのサポートインフラストラクチャを含むレベルで正しく機能していることを確認します。 ASP.NET Core では、単体テストフレームワークとテスト web ホストおよびメモリ内テストサーバーを使用した統合テストがサポートされています。

このトピックでは、単体テストの基本を理解していることを前提としています。 テストの概念を理解していない場合は、 [.Net Core の単体テストと .NET Standard に](/dotnet/core/testing/)関するトピックとそのリンク先を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプルアプリは Razor Pages アプリであり、Razor Pages の基本を理解していることを前提としています。 Razor Pages に慣れていない場合は、次のトピックを参照してください。

* [Razor ページを始める](xref:razor-pages/index)
* [Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)
* [Razor ページの単体テスト](xref:test/razor-pages-tests)

> [!NOTE]
> SPAs をテストする場合は、ブラウザーを自動化できる[Selenium](https://www.seleniumhq.org/)などのツールをお勧めします。

## <a name="introduction-to-integration-tests"></a>統合テストの概要

統合テストでは、[単体テスト](/dotnet/core/testing/)よりも広範なレベルでアプリのコンポーネントを評価します。 単体テストは、個々のクラスメソッドなど、分離されたソフトウェアコンポーネントをテストするために使用されます。 統合テストでは、2つ以上のアプリコンポーネントが連携して、予想される結果を生成することを確認します。要求を完全に処理するために必要なすべてのコンポーネントが含まれる可能性があります。

これらの広範なテストは、アプリのインフラストラクチャとフレームワーク全体をテストするために使用されます。多くの場合、次のコンポーネントが含まれます。

* データベース
* ファイル システム
* ネットワークアプライアンス
* 要求-応答パイプライン

単体テストでは、インフラストラクチャコンポーネントの代わりに、*フェイク*または*モックオブジェクト*と呼ばれる作成済みのコンポーネントを使用します。

単体テストとは対照的に、統合テストは次のとおりです。

* アプリが運用環境で使用する実際のコンポーネントを使用します。
* より多くのコードとデータ処理が必要です。
* 実行に時間がかかります。

そのため、統合テストの使用を最も重要なインフラストラクチャシナリオに限定してください。 単体テストまたは統合テストを使用して動作をテストできる場合は、単体テストを選択します。

> [!TIP]
> データベースとファイルシステムを使用して、データおよびファイルアクセスのすべての順列について、統合テストを記述しないでください。 アプリ全体でデータベースとファイルシステムを操作する場所に関係なく、読み取り、書き込み、更新、削除の各統合テストのセットは、通常、データベースおよびファイルシステムコンポーネントを適切にテストすることができます。 これらのコンポーネントと対話するメソッドロジックのルーチンテストでは、単体テストを使用します。 単体テストでは、インフラストラクチャのフェイク/モックを使用することにより、テストの実行時間が短縮されます。

> [!NOTE]
> 統合テストの説明では、テスト対象のプロジェクトは、多くの場合、*テスト対象のシステム*、または "SUT" と呼ばれます。
>
> *テストされた ASP.NET Core アプリを参照するには、このトピック全体で "SUT" を使用します。*

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 統合テスト

ASP.NET Core の統合テストには、次のものが必要です。

* テストプロジェクトは、テストを格納して実行するために使用されます。 テストプロジェクトには、SUT への参照が含まれています。
* テストプロジェクトでは、SUT のテスト web ホストを作成し、テストサーバークライアントを使用して、SUT との要求と応答を処理します。
* テストランナーは、テストを実行し、テスト結果を報告するために使用されます。

統合テストは、通常の*配置*、*操作*、および*アサート*の各テストステップを含む一連のイベントに従います。

1. SUT の web ホストが構成されています。
1. アプリケーションに要求を送信するためのテストサーバークライアントが作成されます。
1. テストの*配置*ステップが実行されます。テストアプリは要求を準備します。
1. *Act*テストステップが実行されます。クライアントが要求を送信し、応答を受信します。
1. *アサート*テストステップが実行されます。*実際*の応答は*成功*として検証されるか、*予期される*応答に基づいて*失敗*します。
1. すべてのテストが実行されるまで、プロセスは続行されます。
1. テスト結果が報告されます。

通常、テスト web ホストは、テストの実行のためにアプリの通常の web ホストとは異なる方法で構成されます。 たとえば、テストには別のデータベースまたは異なるアプリ設定が使用される場合があります。

テスト web ホストやメモリ内テストサーバー ([testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)) などのインフラストラクチャコンポーネントは、 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)パッケージによって提供または管理されます。 このパッケージを使用すると、テストの作成と実行を効率化できます。

`Microsoft.AspNetCore.Mvc.Testing` パッケージは、次のタスクを処理します。

* 依存関係ファイル (*deps*) を SUT からテストプロジェクトの*bin*ディレクトリにコピーします。
* テストの実行時に静的ファイルとページ/ビューが検出されるように、[コンテンツルート](xref:fundamentals/index#content-root)を SUT のプロジェクトルートに設定します。
* `TestServer`での SUT のブートストラップ化を効率化する[Webapplicationfactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)クラスを提供します。

[単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)のドキュメントでは、テストプロジェクトとテストランナーを設定する方法と、テストを実行する方法の詳細な手順と、テストおよびテストクラスの名前付け方法に関する推奨事項について説明します。

> [!NOTE]
> アプリのテストプロジェクトを作成する場合は、単体テストを統合テストから別のプロジェクトに分離します。 これにより、インフラストラクチャテストコンポーネントが誤って単体テストに含まれないようにすることができます。 単体テストと統合テストを分離すると、実行されるテストのセットを制御することもできます。

Razor Pages アプリと MVC アプリのテストの構成には、ほぼ違いはありません。 唯一の違いは、テストの名前付け方法です。 Razor Pages アプリでは、通常、ページエンドポイントのテストはページモデルクラスの後に名前が付けられます (たとえば、インデックスページのコンポーネントの統合をテストするには、`IndexPageTests` になります)。 MVC アプリでは、テストは通常、コントローラークラス別に編成され、テスト対象のコントローラーの後に名前が付けられます (たとえば、Home コントローラーのコンポーネントの統合をテストするには、`HomeControllerTests`)。

## <a name="test-app-prerequisites"></a>テストアプリの前提条件

テストプロジェクトは次の条件を持つ必要があります。

* [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)パッケージを参照します。
* プロジェクトファイルで Web SDK を指定します (`<Project Sdk="Microsoft.NET.Sdk.Web">`)。

これらの前提条件は、[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)で見ることができます。 *テスト/RazorPagesProject/RazorPagesProject*ファイルを調べます。 サンプルアプリでは、 [Xunit](https://xunit.github.io/)テストフレームワークと[AngleSharp](https://anglesharp.github.io/) parser ライブラリを使用しています。このため、サンプルアプリでも参照できます。

* [xunit](https://www.nuget.org/packages/xunit)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp)

テストでも Entity Framework Core が使用されます。 アプリの参照:

* [AspNetCore コア (Microsoft. 診断)](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore)
* [AspNetCore コア (Microsoft. Identity)](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore)
* [Microsoft EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore)
* [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory)
* [Microsoft EntityFrameworkCore. ツール](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools)

## <a name="sut-environment"></a>SUT 環境

SUT の[環境](xref:fundamentals/environments)が設定されていない場合、環境は既定で開発になります。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>既定の WebApplicationFactory を使用した基本テスト

[Webapplicationfactory \<TEntryPoint >](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)は、統合テスト用の[testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)を作成するために使用されます。 `TEntryPoint` は、SUT のエントリポイントクラス (通常は `Startup` クラス) です。

テストクラスは、クラスにテストが含まれていることを示す*クラスフィクスチャ*インターフェイス ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture)) を実装し、クラス内のテスト間で共有オブジェクトインスタンスを提供します。

次のテストクラス `BasicTests`では、`WebApplicationFactory` を使用して、SUT をブートストラップし、 [Httpclient](/dotnet/api/system.net.http.httpclient)をテストメソッドに `Get_EndpointsReturnSuccessAndCorrectContentType`します。 メソッドは、応答ステータスコードが成功したかどうかを確認し (200-299 の範囲の状態コード)、`Content-Type` ヘッダーは複数のアプリページで `text/html; charset=utf-8` ます。

[Createclient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)が `HttpClient` のインスタンスを作成します。このインスタンスは、自動的にリダイレクトを追跡し、cookie を処理します。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

既定では、 [GDPR 同意ポリシー](xref:security/gdpr)が有効になっている場合、必須ではない cookie は要求間で保持されません。 TempData プロバイダーで使用されているような、重要ではない cookie を保持するには、テストに必須としてマークします。 Cookie を必須としてマークする手順については、「[必須 cookie](xref:security/gdpr#essential-cookies)」を参照してください。

## <a name="customize-webapplicationfactory"></a>WebApplicationFactory をカスタマイズする

Web ホストの構成は、`WebApplicationFactory` から継承して1つ以上のカスタムファクトリを作成することによって、テストクラスとは別に作成できます。

1. `WebApplicationFactory` から継承し、 [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost)をオーバーライドします。 [Iwebhostbuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)では、 [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices)を使用してサービスコレクションを構成できます。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)でのデータベースのシード処理は、`InitializeDbForTests` メソッドによって実行されます。 この方法については、「[統合テストのサンプル: テストアプリの組織](#test-app-organization)」セクションを参照してください。

   SUT のデータベースコンテキストは `Startup.ConfigureServices` メソッドに登録されます。 テストアプリの `builder.ConfigureServices` コールバックは、アプリの `Startup.ConfigureServices` コードが実行され*た後*に実行されます。 アプリケーションのデータベースとは異なるデータベースをテストに使用するには、アプリのデータベースコンテキストを `builder.ConfigureServices` で置き換える必要があります。

   サンプルアプリでは、データベースコンテキストのサービス記述子を検索し、記述子を使用してサービス登録を削除します。 次に、ファクトリは、テストにメモリ内データベースを使用する新しい `ApplicationDbContext` を追加します。

   インメモリデータベースとは別のデータベースに接続するには、`UseInMemoryDatabase` 呼び出しを変更して、コンテキストを別のデータベースに接続します。 SQL Server テストデータベースを使用するには:

   * プロジェクトファイルで、 [Microsoft EntityFrameworkCore. SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) NuGet パッケージを参照します。
   * データベースへの接続文字列を使用して `UseSqlServer` を呼び出します。

   ```csharp
   services.AddDbContext<ApplicationDbContext>((options, context) => 
   {
       context.UseSqlServer(
           Configuration.GetConnectionString("TestingDbConnectionString"));
   });
   ```

2. テストクラスでは、カスタム `CustomWebApplicationFactory` を使用します。 次の例では、`IndexPageTests` クラスのファクトリを使用します。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   サンプルアプリのクライアントは、`HttpClient` がリダイレクトされるのを防ぐように構成されています。 「[モック認証](#mock-authentication)」セクションで後ほど説明するように、これにより、アプリの最初の応答の結果を確認するテストが許可されます。 最初の応答は、これらのテストの多くで `Location` ヘッダーを持つリダイレクトです。

3. 一般的なテストでは、`HttpClient` およびヘルパーメソッドを使用して、要求と応答を処理します。

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

SUT に対する POST 要求は、アプリの[データ保護のアンチ偽造システム](xref:security/data-protection/introduction)によって自動的に行われるアンチ偽造チェックを満たす必要があります。 テストの POST 要求を配置するには、テストアプリで次のことを行う必要があります。

1. ページに対して要求を行います。
1. アンチ偽造 cookie を解析し、応答から検証トークンを要求します。
1. 偽造防止 cookie と要求検証トークンを使用して POST 要求を行います。

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)の `SendAsync` ヘルパー拡張メソッド (*ヘルパー/HttpClientExtensions*) と `GetDocumentAsync` ヘルパーメソッド (*ヘルパー/htmlhelpers .Cs*) では、 [AngleSharp](https://anglesharp.github.io/)パーサーを使用して、による偽造防止チェックを処理します。次のメソッド:

* `GetDocumentAsync` &ndash; は[HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage)を受け取り、`IHtmlDocument` を返します。 `GetDocumentAsync` は、元の `HttpResponseMessage` に基づいて*仮想応答*を準備するファクトリを使用します。 詳細については、 [AngleSharp のドキュメント](https://github.com/AngleSharp/AngleSharp#documentation)を参照してください。
* `HttpClient` の `SendAsync` 拡張メソッドは、 [HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage)を作成し、 [HttpRequestMessage (sendasync)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_)を呼び出して、SUT に要求を送信します。 `SendAsync` のオーバーロードは、HTML フォーム (`IHtmlFormElement`) と次のものを受け入れます。
  * フォームの [送信] ボタン (`IHtmlElement`)
  * フォーム値コレクション (`IEnumerable<KeyValuePair<string, string>>`)
  * 送信ボタン (`IHtmlElement`) とフォーム値 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/)は、このトピックとサンプルアプリのデモンストレーションのために使用されるサードパーティ製の解析ライブラリです。 ASP.NET Core アプリの統合テストでは、AngleSharp はサポートされていないか、必要ありません。 [Html アジリティ Pack (HAP)](https://html-agility-pack.net/)など、その他のパーサーを使用することもできます。 もう1つの方法は、偽造防止システムの要求検証トークンを処理するコードを記述し、偽造 cookie を直接処理することです。

## <a name="customize-the-client-with-withwebhostbuilder"></a>WithWebHostBuilder を使用したクライアントのカスタマイズ

テストメソッド内で追加の構成が必要な場合、 [Withwebhostbuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder)は、構成によってさらにカスタマイズされた[iwebhostbuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)を含む新しい `WebApplicationFactory` を作成します。

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)の `Post_DeleteMessageHandler_ReturnsRedirectToRoot` テストメソッドは、`WithWebHostBuilder`の使用方法を示しています。 このテストでは、SUT でフォーム送信をトリガーすることによって、データベース内のレコードの削除を実行します。

`IndexPageTests` クラス内の別のテストでは、データベース内のすべてのレコードを削除し、`Post_DeleteMessageHandler_ReturnsRedirectToRoot` メソッドの前に実行できる操作を実行するため、このテストメソッドでデータベースを再シード処理して、SUT が削除するレコードが存在することを確認します。 SUT の `messages` フォームの最初の [削除] ボタンを選択すると、SUT に対する要求でシミュレートされます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>クライアントオプション

次の表に、`HttpClient` インスタンスを作成するときに使用できる既定の[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)を示します。

| オプション | 説明 | 既定 |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | `HttpClient` インスタンスがリダイレクト応答に自動的に従うかどうかを取得または設定します。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | `HttpClient` インスタンスのベースアドレスを取得または設定します。 | `http://localhost` |
| [HandleCookies](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | `HttpClient` インスタンスが cookie を処理する必要があるかどうかを取得または設定します。 | `true` |
| [Max自動リダイレクト](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | `HttpClient` インスタンスが従う必要があるリダイレクト応答の最大数を取得または設定します。 | 7 |

`WebApplicationFactoryClientOptions` クラスを作成し、 [createclient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)メソッドに渡します (既定値については、コード例を参照してください)。

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>モックサービスの注入

サービスは、ホストビルダーで[ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)を呼び出すことによって、テストでオーバーライドできます。 **モックサービスを挿入するには、SUT が `Startup.ConfigureServices` メソッドを持つ `Startup` クラスを持っている必要があります。**

サンプルの SUT には、引用符を返すスコープ付きサービスが含まれています。 インデックスページが要求されると、インデックスページの非表示フィールドに引用符が埋め込まれます。

*サービス/IQuoteService .cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

*サービス/QuoteService (cs*):

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

*Pages/Index.cshtml.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

*Pages/Index. .cs*:

[!code-cshtml[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

次のマークアップは、SUT アプリの実行時に生成されます。

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

統合テストでサービスと引用の挿入をテストするために、テストでは、モックサービスが SUT に挿入されます。 モックサービスは、アプリの `QuoteService` を、`TestQuoteService` というテストアプリによって提供されるサービスに置き換えます。

*IntegrationTests.IndexPageTests.cs*:

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

`ConfigureTestServices` が呼び出され、スコープサービスが登録されます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

テストの実行中に生成されたマークアップには `TestQuoteService` によって指定された見積もりテキストが反映されるため、アサーションは次のように渡されます。

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>モック認証

`AuthTests` クラスのテストは、セキュリティで保護されたエンドポイントであることを確認します。

* 認証されていないユーザーをアプリのログインページにリダイレクトします。
* 認証されたユーザーのコンテンツを返します。

SUT では、`/SecurePage` ページは、 [authorizepage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage)規約を使用して、ページに[authorizepage](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter)を適用します。 詳細については、「 [Razor Pages の承認規則](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)」を参照してください。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

`Get_SecurePageRedirectsAnUnauthenticatedUser` テストでは、 [Allowautoredirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect)を `false`に設定してリダイレクトを許可しないように[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)が設定されています。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

クライアントがリダイレクトに従うことを禁止することで、次のチェックを行うことができます。

* SUT によって返される状態コードは、予期された[httpstatuscode. リダイレクト](/dotnet/api/system.net.httpstatuscode)結果と比較して確認できます。これは、ログインページにリダイレクトした後の最終的な状態コードではなく、 [httpstatuscode. OK](/dotnet/api/system.net.httpstatuscode)です。
* 応答ヘッダーの `Location` ヘッダー値は、最後のログインページ応答ではなく `http://localhost/Identity/Account/Login` で開始されていることを確認し、`Location` ヘッダーが存在しないことを確認します。

テストアプリでは、認証と承認の側面をテストするために[ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)の <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> をモックすることができます。 最小のシナリオでは、AuthenticateResult が返さ[れます。成功](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*):

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

`TestAuthHandler` は、`AddAuthentication` が `ConfigureTestServices`に登録されている `Test` に認証スキームが設定されている場合に、ユーザーを認証するために呼び出されます。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

`WebApplicationFactoryClientOptions`の詳細については、「[クライアントオプション](#client-options)」セクションを参照してください。

## <a name="set-the-environment"></a>環境を設定する

既定では、SUT のホストとアプリ環境は、開発環境を使用するように構成されています。 SUT の環境をオーバーライドするには、次のようにします。

* `ASPNETCORE_ENVIRONMENT` 環境変数 (たとえば、`Staging`、`Production`、または `Testing`などのカスタム値) を設定します。
* テストアプリの `CreateHostBuilder` をオーバーライドして、`ASPNETCORE`で始まる環境変数を読み取ります。

```csharp
protected override IHostBuilder CreateHostBuilder() => 
    base.CreateHostBuilder()
        .ConfigureHostConfiguration(
            config => config.AddEnvironmentVariables("ASPNETCORE"));
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>テストインフラストラクチャがアプリコンテンツのルートパスを推測する方法

`WebApplicationFactory` コンストラクターは、`TEntryPoint` アセンブリ `System.Reflection.Assembly.FullName`と同じキーを持つ統合テストを含むアセンブリの[WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute)を検索することによって、アプリの[コンテンツルート](xref:fundamentals/index#content-root)パスを推論します。 正しいキーを持つ属性が見つからない場合 `WebApplicationFactory`、ソリューションファイル ( *.sln*) の検索にフォールバックし、`TEntryPoint` アセンブリ名をソリューションディレクトリに追加します。 アプリのルートディレクトリ (コンテンツのルートパス) は、ビューとコンテンツファイルを検出するために使用されます。

## <a name="disable-shadow-copying"></a>シャドウコピーを無効にする

シャドウコピーにより、テストは出力ディレクトリとは異なるディレクトリで実行されます。 テストが適切に機能するには、シャドウコピーを無効にする必要があります。 [サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)では xunit を使用し、正しい構成設定を含む xunit *. json*ファイルを含めることによって xunit のシャドウコピーを無効にします。 詳細については、「 [JSON を使用した xUnit の構成](https://xunit.github.io/docs/configuring-with-json.html)」を参照してください。

次のコンテンツを使用して、テストプロジェクトのルートに*xunit のランナー*ファイルを追加します。

```json
{
  "shadowCopy": false
}
```

## <a name="disposal-of-objects"></a>オブジェクトの破棄

`IClassFixture` 実装のテストが実行された後、xUnit が[Webapplicationfactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)を破棄すると[Testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)と[httpclient](/dotnet/api/system.net.http.httpclient)が破棄されます。 開発者によってインスタンス化されたオブジェクトが破棄を必要とする場合は、`IClassFixture` の実装でそれらを破棄します。 詳細については、「 [Dispose メソッドの実装](/dotnet/standard/garbage-collection/implementing-dispose)」を参照してください。

## <a name="integration-tests-sample"></a>統合テストのサンプル

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)は、次の2つのアプリで構成されています。

| アプリ | プロジェクトディレクトリ | 説明 |
| --- | ----------------- | ----------- |
| メッセージアプリ (SUT) | *src/RazorPagesProject* | ユーザーがメッセージを追加、削除、削除、および分析することを許可します。 |
| アプリのテスト | *テスト/RazorPagesProject* | SUT の統合テストに使用されます。 |

テストは、 [Visual Studio](https://visualstudio.microsoft.com)などの IDE の組み込みのテスト機能を使用して実行できます。 [Visual Studio Code](https://code.visualstudio.com/)またはコマンドラインを使用している場合は、*テスト/RazorPagesProject*ディレクトリのコマンドプロンプトで次のコマンドを実行します。

```console
dotnet test
```

### <a name="message-app-sut-organization"></a>メッセージアプリ (SUT) 組織

SUT は、次の特性を持つ Razor Pages メッセージシステムです。

* アプリのインデックスページ (*pages/index. cshtml*と*pages/index. cshtml. .cs*) には、メッセージの追加、削除、および分析を制御する UI およびページモデルのメソッドが用意されています (メッセージごとの平均語句)。
* メッセージは、`Id` (キー) と `Text` (message) の2つのプロパティを持つ `Message` クラス (*Data/message .cs*) によって記述されます。 `Text` プロパティは必須であり、200文字までに制限されています。
* メッセージは[Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224;を使用して格納されます。
* アプリには、データベースコンテキストクラス (Data/AppDbContext .cs) にデータアクセス層 (DAL) が含まれています。これは、`AppDbContext` (*data/AppDbContext .cs*) です。
* アプリケーションの起動時にデータベースが空の場合、メッセージストアは3つのメッセージで初期化されます。
* アプリには、認証されたユーザーのみがアクセスできる `/SecurePage` が含まれています。

&#8224;EF トピック「InMemory を使用した[テスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストにメモリ内データベースを使用する方法について説明しています。 このトピックでは、 [Xunit](https://xunit.github.io/)テストフレームワークを使用します。 テストの概念とテストの実装は、テストフレームワークごとに似ていますが、同一ではありません。

アプリはリポジトリパターンを使用せず、[作業単位 (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages はこれらの開発パターンをサポートしています。 詳細については、「[インフラストラクチャの永続化層](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)と[テストコントローラーのロジック](/aspnet/core/mvc/controllers/testing)の設計 (例では、リポジトリパターンを実装します)」を参照してください。

### <a name="test-app-organization"></a>テストアプリの組織

テストアプリは、 *tests/RazorPagesProject*ディレクトリ内のコンソールアプリです。

| テストアプリケーションディレクトリ | 説明 |
| ------------------ | ----------- |
| *AuthTests* | 次のテストメソッドが含まれます。<ul><li>認証されていないユーザーによるセキュリティで保護されたページへのアクセス。</li><li>認証されたユーザーがモック <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>を使用してセキュリティで保護されたページにアクセスする。</li><li>GitHub ユーザープロファイルを取得し、プロファイルのユーザーログインを確認する。</li></ul> |
| *BasicTests* | ルーティングおよびコンテンツタイプのテストメソッドが含まれています。 |
| *テストの統合* | カスタム `WebApplicationFactory` クラスを使用したインデックスページの統合テストが含まれています。 |
| *ヘルパー/ユーティリティ* | <ul><li>*Utilities.cs*には、テストデータを使用してデータベースをシード処理するために使用される `InitializeDbForTests` メソッドが含まれています。</li><li>*HtmlHelpers.cs*は、テストメソッドで使用するために AngleSharp `IHtmlDocument` を返すメソッドを提供します。</li><li>*HttpClientExtensions.cs*は、`SendAsync` のオーバーロードを使用して、SUT に要求を送信します。</li></ul> |

テストフレームワークは[Xunit](https://xunit.github.io/)です。 統合テストは、 [Testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)を含む[AspNetCore](/dotnet/api/microsoft.aspnetcore.testhost)を使用して実行されます。 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)パッケージはテストホストとテストサーバーを構成するために使用されるため、`TestHost` および `TestServer` パッケージはテストアプリのプロジェクトファイルまたはテストの開発者構成で直接パッケージ参照を必要としません。アプリケーション.

**テスト用のデータベースのシード処理**

統合テストでは、通常、テストを実行する前に、データベース内に小さなデータセットが必要です。 たとえば、削除テストでは、データベースレコードの削除を呼び出します。そのため、削除要求を成功させるには、データベースに少なくとも1つのレコードが必要です。

サンプルアプリでは、テストが実行時に使用できる*Utilities.cs*内の3つのメッセージを使用してデータベースをシードします。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

SUT のデータベースコンテキストは `Startup.ConfigureServices` メソッドに登録されます。 テストアプリの `builder.ConfigureServices` コールバックは、アプリの `Startup.ConfigureServices` コードが実行され*た後*に実行されます。 テストに別のデータベースを使用するには、アプリのデータベースコンテキストを `builder.ConfigureServices` で置き換える必要があります。 詳細については、「 [WebApplicationFactory のカスタマイズ](#customize-webapplicationfactory)」セクションを参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

統合テストでは、アプリケーションのコンポーネントが、データベース、ファイルシステム、ネットワークなど、アプリのサポートインフラストラクチャを含むレベルで正しく機能していることを確認します。 ASP.NET Core では、単体テストフレームワークとテスト web ホストおよびメモリ内テストサーバーを使用した統合テストがサポートされています。

このトピックでは、単体テストの基本を理解していることを前提としています。 テストの概念を理解していない場合は、 [.Net Core の単体テストと .NET Standard に](/dotnet/core/testing/)関するトピックとそのリンク先を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このサンプルアプリは Razor Pages アプリであり、Razor Pages の基本を理解していることを前提としています。 Razor Pages に慣れていない場合は、次のトピックを参照してください。

* [Razor ページを始める](xref:razor-pages/index)
* [Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)
* [Razor ページの単体テスト](xref:test/razor-pages-tests)

> [!NOTE]
> SPAs をテストする場合は、ブラウザーを自動化できる[Selenium](https://www.seleniumhq.org/)などのツールをお勧めします。

## <a name="introduction-to-integration-tests"></a>統合テストの概要

統合テストでは、[単体テスト](/dotnet/core/testing/)よりも広範なレベルでアプリのコンポーネントを評価します。 単体テストは、個々のクラスメソッドなど、分離されたソフトウェアコンポーネントをテストするために使用されます。 統合テストでは、2つ以上のアプリコンポーネントが連携して、予想される結果を生成することを確認します。要求を完全に処理するために必要なすべてのコンポーネントが含まれる可能性があります。

これらの広範なテストは、アプリのインフラストラクチャとフレームワーク全体をテストするために使用されます。多くの場合、次のコンポーネントが含まれます。

* データベース
* ファイル システム
* ネットワークアプライアンス
* 要求-応答パイプライン

単体テストでは、インフラストラクチャコンポーネントの代わりに、*フェイク*または*モックオブジェクト*と呼ばれる作成済みのコンポーネントを使用します。

単体テストとは対照的に、統合テストは次のとおりです。

* アプリが運用環境で使用する実際のコンポーネントを使用します。
* より多くのコードとデータ処理が必要です。
* 実行に時間がかかります。

そのため、統合テストの使用を最も重要なインフラストラクチャシナリオに限定してください。 単体テストまたは統合テストを使用して動作をテストできる場合は、単体テストを選択します。

> [!TIP]
> データベースとファイルシステムを使用して、データおよびファイルアクセスのすべての順列について、統合テストを記述しないでください。 アプリ全体でデータベースとファイルシステムを操作する場所に関係なく、読み取り、書き込み、更新、削除の各統合テストのセットは、通常、データベースおよびファイルシステムコンポーネントを適切にテストすることができます。 これらのコンポーネントと対話するメソッドロジックのルーチンテストでは、単体テストを使用します。 単体テストでは、インフラストラクチャのフェイク/モックを使用することにより、テストの実行時間が短縮されます。

> [!NOTE]
> 統合テストの説明では、テスト対象のプロジェクトは、多くの場合、*テスト対象のシステム*、または "SUT" と呼ばれます。
>
> *テストされた ASP.NET Core アプリを参照するには、このトピック全体で "SUT" を使用します。*

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 統合テスト

ASP.NET Core の統合テストには、次のものが必要です。

* テストプロジェクトは、テストを格納して実行するために使用されます。 テストプロジェクトには、SUT への参照が含まれています。
* テストプロジェクトでは、SUT のテスト web ホストを作成し、テストサーバークライアントを使用して、SUT との要求と応答を処理します。
* テストランナーは、テストを実行し、テスト結果を報告するために使用されます。

統合テストは、通常の*配置*、*操作*、および*アサート*の各テストステップを含む一連のイベントに従います。

1. SUT の web ホストが構成されています。
1. アプリケーションに要求を送信するためのテストサーバークライアントが作成されます。
1. テストの*配置*ステップが実行されます。テストアプリは要求を準備します。
1. *Act*テストステップが実行されます。クライアントが要求を送信し、応答を受信します。
1. *アサート*テストステップが実行されます。*実際*の応答は*成功*として検証されるか、*予期される*応答に基づいて*失敗*します。
1. すべてのテストが実行されるまで、プロセスは続行されます。
1. テスト結果が報告されます。

通常、テスト web ホストは、テストの実行のためにアプリの通常の web ホストとは異なる方法で構成されます。 たとえば、テストには別のデータベースまたは異なるアプリ設定が使用される場合があります。

テスト web ホストやメモリ内テストサーバー ([testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)) などのインフラストラクチャコンポーネントは、 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)パッケージによって提供または管理されます。 このパッケージを使用すると、テストの作成と実行を効率化できます。

`Microsoft.AspNetCore.Mvc.Testing` パッケージは、次のタスクを処理します。

* 依存関係ファイル (*deps*) を SUT からテストプロジェクトの*bin*ディレクトリにコピーします。
* テストの実行時に静的ファイルとページ/ビューが検出されるように、[コンテンツルート](xref:fundamentals/index#content-root)を SUT のプロジェクトルートに設定します。
* `TestServer`での SUT のブートストラップ化を効率化する[Webapplicationfactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)クラスを提供します。

[単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)のドキュメントでは、テストプロジェクトとテストランナーを設定する方法と、テストを実行する方法の詳細な手順と、テストおよびテストクラスの名前付け方法に関する推奨事項について説明します。

> [!NOTE]
> アプリのテストプロジェクトを作成する場合は、単体テストを統合テストから別のプロジェクトに分離します。 これにより、インフラストラクチャテストコンポーネントが誤って単体テストに含まれないようにすることができます。 単体テストと統合テストを分離すると、実行されるテストのセットを制御することもできます。

Razor Pages アプリと MVC アプリのテストの構成には、ほぼ違いはありません。 唯一の違いは、テストの名前付け方法です。 Razor Pages アプリでは、通常、ページエンドポイントのテストはページモデルクラスの後に名前が付けられます (たとえば、インデックスページのコンポーネントの統合をテストするには、`IndexPageTests` になります)。 MVC アプリでは、テストは通常、コントローラークラス別に編成され、テスト対象のコントローラーの後に名前が付けられます (たとえば、Home コントローラーのコンポーネントの統合をテストするには、`HomeControllerTests`)。

## <a name="test-app-prerequisites"></a>テストアプリの前提条件

テストプロジェクトは次の条件を持つ必要があります。

* 次のパッケージを参照します。
  * [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)
  * [Microsoft. AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing/)
* プロジェクトファイルで Web SDK を指定します (`<Project Sdk="Microsoft.NET.Sdk.Web">`)。 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)を参照するときは、Web SDK が必要です。

これらの前提条件は、[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)で見ることができます。 *テスト/RazorPagesProject/RazorPagesProject*ファイルを調べます。 サンプルアプリでは、 [Xunit](https://xunit.github.io/)テストフレームワークと[AngleSharp](https://anglesharp.github.io/) parser ライブラリを使用しています。このため、サンプルアプリでも参照できます。

* [xunit](https://www.nuget.org/packages/xunit/)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio/)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp/)

## <a name="sut-environment"></a>SUT 環境

SUT の[環境](xref:fundamentals/environments)が設定されていない場合、環境は既定で開発になります。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>既定の WebApplicationFactory を使用した基本テスト

[Webapplicationfactory \<TEntryPoint >](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)は、統合テスト用の[testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)を作成するために使用されます。 `TEntryPoint` は、SUT のエントリポイントクラス (通常は `Startup` クラス) です。

テストクラスは、クラスにテストが含まれていることを示す*クラスフィクスチャ*インターフェイス ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture)) を実装し、クラス内のテスト間で共有オブジェクトインスタンスを提供します。

次のテストクラス `BasicTests`では、`WebApplicationFactory` を使用して、SUT をブートストラップし、 [Httpclient](/dotnet/api/system.net.http.httpclient)をテストメソッドに `Get_EndpointsReturnSuccessAndCorrectContentType`します。 メソッドは、応答ステータスコードが成功したかどうかを確認し (200-299 の範囲の状態コード)、`Content-Type` ヘッダーは複数のアプリページで `text/html; charset=utf-8` ます。

[Createclient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)が `HttpClient` のインスタンスを作成します。このインスタンスは、自動的にリダイレクトを追跡し、cookie を処理します。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

既定では、 [GDPR 同意ポリシー](xref:security/gdpr)が有効になっている場合、必須ではない cookie は要求間で保持されません。 TempData プロバイダーで使用されているような、重要ではない cookie を保持するには、テストに必須としてマークします。 Cookie を必須としてマークする手順については、「[必須 cookie](xref:security/gdpr#essential-cookies)」を参照してください。

## <a name="customize-webapplicationfactory"></a>WebApplicationFactory をカスタマイズする

Web ホストの構成は、`WebApplicationFactory` から継承して1つ以上のカスタムファクトリを作成することによって、テストクラスとは別に作成できます。

1. `WebApplicationFactory` から継承し、 [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost)をオーバーライドします。 [Iwebhostbuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)では、 [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices)を使用してサービスコレクションを構成できます。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)でのデータベースのシード処理は、`InitializeDbForTests` メソッドによって実行されます。 この方法については、「[統合テストのサンプル: テストアプリの組織](#test-app-organization)」セクションを参照してください。

2. テストクラスでは、カスタム `CustomWebApplicationFactory` を使用します。 次の例では、`IndexPageTests` クラスのファクトリを使用します。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   サンプルアプリのクライアントは、`HttpClient` がリダイレクトされるのを防ぐように構成されています。 「[モック認証](#mock-authentication)」セクションで後ほど説明するように、これにより、アプリの最初の応答の結果を確認するテストが許可されます。 最初の応答は、これらのテストの多くで `Location` ヘッダーを持つリダイレクトです。

3. 一般的なテストでは、`HttpClient` およびヘルパーメソッドを使用して、要求と応答を処理します。

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

SUT に対する POST 要求は、アプリの[データ保護のアンチ偽造システム](xref:security/data-protection/introduction)によって自動的に行われるアンチ偽造チェックを満たす必要があります。 テストの POST 要求を配置するには、テストアプリで次のことを行う必要があります。

1. ページに対して要求を行います。
1. アンチ偽造 cookie を解析し、応答から検証トークンを要求します。
1. 偽造防止 cookie と要求検証トークンを使用して POST 要求を行います。

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)の `SendAsync` ヘルパー拡張メソッド (*ヘルパー/HttpClientExtensions*) と `GetDocumentAsync` ヘルパーメソッド (*ヘルパー/htmlhelpers .Cs*) では、 [AngleSharp](https://anglesharp.github.io/)パーサーを使用して、による偽造防止チェックを処理します。次のメソッド:

* `GetDocumentAsync` &ndash; は[HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage)を受け取り、`IHtmlDocument` を返します。 `GetDocumentAsync` は、元の `HttpResponseMessage` に基づいて*仮想応答*を準備するファクトリを使用します。 詳細については、 [AngleSharp のドキュメント](https://github.com/AngleSharp/AngleSharp#documentation)を参照してください。
* `HttpClient` の `SendAsync` 拡張メソッドは、 [HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage)を作成し、 [HttpRequestMessage (sendasync)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_)を呼び出して、SUT に要求を送信します。 `SendAsync` のオーバーロードは、HTML フォーム (`IHtmlFormElement`) と次のものを受け入れます。
  * フォームの [送信] ボタン (`IHtmlElement`)
  * フォーム値コレクション (`IEnumerable<KeyValuePair<string, string>>`)
  * 送信ボタン (`IHtmlElement`) とフォーム値 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/)は、このトピックとサンプルアプリのデモンストレーションのために使用されるサードパーティ製の解析ライブラリです。 ASP.NET Core アプリの統合テストでは、AngleSharp はサポートされていないか、必要ありません。 [Html アジリティ Pack (HAP)](https://html-agility-pack.net/)など、その他のパーサーを使用することもできます。 もう1つの方法は、偽造防止システムの要求検証トークンを処理するコードを記述し、偽造 cookie を直接処理することです。

## <a name="customize-the-client-with-withwebhostbuilder"></a>WithWebHostBuilder を使用したクライアントのカスタマイズ

テストメソッド内で追加の構成が必要な場合、 [Withwebhostbuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder)は、構成によってさらにカスタマイズされた[iwebhostbuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)を含む新しい `WebApplicationFactory` を作成します。

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)の `Post_DeleteMessageHandler_ReturnsRedirectToRoot` テストメソッドは、`WithWebHostBuilder`の使用方法を示しています。 このテストでは、SUT でフォーム送信をトリガーすることによって、データベース内のレコードの削除を実行します。

`IndexPageTests` クラス内の別のテストでは、データベース内のすべてのレコードを削除し、`Post_DeleteMessageHandler_ReturnsRedirectToRoot` メソッドの前に実行できる操作を実行するため、このテストメソッドでデータベースを再シード処理して、SUT が削除するレコードが存在することを確認します。 SUT の `messages` フォームの最初の [削除] ボタンを選択すると、SUT に対する要求でシミュレートされます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>クライアントオプション

次の表に、`HttpClient` インスタンスを作成するときに使用できる既定の[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)を示します。

| オプション | 説明 | 既定 |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | `HttpClient` インスタンスがリダイレクト応答に自動的に従うかどうかを取得または設定します。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | `HttpClient` インスタンスのベースアドレスを取得または設定します。 | `http://localhost` |
| [HandleCookies](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | `HttpClient` インスタンスが cookie を処理する必要があるかどうかを取得または設定します。 | `true` |
| [Max自動リダイレクト](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | `HttpClient` インスタンスが従う必要があるリダイレクト応答の最大数を取得または設定します。 | 7 |

`WebApplicationFactoryClientOptions` クラスを作成し、 [createclient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient)メソッドに渡します (既定値については、コード例を参照してください)。

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>モックサービスの注入

サービスは、ホストビルダーで[ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)を呼び出すことによって、テストでオーバーライドできます。 **モックサービスを挿入するには、SUT が `Startup.ConfigureServices` メソッドを持つ `Startup` クラスを持っている必要があります。**

サンプルの SUT には、引用符を返すスコープ付きサービスが含まれています。 インデックスページが要求されると、インデックスページの非表示フィールドに引用符が埋め込まれます。

*サービス/IQuoteService .cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

*サービス/QuoteService (cs*):

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

*Pages/Index.cshtml.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

*Pages/Index. .cs*:

[!code-cshtml[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

次のマークアップは、SUT アプリの実行時に生成されます。

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

統合テストでサービスと引用の挿入をテストするために、テストでは、モックサービスが SUT に挿入されます。 モックサービスは、アプリの `QuoteService` を、`TestQuoteService` というテストアプリによって提供されるサービスに置き換えます。

*IntegrationTests.IndexPageTests.cs*:

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

`ConfigureTestServices` が呼び出され、スコープサービスが登録されます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

テストの実行中に生成されたマークアップには `TestQuoteService` によって指定された見積もりテキストが反映されるため、アサーションは次のように渡されます。

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>モック認証

`AuthTests` クラスのテストは、セキュリティで保護されたエンドポイントであることを確認します。

* 認証されていないユーザーをアプリのログインページにリダイレクトします。
* 認証されたユーザーのコンテンツを返します。

SUT では、`/SecurePage` ページは、 [authorizepage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage)規約を使用して、ページに[authorizepage](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter)を適用します。 詳細については、「 [Razor Pages の承認規則](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)」を参照してください。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

`Get_SecurePageRedirectsAnUnauthenticatedUser` テストでは、 [Allowautoredirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect)を `false`に設定してリダイレクトを許可しないように[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)が設定されています。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

クライアントがリダイレクトに従うことを禁止することで、次のチェックを行うことができます。

* SUT によって返される状態コードは、予期された[httpstatuscode. リダイレクト](/dotnet/api/system.net.httpstatuscode)結果と比較して確認できます。これは、ログインページにリダイレクトした後の最終的な状態コードではなく、 [httpstatuscode. OK](/dotnet/api/system.net.httpstatuscode)です。
* 応答ヘッダーの `Location` ヘッダー値は、最後のログインページ応答ではなく `http://localhost/Identity/Account/Login` で開始されていることを確認し、`Location` ヘッダーが存在しないことを確認します。

テストアプリでは、認証と承認の側面をテストするために[ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)の <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1> をモックすることができます。 最小のシナリオでは、AuthenticateResult が返さ[れます。成功](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*):

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

`TestAuthHandler` は、`AddAuthentication` が `ConfigureTestServices`に登録されている `Test` に認証スキームが設定されている場合に、ユーザーを認証するために呼び出されます。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

`WebApplicationFactoryClientOptions`の詳細については、「[クライアントオプション](#client-options)」セクションを参照してください。

## <a name="set-the-environment"></a>環境を設定する

既定では、SUT のホストとアプリ環境は、開発環境を使用するように構成されています。 SUT の環境をオーバーライドするには、次のようにします。

* `ASPNETCORE_ENVIRONMENT` 環境変数 (たとえば、`Staging`、`Production`、または `Testing`などのカスタム値) を設定します。
* テストアプリの `CreateHostBuilder` をオーバーライドして、`ASPNETCORE`で始まる環境変数を読み取ります。

```csharp
protected override IHostBuilder CreateHostBuilder() => 
    base.CreateHostBuilder()
        .ConfigureHostConfiguration(
            config => config.AddEnvironmentVariables("ASPNETCORE"));
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>テストインフラストラクチャがアプリコンテンツのルートパスを推測する方法

`WebApplicationFactory` コンストラクターは、`TEntryPoint` アセンブリ `System.Reflection.Assembly.FullName`と同じキーを持つ統合テストを含むアセンブリの[WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute)を検索することによって、アプリの[コンテンツルート](xref:fundamentals/index#content-root)パスを推論します。 正しいキーを持つ属性が見つからない場合 `WebApplicationFactory`、ソリューションファイル ( *.sln*) の検索にフォールバックし、`TEntryPoint` アセンブリ名をソリューションディレクトリに追加します。 アプリのルートディレクトリ (コンテンツのルートパス) は、ビューとコンテンツファイルを検出するために使用されます。

## <a name="disable-shadow-copying"></a>シャドウコピーを無効にする

シャドウコピーにより、テストは出力ディレクトリとは異なるディレクトリで実行されます。 テストが適切に機能するには、シャドウコピーを無効にする必要があります。 [サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)では xunit を使用し、正しい構成設定を含む xunit *. json*ファイルを含めることによって xunit のシャドウコピーを無効にします。 詳細については、「 [JSON を使用した xUnit の構成](https://xunit.github.io/docs/configuring-with-json.html)」を参照してください。

次のコンテンツを使用して、テストプロジェクトのルートに*xunit のランナー*ファイルを追加します。

```json
{
  "shadowCopy": false
}
```

Visual Studio を使用している場合は、ファイルの **[出力ディレクトリにコピー]** プロパティを **[常にコピー]** する に設定します。 Visual Studio を使用していない場合は、テストアプリのプロジェクトファイルに `Content` ターゲットを追加します。

```xml
<ItemGroup>
  <Content Update="xunit.runner.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </Content>
</ItemGroup>
```

## <a name="disposal-of-objects"></a>オブジェクトの破棄

`IClassFixture` 実装のテストが実行された後、xUnit が[Webapplicationfactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1)を破棄すると[Testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)と[httpclient](/dotnet/api/system.net.http.httpclient)が破棄されます。 開発者によってインスタンス化されたオブジェクトが破棄を必要とする場合は、`IClassFixture` の実装でそれらを破棄します。 詳細については、「 [Dispose メソッドの実装](/dotnet/standard/garbage-collection/implementing-dispose)」を参照してください。

## <a name="integration-tests-sample"></a>統合テストのサンプル

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)は、次の2つのアプリで構成されています。

| アプリ | プロジェクトディレクトリ | 説明 |
| --- | ----------------- | ----------- |
| メッセージアプリ (SUT) | *src/RazorPagesProject* | ユーザーがメッセージを追加、削除、削除、および分析することを許可します。 |
| アプリのテスト | *テスト/RazorPagesProject* | SUT の統合テストに使用されます。 |

テストは、 [Visual Studio](https://visualstudio.microsoft.com)などの IDE の組み込みのテスト機能を使用して実行できます。 [Visual Studio Code](https://code.visualstudio.com/)またはコマンドラインを使用している場合は、*テスト/RazorPagesProject*ディレクトリのコマンドプロンプトで次のコマンドを実行します。

```dotnetcli
dotnet test
```

### <a name="message-app-sut-organization"></a>メッセージアプリ (SUT) 組織

SUT は、次の特性を持つ Razor Pages メッセージシステムです。

* アプリのインデックスページ (*pages/index. cshtml*と*pages/index. cshtml. .cs*) には、メッセージの追加、削除、および分析を制御する UI およびページモデルのメソッドが用意されています (メッセージごとの平均語句)。
* メッセージは、`Id` (キー) と `Text` (message) の2つのプロパティを持つ `Message` クラス (*Data/message .cs*) によって記述されます。 `Text` プロパティは必須であり、200文字までに制限されています。
* メッセージは[Entity Framework のメモリ内データベース](/ef/core/providers/in-memory/)&#8224;を使用して格納されます。
* アプリには、データベースコンテキストクラス (Data/AppDbContext .cs) にデータアクセス層 (DAL) が含まれています。これは、`AppDbContext` (*data/AppDbContext .cs*) です。
* アプリケーションの起動時にデータベースが空の場合、メッセージストアは3つのメッセージで初期化されます。
* アプリには、認証されたユーザーのみがアクセスできる `/SecurePage` が含まれています。

&#8224;EF トピック「InMemory を使用した[テスト](/ef/core/miscellaneous/testing/in-memory)」では、MSTest を使用したテストにメモリ内データベースを使用する方法について説明しています。 このトピックでは、 [Xunit](https://xunit.github.io/)テストフレームワークを使用します。 テストの概念とテストの実装は、テストフレームワークごとに似ていますが、同一ではありません。

アプリはリポジトリパターンを使用せず、[作業単位 (UoW) パターン](https://martinfowler.com/eaaCatalog/unitOfWork.html)の有効な例ではありませんが、Razor Pages はこれらの開発パターンをサポートしています。 詳細については、「[インフラストラクチャの永続化層](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)と[テストコントローラーのロジック](/aspnet/core/mvc/controllers/testing)の設計 (例では、リポジトリパターンを実装します)」を参照してください。

### <a name="test-app-organization"></a>テストアプリの組織

テストアプリは、 *tests/RazorPagesProject*ディレクトリ内のコンソールアプリです。

| テストアプリケーションディレクトリ | 説明 |
| ------------------ | ----------- |
| *AuthTests* | 次のテストメソッドが含まれます。<ul><li>認証されていないユーザーによるセキュリティで保護されたページへのアクセス。</li><li>認証されたユーザーがモック <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>を使用してセキュリティで保護されたページにアクセスする。</li><li>GitHub ユーザープロファイルを取得し、プロファイルのユーザーログインを確認する。</li></ul> |
| *BasicTests* | ルーティングおよびコンテンツタイプのテストメソッドが含まれています。 |
| *テストの統合* | カスタム `WebApplicationFactory` クラスを使用したインデックスページの統合テストが含まれています。 |
| *ヘルパー/ユーティリティ* | <ul><li>*Utilities.cs*には、テストデータを使用してデータベースをシード処理するために使用される `InitializeDbForTests` メソッドが含まれています。</li><li>*HtmlHelpers.cs*は、テストメソッドで使用するために AngleSharp `IHtmlDocument` を返すメソッドを提供します。</li><li>*HttpClientExtensions.cs*は、`SendAsync` のオーバーロードを使用して、SUT に要求を送信します。</li></ul> |

テストフレームワークは[Xunit](https://xunit.github.io/)です。 統合テストは、 [Testserver](/dotnet/api/microsoft.aspnetcore.testhost.testserver)を含む[AspNetCore](/dotnet/api/microsoft.aspnetcore.testhost)を使用して実行されます。 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)パッケージはテストホストとテストサーバーを構成するために使用されるため、`TestHost` および `TestServer` パッケージはテストアプリのプロジェクトファイルまたはテストの開発者構成で直接パッケージ参照を必要としません。アプリケーション.

**テスト用のデータベースのシード処理**

統合テストでは、通常、テストを実行する前に、データベース内に小さなデータセットが必要です。 たとえば、削除テストでは、データベースレコードの削除を呼び出します。そのため、削除要求を成功させるには、データベースに少なくとも1つのレコードが必要です。

サンプルアプリでは、テストが実行時に使用できる*Utilities.cs*内の3つのメッセージを使用してデータベースをシードします。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* [単体テスト](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* <xref:test/razor-pages-tests>
* <xref:fundamentals/middleware/index>
* <xref:mvc/controllers/testing>
