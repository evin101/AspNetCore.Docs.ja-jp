---
title: ASP.NET Core の Razor SDK
author: Rick-Anderson
description: ASP.NET Core の Razor ページを使用して、ページのコーディングに重点を置いたシナリオをより簡略化して、MVC を使用する場合よりも生産性を高める方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 08/23/2019
uid: razor-pages/sdk
ms.openlocfilehash: 606d2bdca3fa4fb1c81df73ac697d2175c3ab633
ms.sourcegitcommit: 07d98ada57f2a5f6d809d44bdad7a15013109549
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/15/2019
ms.locfileid: "72334036"
---
# <a name="aspnet-core-razor-sdk"></a>ASP.NET Core の Razor SDK

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)

## <a name="overview"></a>概要

@No__t-0 には、@no__t 1 の MSBuild SDK (Razor SDK) が含まれています。 Razor SDK とは、次のとおりです。

::: moniker range=">= aspnetcore-3.0"

* は、ASP.NET Core MVC ベースまたは[Blazor](xref:blazor/index)プロジェクトの[Razor](xref:mvc/views/razor)ファイルを含むプロジェクトをビルド、パッケージ、および発行するために必要です。
* には、Razor (*cshtml*または*razor*) ファイルのコンパイルをカスタマイズするための定義済みのターゲット、プロパティ、および項目のセットが含まれています。

Razor SDK には、`Include` 属性が `**\*.cshtml` および @no__t グロビングパターンに設定されている @no__t 0 の項目が含まれています。 一致するファイルがパブリッシュされます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* ASP.NET Core MVC ベース プロジェクト用の [Razor](xref:mvc/views/razor) ファイルを含むプロジェクトのビルド、パッケージ、および発行に関わる表示と操作を標準化できます。
* Razor ファイルのコンパイルのカスタマイズを可能にする事前定義されたターゲット、プロパティ、項目のセットを含みます。

Razor SDK には、`Include` 属性が @no__t グロビングパターンに設定された @no__t 0 の項目が含まれています。 一致するファイルがパブリッシュされます。

::: moniker-end

## <a name="prerequisites"></a>必要条件

[!INCLUDE[](~/includes/2.1-SDK.md)]

## <a name="use-the-razor-sdk"></a>Razor SDK を使用する

ほとんどの web アプリは、Razor SDK を明示的に参照する必要がありません。

::: moniker range=">= aspnetcore-3.0"

