---
title: ASP.NET Core での Razor ページの概要
author: Rick-Anderson
description: ASP.NET Core の Razor ページを使用して、ページのコーディングに重点を置いたシナリオをより簡略化して、MVC を使用する場合よりも生産性を高める方法について説明します。
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 10/07/2019
uid: razor-pages/index
ms.openlocfilehash: 61e15b9b9b8f84de36621c301ecb9d33b21dff88
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034274"
---
# <a name="introduction-to-razor-pages-in-aspnet-core"></a>ASP.NET Core での Razor ページの概要

::: moniker range=">= aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Ryan Nowak](https://github.com/rynowak)

Razor ページを利用することで、ページのコーディングに今まで以上に集中できます。また、コントローラーとビューを使用する場合より生産的になります。

モデル ビュー コントローラーのアプローチを使用するチュートリアルをお探しの場合は、「[Get started with ASP.NET Core MVC](xref:tutorials/first-mvc-app/start-mvc)」 (ASP.NET Core MVC の概要) を参照してください。

このドキュメントでは、Razor ページの概要について説明します。 手順を追って説明するチュートリアルではありません。 セクションの一部を理解できない場合は、「[Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)」を参照してください。 ASP.NET Core の概要については、「[ASP.NET Core の概要](xref:index)」を参照してください。

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

<a name="rpvs17"></a>

## <a name="create-a-razor-pages-project"></a>Razor ページ プロジェクトを作成する

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Razor ページ プロジェクトを作成する詳細な手順については、「[Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)」を参照してください。

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

コマンド ラインから `dotnet new webapp` を実行します。

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

コマンド ラインから `dotnet new webapp` を実行します。

Visual Studio for Mac から生成された *.csproj* ファイルを開きます。

---

## <a name="razor-pages"></a>Razor ページ

Razor ページは *Startup.cs* で有効になっています。

[!code-cs[](index/3.0sample/RazorPagesIntro/Startup.cs?name=snippet_Startup&highlight=12)]

基本ページを検討します。<a name="OnGet"></a>

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index.cshtml?highlight=1)]

上記のコードは、コントローラーとビューを含んだ ASP.NET Core アプリで使われる [Razor ビュー ファイル](xref:tutorials/first-mvc-app/adding-view)によく似ています。 違いは [@page](xref:mvc/views/razor#page) ディレクティブにあります。 `@page` はファイルを MVC アクションにします。つまり、コントローラーを経由せずに要求を直接処理します。 `@page` はページで最初の Razor ディレクティブである必要があります。 `@page` はその他の [Razor](xref:mvc/views/razor) コンストラクトの動作に影響します。 Razor Pages ファイル名には *.cshtml* サフィックスが付きます。

`PageModel` クラスを使用している類似したページが、次の 2 つのファイルにあります。 *Pages/Index2.cshtml* ファイル:

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml)]

*Pages/Index2.cshtml.cs* ページ モデル:

[!code-cs[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

規則により、`PageModel` クラス ファイルは、Razor ページ ファイルと同じ名前に *.cs* が付加された名前になります。 たとえば、上の Razor ページは *Pages/Index2.cshtml* になります。 `PageModel` クラスを含むファイル名は、*Pages/Index2.cshtml.cs* になります。

URL パスのページへの関連付けは、ファイル システム内のページの場所によって決定されます。 次の表に、Razor ページ パスと一致 URL を示します。

| ファイル名とパス               | 一致 URL |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` または `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` または `/Store/Index` |

メモ:

* 既定では、ランタイムが *Pages* フォルダー内で Razor ページ ファイルを検索します。
* `Index` は、URL にページが含まれない場合の既定のページになります。

## <a name="write-a-basic-form"></a>基本フォームを作成する

Razor ページは、アプリの構築時に Web ブラウザーで使用される一般的なパターンを実装しやすくするために設計されています。 [モデル バインド](xref:mvc/models/model-binding)、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、および HTML ヘルパーはすべて、Razor ページ クラスで定義されたプロパティで*機能します*。 `Contact` モデルの基本的な "お問い合わせ" フォームを実装するページを考察します。

このドキュメントのサンプルでは、[Startup.cs](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/razor-pages/index/3.0sample/RazorPagesContacts/Startup.cs#L23-L24) ファイルで `DbContext` が初期化されます。

[!code-cs[](index/3.0sample/RazorPagesContacts/Startup.cs?name=snippet)]

データ モデル:

[!code-cs[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

db コンテキスト:

[!code-cs[](index/3.0sample/RazorPagesContacts/Data/CustomerDbContext.cs)]

*Pages/Create.cshtml* ビュー ファイル:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

*Pages/Create.cshtml.cs* ページ モデル:

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_ALL)]

規則により、`PageModel` クラスは `<PageName>Model` と呼ばれ、ページと同じ名前空間にあります。

`PageModel` クラスでは、ページの表示からロジックを分離できます。 これは、ページに送信される要求のページ ハンドラーと、ページのレンダリングに使用されるデータを定義します。 この分離により可能になること:

* [依存関係の挿入](xref:fundamentals/dependency-injection)によるページの依存関係の管理。
* [単体テスト](xref:test/razor-pages-tests)

このページには、(ユーザーがフォームを投稿したときに) `POST` 要求で実行される `OnPostAsync` *ハンドラー メソッド*があります。 任意の HTTP 動詞のハンドラー メソッドを追加できます。 最も一般的なハンドラーは次のとおりです。

* ページに必要な状態を初期化するための `OnGet`。 前のコードでは、`OnGet` メソッドにより *CreateModel.cshtml* Razor ページが表示されます。
* フォームの送信を処理するための `OnPost`。

`Async` 名前付けサフィックスは省略可能ですが、非同期関数の規則でよく使用されます。 上記のコードは、Razor ページでは一般的です。

コントローラーとビューを利用する ASP.NET アプリに慣れている場合:

* 前の例の `OnPostAsync` コードは、一般的なコントローラー コードに似ています。
* [モデル バインド](xref:mvc/models/model-binding)、[検証](xref:mvc/models/validation)、アクションの結果など、MVC プリミティブのほとんどは Controllers ページや Razor ページと同じように動作します。 

上記の `OnPostAsync` メソッド:

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync)]

`OnPostAsync` の基本的な流れは次のとおりです。

検証エラーを確認します。

* エラーがない場合は、データを保存し、リダイレクトします。
* エラーがある場合は、検証メッセージとともにページをもう一度表示します。 多くの場合、検証エラーはクライアントで検出され、サーバーには送信されません。

*Pages/Create.cshtml* ビュー ファイル:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

*Pages/Create.cshtml* からレンダリングされた HTML:

[!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.html)]

前のコードで投稿したフォーム:

* 有効なデータ:

  * `OnPostAsync` ハンドラー メソッドにより <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*> ヘルパー メソッドが呼び出されます。 `RedirectToPage` は <xref:Microsoft.AspNetCore.Mvc.RedirectToPageResult> のインスタンスを返します。 `RedirectToPage`:

    * はアクションの結果です。
    * は、(コントローラーやビューで使用される) `RedirectToAction` や`RedirectToRoute` に似ています。
    * はページ用にカスタマイズされています。 上記のサンプルでは、ルート インデックス ページ (`/Index`) にリダイレクトします。 `RedirectToPage` については、「[ページの URL の生成](#url_gen)」セクションで詳しく説明されています。

* サーバーに検証エラーが渡される:

  * `OnPostAsync` ハンドラー メソッドにより <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Page*> ヘルパー メソッドが呼び出されます。 `Page` は <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageResult> のインスタンスを返します。 `Page` を返すのは、コントローラーのアクションが `View` を返す方法に似ています。 `PageResult` はハンドラー メソッドの既定の戻り値の型です。 `void` を返すハンドラー メソッドがページをレンダリングします。
  * 前の例では、値のないフォームが投稿された結果、[ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) から false が返されます。 このサンプルでは、検証エラーはクライアントに表示されません。 検証エラーの処理については、このドキュメントの後半で説明します。

  [!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

* クライアント側の検証で検証エラーが検出される:

  * データはサーバーに投稿されて**いません**。
  * クライアント側の検証については、このドキュメントの後半で説明します。

`Customer` プロパティは [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 属性を使用してモデル バインドにオプトインします。

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=15-16)]

`[BindProperty]` は、クライアントが変更するべきではないプロパティを含むモデルで使用**しないで**ください。 詳細については、「[過剰ポスティング](xref:data/ef-rp/crud#overposting)」をご覧ください。

既定では、Razor Pages はプロパティを非 `GET` 動詞とのみバインドします。 プロパティにバインドすると、HTTP データをモデル型に変換する目的でコードを記述する必要がなくなります。 同じプロパティを使用してバインドすることでコードを減らし、フィールド (`<input asp-for="Customer.Name">`) からレンダリングして入力を受け入れます。

[!INCLUDE[](~/includes/bind-get.md)]

*Pages/Create.cshtml* ビュー ファイルのレビュー:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml?highlight=3,9)]

* 前のコードでは、[入力タグ ヘルパー](xref:mvc/views/working-with-forms#the-input-tag-helper) `<input asp-for="Customer.Name" />` によって HTML `<input>` 要素が `Customer.Name` モデル式にバインドされます。
* [`@addTagHelper`](xref:mvc/views/tag-helpers/intro#addtaghelper-makes-tag-helpers-available) でタグ ヘルパーを使用可能にします。

### <a name="the-home-page"></a>ホーム ページ

*Index.cshtml* はホーム ページです。

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml)]

関連付けられた `PageModel` クラス (*Index.cshtml.cs*):

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet)]

*Index.cshtml* ファイルには、次のマークアップが含まれています。

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=21)]

`<a /a>` [アンカー タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)は `asp-route-{value}` 属性を使用して編集ページへのリンクを生成しました。 リンクには、連絡先 ID とともにルート データが含まれています。 たとえば、`https://localhost:5001/Edit/1` のようにします。 [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)を使うと、Razor ファイルでの HTML 要素の作成とレンダリングに、サーバー側コードを組み込むことができます。

*Index.cshtml* ファイルには、各顧客の連絡先の削除ボタンを作成するマークアップが含まれています。

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=22-23)]

