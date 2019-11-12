---
title: ASP.NET Core のルーティング
author: rick-anderson
description: ASP.NET Core のルーティングでどのように要求 URI をエンドポイント セレクターにマッピングし、受信要求をエンドポイントに配布するかについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/24/2019
uid: fundamentals/routing
ms.openlocfilehash: be4493cc927bd5437a2c9dab00b6a555756195bb
ms.sourcegitcommit: eb2fe5ad2e82fab86ca952463af8d017ba659b25
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/01/2019
ms.locfileid: "73416133"
---
# <a name="routing-in-aspnet-core"></a>ASP.NET Core のルーティング

作成者: [Ryan Nowak](https://github.com/rynowak)、[Steve Smith](https://ardalis.com/)、[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ルーティングでは、要求 URI をエンドポイントにマッピングし、受信要求をそれらのエンドポイントに配布します。 ルートはアプリに定義され、アプリの起動時に構成されます。 ルートは、要求に含まれている URL から値を任意で抽出できます。その値を要求処理に利用できます。 ルーティングでは、アプリからのルート情報を利用し、エンドポイントにマッピングする URL を生成することもできます。

> [!IMPORTANT]
> 本文では、ASP.NET Core ルーティングについて詳しく取り上げます。 ASP.NET Core MVC ルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。 Razor Pages のルーティング規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="routing-basics"></a>ルーティングの基本

ほとんどのアプリでは、URL を読みやすくてわかりやすいものにするために、基本的なでわかりやすいルーティング スキームを選択する必要があります。 既定の規則ルート `{controller=Home}/{action=Index}/{id?}`:

* 基本的でわかりやすいルーティング スキームをサポートしています。
* UI ベースのアプリの便利な開始点となります。

開発者は一般的に、特殊な状況でのアプリのトラフィックが多い領域 (ブログや E コマース エンドポイントなど) に対しては、[属性ルーティング](xref:mvc/controllers/routing#attribute-routing)または専用規則ルートを使用して簡易ルートをさらに追加します。

Web API では、属性ルーティングを使用して、HTTP 動詞で操作を表現するリソースのセットとしてアプリの機能をモデル化する必要があります。 つまり、同じ論理リソース上の多くの操作 (たとえば GET や POST) で、同じ URL を使用します。 属性ルーティングでは、API のパブリック エンドポイント レイアウトを慎重に設計するために必要となるコントロールのレベルが提供されます。

Razor Pages アプリでは既定の規則ルーティングを使用して、アプリの *Pages* フォルダーの名前付きリソースを提供します。 Razor Pages のルーティング動作をカスタマイズできる追加の規則を使用できます。 詳細については、次のトピックを参照してください。 <xref:razor-pages/index> および <xref:razor-pages/razor-pages-conventions>

URL 生成サポートを使用すると、アプリを相互にリンクする URL をハード コーディングすることなくアプリを開発できます。 このサポートにより、基本的なルーティング構成を使用して作業を開始し、アプリのリソース レイアウトが決まった後でルートを変更することができます。

ルーティングでは*エンドポイント* (`Endpoint`) を使用して、アプリの論理エンドポイントを表します。

エンドポイントでは、要求と、任意のメタデータのコレクションを処理するデリゲートが定義されます。 メタデータは、各エンドポイントにアタッチされている構成とポリシーに基づいて横断的な関心事を実装するために使用されます。

ルーティング システムには次の特性があります。

* ルート テンプレート構文は、トークン化されたルート パラメーターでルートを定義するために使用されます。
* 規則スタイルおよび属性スタイルのエンドポイント構成が許可されます。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> は、URL パラメーターに特定のエンドポイント制約で有効な値が含まれているかどうかを確認するために使用されます。
* MVC/Razor Pages などのアプリ モデルでは、ルーティング シナリオの予測可能な実装を持つ、すべてのエンドポイントを登録します。
* ルーティングの実装では、必要に応じて、ミドルウェア パイプラインでのルーティングに関する決定を行います。
* ルーティング ミドルウェアの後に表示されるミドルウェアでは、特定の要求 URI に関するルーティング ミドルウェアのエンドポイント決定の結果を検査できます。
* ミドルウェア パイプラインの任意の場所でアプリのすべてのエンドポイントを列挙することができます。
* アプリではルーティングを使用し、エンドポイント情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。
* URL の生成は、任意の拡張機能をサポートする、アドレスに基づきます。

  * リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) は、URI を生成するために[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) を使用して任意の場所で解決できます。
  * リンク ジェネレーター API を DI で使用できない場合、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> で URL を構築するためのメソッドが提供されます。

> [!NOTE]
> エンドポイント リンクは、MVC/Razor Pages のアクションとページに限られます。 エンドポイント リンク機能の拡張は、今後のリリースで予定されています。

ルーティングは <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> クラスにより[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに接続されます。 [ASP.NET Core MVC](xref:mvc/overview) では、ミドルウェア パイプラインにその構成の一部としてルーティングが追加され、MVC および Razor Pages アプリでのルーティングが処理されます。 スタンドアロン コンポーネントとしてルーティングを利用する方法については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションを参照してください。

### <a name="url-matching"></a>URL 一致

URL 一致というプロセスでは、ルーティングによって、受信要求が*エンドポイント*に配布されます。 このプロセスは URL パスのデータに基づきますが、要求内のあらゆるデータを考慮することもできます。 アプリの規模を大きくしたり、より複雑にしたりするとき、要求を個別のハンドラーに配布する機能が鍵となります。

ルーティング ミドルウェアの実行時には、エンドポイント (`Endpoint`) が設定されて、<xref:Microsoft.AspNetCore.Http.HttpContext> 上の機能に値がルーティングされます。 現在の要求の場合:

* `HttpContext.GetEndpoint` を呼び出すと、エンドポイントが取得されます。
* `HttpRequest.RouteValues` では、ルート値のコレクションが取得されます。

ルーティング ミドルウェアの後で実行されたミドルウェアでは、エンドポイントを参照し、アクションを実行することができます。 たとえば、認可ミドルウェアでは、エンドポイントのメタデータ コレクションに対し、認可ポリシーを問い合わせることができます。 要求処理パイプライン内のすべてのミドルウェアが実行された後、選択したエンドポイントのデリゲートが呼び出されます。

エンドポイント ルーティングのルーティング システムでは、配布に関するすべての決定が行われます。 ミドルウェアでは選択されたエンドポイントに基づいてポリシーが適用されるため、セキュリティ ポリシーの適用や配布に影響する可能性のある決定は、ルーティング システム内で行うことが重要です。

エンドポイントのデリゲートが実行されると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) は、ルートから生成された*ルート値*のディクショナリです。 この値は通常、URL のトークン化で決定されます。この値を利用してユーザー入力を受け取ったり、アプリ内で決定をさらに配布したりできます。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) は、一致したルートに関連する追加データのプロパティ バッグです。 一致したルートに基づいてアプリで決定を行えるように、<xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> が提供されます。これは状態データと各ルートの関連付けをサポートするためのものです。 この値は開発者が定義するものです。ルーティングの動作に影響を与えることは**ありません**。 また、[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) に一時退避される値はどのような型でも構いません。一方、[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) の場合は、文字列間で変換できる必要があります。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) は、過去に要求に一致したルートの一覧です。 ルートはルートの中に入れ子にすることができます。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> プロパティは、結果的に一致をもたらしたルートの論理ツリーを通るパスを表します。 一般的に、<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最初の項目はルート コレクションであり、これは URL 生成に使用するものです。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最後の項目は、一致したルート ハンドラーです。

<a name="lg"></a>

### <a name="url-generation-with-linkgenerator"></a>LinkGenerator による URL の生成

URL 生成は、ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。 これにより、エンドポイントとそれにアクセスする URL を論理的に分離できます。

エンドポイント ルーティングには、リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) が含まれています。 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> は、DI から取得できるシングルトン サービスです。 API は、実行中の要求のコンテキスト外で使用することができます。 MVC の <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> と、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、HTML ヘルパー、[アクション結果](xref:mvc/controllers/actions)など、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> に依存するシナリオではリンク ジェネレーターを使用して、リンク生成機能を提供します。

リンク ジェネレーターは、*アドレス* と*アドレス スキーム* の概念に基づいています。 アドレス スキームは、リンク生成で考慮すべきエンドポイントを決定する方法です。 たとえば、MVC/Razor Pages からの、多くのユーザーに馴染みのあるルート名やルート値シナリオは、アドレス スキームとして実装されます。

リンク ジェネレーターでは、次の拡張メソッドを使用して、MVC/Razor Pages アクションおよびページにリンクできます。

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

これらのメソッドのオーバーロードでは、`HttpContext` を含む引数が受け入れられます。 これらのメソッドは `Url.Action` および `Url.Page` と機能的には同等ですが、柔軟性とオプションがさらに提供されます。

`GetPath*` メソッドは、絶対パスを含む URI を生成するという点で `Url.Action` および `Url.Page` に最も似ています。 `GetUri*` メソッドでは常に、スキームとホストを含む絶対 URI が生成されます。 `HttpContext` を受け入れるメソッドでは、実行中の要求のコンテキストで URI が生成されます。 実行中の要求からのアンビエント ルート値、URL ベース パス、スキーム、およびホストは、オーバーライドされない限り使用されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> はアドレスと共に呼び出されます。 URI の生成は、次の 2 つの手順で行われます。

1. アドレスは、そのアドレスと一致するエンドポイントのリストにバインドされます。
1. 各エンドポイントの `RoutePattern` は、指定された値と一致するルート パターンが見つかるまで評価されます。 結果の出力は、リンク ジェネレーターに指定された他の URI 部分と結合され、返されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> によって提供されるメソッドでは、すべての種類のアドレスの標準的なリンク生成機能がサポートされます。 リンク ジェネレーターを使用する最も便利な方法は、特定のアドレスの種類の操作を実行する拡張メソッドを使用することです。

| 拡張メソッド | 説明 |
| ---------------- | ----------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 指定された値に基づき、絶対パスを含む URI を生成します。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 指定された値に基づき、絶対 URI を生成します。             |

> [!WARNING]
> <xref:Microsoft.AspNetCore.Routing.LinkGenerator> メソッド呼び出しによる次の影響に注意してください。
>
> * 受信要求の `Host` ヘッダーが確認されないアプリ構成では、`GetUri*` 拡張メソッドは注意して使用してください。 受信要求の `Host` ヘッダーが確認されていない場合、信頼されていない要求入力を、ビュー/ページの URI でクライアントに送り返すことができます。 すべての運用アプリで、`Host` ヘッダーを既知の有効な値と照らし合わせて確認するようにサーバーを構成することをお勧めします。
>
> * ミドルウェアで `Map` または `MapWhen` と組み合わせて、<xref:Microsoft.AspNetCore.Routing.LinkGenerator> を使用する場合は注意してください。 `Map*` では、実行中の要求の基本パスが変更され、リンク生成の出力に影響します。 すべての <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API で基本パスを指定することができます。 リンク生成への `Map*` の影響を元に戻すための空の基本パスを必ず指定してください。

## <a name="endpoint-routing-differences-from-earlier-versions-of-routing"></a>エンドポイント ルーティングと以前のバージョンのルーティングとの相違点

エンドポイント ルーティングと ASP.NET Core 2.2 よりも前のバージョンのルーティングにはいくつか違いがあります。

* エンドポイント ルーティング システムでは、<xref:Microsoft.AspNetCore.Routing.Route> からの継承を含む、<xref:Microsoft.AspNetCore.Routing.IRouter> ベースの拡張機能がサポートされません。

* エンドポイント ルーティングでは [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) がサポートされません。 互換性 shim を引き続き使用するには、2.1 の[互換性バージョン](xref:mvc/compatibility-version) (`.SetCompatibilityVersion(CompatibilityVersion.Version_2_1)`) を使用します。

* エンドポイント ルーティングでは、規則ルートを使用する場合に生成される URI の大文字と小文字の指定の動作が異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のルートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "blog", new { id = 17, });
  ```

  <xref:Microsoft.AspNetCore.Routing.IRouter> ベースのルーティングでは、このコードで `/blog/ReadPost/17` の URI が生成され、指定されたルート値の大文字と小文字の指定が優先されます。 ASP.NET Core 2.2 以降のエンドポイント ルーティングでは、`/Blog/ReadPost/17` ("Blog" の先頭は大文字) が生成されます。 エンドポイント ルーティングでは `IOutboundParameterTransformer` インターフェイスが提供されます。それを使用して、この動作をグローバルにカスタマイズしたり、URL のマッピングで異なる規則を適用したりすることができます。

  詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

* 存在しないコントローラー/アクションまたはページへのリンクを試みる場合、規則ルートがある MVC/Razor Pages によって使用されるリンク生成の動作は異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のように既定のテンプレートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "Blog", new { id = 17, });
  ```

  `IRouter` ベースのルーティングでは、`BlogController` が存在しない場合や `ReadPost` アクション メソッドがない場合でも、結果は常に `/Blog/ReadPost/17` となります。 予想どおり、アクション メソッドが存在する場合は、ASP.NET Core 2.2 以降のエンドポイント ルーティングで `/Blog/ReadPost/17` が生成されます。 *しかし、アクションが存在しない場合は、エンドポイント ルーティングで空の文字列が生成されます。* 概念的には、エンドポイント ルーティングでは、アクションが存在しない場合はエンドポイントが存在するとは見なされません。