Razor SDK を使用して Razor ビューまたは Razor Pages を含むクラスライブラリをビルドするには、Razor クラスライブラリ (RCL) プロジェクトテンプレートから始めることをお勧めします。 Blazor (*razor*) ファイルのビルドに使用される rcl では、 [Microsoft.AspNetCore.Components](https://www.nuget.org/packages/Microsoft.AspNetCore.Components)パッケージへの参照が最低限必要です。 Razor ビューまたはページ (*cshtml*ファイル) をビルドするために使用される rcl では、`netcoreapp3.0` 以降を対象とする必要があります。また、プロジェクトファイルの[AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)に @no__t が2つあります。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Razor SDK を使用して Razor ビューまたは Razor ページを含むクラス ライブラリを構築する方法は次のとおりです。

* `Microsoft.NET.Sdk.Razor` の代わりに `Microsoft.NET.Sdk` を使用します。

  ```xml
  <Project SDK="Microsoft.NET.Sdk.Razor">
    <!-- omitted for brevity -->
  </Project>
  ```

* 通常、Razor Pages および Razor ビューのビルドとコンパイルに必要な追加の依存関係を受け取るには、`Microsoft.AspNetCore.Mvc` へのパッケージ参照が必要です。 少なくとも、プロジェクトでパッケージ参照を追加する必要があります。

  * `Microsoft.AspNetCore.Razor.Design` 
  * `Microsoft.AspNetCore.Mvc.Razor.Extensions`
  * `Microsoft.AspNetCore.Mvc.Razor`
    
  @No__t 0 パッケージは、プロジェクトの Razor コンパイルタスクとターゲットを提供します。

  前述のパッケージは、`Microsoft.AspNetCore.Mvc` に組み込まれています。 次のマークアップは、Razor SDK を使用して ASP.NET Core Razor Pages アプリの Razor ファイルを作成するプロジェクトファイルを示しています。
    
  [!code-xml[](sdk/sample/RazorSDK.csproj)]
  
::: moniker-end

::: moniker range="= aspnetcore-2.1"

> [!WARNING]
> @No__t-0 および `Microsoft.AspNetCore.Mvc.Razor.Extensions` のパッケージは、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)に含まれています。 ただし、バージョンが少ない `Microsoft.AspNetCore.App` のパッケージ参照では、`Microsoft.AspNetCore.Razor.Design` の最新バージョンを含まないアプリにメタパッケージが提供されます。 プロジェクトは、Razor の最新のビルド時修正プログラムが含まれるように、`Microsoft.AspNetCore.Razor.Design` (または `Microsoft.AspNetCore.Mvc`) の一貫したバージョンを参照する必要があります。 詳細については、次を参照してください。[この GitHub の問題](https://github.com/aspnet/Razor/issues/2553)します。

::: moniker-end

### <a name="properties"></a>プロパティ

Razor の SDK の動作は、プロジェクトをビルドする際に次のプロパティにより制御されます。

* `RazorCompileOnBuild` @no__t `true` の場合、プロジェクトのビルドの一部として Razor アセンブリがコンパイルされて出力されます。 既定値は `true` です。
* `RazorCompileOnPublish` @no__t `true` の場合、プロジェクトの発行の一部として Razor アセンブリがコンパイルされて出力されます。 既定値は `true` です。

次の表のプロパティと項目は、Razor SDK への入力と出力を構成するために使用されます。

::: moniker range=">= aspnetcore-3.0"

> [!WARNING]
> ASP.NET Core 3.0 以降では、プロジェクトファイル内の `RazorCompileOnBuild` または `RazorCompileOnPublish` の MSBuild プロパティが無効になっている場合、MVC ビューまたは Razor Pages は既定では提供されません。 アプリケーションがランタイムコンパイルに依存して[Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation)ファイルを処理する場合は、RuntimeCompilation パッケージへの明示的な参照を追加する必要があります。

::: moniker-end

| 項目 | 説明 |
| ----- | ----------- |
| `RazorGenerate` | コード生成の入力である項目要素 (*cshtml*ファイル)。 |
| `RazorComponent` | Razor コンポーネントのコード生成に入力する項目要素 (*razor*ファイル)。 |
| `RazorCompile` | Razor コンパイルターゲットへの入力である項目要素 ( *.cs*ファイル)。 Razor アセンブリにコンパイルする追加ファイルを指定するには、この `ItemGroup` を使用します。 |
| `RazorTargetAssemblyAttribute` | Razor アセンブリ用の属性をコード生成するために使用する項目要素です。 (例:  <br>`RazorAssemblyAttribute`<br>`Include="System.Reflection.AssemblyMetadataAttribute"`<br>`_Parameter1="BuildSource" _Parameter2="https://docs.microsoft.com/">` |
| `RazorEmbeddedResource` | 生成された Razor アセンブリに埋め込みリソースとして追加された項目要素。 |

| property | 説明 |
| -------- | ----------- |
| `RazorTargetName` | Razor によって生成されたアセンブリの (拡張子なしの) ファイル名です。 | 
| `RazorOutputPath` | Razor の出力ディレクトリです。 |
| `RazorCompileToolset` | Razor アセンブリをビルドするために使用するツールセットを決定するために使用します。 有効な値は `Implicit`、`RazorSDK`、`PrecompilationTool` です。 |
| [EnableDefaultContentItems](https://github.com/aspnet/websdk/blob/rel-2.0.0/src/ProjectSystem/Microsoft.NET.Sdk.Web.ProjectSystem.Targets/netstandard1.0/Microsoft.NET.Sdk.Web.ProjectSystem.targets#L21) | 既定値は `true` です。 @No__t 0 の場合は、 *web.config、* *json*、および*cshtml*ファイルがプロジェクトのコンテンツとして含まれます。 @No__t-0 を使用して参照すると、 *wwwroot*ファイルと config ファイルの下にあるファイルも含まれます。 |
| `EnableDefaultRazorGenerateItems` | `true` の場合、`RazorGenerate` 項目の `Content` 項目の *.cshtml* ファイルが含まれます。 |
| `GenerateRazorTargetAssemblyInfo` | @No__t-0 の場合、@no__t によって指定された属性を含む .cs ファイルを生成し、そのファイルをコンパイル出力に含めます *。* |
| `EnableDefaultRazorTargetAssemblyInfoAttributes` | `true` の場合、`RazorAssemblyAttribute` にアセンブリ属性の既定のセットが追加されます。 |
| `CopyRazorGenerateFilesToPublishDirectory` | @No__t-0 の場合、`RazorGenerate` 項目 (*cshtml*) ファイルを発行ディレクトリにコピーします。 通常、ビルド時または発行時にコンパイルに参加する場合、Razor ファイルは発行されたアプリには必要ありません。 既定値は `false` です。 |
| `CopyRefAssembliesToPublishDirectory` | `true` の場合、発行ディレクトリに参照アセンブリ項目がコピーされます。 通常、ビルド時または発行時に Razor コンパイルが発生した場合、公開されたアプリには参照アセンブリは必要ありません。 発行されたアプリがランタイムコンパイルを必要とする場合は、`true` に設定します。 たとえば、アプリで実行時に、または埋め込みビューを使用する場合は、値を `true` に設定し*ます*。 既定値は `false` です。 |
| `IncludeRazorContentInPack` | @No__t 0 の場合、すべての Razor コンテンツ項目 (*cshtml*ファイル) は、生成された NuGet パッケージに含めるようにマークが付けられます。 既定値は `false` です。 |
| `EmbedRazorGenerateSources` | `true` の場合、生成された Razor アセンブリに、埋め込みファイルとして RazorGenerate ( *.cshtml*) 項目が追加されます。 既定値は `false` です。 |
| `UseRazorBuildServer` | `true` の場合、コードの生成作業をオフロードするために、永続的なビルド サーバーが使用されます。 既定値は、`UseSharedCompilation` の値です。 |
| `GenerateMvcApplicationPartsAssemblyAttributes` | @No__t が0の場合、アプリケーションパーツの検出を実行するために、SDK によって実行時に MVC によって使用される追加の属性が生成されます。 |

プロパティの詳細については、「[MSBuild プロパティ](/visualstudio/msbuild/msbuild-properties)」を参照してください。

### <a name="targets"></a>ターゲット

Razor SDK では、次の 2 つの主要なターゲットが定義されています。

* `RazorGenerate` &ndash; コードは `RazorGenerate` 項目要素から .cs ファイルを生成*し*ます。 このターゲットの前または後に実行できる追加のターゲットを指定するには、`RazorGenerateDependsOn` プロパティを使用します。
* `RazorCompile` &ndash; は、生成された *.cs*ファイルを Razor アセンブリにコンパイルします。 このターゲットの前または後に実行できる追加のターゲットを指定するには、`RazorCompileDependsOn` を使用します。
* `RazorComponentGenerate` &ndash; コードは @no__t 3 つの項目要素の .cs ファイルを生成*し*ます。 このターゲットの前または後に実行できる追加のターゲットを指定するには、`RazorComponentGenerateDependsOn` プロパティを使用します。

### <a name="runtime-compilation-of-razor-views"></a>Razor ビューの実行時のコンパイル

* 既定では、Razor SDK は、実行時のコンパイルを実行するために必要な参照アセンブリを公開しません。 この結果、アプリケーション モデルが実行時のコンパイルに依存している場合には、コンパイルが失敗します。たとえば、アプリが公開後に埋め込まれたビューを使用したり、ビューを変更したりする場合などです。 `CopyRefAssembliesToPublishDirectory` を `true` に設定して、参照アセンブリの公開を続行します。

* Web アプリの場合は、アプリが @no__t 0 SDK を対象としていることを確認します。

## <a name="razor-language-version"></a>Razor 言語バージョン

@No__t-0 SDK を対象とする場合、Razor 言語バージョンはアプリのターゲットフレームワークのバージョンから推測されます。 @No__t 0 SDK を対象とするプロジェクトの場合、またはまれに、アプリが推定値とは異なる Razor 言語バージョンを必要とする場合は、アプリのプロジェクトファイルで `<RazorLangVersion>` プロパティを設定することによってバージョンを構成できます。

```xml
<PropertyGroup>
  <RazorLangVersion>{VERSION}</RazorLangVersion>
</PropertyGroup>
```

Razor の言語バージョンは、ビルドされたランタイムのバージョンと密接に統合されています。 ランタイム向けに設計されていない言語バージョンをターゲットにすることはサポートされていないため、ビルドエラーが発生する可能性があります。

## <a name="additional-resources"></a>その他の技術情報

* [.NET Core の csproj 形式に追加されたもの](/dotnet/core/tools/csproj)
* [MSBuild プロジェクトの共通項目](/visualstudio/msbuild/common-msbuild-project-items)