レンダリングされた HTML:

```HTML
<button type="submit" formaction="/Customers?id=1&amp;handler=delete">delete</button>
```

HTML で削除ボタンがレンダリングされる場合、その [formaction](https://developer.mozilla.org/docs/Web/HTML/Element/button#attr-formaction) には次のパラメーターが含まれています。

* `asp-route-id` 属性によって指定された顧客の連絡先 ID。
* `asp-page-handler` 属性によって指定された `handler`。

ボタンが選択されると、フォームの `POST` 要求がサーバーに送信されます。 慣例により、ハンドラー メソッドの名前はスキーム `OnPost[handler]Async` に従った `handler` パラメーターの値に基づいて選択されます。

この例では `handler` が `delete` であるため、`OnPostDeleteAsync` ハンドラー メソッドを使用して `POST` 要求が処理されます。 `asp-page-handler` が `remove` などの別の値に設定されている場合、名前が `OnPostRemoveAsync` のハンドラー メソッドが選択されます。

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet2)]

`OnPostDeleteAsync` メソッド:

* クエリ文字列から `id` を取得します。
* `FindAsync` を使用してデータベースから顧客の連絡先を照会します。
* 顧客の連絡先が見つからない場合、それは削除されており、データベースが更新されています。
* ルート インデックス ページ (`/Index`) にリダイレクトされるように、<xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*> を呼び出します。

### <a name="the-editcshtml-file"></a>Edit.cshtml ファイル

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml?highlight=1)]

最初の行には `@page "{id:int}"` ディレクティブが含まれています。 ルーティングの制約 `"{id:int}"` は、`int` ルート データを含むページへの要求を受け入れるようにページに指示します。 ページへの要求に `int` に変換できるルート データが含まれていない場合は、ランタイムで HTTP 404 (見つかりません) エラーが返されます。 ID を省略するには、次のように `?` をルート制約に追加します。

 ```cshtml
@page "{id:int?}"
```

*Edit.cshtml.cs* ファイル:

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml.cs?name=snippet)]

## <a name="validation"></a>検証

検証規則:

* はモデル クラスで指定 (宣言) されます。
* はアプリ内のあらゆる場所で適用されます。

<xref:System.ComponentModel.DataAnnotations> 名前空間には、クラスまたはプロパティに宣言的に適用される一連の組み込みの検証属性があります。 また、DataAnnotations には、書式設定を支援し、どの検証を行わない [`[DataType]`](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) のような書式設定属性もあります。

`Customer` モデルを考えてみましょう。

[!code-cs[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

次の *Create.cshtml* ビュー ファイルを使用:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=3,8-9,15-99)]

上のコードでは以下の操作が行われます。

* jQuery と jQuery 検証スクリプトが含まれます。
* `<div />` と `<span />` [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)を使用して次を有効にします。

  * クライアント側の検証。
  * 検証エラー レンダリング。

* 次の HTML が生成されます。

  [!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create5.html)]

名前値なしで Create フォームを投稿すると、このフォームに "The Name field is required." (名前フィールドは必須です。) というエラー メッセージ が表示されます。 JavaScript がクライアントで有効になっている場合、サーバーに投稿されず、エラーがブラウザーに表示されます。

