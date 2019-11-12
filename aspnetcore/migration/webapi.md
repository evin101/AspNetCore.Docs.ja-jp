---
title: ASP.NET Web API から ASP.NET Core に移行します。
author: ardalis
description: ASP.NET 4.x Web API から ASP.NET Core mvc web API の実装を移行する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 12/10/2018
uid: migration/webapi
ms.openlocfilehash: 74c7730a667ebc979241489733cdace149cacdf2
ms.sourcegitcommit: d6e51c60439f03a8992bda70cc982ddb15d3f100
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/03/2019
ms.locfileid: "67555743"
---
# <a name="migrate-from-aspnet-web-api-to-aspnet-core"></a>ASP.NET Web API から ASP.NET Core に移行します。

作成者: [Scott Addie](https://twitter.com/scott_addie)と[Steve Smith](https://ardalis.com/)

ASP.NET 4.x Web API は、クライアント、ブラウザーやモバイル デバイスなどの広範な範囲に到達する HTTP サービスです。 ASP.NET Core は ASP.NET 4.x の MVC を統一し、Web API アプリを ASP.NET Core MVC と呼ばれる単純なプログラミング モデルにモデル化します。 この記事では、ASP.NET 4.x Web API から ASP.NET Core MVC への移行に必要な手順を示します。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/migration/webapi/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>必須コンポーネント

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs2019-2.2.md)]

## <a name="review-aspnet-4x-web-api-project"></a>ASP.NET 4.x Web API プロジェクトを確認してください。

この記事では、開始点として、 *ProductsApp*で作成したプロジェクト[ASP.NET Web API 2 の概要](/aspnet/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)します。 そのプロジェクトでは、単純な ASP.NET 4.x Web API プロジェクトを次のように構成します。

*Global.asax.cs*、呼び出しに`WebApiConfig.Register`:

[!code-csharp[](webapi/sample/ProductsApp/Global.asax.cs?highlight=14)]

`WebApiConfig`クラスがある、 *App_Start*フォルダーが静的と`Register`メソッド。

[!code-csharp[](webapi/sample/ProductsApp/App_Start/WebApiConfig.cs)]

このクラスを構成します[属性ルーティング](/aspnet/web-api/overview/web-api-routing-and-actions/attribute-routing-in-web-api-2)、実際には、プロジェクトで使用されています。 ASP.NET Web API で使用される、ルーティング テーブルも構成します。 ASP.NET 4.x Web API Url の形式に一致が必要ですがこの場合、`/api/{controller}/{id}`で`{id}`で省略可能します。

*ProductsApp*プロジェクトには、1 つのコント ローラーが含まれています。 コント ローラーが継承`ApiController`2 つのアクションが含まれています。

[!code-csharp[](webapi/sample/ProductsApp/Controllers/ProductsController.cs?highlight=28,33)]

`Product`によって使用されるモデル`ProductsController`は単純なクラスです。

[!code-csharp[](webapi/sample/ProductsApp/Models/Product.cs)]

次のセクションでは、ASP.NET Core mvc Web API プロジェクトの移行について説明します。

## <a name="create-destination-project"></a>コピー先のプロジェクトを作成します。

Visual Studio で、次の手順を完了するには。

* 移動して**ファイル** > **新しい** > **プロジェクト** > **他のプロジェクト タイプ** > **Visual Studio ソリューション**します。 選択**空のソリューション**、ソリューションの名前と*WebAPIMigration*します。 をクリックして、 **OK**ボタンをクリックします。
* 既存の追加*ProductsApp*プロジェクトがソリューションにします。
* 新しい追加**ASP.NET Core Web アプリケーション**プロジェクトがソリューションにします。 選択、 **.NET Core**ドロップダウン リストからフレームワークをターゲットにして、選択、 **API**プロジェクト テンプレート。 プロジェクトに名前を*ProductsCore*、 をクリックし、 **OK**ボタン。

ソリューションには、2 つのプロジェクトが含まれるようになりました。 次のセクションでは、移行について説明します、 *ProductsApp*プロジェクトのコンテンツを*ProductsCore*プロジェクト。

## <a name="migrate-configuration"></a>構成を移行します。

ASP.NET Core を使用していない、 *App_Start*フォルダーまたは*Global.asax*ファイル、および*web.config*発行時にファイルが追加します。 *Startup.cs*の置換は、 *Global.asax*プロジェクトのルートにあるとします。 `Startup`クラスはすべてのアプリのスタートアップ タスクを処理します。 詳細については、「 <xref:fundamentals/startup> 」を参照してください。

ASP.NET Core mvc で属性ルーティングは、既定で含まれてとき<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc*>で呼び出される`Startup.Configure`します。 次`UseMvc`置換を呼び出し、 *ProductsApp*プロジェクトの*App_Start/WebApiConfig.cs*ファイル。

[!code-csharp[](webapi/sample/ProductsCore/Startup.cs?name=snippet_Configure&highlight=13])]

## <a name="migrate-models-and-controllers"></a>モデルとコント ローラーを移行します。

経由でコピー、 *ProductApp*プロジェクトのコント ローラーと、モデルを使用します。 この場合は、以下の手順に従ってください。

1. コピー *Controllers/ProductsController.cs*から元のプロジェクトに新しいものです。
1. 全体をコピーします*モデル*フォルダーを元のプロジェクトから新しいものにします。
1. 新しいプロジェクトの名前に一致するように、コピーしたファイルの名前空間を変更 (*ProductsCore*)。 調整、`using ProductsApp.Models;`ステートメント*ProductsController.cs*すぎます。

この時点では、コンパイル エラーの数のアプリの結果を構築します。 ASP.NET Core で、次のコンポーネントが存在しないため、エラーが発生します。

