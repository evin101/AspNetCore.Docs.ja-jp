---
title: ASP.NET Core での静的資産のバンドルと縮小
author: scottaddie
description: バンドルと縮小の手法を適用して ASP.NET Core web アプリケーションで静的リソースを最適化する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 06/17/2019
uid: client-side/bundling-and-minification
ms.openlocfilehash: 7499381a24a2513a4fbd1205f245e624c86647c3
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080559"
---
# <a name="bundle-and-minify-static-assets-in-aspnet-core"></a>ASP.NET Core での静的資産のバンドルと縮小

作成者: [Scott Addie](https://twitter.com/Scott_Addie)と[David 松](https://twitter.com/davidpine7)

この記事では、ASP.NET Core web アプリでこれらの機能を使用する方法など、バンドルと縮小を適用する利点について説明します。

## <a name="what-is-bundling-and-minification"></a>バンドルと縮小とは

バンドルと縮小は、web アプリで適用できる2つの異なるパフォーマンス最適化です。 サーバー要求の数を減らし、要求された静的な資産のサイズを小さくすることで、バンドルと縮小を一緒に使用してパフォーマンスを向上させます。

バンドルと縮小は、主に最初のページ要求の読み込み時間を短縮します。 Web ページが要求されると、ブラウザーは静的なアセット (JavaScript、CSS、およびイメージ) をキャッシュします。 そのため、同じサイトで同じ資産を要求している同じページまたはページを要求した場合、バンドルと縮小によってパフォーマンスが向上することはありません。 有効期限ヘッダーがアセットに対して正しく設定されていない場合、バンドルと縮小が使用されていない場合、ブラウザーの鮮度ヒューリスティックによって、数日後に古い資産が古くなっているとマークされます。 さらに、ブラウザーでは、各資産に対する検証要求が必要です。 この場合、バンドルと縮小によって、最初のページ要求の後でもパフォーマンスが向上します。

### <a name="bundling"></a>まとめる

バンドルは、複数のファイルを1つのファイルに結合します。 バンドルを行うと、web ページなどの web アセットをレンダリングするために必要なサーバー要求の数が減ります。 CSS や JavaScript などに対して、任意の数の個別バンドルを作成できます。ファイルが減るほど、ブラウザーからサーバーまたはアプリケーションを提供するサービスへの HTTP 要求が減少します。 その結果、最初のページ読み込みのパフォーマンスが向上します。

### <a name="minification"></a>縮小

縮小機能を変更せずに、コードから不要な文字を削除します。 結果として、要求された資産 (CSS、画像、JavaScript ファイルなど) のサイズが大幅に削減されます。 縮小の一般的な副作用には、変数名を1文字に短縮し、コメントや不要な空白を削除することがあります。

次の JavaScript 関数を考えてみます。

[!code-javascript[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/wwwroot/js/site.js)]

縮小により、関数が次のように短縮されます。

[!code-javascript[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/wwwroot/js/site.min.js)]

コメントや不要な空白文字を削除するだけでなく、次のパラメーター名と変数名が次のように変更されました。

元 | 名前の変更
--- | :---:
`imageTagAndImageID` | `t`
`imageContext` | `a`
`imageElement` | `r`

## <a name="impact-of-bundling-and-minification"></a>バンドルと縮小の影響

次の表は、アセットを個別に読み込んで、バンドルと縮小を使用する場合の違いをまとめたものです。

アクション | B/M を使用 | B/M なし | [Change]
--- | :---: | :---: | :---:
ファイル要求  | 7   | 18     | 157%
転送された KB | 156 | 264.68 | 70%
読み込み時間 (ミリ秒) | 885 | 2360   | 167%

ブラウザーは、HTTP 要求ヘッダーに関してかなり冗長です。 送信された合計バイト数のメトリックは、バンドルするときに大幅に削減されていました。 読み込み時間は大幅に改善されていますが、この例はローカルで実行されています。 ネットワーク経由で転送される資産でバンドルと縮小を使用すると、パフォーマンスが向上します。

## <a name="choose-a-bundling-and-minification-strategy"></a>バンドルと縮小の戦略を選択する

MVC と Razor Pages のプロジェクトテンプレートには、JSON 構成ファイルで構成されるバンドルと縮小のためのすぐに使用できるソリューションが用意されています。 [Grunt](xref:client-side/using-grunt)タスクランナーなどのサードパーティのツールでは、同じタスクを少し複雑にして実行します。 サードパーティ製のツールは、開発ワークフローが、ラインリングやイメージの最適化などの&mdash;バンドルと縮小を超える処理を必要とする場合に、非常に適しています。 デザイン時のバンドルと縮小を使用することにより、アプリの展開の前に縮小したファイルが作成されます。 配置前のバンドルと縮小によって、サーバーの負荷が軽減されます。 ただし、デザイン時のバンドルと縮小がビルドの複雑さを増し、静的ファイルでのみ機能することを認識することが重要です。

## <a name="configure-bundling-and-minification"></a>バンドルと縮小の構成

::: moniker range="<= aspnetcore-2.0"

ASP.NET Core 2.0 以前では、MVC および Razor Pages のプロジェクトテンプレートには、各バンドルのオプションを定義する bundleconfig 構成ファイルが用意されて*い*ます。

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

ASP.NET Core 2.1 以降では、MVC または Razor Pages プロジェクトルートに*bundleconfig*という名前の新しい json ファイルを追加します。 次の JSON を開始点としてそのファイルに含めます。

::: moniker-end

[!code-json[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/bundleconfig.json)]

Bundleconfig ファイルは、各バンドルのオプションを定義し*ます*。 前の例では、カスタム JavaScript (*wwwroot/js/* node.js) と stylesheet (*wwwroot/css/.css*) ファイルに対して1つのバンドル構成が定義されています。

構成オプションは次のとおりです。

* `outputFileName`:出力するバンドルファイルの名前。 には、 *bundleconfig*ファイルからの相対パスを含めることができます。 **必須**
* `inputFiles`:まとめてバンドルするファイルの配列。 これらは、構成ファイルへの相対パスです。 **省略可能**、* 空の値を指定すると、空の出力ファイルが生成されます。 [グロビング](https://www.tldp.org/LDP/abs/html/globbingref.html)パターンがサポートされています。
* `minify`:出力の種類の縮小オプション。 **省略可能**、*既定`minify: { enabled: true }`値-*
  * 構成オプションは、出力ファイルの種類ごとに使用できます。
    * [CSS ミニ識別子](https://github.com/madskristensen/BundlerMinifier/wiki/cssminifier)
    * [JavaScript ミニ識別子](https://github.com/madskristensen/BundlerMinifier/wiki/JavaScript-Minifier-settings)
    * [HTML ミニ識別子](https://github.com/madskristensen/BundlerMinifier/wiki)
* `includeInProject`:生成されたファイルをプロジェクトファイルに追加するかどうかを示すフラグです。 **省略可能**、*既定値-false*
* `sourceMap` :バンドルされたファイルのソースマップを生成するかどうかを示すフラグです。 **省略可能**、*既定値-false*
* `sourceMapRootPath`:生成されたソースマップファイルを格納するためのルートパス。

## <a name="build-time-execution-of-bundling-and-minification"></a>バンドルと縮小のビルド時の実行

[BuildBundlerMinifier](https://www.nuget.org/packages/BuildBundlerMinifier/) NuGet パッケージを使用すると、ビルド時にバンドルと縮小を実行できます。 パッケージは、ビルド時とクリーンアップ時に実行される[MSBuild ターゲット](/visualstudio/msbuild/msbuild-targets)を挿入します。 *Bundleconfig*ファイルは、定義された構成に基づいて出力ファイルを生成するために、ビルドプロセスによって分析されます。

> [!NOTE]
> BuildBundlerMinifier は、Microsoft がサポートしていない GitHub のコミュニティ主導のプロジェクトに属しています。 [ここで](https://github.com/madskristensen/BundlerMinifier/issues)問題を提出する必要があります。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

*BuildBundlerMinifier*パッケージをプロジェクトに追加します。

プロジェクトをビルドします。 [出力] ウィンドウに次のように表示されます。

```console
1>------ Build started: Project: BuildBundlerMinifierApp, Configuration: Debug Any CPU ------
1>
1>Bundler: Begin processing bundleconfig.json
1>  Minified wwwroot/css/site.min.css
1>  Minified wwwroot/js/site.min.js
1>Bundler: Done processing bundleconfig.json
1>BuildBundlerMinifierApp -> C:\BuildBundlerMinifierApp\bin\Debug\netcoreapp2.0\BuildBundlerMinifierApp.dll
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```

プロジェクトを消去します。 [出力] ウィンドウに次のように表示されます。

```console
1>------ Clean started: Project: BuildBundlerMinifierApp, Configuration: Debug Any CPU ------
1>
1>Bundler: Cleaning output from bundleconfig.json
1>Bundler: Done cleaning output file from bundleconfig.json
========== Clean: 1 succeeded, 0 failed, 0 skipped ==========
```

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

*BuildBundlerMinifier*パッケージをプロジェクトに追加します。

```dotnetcli
dotnet add package BuildBundlerMinifier
```

ASP.NET Core 1.x を使用している場合は、新しく追加したパッケージを復元します。

```dotnetcli
dotnet restore
```

プロジェクトをビルドします。

```dotnetcli
dotnet build
```

次のように表示されます。

```console
Microsoft (R) Build Engine version 15.4.8.50001 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.


    Bundler: Begin processing bundleconfig.json
    Bundler: Done processing bundleconfig.json
    BuildBundlerMinifierApp -> C:\BuildBundlerMinifierApp\bin\Debug\netcoreapp2.0\BuildBundlerMinifierApp.dll
```

プロジェクトを消去します。

```dotnetcli
dotnet clean
```

次の出力が表示されます。

```console
Microsoft (R) Build Engine version 15.4.8.50001 for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.


  Bundler: Cleaning output from bundleconfig.json
  Bundler: Done cleaning output file from bundleconfig.json
```

---

## <a name="ad-hoc-execution-of-bundling-and-minification"></a>バンドルと縮小のアドホック実行

プロジェクトをビルドしなくても、バンドルタスクと縮小タスクをアドホックベースで実行できます。 [BundlerMinifier](https://www.nuget.org/packages/BundlerMinifier.Core/) NuGet パッケージをプロジェクトに追加します。

[!code-xml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/BuildBundlerMinifierApp.csproj?range=10)]

> [!NOTE]
> BundlerMinifier は、Microsoft がサポートしていない GitHub のコミュニティ主導のプロジェクトに属しています。 [ここで](https://github.com/madskristensen/BundlerMinifier/issues)問題を提出する必要があります。

このパッケージは、 *dotnet*ツールを含むように .NET Core CLI を拡張します。 次のコマンドは、パッケージマネージャーコンソール (PMC) ウィンドウまたはコマンドシェルで実行できます。

```dotnetcli
dotnet bundle
```

> [!IMPORTANT]
> NuGet パッケージマネージャーは、* .csproj ファイルにノードとし`<PackageReference />`て依存関係を追加します。 コマンドは、 `<DotNetCliToolReference />`ノードが使用されている場合にのみ .NET Core CLI に登録されます。 `dotnet bundle` 必要に応じて * .csproj ファイルを変更します。

## <a name="add-files-to-workflow"></a>ワークフローへのファイルの追加

次のように、追加の*カスタム .css*ファイルが追加される例を考えてみましょう。

[!code-css[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/wwwroot/css/custom.css)]

*カスタム .css*を縮小して、*サイト* *の .css ファイルに*バンドルするには、 *bundleconfig*への相対パスを追加します。

[!code-json[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/bundleconfig2.json?highlight=6)]

> [!NOTE]
> または、次のグロビングパターンを使用することもできます。
>
> ```json
> "inputFiles": ["wwwroot/**/*(*.css|!(*.min.css))"]
> ```
>
> このグロビングパターンは、すべての CSS ファイルに一致し、縮小されたファイルパターンは除外されます。

アプリケーションをビルドします。 [ *.Css* ] を開き、[*カスタム .css* ] の内容がファイルの末尾に追加されていることを確認します。

## <a name="environment-based-bundling-and-minification"></a>環境ベースのバンドルと縮小

ベストプラクティスとして、アプリのバンドルファイルと縮小版ファイルを運用環境で使用することをお勧めします。 開発時には、元のファイルによってアプリのデバッグが簡単になります。

ビューで[環境タグヘルパー](xref:mvc/views/tag-helpers/builtin-th/environment-tag-helper)を使用して、ページに含めるファイルを指定します。 環境タグヘルパーは、特定の[環境](xref:fundamentals/environments)で実行されている場合にのみコンテンツをレンダリングします。

次`environment`のタグは、 `Development`環境で実行されているときに、未処理の CSS ファイルをレンダリングします。

::: moniker range=">= aspnetcore-2.0"

[!code-cshtml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/Pages/_Layout.cshtml?highlight=3&range=21-24)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-cshtml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/Pages/_Layout.cshtml?highlight=3&range=9-12)]

::: moniker-end

次`environment`のタグは、以外`Development`の環境で実行されている場合に、バンドルされた、縮小された CSS ファイルをレンダリングします。 たとえば、または`Production` `Staging`でを実行すると、次のスタイルシートのレンダリングがトリガーされます。

::: moniker range=">= aspnetcore-2.0"

[!code-cshtml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/Pages/_Layout.cshtml?highlight=5&range=25-30)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-cshtml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/Pages/_Layout.cshtml?highlight=3&range=13-18)]

::: moniker-end

## <a name="consume-bundleconfigjson-from-gulp"></a>Gulp から bundleconfig を使用する

アプリのバンドルと縮小ワークフローで追加の処理が必要になる場合があります。 例としては、イメージの最適化、キャッシュ処理、CDN 資産の処理などがあります。 これらの要件を満たすために、Gulp を使用するようにバンドルと縮小のワークフローを変換できます。

### <a name="use-the-bundler--minifier-extension"></a>Bundler & Minifier 拡張機能を使用する

Visual Studio [Bundler の & Minifier](https://marketplace.visualstudio.com/items?itemName=MadsKristensen.BundlerMinifier)拡張機能は、への変換を処理します。

> [!NOTE]
> Bundler & Minifier 拡張機能は、Microsoft がサポートしていない GitHub のコミュニティ主導のプロジェクトに属しています。 [ここで](https://github.com/madskristensen/BundlerMinifier/issues)問題を提出する必要があります。

ソリューションエクスプローラーで*bundleconfig*ファイルを右クリックし、[ **Bundler &**  > minifier**Convert To Gulp...** ] を選択します。

![Gulp コンテキストメニュー項目に変換する](../client-side/bundling-and-minification/_static/convert-to-gulp.png)

*Gulpfile*ファイルと*パッケージ*ファイルがプロジェクトに追加されます。 *パッケージの json*ファイルの`devDependencies`セクションに一覧表示されている[npm](https://www.npmjs.com/)パッケージのサポートがインストールされます。

PMC ウィンドウで次のコマンドを実行して、Gulp CLI をグローバルな依存関係としてインストールします。

```console
npm i -g gulp-cli
```

*Gulpfile*ファイルは、入力、出力、および設定の*bundleconfig*ファイルを読み取ります。

[!code-javascript[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/gulpfile.js?range=1-12&highlight=10)]

### <a name="convert-manually"></a>手動で変換

Visual Studio または Bundler & Minifier の拡張機能が使用できない場合は、手動で変換します。

次`devDependencies`のを含む*パッケージの json*ファイルをプロジェクトのルートに追加します。

> [!WARNING]
> モジュール`gulp-uglify`は、ECMAScript (ES) 2015/ES6 以降をサポートしていません。 ES2015/ES6 以降を使用する`gulp-uglify`のではなく[、gulp を](https://www.npmjs.com/package/gulp-terser)インストールします。

[!code-json[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/package.json?range=5-13)]

次のコマンドを*package. json*と同じレベルで実行して、依存関係をインストールします。

```console
npm i
```

グローバルな依存関係として Gulp CLI をインストールします。

```console
npm i -g gulp-cli
```

次の*gulpfile*ファイルをプロジェクトのルートにコピーします。

[!code-javascript[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/gulpfile.js?range=1-11,14-)]

### <a name="run-gulp-tasks"></a>Gulp タスクの実行

Visual Studio でプロジェクトをビルドする前に Gulp の縮小タスクをトリガーするには、次の[MSBuild ターゲット](/visualstudio/msbuild/msbuild-targets)を * .csproj ファイルに追加します。

[!code-xml[](../client-side/bundling-and-minification/samples/BuildBundlerMinifierApp/BuildBundlerMinifierApp.csproj?range=14-16)]

この例では、 `MyPreCompileTarget`ターゲット内で定義されているすべてのタスクが、定義済み`Build`のターゲットの前に実行されます。 次のような出力が Visual Studio の出力ウィンドウに表示されます。

```console
1>------ Build started: Project: BuildBundlerMinifierApp, Configuration: Debug Any CPU ------
1>BuildBundlerMinifierApp -> C:\BuildBundlerMinifierApp\bin\Debug\netcoreapp2.0\BuildBundlerMinifierApp.dll
1>[14:17:49] Using gulpfile C:\BuildBundlerMinifierApp\gulpfile.js
1>[14:17:49] Starting 'min:js'...
1>[14:17:49] Starting 'min:css'...
1>[14:17:49] Starting 'min:html'...
1>[14:17:49] Finished 'min:js' after 83 ms
1>[14:17:49] Finished 'min:css' after 88 ms
========== Build: 1 succeeded, 0 failed, 0 up-to-date, 0 skipped ==========
```


## <a name="additional-resources"></a>その他の技術情報

* [Grunt の使用](xref:client-side/using-grunt)
* [複数の環境の使用](xref:fundamentals/environments)
* [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)