`[StringLength(10)]` 属性によって、レンダリングされた HTML で `data-val-length-max="10"` が生成されます。 `data-val-length-max` により、指定の最大長を超える入力がブラウザーで禁止されます。 [Fiddler](https://www.telerik.com/fiddler) のようなツールを投稿の編集と返信に使用する場合:

* 名前の長さ値が 10 を超えています。
* "The field Name must be a string with a maximum length of 10." (名前フィールドは最大長が 10 の文字列になります。) というエラー メッセージが 返されます。

次の `Movie` モデルがあるとします。

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet1)]

検証属性では、適用対象のモデル プロパティに適用する動作が指定されます。

* `Required` および `MinimumLength` 属性は、プロパティに値が必要であることを示します。ただし、この検証を満たすためにユーザーが空白を入力することは禁止されていません。
* `RegularExpression` 属性は、入力できる文字を制限するために使用されます。 上のコード "Genre" では:

  * 文字のみを使用する必要があります。
  * 最初の文字は大文字にする必要があります。 空白、数字、特殊文字は使用できません。

* `RegularExpression` "評価":

  * 最初の文字が大文字である必要があります。
  * 後続のスペースでは、特殊文字と数字が使用できます。 "PG-13" は評価に対して有効ですが、"Genre" に対しては失敗します。

* `Range` 属性は、指定した範囲内に値を制限します。
* `StringLength` 属性により、文字列プロパティの最大長が設定されます。任意で最小長も設定できます。
* 値の型 (`decimal`、`int`、`float`、`DateTime` など) は本質的に必須ではなく、`[Required]` 属性を必要としません。

`Movie` モデルの [作成] ページには、エラーと無効な値が表示されます。

![複数 jQuery クライアント側検証エラーが表示されたムービー ビュー フォーム](~/tutorials/razor-pages/validation/_static/val.png)

詳細については次を参照してください:

* [Movie アプリに検証を追加する](xref:tutorials/razor-pages/validation)
* [ASP.NET Core のモデル検証](xref:mvc/models/validation).

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>OnGet ハンドラー フォールバックを使用した HEAD 要求の処理

`HEAD` 要求により、特定のリソースのヘッダーを取得できます。 `GET` 要求とは異なり、`HEAD` 要求から応答本文は返されません。

通常、`HEAD` 要求に対して `OnHead` ハンドラーが作成され、呼び出されます。

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Privacy.cshtml.cs?name=snippet)]

`OnHead` ハンドラーが定義されていない場合、Razor Pages は `OnGet` ハンドラーの呼び出しにフォールバックします。

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-razor-pages"></a>XSRF/CSRF と Razor ページ

Razor Pages は、[偽造防止検証](xref:security/anti-request-forgery)によって保護されます。 [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) により HTML フォーム要素に偽造防止トークンが挿入されます。

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-razor-pages"></a>Razor ページでのレイアウト、パーシャル、テンプレート、およびタグ ヘルパーの使用

ページは、Razor ビュー エンジンのすべての機能で動作します。 レイアウト、パーシャル、テンプレート、タグ ヘルパー、 *_ViewStart.cshtml*、 *_ViewImports.cshtml* は、従来の Razor ビューの場合と同じように動作します。

これらの機能の一部を利用してこのページをまとめてみましょう。

[レイアウト ページ](xref:mvc/views/layout)を *Pages/Shared/_Layout.cshtml* に追加します。

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Shared/_Layout2.cshtml?hightlight=12)]

[レイアウト](xref:mvc/views/layout)は次のことを行います。

* (ページでレイアウトを止めない限り) 各ページのレイアウトを制御します。
* JavaScript やスタイルシートなどの HTML 構造をインポートします。
* `@RenderBody()` が呼び出されるところで Razor ページの内容が表示されます。

詳細については、[レイアウトに関するページ](xref:mvc/views/layout)を参照してください。

[Layout](xref:mvc/views/layout#specifying-a-layout) プロパティは *Pages/_ViewStart.cshtml* で設定されています。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

レイアウトは、*Pages/Shared* フォルダーにあります。 ページは現在のページと同じフォルダーから開始して、階層的に他のビュー (レイアウト、テンプレート、パーシャル) を検索します。 *Pages/Shared* フォルダー内のレイアウトは、*Pages* フォルダー配下の任意の Razor ページから使用できます。

レイアウト ファイルは *Pages/Shared* フォルダーに入ります。

レイアウト ファイルを *Views/Shared* フォルダー内に配置**しない**ことをお勧めします。 *Views/Shared* は MVC ビュー パターンです。 Razor ページは、パス規則ではなく、フォルダー階層に依存することを意図しています。

Razor ページからのビュー検索には、*Pages* フォルダーが含まれます。 MVC コントローラーで使用されているレイアウト、テンプレート、パーシャルと、従来の Razor ビューは*機能します*。

*Pages/_ViewImports.cshtml* ファイルを追加します。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

`@namespace` はこのチュートリアルで後ほど説明します。 `@addTagHelper` ディレクティブにより、[組み込みタグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/Index)が *Pages* フォルダー内のすべてのページにもたらされます。

<a name="namespace"></a>

ページに設定される `@namespace` ディレクティブ:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

`@namespace` ディレクティブは、ページの名前空間を設定します。 `@model` ディレクティブには、名前空間を含める必要はありません。

`@namespace` ディレクティブが *_ViewImports.cshtml* に含まれていると、指定した名前空間が `@namespace` ディレクティブをインポートするページで生成された名前空間のプレフィックスを提供します。 生成された名前空間の残りの部分 (サフィックスの部分) は、 *_ViewImports.cshtml* を含むフォルダーとページを含むフォルダー間のドットで区切られた相対パスです。

たとえば、`PageModel` クラス *Pages/Customers/Edit.cshtml.cs* は名前空間を明示的に設定します。

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

*Pages/_ViewImports.cshtml* ファイルは次の名前空間を設定します。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

*Pages/Customers/Edit.cshtml* Razor ページの生成された名前空間は、`PageModel` クラスと同じです。

`@namespace`  *は従来の Razor ビューでも機能します。*

*Pages/Create.cshtml* ビュー ファイル を考えてみましょう。

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=2-3)]

更新後の *Pages/Create.cshtml* ビュー ファイル、 *_ViewImports.cshtml*、前のレイアウト ファイル:

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.cshtml?highlight=2)]

前のコードでは、 *_ViewImports.cshtml* によって名前空間とタグ ヘルパーがインポートされました。 レイアウト ファイルによって JavaScript ファイルがインポートされました。

