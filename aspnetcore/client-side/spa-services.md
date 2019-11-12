---
title: JavaScript サービスを使用して ASP.NET Core にシングルページアプリケーションを作成する
author: scottaddie
description: JavaScript サービスを使用して、ASP.NET Core によってサポートされるシングルページアプリケーション (SPA) を作成する利点について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: H1Hack27Feb2017
ms.date: 09/06/2019
uid: client-side/spa-services
ms.openlocfilehash: 7aff46f739239246191763e0590046b2d9995922
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080508"
---
# <a name="use-javascript-services-to-create-single-page-applications-in-aspnet-core"></a>JavaScript サービスを使用して ASP.NET Core にシングルページアプリケーションを作成する

作成者: [Scott Addie](https://github.com/scottaddie)と[Fiyaz Hasan](https://fiyazhasan.me/)

シングル ページ アプリケーション (SPA) は、その固有の機能豊富なユーザー エクスペリエンスのための web アプリケーションの人気のある型です。 クライアント側の SPA フレームワークまたはライブラリ ([角度](https://angular.io/)や[反応](https://facebook.github.io/react/)など) をサーバー側のフレームワーク (ASP.NET Core など) と統合することは困難な場合があります。 JavaScript サービスは、統合プロセスの摩擦を減らすために開発されました。 これにより、別のクライアントおよびサーバー テクノロジ スタックとの間のシームレスな操作ができます。

::: moniker range=">= aspnetcore-3.0"

> [!WARNING]
> この記事で説明されている機能は、ASP.NET Core 3.0 の場合は廃止されます。 [AspNetCore SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices.Extensions) NuGet パッケージでは、より単純な SPA フレームワーク統合メカニズムを使用できます。 詳細については、「 [[アナウンスメント] Obsoleting AspNetCore. SpaServices And AspNetCore. NodeServices](https://github.com/aspnet/AspNetCore/issues/12890)」を参照してください。

::: moniker-end

## <a name="what-is-javascript-services"></a>JavaScript サービスとは

JavaScript サービスは、ASP.NET Core のクライアント側テクノロジのコレクションです。 その目的は、Spa を構築するための開発者の推奨されるサーバー側プラットフォームとしての ASP.NET Core の位置です。

JavaScript サービスは、次の2つの異なる NuGet パッケージで構成されています。

* [Microsoft.AspNetCore.NodeServices](https://www.nuget.org/packages/Microsoft.AspNetCore.NodeServices/) (NodeServices)
* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) (SpaServices)

これらのパッケージは、次のシナリオで役立ちます。

* サーバーでの JavaScript を実行します。
* SPA フレームワークやライブラリを使用します。
* Webpack とクライアント側アセットをビルドします。

SpaServices パッケージを使用してこの記事内のフォーカスの多くは配置されます。

## <a name="what-is-spaservices"></a>SpaServices とは

SpaServices は、Spa を構築するための開発者の推奨されるサーバー側プラットフォームとしての ASP.NET Core の位置に作成されました。 SpaServices は、ASP.NET Core で SPAs を開発するためには必要ありません。また、開発者は特定のクライアントフレームワークにロックされません。

SpaServices は、次のような便利なインフラストラクチャを提供します。

* [サーバー側の事前](#server-side-prerendering)
* [Webpack 開発ミドルウェア](#webpack-dev-middleware)
* [ホットなモジュールの交換](#hot-module-replacement)
* [ルーティングのヘルパー](#routing-helpers)

総称して、これらのインフラストラクチャ コンポーネントは、開発ワークフローと、実行時のエクスペリエンスの両方を強化します。 コンポーネントを個別に採用することができます。

## <a name="prerequisites-for-using-spaservices"></a>SpaServices を使用するための前提条件

SpaServices を使用するには、次のようにインストールします。

* [Node.js](https://nodejs.org/) (バージョン 6 以降) で npm

  * これらのコンポーネントがインストールされを検出できることを確認するには、コマンドラインから、次を実行します。

    ```console
    node -v && npm -v
    ```

  * Azure の web サイトにデプロイする場合、必要な&mdash;操作はありません node.js がインストールされ、サーバー環境で使用できます。

* [!INCLUDE [](~/includes/net-core-sdk-download-link.md)]

  * Visual Studio 2017 を使用している Windows では、 **.Net Core クロスプラットフォーム開発**ワークロードを選択して SDK をインストールします。

* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) NuGet パッケージ

## <a name="server-side-prerendering"></a>サーバー側の事前

ユニバーサル (isomorphic とも呼ばれます) のアプリケーションは、サーバーとクライアントの両方で実行できる JavaScript アプリケーションです。 Angular、React、およびその他の一般的なフレームワークは、このアプリケーションの開発スタイルのユニバーサル プラットフォームを提供します。 考え方は、まず、Node.js を使用して、サーバー上のフレームワーク コンポーネントをレンダリングし、さらに、クライアントを実行しを委任します。

ASP.NET Core[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)によって提供される SpaServices、サーバー上の JavaScript 関数を呼び出すことによってサーバー側の事前の実装を簡略化します。

### <a name="server-side-prerendering-prerequisites"></a>サーバー側のプリレンダリングの前提条件

[Aspnet プリレンダリング](https://www.npmjs.com/package/aspnet-prerendering)npm パッケージをインストールします。

```console
npm i -S aspnet-prerendering
```

### <a name="server-side-prerendering-configuration"></a>サーバー側のプリレンダリング構成

タグ ヘルパーは、プロジェクトの名前空間の登録を使用して探索可能にされて *_ViewImports.cshtml*ファイル。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/_ViewImports.cshtml?highlight=3)]

これらのタグ ヘルパーは Razor ビュー内の HTML のような構文を活用することで、低レベルの Api と直接通信の複雑さで抽象化します。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=5)]

### <a name="asp-prerender-module-tag-helper"></a>asp-prerender-module タグヘルパー

`asp-prerender-module`タグ ヘルパーの前のコード例で使用される実行*ClientApp/dist/main-server.js* Node.js を使用してサーバーにします。 わかりやすくするためのために、 *main server.js*ファイルは、TypeScript、JavaScript にトランス パイルもタスクの成果物、 [Webpack](https://webpack.github.io/)プロセスを構築します。 Webpack のエントリ ポイントのエイリアスを定義します`main-server`; と、このエイリアスの依存関係グラフのトラバーサルが始まり、 *ClientApp/ブート-server.ts*ファイル。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=53)]

次の Angular の例では、 *ClientApp/ブート-server.ts*ファイルを利用、`createServerRenderer`関数と`RenderResult`の入力、 `aspnet-prerendering` npm パッケージを Node.js を使用してサーバー レンダリングを構成します。 サーバー側のレンダリングが厳密に型指定された JavaScript にラップされて解決関数の呼び出しに渡される宛ての HTML マークアップ`Promise`オブジェクト。 `Promise`オブジェクトの重要性は、DOM のプレース ホルダー要素の挿入のページに HTML マークアップを非同期的に提供します。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-34,79-)]

### <a name="asp-prerender-data-tag-helper"></a>asp-prerender-データタグヘルパー

組み合わせると、`asp-prerender-module`タグ ヘルパーの`asp-prerender-data`タグ ヘルパーは、Razor ビューからサーバー側 JavaScript にコンテキスト情報を渡すために使用できます。 たとえば、次のマークアップは合格ユーザー データを`main-server`モジュール。

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=9-12)]

受信した`UserName`引数の組み込みの JSON シリアライザーを使用してシリアル化し、は、`params.data`オブジェクト。 次の例で Angular 内でパーソナライズされたあいさつ文を構築するデータの使用、`h1`要素。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,38-52,79-)]

