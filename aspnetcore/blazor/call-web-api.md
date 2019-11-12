---
title: ASP.NET Core Blazor から web API を呼び出す
author: guardrex
description: クロスオリジンリソース共有 (CORS) 要求の作成など、JSON ヘルパーを使用して Blazor アプリから web API を呼び出す方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/call-web-api
ms.openlocfilehash: b08fdf5c2f9a523314b1744a33087eb64fa4c14a
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2019
ms.locfileid: "72390830"
---
# <a name="call-a-web-api-from-aspnet-core-blazor"></a>ASP.NET Core Blazor から web API を呼び出す

作成者: [Luke Latham](https://github.com/guardrex)、 [Daniel Roth](https://github.com/danroth27)、[Juan De la Cruz](https://github.com/juandelacruz23)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Blazor WebAssembly は、構成済みの @no__t 0 サービスを使用して web Api を呼び出します。 Blazor JSON ヘルパーまたは <xref:System.Net.Http.HttpRequestMessage> を使用して、JavaScript [FETCH API](https://developer.mozilla.org/docs/Web/API/Fetch_API)オプションを含めることができるように要求を作成します。

Blazor サーバーアプリは、通常 <xref:System.Net.Http.IHttpClientFactory> を使用して作成された @no__t 0 のインスタンスを使用して web Api を呼び出します。 詳細については、「<xref:fundamentals/http-requests>」を参照してください。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

Blazor WebAssembly 例については、サンプルアプリの次のコンポーネントを参照してください。

* Web API の呼び出し (*Pages/CallWebAPI*)
* HTTP 要求テスター (*Components/HTTPRequestTester*)

## <a name="httpclient-and-json-helpers"></a>HttpClient と JSON のヘルパー

Blazor WebAssembly では、 [Httpclient](xref:fundamentals/http-requests)は、要求を配信元サーバーに返すための事前に構成されたサービスとして利用できます。 @No__t-0 の JSON ヘルパーを使用するには、`Microsoft.AspNetCore.Blazor.HttpClient` にパッケージ参照を追加します。 `HttpClient` および JSON ヘルパーは、サードパーティの web API エンドポイントを呼び出すためにも使用されます。 `HttpClient` は、ブラウザーの[FETCH API](https://developer.mozilla.org/docs/Web/API/Fetch_API)を使用して実装され、同じオリジンポリシーの適用などの制限が適用されます。

クライアントのベースアドレスは、元のサーバーのアドレスに設定されます。 @No__t-1 ディレクティブを使用して @no__t 0 のインスタンスを挿入します。

```cshtml
@using System.Net.Http
@inject HttpClient Http
```

次の例では、Todo web API が作成、読み取り、更新、および削除 (CRUD) の各操作を処理します。 これらの例は、を格納する @no__t 0 のクラスに基づいています。

* ID (`Id`、`long`) &ndash; アイテムの一意の ID。
* 名前 (`Name`、`string`) &ndash; 項目の名前。
* 状態 (`IsComplete`、`bool`) &ndash; Todo 項目が終了したかどうかを示します。

```csharp
private class TodoItem
{
    public long Id { get; set; }
    public string Name { get; set; }
    public bool IsComplete { get; set; }
}
```

JSON ヘルパーメソッドは、要求を URI (次の例では web API) に送信し、応答を処理します。

* `GetJsonAsync` &ndash; は HTTP GET 要求を送信し、JSON 応答本文を解析してオブジェクトを作成します。

  次のコードでは、`_todoItems` がコンポーネントによって表示されます。 @No__t-0 メソッドは、コンポーネントのレンダリングが終了したときにトリガーされます ([On初期化 Edasync](xref:blazor/components#lifecycle-methods))。 完全な例については、サンプルアプリを参照してください。

  ```cshtml
  @using System.Net.Http
  @inject HttpClient Http

  @code {
      private TodoItem[] _todoItems;

      protected override async Task OnInitializedAsync() => 
          _todoItems = await Http.GetJsonAsync<TodoItem[]>("api/TodoItems");
  }
  ```

* `PostJsonAsync` &ndash; は、JSON でエンコードされたコンテンツを含む HTTP POST 要求を送信し、JSON 応答本文を解析してオブジェクトを作成します。

  次のコードでは、コンポーネントのバインドされた要素によって `_newItemName` が提供されています。 @No__t-0 メソッドは、`<button>` 要素を選択することによってトリガーされます。 完全な例については、サンプルアプリを参照してください。

  ```cshtml
  @using System.Net.Http
  @inject HttpClient Http

  <input @bind="_newItemName" placeholder="New Todo Item" />
  <button @onclick="@AddItem">Add</button>

  @code {
      private string _newItemName;

      private async Task AddItem()
      {
          var addItem = new TodoItem { Name = _newItemName, IsComplete = false };
          await Http.PostJsonAsync("api/TodoItems", addItem);
      }
  }
  ```

* `PutJsonAsync` &ndash; は、JSON でエンコードされたコンテンツを含む HTTP PUT 要求を送信します。

  次のコードでは、`Name` と `IsCompleted` の `_editItem` の値が、コンポーネントのバインドされた要素によって提供されています。 項目の `Id` は、UI の別の部分で項目が選択され、`EditItem` が呼び出されたときに設定されます。 @No__t-0 メソッドは、Save `<button>` 要素を選択することによってトリガーされます。 完全な例については、サンプルアプリを参照してください。

  ```cshtml
  @using System.Net.Http
  @inject HttpClient Http

  <input type="checkbox" @bind="_editItem.IsComplete" />
  <input @bind="_editItem.Name" />
  <button @onclick="@SaveItem">Save</button>

  @code {
      private TodoItem _editItem = new TodoItem();

      private void EditItem(long id)
      {
          var editItem = _todoItems.Single(i => i.Id == id);
          _editItem = new TodoItem { Id = editItem.Id, Name = editItem.Name, 
              IsComplete = editItem.IsComplete };
      }

      private async Task SaveItem() =>
          await Http.PutJsonAsync($"api/TodoItems/{_editItem.Id}, _editItem);
  }
  ```

<xref:System.Net.Http> には、HTTP 要求を送信して HTTP 応答を受信するための拡張メソッドが追加されています。 [Httpclient。 DeleteAsync](xref:System.Net.Http.HttpClient.DeleteAsync*)は、HTTP DELETE 要求を web API に送信するために使用されます。

次のコードでは、Delete `<button>` 要素は `DeleteItem` メソッドを呼び出します。 バインドされた `<input>` 要素は、削除する項目の `id` を提供します。 完全な例については、サンプルアプリを参照してください。

```cshtml
@using System.Net.Http
@inject HttpClient Http

<input @bind="_id" />
<button @onclick="@DeleteItem">Delete</button>

@code {
    private long _id;

    private async Task DeleteItem() =>
        await Http.DeleteAsync($"api/TodoItems/{_id}");
}
```

## <a name="cross-origin-resource-sharing-cors"></a>クロスオリジンリソース共有 (CORS)

ブラウザーのセキュリティでは、web ページが web ページを提供したものとは異なるドメインに要求を作成できないようにします。 この制限は、*同じオリジンポリシー*と呼ばれます。 同じ配信元ポリシーでは、悪意のあるサイトが別のサイトから機密データを読み取れないようにします。 ブラウザーから別のオリジンのエンドポイントへの要求を行うには、*エンドポイント*が[クロスオリジンリソース共有 (CORS)](https://www.w3.org/TR/cors/)を有効にする必要があります。

このサンプルアプリでは、呼び出し Web API コンポーネント (*Pages/CallWebAPI*) で CORS を使用する方法を示しています。

他のサイトがアプリに対してクロスオリジンリソース共有 (CORS) 要求を行うことができるようにするには、「<xref:security/cors>」を参照してください。

## <a name="httpclient-and-httprequestmessage-with-fetch-api-request-options"></a>HttpClient と HttpRequestMessage with Fetch API 要求オプション

Blazor webassembly で WebAssembly 実行する場合は、 [Httpclient](xref:fundamentals/http-requests)と <xref:System.Net.Http.HttpRequestMessage> を使用して要求をカスタマイズします。 たとえば、要求 URI、HTTP メソッド、および必要な要求ヘッダーを指定できます。

要求の `WebAssemblyHttpMessageHandler.FetchArgs` プロパティを使用して、基になる JavaScript [FETCH API](https://developer.mozilla.org/docs/Web/API/Fetch_API)に要求オプションを指定します。 次の例に示すように、`credentials` プロパティは、次のいずれかの値に設定されます。

* `FetchCredentialsOption.Include` ("include") &ndash; は、クロスオリジン要求の場合でも、ブラウザーが資格情報 (cookie や HTTP 認証ヘッダーなど) を送信するように通知します。 CORS ポリシーが資格情報を許可するように構成されている場合にのみ許可されます。
* `FetchCredentialsOption.Omit` ("省略") @no__t、ブラウザーが資格情報を送信しないことを通知します (cookie や HTTP 認証ヘッダーなど)。
* `FetchCredentialsOption.SameOrigin` ("同じオリジン") &ndash; は、ターゲット URL が呼び出し元アプリケーションと同じオリジンにある場合にのみ、ブラウザーが資格情報 (cookie や HTTP auth ヘッダーなど) を送信するように通知します。

```cshtml
@using System.Net.Http
@using System.Net.Http.Headers
@inject HttpClient Http

@code {
    private async Task PostRequest()
    {
        Http.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Bearer", "{OAUTH TOKEN}");

        var requestMessage = new HttpRequestMessage()
        {
            Method = new HttpMethod("POST"),
            RequestUri = new Uri("https://localhost:10000/api/TodoItems"),
            Content = 
                new StringContent(
                    @"{""name"":""A New Todo Item"",""isComplete"":false}")
        };

        requestMessage.Content.Headers.ContentType = 
            new System.Net.Http.Headers.MediaTypeHeaderValue(
                "application/json");

        requestMessage.Content.Headers.TryAddWithoutValidation(
            "x-custom-header", "value");
        
        requestMessage.Properties[WebAssemblyHttpMessageHandler.FetchArgs] = new
        { 
            credentials = FetchCredentialsOption.Include
        };

        var response = await Http.SendAsync(requestMessage);
        var responseStatusCode = response.StatusCode;
        var responseBody = await response.Content.ReadAsStringAsync();
    }
}
```

Fetch API オプションの詳細については、 [MDN の web ドキュメント: WindowOrWorkerGlobalScope ():P arameters](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/fetch#Parameters)を参照してください。

CORS 要求で資格情報 (承認 cookie/ヘッダー) を送信する場合、CORS ポリシーによって `Authorization` ヘッダーが許可されている必要があります。

次のポリシーには、の構成が含まれています。

* 要求オリジン (`http://localhost:5000`、`https://localhost:5001`)。
* 任意のメソッド (動詞)。
* `Content-Type` および `Authorization` ヘッダーです。 カスタムヘッダー (たとえば `x-custom-header`) を許可するには <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> を呼び出すときにヘッダーを一覧表示します。
* クライアント側の JavaScript コードによって設定された資格情報 (`credentials` プロパティが `include` に設定されている)。

```csharp
app.UseCors(policy => 
    policy.WithOrigins("http://localhost:5000", "https://localhost:5001")
    .AllowAnyMethod()
    .WithHeaders(HeaderNames.ContentType, HeaderNames.Authorization, "x-custom-header")
    .AllowCredentials());
```

詳細については、「<xref:security/cors>」とサンプルアプリの HTTP 要求テスターコンポーネント (*Components/HTTPRequestTester*) を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/http-requests>
* <xref:security/enforcing-ssl>
* [Kestrel HTTPS エンドポイントの構成](xref:fundamentals/servers/kestrel#endpoint-configuration)
* [W3C でのクロスオリジンリソース共有 (CORS)](https://www.w3.org/TR/cors/)