[Razor ページのスタート プロジェクト](#rpvs17)には、クライアント側の検証をフックする *Pages/_ValidationScriptsPartial.cshtml* が含まれています。

部分ビューの詳細については、「<xref:mvc/views/partial>」を参照してください。

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>ページの URL の生成

上に示した `Create` ページでは、`RedirectToPage` を使用します。

[!code-cs[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=28)]

アプリには次のファイル/フォルダー構造があります。

* */Pages*

  * *Index.cshtml*
  * *Privacy.cshtml*
  * */Customers*

    * *Create.cshtml*
    * *Edit.cshtml*
    * *Index.cshtml*

成功すると、*Pages/Customers/Create.cshtml* ページと *Pages/Customers/Edit.cshtml* ページが *Pages/Customers/Index.cshtml* にリダイレクトされます。 文字列 `./Index` は、前のページにアクセスするために使用される相対ページ名です。 これは、*Pages/Customers/Index.cshtml* ページへの URI を生成するために使われます。 次に例を示します。

* `Url.Page("./Index", ...)`
* `<a asp-page="./Index">Customers Index Page</a>`
* `RedirectToPage("./Index")`

絶対ページ名 `/Index` は、*Pages/Index.cshtml* ページへの URL を生成するために使われます。 次に例を示します。

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">Home Index Page</a>`
* `RedirectToPage("/Index")`

ページ名は、先頭の `/` を含む、ルート */Pages* フォルダーからページへのパスです (たとえば `/Index`)。 先述の URL 生成サンプルでは、URL のハードコーディングに関する拡張オプションと機能が提供されます。 URL の生成は[ルーティング](xref:mvc/controllers/routing)を使用し、ターゲット パスで定義されたルート方法に従って、パラメーターの生成とエンコードができます。

ページの URL 生成は、相対名をサポートします。 次の表に、*Pages/Customers/Create.cshtml* の異なる `RedirectToPage` パラメーターで選択されたインデックス ページを示します。

| RedirectToPage(x)| ページ |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

<!-- Test via ~/razor-pages/index/3.0sample/RazorPagesContacts/Pages/Customers/Details.cshtml.cs -->

`RedirectToPage("Index")`、`RedirectToPage("./Index")`、`RedirectToPage("../Index")` は*相対名*です。 `RedirectToPage` パラメーターは現在のページのパスと*組み合わされて*、ターゲット ページの名前を計算します。

相対名のリンクは、複雑な構造を持つサイトを構築する際に役立ちます。 あるフォルダー内のページ間をリンクする目的で相対名を使用するとき:

* フォルダー名を変更しても相対リンクは壊れません。
* フォルダー名が含まれていないため、リンクは壊れません。

別の [[区分]](xref:mvc/controllers/areas) のページにリダイレクトするには、その区分を指定します。

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

詳細については、次のトピックを参照してください。 <xref:mvc/controllers/areas> および <xref:razor-pages/razor-pages-conventions>

## <a name="viewdata-attribute"></a>ViewData 属性

データは <xref:Microsoft.AspNetCore.Mvc.ViewDataAttribute> を含むページに渡すことができます。 `[ViewData]` 属性を含むプロパティにはその値が格納され、<xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary> から読み込まれます。

次の例では、`AboutModel` により、`[ViewData]` 属性が `Title` プロパティに適用されます。

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

[About] ページでは、モデル プロパティとして `Title` プロパティにアクセスします。

```cshtml
<h1>@Model.Title</h1>
```

レイアウトでは、タイトルは ViewData ディクショナリから読み込まれます。

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET Core により <xref:Microsoft.AspNetCore.Mvc.Controller.TempData> が公開されます。 このプロパティは、読み取られるまでデータを格納します。 <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Keep*> メソッドと <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Peek*> メソッドは、削除せずにデータを確認するために使用できます。 `TempData` は、複数の要求に対してデータが必要な場合のリダイレクトに役立ちます。

次のコードは、`TempData` を使用して `Message` の値を設定します。

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

*Pages/Customers/Index.cshtml* ファイル内の次のマークアップは、`TempData` を使用して `Message` の値を表示します。

```cshtml
<h3>Msg: @Model.Message</h3>
```

*Pages/Customers/Index.cshtml.cs* ページは、`[TempData]` 属性を `Message` プロパティに適用します。

```cs
[TempData]
public string Message { get; set; }
```

詳細については、「[TempData](xref:fundamentals/app-state#tempdata)」を参照してください。

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>ページあたり複数のハンドラー

次のページでは、`asp-page-handler` タグ ヘルパーを使用して 2 つのハンドラーにマークアップが生成されます。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

前の例のフォームには、それぞれが `FormActionTagHelper` を使用して異なる URL に送信する 2 つの送信ボタンがあります。 `asp-page-handler` 属性は、`asp-page` のコンパニオンです。 `asp-page-handler` はページごとに定義されている各ハンドラー メソッドに送信する URL を生成します。 サンプルは現在のページにリンクしているため、`asp-page` は指定されません。

ページ モデル:

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

上記のコードは、*名前付きハンドラー メソッド*を使用しています。 名前付きハンドラー メソッドは、名前の `On<HTTP Verb>` の後および `Async` の前 (ある場合) のテキストを取得して作成されます。 前の例では、ページ メソッドは OnPost**JoinList**Async と OnPost**JoinListUC**Async です。 *OnPost* と *Async* を削除すると、ハンドラー名は `JoinList` と `JoinListUC` になります。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

上記のコードを使用すると、`OnPostJoinListAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH?handler=JoinList` になります。 `OnPostJoinListUCAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC` です。

## <a name="custom-routes"></a>カスタム ルート

`@page` ディレクティブを次に使用します:

* カスタム ルートをページに指定します。 たとえば、[バージョン情報] ページへのルートを `@page "/Some/Other/Path"` を使用して `/Some/Other/Path` に設定することができます。
* ページの既定のルートにセグメントを追加します。 たとえば、"item" セグメントを `@page "item"` を使用してページの既定のルートに追加することができます。
* ページの既定のルートにパラメーターを追加します。 たとえば、`@page "{id}"` を含むページに ID パラメーター `id` を必須とすることができます。

パスの先頭のチルダ (`~`) によって指定されたルートの相対パスがサポートされます。 たとえば、`@page "~/Some/Other/Path"` は `@page "/Some/Other/Path"` と同じです。

ルート テンプレート `@page "{handler?}"` を指定することで、URL のクエリ文字列 `?handler=JoinList` をルート セグメント `/JoinList` に変更することができます。

URL 内のクエリ文字列 `?handler=JoinList` が気に入らない場合は、ルートを変更して URL のパス部分にハンドラー名を挿入することができます。 `@page` ディレクティブの後に二重引用符で囲んだルート テンプレートを追加して、ルートをカスタマイズすることができます。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

上記のコードを使用すると、`OnPostJoinListAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH/JoinList` になります。 `OnPostJoinListUCAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH/JoinListUC` です。

`handler` の後の `?` は、ルート パラメーターが省略可能なことを意味します。

## <a name="advanced-configuration-and-settings"></a>詳細な構成と設定

次のセクションの構成と設定はほとんどのアプリで必要ありません。

高度なオプションを構成するには、拡張メソッド <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> を使用します。

[!code-cs[](index/3.0sample/RazorPagesContacts/StartupRPoptions.cs?name=snippet)]

<xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions> を使用してページのルート ディレクトリを設定したり、ページのアプリケーション モデルの規則を追加したりできます。 規則の詳細については、「[Razor ページの承認規則](xref:security/authorization/razor-pages-authorization)」を参照してください。

ビューをプリコンパイルするには、[Razor ビューのコンパイル](xref:mvc/views/view-compilation)に関するページをご覧ください。

### <a name="specify-that-razor-pages-are-at-the-content-root"></a>Razor ページをコンテンツのルートに指定する

Razor ページのルートは既定で */Pages* ディレクトリです。 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.WithRazorPagesAtContentRoot*> を追加して、Razor Pages をアプリの[コンテンツ ルート](xref:fundamentals/index#content-root) (<xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath>) に置くように指定します。

[!code-cs[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesAtContentRoot.cs?name=snippet)]

### <a name="specify-that-razor-pages-are-at-a-custom-root-directory"></a>Razor ページをカスタム ルート ディレクトリに指定する

(相対パスを指定して) Razor ページをアプリのカスタム ルート ディレクトリに置くように指定するには、<xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcCoreBuilderExtensions.WithRazorPagesRoot*> を追加します。

[!code-cs[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesRoot.cs?name=snippet)]

## <a name="additional-resources"></a>その他の技術情報

* この概要に基づく、[Razor Pages の概要](xref:tutorials/razor-pages/razor-pages-start)に関するページをご覧ください
* [サンプル コードのダウンロードまたは表示](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/index/3.0sample)
* <xref:index>
* <xref:mvc/views/razor>
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT) および [Ryan Nowak](https://github.com/rynowak)

Razor ページは、ページ コーディングに重点を置いたシナリオをより簡略化し、生産性を高める ASP.NET Core MVC の新たな側面です。

モデル ビュー コントローラーのアプローチを使用するチュートリアルをお探しの場合は、「[Get started with ASP.NET Core MVC](xref:tutorials/first-mvc-app/start-mvc)」 (ASP.NET Core MVC の概要) を参照してください。

このドキュメントでは、Razor ページの概要について説明します。 手順を追って説明するチュートリアルではありません。 セクションの一部を理解できない場合は、「[Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)」を参照してください。 ASP.NET Core の概要については、「[ASP.NET Core の概要](xref:index)」を参照してください。

## <a name="prerequisites"></a>必須コンポーネント

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

<a name="rpvs17"></a>

## <a name="create-a-razor-pages-project"></a>Razor ページ プロジェクトを作成する

# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

Razor ページ プロジェクトを作成する詳細な手順については、「[Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)」を参照してください。

# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

コマンド ラインから `dotnet new webapp` を実行します。

Visual Studio for Mac から生成された *.csproj* ファイルを開きます。

# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

コマンド ラインから `dotnet new webapp` を実行します。

---

## <a name="razor-pages"></a>Razor ページ

Razor ページは *Startup.cs* で有効になっています。

[!code-cs[](index/sample/RazorPagesIntro/Startup.cs?name=snippet_Startup)]

基本ページを検討します。<a name="OnGet"></a>

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index.cshtml)]

上記のコードは、コントローラーとビューを含んだ ASP.NET Core アプリで使われる [Razor ビュー ファイル](xref:tutorials/first-mvc-app/adding-view)によく似ています。 違いは、`@page` ディレクティブにあります。 `@page` はファイルを MVC アクションにします。つまり、コントローラーを経由せずに要求を直接処理します。 `@page` はページで最初の Razor ディレクティブである必要があります。 `@page` はその他の Razor コンストラクトの動作に影響します。

`PageModel` クラスを使用している類似したページが、次の 2 つのファイルにあります。 *Pages/Index2.cshtml* ファイル:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index2.cshtml)]

