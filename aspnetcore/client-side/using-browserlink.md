---
title: ASP.NET Core でのブラウザー リンク
author: ncarandini
description: ブラウザー リンクの 1 つまたは複数の web ブラウザーでの開発環境をリンクしている Visual Studio の機能の方法について説明します。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 09/22/2017
uid: client-side/using-browserlink
ms.openlocfilehash: 452ba5149563c186750466f471c7b950f0017614
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/27/2019
ms.locfileid: "64894709"
---
# <a name="browser-link-in-aspnet-core"></a>ASP.NET Core でのブラウザー リンク

作成者: [Nicolò Carandini](https://github.com/ncarandini)、 [Mike Wasson](https://github.com/MikeWasson)、および[Tom Dykstra](https://github.com/tdykstra)

ブラウザー リンクは、Visual studio 開発環境と 1 つまたは複数の web ブラウザーの間の通信チャネルを作成する機能です。 クロス ブラウザー テスト用に便利です、いくつかのブラウザーで web のアプリケーションは、一度にを更新するブラウザー リンクを使用できます。

## <a name="browser-link-setup"></a>ブラウザー リンクのセットアップ

::: moniker range=">= aspnetcore-2.1"

ASP.NET Core 2.1 に移行して、ASP.NET Core 2.0 プロジェクトを変換するときに、 [Microsoft.AspNetCore.App メタパッケージ](xref:fundamentals/metapackage-app)、インストール、 [Microsoft.VisualStudio.Web.BrowserLink](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.BrowserLink/)のパッケージ化します。BrowserLink 機能します。 ASP.NET Core 2.1 のプロジェクト テンプレートを使用して、`Microsoft.AspNetCore.App`既定メタパッケージ。

::: moniker-end

::: moniker range="= aspnetcore-2.0"

ASP.NET Core 2.0 **Web アプリケーション**、**空**、および**Web API**プロジェクト テンプレートの使用、 [Microsoft.AspNetCore.All メタパッケージ](xref:fundamentals/metapackage)、のパッケージ参照が含まれています[Microsoft.VisualStudio.Web.BrowserLink](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.BrowserLink/)します。 したがってを使用して、`Microsoft.AspNetCore.All`メタパッケージにはブラウザー リンクを使用できるようにするさらに操作は必要ありません。

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

ASP.NET Core 1.x **Web アプリケーション**プロジェクト テンプレートのパッケージ参照がある、 [Microsoft.VisualStudio.Web.BrowserLink](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.BrowserLink/)パッケージ。 **空**または**Web API**プロジェクト テンプレートでは、パッケージ参照を追加する必要があります`Microsoft.VisualStudio.Web.BrowserLink`します。

これは、Visual Studio の機能では、最も簡単な方法にパッケージを追加するため、**空**または**Web API**テンプレート プロジェクトを開くことです、**パッケージ マネージャー コンソール**(**ビュー** > **その他の Windows** > **パッケージ マネージャー コンソール**) し、次のコマンドを実行します。

```console
install-package Microsoft.VisualStudio.Web.BrowserLink
```

また、使用することができます**NuGet パッケージ マネージャー**します。 プロジェクト名を右クリックして**ソリューション エクスプ ローラー**選択**NuGet パッケージの管理**:

![開いている NuGet パッケージ マネージャー](using-browserlink/_static/open-nuget-package-manager.png)

検索して、パッケージをインストールします。

![パッケージで NuGet パッケージ マネージャーを追加します。](using-browserlink/_static/add-package-with-nuget-package-manager.png)

::: moniker-end

### <a name="configuration"></a>構成

`Startup.Configure` メソッドの場合:

```csharp
app.UseBrowserLink();
```

内でコードは、通常、`if`ブロックしかここで示すように、開発環境で Browser Link を有効にします。

```csharp
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseBrowserLink();
}
```

詳細については、「[Use multiple environments](xref:fundamentals/environments)」(複数の環境の使用) を参照してください。

## <a name="how-to-use-browser-link"></a>ブラウザー リンクを使用する方法

Visual Studio がブラウザー リンクのツール バー コントロールを横に表示されますがある場合、ASP.NET Core プロジェクトを開いて、**デバッグ ターゲット**ツール バー コントロール。

![ブラウザー リンクのドロップダウン メニュー](using-browserlink/_static/browserLink-dropdown-menu.png)

ブラウザー リンク ツール バー コントロールでは、次の操作を実行できます。

* 複数のブラウザーで web アプリケーションを一度にを更新します。
* 開く、**ブラウザー リンク ダッシュ ボード**します。
* 有効または無効に**Browser Link**します。 メモ:Visual Studio 2017 (15.3) で既定では、ブラウザー リンクが無効です。
* 有効または無効に[CSS 自動同期](#enable-or-disable-css-auto-sync)します。

> [!NOTE]
> Visual Studio プラグインによって、最も顕著な*Web 拡張機能パック 2015*と*Web 拡張機能パック 2017*、ブラウザー リンク拡張機能を提供して、ASP でいくつかの追加機能は動作しません。NET Core プロジェクト。

## <a name="refresh-the-web-app-in-several-browsers-at-once"></a>一度に複数のブラウザーで web アプリを更新します。

プロジェクトを開始するときに起動する 1 つの web ブラウザーを選択するドロップダウン メニューを使用して、**デバッグ ターゲット**ツール バー コントロール。

![F5 キーを押してドロップダウン メニュー](using-browserlink/_static/debug-target-dropdown-menu.png)

一度に複数のブラウザーを開く、次のように選択します**を参照しています...** 同じドロップダウン リストから。 ブラウザーを選択して CTRL キーを押しながらクリックして**参照**:

![一度に多くのブラウザーを開く](using-browserlink/_static/open-many-browsers-at-once.png)

Index ビューで開いている Visual Studio を示すスクリーン ショットと 2 つの開いているブラウザーを次に示します。

![2 つのブラウザーの例との同期します。](using-browserlink/_static/sync-with-two-browsers-example.png)

ブラウザー リンクのツール バー コントロールをプロジェクトに接続されているブラウザーを表示するポインターを合わせます。

![ポイントしたときのヒント](using-browserlink/_static/hoover-tip.png)

インデックス ビューを変更し、ブラウザー リンクの更新 ボタンをクリックすると、接続されているすべてのブラウザーは更新されます。

![browsers-sync-to-changes](using-browserlink/_static/browsers-sync-to-changes.png)

ブラウザー リンクは、アプリケーションの URL に移動して Visual Studio の外部からを起動しているブラウザーでも機能します。

### <a name="the-browser-link-dashboard"></a>ブラウザー リンク ダッシュ ボード

ブラウザー リンクのドロップダウン メニューを開いているブラウザーとの接続を管理するのには、ブラウザー リンク ダッシュ ボードを開きます。

![オープン browserslink ダッシュ ボード](using-browserlink/_static/open-browserlink-dashboard.png)

ブラウザーが接続されていない場合を選択して非デバッグ セッションを開始することができます、*ブラウザーで表示*リンク。

![browserlink-dashboard-no-connections](using-browserlink/_static/browserlink-dashboard-no-connections.png)

それ以外の場合、接続されたブラウザーは、各ブラウザーで表示されているページへのパスと表示されます。

![browserlink-dashboard-two-connections](using-browserlink/_static/browserlink-dashboard-two-connections.png)

必要に応じて、その 1 つのブラウザーを更新する表示されているブラウザー名をクリックすることができます。

### <a name="enable-or-disable-browser-link"></a>有効にするか、ブラウザー リンクを無効にします。

無効にすると、ブラウザー リンクを再度有効には、それらを再接続するようにブラウザーを更新する必要があります。

### <a name="enable-or-disable-css-auto-sync"></a>有効にするか、CSS 自動同期を無効にします。

CSS 自動同期を有効にすると、CSS ファイルを変更するときに接続されているブラウザーが自動的に更新します。

## <a name="how-it-works"></a>しくみ

ブラウザー リンクでは、SignalR を使用して、Visual Studio とブラウザー間の通信チャネルを作成します。 Browser Link を有効にすると、Visual Studio は、複数のクライアント (ブラウザー) が接続できる SignalR サーバーとして機能します。 また、ブラウザー リンクは、ASP.NET Core の要求パイプラインでミドルウェア コンポーネントを登録します。 このコンポーネントは特殊な挿入`<script>`ページ要求ごとに、サーバーからの参照。 選択して、スクリプト参照を確認できます**ソースの表示**の末尾にスクロールし、ブラウザーで、`<body>`コンテンツをタグします。

```html
    <!-- Visual Studio Browser Link -->
    <script type="application/json" id="__browserLink_initializationData">
        {"requestId":"a717d5a07c1741949a7cefd6fa2bad08","requestMappingFromServer":false}
    </script>
    <script type="text/javascript" src="http://localhost:54139/b6e36e429d034f578ebccd6a79bf19bf/browserLink" async="async"></script>
    <!-- End Browser Link -->
</body>
```

ソース ファイルは変更されません。 ミドルウェア コンポーネントは、スクリプト参照を動的に挿入します。

ブラウザー側のコードは、すべての JavaScript であるために、ブラウザーのプラグインを必要とせず、SignalR がサポートされているすべてのブラウザーで動作します。