タグヘルパーで渡されるプロパティ名は、文字表記の**case**表記で表されます。 同じプロパティ名で表現は JavaScript とは異なり**camelCase**します。 既定の JSON シリアル化の構成では、この違いを担当します。

展開すると、上記のコード例に、データ渡し可能サーバーからビューに hydrating、`globals`プロパティに提供される、`resolve`関数。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,57-77,79-)]

`postList`配列内で定義されている、`globals`オブジェクトはグローバルで、ブラウザーにアタッチされて`window`オブジェクト。 グローバル スコープにこの変数のホイストでは、特に、サーバーで 1 回と、クライアントでは、同じデータの読み込みに関連している、作業の重複がなくなります。

![ウィンドウ オブジェクトにアタッチされているグローバルの postList 変数](spa-services/_static/global_variable.png)

## <a name="webpack-dev-middleware"></a>Webpack 開発ミドルウェア

[Webpack 開発ミドルウェア](https://webpack.js.org/guides/development/#using-webpack-dev-middleware)Webpack がオンデマンドでリソースをビルドするための合理的な開発ワークフローが導入されています。 ミドルウェアが自動的にコンパイルし、ページがブラウザーに再読み込みする際、クライアント側のリソースの機能します。 別の方法では、サードパーティの依存関係またはカスタム コードが変更されたときに、プロジェクトの npm ビルド スクリプトを使用して Webpack を手動で起動します。 Npm スクリプトを作成する、 *package.json*ファイルは、次の例に示します。

```json
"build": "npm run build:vendor && npm run build:custom",
```

### <a name="webpack-dev-middleware-prerequisites"></a>Webpack Dev ミドルウェアの前提条件

[Aspnet-webpack](https://www.npmjs.com/package/aspnet-webpack) npm パッケージをインストールします。

```console
npm i -D aspnet-webpack
```

### <a name="webpack-dev-middleware-configuration"></a>Webpack 開発ミドルウェアの構成

次のコードを使用して HTTP 要求パイプラインに Webpack 開発ミドルウェアが登録されている、 *Startup.cs*ファイルの`Configure`メソッド。

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_WebpackMiddlewareRegistration&highlight=4)]