*Pages/Index2.cshtml.cs* ページ モデル:

[!code-cs[](index/sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

規則により、`PageModel` クラス ファイルは、Razor ページ ファイルと同じ名前に *.cs* が付加された名前になります。 たとえば、上の Razor ページは *Pages/Index2.cshtml* になります。 `PageModel` クラスを含むファイル名は、*Pages/Index2.cshtml.cs* になります。

URL パスのページへの関連付けは、ファイル システム内のページの場所によって決定されます。 次の表に、Razor ページ パスと一致 URL を示します。

| ファイル名とパス               | 一致 URL |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` または `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` または `/Store/Index` |

メモ:

* 既定では、ランタイムが *Pages* フォルダー内で Razor ページ ファイルを検索します。
* `Index` は、URL にページが含まれない場合の既定のページになります。

## <a name="write-a-basic-form"></a>基本フォームを作成する

Razor ページは、アプリの構築時に Web ブラウザーで使用される一般的なパターンを実装しやすくするために設計されています。 [モデル バインド](xref:mvc/models/model-binding)、[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)、および HTML ヘルパーはすべて、Razor ページ クラスで定義されたプロパティで*機能します*。 `Contact` モデルの基本的な "お問い合わせ" フォームを実装するページを考察します。

このドキュメントのサンプルでは、[Startup.cs](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/razor-pages/index/sample/RazorPagesContacts/Startup.cs#L15-L16) ファイルで `DbContext` が初期化されます。

[!code-cs[](index/sample/RazorPagesContacts/Startup.cs?highlight=15-16)]

データ モデル:

[!code-cs[](index/sample/RazorPagesContacts/Data/Customer.cs)]

db コンテキスト:

[!code-cs[](index/sample/RazorPagesContacts/Data/AppDbContext.cs)]

*Pages/Create.cshtml* ビュー ファイル:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml)]

*Pages/Create.cshtml.cs* ページ モデル:

[!code-cs[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_ALL)]

規則により、`PageModel` クラスは `<PageName>Model` と呼ばれ、ページと同じ名前空間にあります。

`PageModel` クラスでは、ページの表示からロジックを分離できます。 これは、ページに送信される要求のページ ハンドラーと、ページのレンダリングに使用されるデータを定義します。 この分離により可能になること:

* [依存関係の挿入](xref:fundamentals/dependency-injection)によるページの依存関係の管理。
* ページを[単体テスト](xref:test/razor-pages-tests)します。

このページには、(ユーザーがフォームを投稿したときに) `POST` 要求で実行される `OnPostAsync` *ハンドラー メソッド*があります。 任意の HTTP 動詞のハンドラー メソッドを追加できます。 最も一般的なハンドラーは次のとおりです。

* ページに必要な状態を初期化するための `OnGet`。 [OnGet](#OnGet) サンプル。
* フォームの送信を処理するための `OnPost`。

`Async` 名前付けサフィックスは省略可能ですが、非同期関数の規則でよく使用されます。 上記のコードは、Razor ページでは一般的です。

コントローラーとビューを利用する ASP.NET アプリに慣れている場合:

* 前の例の `OnPostAsync` コードは、一般的なコントローラー コードに似ています。
* [モデル バインド](xref:mvc/models/model-binding)、[検証](xref:mvc/models/validation)、[検証](xref:mvc/models/validation) アクションの結果などのほとんどの MVC プリミティブは共有されます。

上記の `OnPostAsync` メソッド:

[!code-cs[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync)]

`OnPostAsync` の基本的な流れは次のとおりです。

検証エラーを確認します。

* エラーがない場合は、データを保存し、リダイレクトします。
* エラーがある場合は、検証メッセージとともにページをもう一度表示します。 クライアント側の検証は、従来の ASP.NET Core MVC アプリケーションと同じです。 多くの場合、検証エラーはクライアントで検出され、サーバーには送信されません。

データが正常に入力されると、`OnPostAsync` ハンドラー メソッドが `RedirectToPage` ヘルパー メソッドを呼び出して `RedirectToPageResult` のインスタンスを返します。 `RedirectToPage` は、`RedirectToAction` や `RedirectToRoute` と同じような新しいアクション結果ですが、ページ用にカスタマイズされています。 上記のサンプルでは、ルート インデックス ページ (`/Index`) にリダイレクトします。 `RedirectToPage` については、「[ページの URL の生成](#url_gen)」セクションで詳しく説明されています。

送信されたフォームに検証エラー (サーバーに渡される) があると、`OnPostAsync` ハンドラー メソッドが `Page` ヘルパー メソッドを呼び出します。 `Page` は `PageResult` のインスタンスを返します。 `Page` を返すのは、コントローラーのアクションが `View` を返す方法に似ています。 `PageResult` はハンドラー メソッドの既定の戻り値の型です。 `void` を返すハンドラー メソッドがページをレンダリングします。

`Customer` プロパティは `[BindProperty]` 属性を使用してモデル バインドにオプトインします。

[!code-cs[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_PageModel&highlight=10-11)]

既定では、Razor Pages はプロパティを非 `GET` 動詞とのみバインドします。 プロパティをバインドすることで、記述すべきコードの量を削減できます。 同じプロパティを使用してバインドすることでコードを減らし、フィールド (`<input asp-for="Customer.Name">`) からレンダリングして入力を受け入れます。

[!INCLUDE[](~/includes/bind-get.md)]

ホーム ページ (*Index.cshtml*):

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml)]

関連付けられた `PageModel` クラス (*Index.cshtml.cs*):

[!code-cs[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs)]

*Index.cshtml* ファイルには、各連絡先の編集リンクを作成するために次のマークアップが含まれています。

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=21)]

`<a asp-page="./Edit" asp-route-id="@contact.Id">Edit</a>` [アンカー タグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)は `asp-route-{value}` 属性を使用して編集ページへのリンクを生成しました。 リンクには、連絡先 ID とともにルート データが含まれています。 たとえば、`https://localhost:5001/Edit/1` のようにします。 [タグ ヘルパー](xref:mvc/views/tag-helpers/intro)を使うと、Razor ファイルでの HTML 要素の作成とレンダリングに、サーバー側コードを組み込むことができます。 タグ ヘルパーは `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` によって有効になります

*Pages/Edit.cshtml* ファイル:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Edit.cshtml?highlight=1)]

