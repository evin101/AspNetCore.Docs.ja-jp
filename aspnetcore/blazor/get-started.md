---
title: ASP.NET Core Blazor の概要
author: guardrex
description: 選択したツールで Blazor アプリを構築して、Blazor を開始しましょう。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
no-loc:
- Blazor
uid: blazor/get-started
ms.openlocfilehash: d356a06849f54434c492dc68f57f7edc8805de22
ms.sourcegitcommit: 5974e3e66dab3398ecf2324fbb82a9c5636f70de
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/03/2019
ms.locfileid: "74778779"
---
# <a name="get-started-with-aspnet-core-opno-locblazor"></a>ASP.NET Core Blazor の概要

作成者: [Daniel Roth](https://github.com/danroth27)、[Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazorの概要:

::: moniker range=">= aspnetcore-3.1"

1. [.NET Core 3.1 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)をインストールします。

1. 必要に応じて[Blazor WebAssembly テンプレート](xref:blazor/hosting-models#blazor-webassembly)をインストールします。
   * [.NET Core 3.1 以降 (プレビュー) SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)をインストールします。
   * コマンドシェルで次のコマンドを実行します。 [Microsoft.AspNetCore.Blazor.Templates](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/)パッケージにはプレビューバージョンがありますが、プレビュー段階で Blazor WebAssembly はプレビュー版です。

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview4.19579.2
   ```

1. ツールの選択に関するガイダンスに従ってください。

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1\. **ASP.NET と Web 開発**ワークロードを使用して、 [Visual Studio 16.4](https://visualstudio.microsoft.com/vs/preview/)以降をインストールします。

   2\. 新しいプロジェクトを作成します。

   3\. **Blazor アプリ**を選択します。 **[次へ]** を選択します。

   4\. **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **場所**エントリが正しいことを確認するか、プロジェクトの場所を指定します。 **[作成]** を選択します。

   5\1 Blazor WebAssembly については、 **Blazor WebAssembly**テンプレートを選択してください。 Blazor サーバーのエクスペリエンスについては、 **Blazor サーバー アプリ**テンプレートを選択してください。 **[作成]** を選択します。 2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   6\. **Ctrl** + **F5** キーを押してアプリを実行します。

   > [!NOTE]
   > ASP.NET Core Blazor (Preview 6 以前) の以前のプレビューリリースに Blazor Visual Studio 拡張機能をインストールした場合は、拡張機能をアンインストールできます。 Visual Studio でテンプレートを表示するには、コマンドシェルに Blazor テンプレートをインストールするだけで十分です。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1\. [Visual Studio Code](https://code.visualstudio.com/) のインストール。

   2\. [C# Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)の最新版をインストールします。

   3\. Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Blazor サーバーのエクスペリエンスについては、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   4\. Visual Studio Code で*WebApplication1*フォルダーを開きます。

   5\. Blazor サーバープロジェクトの場合、IDE は、プロジェクトをビルドおよびデバッグするためにアセットを追加するように要求します。 **[はい]** を選択します。

   6\. Blazor サーバー アプリを使用している場合は、Visual Studio Code デバッガーを使用してアプリを実行します。 Blazor WebAssembly を使用している場合は、アプリのプロジェクトフォルダーから `dotnet run` を実行します。

   7\. ブラウザーで、`https://localhost:5001` に移動します。

   # <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

   1\. [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)をインストールします。 [更新チャネルをプレビューに](/visualstudio/mac/install-preview)切り替えます。

   2\. **ファイル** > **新しいソリューション**を選択するか、**新しいプロジェクト**を作成します。

   3\. サイドバーで、[ **.NET Core** > **アプリ**] を選択します。

   4\. **Blazor サーバー アプリ**テンプレートを選択します。 現時点では、Visual Studio for Mac で使用できるのは Blazor サーバーテンプレートのみです。 Blazor WebAssembly のエクスペリエンスについては、 **[.NET Core CLI]** タブの指示に従ってください。Blazor サーバーテンプレートを選択したら、 **[次へ]** を選択します。 2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   <!-- For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. -->

   5\. **ターゲットフレームワーク**は、既定で **.NET Core 3.0** (または、3.1 Preview SDK がインストールされている場合は **.NET Core 3.1** ) に設定されます。 フレームワークを選択し、 **[次へ]** を選択します。

   6\. [Project Name] \ (**プロジェクト名**\) フィールドに、アプリに `WebApplication1`という名前を指定します。 **[作成]** を選択します。

   7\. *デバッガーを使用せず*にアプリを実行するには、[**実行** > **デバッグなしで実行**] を選択します。 **デバッグを開始**してアプリを実行し、*デバッガーで*アプリを実行します。

       If a prompt appears to trust the development certificate, trust the certificate and continue.

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor サーバーのエクスペリエンスについては、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   ブラウザーで、`https://localhost:5001` に移動します。

   ---

::: moniker-end

::: moniker range="< aspnetcore-3.1"

1. 最新の[.Net Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0)をインストールします。

1. 必要に応じて[Blazor WebAssembly テンプレート](xref:blazor/hosting-models#blazor-webassembly)をインストールします。
   * [.NET Core 3.1 以降 (プレビュー) SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)をインストールします。
   * コマンドシェルで次のコマンドを実行します。 [Microsoft.AspNetCore.Blazor.Templates](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/)パッケージにはプレビューバージョンがありますが、プレビュー段階で Blazor WebAssembly はプレビュー版です。

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview4.19579.2
   ```

1. ツールの選択に関するガイダンスに従ってください。

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1\. **ASP.NET と Web 開発**ワークロードを使用して、最新の[Visual Studio](https://visualstudio.com/vs/)をインストールします。

   2\. 必要に応じて、アプリ開発 Blazor のための**ASP.NET および Web 開発**ワークロードと共に[Visual Studio 16.4 Preview 2 以降](https://visualstudio.microsoft.com/vs/preview/)をインストールします。

   3\. 新しいプロジェクトを作成します。

   4\. **Blazor アプリ**を選択します。 **[次へ]** を選択します。

   5\. **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **場所**エントリが正しいことを確認するか、プロジェクトの場所を指定します。 **[作成]** を選択します。

   6\. Blazor webassembly については、 **Blazor Webassembly**テンプレートを選択してください。 Blazor サーバーのエクスペリエンスについては、 **Blazor サーバー アプリ**テンプレートを選択してください。 **[作成]** を選択します。 2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   7\. **F5 キー**を押してアプリを実行します。

   > [!NOTE]
   > ASP.NET Core Blazor (Preview 6 以前) の以前のプレビューリリースに Blazor Visual Studio 拡張機能をインストールした場合は、拡張機能をアンインストールできます。 Visual Studio でテンプレートを表示するには、コマンドシェルに Blazor テンプレートをインストールするだけで十分です。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1\. [Visual Studio Code](https://code.visualstudio.com/) のインストール。

   2\. [C# Visual Studio Code 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) の最新版をインストールします。

   3\. Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Blazor サーバーのエクスペリエンスについては、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   4\. Visual Studio Code で*WebApplication1*フォルダーを開きます。

   5\. Blazor サーバープロジェクトの場合、IDE は、プロジェクトをビルドおよびデバッグするためにアセットを追加するように要求します。 **[はい]** を選択します。

   6\. Blazor サーバー アプリを使用している場合は、Visual Studio Code デバッガーを使用してアプリを実行します。 Blazor WebAssembly を使用している場合は、アプリのプロジェクトフォルダーから `dotnet run` を実行します。

   7\. ブラウザーで、`https://localhost:5001` に移動します。

   # <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

   1\. [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)をインストールします。 [更新チャネルをプレビューに](/visualstudio/mac/install-preview)切り替えます。

   2\. **ファイル** > **新しいソリューション**を選択するか、**新しいプロジェクト**を作成します。

   3\. サイドバーで、[ **.NET Core** > **アプリ**] を選択します。

   4\. **Blazor サーバー アプリ**テンプレートを選択します。 現時点では、Visual Studio for Mac で使用できるのは Blazor サーバーテンプレートのみです。 Blazor WebAssemblyのエクスペリエンスについては、 **[.NET Core CLI]** タブの指示に従ってください。Blazor サーバーテンプレートを選択したら、 **[次へ]** を選択します。 2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   <!-- For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. -->

   5\. **ターゲットフレームワーク**は、既定で **.NET Core 3.0** (または、3.1 Preview SDK がインストールされている場合は **.NET Core 3.1** ) に設定されます。 フレームワークを選択し、 **[次へ]** を選択します。

   6\. [Project Name] \ (**プロジェクト名**\) フィールドに、アプリに `WebApplication1`という名前を指定します。 **[作成]** を選択します。

   7\. *デバッガーを使用せず*にアプリを実行するには、[**実行** > **デバッグなしで実行**] を選択します。 **デバッグを開始**してアプリを実行し、*デバッガーで*アプリを実行します。

       If a prompt appears to trust the development certificate, trust the certificate and continue.

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor サーバーのエクスペリエンスについては、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   2つの Blazor ホスティングモデル、 *Blazor サーバー* 、 *Blazor WebAssembly* については、「<xref:blazor/hosting-models>」を参照してください。

   ブラウザーで、`https://localhost:5001` に移動します。

   ---

::: moniker-end

サイドバーのタブからは、複数のページを使用できます。

* Home
* Counter
* Fetch data

Counter ページ上で **[Click me]** ボタンを選択し、ページを更新することなくカウンターをインクリメントします。 通常、Web ページのカウンターを増やすには JavaScript を記述する必要がありますが、Blazor では C# を使用できます。

*Pages/Counter.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

上部の `@page` ディレクティブで指定されているように、ブラウザーでの `/counter` の要求によって、`Counter` コンポーネントがそのコンテンツをレンダリングします。 コンポーネントは、レンダリングツリーのメモリ内表現にレンダリングされ、柔軟で効率的な方法で UI を更新するために使用できます。

**Click me** ボタンが選択されるたびに、次のようになります。

* `onclick` イベントが発生します。
* `IncrementCount` メソッドが呼び出されます。
* `currentCount` がインクリメントされます。
* コンポーネントが再び表示されます。

ランタイムは、新しいコンテンツを前のコンテンツと比較し、変更されたコンテンツのみをドキュメントオブジェクトモデル (DOM) に適用します。

HTML 構文を使用してコンポーネントを別のコンポーネントに追加します。 たとえば、`Index` コンポーネントに `<Counter />` 要素を追加して、アプリのホームページに `Counter` コンポーネントを追加します。

*Pages/Index.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Index1.razor?highlight=7)]

アプリを実行します。 ホームページには、`Counter` コンポーネントによって提供される独自のカウンターがあります。

コンポーネントのパラメーターは、属性または[子コンテンツ](xref:blazor/components#child-content)を使用して指定されます。これにより、子コンポーネントのプロパティを設定できます。 `Counter` コンポーネントにパラメーターを追加するには、コンポーネントの `@code` ブロックを更新します。

* `[Parameter]` 属性を持つ `IncrementAmount` のパブリックプロパティを追加します。
* `currentCount` の値を増やすときに `IncrementAmount` を使うように `IncrementCount` メソッドを変更します。

*Pages/Counter.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter2.razor?highlight=12-13,17)]

属性を使用して、`Index` コンポーネントの `<Counter>` 要素に `IncrementAmount` を指定します。

*Pages/Index.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Index2.razor?highlight=7)]

アプリを実行します。 `Index` コンポーネントには、 **Click me** ボタンが選択されるたびに10ずつ増加する独自のカウンターがあります。 `/counter` の `Counter` コンポーネント (*Counter*) は、1つずつ増加し続けます。

## <a name="next-steps"></a>次のステップ:

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>その他の技術情報

* <xref:blazor/templates>
* <xref:signalr/introduction>