`UseWebpackDevMiddleware`する前に、拡張メソッドを呼び出す必要があります[静的ファイルをホストしている登録](xref:fundamentals/static-files)を使用して、`UseStaticFiles`拡張メソッド。 セキュリティ上の理由から、アプリは開発モードで実行時にのみ、ミドルウェアを登録します。

*Webpack.config.js*ファイルの`output.publicPath`プロパティに通知を監視するミドルウェア、`dist`フォルダーの変更。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,13-16)]

## <a name="hot-module-replacement"></a>ホットなモジュールの交換

Webpack の考える[ホット モジュールの交換](https://webpack.js.org/concepts/hot-module-replacement/)(HMR) 機能の進化したものとして[Webpack 開発ミドルウェア](#webpack-dev-middleware)します。 すべて同じメリットが導入されて HMR が自動的に変更をコンパイルした後、ページのコンテンツを更新することでさらに、開発ワークフローを効率化します。 メモリ内の現在の状態と、SPA のデバッグ セッションに支障をきたすのブラウザーの更新でこれを混同しないでください。 Webpack 開発ミドルウェア サービスと、ブラウザーに変更がプッシュされることを意味すると、ブラウザーの間のライブ リンクがあります。

### <a name="hot-module-replacement-prerequisites"></a>ホットモジュール置換の前提条件

[Webpack-hot ミドルウェア](https://www.npmjs.com/package/webpack-hot-middleware)npm パッケージをインストールします。

```console
npm i -D webpack-hot-middleware
```

### <a name="hot-module-replacement-configuration"></a>ホットモジュールの交換の構成

MVC の HTTP 要求パイプラインに HMR コンポーネントを登録する必要があります、`Configure`メソッド。

```csharp
app.UseWebpackDevMiddleware(new WebpackDevMiddlewareOptions {
    HotModuleReplacement = true
});
```

そうであったとして[Webpack 開発ミドルウェア](#webpack-dev-middleware)、`UseWebpackDevMiddleware`する前に、拡張メソッドを呼び出す必要があります、`UseStaticFiles`拡張メソッド。 セキュリティ上の理由から、アプリは開発モードで実行時にのみ、ミドルウェアを登録します。

*Webpack.config.js*ファイルを定義する必要があります、`plugins`場合でも、それを空の配列。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,25)]

ブラウザーでアプリを読み込んだ後は、開発者ツールのコンソール タブは、HMR アクティブ化の確認を提供します。

![ホットのモジュールの交換接続メッセージ](spa-services/_static/hmr_connected.png)

## <a name="routing-helpers"></a>ルーティングのヘルパー

ほとんどの ASP.NET Core ベースの SPAs では、サーバー側のルーティングに加えて、クライアント側のルーティングが必要になることがよくあります。 SPA と MVC ルーティング システムは、競合することがなく個別に作業できます。 ただし、課題を 1 つエッジ ケース読んだり: 404 の HTTP 応答を識別します。

シナリオの場合を検討してください、拡張子のないルートの`/some/page`使用されます。 要求がパターン一致、サーバー側ルートがそのパターンのクライアント側ルートが一致するものとします。 ここでの受信要求を考えてみます`/images/user-512.png`、一般に、サーバー上の画像ファイルを検索する要求。 要求されたリソースパスがどのサーバー側のルートまたは静的ファイルとも一致しない場合、クライアント側アプリケーションが&mdash;通常は 404 HTTP 状態コードを返すように処理することはほとんどありません。

### <a name="routing-helpers-prerequisites"></a>ルーティングヘルパーの前提条件

クライアント側ルーティングの npm パッケージをインストールします。 Angular を使用して、例として。

```console
npm i -S @angular/router
```

### <a name="routing-helpers-configuration"></a>ルーティングヘルパーの構成

という名前の拡張メソッド`MapSpaFallbackRoute`で使用されて、`Configure`メソッド。

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_MvcRoutingTable&highlight=7-9)]