最初の行には `@page "{id:int}"` ディレクティブが含まれています。 ルーティングの制約 `"{id:int}"` は、`int` ルート データを含むページへの要求を受け入れるようにページに指示します。 ページへの要求に `int` に変換できるルート データが含まれていない場合は、ランタイムで HTTP 404 (見つかりません) エラーが返されます。 ID を省略するには、次のように `?` をルート制約に追加します。

 ```cshtml
@page "{id:int?}"
```

*Pages/Edit.cshtml.cs* ファイル:

[!code-cs[](index/sample/RazorPagesContacts/Pages/Edit.cshtml.cs)]

*Index.cshtml* ファイルには、各顧客の連絡先の削除ボタンを作成するマークアップも含まれています。

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=22-23)]

HTML で削除ボタンがレンダリングされる場合、その `formaction` には次のパラメーターが含まれています。

* `asp-route-id` 属性によって指定された顧客の連絡先 ID。
* `asp-page-handler` 属性によって指定された `handler`。

顧客の連絡先 ID `1` でレンダリングされた削除ボタンの例を示します。

```html
<button type="submit" formaction="/?id=1&amp;handler=delete">delete</button>
```

ボタンが選択されると、フォームの `POST` 要求がサーバーに送信されます。 慣例により、ハンドラー メソッドの名前はスキーム `OnPost[handler]Async` に従った `handler` パラメーターの値に基づいて選択されます。

この例では `handler` が `delete` であるため、`OnPostDeleteAsync` ハンドラー メソッドを使用して `POST` 要求が処理されます。 `asp-page-handler` が `remove` などの別の値に設定されている場合、名前が `OnPostRemoveAsync` のハンドラー メソッドが選択されます。 次のコードは、`OnPostDeleteAsync` ハンドラーを示しています。

[!code-cs[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs?range=26-37)]

`OnPostDeleteAsync` メソッド:

* クエリ文字列から `id` を受け入れます。 *Index.cshtml* ページ ディレクティブにルーティング制約 `"{id:int?}"` が含まれていた場合、`id` はルート データから取得されることがあります。 `id` のルート データは `https://localhost:5001/Customers/2` のように URI で指定されます。
* `FindAsync` を使用してデータベースから顧客の連絡先を照会します。
* 顧客の連絡先が見つかった場合、その連絡先は顧客の連絡先の一覧から削除されています。 データベースが更新されます。
* ルート インデックス ページ (`/Index`) にリダイレクトされるように、`RedirectToPage` を呼び出します。

## <a name="mark-page-properties-as-required"></a>必要に応じてページのプロパティをマークする

`PageModel` 上でのプロパティを [必要](/dotnet/api/system.componentmodel.dataannotations.requiredattribute)属性で装飾できます。

[!code-cs[](index/sample/Create.cshtml.cs?highlight=3,15-16)]

詳細については、[モデルの検証](xref:mvc/models/validation)に関するページを参照してください。

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>OnGet ハンドラー フォールバックを使用した HEAD 要求の処理

`HEAD`HEAD 要求を使用すると、特定のリソースに対するヘッダーを取得できます。 `GET` 要求とは異なり、`HEAD` 要求から応答本文は返されません。

通常、`HEAD` 要求に対して `OnHead` ハンドラーが作成され、呼び出されます。 

```csharp
public void OnHead()
{
    HttpContext.Response.Headers.Add("HandledBy", "Handled by OnHead!");
}
```

