---
title: Razor コンポーネントクラスライブラリの ASP.NET Core
author: guardrex
description: コンポーネントを外部コンポーネントライブラリから Blazor アプリに組み込む方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/23/2019
uid: blazor/class-libraries
ms.openlocfilehash: 2e042b43c6db24e0ecac727be100575fe1275e17
ms.sourcegitcommit: 6d26ab647ede4f8e57465e29b03be5cb130fc872
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/07/2019
ms.locfileid: "71999778"
---
# <a name="aspnet-core-razor-components-class-libraries"></a>Razor コンポーネントクラスライブラリの ASP.NET Core

作成者: [Simon Timms](https://github.com/stimms)

コンポーネントは、プロジェクト間で[Razor クラスライブラリ (RCL)](xref:razor-pages/ui-class)で共有できます。 *Razor コンポーネントクラスライブラリ*は、次のものからインクルードできます。

* ソリューション内の別のプロジェクト。
* NuGet パッケージ。
* 参照された .NET ライブラリです。

コンポーネントが通常の .NET 型であるのと同様に、RCL によって提供されるコンポーネントは通常の .NET アセンブリです。

## <a name="create-an-rcl"></a>RCL を作成する

@No__t 0 の記事のガイダンスに従って、Blazor の環境を構成します。

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 新しいプロジェクトを作成します。
1. **[Razor クラスライブラリ]** を選択します。 **[次へ]** を選択します。
1. **[新しい Razor クラスライブラリを作成]** します ダイアログで、 **[作成]** を選択します。
1. **プロジェクト名** フィールドにプロジェクト名を入力するか、既定のプロジェクト名をそのまま使用します。 このトピックの例では、プロジェクト名 `MyComponentLib1` を使用します。 **[作成]** を選択します。
1. RCL をソリューションに追加します。
   1. ソリューションを右クリックします。 [@No__t の**追加**] を選択して、**既存のプロジェクト**を1つ選択します。
   1. RCL のプロジェクトファイルに移動します。
   1. RCL のプロジェクトファイル ( *.csproj*) を選択します。
1. アプリから RCL の参照を追加します。
   1. アプリプロジェクトを右クリックします。 [ **Add** > **Reference**] を選択します。
   1. RCL プロジェクトを選択します。 **[OK]** を選択します。

# <a name="net-core-clitabnetcore-cli"></a>[.NET Core CLI](#tab/netcore-cli)

1. コマンドシェルで[dotnet new](/dotnet/core/tools/dotnet-new)コマンドを使用して、 **Razor クラスライブラリ**テンプレート (`razorclasslib`) を使用します。 次の例では、`MyComponentLib1` という名前の RCL が作成されます。 コマンドの実行時に、`MyComponentLib1` を含むフォルダーが自動的に作成されます。

   ```dotnetcli
   dotnet new razorclasslib -o MyComponentLib1
   ```

1. 既存のプロジェクトにライブラリを追加するには、コマンドシェルで dotnet の [[参照の追加](/dotnet/core/tools/dotnet-add-reference)] コマンドを使用します。 次の例では、RCL がアプリに追加されています。 ライブラリへのパスを使用して、アプリのプロジェクトフォルダーから次のコマンドを実行します。

   ```dotnetcli
   dotnet add reference {PATH TO LIBRARY}
   ```

---

## <a name="consume-a-library-component"></a>ライブラリコンポーネントの使用

別のプロジェクトのライブラリで定義されているコンポーネントを使用するには、次のいずれかの方法を使用します。

* 名前空間で完全な型名を使用します。
* Razor の[\@using](xref:mvc/views/razor#using) ディレクティブを使用します。 個々のコンポーネントを名前で追加することができます。

次の例では、`MyComponentLib1` は、`SalesReport` コンポーネントを含むコンポーネントライブラリです。

@No__t 0 のコンポーネントは、名前空間を持つ完全な型名を使用して参照できます。

```cshtml
<h1>Hello, world!</h1>

Welcome to your new app.

<MyComponentLib1.SalesReport />
```

また、ライブラリが @no__t 0 ディレクティブを使用してスコープ内にある場合は、このコンポーネントを参照することもできます。

```cshtml
@using MyComponentLib1

<h1>Hello, world!</h1>

Welcome to your new app.

<SalesReport />
```

ライブラリのコンポーネントをプロジェクト全体で使用できるようにするには、最上位の*インポート*ファイルに `@using MyComponentLib1` ディレクティブを含めます。 ディレクティブを任意のレベルの*インポート*ファイルに追加して、名前空間をフォルダー内の1つのページまたは一連のページに適用します。

## <a name="build-pack-and-ship-to-nuget"></a>NuGet のビルド、パック、出荷

コンポーネントライブラリは標準の .NET ライブラリであるため、パッケージを NuGet にパッケージ化して配布することは、ライブラリをパッケージ化して NuGet に配布する場合と変わりはありません。 パッケージ化は、コマンドシェルで[dotnet pack](/dotnet/core/tools/dotnet-pack)コマンドを使用して実行されます。

```dotnetcli
dotnet pack
```

コマンドシェルで[dotnet nuget push](/dotnet/core/tools/dotnet-nuget-push)コマンドを使用して、パッケージを nuget にアップロードします。

## <a name="create-a-razor-components-class-library-with-static-assets"></a>静的なアセットを含む Razor コンポーネントクラスライブラリを作成する

RCL には、静的なアセットを含めることができます。 この静的アセットは、ライブラリを使用するすべてのアプリで使用できます。 詳細については、「 <xref:razor-pages/ui-class#create-an-rcl-with-static-assets> 」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:razor-pages/ui-class>