ルートは、構成されている順序で評価されます。 その結果、`default`パターン一致の前のコード例ではルートが最初に使用します。

## <a name="create-a-new-project"></a>新しいプロジェクトを作成する

JavaScript サービスには、構成済みのアプリケーションテンプレートが用意されています。 SpaServices は、これらのテンプレートで、角度、反応、Redux などのさまざまなフレームワークやライブラリと共に使用されます。

これらのテンプレートは、次のコマンドを実行して .NET Core CLI を使用してインストールできます。

```dotnetcli
dotnet new --install Microsoft.AspNetCore.SpaTemplates::*
```

使用可能な SPA テンプレートの一覧が表示されます。

| テンプレート                                 | 短い形式の名前 | 言語 | Tags        |
| ------------------------------------------| :--------: | :------: | :---------: |
| Angular と ASP.NET Core の MVC             | angular    | [C#]     | Web/MVC/SPA |
| React.js 付きの ASP.NET Core の MVC            | react      | [C#]     | Web/MVC/SPA |
| MVC の ASP.NET Core react.js と Redux  | reactredux | [C#]     | Web/MVC/SPA |

SPA テンプレートのいずれかを使用して新しいプロジェクトを作成するには含める、**短い名前**内のテンプレートの[新しい dotnet](/dotnet/core/tools/dotnet-new)コマンド。 次のコマンドでは、サーバー側用に構成された ASP.NET Core MVC で Angular アプリケーションを作成します。

```dotnetcli
dotnet new angular
```

### <a name="set-the-runtime-configuration-mode"></a>ランタイムの構成モードを設定します。

2 つのプライマリのランタイム構成モードは次のとおりです。

* **開発**:
  * デバッグを容易にするソース マップが含まれています。
  * パフォーマンスのクライアント側のコードを最適化しません。
* **実稼働**:
  * ソース マップを除外します。
  * バンドルと縮小を使用して、クライアント側のコードを最適化します。

ASP.NET Core という環境変数を使用して`ASPNETCORE_ENVIRONMENT`構成モードを格納します。 詳細については、「[環境を設定する](xref:fundamentals/environments#set-the-environment)」を参照してください。

### <a name="run-with-net-core-cli"></a>.NET Core CLI で実行する

プロジェクトのルートで、次のコマンドを実行してには、必要な NuGet と npm パッケージを復元します。

```dotnetcli
dotnet restore && npm i
```

ビルドし、アプリケーションを実行します。

```dotnetcli
dotnet run
```

アプリケーションの起動時に従い、localhost 上で、[ランタイム構成モード](#set-the-runtime-configuration-mode)します。 移動する`http://localhost:5000`ブラウザーにランディング ページが表示されます。

### <a name="run-with-visual-studio-2017"></a>Visual Studio 2017 で実行する

開く、 *.csproj*によって生成されたファイル、[新しい dotnet](/dotnet/core/tools/dotnet-new)コマンド。 必要な NuGet と npm パッケージは、プロジェクトを開くと自動的に復元されます。 この復元プロセスは、数分かかる場合があり、アプリケーションが完了するときに実行する準備ができます。 キーを押して、緑色の実行 ボタンをクリックします`Ctrl + F5`、し、アプリケーションのランディング ページで、ブラウザーが開かれます。 アプリケーションによる localhost で実行、[ランタイム構成モード](#set-the-runtime-configuration-mode)します。

## <a name="test-the-app"></a>アプリのテスト

SpaServices テンプレートは、事前構成を使用してクライアント側のテストを実行して[Karma](https://karma-runner.github.io/1.0/index.html)と[Jasmine](https://jasmine.github.io/)します。 Jasmine は、Karma テスト ランナーはこれらのテストには、人気のある単位の JavaScript 用のテスト フレームワークです。 Karma を使用するように構成、 [Webpack 開発ミドルウェア](#webpack-dev-middleware)されるよう、開発者が停止し、変更されるたびにテストを実行するために必要はありません。 テスト_ケースまたはテスト ケース自体に対して実行されるコードであるかどうか、テストが自動的に実行されます。

Angular アプリケーションを使用して、例として、2 つの Jasmine テスト_ケースが既に提供されている、`CounterComponent`で、 *counter.component.spec.ts*ファイル。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/app/components/counter/counter.component.spec.ts?range=15-28)]

コマンド プロンプトを開き、 *ClientApp*ディレクトリ。 次のコマンドを実行します。

```console
npm test
```

スクリプトで定義されている設定を読み取り、Karma テスト ランナーを起動する、 *karma.conf.js*ファイル。 その他の設定の間で、 *karma.conf.js*経由で実行するテスト ファイルを識別します。 その`files`配列。

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/test/karma.conf.js?range=4-5,8-11)]

## <a name="publish-the-app"></a>アプリの発行

デプロイの準備完了パッケージに生成されたクライアント側アセットと発行された ASP.NET Core の成果物を組み合わせることは面倒なことができます。 さいわいにも、SpaServices がという名前のカスタム MSBuild ターゲットとそのパブリケーション全体のプロセスを調整`RunWebpack`:

[!code-xml[](../client-side/spa-services/sample/SpaServicesSampleApp/SpaServicesSampleApp.csproj?range=31-45)]

MSBuild ターゲットでは、次の責任があります。

1. Npm パッケージを復元します。
1. サードパーティのクライアント側アセットの実稼働レベルのビルドを作成します。
1. カスタムクライアント側アセットの実稼働レベルのビルドを作成します。
1. Webpack によって生成されたアセットを publish フォルダーにコピーします。

MSBuild ターゲットが実行するときに呼び出されます。

```dotnetcli
dotnet publish -c Release
```

## <a name="additional-resources"></a>その他の技術情報

* [Angular Docs](https://angular.io/docs)