ASP.NET Core 2.1 以降では、`OnHead` ハンドラーが定義されていない場合、Razor Pages は `OnGet` ハンドラーの呼び出しにフォールバックします。 この動作は、`Startup.ConfigureServices` での [SetCompatibilityVersion](xref:mvc/compatibility-version) への呼び出しによって有効になります。

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

既定のテンプレートでは、ASP.NET Core 2.1 および 2.2 で `SetCompatibilityVersion` の呼び出しが生成されます。 `SetCompatibilityVersion` は実質的に Razor ページのオプション `AllowMappingHeadRequestsToGetHandler` を `true` に設定します。

`SetCompatibilityVersion` とのすべての動作にオプトインするのではなく、明示的に*特定の*動作にオプトインすることもできます。 次のコードでは、`OnGet` ハンドラーに `HEAD` 要求をマップできるようにすることにオプトインしています。

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.AllowMappingHeadRequestsToGetHandler = true;
    });
```

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-razor-pages"></a>XSRF/CSRF と Razor ページ

[偽造防止検証](xref:security/anti-request-forgery)のためにコードを記述する必要はありません。 偽造防止トークンの生成と検証は、自動的に Razor ページに含まれます。

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-razor-pages"></a>Razor ページでのレイアウト、パーシャル、テンプレート、およびタグ ヘルパーの使用

ページは、Razor ビュー エンジンのすべての機能で動作します。 レイアウト、パーシャル、テンプレート、タグ ヘルパー、 *_ViewStart.cshtml*、 *_ViewImports.cshtml* は、従来の Razor ビューと同じように動作します。

これらの機能の一部を利用してこのページをまとめてみましょう。

[レイアウト ページ](xref:mvc/views/layout)を *Pages/Shared/_Layout.cshtml* に追加します。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_LayoutSimple.cshtml)]

[レイアウト](xref:mvc/views/layout)は次のことを行います。

* (ページでレイアウトを止めない限り) 各ページのレイアウトを制御します。
* JavaScript やスタイルシートなどの HTML 構造をインポートします。

詳細については、[レイアウトのページ](xref:mvc/views/layout)を参照してください。

[Layout](xref:mvc/views/layout#specifying-a-layout) プロパティは *Pages/_ViewStart.cshtml* で設定されています。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

レイアウトは、*Pages/Shared* フォルダーにあります。 ページは現在のページと同じフォルダーから開始して、階層的に他のビュー (レイアウト、テンプレート、パーシャル) を検索します。 *Pages/Shared* フォルダー内のレイアウトは、*Pages* フォルダー配下の任意の Razor ページから使用できます。

レイアウト ファイルは *Pages/Shared* フォルダーに入ります。

レイアウト ファイルを *Views/Shared* フォルダー内に配置**しない**ことをお勧めします。 *Views/Shared* は MVC ビュー パターンです。 Razor ページは、パス規則ではなく、フォルダー階層に依存することを意図しています。

Razor ページからのビュー検索には、*Pages* フォルダーが含まれます。 MVC コントローラーで使用しているレイアウト、テンプレート、およびパーシャルと、従来の Razor ビューは*機能します*。

*Pages/_ViewImports.cshtml* ファイルを追加します。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

`@namespace` はこのチュートリアルで後ほど説明します。 `@addTagHelper` ディレクティブにより、[組み込みタグ ヘルパー](xref:mvc/views/tag-helpers/builtin-th/Index)が *Pages* フォルダー内のすべてのページにもたらされます。

<a name="namespace"></a>

ページで `@namespace` ディレクティブが明示的に使用されている場合:

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

ディレクティブは、ページの名前空間を設定します。 `@model` ディレクティブには、名前空間を含める必要はありません。

`@namespace` ディレクティブが *_ViewImports.cshtml* に含まれていると、指定した名前空間が `@namespace` ディレクティブをインポートするページで生成された名前空間のプレフィックスを提供します。 生成された名前空間の残りの部分 (サフィックスの部分) は、 *_ViewImports.cshtml* を含むフォルダーとページを含むフォルダー間のドットで区切られた相対パスです。

たとえば、`PageModel` クラス *Pages/Customers/Edit.cshtml.cs* は名前空間を明示的に設定します。

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

*Pages/_ViewImports.cshtml* ファイルは次の名前空間を設定します。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

*Pages/Customers/Edit.cshtml* Razor ページの生成された名前空間は、`PageModel` クラスと同じです。

`@namespace`  *は従来の Razor ビューでも機能します。*

元の *Pages/Create.cshtml* ビュー ファイル:

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml?highlight=2)]

更新された *Pages/Create.cshtml* ビュー ファイル:

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/Create.cshtml?highlight=2)]

[Razor ページのスタート プロジェクト](#rpvs17)には、クライアント側の検証をフックする *Pages/_ValidationScriptsPartial.cshtml* が含まれています。

部分ビューの詳細については、「<xref:mvc/views/partial>」を参照してください。

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>ページの URL の生成

上に示した `Create` ページでは、`RedirectToPage` を使用します。

[!code-cs[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=10)]

アプリには次のファイル/フォルダー構造があります。

* */Pages*

  * *Index.cshtml*
  * */Customers*

    * *Create.cshtml*
    * *Edit.cshtml*
    * *Index.cshtml*

成功すると、*Pages/Customers/Create.cshtml* ページと *Pages/Customers/Edit.cshtml* ページが *Pages/Index.cshtml* にリダイレクトされます。 文字列 `/Index` は前のページにアクセスするための URI の一部です。 文字列 `/Index` は、*Pages/Index.cshtml* ページへの URI を生成するために使用できます。 次に例を示します。

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">My Index Page</a>`
* `RedirectToPage("/Index")`

ページ名は、先頭の `/` を含む、ルート */Pages* フォルダーからページへのパスです (たとえば `/Index`)。 先述の URL 生成サンプルでは、URL のハードコーディングに関する拡張オプションと機能が提供されます。 URL の生成は[ルーティング](xref:mvc/controllers/routing)を使用し、ターゲット パスで定義されたルート方法に従って、パラメーターの生成とエンコードができます。

ページの URL 生成は、相対名をサポートします。 次の表に、*Pages/Customers/Create.cshtml* の異なる `RedirectToPage` パラメーターで選択されたインデックス ページを示します。

| RedirectToPage(x)| ページ |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

`RedirectToPage("Index")`、`RedirectToPage("./Index")`、および `RedirectToPage("../Index")` は*相対名*です。 `RedirectToPage` パラメーターは現在のページのパスと*組み合わされて*、ターゲット ページの名前を計算します。  <!-- Review: Original had The provided string is combined with the page name of the current page to compute the name of the destination page.  page name, not page path -->

