---
title: Swashbuckle と ASP.NET Core の概要
author: zuckerthoben
description: Swashbuckle を ASP.NET Core Web API プロジェクトに追加し、Swagger UI を統合する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 08/21/2019
uid: tutorials/get-started-with-swashbuckle
ms.openlocfilehash: d3cef72de22e54f7e65ddf9f1446eb32256d0c71
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2019
ms.locfileid: "71924979"
---
# <a name="get-started-with-swashbuckle-and-aspnet-core"></a>Swashbuckle と ASP.NET Core の概要

作成者: [Shayne Boyer](https://twitter.com/spboyer) および [Scott Addie](https://twitter.com/Scott_Addie)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Swashbuckle には 3 つの主要なコンポーネントがあります。

* [Swashbuckle.AspNetCore.Swagger](https://www.nuget.org/packages/Swashbuckle.AspNetCore.Swagger/): `SwaggerDocument` オブジェクトを JSON エンドポイントとして公開するための Swagger オブジェクト モデルとミドルウェア。

* [Swashbuckle.AspNetCore.SwaggerGen](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerGen/): ルート、コントローラー、モデルから直接 `SwaggerDocument` オブジェクトを構築する Swagger ジェネレーター。 通常、Swagger エンドポイント ミドルウェアと組み合わせて Swagger JSON を自動的に公開します。

* [Swashbuckle.AspNetCore.SwaggerUI](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerUI/): Swagger UI ツールの埋め込みバージョン。 Swagger JSON を解釈して、Web API の機能を説明するカスタマイズ可能な機能豊富なエクスペリエンスを構築します。 これには、パブリック メソッド用の組み込みのテスト ハーネスが含まれます。

## <a name="package-installation"></a>パッケージ インストール

Swashbuckle は、次の方法で追加できます。

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* **[パッケージ マネージャー コンソール]** ウィンドウから:
  * **[ビュー]**  >  **[Other Windows]** \(その他の Windows\) >  **[パッケージ マネージャー コンソール]** に移動します。
  * *TodoApi.csproj* ファイルが存在するディレクトリに移動します。
  * 次のコマンドを実行します。

    ```powershell
    Install-Package Swashbuckle.AspNetCore -Version 5.0.0-rc4
    ```

* **[NuGet パッケージの管理]** ダイアログ ボックスから:
  * **[ソリューション エクスプローラー]**  >  **[NuGet パッケージの管理]** でプロジェクトを右クリックします。
  * **パッケージ ソース**を "nuget.org" に設定します。
  * [プレリリースを含める] オプションが有効になっていることを確認します
  * 検索ボックスに「Swashbuckle.AspNetCore」と入力します。
  * **[参照]** タブから最新の "Swashbuckle.AspNetCore"パッケージを選択して、 **[インストール]** をクリックします

### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* **[Solution Pad]**  >  **[パッケージを追加]** で [*パッケージ*] フォルダーを右クリックします。
* **[パッケージを追加]** ウィンドウの **[ソース]** ドロップダウンを "nuget.org" に設定します。
* [プレリリースパッケージを表示する] オプションが有効になっていることを確認します
* 検索ボックスに「Swashbuckle.AspNetCore」と入力します。
* 結果ウィンドウから最新の Swashbuckle.AspNetCore パッケージを選択し、 **[パッケージを追加]** をクリックします

### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

**統合端末**からから次のコマンドを実行します。

```dotnetcli
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore -v 5.0.0-rc4
```

### <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

次のコマンドを実行します。

```dotnetcli
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore -v 5.0.0-rc4
```

---

## <a name="add-and-configure-swagger-middleware"></a>Swagger ミドルウェアを追加して構成する

`Startup` クラスで、次の名前空間をインポートし、`OpenApiInfo` クラスを使用します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_InfoClassNamespace)]

`Startup.ConfigureServices` メソッドで Swagger ジェネレーターをサービス コレクションに追加します。

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=8-11)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=9-12)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=8-11)]

::: moniker-end

`Startup.Configure` メソッドで、生成された JSON ドキュメントと Swagger UI 対応のミドルウェアを有効にします。

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,8-11)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,8-11)]

::: moniker-end