* `ApiController` クラス
* `System.Web.Http` 名前空間
* `IHttpActionResult` インターフェイス

次のように、エラーを修正します。

1. 変更`ApiController`に<xref:Microsoft.AspNetCore.Mvc.ControllerBase>します。 追加`using Microsoft.AspNetCore.Mvc;`解決するのには、`ControllerBase`参照。
1. `using System.Web.Http;`を削除します。
1. 変更、`GetProduct`アクションの戻り値の型から`IHttpActionResult`に`ActionResult<Product>`します。

簡略化、`GetProduct`アクションの`return`次のステートメント。

```csharp
return product;
```

## <a name="configure-routing"></a>ルーティングを構成します。

次のようにルーティングを構成します。

1. 装飾、`ProductsController`クラスで、次の属性。

    ```csharp
    [Route("api/[controller]")]
    [ApiController]
    ```

    上記の[[ルート]](xref:Microsoft.AspNetCore.Mvc.RouteAttribute)属性は、コント ローラーの属性のルーティング パターンを構成します。 [[ApiController]](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)属性は、このコント ローラーで、すべてのアクションの要件をルーティングする属性。

    などのトークンをサポートする属性ルーティング`[controller]`と`[action]`します。 時に、各トークンに置き換えられますコント ローラーまたはアクションの名前、属性が適用されています。 トークンは、プロジェクト魔法の文字列の数を減らします。 トークンには、ルートに対応するコント ローラーとの同期を維持し、自動リファクタリングの名前を変更する際のアクションが適用されることも確認してください。
1. ASP.NET Core 2.2 には、プロジェクトの互換性モードを設定します。

    [!code-csharp[](webapi/sample/ProductsCore/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

    上記の変更:

    * 使用するために必要な`[ApiController]`コント ローラー レベルでの属性。
    * 可能性のある ASP.NET Core 2.2 で導入された動作を分割することにオプトインします。
1. HTTP Get 要求を有効にする、`ProductController`アクション。
    * 適用、 [[HttpGet]](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute)属性を`GetAllProducts`アクション。
    * 適用、`[HttpGet("{id}")]`属性を`GetProduct`アクション。

上記の変更と未使用の削除後`using`ステートメント、 *ProductsController.cs*ファイルは、次のようになります。

[!code-csharp[](webapi/sample/ProductsCore/Controllers/ProductsController.cs)]

移行されたプロジェクトを実行しを参照`/api/products`します。 3 つの製品の完全な一覧が表示されます。 `/api/products/1` を参照します。 最初の製品が表示されます。

## <a name="compatibility-shim"></a>互換性 shim

[Microsoft.AspNetCore.Mvc.WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim)ライブラリは ASP.NET 4.x Web API プロジェクトを ASP.NET Core に移動する互換性 shim を提供します。 互換性 shim は、多くの ASP.NET 4.x Web API 2 の規則をサポートするために ASP.NET Core を拡張します。 このドキュメントで既に移植サンプルは、基本互換性 shim は必要なかったことです。 大規模なプロジェクトは、互換性 shim を使用してできる一時的に ASP.NET Core と ASP.NET 4.x Web API 2 API のギャップを埋めるに便利です。

Web API の互換性 shim は、移行する大規模な ASP.NET 4.x Web API プロジェクトを ASP.NET Core をサポートするために、一時的な措置として使用します。 時間の経過と共に ASP.NET Core のパターンを使用して、互換性 shim を使用する代わりにプロジェクトを更新する必要があります。

含まれる互換性機能`Microsoft.AspNetCore.Mvc.WebApiCompatShim`が含まれます。

* 追加、`ApiController`コント ローラーの基本型を更新する必要があるように入力します。
* Web API スタイルのモデル バインドを有効にします。 ASP.NET Core MVC のモデル バインドの関数と同様に ASP.NET の 4.x MVC 5 の場合は、既定では。 互換性 shim の変更はモデル バインドを ASP.NET 4.x Web API 2 のモデル バインドの規則に似ています。 たとえば、複合型は、要求本文から自動的にバインドされます。
* コント ローラー アクションは、型のパラメーターを実行できるように、モデル バインドを拡張`HttpRequestMessage`します。
* 型の結果を返すアクションを許可するメッセージ フォーマッタを追加します。`HttpResponseMessage`します。
* Web API 2 の操作が応答を処理するために使用する追加の応答メソッドを追加します。
  * `HttpResponseMessage` ジェネレーター。
    * `CreateResponse<T>`
    * `CreateErrorResponse`
  * 結果のアクション メソッド:
    * `BadRequestErrorMessageResult`
    * `ExceptionResult`
    * `InternalServerErrorResult`
    * `InvalidModelStateResult`
    * `NegotiatedContentResult`
    * `ResponseMessageResult`
* インスタンスを追加`IContentNegotiator`アプリへの依存関係の挿入 (DI) コンテナーと、コンテンツ ネゴシエーションに関連する型から使用できるように[Microsoft.AspNet.WebApi.Client](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client/)します。 このような種類の例として、`DefaultContentNegotiator`と`MediaTypeFormatter`します。

互換性 shim を使用するには。

1. インストール、 [Microsoft.AspNetCore.Mvc.WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim) NuGet パッケージ。
1. 互換性 shim のサービスを呼び出すことによって、アプリの DI コンテナーを登録`services.AddMvc().AddWebApiConventions()`で`Startup.ConfigureServices`します。
1. Web API に固有のルーティングを使用して定義`MapWebApiRoute`上、`IRouteBuilder`アプリの`IApplicationBuilder.UseMvc`呼び出します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:web-api/index>
* <xref:web-api/action-return-types>
* <xref:mvc/compatibility-version>