* エンドポイント ルーティングで使用する場合は、リンク生成の*アンビエント値の無効化アルゴリズム* の動作が異なります。

  *アンビエント値の無効化* は、現在実行中の要求からのルート値 (アンビエント値) のうち、どちらがリンク生成操作で使用できのかを判断するアルゴリズムです。 規則ルーティングでは、異なるアクションへのリンク時に余分なルート値が常に無効化されました。 属性ルーティングは、ASP.NET Core 2.2 のリリースより前ではこのように動作しませんでした。 以前のバージョンの ASP.NET Core では、同じルート パラメーター名を使用する別のアクションへのリンクでリンク生成エラーが発生しました。 ASP.NET Core 2.2 以降では、両方の形式のルーティングで、別のアクションへのリンク時に値が無効化されます。

  ASP.NET Core 2.1 以前での次の例について考えてみます。 別のアクション (または別のページ) にリンクする場合、好ましくない方法でルート値が再利用される可能性があります。

  */Pages/Store/Product.cshtml* の場合:

  ```cshtml
  @page "{id}"
  @Url.Page("/Login")
  ```

  */Pages/Login.cshtml* の場合:

  ```cshtml
  @page "{id?}"
  ```

  ASP.NET Core 2.1 以前で URI が `/Store/Product/18` である場合、`@Url.Page("/Login")` によって Store/Info ページに生成されるリンクは `/Login/18` となります。 リンク先が完全にアプリの異なる部分であっても、18 という `id` の値が再利用されます。 `/Login` ページのコンテキスト内の `id` ルート値は、商品 ID 値ではなく、ユーザー ID 値であると考えられます。

  ASP.NET Core 2.2 以降でのエンドポイント ルーティングでは、結果は `/Login` となります。 リンク先が異なるアクションやページである場合、アンビエント値は再利用されません。

* ラウンド トリップ ルート パラメーターの構文: 二重アスタリスク (`**`) キャッチオール パラメーター構文を使用する場合、スラッシュはエンコードされません。

  リンクの生成中に、ルーティング システムでは、スラッシュを除く二重アスタリスク (`**`) キャッチオール パラメーター (`{**myparametername}` など) でキャプチャされた値をエンコードします。 二重アスタリスク キャッチオールは、ASP.NET Core 2.2 以降の `IRouter` ベース ルーティングでサポートされます。

  以前のバージョンの ASP.NET Core での単一のアスタリスク キャッチオール パラメーター構文 (`{*myparametername}`) は引き続きサポートされ、スラッシュはエンコードされます。

  | ルート              | 以下で生成されるリンク<br>`Url.Action(new { category = "admin/products" })`&hellip; |
  | ------------------ | --------------------------------------------------------------------- |
  | `/search/{*page}`  | `/search/admin%2Fproducts` (スラッシュはエンコードされます)             |
  | `/search/{**page}` | `/search/admin/products`                                              |

### <a name="middleware-example"></a>ミドルウェアの例

次の例では、ミドルウェアで <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API を使用して、商品をリストするアクション メソッドへのリンクを作成します。 リンク ジェネレーターは、クラスに挿入し、`GenerateLink` を呼び出すことで、アプリのどのクラスでも使用できます。

```csharp
using Microsoft.AspNetCore.Routing;

public class ProductsLinkMiddleware
{
    private readonly LinkGenerator _linkGenerator;

    public ProductsLinkMiddleware(RequestDelegate next, LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        var url = _linkGenerator.GetPathByAction("ListProducts", "Store");

        httpContext.Response.ContentType = "text/plain";

        await httpContext.Response.WriteAsync($"Go to {url} to see our products.");
    }
}
```

### <a name="create-routes"></a>ルートを作成する

ほとんどのアプリは、<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> を呼び出すか、<xref:Microsoft.AspNetCore.Routing.IRouteBuilder> で定義されている同様の拡張メソッドの 1 つを呼び出してルートを作成します。 いずれの <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 拡張メソッドでも、<xref:Microsoft.AspNetCore.Routing.Route> のインスタンスが作成され、それがルート コレクションに追加されます。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> では、ルート ハンドラー パラメーターは受け入れられません。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> は、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> によって処理されるルートを追加するだけです。 MVC でのルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。

次のコード例は、一般的な ASP.NET Core MVC ルート定義によって使用される <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 呼び出しの例です。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

このテンプレートでは URL パスが照合され、ルート値が抽出されます。 たとえば、`/Products/Details/17` というパスでは、`{ controller = Products, action = Details, id = 17 }` というルート値が生成されます。

ルート値は、URL パスをセグメントに分割し、各セグメントと、ルート テンプレートの*ルート パラメーター* 名を照合することで決定されます。 ルート パラメーターが指定されます。 パラメーターは、中かっこ `{ ... }` でパラメーター名を囲むことで定義されます。

上のテンプレートでは URL パス `/` の照合も行い、`{ controller = Home, action = Index }` という値を生成します。 これは、ルート パラメーターの `{controller}` と `{action}` に既定値が与えられ、`id` ルート パラメーターが任意であるためです。 ルート パラメーター名の後の等号 (`=`) とそれに続く値により、パラメーターの既定値が定義されます。 ルート パラメーター名の後の疑問符 (`?`) により、オプションのパラメーターが定義されます。