前述の `UseSwaggerUI` メソッド呼び出しにより、[静的ファイル ミドルウェア](xref:fundamentals/static-files)が有効になります。 .NET Framework または .NET Core 1.x を対象とする場合は、[Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) NuGet パッケージをプロジェクトに追加します。

アプリを起動し、`http://localhost:<port>/swagger/v1/swagger.json` に移動します。 [Swagger 仕様 (swagger.json)](xref:tutorials/web-api-help-pages-using-swagger#swagger-specification-swaggerjson) に基づき、エンドポイントについて説明するドキュメントが生成され、表示されます。

Swagger UI は `http://localhost:<port>/swagger` にあります。 Swagger UI から API を探し、他のプログラムに組み込みます。

> [!TIP]
> アプリのルート (`http://localhost:<port>/`) で Swagger UI にサービスを提供するには、`RoutePrefix` プロパティを空の文字列に設定します。
>
> [!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup3.cs?name=snippet_UseSwaggerUI&highlight=4)]

IIS やリバース プロキシを使用するディレクトリを使用している場合は、`./` プレフィックスを使用して Swagger のエンドポイントを相対パスに設定します。 たとえば、`./swagger/v1/swagger.json` のようにします。 `/swagger/v1/swagger.json` を使用することで、アプリが URL の真のルート (使用されている場合は、これに加えてルート プレフィックス) にある JSON ファイルを検索するように指示されます。 たとえば、`http://localhost:<port>/<virtual_directory>/<route_prefix>/swagger/v1/swagger.json` の代わりに `http://localhost:<port>/<route_prefix>/swagger/v1/swagger.json` を使用します。

## <a name="customize-and-extend"></a>カスタマイズと拡張

Swagger は、テーマに合わせてオブジェクト モデルを文書化して、UI をカスタマイズするオプションを提供します。

`Startup` クラスに、次の名前空間を追加します。

```csharp
using System;
using System.Reflection;
using System.IO;
```

### <a name="api-info-and-description"></a>API 情報と説明

`AddSwaggerGen` メソッドに渡された構成アクションによって、作成者、ライセンス、説明などの情報が追加されます。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup4.cs?name=snippet_AddSwaggerGen)]

Swagger UI には、バージョンの情報が表示されます。

![バージョン情報を含む Swagger UI: 説明、作成者、およびその他のリンクの表示](web-api-help-pages-using-swagger/_static/custom-info.png)

### <a name="xml-comments"></a>XML コメント

XML コメントは、次の方法で有効にすることができます。

#### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-2.0"

* **ソリューション エクスプローラー**でプロジェクトを右クリックし、 **[<project_name>.csproj の編集]** を選択します。
* 強調表示された行を手動で *.csproj* ファイルに追加します。

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* **ソリューション エクスプローラー**でプロジェクトを右クリックして、 **[プロパティ]** を選択します。
* **[ビルド]** タブの **[出力]** セクションの下にある **[XML ドキュメント ファイル]** チェック ボックスをオンにします。

::: moniker-end

#### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

::: moniker range=">= aspnetcore-2.0"

* *Solution Pad* から **Ctrl** キーを押しながらプロジェクト名をクリックします。 **[ツール]**  >  **[ファイルの編集]** に移動します。
* 強調表示された行を手動で *.csproj* ファイルに追加します。

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* **[プロジェクト オプション]** ダイアログ > **[ビルド]** > **[コンパイラ]** を開きます。
* **[全般オプション]** セクションの下にある **[XML ドキュメントを生成する]** チェック ボックスをオンにします。

::: moniker-end

#### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

強調表示された行を手動で *.csproj* ファイルに追加します。

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

#### <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

強調表示された行を手動で *.csproj* ファイルに追加します。

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

---

XML コメントを有効にすると、ドキュメントに未記載のパブリック型とメンバーのデバッグ情報を提供することができます。 文書化されない型とメンバーは警告メッセージで示されます。 たとえば、次のメッセージは警告コード 1591 の違反を示します。

```text
warning CS1591: Missing XML comment for publicly visible type or member 'TodoController.GetAll()'
```