相対名のリンクは、複雑な構造を持つサイトを構築する際に役立ちます。 相対名を使用してフォルダー内のページ間をリンクする場合、そのフォルダー名を変更することができます。 すべてのリンクは引き続き機能します (リンクにはフォルダー名が含まれていないため)。

別の [[区分]](xref:mvc/controllers/areas) のページにリダイレクトするには、その区分を指定します。

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

詳細については、<xref:mvc/controllers/areas> を参照してください。

## <a name="viewdata-attribute"></a>ViewData 属性

データは [ViewDataAttribute](/dotnet/api/microsoft.aspnetcore.mvc.viewdataattribute) とのページに渡すことができます。 コントローラーまたは `[ViewData]` で装飾された Razor ページのモデルのプロパティは、値を [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) に格納し、読み込むことができます。

次の例では、`AboutModel` には `[ViewData]` で装飾された `Title` プロパティが含まれています。 `Title` プロパティは、[About] ページのタイトルに設定されます。

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

[About] ページでは、モデル プロパティとして `Title` プロパティにアクセスします。

```cshtml
<h1>@Model.Title</h1>
```

レイアウトでは、タイトルは ViewData ディクショナリから読み込まれます。

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET Core は [コントローラー](/dotnet/api/microsoft.aspnetcore.mvc.controller)上で [TempData](/dotnet/api/microsoft.aspnetcore.mvc.controller.tempdata?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Controller_TempData) プロパティを公開します。 このプロパティは、読み取られるまでデータを格納します。 `Keep` メソッドと `Peek` メソッドは、削除せずにデータを確認するために使用できます。 `TempData` は、複数の要求にデータが必要な場合のリダイレクトに役立ちます。

次のコードは、`TempData` を使用して `Message` の値を設定します。

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

*Pages/Customers/Index.cshtml* ファイル内の次のマークアップは、`TempData` を使用して `Message` の値を表示します。

```cshtml
<h3>Msg: @Model.Message</h3>
```

*Pages/Customers/Index.cshtml.cs* ページは、`[TempData]` 属性を `Message` プロパティに適用します。

```cs
[TempData]
public string Message { get; set; }
```

詳細については、「[TempData](xref:fundamentals/app-state#tempdata)」を参照してください。

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>ページあたり複数のハンドラー

次のページでは、`asp-page-handler` タグ ヘルパーを使用して 2 つのハンドラーにマークアップが生成されます。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

<!-- Review: the FormActionTagHelper applies to all <form /> elements on a Razor page, even when there's no `asp-` attribute   -->

前の例のフォームには、それぞれが `FormActionTagHelper` を使用して異なる URL に送信する 2 つの送信ボタンがあります。 `asp-page-handler` 属性は、`asp-page` のコンパニオンです。 `asp-page-handler` はページごとに定義されている各ハンドラー メソッドに送信する URL を生成します。 サンプルは現在のページにリンクしているため、`asp-page` は指定されません。

ページ モデル:

[!code-cs[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

上記のコードは、*名前付きハンドラー メソッド*を使用しています。 名前付きハンドラー メソッドは、名前の `On<HTTP Verb>` の後および `Async` の前 (ある場合) のテキストを取得して作成されます。 前の例では、ページ メソッドは OnPost**JoinList**Async と OnPost**JoinListUC**Async です。 *OnPost* と *Async* を削除すると、ハンドラー名は `JoinList` と `JoinListUC` になります。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

上記のコードを使用すると、`OnPostJoinListAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH?handler=JoinList` になります。 `OnPostJoinListUCAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC` です。

## <a name="custom-routes"></a>カスタム ルート

`@page` ディレクティブを次に使用します:

* カスタム ルートをページに指定します。 たとえば、[バージョン情報] ページへのルートを `@page "/Some/Other/Path"` を使用して `/Some/Other/Path` に設定することができます。
* ページの既定のルートにセグメントを追加します。 たとえば、"item" セグメントを `@page "item"` を使用してページの既定のルートに追加することができます。
* ページの既定のルートにパラメーターを追加します。 たとえば、`@page "{id}"` を含むページに ID パラメーター `id` を必須とすることができます。

パスの先頭のチルダ (`~`) によって指定されたルートの相対パスがサポートされます。 たとえば、`@page "~/Some/Other/Path"` は `@page "/Some/Other/Path"` と同じです。

ルート テンプレート `@page "{handler?}"` を指定することで、URL のクエリ文字列 `?handler=JoinList` をルート セグメント `/JoinList` に変更することができます。

URL 内のクエリ文字列 `?handler=JoinList` が気に入らない場合は、ルートを変更して URL のパス部分にハンドラー名を挿入することができます。 `@page` ディレクティブの後に二重引用符で囲んだルート テンプレートを追加して、ルートをカスタマイズすることができます。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

上記のコードを使用すると、`OnPostJoinListAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH/JoinList` になります。 `OnPostJoinListUCAsync` に送信される URL パスは `https://localhost:5001/Customers/CreateFATH/JoinListUC` です。

`handler` の後の `?` は、ルート パラメーターが省略可能なことを意味します。

## <a name="configuration-and-settings"></a>構成と設定

高度なオプションを構成するには、MVC ビルダーで拡張メソッド `AddRazorPagesOptions` を使用します。

[!code-cs[](index/sample/RazorPagesContacts/StartupAdvanced.cs?name=snippet_1)]

現在は、`RazorPagesOptions` を使用してページのルート ディレクトリを設定したり、ページのアプリケーション モデルの規則を追加したりすることができます。 将来、この方法でより多くの機能拡張を可能にしたいと考えています。

ビューをプリコンパイルするには、「[Razor view compilation](xref:mvc/views/view-compilation)」 (Razor ビュー コンパイル) を参照してください。

[サンプル コードをダウンロードまたは表示します](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/index/sample)。

この概要に基づく、「[Razor ページの概要](xref:tutorials/razor-pages/razor-pages-start)」を参照してください。

### <a name="specify-that-razor-pages-are-at-the-content-root"></a>Razor ページをコンテンツのルートに指定する

Razor ページのルートは既定で */Pages* ディレクトリです。 [WithRazorPagesAtContentRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvcbuilderextensions.withrazorpagesatcontentroot) を [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) に追加して、Razor Pages をアプリの[コンテンツ ルート](xref:fundamentals/index#content-root) ([ContentRootPath](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.contentrootpath)) に置くように指定します。

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesAtContentRoot();
```

### <a name="specify-that-razor-pages-are-at-a-custom-root-directory"></a>Razor ページをカスタム ルート ディレクトリに指定する

(相対パスを指定して) Razor ページをアプリのカスタム ルート ディレクトリに指定するには、[WithRazorPagesRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvccorebuilderextensions.withrazorpagesroot) を [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) に追加します。

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesRoot("/path/to/razor/pages");
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:index>
* <xref:mvc/views/razor>
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>

::: moniker-end