既定値のルート パラメーターはルートが一致するとルート値を*常に*生成します。 オプションのパラメーターは、対応する URL パス セグメントがなかった場合、ルート値を生成しません。 ルート テンプレートのシナリオと構文の詳しい説明については、「[ルート テンプレート参照](#route-template-reference)」セクションを参照してください。

次の例では、ルート パラメーター定義 `{id:int}` により、`id` ルート パラメーターの[ルート制約](#route-constraint-reference)が定義されます。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

このテンプレートは `/Products/Details/Apples` ではなく、`/Products/Details/17` のような URL パスを照合します。 ルート制約は <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を実装し、ルート値を調べて正しいことを確認します。 この例では、ルート値 `id` は整数に変換できなければなりません。 フレームワークによって提供されるルート制約については、「[ルート制約参照](#route-constraint-reference)」を参照してください。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> の追加オーバーロードは、`constraints`、`dataTokens`、`defaults` の値を受け取ります。 このようなパラメーターは一般的に、匿名型のオブジェクトを渡し、匿名型のプロパティ名がルート パラメーター名と一致するというような使われ方をします。

次の <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 例では、同等のルートを作成します。

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 制約と既定値を定義するインライン構文は単純なルートの場合に便利です。 しかし、インライン構文でサポートされていない、データ トークンなどのシナリオがあります。

次の例では、その他のいくつかのシナリオを示します。

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{**article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上記のテンプレートでは `/Blog/All-About-Routing/Introduction` のような URL パスを照合し、値 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }` を抽出します。 `controller` と `action` の既定のルート値は、テンプレートに対応するルート パラメーターがなくても、ルートにより生成されます。 既定値はルート テンプレートで指定できます。 `article` ルート パラメーターは、ルート パラメーター名の前に二重アスタリスク (`**`) があるときに、*キャッチオール* として定義されます。 キャッチオール ルート パラメーターは、URL パスの残りの部分をキャプチャします。空の文字列も照合できます。

次の例では、ルート制約とデータ トークンを追加します。

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上記のテンプレートでは `/en-US/Products/5` のような URL パスを照合し、値 `{ controller = Products, action = Details, id = 5 }` とデータ トークン `{ locale = en-US }` を抽出します。

![ローカル変数ウィンドウ トークン](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>ルート クラス URL の生成

<xref:Microsoft.AspNetCore.Routing.Route> クラスは、一連のルート値をそのルート テンプレートと組み合わせる方法でも URL を生成できます。 論理的には、URL パスの照合とは逆のプロセスになります。

> [!TIP]
> URL 生成を理解する方法として、どのような URL を生成したいのかを考えてください。そして、ルート テンプレートがその URL にどのように照合されるのかを考えてください。 どのような値が生成されるでしょうか。 <xref:Microsoft.AspNetCore.Routing.Route> クラスにおける URL 生成のしくみと大体同じになります。

次の例では、一般的な ASP.NET Core MVC の既定のルートを使用します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

ルート値が `{ controller = Products, action = List }` の場合、URL `/Products/List` が生成されます。 ルート値が対応ルート パラメーターの代替となり、URL パスが作られます。 `id` は任意のルート パラメーターであるため、URL は `id` の値なしで正常に生成されます。

ルート値が `{ controller = Home, action = Index }` の場合、URL `/` が生成されます。 指定されたルート値は既定値と一致し、その既定値に対応するセグメントを省略しても問題は発生しません。

生成された両方の URL では、次のルート定義 (`/Home/Index` と `/`) でラウンドトリップされ、URL の生成に使用された同じルート値が生成されます。

> [!NOTE]
> アプリで ASP.NET Core MVC が使用される場合、ルーティングを直接呼び出す代わりに、<xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> を使用して URL を生成する必要があります。

URL 生成の詳細については、「[URL 生成参照](#url-generation-reference)」セクションを参照してください。

## <a name="use-routing-middleware"></a>ルーティング ミドルウェアの使用

アプリのプロジェクト ファイルの [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照します。

`Startup.ConfigureServices` でサービス コンテナーにルーティングを追加した例:

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

ルートは `Startup.Configure` メソッドで設定する必要があります。 サンプル アプリでは次の API を使用します。

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> &ndash; HTTP GET 要求のみを照合します。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下の表は、応答と所与の URI をまとめたものです。

| URI                    | 応答                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 要求はフォール スルーし、一致するものはありません。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 要求はフォール スルーし、HTTP GET のみが一致します。 |
| `GET /hello/Joe/Smith` | 要求はフォール スルーし、一致するものはありません。              |

このフレームワークでは、ルートを作成するために拡張メソッド セット (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) が提供されます。

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

`Map[Verb]` メソッドは制約を利用し、メソッド名の HTTP Verb にルートを制限します。 たとえば、 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> や <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>を参照してください。

## <a name="route-template-reference"></a>ルート テンプレート参照

中括弧 (`{ ... }`) 内のトークンは*ルート パラメーター*を定義します。ルートが一致した場合、これがバインドされます。 1 つのルート セグメントで複数のルート パラメーターを定義できますが、リテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。 このようなルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL パスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします (`{{` または `}}`)。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾のピリオド (`.`) は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターのプレフィックスとしてアスタリスク (`*`) または二重アスタリスク (`**`) を使用し、URI の残りの部分にバインドできます。 これらは、*キャッチオール* パラメーターと呼ばれています。 たとえば、`blog/{**slug}` は、`/blog` で始まり、その後に (`slug` ルート値に割り当てられる) 任意の値が続くあらゆる URI に一致します。 キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り (`/`) 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。 パス区切り文字をラウンドトリップさせるには、`**` ルート パラメーター プレフィックスを使用します。 `{ path = "my/path" }` のルート `foo/{**path}` では、`foo/my/path` が生成されます。

ルート パラメーターには、*既定値* が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、`id?` のように、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 オプション値と既定ルート パラメーターの違いは、既定値を含むルート パラメーターは常に値を生成するのに対し、オプションのパラメーターには、要求 URL によって値が指定されたときにのみ値が与えられることにあります。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 コロン (`:`) と制約名をルート パラメーター名の後に追加すると、ルート パラメーターの*インライン制約*が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ (`(...)`) で囲まれます。 別のコロン (`:`) と制約名を追加することで、複数のインライン制約を指定できます。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

ルート パラメーターには、リンクを生成し、ページおよびアクションを URL と一致させるときにパラメーターの値を変換する、パラメーター トランスフォーマーが含まれている場合もあります。 制約と同様に、パラメーター トランスフォーマーをルート パラメーターにインラインで追加することができます。その場合、ルート パラメーター名の後にコロン (`:`) とトランスフォーマー名を追加します。 たとえば、ルート テンプレート `blog/{article:slugify}` では、`slugify` トランスフォーマーが指定されます。 パラメーター トランスフォーマーの詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

次の表は、ルート テンプレートの例とその動作をまとめたものです。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされます (`id` は 123 に設定されます)。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます (`id` は無視されます)。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

> [!TIP]
> [ログ](xref:fundamentals/logging/index)を有効にすると、<xref:Microsoft.AspNetCore.Routing.Route> など、組み込みのルーティング実装で要求を照合するしくみを確認できます。

## <a name="reserved-routing-names"></a>ルーティングの予約名

次のキーワードは予約名であり、ルート名やパラメーターとして使用できません。

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかをはい/いいえで決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> **入力の検証**には制約を使用しないでください。 **入力の検証**に制約が使用されると、無効な入力の結果、*400 - Bad Request* と適切なエラー メッセージではなく、*404 - Not Found* が表示されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの**違いを明らかにする**ために使用されます。

次の表は、ルート制約の例とそれに求められる動作をまとめたものです。

| 制約 | 例 | 一致の例 | メモ |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`、 `-123456789` | あらゆる整数に一致する |
| `bool` | `{active:bool}` | `true`、 `FALSE` | `true` または `false` に一致する (大文字と小文字を区別しません) |
| `datetime` | `{dob:datetime}` | `2016-12-31`、 `2016-12-31 7:32pm` | 有効な `DateTime` 値に一致する (インバリアント カルチャで - 警告参照) |
| `decimal` | `{price:decimal}` | `49.99`、 `-1,000.01` | 有効な `decimal` 値に一致する (インバリアント カルチャで - 警告参照) |
| `double` | `{weight:double}` | `1.234`、 `-1,001.01e8` | 有効な `double` 値に一致する (インバリアント カルチャで - 警告参照) |
| `float` | `{weight:float}` | `1.234`、 `-1,001.01e8` | 有効な `float` 値に一致する (インバリアント カルチャで - 警告参照) |
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`、 `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 有効な `Guid` 値に一致する |
| `long` | `{ticks:long}` | `123456789`、 `-123456789` | 有効な `long` 値に一致する |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | 8 文字以内の文字列であることが必要 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 8 文字以上、16 文字以内の文字列であることが必要 |
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要 |
| `max(value)` | `{age:max(120)}` | `91` | 120 以下の整数値であることが必要 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 18 以上、120 以下の整数値であることが必要 |
| `alpha` | `{name:alpha}` | `Rick` | 1 つまたは複数のアルファベット文字で構成される文字列が必要 (`a`-`z`、大文字と小文字を区別) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 正規表現に一致する文字列が必要 (正規表現の定義についてはヒントを参照) |
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制する |

コロンで区切られた複数の制約を単一のパラメーターに適用することができます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や `DateTime` など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

## <a name="regular-expressions"></a>正規表現

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 ルーティングで正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、C# ソース ファイルで `\\` (二重円記号) 文字として文字列に `\` (単一の円記号) 文字を指定し、文字列エスケープ文字である `\` をエスケープする必要があります ([逐語的な文字列リテラル](/dotnet/csharp/language-reference/keywords/string)を使用しない限り)。 ルーティング パラメーター区切り記号文字 (`{`、`}`、`[`、`]`) をエスケープするには、表現の文字を二重にします (`{{`、`}`、`[[`、`]]`)。 次の表に、正規表現とエスケープ適用後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、キャレット (`^`) 文字で始まり、文字列の開始位置と一致します。 表現は、多くの場合、ドル記号 (`$`) で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現はルート パラメーター値全体に一致します。 `^` 文字と `$` 文字がなければ、正規表現は、文字列内のあらゆる部分文字列に一致します。それは多くの場合、意図に反することです。 下の表では例を示し、それらが一致する、または一致しない理由について説明します。

| 正規表現   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 (テンプレート内でインラインではなく) 制約ディクショナリに渡された制約が既知の制約に一致しない場合も、正規表現として扱われます。

## <a name="custom-route-constraints"></a>カスタム ルート制約

組み込みのルート制約だけでなく、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装してカスタム ルート制約を作成することができます。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスには、1 つのメソッド `Match` が含まれています。これは、制約が満たされている場合は `true` を、それ以外の場合は `false` を返します。

カスタムの <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を使うには、アプリのサービス コンテナー内にあるアプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類を登録する必要があります。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、ルート制約キーを、その制約を検証する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> の実装にマッピングするディクショナリです。 アプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

これで、制約の種類を登録するときに指定した名前を使って、通常の方法でルートに制約を適用できます。 次に例を示します。

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="parameter-transformer-reference"></a>パラメーター トランスフォーマー参照

パラメーター トランスフォーマー:

* <xref:Microsoft.AspNetCore.Routing.Route> のリンクの生成時に実行されます。
* `Microsoft.AspNetCore.Routing.IOutboundParameterTransformer`を実装します。
* <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用して構成されます。
* パラメーターのルート値を取得し、それを新しい文字列値に変換します。
* 変換された値は生成されたリンクで使用されるようになります。

たとえば、`Url.Action(new { article = "MyTestArticle" })` のルート パターン `blog\{article:slugify}` のカスタム `slugify` パラメーター トランスフォーマーでは、`blog\my-test-article` が生成されます。

ルート パターンでパラメーター トランスフォーマーを使用するには、まず `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用してこれを構成します。

```csharp
services.AddRouting(options =>
{
    // Replace the type and the name used to refer to it with your own
    // IOutboundParameterTransformer implementation
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

パラメーター トランスフォーマーは、エンドポイントが解決される URI を変換するためにフレームワークで使用されます。 たとえば、ASP.NET Core MVC ではパラメーター トランスフォーマーを使用して、`area`、`controller`、`action`、`page` を照合するために使用されるルート値を変換します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

上記のルートでは、アクション `SubscriptionManagementController.GetAll()` は URI `/subscription-management/get-all` と一致します。 パラメーター トランスフォーマーでは、リンクを生成するために使用されるルート値は変更されません。 たとえば、`Url.Action("GetAll", "SubscriptionManagement")` では `/subscription-management/get-all` が出力されます。

ASP.NET Core では、生成されたルートと共にパラメーター トランスフォーマーを使用するための API 規則が提供されます。

* ASP.NET Core MVC には、`Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention` API 規則が備わっています。 この規則では、指定されたパラメーター トランスフォーマーがアプリ内のすべての属性ルートに適用されます。 パラメーター トランスフォーマーでは、置き換えられる属性ルート トークンが変換されます。 詳細については、「[パラメーター トランスフォーマーを使用してトークンの置換をカスタマイズする](/aspnet/core/mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)」をご覧ください。
* Razor Pages には、`Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention` API 規則があります。 この規則では、指定されたパラメーター トランスフォーマーが自動で検出されたすべての Razor Pages に適用されます。 パラメーター トランスフォーマーでは、Razor Pages ルートのフォルダーとファイル名のセグメントが変換されます。 詳細については、[パラメーター トランスフォーマーを使用したページ ルートのカスタマイズ](/aspnet/core/razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)に関する記事をご覧ください。

## <a name="url-generation-reference"></a>URL 生成参照

次の例では、ルートのリンクを生成する方法を確認できます。ルート値のディクショナリと <xref:Microsoft.AspNetCore.Routing.RouteCollection> が指定されています。

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上のサンプルの終わりで生成された <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> は `/package/create/123` です。 ディクショナリにより、"Track Package Route" テンプレート `package/{operation}/{id}` のルート値 `operation` と `id` が提供されます。 詳細については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションのサンプル コードまたは[サンプル アプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)を参照してください。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> コンストラクターの 2 番目のパラメーターは*アンビエント値*の集合です。 アンビエント値は、開発者が要求コンテキスト内で指定する必要がある値の数が制限されるため、使用すると便利です。 現在の要求の現在のルート値は、リンク生成の場合、アンビエント値として見なされます。 ASP.NET Core MVC アプリの `HomeController` の `About` アクションでは、コントローラー ルート値を指定し、`Index` アクションにリンクする必要はありません。`Home` のアンビエント値が使用されます。

パラメーターに一致しないアンビエント値は無視されます。 アンビエント値は、明示的に指定された値でアンビエント値がオーバーライドされる場合にも無視されます。 URL では左から右に照合されます。

明示的に指定されているが、ルートのセグメントと一致しない値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

パラメーターに対応しない既定値がルートにあり、その値が明示的に指定される場合、それは既定値に一致する必要があります。

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

リンク生成では、`controller` と `action` について一致する値が指定されるときにのみ、このルートのリンクが生成されます。

## <a name="complex-segments"></a>複雑なセグメント

複雑なセグメント (例: `[Route("/x{token}y")]`) は、リテラルを右から左に最短一致の方法で照合することによって処理されます。 複雑なセグメントの一致方法に関する詳しい説明については、[こちらのコード](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)をご覧ください。 この[コード サンプル](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)は ASP.NET Core では使われていませんが、複雑なセグメントに関する優れた説明が提供されています。
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/aspnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

## <a name="configuring-endpoint-metadata"></a>エンドポイント メタデータの構成

次のリンクでは、エンドポイント メタデータの構成に関する情報を提供します。

* [エンドポイント ルーティングで CORS を有効にする](xref:security/cors#enable-cors-with-endpoint-routing)
* カスタム `[MinimumAgeAuthorize]` 属性を使用する [IAuthorizationPolicyProvider のサンプル](https://github.com/aspnet/AspNetCore/tree/release/3.0/src/Security/samples/CustomPolicyProvider)
* [[Authorize] 属性を使用して認証をテストする](xref:security/authentication/identity#test-identity)
* <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*>
* [[Authorize] 属性を使用してスキームを選択する](xref:security/authorization/limitingidentitybyscheme#selecting-the-scheme-with-the-authorize-attribute)
* [[Authorize] 属性を使用してポリシーを適用する](xref:security/authorization/policies#applying-policies-to-mvc-controllers)
* <xref:security/authorization/roles>

<a name="hostmatch"></a>

## <a name="host-matching-in-routes-with-requirehost"></a>RequireHost とルートが一致するホスト

`RequireHost` では、指定したホストが必要であるという制約がルートに適用されます。 `RequireHost` または `[Host]` パラメーターには、次のいずれかを指定できます。

* ホスト: `www.domain.com` (`www.domain.com` の任意のポートと一致します)
* ホストとワイルドカード: `*.domain.com` (`www.domain.com`、`subdomain.domain.com`、または `www.subdomain.domain.com` の任意のポートと一致します)
* ポート: `*:5000` (任意のホストのポート 5000 と一致します)
* ホストとポート: `www.domain.com:5000`、`*.domain.com:5000` (ホストとポートと一致します)

`RequireHost` または `[Host]` を使用して、複数のパラメーターを指定できます。 制約は、いずれかのパラメーターに対して有効なホストと一致します。 たとえば、`[Host("domain.com", "*.domain.com")]` は `domain.com`、`www.domain.com`、または `subdomain.domain.com` と一致します。

次のコードでは、`RequireHost` を使用して、指定したホストをルートに対して要求します。

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapGet("/", context => context.Response.WriteAsync("Hi Contoso!"))
            .RequireHost("contoso.com");
        endpoints.MapGet("/", context => context.Response.WriteAsync("Hi AdventureWorks!"))
            .RequireHost("adventure-works.com");
        endpoints.MapHealthChecks("/healthz").RequireHost("*:8080");
    });
}
```

次のコードでは、`[Host]` 属性を使用して、指定したホストをコントローラーに対して要求します。

```csharp
[Host("contoso.com", "adventure-works.com")]
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        return View();
    }

    [Host("example.com:8080")]
    public IActionResult Privacy()
    {
        return View();
    }

}
```

`[Host]` 属性がコントローラーとアクション メソッドの両方に適用される場合は、次のようになります。

* アクションの属性が使用されます。
* コントローラーの属性は無視されます。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

ルーティングでは、要求 URI をエンドポイントにマッピングし、受信要求をそれらのエンドポイントに配布します。 ルートはアプリに定義され、アプリの起動時に構成されます。 ルートは、要求に含まれている URL から値を任意で抽出できます。その値を要求処理に利用できます。 ルーティングでは、アプリからのルート情報を利用し、エンドポイントにマッピングする URL を生成することもできます。

ASP.NET Core 2.2 で最新のルーティング シナリオを使用するには、`Startup.ConfigureServices` で[互換性バージョン](xref:mvc/compatibility-version)を MVC サービス登録に指定します。

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

<xref:Microsoft.AspNetCore.Mvc.MvcOptions.EnableEndpointRouting> オプションでは、ルーティングで内部的にエンドポイント ベースのロジックを使用するか、ASP.NET Core 2.1 以前の <xref:Microsoft.AspNetCore.Routing.IRouter> ベースのロジックを使用する必要があるかどうかを決定します。 互換性バージョンが 2.2 以降に設定されている場合、既定値は `true` となります。 以前のルーティング ロジックを使用する場合は、値を `false` に設定します。

```csharp
// Use the routing logic of ASP.NET Core 2.1 or earlier:
services.AddMvc(options => options.EnableEndpointRouting = false)
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

<xref:Microsoft.AspNetCore.Routing.IRouter> ベースのルーティングの詳細については、[このトピックの ASP.NET Core 2.1 バージョン](/aspnet/core/fundamentals/routing?view=aspnetcore-2.1)を参照してください。

> [!IMPORTANT]
> 本文では、ASP.NET Core ルーティングについて詳しく取り上げます。 ASP.NET Core MVC ルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。 Razor Pages のルーティング規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="routing-basics"></a>ルーティングの基本

ほとんどのアプリでは、URL を読みやすくてわかりやすいものにするために、基本的なでわかりやすいルーティング スキームを選択する必要があります。 既定の規則ルート `{controller=Home}/{action=Index}/{id?}`:

* 基本的でわかりやすいルーティング スキームをサポートしています。
* UI ベースのアプリの便利な開始点となります。

開発者は一般的に、特殊な状況でのアプリのトラフィックが多い領域 (ブログや E コマース エンドポイントなど) に対しては、[属性ルーティング](xref:mvc/controllers/routing#attribute-routing)または専用規則ルートを使用して簡易ルートをさらに追加します。

Web API では、属性ルーティングを使用して、HTTP 動詞で操作を表現するリソースのセットとしてアプリの機能をモデル化する必要があります。 つまり、同じ論理リソース上の多くの操作 (たとえば GET や POST) で、同じ URL を使用します。 属性ルーティングでは、API のパブリック エンドポイント レイアウトを慎重に設計するために必要となるコントロールのレベルが提供されます。

Razor Pages アプリでは既定の規則ルーティングを使用して、アプリの *Pages* フォルダーの名前付きリソースを提供します。 Razor Pages のルーティング動作をカスタマイズできる追加の規則を使用できます。 詳細については、次のトピックを参照してください。 <xref:razor-pages/index> および <xref:razor-pages/razor-pages-conventions>

URL 生成サポートを使用すると、アプリを相互にリンクする URL をハード コーディングすることなくアプリを開発できます。 このサポートにより、基本的なルーティング構成を使用して作業を開始し、アプリのリソース レイアウトが決まった後でルートを変更することができます。

ルーティングでは*エンドポイント* (`Endpoint`) を使用して、アプリの論理エンドポイントを表します。

エンドポイントでは、要求と、任意のメタデータのコレクションを処理するデリゲートが定義されます。 メタデータは、各エンドポイントにアタッチされている構成とポリシーに基づいて横断的な関心事を実装するために使用されます。

ルーティング システムには次の特性があります。

* ルート テンプレート構文は、トークン化されたルート パラメーターでルートを定義するために使用されます。
* 規則スタイルおよび属性スタイルのエンドポイント構成が許可されます。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> は、URL パラメーターに特定のエンドポイント制約で有効な値が含まれているかどうかを確認するために使用されます。
* MVC/Razor Pages などのアプリ モデルでは、ルーティング シナリオの予測可能な実装を持つ、すべてのエンドポイントを登録します。
* ルーティングの実装では、必要に応じて、ミドルウェア パイプラインでのルーティングに関する決定を行います。
* ルーティング ミドルウェアの後に表示されるミドルウェアでは、特定の要求 URI に関するルーティング ミドルウェアのエンドポイント決定の結果を検査できます。
* ミドルウェア パイプラインの任意の場所でアプリのすべてのエンドポイントを列挙することができます。
* アプリではルーティングを使用し、エンドポイント情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。
* URL の生成は、任意の拡張機能をサポートする、アドレスに基づきます。

  * リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) は、URI を生成するために[依存関係挿入 (DI)](xref:fundamentals/dependency-injection) を使用して任意の場所で解決できます。
  * リンク ジェネレーター API を DI で使用できない場合、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> で URL を構築するためのメソッドが提供されます。

> [!NOTE]
> ASP.NET Core 2.2 のエンドポイント ルーティングのリリースでは、エンドポイント リンクは、MVC/Razor Pages アクションおよびページに制限されます。 エンドポイント リンク機能の拡張は、今後のリリースで予定されています。

ルーティングは <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> クラスにより[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに接続されます。 [ASP.NET Core MVC](xref:mvc/overview) では、ミドルウェア パイプラインにその構成の一部としてルーティングが追加され、MVC および Razor Pages アプリでのルーティングが処理されます。 スタンドアロン コンポーネントとしてルーティングを利用する方法については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションを参照してください。

### <a name="url-matching"></a>URL 一致

URL 一致というプロセスでは、ルーティングによって、受信要求が*エンドポイント*に配布されます。 このプロセスは URL パスのデータに基づきますが、要求内のあらゆるデータを考慮することもできます。 アプリの規模を大きくしたり、より複雑にしたりするとき、要求を個別のハンドラーに配布する機能が鍵となります。

エンドポイント ルーティングのルーティング システムでは、配布に関するすべての決定が行われます。 ミドルウェアでは選択されたエンドポイントに基づいてポリシーが適用されるため、セキュリティ ポリシーの適用や配布に影響する可能性のある決定は、ルーティング システム内で行うことが重要です。

エンドポイントのデリゲートが実行されると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) は、ルートから生成された*ルート値*のディクショナリです。 この値は通常、URL のトークン化で決定されます。この値を利用してユーザー入力を受け取ったり、アプリ内で決定をさらに配布したりできます。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) は、一致したルートに関連する追加データのプロパティ バッグです。 一致したルートに基づいてアプリで決定を行えるように、<xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> が提供されます。これは状態データと各ルートの関連付けをサポートするためのものです。 この値は開発者が定義するものです。ルーティングの動作に影響を与えることは**ありません**。 また、[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) に一時退避される値はどのような型でも構いません。一方、[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) の場合は、文字列間で変換できる必要があります。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) は、過去に要求に一致したルートの一覧です。 ルートはルートの中に入れ子にすることができます。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> プロパティは、結果的に一致をもたらしたルートの論理ツリーを通るパスを表します。 一般的に、<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最初の項目はルート コレクションであり、これは URL 生成に使用するものです。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最後の項目は、一致したルート ハンドラーです。

<a name="lg"></a>

### <a name="url-generation-with-linkgenerator"></a>LinkGenerator による URL の生成

URL 生成は、ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。 これにより、エンドポイントとそれにアクセスする URL を論理的に分離できます。

エンドポイント ルーティングには、リンク ジェネレーター API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) が含まれています。 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> は、DI から取得できるシングルトン サービスです。 API は、実行中の要求のコンテキスト外で使用することができます。 MVC の <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> と、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、HTML ヘルパー、[アクション結果](xref:mvc/controllers/actions)など、<xref:Microsoft.AspNetCore.Mvc.IUrlHelper> に依存するシナリオではリンク ジェネレーターを使用して、リンク生成機能を提供します。

リンク ジェネレーターは、*アドレス* と*アドレス スキーム* の概念に基づいています。 アドレス スキームは、リンク生成で考慮すべきエンドポイントを決定する方法です。 たとえば、MVC/Razor Pages からの、多くのユーザーに馴染みのあるルート名やルート値シナリオは、アドレス スキームとして実装されます。

リンク ジェネレーターでは、次の拡張メソッドを使用して、MVC/Razor Pages アクションおよびページにリンクできます。

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

これらのメソッドのオーバーロードでは、`HttpContext` を含む引数が受け入れられます。 これらのメソッドは `Url.Action` および `Url.Page` と機能的には同等ですが、柔軟性とオプションがさらに提供されます。

`GetPath*` メソッドは、絶対パスを含む URI を生成するという点で `Url.Action` および `Url.Page` に最も似ています。 `GetUri*` メソッドでは常に、スキームとホストを含む絶対 URI が生成されます。 `HttpContext` を受け入れるメソッドでは、実行中の要求のコンテキストで URI が生成されます。 実行中の要求からのアンビエント ルート値、URL ベース パス、スキーム、およびホストは、オーバーライドされない限り使用されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> はアドレスと共に呼び出されます。 URI の生成は、次の 2 つの手順で行われます。

1. アドレスは、そのアドレスと一致するエンドポイントのリストにバインドされます。
1. 各エンドポイントの `RoutePattern` は、指定された値と一致するルート パターンが見つかるまで評価されます。 結果の出力は、リンク ジェネレーターに指定された他の URI 部分と結合され、返されます。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> によって提供されるメソッドでは、すべての種類のアドレスの標準的なリンク生成機能がサポートされます。 リンク ジェネレーターを使用する最も便利な方法は、特定のアドレスの種類の操作を実行する拡張メソッドを使用することです。

| 拡張メソッド   | 説明                                                         |
| ------------------ | ------------------------------------------------------------------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 指定された値に基づき、絶対パスを含む URI を生成します。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 指定された値に基づき、絶対 URI を生成します。             |

> [!WARNING]
> <xref:Microsoft.AspNetCore.Routing.LinkGenerator> メソッド呼び出しによる次の影響に注意してください。
>
> * 受信要求の `Host` ヘッダーが確認されないアプリ構成では、`GetUri*` 拡張メソッドは注意して使用してください。 受信要求の `Host` ヘッダーが確認されていない場合、信頼されていない要求入力を、ビュー/ページの URI でクライアントに送り返すことができます。 すべての運用アプリで、`Host` ヘッダーを既知の有効な値と照らし合わせて確認するようにサーバーを構成することをお勧めします。
>
> * ミドルウェアで `Map` または `MapWhen` と組み合わせて、<xref:Microsoft.AspNetCore.Routing.LinkGenerator> を使用する場合は注意してください。 `Map*` では、実行中の要求の基本パスが変更され、リンク生成の出力に影響します。 すべての <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API で基本パスを指定することができます。 リンク生成への `Map*` の影響を元に戻すための空の基本パスを必ず指定してください。

## <a name="differences-from-earlier-versions-of-routing"></a>以前のバージョンのルーティングとの相違点

ASP.NET Core 2.2 以降でのエンドポイント ルーティングと、ASP.NET Core での以前のバージョンのルーティングには、次のようないくつかの違いがあります。

* エンドポイント ルーティング システムでは、<xref:Microsoft.AspNetCore.Routing.Route> からの継承を含む、<xref:Microsoft.AspNetCore.Routing.IRouter> ベースの拡張機能がサポートされません。

* エンドポイント ルーティングでは [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) がサポートされません。 互換性 shim を引き続き使用するには、2.1 の[互換性バージョン](xref:mvc/compatibility-version) (`.SetCompatibilityVersion(CompatibilityVersion.Version_2_1)`) を使用します。

* エンドポイント ルーティングでは、規則ルートを使用する場合に生成される URI の大文字と小文字の指定の動作が異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のルートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "blog", new { id = 17, });
  ```

  <xref:Microsoft.AspNetCore.Routing.IRouter> ベースのルーティングでは、このコードで `/blog/ReadPost/17` の URI が生成され、指定されたルート値の大文字と小文字の指定が優先されます。 ASP.NET Core 2.2 以降のエンドポイント ルーティングでは、`/Blog/ReadPost/17` ("Blog" の先頭は大文字) が生成されます。 エンドポイント ルーティングでは `IOutboundParameterTransformer` インターフェイスが提供されます。それを使用して、この動作をグローバルにカスタマイズしたり、URL のマッピングで異なる規則を適用したりすることができます。

  詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

* 存在しないコントローラー/アクションまたはページへのリンクを試みる場合、規則ルートがある MVC/Razor Pages によって使用されるリンク生成の動作は異なります。

  次の既定のルート テンプレートについて考えてみます。

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  次のように既定のテンプレートを使用して、アクションへのリンクを生成するとします。

  ```csharp
  var link = Url.Action("ReadPost", "Blog", new { id = 17, });
  ```

  `IRouter` ベースのルーティングでは、`BlogController` が存在しない場合や `ReadPost` アクション メソッドがない場合でも、結果は常に `/Blog/ReadPost/17` となります。 予想どおり、アクション メソッドが存在する場合は、ASP.NET Core 2.2 以降のエンドポイント ルーティングで `/Blog/ReadPost/17` が生成されます。 *しかし、アクションが存在しない場合は、エンドポイント ルーティングで空の文字列が生成されます。* 概念的には、エンドポイント ルーティングでは、アクションが存在しない場合はエンドポイントが存在するとは見なされません。

* エンドポイント ルーティングで使用する場合は、リンク生成の*アンビエント値の無効化アルゴリズム* の動作が異なります。

  *アンビエント値の無効化* は、現在実行中の要求からのルート値 (アンビエント値) のうち、どちらがリンク生成操作で使用できのかを判断するアルゴリズムです。 規則ルーティングでは、異なるアクションへのリンク時に余分なルート値が常に無効化されました。 属性ルーティングは、ASP.NET Core 2.2 のリリースより前ではこのように動作しませんでした。 以前のバージョンの ASP.NET Core では、同じルート パラメーター名を使用する別のアクションへのリンクでリンク生成エラーが発生しました。 ASP.NET Core 2.2 以降では、両方の形式のルーティングで、別のアクションへのリンク時に値が無効化されます。

  ASP.NET Core 2.1 以前での次の例について考えてみます。 別のアクション (または別のページ) にリンクする場合、好ましくない方法でルート値が再利用される可能性があります。

  */Pages/Store/Product.cshtml* の場合:

  ```cshtml
  @page "{id}"
  @Url.Page("/Login")
  ```

  */Pages/Login.cshtml* の場合:

  ```cshtml
  @page "{id?}"
  ```

  ASP.NET Core 2.1 以前で URI が `/Store/Product/18` である場合、`@Url.Page("/Login")` によって Store/Info ページに生成されるリンクは `/Login/18` となります。 リンク先が完全にアプリの異なる部分であっても、18 という `id` の値が再利用されます。 `/Login` ページのコンテキスト内の `id` ルート値は、商品 ID 値ではなく、ユーザー ID 値であると考えられます。

  ASP.NET Core 2.2 以降でのエンドポイント ルーティングでは、結果は `/Login` となります。 リンク先が異なるアクションやページである場合、アンビエント値は再利用されません。

* ラウンド トリップ ルート パラメーターの構文: 二重アスタリスク (`**`) キャッチオール パラメーター構文を使用する場合、スラッシュはエンコードされません。

  リンクの生成中に、ルーティング システムでは、スラッシュを除く二重アスタリスク (`**`) キャッチオール パラメーター (`{**myparametername}` など) でキャプチャされた値をエンコードします。 二重アスタリスク キャッチオールは、ASP.NET Core 2.2 以降の `IRouter` ベース ルーティングでサポートされます。

  以前のバージョンの ASP.NET Core での単一のアスタリスク キャッチオール パラメーター構文 (`{*myparametername}`) は引き続きサポートされ、スラッシュはエンコードされます。

  | ルート              | 以下で生成されるリンク<br>`Url.Action(new { category = "admin/products" })`&hellip; |
  | ------------------ | --------------------------------------------------------------------- |
  | `/search/{*page}`  | `/search/admin%2Fproducts` (スラッシュはエンコードされます)             |
  | `/search/{**page}` | `/search/admin/products`                                              |

### <a name="middleware-example"></a>ミドルウェアの例

次の例では、ミドルウェアで <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API を使用して、商品をリストするアクション メソッドへのリンクを作成します。 リンク ジェネレーターは、クラスに挿入し、`GenerateLink` を呼び出すことで、アプリのどのクラスでも使用できます。

```csharp
using Microsoft.AspNetCore.Routing;

public class ProductsLinkMiddleware
{
    private readonly LinkGenerator _linkGenerator;

    public ProductsLinkMiddleware(RequestDelegate next, LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        var url = _linkGenerator.GetPathByAction("ListProducts", "Store");

        httpContext.Response.ContentType = "text/plain";

        await httpContext.Response.WriteAsync($"Go to {url} to see our products.");
    }
}
```

### <a name="create-routes"></a>ルートを作成する

ほとんどのアプリは、<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> を呼び出すか、<xref:Microsoft.AspNetCore.Routing.IRouteBuilder> で定義されている同様の拡張メソッドの 1 つを呼び出してルートを作成します。 いずれの <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 拡張メソッドでも、<xref:Microsoft.AspNetCore.Routing.Route> のインスタンスが作成され、それがルート コレクションに追加されます。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> では、ルート ハンドラー パラメーターは受け入れられません。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> は、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> によって処理されるルートを追加するだけです。 MVC でのルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。

次のコード例は、一般的な ASP.NET Core MVC ルート定義によって使用される <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 呼び出しの例です。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

このテンプレートでは URL パスが照合され、ルート値が抽出されます。 たとえば、`/Products/Details/17` というパスでは、`{ controller = Products, action = Details, id = 17 }` というルート値が生成されます。

ルート値は、URL パスをセグメントに分割し、各セグメントと、ルート テンプレートの*ルート パラメーター* 名を照合することで決定されます。 ルート パラメーターが指定されます。 パラメーターは、中かっこ `{ ... }` でパラメーター名を囲むことで定義されます。

上のテンプレートでは URL パス `/` の照合も行い、`{ controller = Home, action = Index }` という値を生成します。 これは、ルート パラメーターの `{controller}` と `{action}` に既定値が与えられ、`id` ルート パラメーターが任意であるためです。 ルート パラメーター名の後の等号 (`=`) とそれに続く値により、パラメーターの既定値が定義されます。 ルート パラメーター名の後の疑問符 (`?`) により、オプションのパラメーターが定義されます。

既定値のルート パラメーターはルートが一致するとルート値を*常に*生成します。 オプションのパラメーターは、対応する URL パス セグメントがなかった場合、ルート値を生成しません。 ルート テンプレートのシナリオと構文の詳しい説明については、「[ルート テンプレート参照](#route-template-reference)」セクションを参照してください。

次の例では、ルート パラメーター定義 `{id:int}` により、`id` ルート パラメーターの[ルート制約](#route-constraint-reference)が定義されます。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

このテンプレートは `/Products/Details/Apples` ではなく、`/Products/Details/17` のような URL パスを照合します。 ルート制約は <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を実装し、ルート値を調べて正しいことを確認します。 この例では、ルート値 `id` は整数に変換できなければなりません。 フレームワークによって提供されるルート制約については、「[ルート制約参照](#route-constraint-reference)」を参照してください。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> の追加オーバーロードは、`constraints`、`dataTokens`、`defaults` の値を受け取ります。 このようなパラメーターは一般的に、匿名型のオブジェクトを渡し、匿名型のプロパティ名がルート パラメーター名と一致するというような使われ方をします。

次の <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 例では、同等のルートを作成します。

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 制約と既定値を定義するインライン構文は単純なルートの場合に便利です。 しかし、インライン構文でサポートされていない、データ トークンなどのシナリオがあります。

次の例では、その他のいくつかのシナリオを示します。

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{**article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上記のテンプレートでは `/Blog/All-About-Routing/Introduction` のような URL パスを照合し、値 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }` を抽出します。 `controller` と `action` の既定のルート値は、テンプレートに対応するルート パラメーターがなくても、ルートにより生成されます。 既定値はルート テンプレートで指定できます。 `article` ルート パラメーターは、ルート パラメーター名の前に二重アスタリスク (`**`) があるときに、*キャッチオール* として定義されます。 キャッチオール ルート パラメーターは、URL パスの残りの部分をキャプチャします。空の文字列も照合できます。

次の例では、ルート制約とデータ トークンを追加します。

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上記のテンプレートでは `/en-US/Products/5` のような URL パスを照合し、値 `{ controller = Products, action = Details, id = 5 }` とデータ トークン `{ locale = en-US }` を抽出します。

![ローカル変数ウィンドウ トークン](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>ルート クラス URL の生成

<xref:Microsoft.AspNetCore.Routing.Route> クラスは、一連のルート値をそのルート テンプレートと組み合わせる方法でも URL を生成できます。 論理的には、URL パスの照合とは逆のプロセスになります。

> [!TIP]
> URL 生成を理解する方法として、どのような URL を生成したいのかを考えてください。そして、ルート テンプレートがその URL にどのように照合されるのかを考えてください。 どのような値が生成されるでしょうか。 <xref:Microsoft.AspNetCore.Routing.Route> クラスにおける URL 生成のしくみと大体同じになります。

次の例では、一般的な ASP.NET Core MVC の既定のルートを使用します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

ルート値が `{ controller = Products, action = List }` の場合、URL `/Products/List` が生成されます。 ルート値が対応ルート パラメーターの代替となり、URL パスが作られます。 `id` は任意のルート パラメーターであるため、URL は `id` の値なしで正常に生成されます。

ルート値が `{ controller = Home, action = Index }` の場合、URL `/` が生成されます。 指定されたルート値は既定値と一致し、その既定値に対応するセグメントを省略しても問題は発生しません。

生成された両方の URL では、次のルート定義 (`/Home/Index` と `/`) でラウンドトリップされ、URL の生成に使用された同じルート値が生成されます。

> [!NOTE]
> アプリで ASP.NET Core MVC が使用される場合、ルーティングを直接呼び出す代わりに、<xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> を使用して URL を生成する必要があります。

URL 生成の詳細については、「[URL 生成参照](#url-generation-reference)」セクションを参照してください。

## <a name="use-routing-middleware"></a>ルーティング ミドルウェアの使用

アプリのプロジェクト ファイルの [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照します。

`Startup.ConfigureServices` でサービス コンテナーにルーティングを追加した例:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

ルートは `Startup.Configure` メソッドで設定する必要があります。 サンプル アプリでは次の API を使用します。

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> &ndash; HTTP GET 要求のみを照合します。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下の表は、応答と所与の URI をまとめたものです。

| URI                    | 応答                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 要求はフォール スルーし、一致するものはありません。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 要求はフォール スルーし、HTTP GET のみが一致します。 |
| `GET /hello/Joe/Smith` | 要求はフォール スルーし、一致するものはありません。              |

このフレームワークでは、ルートを作成するために拡張メソッド セット (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) が提供されます。

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

`Map[Verb]` メソッドは制約を利用し、メソッド名の HTTP Verb にルートを制限します。 たとえば、 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> や <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>を参照してください。

## <a name="route-template-reference"></a>ルート テンプレート参照

中括弧 (`{ ... }`) 内のトークンは*ルート パラメーター*を定義します。ルートが一致した場合、これがバインドされます。 1 つのルート セグメントで複数のルート パラメーターを定義できますが、リテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。 このようなルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL パスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします (`{{` または `}}`)。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾のピリオド (`.`) は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターのプレフィックスとしてアスタリスク (`*`) または二重アスタリスク (`**`) を使用し、URI の残りの部分にバインドできます。 これらは、*キャッチオール* パラメーターと呼ばれています。 たとえば、`blog/{**slug}` は、`/blog` で始まり、その後に (`slug` ルート値に割り当てられる) 任意の値が続くあらゆる URI に一致します。 キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り (`/`) 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。 パス区切り文字をラウンドトリップさせるには、`**` ルート パラメーター プレフィックスを使用します。 `{ path = "my/path" }` のルート `foo/{**path}` では、`foo/my/path` が生成されます。

ルート パラメーターには、*既定値* が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、`id?` のように、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 オプション値と既定ルート パラメーターの違いは、既定値を含むルート パラメーターは常に値を生成するのに対し、オプションのパラメーターには、要求 URL によって値が指定されたときにのみ値が与えられることにあります。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 コロン (`:`) と制約名をルート パラメーター名の後に追加すると、ルート パラメーターの*インライン制約*が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ (`(...)`) で囲まれます。 別のコロン (`:`) と制約名を追加することで、複数のインライン制約を指定できます。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

ルート パラメーターには、リンクを生成し、ページおよびアクションを URL と一致させるときにパラメーターの値を変換する、パラメーター トランスフォーマーが含まれている場合もあります。 制約と同様に、パラメーター トランスフォーマーをルート パラメーターにインラインで追加することができます。その場合、ルート パラメーター名の後にコロン (`:`) とトランスフォーマー名を追加します。 たとえば、ルート テンプレート `blog/{article:slugify}` では、`slugify` トランスフォーマーが指定されます。 パラメーター トランスフォーマーの詳細については、「[パラメーター トランスフォーマー参照](#parameter-transformer-reference)」セクションを参照してください。

次の表は、ルート テンプレートの例とその動作をまとめたものです。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされます (`id` は 123 に設定されます)。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます (`id` は無視されます)。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

> [!TIP]
> [ログ](xref:fundamentals/logging/index)を有効にすると、<xref:Microsoft.AspNetCore.Routing.Route> など、組み込みのルーティング実装で要求を照合するしくみを確認できます。

## <a name="reserved-routing-names"></a>ルーティングの予約名

次のキーワードは予約名であり、ルート名やパラメーターとして使用できません。

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかをはい/いいえで決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> **入力の検証**には制約を使用しないでください。 **入力の検証**に制約が使用されると、無効な入力の結果、*400 - Bad Request* と適切なエラー メッセージではなく、*404 - Not Found* が表示されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの**違いを明らかにする**ために使用されます。

次の表は、ルート制約の例とそれに求められる動作をまとめたものです。

| 制約 | 例 | 一致の例 | メモ |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`、 `-123456789` | あらゆる整数に一致する |
| `bool` | `{active:bool}` | `true`、 `FALSE` | `true` または `false` に一致する (大文字と小文字を区別しません) |
| `datetime` | `{dob:datetime}` | `2016-12-31`、 `2016-12-31 7:32pm` | 有効な `DateTime` 値に一致する (インバリアント カルチャで - 警告参照) |
| `decimal` | `{price:decimal}` | `49.99`、 `-1,000.01` | 有効な `decimal` 値に一致する (インバリアント カルチャで - 警告参照) |
| `double` | `{weight:double}` | `1.234`、 `-1,001.01e8` | 有効な `double` 値に一致する (インバリアント カルチャで - 警告参照) |
| `float` | `{weight:float}` | `1.234`、 `-1,001.01e8` | 有効な `float` 値に一致する (インバリアント カルチャで - 警告参照) |
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`、 `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 有効な `Guid` 値に一致する |
| `long` | `{ticks:long}` | `123456789`、 `-123456789` | 有効な `long` 値に一致する |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | 8 文字以内の文字列であることが必要 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 8 文字以上、16 文字以内の文字列であることが必要 |
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要 |
| `max(value)` | `{age:max(120)}` | `91` | 120 以下の整数値であることが必要 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 18 以上、120 以下の整数値であることが必要 |
| `alpha` | `{name:alpha}` | `Rick` | 1 つまたは複数のアルファベット文字で構成される文字列が必要 (`a`-`z`、大文字と小文字を区別) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 正規表現に一致する文字列が必要 (正規表現の定義についてはヒントを参照) |
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制する |

コロンで区切られた複数の制約を単一のパラメーターに適用することができます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や `DateTime` など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

## <a name="regular-expressions"></a>正規表現

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 ルーティングで正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、C# ソース ファイルで `\\` (二重円記号) 文字として文字列に `\` (単一の円記号) 文字を指定し、文字列エスケープ文字である `\` をエスケープする必要があります ([逐語的な文字列リテラル](/dotnet/csharp/language-reference/keywords/string)を使用しない限り)。 ルーティング パラメーター区切り記号文字 (`{`、`}`、`[`、`]`) をエスケープするには、表現の文字を二重にします (`{{`、`}`、`[[`、`]]`)。 次の表に、正規表現とエスケープ適用後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、キャレット (`^`) 文字で始まり、文字列の開始位置と一致します。 表現は、多くの場合、ドル記号 (`$`) で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現はルート パラメーター値全体に一致します。 `^` 文字と `$` 文字がなければ、正規表現は、文字列内のあらゆる部分文字列に一致します。それは多くの場合、意図に反することです。 下の表では例を示し、それらが一致する、または一致しない理由について説明します。

| 正規表現   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 (テンプレート内でインラインではなく) 制約ディクショナリに渡された制約が既知の制約に一致しない場合も、正規表現として扱われます。

## <a name="custom-route-constraints"></a>カスタム ルート制約

組み込みのルート制約だけでなく、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装してカスタム ルート制約を作成することができます。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスには、1 つのメソッド `Match` が含まれています。これは、制約が満たされている場合は `true` を、それ以外の場合は `false` を返します。

カスタムの <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を使うには、アプリのサービス コンテナー内にあるアプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類を登録する必要があります。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、ルート制約キーを、その制約を検証する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> の実装にマッピングするディクショナリです。 アプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

これで、制約の種類を登録するときに指定した名前を使って、通常の方法でルートに制約を適用できます。 次に例を示します。

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="parameter-transformer-reference"></a>パラメーター トランスフォーマー参照

パラメーター トランスフォーマー:

* <xref:Microsoft.AspNetCore.Routing.Route> のリンクの生成時に実行されます。
* `Microsoft.AspNetCore.Routing.IOutboundParameterTransformer`を実装します。
* <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用して構成されます。
* パラメーターのルート値を取得し、それを新しい文字列値に変換します。
* 変換された値は生成されたリンクで使用されるようになります。

たとえば、`Url.Action(new { article = "MyTestArticle" })` のルート パターン `blog\{article:slugify}` のカスタム `slugify` パラメーター トランスフォーマーでは、`blog\my-test-article` が生成されます。

ルート パターンでパラメーター トランスフォーマーを使用するには、まず `Startup.ConfigureServices` で <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> を使用してこれを構成します。

```csharp
services.AddRouting(options =>
{
    // Replace the type and the name used to refer to it with your own
    // IOutboundParameterTransformer implementation
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

パラメーター トランスフォーマーは、エンドポイントが解決される URI を変換するためにフレームワークで使用されます。 たとえば、ASP.NET Core MVC ではパラメーター トランスフォーマーを使用して、`area`、`controller`、`action`、`page` を照合するために使用されるルート値を変換します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

上記のルートでは、アクション `SubscriptionManagementController.GetAll()` は URI `/subscription-management/get-all` と一致します。 パラメーター トランスフォーマーでは、リンクを生成するために使用されるルート値は変更されません。 たとえば、`Url.Action("GetAll", "SubscriptionManagement")` では `/subscription-management/get-all` が出力されます。

ASP.NET Core では、生成されたルートと共にパラメーター トランスフォーマーを使用するための API 規則が提供されます。

* ASP.NET Core MVC には、`Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention` API 規則が備わっています。 この規則では、指定されたパラメーター トランスフォーマーがアプリ内のすべての属性ルートに適用されます。 パラメーター トランスフォーマーでは、置き換えられる属性ルート トークンが変換されます。 詳細については、「[パラメーター トランスフォーマーを使用してトークンの置換をカスタマイズする](/aspnet/core/mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)」をご覧ください。
* Razor Pages には、`Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention` API 規則があります。 この規則では、指定されたパラメーター トランスフォーマーが自動で検出されたすべての Razor Pages に適用されます。 パラメーター トランスフォーマーでは、Razor Pages ルートのフォルダーとファイル名のセグメントが変換されます。 詳細については、[パラメーター トランスフォーマーを使用したページ ルートのカスタマイズ](/aspnet/core/razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)に関する記事をご覧ください。

## <a name="url-generation-reference"></a>URL 生成参照

次の例では、ルートのリンクを生成する方法を確認できます。ルート値のディクショナリと <xref:Microsoft.AspNetCore.Routing.RouteCollection> が指定されています。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上のサンプルの終わりで生成された <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> は `/package/create/123` です。 ディクショナリにより、"Track Package Route" テンプレート `package/{operation}/{id}` のルート値 `operation` と `id` が提供されます。 詳細については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションのサンプル コードまたは[サンプル アプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)を参照してください。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> コンストラクターの 2 番目のパラメーターは*アンビエント値*の集合です。 アンビエント値は、開発者が要求コンテキスト内で指定する必要がある値の数が制限されるため、使用すると便利です。 現在の要求の現在のルート値は、リンク生成の場合、アンビエント値として見なされます。 ASP.NET Core MVC アプリの `HomeController` の `About` アクションでは、コントローラー ルート値を指定し、`Index` アクションにリンクする必要はありません。`Home` のアンビエント値が使用されます。

パラメーターに一致しないアンビエント値は無視されます。 アンビエント値は、明示的に指定された値でアンビエント値がオーバーライドされる場合にも無視されます。 URL では左から右に照合されます。

明示的に指定されているが、ルートのセグメントと一致しない値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

パラメーターに対応しない既定値がルートにあり、その値が明示的に指定される場合、それは既定値に一致する必要があります。

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

リンク生成では、`controller` と `action` について一致する値が指定されるときにのみ、このルートのリンクが生成されます。

## <a name="complex-segments"></a>複雑なセグメント

複雑なセグメント (例: `[Route("/x{token}y")]`) は、リテラルを右から左に最短一致の方法で照合することによって処理されます。 複雑なセグメントの一致方法に関する詳しい説明については、[こちらのコード](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)をご覧ください。 この[コード サンプル](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)は ASP.NET Core では使われていませんが、複雑なセグメントに関する優れた説明が提供されています。
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/aspnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ルーティングでは要求 URI をルート ハンドラーにマッピングし、受信要求を配布します。 ルートはアプリに定義され、アプリの起動時に構成されます。 ルートは、要求に含まれている URL から値を任意で抽出できます。その値を要求処理に利用できます。 ルーティングでは、アプリからの構成済みルートを使用して、ルート ハンドラーにマッピングする URL を生成できます。

ASP.NET Core 2.1 で最新のルーティング シナリオを使用するには、`Startup.ConfigureServices` で[互換性バージョン](xref:mvc/compatibility-version)を MVC サービス登録に指定します。

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

> [!IMPORTANT]
> 本文では、ASP.NET Core ルーティングについて詳しく取り上げます。 ASP.NET Core MVC ルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。 Razor Pages のルーティング規則については、「<xref:razor-pages/razor-pages-conventions>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="routing-basics"></a>ルーティングの基本

ほとんどのアプリでは、URL を読みやすくてわかりやすいものにするために、基本的なでわかりやすいルーティング スキームを選択する必要があります。 既定の規則ルート `{controller=Home}/{action=Index}/{id?}`:

* 基本的でわかりやすいルーティング スキームをサポートしています。
* UI ベースのアプリの便利な開始点となります。

開発者は一般的に、特殊な状況でのアプリのトラフィックが多い領域 (ブログや E コマース エンドポイントなど) に対しては、[属性ルーティング](xref:mvc/controllers/routing#attribute-routing)または専用規則ルートを使用して簡易ルートをさらに追加します。

Web API では、属性ルーティングを使用して、HTTP 動詞で操作を表現するリソースのセットとしてアプリの機能をモデル化する必要があります。 つまり、同じ論理リソース上の多くの操作 (たとえば GET や POST) で、同じ URL を使用します。 属性ルーティングでは、API のパブリック エンドポイント レイアウトを慎重に設計するために必要となるコントロールのレベルが提供されます。

Razor Pages アプリでは既定の規則ルーティングを使用して、アプリの *Pages* フォルダーの名前付きリソースを提供します。 Razor Pages のルーティング動作をカスタマイズできる追加の規則を使用できます。 詳細については、次のトピックを参照してください。 <xref:razor-pages/index> および <xref:razor-pages/razor-pages-conventions>

URL 生成サポートを使用すると、アプリを相互にリンクする URL をハード コーディングすることなくアプリを開発できます。 このサポートにより、基本的なルーティング構成を使用して作業を開始し、アプリのリソース レイアウトが決まった後でルートを変更することができます。

ルーティングにおける*ルート* (<xref:Microsoft.AspNetCore.Routing.IRouter> の実装) の利用目的:

* 受信要求を*ルート ハンドラー*にマッピングする。
* 応答で使用される URL を生成する。

既定では、アプリにはルート コレクションが 1 つあります。 要求が到着すると、コレクション内のルートは、コレクションに存在する順序で処理されます。 フレームワークでは、コレクション内の各ルートに対して <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> メソッドを呼び出し、受信要求 URL とコレクション内のルートとの照合を試みます。 応答ではルーティングを使用し、ルート情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。

ルーティング システムには次の特性があります。

* ルート テンプレート構文は、トークン化されたルート パラメーターでルートを定義するために使用されます。
* 規則スタイルおよび属性スタイルのエンドポイント構成が許可されます。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> は、URL パラメーターに特定のエンドポイント制約で有効な値が含まれているかどうかを確認するために使用されます。
* MVC/Razor Pages などのアプリ モデルでは、ルーティング シナリオの予測可能な実装を持つ、すべてのルートを登録します。
* 応答ではルーティングを使用し、ルート情報に基づいて URL (リダイレクトやリンクなど) を生成できます。そのため、URL をハードコーディングする必要がなく、保守管理が容易になります。
* URL の生成は、任意の拡張機能をサポートする、ルートに基づきます。 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> では、URL を構築するためのメソッドが提供されます。

ルーティングは <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> クラスにより[ミドルウェア](xref:fundamentals/middleware/index) パイプラインに接続されます。 [ASP.NET Core MVC](xref:mvc/overview) では、ミドルウェア パイプラインにその構成の一部としてルーティングが追加され、MVC および Razor Pages アプリでのルーティングが処理されます。 スタンドアロン コンポーネントとしてルーティングを利用する方法については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションを参照してください。

### <a name="url-matching"></a>URL 一致

URL 一致というプロセスでは、ルーティングによって、受信要求が*ハンドラー*に配布されます。 このプロセスは URL パスのデータに基づきますが、要求内のあらゆるデータを考慮することもできます。 アプリの規模を大きくしたり、より複雑にしたりするとき、要求を個別のハンドラーに配布する機能が鍵となります。

受信要求は <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> に入ります。これが各ルートで順に <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> メソッドを呼び出します。 <xref:Microsoft.AspNetCore.Routing.IRouter> インスタンスでは、[RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler*) を非 null の <xref:Microsoft.AspNetCore.Http.RequestDelegate> に設定することで、要求を*処理する*かどうかを選択します。 要求に適したハンドラーがルートに見つかると、ルート処理が停止します。ハンドラーが呼び出され、要求を処理します。 要求を処理するためのルート ハンドラーが見つからない場合、ミドルウェアによって、要求が要求パイプラインの次のミドルウェアに渡されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> に最初に入力されるのが、現在の要求に関連付けられている [RouteContext.HttpContext](xref:Microsoft.AspNetCore.Routing.RouteContext.HttpContext*) です。 [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler) と [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData*) は、ルートが一致した後に設定される出力です。

また、<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> を呼び出す一致が見つかると、それまでに実行された要求処理に基づいて、[RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) のプロパティが適切な値に設定されます。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) は、ルートから生成された*ルート値*のディクショナリです。 この値は通常、URL のトークン化で決定されます。この値を利用してユーザー入力を受け取ったり、アプリ内で決定をさらに配布したりできます。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) は、一致したルートに関連する追加データのプロパティ バッグです。 一致したルートに基づいてアプリで決定を行えるように、<xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> が提供されます。これは状態データと各ルートの関連付けをサポートするためのものです。 この値は開発者が定義するものです。ルーティングの動作に影響を与えることは**ありません**。 また、[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) に一時退避される値はどのような型でも構いません。一方、[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) の場合は、文字列間で変換できる必要があります。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) は、過去に要求に一致したルートの一覧です。 ルートはルートの中に入れ子にすることができます。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> プロパティは、結果的に一致をもたらしたルートの論理ツリーを通るパスを表します。 一般的に、<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最初の項目はルート コレクションであり、これは URL 生成に使用するものです。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> の最後の項目は、一致したルート ハンドラーです。

<a name="lg"></a>

### <a name="url-generation"></a>URL 生成

URL 生成は、ルーティングにおいて、一連のルート値に基づいて URL パスを作成するプロセスです。 これにより、ルート ハンドラーとそれにアクセスする URL を論理的に分離できます。

URL 生成は同様の繰り返しプロセスに従いますが、最初にユーザーまたはフレームワーク コードでルート コレクションの <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> メソッドを呼び出します。 非 null の <xref:Microsoft.AspNetCore.Routing.VirtualPathData> が返されるまで、各*ルート* で順番に <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> メソッドが呼び出されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の第一入力:

* [VirtualPathContext.HttpContext](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext)
* [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values)
* [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues)

ルートでは <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> と <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> によって指定されるルート値を主に使用し、URL を生成できるかどうかと、どの値を含めるかを判断します。 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> は、現在の要求を照合することで生成された一連のルート値です。 対照的に、<xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> は、現在の操作に適した URL の生成方法を指定するルート値です。 ルートで現在のコンテキストに関連付けられているサービスまたは追加データを取得する必要がある場合、<xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext> が指定されます。

> [!TIP]
> [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values*) は [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues*) のオーバーライド セットであると考えてください。 URL 生成では、同じルートまたはルート値を利用することでリンクの URL 生成を生成するために、現在の要求からのルート値の再利用が試行されます。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の出力は <xref:Microsoft.AspNetCore.Routing.VirtualPathData> です。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> は <xref:Microsoft.AspNetCore.Routing.RouteData> の並列です。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> には出力 URL として <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> が含まれ、ルートで設定する追加プロパティがいくつか含まれます。

[VirtualPathData.VirtualPath](xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath*) プロパティには、ルートによって生成された*仮想パス*が含まれます。 必要に応じて、パスをさらに処理する必要がある場合があります。 生成された URL を HTML で表示するには、アプリの基礎パスを先頭に追加する必要があります。

[VirtualPathData.Router](xref:Microsoft.AspNetCore.Routing.VirtualPathData.Router*) は URL を正常に生成したルートの参照です。

[VirtualPathData.DataTokens](xref:Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens*) プロパティは、URL を生成したルートに関連する追加データのディクショナリです。 これは [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) の並列です。

### <a name="create-routes"></a>ルートを作成する

ルーティングにより、<xref:Microsoft.AspNetCore.Routing.IRouter> の標準実装として <xref:Microsoft.AspNetCore.Routing.Route> クラスが与えられます。 <xref:Microsoft.AspNetCore.Routing.Route> では*ルート テンプレート* 構文を利用して、<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> の呼び出し時に URL パスに対して照合するパターンを定義します。 <xref:Microsoft.AspNetCore.Routing.Route> は同じルート テンプレートを利用し、<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> の呼び出し時に URL を生成します。

ほとんどのアプリは、<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> を呼び出すか、<xref:Microsoft.AspNetCore.Routing.IRouteBuilder> で定義されている同様の拡張メソッドの 1 つを呼び出してルートを作成します。 いずれの <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 拡張メソッドでも、<xref:Microsoft.AspNetCore.Routing.Route> のインスタンスが作成され、それがルート コレクションに追加されます。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> では、ルート ハンドラー パラメーターは受け入れられません。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> は、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> によって処理されるルートを追加するだけです。 既定のハンドラーは `IRouter` であり、そのハンドラーで要求が処理されない可能性があります。 たとえば、ASP.NET Core MVC は通常、利用できるコントローラーやアクションに一致する要求のみを処理する既定のハンドラーとして構成されます。 MVC でのルーティングの詳細については、「<xref:mvc/controllers/routing>」を参照してください。

次のコード例は、一般的な ASP.NET Core MVC ルート定義によって使用される <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 呼び出しの例です。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

このテンプレートでは URL パスが照合され、ルート値が抽出されます。 たとえば、`/Products/Details/17` というパスでは、`{ controller = Products, action = Details, id = 17 }` というルート値が生成されます。

ルート値は、URL パスをセグメントに分割し、各セグメントと、ルート テンプレートの*ルート パラメーター* 名を照合することで決定されます。 ルート パラメーターが指定されます。 パラメーターは、中かっこ `{ ... }` でパラメーター名を囲むことで定義されます。

上のテンプレートでは URL パス `/` の照合も行い、`{ controller = Home, action = Index }` という値を生成します。 これは、ルート パラメーターの `{controller}` と `{action}` に既定値が与えられ、`id` ルート パラメーターが任意であるためです。 ルート パラメーター名の後の等号 (`=`) とそれに続く値により、パラメーターの既定値が定義されます。 ルート パラメーター名の後の疑問符 (`?`) により、オプションのパラメーターが定義されます。

既定値のルート パラメーターはルートが一致するとルート値を*常に*生成します。 オプションのパラメーターは、対応する URL パス セグメントがなかった場合、ルート値を生成しません。 ルート テンプレートのシナリオと構文の詳しい説明については、「[ルート テンプレート参照](#route-template-reference)」セクションを参照してください。

次の例では、ルート パラメーター定義 `{id:int}` により、`id` ルート パラメーターの[ルート制約](#route-constraint-reference)が定義されます。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

このテンプレートは `/Products/Details/Apples` ではなく、`/Products/Details/17` のような URL パスを照合します。 ルート制約は <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を実装し、ルート値を調べて正しいことを確認します。 この例では、ルート値 `id` は整数に変換できなければなりません。 フレームワークによって提供されるルート制約については、「[ルート制約参照](#route-constraint-reference)」を参照してください。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> の追加オーバーロードは、`constraints`、`dataTokens`、`defaults` の値を受け取ります。 このようなパラメーターは一般的に、匿名型のオブジェクトを渡し、匿名型のプロパティ名がルート パラメーター名と一致するというような使われ方をします。

次の <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 例では、同等のルートを作成します。

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 制約と既定値を定義するインライン構文は単純なルートの場合に便利です。 しかし、インライン構文でサポートされていない、データ トークンなどのシナリオがあります。

次の例では、その他のいくつかのシナリオを示します。

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上記のテンプレートでは `/Blog/All-About-Routing/Introduction` のような URL パスを照合し、値 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }` を抽出します。 `controller` と `action` の既定のルート値は、テンプレートに対応するルート パラメーターがなくても、ルートにより生成されます。 既定値はルート テンプレートで指定できます。 `article` ルート パラメーターは、ルート パラメーター名の前にアスタリスク (`*`) があるときに、*キャッチオール* として定義されます。 キャッチオール ルート パラメーターは、URL パスの残りの部分をキャプチャします。空の文字列も照合できます。

次の例では、ルート制約とデータ トークンを追加します。

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上記のテンプレートでは `/en-US/Products/5` のような URL パスを照合し、値 `{ controller = Products, action = Details, id = 5 }` とデータ トークン `{ locale = en-US }` を抽出します。

![ローカル変数ウィンドウ トークン](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>ルート クラス URL の生成

<xref:Microsoft.AspNetCore.Routing.Route> クラスは、一連のルート値をそのルート テンプレートと組み合わせる方法でも URL を生成できます。 論理的には、URL パスの照合とは逆のプロセスになります。

> [!TIP]
> URL 生成を理解する方法として、どのような URL を生成したいのかを考えてください。そして、ルート テンプレートがその URL にどのように照合されるのかを考えてください。 どのような値が生成されるでしょうか。 <xref:Microsoft.AspNetCore.Routing.Route> クラスにおける URL 生成のしくみと大体同じになります。

次の例では、一般的な ASP.NET Core MVC の既定のルートを使用します。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

ルート値が `{ controller = Products, action = List }` の場合、URL `/Products/List` が生成されます。 ルート値が対応ルート パラメーターの代替となり、URL パスが作られます。 `id` は任意のルート パラメーターであるため、URL は `id` の値なしで正常に生成されます。

ルート値が `{ controller = Home, action = Index }` の場合、URL `/` が生成されます。 指定されたルート値は既定値と一致し、その既定値に対応するセグメントを省略しても問題は発生しません。

生成された両方の URL では、次のルート定義 (`/Home/Index` と `/`) でラウンドトリップされ、URL の生成に使用された同じルート値が生成されます。

> [!NOTE]
> アプリで ASP.NET Core MVC が使用される場合、ルーティングを直接呼び出す代わりに、<xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> を使用して URL を生成する必要があります。

URL 生成の詳細については、「[URL 生成参照](#url-generation-reference)」セクションを参照してください。

## <a name="use-routing-middleware"></a>ルーティング ミドルウェアの使用

アプリのプロジェクト ファイルの [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)を参照します。

`Startup.ConfigureServices` でサービス コンテナーにルーティングを追加した例:

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

ルートは `Startup.Configure` メソッドで設定する必要があります。 サンプル アプリでは次の API を使用します。

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> &ndash; HTTP GET 要求のみを照合します。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下の表は、応答と所与の URI をまとめたものです。

| URI                    | 応答                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 要求はフォール スルーし、一致するものはありません。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 要求はフォール スルーし、HTTP GET のみが一致します。 |
| `GET /hello/Joe/Smith` | 要求はフォール スルーし、一致するものはありません。              |

ルートを 1 つ構成する場合、<xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*> を呼び出し、`IRouter` インスタンスを渡します。 <xref:Microsoft.AspNetCore.Routing.RouteBuilder> を使用する必要はありません。

このフレームワークでは、ルートを作成するために拡張メソッド セット (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) が提供されます。

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> など、リストされているメソッドの一部では、<xref:Microsoft.AspNetCore.Http.RequestDelegate> が必要です。 <xref:Microsoft.AspNetCore.Http.RequestDelegate> は、ルートが一致したとき、*ルート ハンドラー*として使用されます。 この仲間の他のメソッドでは、ルート ハンドラーとして使用されるミドルウェア パイプラインを構成できます。 `Map*` メソッドで、<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*> などのハンドラーを受け入れない場合、<xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> が使用されます。

`Map[Verb]` メソッドは制約を利用し、メソッド名の HTTP Verb にルートを制限します。 たとえば、 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> や <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>を参照してください。

## <a name="route-template-reference"></a>ルート テンプレート参照

中括弧 (`{ ... }`) 内のトークンは*ルート パラメーター*を定義します。ルートが一致した場合、これがバインドされます。 1 つのルート セグメントで複数のルート パラメーターを定義できますが、リテラル値で区切る必要があります。 たとえば、`{controller=Home}{action=Index}` は有効なルートになりません。`{controller}` と `{action}` の間にリテラル値がないためです。 このようなルート パラメーターには名前を付ける必要があります。付加的な属性を指定することもあります。

ルート パラメーター以外のリテラル テキスト (`{id}` など) とパス区切り `/` は URL のテキストに一致する必要があります。 テキスト照合は復号された URL パスを基盤とし、大文字と小文字が区別されます。 リテラル ルート パラメーターの区切り記号 (`{` または `}`) を照合するには、文字を繰り返して区切り記号をエスケープします (`{{` または `}}`)。

任意のファイル拡張子が付いたファイル名のキャプチャを試行する URL パターンには、追加の考慮事項があります。 たとえば、テンプレート `files/{filename}.{ext?}` について考えてみます。 `filename` と `ext` の両方の値が存在するときに、両方の値が入力されます。 URL に `filename` の値だけが存在する場合、末尾のピリオド (`.`) は任意であるため、このルートは一致となります。 次の URL はこのルートに一致します。

* `/files/myFile.txt`
* `/files/myFile`

ルート パラメーターのプレフィックスとしてアスタリスク (`*`) を使用し、URI の残りの部分にバインドできます。 これは*キャッチオール* パラメーターと呼ばれています。 たとえば、`blog/{*slug}` は、`/blog` で始まり、その後に (`slug` ルート値に割り当てられる) 任意の値が続くあらゆる URI に一致します。 キャッチオール パラメーターは空の文字列に一致することもあります。

キャッチオール パラメーターでは、パス区切り (`/`) 文字を含め、URL の生成にルートが使用されるときに適切な文字がエスケープされます。 たとえば、ルート値が `{ path = "my/path" }` のルート `foo/{*path}` では、`foo/my%2Fpath` が生成されます。 エスケープされたスラッシュに注意してください。

ルート パラメーターには、*既定値* が含まれることがあります。パラメーター名の後に既定値を指定し、等号 (`=`) で区切ることで指定されます。 たとえば、`{controller=Home}` では、`controller` の既定値として `Home` が定義されます。 パラメーターの URL に値がない場合、既定値が使用されます。 ルート パラメーターは、`id?` のように、パラメーター名の終わりに疑問符 (`?`) を追加することでオプションとして扱われます。 オプション値と既定ルート パラメーターの違いは、既定値を含むルート パラメーターは常に値を生成するのに対し、オプションのパラメーターには、要求 URL によって値が指定されたときにのみ値が与えられることにあります。

ルート パラメーターには、URL からバインドされるルート値に一致しなければならないという制約が含まれることがあります。 コロン (`:`) と制約名をルート パラメーター名の後に追加すると、ルート パラメーターの*インライン制約*が指定されます。 その制約で引数が要求される場合、制約名の後にかっこ (`(...)`) で囲まれます。 別のコロン (`:`) と制約名を追加することで、複数のインライン制約を指定できます。

制約名と引数が <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> サービスに渡され、URL 処理で使用する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> のインスタンスが作成されます。 たとえば、ルート テンプレート `blog/{article:minlength(10)}` によって、制約 `minlength` と引数 `10` が指定されます。 ルート制約の詳細とこのフレームワークによって指定される制約のリストについては、「[ルート制約参照](#route-constraint-reference)」セクションを参照してください。

次の表は、ルート テンプレートの例とその動作をまとめたものです。

| ルート テンプレート                           | 一致する URI の例    | 要求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 単一パス `/hello` にのみ一致します。                                     |
| `{Page=Home}`                            | `/`                     | 一致し、`Page` が `Home` に設定されます。                                         |
| `{Page=Home}`                            | `/Contact`              | 一致し、`Page` が `Contact` に設定されます。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | `Products` コントローラーと `List` アクションにマッピングされます。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | `Products` コントローラーと `Details` アクションにマッピングされます (`id` は 123 に設定されます)。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | `Home` コントローラーと `Index` メソッドにマッピングされます (`id` は無視されます)。        |

一般的に、テンプレートの利用が最も簡単なルーティングの手法となります。 ルート テンプレート以外では、制約と既定値も指定できます。

> [!TIP]
> [ログ](xref:fundamentals/logging/index)を有効にすると、<xref:Microsoft.AspNetCore.Routing.Route> など、組み込みのルーティング実装で要求を照合するしくみを確認できます。

## <a name="reserved-routing-names"></a>ルーティングの予約名

次のキーワードは予約名であり、ルート名やパラメーターとして使用できません。

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>ルート制約参照

ルート制約は、受信 URL と一致し、URL パスがルート値にトークン化されたときに実行されます。 ルート制約では、通常、ルート テンプレート経由で関連付けられるルート値を調べ、値が許容できるかどうかをはい/いいえで決定します。 一部のルート制約では、ルート値以外のデータを使用し、要求をルーティングできるかどうかが考慮されます。 たとえば、<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> はその HTTP Verb に基づいて要求を承認または却下します。 制約は、要求のルーティングとリンクの生成で使用されます。

> [!WARNING]
> **入力の検証**には制約を使用しないでください。 **入力の検証**に制約が使用されると、無効な入力の結果、*400 - Bad Request* と適切なエラー メッセージではなく、*404 - Not Found* が表示されます。 ルート制約は、特定のルートに対する入力の妥当性を検証するためではなく、似たようなルートの**違いを明らかにする**ために使用されます。

次の表は、ルート制約の例とそれに求められる動作をまとめたものです。

| 制約 | 例 | 一致の例 | メモ |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`、 `-123456789` | あらゆる整数に一致する |
| `bool` | `{active:bool}` | `true`、 `FALSE` | `true` または `false` に一致する (大文字と小文字を区別しません) |
| `datetime` | `{dob:datetime}` | `2016-12-31`、 `2016-12-31 7:32pm` | 有効な `DateTime` 値に一致する (インバリアント カルチャで - 警告参照) |
| `decimal` | `{price:decimal}` | `49.99`、 `-1,000.01` | 有効な `decimal` 値に一致する (インバリアント カルチャで - 警告参照) |
| `double` | `{weight:double}` | `1.234`、 `-1,001.01e8` | 有効な `double` 値に一致する (インバリアント カルチャで - 警告参照) |
| `float` | `{weight:float}` | `1.234`、 `-1,001.01e8` | 有効な `float` 値に一致する (インバリアント カルチャで - 警告参照) |
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`、 `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 有効な `Guid` 値に一致する |
| `long` | `{ticks:long}` | `123456789`、 `-123456789` | 有効な `long` 値に一致する |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 4 文字以上の文字列であることが必要 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | 8 文字以内の文字列であることが必要 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 厳密に 12 文字の文字列であることが必要 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 8 文字以上、16 文字以内の文字列であることが必要 |
| `min(value)` | `{age:min(18)}` | `19` | 18 以上の整数値であることが必要 |
| `max(value)` | `{age:max(120)}` | `91` | 120 以下の整数値であることが必要 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 18 以上、120 以下の整数値であることが必要 |
| `alpha` | `{name:alpha}` | `Rick` | 1 つまたは複数のアルファベット文字で構成される文字列が必要 (`a`-`z`、大文字と小文字を区別) |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 正規表現に一致する文字列が必要 (正規表現の定義についてはヒントを参照) |
| `required` | `{name:required}` | `Rick` | URL 生成中、非パラメーターが提示されるように強制する |

コロンで区切られた複数の制約を単一のパラメーターに適用することができます。 たとえば、次の制約では、パラメーターが 1 以上の整数値に制限されます。

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> URL の妥当性を検証し、CLR 型 (`int` や `DateTime` など) に変換されるルート制約では、常にインバリアント カルチャが使用されます。 これらの制約では、URL がローカライズ不可であることが前提となります。 フレームワークから提供されるルート制約がルート値に格納されている値を変更することはありません。 URL から解析されたルート値はすべて文字列として格納されます。 たとえば、`float` 制約はルート値を浮動小数に変換しますが、変換された値は、浮動小数に変換できることを検証するためにだけ利用されます。

## <a name="regular-expressions"></a>正規表現

ASP.NET Core フレームワークでは、正規表現コンストラクターに `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant` が追加されます。 これらのメンバーの詳細については、「<xref:System.Text.RegularExpressions.RegexOptions>」を参照してください。

正規表現では、ルーティングや C# 言語で使用されるものに似た区切り記号とトークンが使用されます。 正規表現トークンはエスケープする必要があります。 ルーティングで正規表現 `^\d{3}-\d{2}-\d{4}$` を使用するには、C# ソース ファイルで `\\` (二重円記号) 文字として文字列に `\` (単一の円記号) 文字を指定し、文字列エスケープ文字である `\` をエスケープする必要があります ([逐語的な文字列リテラル](/dotnet/csharp/language-reference/keywords/string)を使用しない限り)。 ルーティング パラメーター区切り記号文字 (`{`、`}`、`[`、`]`) をエスケープするには、表現の文字を二重にします (`{{`、`}`、`[[`、`]]`)。 次の表に、正規表現とエスケープ適用後のものを示します。

| 正規表現    | エスケープ適用後の正規表現     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

ルーティングで使用される正規表現は、多くの場合、キャレット (`^`) 文字で始まり、文字列の開始位置と一致します。 表現は、多くの場合、ドル記号 (`$`) で終わり、文字列の末尾と一致します。 `^` 文字と `$` 文字により、正規表現はルート パラメーター値全体に一致します。 `^` 文字と `$` 文字がなければ、正規表現は、文字列内のあらゆる部分文字列に一致します。それは多くの場合、意図に反することです。 下の表では例を示し、それらが一致する、または一致しない理由について説明します。

| 正規表現   | String    | 一致したもの | コメント               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | 123abc456 | はい   | サブ文字列の一致     |
| `[a-z]{2}`   | mz        | はい   | 一致する表現    |
| `[a-z]{2}`   | MZ        | はい   | 大文字と小文字の使い方が違う    |
| `^[a-z]{2}$` | hello     | いいえ    | 上の `^` と `$` を参照 |
| `^[a-z]{2}$` | 123abc456 | いいえ    | 上の `^` と `$` を参照 |

正規表現構文の詳細については、[.NET Framework 正規表現](/dotnet/standard/base-types/regular-expression-language-quick-reference)に関するページを参照してください。

既知の入力可能値の集まりにパラメーターを制限するには、正規表現を使用します。 たとえば、`{action:regex(^(list|get|create)$)}` の場合、`action` ルート値は `list`、`get`、`create` とのみ照合されます。 制約ディクショナリに渡された場合、文字列 `^(list|get|create)$` で同じものになります。 (テンプレート内でインラインではなく) 制約ディクショナリに渡された制約が既知の制約に一致しない場合も、正規表現として扱われます。

## <a name="custom-route-constraints"></a>カスタム ルート制約

組み込みのルート制約だけでなく、<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスを実装してカスタム ルート制約を作成することができます。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> インターフェイスには、1 つのメソッド `Match` が含まれています。これは、制約が満たされている場合は `true` を、それ以外の場合は `false` を返します。

カスタムの <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> を使うには、アプリのサービス コンテナー内にあるアプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> に、ルート制約の種類を登録する必要があります。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、ルート制約キーを、その制約を検証する <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> の実装にマッピングするディクショナリです。 アプリの <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> は、`Startup.ConfigureServices` で、[services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 呼び出しの一部として、または `services.Configure<RouteOptions>` を使って <xref:Microsoft.AspNetCore.Routing.RouteOptions> を直接構成することで、更新できます。 次に例を示します。

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

これで、制約の種類を登録するときに指定した名前を使って、通常の方法でルートに制約を適用できます。 次に例を示します。

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="url-generation-reference"></a>URL 生成参照

次の例では、ルートのリンクを生成する方法を確認できます。ルート値のディクショナリと <xref:Microsoft.AspNetCore.Routing.RouteCollection> が指定されています。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上のサンプルの終わりで生成された <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> は `/package/create/123` です。 ディクショナリにより、"Track Package Route" テンプレート `package/{operation}/{id}` のルート値 `operation` と `id` が提供されます。 詳細については、「[ルーティング ミドルウェアの使用](#use-routing-middleware)」セクションのサンプル コードまたは[サンプル アプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)を参照してください。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> コンストラクターの 2 番目のパラメーターは*アンビエント値*の集合です。 アンビエント値は、開発者が要求コンテキスト内で指定する必要がある値の数が制限されるため、使用すると便利です。 現在の要求の現在のルート値は、リンク生成の場合、アンビエント値として見なされます。 ASP.NET Core MVC アプリの `HomeController` の `About` アクションでは、コントローラー ルート値を指定し、`Index` アクションにリンクする必要はありません。`Home` のアンビエント値が使用されます。

パラメーターに一致しないアンビエント値は無視されます。 アンビエント値は、明示的に指定された値でアンビエント値がオーバーライドされる場合にも無視されます。 URL では左から右に照合されます。

明示的に指定されているが、ルートのセグメントと一致しない値は、クエリ文字列に追加されます。 次の表は、ルート テンプレート `{controller}/{action}/{id?}` の使用時の結果をまとめたものです。

| アンビエント値                     | 明示的な値                        | 結果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| controller = "Home"                | action = "About"                       | `/Home/About`           |
| controller = "Home"                | controller = "Order", action = "About" | `/Order/About`          |
| controller = "Home", color = "Red" | action = "About"                       | `/Home/About`           |
| controller = "Home"                | action = "About", color = "Red"        | `/Home/About?color=Red` |

パラメーターに対応しない既定値がルートにあり、その値が明示的に指定される場合、それは既定値に一致する必要があります。

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

リンク生成では、`controller` と `action` について一致する値が指定されるときにのみ、このルートのリンクが生成されます。

## <a name="complex-segments"></a>複雑なセグメント

複雑なセグメント (例: `[Route("/x{token}y")]`) は、リテラルを右から左に最短一致の方法で照合することによって処理されます。 複雑なセグメントの一致方法に関する詳しい説明については、[こちらのコード](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)をご覧ください。 この[コード サンプル](https://github.com/aspnet/AspNetCore/blob/release/2.2/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)は ASP.NET Core では使われていませんが、複雑なセグメントに関する優れた説明が提供されています。
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/aspnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

::: moniker-end