プロジェクト全体で警告を非表示にするには、プロジェクト ファイルで無視する警告コードの一覧をセミコロン区切りで定義します。 警告コードを `$(NoWarn);` に追加すると、[C# の既定値](https://github.com/dotnet/sdk/blob/2eb6c546931b5bcb92cd3128b93932a980553ea1/src/Tasks/Microsoft.NET.Build.Tasks/targets/Microsoft.NET.Sdk.CSharp.props#L16)も適用されます。

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

特定のメンバーに向けた警告のみを非表示にするには、コードを [#pragma warning](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-pragma-warning) プリプロセッサ ディレクティブで囲みます。 この方法は、API ドキュメント経由で公開すべきではないコードの場合に役立ちます。次の例では、警告コード CS1591 が `Program` クラス全体で無視されます。 警告コードの強制は、クラス定義の終了時に復元されます。 コンマ区切りのリストで複数の警告コードを指定します。

```csharp
namespace TodoApi
{
#pragma warning disable CS1591
    public class Program
    {
        public static void Main(string[] args) =>
            BuildWebHost(args).Run();

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
#pragma warning restore CS1591
}
```

上記の手順で生成される XML ファイルを使用するように、Swagger を構成します。 Linux または Windows 以外のオペレーティング システムでは、ファイル名やパスで大文字小文字が区別されます。 たとえば、*TodoApi.XML* ファイルは Windows では有効ですが、CentOS では無効です。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=31-33)]

::: moniker-end

::: moniker range="= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup1x.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

前述のコードでは、[Reflection](/dotnet/csharp/programming-guide/concepts/reflection) を使用し、Web API プロジェクトの名前に一致する XML ファイル名が構築されます。 [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory*) プロパティは、XML ファイルのパス構築に使用されます。 Swagger の一部の機能 (たとえば、入力パラメーターや HTTP メソッドのスキーマ、それぞれの属性からの応答コード) は、XML ドキュメント ファイルを使わなくても動作します。 メソッドのサマリーや、パラメーターと応答コードの記述など、ほとんどの機能には、XML ファイルの使用が必須です。

アクションにトリプル スラッシュのコメントを追加すると、セクション ヘッダーに説明が追加され、Swagger UI が向上します。 `Delete` アクションの上に [\<summary>](/dotnet/csharp/programming-guide/xmldoc/summary) 要素を追加します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Delete&highlight=1-3)]

Swagger UI には、前述のコードの `<summary>` 要素の内部テキストが表示されます。

![DELETE メソッドの XML コメント 'Deletes a specific TodoItem.' を 表示している Swagger UI](web-api-help-pages-using-swagger/_static/triple-slash-comments.png)

UI は、生成された JSON スキーマによって決まります。

```json
"delete": {
    "tags": [
        "Todo"
    ],
    "summary": "Deletes a specific TodoItem.",
    "operationId": "ApiTodoByIdDelete",
    "consumes": [],
    "produces": [],
    "parameters": [
        {
            "name": "id",
            "in": "path",
            "description": "",
            "required": true,
            "type": "integer",
            "format": "int64"
        }
    ],
    "responses": {
        "200": {
            "description": "Success"
        }
    }
}
```

[\<remarks>](/dotnet/csharp/programming-guide/xmldoc/remarks) 要素を `Create` アクション メソッドのドキュメントに追加します。 これは、`<summary>` 要素で指定された情報を補足し、Swagger UI をより堅牢にします。 `<remarks>` 要素の内容は、テキスト、JSON、または XML で構成されます。

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

追加コメントで UI が向上している点にご注目ください。

![追加のコメントが表示された Swagger UI](web-api-help-pages-using-swagger/_static/xml-comments-extended.png)

### <a name="data-annotations"></a>データの注釈

[System.ComponentModel.DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) 名前空間にある属性をモデルに追加し、Swagger UI コンポーネントの向上に役立てます。

`[Required]` 属性を `TodoItem` クラスの `Name` プロパティに追加します。

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Models/TodoItem.cs?highlight=10)]

この属性の有無は、UI 動作を変更し、基になる JSON スキーマを変更します。

```json
"definitions": {
    "TodoItem": {
        "required": [
            "name"
        ],
        "type": "object",
        "properties": {
            "id": {
                "format": "int64",
                "type": "integer"
            },
            "name": {
                "type": "string"
            },
            "isComplete": {
                "default": false,
                "type": "boolean"
            }
        }
    }
},
```

