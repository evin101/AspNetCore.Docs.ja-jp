---
title: ASP.NET Core Blazor の依存関係の挿入
author: guardrex
description: Blazor アプリがコンポーネントにサービスを挿入する方法を確認します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/dependency-injection
ms.openlocfilehash: b548f0e50e1a60b74969e5bbee43860be9ba5a7f
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391137"
---
# <a name="aspnet-core-blazor-dependency-injection"></a>ASP.NET Core Blazor の依存関係の挿入

作成者: [Rainer Stropek](https://www.timecockpit.com)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor では、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)がサポートされています。 アプリは、組み込みのサービスをコンポーネントに挿入することによって使用できます。 アプリでは、カスタムサービスを定義して登録し、DI を使用してアプリ全体で使用できるようにすることもできます。

DI は、中央の場所で構成されたサービスにアクセスするための手法です。 これは、Blazor アプリで次のような場合に役立ちます。

* 1つのサービスクラスのインスタンスを、*シングルトン*サービスと呼ばれる多数のコンポーネントで共有します。
* 参照の抽象化を使用して、具象サービスクラスからコンポーネントを分離します。 たとえば、アプリ内のデータにアクセスするためのインターフェイス `IDataAccess` を考えてみます。 インターフェイスは、具象 @no__t 0 クラスによって実装され、アプリのサービスコンテナーにサービスとして登録されます。 コンポーネントが DI を使用して @no__t 0 の実装を受け取ると、コンポーネントは具象型に結合されません。 の実装はスワップできます。たとえば、単体テストのモック実装では、

## <a name="default-services"></a>既定のサービス

既定のサービスは、アプリのサービスコレクションに自動的に追加されます。

| [サービス] | 有効期間 | 説明 |
| ------- | -------- | ----------- |
| <xref:System.Net.Http.HttpClient> | シングルトン | URI によって識別されるリソースから HTTP 要求を送信し、HTTP 応答を受信するためのメソッドを提供します。 この @no__t のインスタンスは、ブラウザーを使用してバックグラウンドで HTTP トラフィックを処理することに注意してください。 [BaseAddress](xref:System.Net.Http.HttpClient.BaseAddress)は、アプリのベース URI プレフィックスに自動的に設定されます。 詳細については、「<xref:blazor/call-web-api>」を参照してください。 |
| `IJSRuntime` | シングルトン | JavaScript 呼び出しがディスパッチされる JavaScript ランタイムのインスタンスを表します。 詳細については、「<xref:blazor/javascript-interop>」を参照してください。 |
| `NavigationManager` | シングルトン | Uri とナビゲーション状態を操作するためのヘルパーが含まれています。 詳細については、「 [URI およびナビゲーション状態ヘルパー](xref:blazor/routing#uri-and-navigation-state-helpers)」を参照してください。 |

カスタムサービスプロバイダーは、表に示されている既定のサービスを自動的に提供しません。 カスタムサービスプロバイダーを使用し、表に示されているいずれかのサービスが必要な場合は、必要なサービスを新しいサービスプロバイダーに追加します。

## <a name="add-services-to-an-app"></a>アプリにサービスを追加する

新しいアプリを作成した後、@no__t 0 の方法を調べます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add custom services here
}
```

@No__t-0 メソッドには、サービス記述子オブジェクト (<xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor>) のリストである <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection> が渡されます。 サービスは、サービスコレクションにサービス記述子を提供することによって追加されます。 次の例は、`IDataAccess` インターフェイスとその具象実装 `DataAccess` の概念を示しています。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IDataAccess, DataAccess>();
}
```

サービスは、次の表に示す有効期間で構成できます。

| 有効期間 | 説明 |
| -------- | ----------- |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Scoped*> | Blazor WebAssembly には、現在、DI スコープという概念はありません。 @no__t 横-0 サービスは、`Singleton` サービスのように動作します。 ただし、Blazor サーバーホスティングモデルでは、@no__t 0 の有効期間がサポートされています。 Blazor Server apps では、スコープ付きサービス登録は*接続*に対してスコープが設定されています。 このため、現在の目的がブラウザーでクライアント側を実行する場合でも、スコープ付きサービスを使用することは、現在のユーザーにスコープを設定する必要があるサービスに対して推奨されます。 |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Singleton*> | DI は、サービスの*1 つのインスタンス*を作成します。 @No__t 0 サービスを必要とするすべてのコンポーネントは、同じサービスのインスタンスを受け取ります。 |
| <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor.Transient*> | コンポーネントは、サービスコンテナーから `Transient` サービスのインスタンスを取得するたびに、サービスの*新しいインスタンス*を受け取ります。 |

DI システムは ASP.NET Core の DI システムに基づいています。 詳細については、「<xref:fundamentals/dependency-injection>」を参照してください。

## <a name="request-a-service-in-a-component"></a>コンポーネントでサービスを要求する

サービスがサービスコレクションに追加された後、 [\@inject](xref:mvc/views/razor#inject) Razor ディレクティブを使用して、サービスをコンポーネントに挿入します。 `@inject` には、次の2つのパラメーターがあります。

* 「@No__t-0」と入力します。
* プロパティ &ndash; 挿入された app service を受け取るプロパティの名前。 プロパティは手動で作成する必要はありません。 コンパイラによってプロパティが作成されます。

詳細については、「<xref:mvc/views/dependency-injection>」を参照してください。

複数の `@inject` ステートメントを使用して、さまざまなサービスを挿入します。

次の例は、`@inject` を使用する方法を示しています。 @No__t-0 を実装するサービスは、コンポーネントのプロパティ `DataRepository` に挿入されます。 コードが @no__t 0 の抽象化を使用するかどうかに注意してください。

[!code-cshtml[](dependency-injection/samples_snapshot/3.x/CustomerList.razor?highlight=2-3,23)]

内部的には、生成されたプロパティ (`DataRepository`) は、`InjectAttribute` 属性で修飾されます。 通常、この属性は直接使用されません。 コンポーネントに基底クラスが必要であり、その基底クラスにも挿入されたプロパティが必要な場合は、`InjectAttribute` を手動で追加します。

```csharp
public class ComponentBase : IComponent
{
    // DI works even if using the InjectAttribute in a component's base class.
    [Inject]
    protected IDataAccess DataRepository { get; set; }
    ...
}
```

基底クラスから派生したコンポーネントでは、`@inject` ディレクティブは必要ありません。 基底クラスの `InjectAttribute` で十分です。

```cshtml
@page "/demo"
@inherits ComponentBase

<h1>Demo Component</h1>
```

## <a name="use-di-in-services"></a>サービスで DI を使用する

複雑なサービスでは、追加のサービスが必要になる場合があります。 前の例では、`DataAccess` で @no__t 既定のサービスが必要になる場合があります。 `@inject` (または `InjectAttribute`) は、サービスで使用できません。 代わりに*コンストラクターの挿入*を使用する必要があります。 必要なサービスは、サービスのコンストラクターにパラメーターを追加することによって追加されます。 DI は、サービスを作成するときに、必要なサービスをコンストラクターで認識し、それに応じてそれを提供します。

```csharp
public class DataAccess : IDataAccess
{
    // The constructor receives an HttpClient via dependency
    // injection. HttpClient is a default service.
    public DataAccess(HttpClient client)
    {
        ...
    }
}
```

コンストラクターインジェクションの前提条件:

* すべての引数が DI によって満たされることができるコンストラクターが1つ存在する必要があります。 DI でカバーされない追加のパラメーターは、既定値を指定した場合に許可されます。
* 該当するコンストラクターは*パブリック*である必要があります。
* 1つの適用可能なコンストラクターが存在する必要があります。 あいまいさが発生した場合、DI は例外をスローします。

## <a name="utility-base-component-classes-to-manage-a-di-scope"></a>DI スコープを管理するためのユーティリティの基本コンポーネントクラス

ASP.NET Core アプリでは、スコープ付きサービスは通常、現在の要求にスコープが設定されます。 要求が完了すると、スコープまたは一時的なサービスが DI システムによって破棄されます。 Blazor Server apps では、要求スコープはクライアント接続の間継続されるため、一時的でスコープのあるサービスは予想よりもかなり長くなる可能性があります。

サービスのスコープをコンポーネントの有効期間に限定するために、は `OwningComponentBase` および `OwningComponentBase<TService>` 基底クラスを使用できます。 これらの基本クラスは、コンポーネントの有効期間にスコープが設定されているサービスを解決する @no__t 型の @no__t 0 のプロパティを公開します。 Razor の基底クラスから継承するコンポーネントを作成するには、`@inherits` ディレクティブを使用します。

```cshtml
@page "/users"
@attribute [Authorize]
@inherits OwningComponentBase<Data.ApplicationDbContext>

<h1>Users (@Service.Users.Count())</h1>
<ul>
    @foreach (var user in Service.Users)
    {
        <li>@user.UserName</li>
    }
</ul>
```

> [!NOTE]
> @No__t-0 または `InjectAttribute` を使用してコンポーネントに挿入されたサービスは、コンポーネントのスコープ内に作成されず、要求スコープに関連付けられます。

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/dependency-injection>
* <xref:mvc/views/dependency-injection>
