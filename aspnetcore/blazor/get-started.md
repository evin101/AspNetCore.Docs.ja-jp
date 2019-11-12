---
title: ASP.NET Core Blazor を使ってみる
author: guardrex
description: 好みのツールで Blazor アプリを構築して、Blazor の使用を開始しましょう。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 11/07/2019
uid: blazor/get-started
ms.openlocfilehash: b5043c7e4549800c1ab49bc37dd8f3568975d4aa
ms.sourcegitcommit: 67116718dc33a7a01696d41af38590fdbb58e014
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/07/2019
ms.locfileid: "73799234"
---
# <a name="get-started-with-aspnet-core-blazor"></a>ASP.NET Core Blazor を使ってみる

作成者: [Daniel Roth](https://github.com/danroth27)、[Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor を使ってみる:

::: moniker range=">= aspnetcore-3.1"

1. [.Net Core 3.1 PREVIEW SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)をインストールします。

1. コマンドシェルで次のコマンドを実行して、 [Blazor WebAssembly](xref:blazor/hosting-models#blazor-webassembly)テンプレートをインストールします。 [Microsoft.AspNetCore.Blazor.Templates](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.Templates/)パッケージはプレビュー版ですが、Blazor WebAssembly はプレビュー段階です。

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview2.19528.8
   ```

1. ツールの選択に関するガイダンスに従ってください。

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \。 **ASP.NET と web 開発**ワークロードを使用して、 [Visual Studio 16.4 Preview 2](https://visualstudio.microsoft.com/vs/preview/)以降をインストールします。

   2 \。 新しいプロジェクトを作成します。

   3 \。 **[Blazor App]** を選択します。 **[次へ]** を選択します。

   4 \。 **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **場所**エントリが正しいことを確認するか、プロジェクトの場所を指定します。 **[作成]** を選択します。

   5 \。 Blazor WebAssembly エクスペリエンスについては、 **Blazor Webassembly**テンプレートを選択してください。 Blazor サーバーエクスペリエンスの場合は、 **Blazor Server アプリ**テンプレートを選択します。 **[作成]** を選択します。 *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   6 \。 **Ctrl** + **F5** キーを押してアプリを実行します。

   > [!NOTE]
   > 以前のプレビューリリースの ASP.NET Core Blazor (Preview 6 以前) 用に Blazor Visual Studio 拡張機能をインストールした場合は、拡張機能をアンインストールできます。 Visual Studio でテンプレートを表示するには、コマンドシェルに Blazor テンプレートをインストールするだけで十分です。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \。 [Visual Studio Code](https://code.visualstudio.com/) のインストール。

   2 \。 [ C# Visual Studio Code 拡張機能の](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)最新版をインストールします。

   3 \。 Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Blazor サーバーエクスペリエンスの場合は、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   4 \。 Visual Studio Code で*WebApplication1*フォルダーを開きます。

   5 \。 Blazor Server プロジェクトの場合、IDE は、プロジェクトをビルドおよびデバッグするためにアセットを追加するように要求します。 **[はい]** を選択します。

   6 \。 Blazor Server アプリを使用している場合は、Visual Studio Code デバッガーを使用してアプリを実行します。 Blazor WebAssembly を使用する場合は、アプリのプロジェクトフォルダーから `dotnet run` を実行します。

   7 \。 ブラウザーで、`https://localhost:5001` に移動します。

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor サーバーエクスペリエンスの場合は、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   ブラウザーで、`https://localhost:5001` に移動します。

   ---

::: moniker-end

::: moniker range="< aspnetcore-3.1"

1. 最新の[.Net Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0)リリースをインストールします。

1. 必要に応じて、 [.Net Core 3.1 PREVIEW SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1)をインストールし、コマンドシェルで次のコマンドを実行して、 [Blazor webassembly](xref:blazor/hosting-models#blazor-webassembly)テンプレートをインストールします。

   ```dotnetcli
   dotnet new -i Microsoft.AspNetCore.Blazor.Templates::3.1.0-preview2.19528.8
   ```

1. ツールの選択に関するガイダンスに従ってください。

   # <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

   1 \。 **ASP.NET と web 開発**ワークロードを使用して、最新の[Visual Studio](https://visualstudio.com/vs/)をインストールします。

   2 \。 必要に応じて、 [Visual Studio 16.4 Preview 2](https://visualstudio.microsoft.com/vs/preview/)以降を、Blazor WebAssembly アプリ開発のための**および web 開発**ワークロードと共にインストールします。

   3 \。 新しいプロジェクトを作成します。

   4 \。 **[Blazor App]** を選択します。 **[次へ]** を選択します。

   5 \。 **[プロジェクト名]** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 **場所**エントリが正しいことを確認するか、プロジェクトの場所を指定します。 **[作成]** を選択します。

   6 \。 Blazor WebAssembly エクスペリエンスについては、 **Blazor Webassembly**テンプレートを選択してください。 Blazor サーバーエクスペリエンスの場合は、 **Blazor Server アプリ**テンプレートを選択します。 **[作成]** を選択します。 *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   7 \。 **F5 キー**を押してアプリを実行します。

   > [!NOTE]
   > 以前のプレビューリリースの ASP.NET Core Blazor (Preview 6 以前) 用に Blazor Visual Studio 拡張機能をインストールした場合は、拡張機能をアンインストールできます。 Visual Studio でテンプレートを表示するには、コマンドシェルに Blazor テンプレートをインストールするだけで十分です。

   # <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

   1 \。 [Visual Studio Code](https://code.visualstudio.com/) のインストール。

   2 \。 [ C# Visual Studio Code 拡張機能の](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)最新版をインストールします。

   3 \。 Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorwasm -o WebApplication1
      ```

      Blazor サーバーエクスペリエンスの場合は、コマンドシェルで次のコマンドを実行します。

      ```dotnetcli
      dotnet new blazorserver -o WebApplication1
      ```

      *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   4 \。 Visual Studio Code で*WebApplication1*フォルダーを開きます。

   5 \。 Blazor Server プロジェクトの場合、IDE は、プロジェクトをビルドおよびデバッグするためにアセットを追加するように要求します。 **[はい]** を選択します。

   6 \。 Blazor Server アプリを使用している場合は、Visual Studio Code デバッガーを使用してアプリを実行します。 Blazor WebAssembly を使用する場合は、アプリのプロジェクトフォルダーから `dotnet run` を実行します。

   7 \。 ブラウザーで、`https://localhost:5001` に移動します。

   <!--

   # [Visual Studio for Mac](#tab/visual-studio-mac)

   1\. Install [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/). Switch the [Update channel to Preview](/visualstudio/mac/install-preview).

   2\. Select **File** > **New Solution** or **New Project**.

   3\. In the sidebar, select **.NET Core** > **App**.

   4\. For a Blazor Server experience, select the **Blazor Server App** template. For a Blazor WebAssembly experience, select the **Blazor WebAssembly App** template. Select **Next**. For information on the two Blazor hosting models, *Blazor Server* and *Blazor WebAssembly*, see <xref:blazor/hosting-models>.

   5\. The **Target Framework** defaults to **.NET Core 3.0**. Select **Next**.

   6\. In the **Project Name** field, enter `WebApplication1`. Select **Create**.

   7\. Select **Run** > **Run Without Debugging** to run the app *without the debugger*. Running with the debugger isn't supported at this time.

   -->

   # <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

   Blazor WebAssembly を実現するには、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   Blazor サーバーエクスペリエンスの場合は、コマンドシェルで次のコマンドを実行します。

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   cd WebApplication1
   dotnet run
   ```

   *Blazor Server*と*Blazor Webassembly*の2つのホスティングモデルの詳細については、<xref:blazor/hosting-models> を参照してください。

   ブラウザーで、`https://localhost:5001` に移動します。

   ---

::: moniker-end

サイドバーのタブからは、複数のページを使用できます。

* Home
* カウンター
* データのフェッチ

Counter ページ上で **[クリックしてください]** ボタンを選択し、ページを更新することなくカウンターをインクリメントします。 通常、web ページ内のカウンターを増やすには JavaScript を記述する必要がC#ありますが、Blazor を使用することもできます。

*Pages/Counter.razor*:

[!code-cshtml[](get-started/samples_snapshot/3.x/Counter1.razor?highlight=7,12-15)]

上部の `@page` ディレクティブで指定されているように、ブラウザーで `/counter` に対する要求を行うと、`Counter` コンポーネントによってそのコンテンツがレンダリングされます。 コンポーネントは、レンダリングツリーのメモリ内表現にレンダリングされ、柔軟で効率的な方法で UI を更新するために使用できます。

**[クリックし**てください] ボタンが選択されるたびに、次のようになります。

* `onclick` イベントが発生します。
* `IncrementCount` メソッドが呼び出された場合。
* `currentCount` がインクリメントされます。
* コンポーネントが再び表示されます。

ランタイムは、新しいコンテンツを前のコンテンツと比較し、変更されたコンテンツのみをドキュメントオブジェクトモデル (DOM) に適用します。

HTML 構文を使用してコンポーネントを別のコンポーネントに追加します。 たとえば、`Index` コンポーネントに `<Counter />` 要素を追加して、`Counter` コンポーネントをアプリのホームページに追加します。

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

アプリを実行します。 `Index` コンポーネントには、 **[クリックし**てください] ボタンが選択されるたびに10ずつ増加する独自のカウンターがあります。 `/counter` の `Counter` コンポーネント (*Counter*) は、1つずつ増加し続けます。

## <a name="next-steps"></a>次のステップ

<xref:tutorials/first-blazor-app>

## <a name="additional-resources"></a>その他の技術情報

* <xref:signalr/introduction>
