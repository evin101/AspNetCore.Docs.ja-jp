---
title: ASP.NET Core での応答キャッシュミドルウェア
author: guardrex
description: ASP.NET Core で応答キャッシュ ミドルウェアを構成し、使用する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 11/04/2019
uid: performance/caching/middleware
ms.openlocfilehash: a8e656e1d59114e2e953323e98e0a2399efca98a
ms.sourcegitcommit: 09f4a5ded39cc8204576fe801d760bd8b611f3aa
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/05/2019
ms.locfileid: "73611457"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>ASP.NET Core での応答キャッシュミドルウェア

[Luke Latham](https://github.com/guardrex)および[John Luo](https://github.com/JunTaoLuo)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/middleware/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

この記事では、ASP.NET Core アプリで応答キャッシュミドルウェアを構成する方法について説明します。 ミドルウェアは、応答をキャッシュ可能にするタイミングを決定し、応答を格納して、キャッシュからの応答を提供します。 HTTP キャッシュと[[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute)属性の概要については、「[応答キャッシュ](xref:performance/caching/response)」を参照してください。

## <a name="configuration"></a>構成

::: moniker range=">= aspnetcore-3.0"

応答キャッシュミドルウェアは、共有フレームワークを介して ASP.NET Core アプリで暗黙的に使用できます。

`Startup.ConfigureServices`で、応答キャッシュミドルウェアをサービスコレクションに追加します。

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

ミドルウェアを <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> 拡張メソッドと共に使用するようにアプリを構成します。これにより、`Startup.Configure`の要求処理パイプラインにミドルウェアが追加されます。

[!code-csharp[](middleware/samples/3.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=16)]

サンプルアプリでは、後続の要求でキャッシュを制御するためのヘッダーを追加します。

* [キャッシュ制御](https://tools.ietf.org/html/rfc7234#section-5.2)&ndash; は、最大10秒間のキャッシュ可能な応答をキャッシュします。
* &ndash;[変更](https://tools.ietf.org/html/rfc7231#section-7.1.4)すると、後続の要求の[`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4)ヘッダーが元の要求のヘッダーと一致する場合にのみ、キャッシュされた応答を提供するようにミドルウェアが構成されます。

[!code-csharp[](middleware/samples_snippets/3.x/AddHeaders.cs)]

応答キャッシュミドルウェアは、200 (OK) 状態コードを生成するサーバー応答のみをキャッシュします。 [エラーページ](xref:fundamentals/error-handling)を含むその他の応答は、ミドルウェアによって無視されます。

> [!WARNING]
> 認証されたクライアントのコンテンツを含む応答は、ミドルウェアがそれらの応答を格納してサービスを提供しないように、キャッシュ不可能としてマークする必要があります。 応答がキャッシュ可能かどうかをミドルウェアが決定する方法の詳細については、「[キャッシュの条件](#conditions-for-caching)」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)を使用するか、 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)パッケージへのパッケージ参照を追加します。

`Startup.ConfigureServices`で、応答キャッシュミドルウェアをサービスコレクションに追加します。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=3)]

ミドルウェアを <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching*> 拡張メソッドと共に使用するようにアプリを構成します。これにより、`Startup.Configure`の要求処理パイプラインにミドルウェアが追加されます。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=14)]

サンプルアプリでは、後続の要求でキャッシュを制御するためのヘッダーを追加します。

* [キャッシュ制御](https://tools.ietf.org/html/rfc7234#section-5.2)&ndash; は、最大10秒間のキャッシュ可能な応答をキャッシュします。
* &ndash;[変更](https://tools.ietf.org/html/rfc7231#section-7.1.4)すると、後続の要求の[`Accept-Encoding`](https://tools.ietf.org/html/rfc7231#section-5.3.4)ヘッダーが元の要求のヘッダーと一致する場合にのみ、キャッシュされた応答を提供するようにミドルウェアが構成されます。

[!code-csharp[](middleware/samples_snippets/2.x/AddHeaders.cs)]

応答キャッシュミドルウェアは、200 (OK) 状態コードを生成するサーバー応答のみをキャッシュします。 [エラーページ](xref:fundamentals/error-handling)を含むその他の応答は、ミドルウェアによって無視されます。

> [!WARNING]
> 認証されたクライアントのコンテンツを含む応答は、ミドルウェアがそれらの応答を格納してサービスを提供しないように、キャッシュ不可能としてマークする必要があります。 応答がキャッシュ可能かどうかをミドルウェアが決定する方法の詳細については、「[キャッシュの条件](#conditions-for-caching)」を参照してください。

::: moniker-end

## <a name="options"></a>オプション

応答キャッシュのオプションを次の表に示します。

| オプション | 説明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize> | 応答本文の最大キャッシュ可能サイズ (バイト単位)。 既定値は `64 * 1024 * 1024` (64 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit> | 応答キャッシュミドルウェアのサイズ制限 (バイト単位)。 既定値は `100 * 1024 * 1024` (100 MB) です。 |
| <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.UseCaseSensitivePaths> | 大文字と小文字が区別されるパスに応答をキャッシュするかどうかを決定します。 既定値は `false`です。 |

次の例では、ミドルウェアをに構成します。

* 本文のサイズが1024バイト以下の応答をキャッシュします。
* 大文字と小文字を区別するパスで応答を格納します。 たとえば、`/page1` と `/Page1` は個別に格納されます。

```csharp
services.AddResponseCaching(options =>
{
    options.MaximumBodySize = 1024;
    options.UseCaseSensitivePaths = true;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

MVC/web API コントローラーまたは Razor Pages ページモデルを使用する場合、 [[ResponseCache]](xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute)属性では、応答キャッシュ用に適切なヘッダーを設定するために必要なパラメーターを指定します。 ミドルウェアを厳密に必要とする `[ResponseCache]` 属性の唯一のパラメーターは <xref:Microsoft.AspNetCore.Mvc.ResponseCacheAttribute.VaryByQueryKeys>であり、実際の HTTP ヘッダーには対応していません。 詳細については、「<xref:performance/caching/response#responsecache-attribute>」を参照してください。

`[ResponseCache]` 属性を使用しない場合、応答のキャッシュは `VaryByQueryKeys`でさまざまに変化する可能性があります。 <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingFeature> を HttpContext から直接使用[します。](xref:Microsoft.AspNetCore.Http.HttpContext.Features)

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();

if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

`VaryByQueryKeys` の `*` と同じ1つの値を使用すると、すべての要求クエリパラメーターによってキャッシュが異なります。

## <a name="http-headers-used-by-response-caching-middleware"></a>応答キャッシュミドルウェアによって使用される HTTP ヘッダー

次の表は、応答のキャッシュに影響を与える HTTP ヘッダーに関する情報を示しています。

| Header | 説明 |
| ------ | ------- |
| `Authorization` | ヘッダーが存在する場合、応答はキャッシュされません。 |
| `Cache-Control` | ミドルウェアは、`public` cache ディレクティブでマークされたキャッシュ応答のみを考慮します。 次のパラメーターを使用してキャッシュを制御します。<ul><li>最長有効期間</li><li>最大-古い&#8224;</li><li>最小-新規</li><li>再検証が必要</li><li>キャッシュなし</li><li>ストアなし</li><li>-if-キャッシュ済み</li><li>private</li><li>public</li><li>s-maxage</li><li>プロキシ-再検証&#8225;</li></ul>&#8224;`max-stale`に制限が指定されていない場合、ミドルウェアは何も実行しません。<br>&#8225;`proxy-revalidate` は `must-revalidate`と同じ効果があります。<br><br>詳細については、「 [RFC 7231: Request Cache-control ディレクティブ](https://tools.ietf.org/html/rfc7234#section-5.2.1)」を参照してください。 |
| `Pragma` | 要求内の `Pragma: no-cache` ヘッダーでは、`Cache-Control: no-cache`と同じ効果が得られます。 このヘッダーは、`Cache-Control` ヘッダー内の関連するディレクティブによってオーバーライドされます (存在する場合)。 HTTP/1.0 との下位互換性のために考慮されます。 |
| `Set-Cookie` | ヘッダーが存在する場合、応答はキャッシュされません。 1つ以上の cookie を設定する要求処理パイプライン内のミドルウェアは、応答キャッシュミドルウェアが応答をキャッシュしないようにします ( [cookie ベースの TempData プロバイダー](xref:fundamentals/app-state#tempdata)など)。  |
| `Vary` | キャッシュされた応答を別のヘッダーによって変更するには、`Vary` ヘッダーを使用します。 たとえば、`Vary: Accept-Encoding` ヘッダーを含めることにより、エンコードによって応答をキャッシュします。これにより、ヘッダー `Accept-Encoding: gzip` と `Accept-Encoding: text/plain` 個別に要求に対する応答がキャッシュされます。 ヘッダー値が `*` の応答は格納されません。 |
| `Expires` | このヘッダーによって古いと見なされる応答は、他の `Cache-Control` ヘッダーでオーバーライドされない限り、格納または取得されません。 |
| `If-None-Match` | 値が `*` ず、応答の `ETag` が指定された値と一致しない場合は、キャッシュから完全な応答が提供されます。 それ以外の場合は、304 (変更なし) の応答が処理されます。 |
| `If-Modified-Since` | `If-None-Match` ヘッダーが存在しない場合、キャッシュされた応答の日付が指定した値より新しい場合は、キャッシュから完全な応答が提供されます。 それ以外の場合は、 *304-変更されていない*応答が提供されます。 |
| `Date` | キャッシュからサービスを提供している場合、`Date` ヘッダーは、元の応答で提供されていない場合、ミドルウェアによって設定されます。 |
| `Content-Length` | キャッシュからサービスを提供している場合、`Content-Length` ヘッダーは、元の応答で提供されていない場合、ミドルウェアによって設定されます。 |
| `Age` | 元の応答で送信された `Age` ヘッダーは無視されます。 ミドルウェアは、キャッシュされた応答を提供するときに新しい値を計算します。 |

## <a name="caching-respects-request-cache-control-directives"></a>キャッシュは、要求のキャッシュ制御ディレクティブに従います。

ミドルウェアは、 [HTTP 1.1 キャッシュ仕様](https://tools.ietf.org/html/rfc7234#section-5.2)の規則を尊重します。 規則では、クライアントによって送信された有効な `Cache-Control` ヘッダーを受け入れるためにキャッシュが必要です。 この仕様では、クライアントは `no-cache` ヘッダー値を使用して要求を行い、すべての要求に対してサーバーに新しい応答を強制的に生成させることができます。 現時点では、ミドルウェアが公式のキャッシュ仕様に準拠しているため、ミドルウェアを使用する場合、このキャッシュ動作を開発者が制御することはありません。

キャッシュの動作を詳細に制御するには、ASP.NET Core の他のキャッシュ機能を調べます。 次のトピックを参照してください。

* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

## <a name="troubleshooting"></a>トラブルシューティング

キャッシュの動作が想定どおりでない場合は、応答がキャッシュ可能であり、キャッシュから提供できることを確認します。 要求の受信ヘッダーと応答の送信ヘッダーを確認します。 デバッグに役立つように[ログ記録](xref:fundamentals/logging/index)を有効にします。

キャッシュ動作のテストとトラブルシューティングを行う場合、ブラウザーでは、望ましくない方法でキャッシュに影響を与える要求ヘッダーを設定できます。 たとえば、ブラウザーでは、ページの更新時に `Cache-Control` ヘッダーを `no-cache` または `max-age=0` に設定できます。 次のツールでは、要求ヘッダーを明示的に設定でき、キャッシュのテストに適しています。

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>キャッシュの条件

* 要求は、200 (OK) 状態コードでサーバー応答を生成する必要があります。
* 要求メソッドは GET または HEAD である必要があります。
* `Startup.Configure`では、応答キャッシュミドルウェアは、キャッシュを必要とするミドルウェアの前に配置する必要があります。 詳細については、「<xref:fundamentals/middleware/index>」を参照してください。
* `Authorization` ヘッダーを指定することはできません。
* `Cache-Control` ヘッダーパラメーターは有効である必要があり、応答は `public` としてマークされ、`private`としてマークされていない必要があります。
* `Cache-Control` ヘッダーが存在しない場合、`Pragma: no-cache` ヘッダーは存在しない必要があります。 `Cache-Control` ヘッダーが存在する場合、`Pragma` ヘッダーがオーバーライドされるためです。
* `Set-Cookie` ヘッダーを指定することはできません。
* `Vary` ヘッダーパラメーターは、`*`と同じである必要があります。
* `Content-Length` ヘッダー値 (設定されている場合) は、応答本文のサイズと一致する必要があります。
* <xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature> は使用されません。
* 応答は、`Expires` ヘッダーおよび `max-age` および `s-maxage` のキャッシュディレクティブで指定されているとおりに古いものである必要があります。
* 応答バッファリングを成功させる必要があります。 応答のサイズは、構成済みまたは既定の <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.SizeLimit>よりも小さくする必要があります。 応答の本文のサイズは、構成済みまたは既定の <xref:Microsoft.AspNetCore.ResponseCaching.ResponseCachingOptions.MaximumBodySize>よりも小さくする必要があります。
* 応答は、 [RFC 7234](https://tools.ietf.org/html/rfc7234)仕様に従ってキャッシュ可能である必要があります。 たとえば、`no-store` ディレクティブは、要求または応答のヘッダーフィールドに存在することはできません。 詳細については、 *「セクション 3:* [RFC 7234](https://tools.ietf.org/html/rfc7234)のキャッシュに応答を格納する」を参照してください。

> [!NOTE]
> クロスサイト要求偽造 (CSRF) 攻撃を防ぐためにセキュリティで保護されたトークンを生成するための偽造防止システムは、応答がキャッシュされないように、`Cache-Control` と `Pragma` ヘッダーを `no-cache` に設定します。 HTML フォーム要素の偽造防止トークンを無効にする方法については、「<xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration>」を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* <xref:performance/caching/memory>
* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