API コントローラーに `[Produces("application/json")]` 属性を追加します。 その目的は、コントローラーのアクションが *application/json* の応答コンテンツ タイプをサポートすることを宣言することです。

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

**[応答コンテンツ タイプ]** ドロップダウンがコント ローラーの GET 操作の既定値としてこのコンテンツ タイプを選択します。

![既定の応答のコンテンツ タイプを持つ Swagger UI](web-api-help-pages-using-swagger/_static/json-response-content-type.png)

Web API のデータの注釈の使用量が多いほど、UI と API のヘルプ ページはわかりやすく便利になります。

### <a name="describe-response-types"></a>応答の種類の説明

Web API を使用する開発者は、何が返されるかを最も考慮します&mdash;具体的には、応答の種類とエラー コードです (標準以外の場合)。 応答の種類とエラー コードは、XML コメントとデータ注釈に示されます。

`Create` アクションでは、成功すると HTTP 201 状態コードが返されます。 HTTP 400 状態コードは、投稿された要求本文が null のときに返されます。 Swagger UI に適切なドキュメントがないと、利用者にはこれらの予期される結果の知識がありません。 その問題を解決するために、次の例では強調表示された行を追加しています。

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

Swagger UI は、予期される HTTP 応答コードを明確に記述するようになりました。

![ステータスコードの POST 応答クラスの説明 'Returns the newly created Todo item' and '400 - If the item is null' および応答メッセージの下に理由を示す Swagger UI](web-api-help-pages-using-swagger/_static/data-annotations-response-types.png)

::: moniker range=">= aspnetcore-2.2"

ASP.NET Core 2.2 以降では、明示的に個別のアクションを `[ProducesResponseType]` で装飾する代わりに、規約を使用できます。 詳細については、<xref:web-api/advanced/conventions> を参照してください。

::: moniker-end

### <a name="customize-the-ui"></a>UI をカスタマイズする

ストック UI は機能も見栄えも優れています。 しかしながら、API ドキュメント ページでブランドやテーマを表す必要があります。 Swashbuckle コンポーネントをブランド化するには、静的ファイルにサービスを提供するリソースを追加し、それらのファイルをホストするフォルダー構造を構築する必要があります。

.NET Framework または .NET Core 1.x を対象としている場合、プロジェクトに [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles) NuGet パッケージを追加します。

```xml
<PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="2.0.0" />
```

.NET Core 2.x を対象とし、[メタパッケージ](xref:fundamentals/metapackage)を使用している場合、前述の NuGet パッケージが既にインストールされています。

静的ファイル ミドルウェアの有効化:

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/3.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

::: moniker-end

[Swagger UI GitHub リポジトリ](https://github.com/swagger-api/swagger-ui/tree/master/dist)から *dist* フォルダーの内容を取得します。 このフォルダーには、Swagger UI ページに必要なアセットが含まれています。

*wwwroot/swagger/ui* フォルダーを作成し、それに *dist* フォルダーのコンテンツをコピーします。

*wwwroot/swagger/ui* で *custom.css* ファイルを作成し、次の CSS でページ ヘッダーをカスタマイズします。

[!code-css[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/wwwroot/swagger/ui/custom.css)]

他に CSS ファイルがあればその後で ui フォルダー内で *index.html* ファイルの *custom.css* を参照します。

[!code-html[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/wwwroot/swagger/ui/index.html?name=snippet_SwaggerUiCss&highlight=3)]

`http://localhost:<port>/swagger/ui/index.html` で *index.html* ページを参照します。 ヘッダーのテキスト ボックスに「`https://localhost:<port>/swagger/v1/swagger.json`」を入力し、 **[探索]** ボタンをクリックします。 結果のページは次のようになります。

![カスタム ヘッダーのタイトルを含む Swagger UI](web-api-help-pages-using-swagger/_static/custom-header.png)

このページでできることはたくさんあります。 [Swagger UI GitHub リポジトリ](https://github.com/swagger-api/swagger-ui)で UI リソースの完全な機能を参照してください。
