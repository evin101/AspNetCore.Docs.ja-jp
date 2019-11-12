---
title: ASP.NET Core でのセッションとアプリの状態
author: rick-anderson
description: 異なる要求の間でセッションとアプリの状態を維持する方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/12/2019
uid: fundamentals/app-state
ms.openlocfilehash: ccb37a422d972ab9113bb4115473d054282dac87
ms.sourcegitcommit: 994da92edb0abf856b1655c18880028b15a28897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/25/2019
ms.locfileid: "71278689"
---
# <a name="session-and-app-state-in-aspnet-core"></a>ASP.NET Core でのセッションとアプリの状態

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、[Steve Smith](https://ardalis.com/)、[Diana LaRose](https://github.com/DianaLaRose)、[Luke Latham](https://github.com/guardrex)

HTTP はステートレス プロトコルです。 手順を追加しないと、HTTP 要求は独立したメッセージであり、ユーザーの値やアプリの状態は保持されません。 この記事では、要求と要求の間でユーザー データとアプリの状態を保持するための複数の方法について説明します。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/app-state/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="state-management"></a>状態管理

状態は、いくつかの方法で格納することができます。 このトピックではそれぞれの方法について説明します。

| 格納の方法 | 格納のメカニズム |
| ---------------- | ----------------- |
| [Cookie](#cookies) | HTTP Cookie (サーバー側アプリのコードを使って格納されるデータを含む場合があります) |
| [セッション状態](#session-state) | HTTP Cookie とサーバー側アプリ コード |
| [TempData](#tempdata) | HTTP Cookie またはセッション状態 |
| [クエリ文字列](#query-strings) | HTTP クエリ文字列 |
| [非表示フィールド](#hidden-fields) | HTTP フォーム フィールド |
| [HttpContext.Items](#httpcontextitems) | サーバー側アプリ コード |
| [キャッシュ](#cache) | サーバー側アプリ コード |
| [依存性の注入](#dependency-injection) | サーバー側アプリ コード |

## <a name="cookies"></a>クッキー

Cookie は、要求と要求の間でデータを格納します。 Cookie は要求ごとに送信されるために、そのサイズは最小に抑える必要があります。 理想的には、識別子だけを Cookie に格納し、データはアプリで格納します。 ほとんどのブラウザーで Cookie のサイズは 4096 バイトに制限されています。 ドメインごとに使用できる Cookie の数も制限されています。

Cookie は改ざんされる可能性があるため、アプリで検証する必要があります。 Cookie は、ユーザーが削除でき、クライアント上で期限切れになります。 ただし、Cookie は一般に、クライアントでデータを永続化するときに最も持続性のある形式です。

Cookie は、多くの場合、パーソナル化に利用されます。既知のユーザーのためにコンテンツをカスタマイズします。 ユーザーは識別されるだけであり、ほとんどの場合は認証されません。 Cookie には、ユーザーの名前、アカウント名、または一意ユーザー ID (GUID など) を格納できます。 その後は、優先される Web サイトの背景色など、ユーザーの個人用設定に Cookie を使ってアクセスできます。

Cookie を発行し、プライバシーの問題を扱うときは、[欧州連合の一般データ保護規則 (GDPR)](https://ec.europa.eu/info/law/law-topic/data-protection) に留意してください。 詳細については、「[General Data Protection Regulation (GDPR) support in ASP.NET Core](xref:security/gdpr)」(ASP.NET Core での一般データ保護規則 (GDPR) のサポート) をご覧ください。

## <a name="session-state"></a>セッション状態

セッション状態は、ユーザーが Web アプリを参照している期間中ユーザー データを格納するための ASP.NET Core のシナリオです。 セッション状態では、アプリで管理されているストアを使用して、クライアントからの要求間でデータを保持します。 セッション データはキャッシュによってバックアップされ、一時的なデータと見なされます。セッション データがなくても、サイトは機能し続けられる必要があります。 重要なアプリケーションのデータはユーザー データベースに格納し、パフォーマンスの最適化としてのみセッションでキャッシュする必要があります。

> [!NOTE]
> [SignalR Hub](xref:signalr/hubs) は HTTP コンテキストとは独立して実行する可能性があるため、[SignalR](xref:signalr/index) アプリではセッションはサポートされていません。 このようなことは、たとえば、ロングポーリング要求が HTTP コンテキストの有効期間を超えてハブによって開かれている場合に発生する可能性があります。

ASP.NET Core は、セッション ID を含む Cookie をクライアントに提供することで、セッションの状態を維持します。Cookie は要求ごとにサーバーに送信されます。 アプリは、セッション ID を使用してセッション データをフェッチします。

セッション状態は次の動作を示します。

* セッション Cookie はブラウザーに固有であるため、セッションはブラウザー間では共有されません。
* セッション Cookie は、ブラウザー セッションが終了するときに削除されます。
* Cookie を受け取り、セッションが期限切れになった場合、同じセッション Cookie を使用する新しいセッションが作成されます。
* 空のセッションは保持されません。要求と要求の間でセッションを維持するには、少なくとも 1 つの値をセッションが持っている必要があります。 セッションが保持されないと、新しい要求ごとに新しいセッション ID が生成されます。
* アプリは、最後の要求から限られた時間だけセッションを維持します。 アプリでは、セッション タイムアウトを設定するか、既定値の 20 分を使用します。 セッション状態は、特定のセッションに固有であるが、セッション間で永続的に保持する必要のないユーザー データの格納に最適です。
* セッション データは、[ISession.Clear](/dotnet/api/microsoft.aspnetcore.http.isession.clear) の実装が呼び出されるか、セッションが期限切れになると、削除されます。
* クライアント ブラウザーが閉じられたこと、またはクライアントでセッション Cookie が削除されるか期限切れになったことを、アプリ コードに通知する既定のメカニズムはありません。
* ASP.NET Core MVC と Razor ページのテンプレートには、一般データ保護規制 (GDPR) のサポートが含まれます。 セッション状態の Cookie は既定では必須になっていません。このため、サイトの訪問者が追跡を許可しない限り、セッション状態は機能しません。 詳細については、<xref:security/gdpr#tempdata-provider-and-session-state-cookies-arent-essential> を参照してください。

> [!WARNING]
> セッション状態には機密データを保存しないでください。 ユーザーがブラウザーを閉じず、セッション Cookie がクリアされない可能性があります。 一部のブラウザーでは、ブラウザー ウィンドウの間で有効なセッションの Cookie が維持されます。 セッションが 1 人のユーザーに制限されず、次のユーザーが同じセッション Cookie でアプリの閲覧を続けることがあります。

メモリ内キャッシュ プロバイダーは、アプリが存在するサーバーのメモリにセッション データを格納します。 サーバー ファームのシナリオでは次のようになります。

* "*固定セッション*" を使用して、個々のサーバー上の特定のアプリのインスタンスに、各セッションを結び付けます。 [Azure App Service](https://azure.microsoft.com/services/app-service/) は[アプリケーション要求ルーティング処理 (ARR)](/iis/extensions/planning-for-arr/using-the-application-request-routing-module) を使って、既定で固定セッションを強制的に使用します。 ただし、固定セッションは拡張性に影響を与え、Web アプリの更新を複雑にすることがあります。 もっとよい方法は、Redis または SQL Server の分散キャッシュを使用することで、固定セッションを必要としません。 詳細については、<xref:performance/caching/distributed> を参照してください。
* セッション Cookie は [IDataProtector](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotector) によって暗号化されます。 各コンピューターでセッション Cookie を読み取るには、データ保護を適切に構成する必要があります。 詳細については、<xref:security/data-protection/introduction> および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事をご覧ください。

### <a name="configure-session-state"></a>セッション状態を構成する

[Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) に含まれる [Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) パッケージは、セッション状態を管理するためのミドルウェアを提供します。 セッション ミドルウェアを有効にするには、`Startup` に次が含まれている必要があります。

* いずれかの [IDistributedCache](/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache) メモリ キャッシュ。 `IDistributedCache` 実装はセッションのバックアップ ストアとして利用されます。 詳細については、<xref:performance/caching/distributed> を参照してください。
* `ConfigureServices` での [AddSession](/dotnet/api/microsoft.extensions.dependencyinjection.sessionservicecollectionextensions.addsession) の呼び出し。
* `Configure` での [UseSession](/dotnet/api/microsoft.aspnetcore.builder.sessionmiddlewareextensions.usesession#Microsoft_AspNetCore_Builder_SessionMiddlewareExtensions_UseSession_Microsoft_AspNetCore_Builder_IApplicationBuilder_) の呼び出し。

次のコードでは、`IDistributedCache` の既定のメモリ内実装でメモリ内セッション プロバイダーを設定する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Startup.cs?name=snippet1&highlight=5-14,34)]

ミドルウェアの順序が重要です。 上の例では、`UseMvc` の後で `UseSession` を呼び出すと、`InvalidOperationException` 例外が発生します。 詳細については、[ミドルウェアの順序](xref:fundamentals/middleware/index#order)に関するページをご覧ください。

[HttpContext.Session](/dotnet/api/microsoft.aspnetcore.http.httpcontext.session) は、セッション状態を構成した後で使用できます。

`UseSession` を呼び出前に `HttpContext.Session` にアクセスすることはできません。

アプリが応答ストリームへの書き込みを開始した後では、新しいセッション Cookie を含む新しいセッションを作成できません。 例外は Web サーバー ログに記録され、ブラウザーには表示されません。

### <a name="load-session-state-asynchronously"></a>セッション状態を非同期的に読み込む

[TryGetValue](/dotnet/api/microsoft.aspnetcore.http.isession.trygetvalue)、[Set](/dotnet/api/microsoft.aspnetcore.http.isession.set)、または [Remove](/dotnet/api/microsoft.aspnetcore.http.isession.remove) メソッドの前に、[ISession.LoadAsync](/dotnet/api/microsoft.aspnetcore.http.isession.loadasync) メソッドが明示的に呼び出された場合にのみ、ASP.NET Core の既定のセッション プロバイダーは、基になっている [IDistributedCache](/dotnet/api/microsoft.extensions.caching.distributed.idistributedcache) バッキング ストアからセッション レコードを非同期に読み込みます。 `LoadAsync` を最初に呼び出さないと、基になっているセッション レコードは同期的に読み込まれ、パフォーマンスが大幅に低下する可能性があります。

アプリにこのパターンを強制させるには、`TryGetValue`、`Set`、または `Remove` の前に `LoadAsync` メソッドが呼び出されない場合に例外をスローするバージョンで、[DistributedSessionStore](/dotnet/api/microsoft.aspnetcore.session.distributedsessionstore) および [DistributedSession](/dotnet/api/microsoft.aspnetcore.session.distributedsession) の実装をラップします。 ラップしたバージョンをサービス コンテナーに登録します。

### <a name="session-options"></a>セッション オプション

セッションの既定値をオーバーライドするには、[SessionOptions](/dotnet/api/microsoft.aspnetcore.builder.sessionoptions) を使用します。

| オプション | 説明 |
| ------ | ----------- |
| [Cookie](/dotnet/api/microsoft.aspnetcore.builder.sessionoptions.cookie) | Cookie の作成に使用される設定を決定します。 [Name](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.name) の既定値は [SessionDefaults.CookieName](/dotnet/api/microsoft.aspnetcore.session.sessiondefaults.cookiename) (`.AspNetCore.Session`) です。 [Path](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.path) の既定値は [SessionDefaults.CookiePath](/dotnet/api/microsoft.aspnetcore.session.sessiondefaults.cookiepath) (`/`) です。 [SameSite](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.samesite) の既定値は [SameSiteMode.Lax](/dotnet/api/microsoft.aspnetcore.http.samesitemode) (`1`) です。 [HttpOnly](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.httponly) の既定値は `true` です。 [IsEssential](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.isessential) の既定値は `false` です。 |
| [IdleTimeout](/dotnet/api/microsoft.aspnetcore.builder.sessionoptions.idletimeout) | `IdleTimeout` は、内容を破棄されることなくセッションがアイドル状態になっていることのできる最大時間を示します。 セッションへのアクセスがあるたびに、タイムアウトはリセットされます。 この設定はセッションの内容にのみ適用され、Cookie には適用されません。 既定値は 20 分です。 |
| [IOTimeout](/dotnet/api/microsoft.aspnetcore.builder.sessionoptions.iotimeout) | ストアからのセッションの読み込み、またはストアに戻すコミットに対して許容される最大時間です。 この設定は非同期操作にのみ適用できます。 このタイムアウトは、[InfiniteTimeSpan](/dotnet/api/system.threading.timeout.infinitetimespan) を使用して無効にすることができます。 既定値は 1 分です。 |

セッションは Cookie を利用し、1 つのブラウザーからの要求を追跡し、識別します。 既定では、この Cookie は `.AspNetCore.Session` という名前になり、パス `/` を使用します。 Cookie の既定値ではドメインが指定されないため、ページのクライアント側スクリプトには使用できません ([HttpOnly](/dotnet/api/microsoft.aspnetcore.http.cookiebuilder.httponly) の既定値が `true` になるため)。

Cookie セッションの既定値をオーバーライドするには、`SessionOptions` を使用します。

[!code-csharp[](app-state/samples_snapshot/2.x/SessionSample/Startup.cs?name=snippet1&highlight=14-19)]

アプリは [IdleTimeout](/dotnet/api/microsoft.aspnetcore.builder.sessionoptions.idletimeout) プロパティを使用し、サーバーのキャッシュ内の内容が破棄されるまでのセッションのアイドル時間を決定します。 このプロパティは Cookie の有効期限に依存しません。 要求が[セッション ミドルウェア](/dotnet/api/microsoft.aspnetcore.session.sessionmiddleware)を通過するたびにタイムアウトがリセットされます。

セッション状態は "*ロックなし*" です。 2 つの要求がセッションの内容を同時に変更しようとした場合、最後の要求が最初の要求をオーバーライドします。 `Session` は*一貫性のあるセッション*として実装されます。つまり、コンテンツは全部まとめて保管されます。 2 つの要求が異なるセッション値を変更しようとしたとき、最後の要求が最初の要求によって行われたセッションの変更をオーバーライドすることがあります。

### <a name="set-and-get-session-values"></a>セッション値の設定および取得

セッション状態には、Razor Pages の [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel) クラスから、または MVC の [Controller](/dotnet/api/microsoft.aspnetcore.mvc.controller) クラスの [HttpContext.Session](/dotnet/api/microsoft.aspnetcore.http.httpcontext.session) でアクセスします。 このプロパティは [ISession](/dotnet/api/microsoft.aspnetcore.http.isession) 実装です。

`ISession` の実装では、整数値や文字列値を設定および取得するための複数の拡張メソッドが提供されています。 [Microsoft.AspNetCore.Http.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.Http.Extensions/) パッケージがプロジェクトによって参照されている場合、拡張メソッドは [Microsoft.AspNetCore.Http](/dotnet/api/microsoft.aspnetcore.http) 名前空間内にあります (拡張メソッドにアクセスするには、`using Microsoft.AspNetCore.Http;` ステートメントを追加します)。 どちらのパッケージも、[Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)に含まれます。

`ISession` 拡張メソッド:

* [Get(ISession, String)](/dotnet/api/microsoft.aspnetcore.http.sessionextensions.get)
* [GetInt32(ISession, String)](/dotnet/api/microsoft.aspnetcore.http.sessionextensions.getint32)
* [GetString(ISession, String)](/dotnet/api/microsoft.aspnetcore.http.sessionextensions.getstring)
* [SetInt32(ISession, String, Int32)](/dotnet/api/microsoft.aspnetcore.http.sessionextensions.setint32)
* [SetString(ISession, String, String)](/dotnet/api/microsoft.aspnetcore.http.sessionextensions.setstring)

次の例では、`IndexModel.SessionKeyName` キー (サンプル アプリ内の `_Name`) のセッション値を Razor Pages ページで取得します。

```csharp
@page
@using Microsoft.AspNetCore.Http
@model IndexModel

...

Name: @HttpContext.Session.GetString(IndexModel.SessionKeyName)
```

次の例では、整数および文字列を設定および取得する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=18-19,22-23)]

分散キャッシュのシナリオを有効にするには、メモリ内キャッシュを使用する場合であっても、すべてのセッション データをシリアル化する必要があります。 最小限の文字列と数値のシリアライザーが提供されています ([ISession](/dotnet/api/microsoft.aspnetcore.http.isession) のメソッドおよびの拡張メソッドをご覧ください)。 複合型は、JSON などの別のメカニズムを使ってユーザーがシリアル化する必要があります。

シリアル化可能なオブジェクトを設定および取得するには、次の拡張メソッドを追加します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Extensions/SessionExtensions.cs?name=snippet1)]

次の例では、シリアル化可能オブジェクトを拡張メソッドで設定および取得する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet2)]

## <a name="tempdata"></a>TempData

ASP.NET Core によって、Razor Pages [TempData](xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.TempData) または Controller <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> が発行されます。 このプロパティには、別の要求で読み取られるまでデータが格納されます。 [Keep (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) メソッドと [Peek (String)](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Peek*) メソッドを使用すると、要求の最後に削除せずにデータを調べることができます。 [Keep()](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ITempDataDictionary.Keep*) によって、リテンション期間についてディクショナリ内のすべての項目がマークされます。 `TempData` は特に、複数の要求にデータが必要な場合のリダイレクトに役立ちます。 `TempData` は、`TempData` プロバイダーによって Cookie またはセッション状態を使用して実装されます。

## <a name="tempdata-samples"></a>TempData のサンプル

顧客を作成する次のページについて考えてみましょう。

[!code-csharp[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet&highlight=15-16,30)]

次のページに `TempData["Message"]` が表示されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexPeek.cshtml?range=1-14)]

前のマークアップでは、`Peek` が使用されているため、要求の最後に `TempData["Message"]` が削除**されません**。 ページを更新すると `TempData["Message"]` が表示されます。

次のマークアップは前のコードと似ていますが、`Keep` を使用して要求の最後にデータを保存しています。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/IndexKeep.cshtml?range=1-14)]

*Indexpeek* ページと *IndexKeep* ページ間を移動しても `TempData["Message"]` は削除されません。

次のコードでは `TempData["Message"]` が表示されますが、要求の最後に `TempData["Message"]` が削除されます。

[!code-cshtml[](app-state/3.0samples/RazorPagesContacts/Pages/Customers/Index.cshtml?range=1-14)]

### <a name="tempdata-providers"></a>TempData プロバイダー

Cookie に TempData を格納するには、Cookie ベースの TempData プロバイダーが既定で使われます。

Cookie データは [IDataProtector](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotector) を使用して暗号化され、[Base64UrlTextEncoder](/dotnet/api/microsoft.aspnetcore.webutilities.base64urltextencoder) でエンコードされ、チャンクされます。 Cookie はチャンクされるため、ASP.NET Core 1.x の 1 Cookie のサイズ上限は適用されません。 暗号化されているデータを圧縮すると、[CRIME](https://wikipedia.org/wiki/CRIME_(security_exploit)) 攻撃や [BREACH](https://wikipedia.org/wiki/BREACH_(security_exploit)) 攻撃など、セキュリティ上の問題を起す可能性があるため、Cookie データは圧縮されません。 Cookie ベース TempData プロバイダーの詳細については、「[CookieTempDataProvider](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.cookietempdataprovider)」を参照してください。

### <a name="choose-a-tempdata-provider"></a>TempData プロバイダーを選択する

TempData プロバイダーを選択するときの考慮事項:

1. アプリは既にセッション状態を使っているかどうか。 使っている場合、(データのサイズを除き) セッション状態 TempData プロバイダーがそのアプリにコストを追加することはありません。
2. アプリでは、比較的少量のデータに対して (最大 500 バイト) TempData がわずかばかり使用されているか。 該当する場合、Cookie TempData プロバイダーは TempData を送信する要求ごとに少額のコストを追加します。 該当しない場合、セッション状態 TempData プロバイダーは便利かもしれません。TempData が尽きるまで、要求のたびに大量のデータをラウンドトリップすることが回避されます。
3. アプリは複数サーバーのサーバー ファームで実行しているか。 そうである場合は、データ保護の外部で Cookie TempData プロバイダーを使用するために、追加の構成は必要ありません (<xref:security/data-protection/introduction> および[キー ストレージ プロバイダー](xref:security/data-protection/implementation/key-storage-providers)に関する記事を参照)。

> [!NOTE]
> ほとんどの Web クライアント (Web ブラウザーなど) は、各 Cookie の最大サイズ、Cookie の合計数、または両方に上限を課します。 Cookie TempData プロバイダーを使用するとき、アプリでそれらの上限が超えないことを確認してください。 データの合計サイズを考慮してください。 暗号化とチャンクによる Cookie のサイズの増加を考慮してください。

### <a name="configure-the-tempdata-provider"></a>TempData プロバイダーを構成する

Cookie ベース TempData プロバイダーは既定で有効になります。

セッション ベースの TempData プロバイダーを有効にするには、[AddSessionStateTempDataProvider](/dotnet/api/microsoft.extensions.dependencyinjection.mvcviewfeaturesmvcbuilderextensions.addsessionstatetempdataprovider) 拡張メソッドを使います。

[!code-csharp[](app-state/samples_snapshot_2/2.x/SessionSample/Startup.cs?name=snippet1&highlight=11,13,32)]

ミドルウェアの順序が重要です。 上の例では、`UseMvc` の後で `UseSession` を呼び出すと、`InvalidOperationException` 例外が発生します。 詳細については、[ミドルウェアの順序](xref:fundamentals/middleware/index#order)に関するページをご覧ください。

> [!IMPORTANT]
> .NET Framework が対象で、セッション ベースの TempData プロバイダーを使う場合は、[Microsoft.AspNetCore.Session](https://www.nuget.org/packages/Microsoft.AspNetCore.Session/) パッケージをプロジェクトに追加します。

## <a name="query-strings"></a>クエリ文字列

限られた量のデータを要求間で渡すことができます。新しい要求のクエリ文字列にそれを追加します。 これは、リンクと埋め込まれた状態がメールまたはソーシャル ネットワークを通して共有されるよう、永久的に状態をキャプチャするのに役立ちます。 URL クエリ文字列はパブリックであるため、機密データにはクエリ文字列を使わないでください。

意図しない共有が発生するだけでなく、クエリ文字列にデータを含めると、[クロスサイト リクエスト フォージェリ (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) 攻撃の機会を与えてしまいます。認証中、ユーザーをだまして悪意のあるサイトに誘導します。 攻撃者はアプリからユーザー データを盗んだり、ユーザーになりすまして悪意のある行為を行ったりできます。 保存されるアプリまたはセッション状態は CSRF 攻撃を防ぐ必要があります。 詳細については、「[Prevent Cross-Site Request Forgery (XSRF/CSRF) attacks](xref:security/anti-request-forgery)」(クロスサイト リクエスト フォージェリ (XSRF/CSRF) 攻撃の防止) を参照してください。

## <a name="hidden-fields"></a>非表示フィールド

データは非表示フォーム フィールドに保存したり、次の要求で転記したりできます。 複数ページ フォームでは、これは一般的です。 クライアントがデータを改ざんする可能性があるため、アプリで非表示フィールドに格納されているデータを常に再検証する必要があります。

## <a name="httpcontextitems"></a>HttpContext.Items

1 つの要求を処理している間データを格納するには、[HttpContext.Items](/dotnet/api/microsoft.aspnetcore.http.httpcontext.items) コレクションを使います。 要求が処理された後、コレクションの内容は破棄されます。 `Items` コレクションは、コンポーネントまたはミドルウェアが要求の間の異なる時点で動作し、パラメーターを受け渡す直接的な方法がない場合に、コンポーネントとミドルウェアが通信できるようにするためによく使われます。

次の例では、[ミドルウェア](xref:fundamentals/middleware/index)により `isVerified` が `Items` コレクションに追加されます。

```csharp
app.Use(async (context, next) =>
{
    // perform some verification
    context.Items["isVerified"] = true;
    await next.Invoke();
});
```

後のパイプラインで、別のミドルウェアが `isVerified` の値にアクセスできます。

```csharp
app.Run(async (context) =>
{
    await context.Response.WriteAsync($"Verified: {context.Items["isVerified"]}");
});
```

1 つのアプリでのみ使用されるミドルウェアの場合、`string` キーが許容されます。 アプリのインスタンス間で共有されるミドルウェアでは、キーの競合を避けるため、一意のオブジェクト キーを使う必要があります。 次の例では、ミドルウェア クラスで定義されている一意のオブジェクト キーを使用する方法を示します。

[!code-csharp[](app-state/samples/2.x/SessionSample/Middleware/HttpContextItemsMiddleware.cs?name=snippet1&highlight=4,13)]

その他のコードは、ミドルウェア クラスで公開されるキーを利用し、`HttpContext.Items` に格納されている値にアクセスできます。

[!code-csharp[](app-state/samples/2.x/SessionSample/Pages/Index.cshtml.cs?name=snippet3)]

この方法には、コード内でキーの文字列を使用する必要がないという利点もあります。

## <a name="cache"></a>キャッシュ

キャッシュは、データを保存し、取得する効率的な方法です。 アプリでは、キャッシュされた項目の有効期間を制御できます。

キャッシュされたデータは、特定の要求、ユーザー、またはセッションに関連付けられていません。 **他のユーザーの要求によって取得される可能性があるので、ユーザー固有データをキャッシュしないように注意してください。**

詳細については、<xref:performance/caching/response> を参照してください。

## <a name="dependency-injection"></a>依存関係の挿入

[依存関係の注入](xref:fundamentals/dependency-injection)を利用し、すべてのユーザーがデータを利用できるようにします。

1. データを含むサービスを定義します。 たとえば、`MyAppData` という名前のクラスを定義します。

    ```csharp
    public class MyAppData
    {
        // Declare properties and methods
    }
    ```

2. サービス クラスを `Startup.ConfigureServices` に追加します。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<MyAppData>();
    }
    ```

3. データ サービス クラスを使用します。

    ```csharp
    public class IndexModel : PageModel
    {
        public IndexModel(MyAppData myService)
        {
            // Do something with the service
            //    Examples: Read data, store in a field or property
        }
    }
    ```

## <a name="common-errors"></a>一般的なエラー

* "'Microsoft.AspNetCore.Session.DistributedSessionStore' を起動しようとしましたが、型 'Microsoft.Extensions.Caching.Distributed.IDistributedCache' のサービスを解決できません。"

  これは通常、少なくとも 1 つの `IDistributedCache` 実装で構成に失敗したことで発生します。 詳細については、次のトピックを参照してください。 <xref:performance/caching/distributed> および <xref:performance/caching/memory>

* セッション ミドルウェアがセッションを永続化できなかった場合 (バッキング ストアを利用できない場合など)、ミドルウェアは例外をログに記録し、要求は普通に続行されます。 これにより、予期しない動作が発生します。

  たとえば、ユーザーがセッションでショッピング カートを格納します。 ユーザーはアイテムをカートに追加しますが、コミットが失敗します。 アプリはこの失敗を認識しないので、アイテムがカートに追加されたことをユーザーに伝えますが、これは正しくありません。

  エラーを確認するための推奨される方法は、アプリがセッションへの書き込みを終了したら、アプリ コードから `await feature.Session.CommitAsync();` を呼び出すことです。 バッキング ストアが利用できない場合、`CommitAsync` は例外をスローします。 `CommitAsync` が失敗した場合、アプリは例外を処理できます。 `LoadAsync` は、データ ストアが利用できない場合に同じ条件で例外をスローします。
  
## <a name="signalr-and-session-state"></a>SignalR とセッション状態

SignalR アプリでは、セッション状態を使用して情報を格納することはできません。 SignalR アプリは、ハブの `Context.Items` に接続ごとの状態を格納できます。 <!-- https://github.com/aspnet/SignalR/issues/2139 -->

## <a name="additional-resources"></a>その他の技術情報

<xref:host-and-deploy/web-farm>
