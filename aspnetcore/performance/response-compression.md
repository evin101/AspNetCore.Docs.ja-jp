---
title: ASP.NET Core での応答圧縮
author: guardrex
description: 応答圧縮と ASP.NET Core アプリで応答圧縮ミドルウェアを使用する方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/09/2019
uid: performance/response-compression
ms.openlocfilehash: e320e87179f9f1b9773a55c380684a3f3f712632
ms.sourcegitcommit: 89fcc6cb3e12790dca2b8b62f86609bed6335be9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/13/2019
ms.locfileid: "68993473"
---
# <a name="response-compression-in-aspnet-core"></a>ASP.NET Core での応答圧縮

作成者: [Luke Latham](https://github.com/guardrex)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

ネットワーク帯域幅はリソースが限られています。 通常、応答のサイズを小さくすると、アプリの応答性が大幅に向上します。 ペイロードサイズを減らす方法の1つは、アプリの応答を圧縮することです。

## <a name="when-to-use-response-compression-middleware"></a>応答圧縮ミドルウェアを使用する場合

IIS、Apache、または Nginx のサーバーベースの応答圧縮テクノロジを使用します。 ミドルウェアのパフォーマンスがサーバーモジュールのパフォーマンスと一致しない場合があります。 現在、 [http.sys サーバー](xref:fundamentals/servers/httpsys)サーバーと[Kestrel](xref:fundamentals/servers/kestrel)サーバーでは、組み込みの圧縮サポートは提供されていません。

次の場合に応答圧縮ミドルウェアを使用します。

* 次のサーバーベースの圧縮テクノロジは使用できません。
  * [IIS 動的圧縮モジュール](https://www.iis.net/overview/reliability/dynamiccachingandcompression)
  * [Apache mod_deflate モジュール](https://httpd.apache.org/docs/current/mod/mod_deflate.html)
  * [Nginx 圧縮と圧縮解除](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)
* 直接ホストする:
  * Http.sys[サーバー](xref:fundamentals/servers/httpsys)(旧称 WebListener)
  * [Kestrel サーバー](xref:fundamentals/servers/kestrel)

## <a name="response-compression"></a>応答圧縮

通常、ネイティブに圧縮されていない応答は、応答圧縮の恩恵を受けることができます。 ネイティブに圧縮されていない応答には、通常、CSS、JavaScript、HTML、XML、および JSON。 PNG ファイルなど、ネイティブに圧縮されたアセットを圧縮することはできません。 ネイティブ圧縮応答をさらに圧縮しようとすると、サイズが小さく、転送時間が短縮されると、圧縮の処理に要する時間が大きくなります。 ファイルの内容や圧縮の効率によっては、約150-1000 バイトを下回るファイルを圧縮しないようにしてください。 小さいファイルを圧縮すると、圧縮されていないファイルよりも大きな圧縮ファイルが生成される可能性があります。

クライアントは、圧縮されたコンテンツを処理できる場合、要求と共に`Accept-Encoding`ヘッダーを送信することによって、その機能をサーバーに通知する必要があります。 サーバーは、圧縮されたコンテンツを送信するときに`Content-Encoding` 、圧縮された応答のエンコード方法に関する情報をヘッダーに含める必要があります。 次の表に、ミドルウェアでサポートされているコンテンツエンコードの表記を示します。

::: moniker range=">= aspnetcore-2.2"

| `Accept-Encoding`ヘッダー値 | サポートされているミドルウェア | 説明 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | はい (既定値)        | [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | いいえ                   | [圧縮データ形式の DEFLATE](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | いいえ                   | [W3C の効率的な XML インターチェンジ](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | [はい]                  | [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | [はい]                  | "エンコードなし" 識別子:応答をエンコードすることはできません。 |
| `pack200-gzip`                  | いいえ                   | [Java アーカイブのネットワーク転送形式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | [はい]                  | 明示的に要求されていない利用可能なコンテンツエンコーディング |

::: moniker-end

::: moniker range="< aspnetcore-2.2"

| `Accept-Encoding`ヘッダー値 | サポートされているミドルウェア | 説明 |
| ------------------------------- | :------------------: | ----------- |
| `br`                            | いいえ                   | [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932) |
| `deflate`                       | いいえ                   | [圧縮データ形式の DEFLATE](https://tools.ietf.org/html/rfc1951) |
| `exi`                           | いいえ                   | [W3C の効率的な XML インターチェンジ](https://tools.ietf.org/id/draft-varga-netconf-exi-capability-00.html) |
| `gzip`                          | はい (既定値)        | [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952) |
| `identity`                      | [はい]                  | "エンコードなし" 識別子:応答をエンコードすることはできません。 |
| `pack200-gzip`                  | いいえ                   | [Java アーカイブのネットワーク転送形式](https://jcp.org/aboutJava/communityprocess/review/jsr200/index.html) |
| `*`                             | [はい]                  | 明示的に要求されていない利用可能なコンテンツエンコーディング |

::: moniker-end

詳細については、「 [IANA 公式コンテンツコーディングリスト](https://www.iana.org/assignments/http-parameters/http-parameters.xml#http-content-coding-registry)」を参照してください。

ミドルウェアを使用すると、カスタム`Accept-Encoding`ヘッダー値の圧縮プロバイダーを追加できます。 詳細については、以下の「[カスタムプロバイダー](#custom-providers) 」を参照してください。

ミドルウェアは、圧縮スキームの優先順位を決定するために`q`クライアントから送信されるときに、品質の値 (qvalue,) の重み付けに対応できます。 詳細については[、RFC 7231 を参照してください。Accept-Encoding](https://tools.ietf.org/html/rfc7231#section-5.3.4)。

圧縮アルゴリズムは、圧縮速度と圧縮の効果のトレードオフの影響を受けます。 このコンテキストの*効果*は、圧縮後の出力のサイズを示します。 最小サイズは*最適*な圧縮によって実現されます。

次の表では、圧縮されたコンテンツの要求、送信、キャッシュ、および受信に関連するヘッダーについて説明します。

| Header             | ロール |
| ------------------ | ---- |
| `Accept-Encoding`  | クライアントからサーバーに送信され、クライアントが使用できるコンテンツエンコーディングスキームを示します。 |
| `Content-Encoding` | ペイロード内のコンテンツのエンコードを示すために、サーバーからクライアントに送信されます。 |
| `Content-Length`   | 圧縮が発生`Content-Length`すると、応答が圧縮されるときに本文の内容が変更されるため、ヘッダーは削除されます。 |
| `Content-MD5`      | 圧縮が行わ`Content-MD5`れると、本文の内容が変更され、ハッシュが有効でなくなったため、ヘッダーは削除されます。 |
| `Content-Type`     | コンテンツの MIME の種類を指定します。 すべての応答で、 `Content-Type`を指定する必要があります。 ミドルウェアは、この値をチェックして、応答を圧縮する必要があるかどうかを判断します。 ミドルウェアはエンコードできる既定の[mime の種類](#mime-types)のセットを指定しますが、mime の種類を置き換えたり追加したりすることができます。 |
| `Vary`             | の`Accept-Encoding`値がであるサーバーからクライアントとプロキシに送信された`Vary`場合、ヘッダーは、要求の`Accept-Encoding`ヘッダーの値に基づいて応答をキャッシュ (変更) する必要があることをクライアントまたはプロキシに示します。 `Vary: Accept-Encoding`ヘッダーを使用してコンテンツを返すと、圧縮された応答と圧縮されていない応答の両方が個別にキャッシュされることになります。 |

[サンプルアプリ](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/response-compression/samples)を使用して、応答圧縮ミドルウェアの機能を検証します。 このサンプルでは、次のことを示します。

* Gzip およびカスタム圧縮プロバイダーを使用したアプリ応答の圧縮。
* 圧縮する mime の種類の既定の一覧に MIME の種類を追加する方法について説明します。

## <a name="package"></a>パッケージ

::: moniker range=">= aspnetcore-3.0"

応答圧縮ミドルウェアは、AspNetCore アプリ ASP.NET Core に暗黙的に含まれる[Microsoft.AspNetCore.ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)パッケージによって提供されます。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ミドルウェアをプロジェクトに含めるには、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)への参照を追加します。これには、 [Microsoft.AspNetCore.ResponseCompression](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCompression/)パッケージが含まれています。

::: moniker-end

## <a name="configuration"></a>構成

::: moniker range=">= aspnetcore-2.2"

次のコードは、既定の MIME の種類と圧縮プロバイダー ([Brotli](#brotli-compression-provider)と[Gzip](#gzip-compression-provider)) に対して応答圧縮ミドルウェアを有効にする方法を示しています。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

次のコードは、既定の MIME の種類と[Gzip 圧縮プロバイダー](#gzip-compression-provider)の応答圧縮ミドルウェアを有効にする方法を示しています。

::: moniker-end

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddResponseCompression();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseResponseCompression();
    }
}
```

メモ:

* `app.UseResponseCompression`の前に`app.UseMvc`を呼び出す必要があります。
* [Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 `Accept-Encoding` [Postman](https://www.getpostman.com/)などのツールを使用して要求ヘッダーを設定し、応答ヘッダー、サイズ、および本文を調査します。

ヘッダーを`Accept-Encoding`使用せずにサンプルアプリに要求を送信し、応答が圧縮されていないことを確認します。 応答`Content-Encoding`に`Vary`ヘッダーとヘッダーが存在しません。

![Fiddler ウィンドウには、要求の結果が表示され、エンコーディングヘッダーはありません。 応答が圧縮されていません。](response-compression/_static/request-uncompressed.png)

::: moniker range=">= aspnetcore-2.2"

`Accept-Encoding: br`ヘッダー (Brotli compression) を使用してサンプルアプリに要求を送信し、応答が圧縮されていることを確認します。 応答`Content-Encoding`に`Vary`は、ヘッダーとヘッダーが存在します。

![Fiddler ウィンドウには、要求の結果が表示されます。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed-br.png)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

`Accept-Encoding: gzip`ヘッダーを使用してサンプルアプリに要求を送信し、応答が圧縮されていることを確認します。 応答`Content-Encoding`に`Vary`は、ヘッダーとヘッダーが存在します。

![Fiddler ウィンドウには、要求の結果がエンコーディングヘッダーと gzip の値で表示されます。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。 応答は圧縮されます。](response-compression/_static/request-compressed.png)

::: moniker-end

## <a name="providers"></a>プロバイダー

::: moniker range=">= aspnetcore-2.2"

### <a name="brotli-compression-provider"></a>Brotli 圧縮プロバイダー

<xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProvider> [Brotli 圧縮データ形式](https://tools.ietf.org/html/rfc7932)で応答を圧縮するには、を使用します。

圧縮プロバイダーがに明示的に追加さ<xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>れていない場合は、次のようになります。

* 既定では、Brotli 圧縮プロバイダーは、 [Gzip 圧縮プロバイダー](#gzip-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Brotoli 圧縮プロバイダーを追加する必要があります。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=5)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

圧縮レベルをに<xref:Microsoft.AspNetCore.ResponseCompression.BrotliCompressionProviderOptions>設定します。 Brotli 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| 圧縮レベル | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<BrotliCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

::: moniker-end

### <a name="gzip-compression-provider"></a>Gzip 圧縮プロバイダー

を使用<xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProvider>して、 [Gzip ファイル形式](https://tools.ietf.org/html/rfc1952)で応答を圧縮します。

圧縮プロバイダーがに明示的に追加さ<xref:Microsoft.AspNetCore.ResponseCompression.CompressionProviderCollection>れていない場合は、次のようになります。

::: moniker range=">= aspnetcore-2.2"

* Gzip 圧縮プロバイダーは、既定では、 [Brotli 圧縮プロバイダー](#brotli-compression-provider)と共に、圧縮プロバイダーの配列に追加されます。
* Brotli 圧縮データ形式がクライアントでサポートされている場合、圧縮は既定で Brotli 圧縮になります。 Brotli がクライアントでサポートされていない場合、クライアントで Gzip 圧縮がサポートされていると、圧縮の既定値は Gzip になります。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* Gzip 圧縮プロバイダーは、既定で圧縮プロバイダーの配列に追加されます。
* クライアントで Gzip 圧縮がサポートされている場合、圧縮の既定値は Gzip です。

::: moniker-end

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();
}
```

圧縮プロバイダーが明示的に追加されている場合は、Gzip 圧縮プロバイダーを追加する必要があります。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6)]

::: moniker-end

圧縮レベルをに<xref:Microsoft.AspNetCore.ResponseCompression.GzipCompressionProviderOptions>設定します。 Gzip 圧縮プロバイダーは、既定で最も高速な圧縮レベル ([CompressionLevel](xref:System.IO.Compression.CompressionLevel)) に設定されています。これにより、圧縮率が最も高くなる可能性があります。 最も効率的な圧縮が必要な場合は、最適な圧縮のためにミドルウェアを構成します。

| 圧縮レベル | 説明 |
| ----------------- | ----------- |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は、結果の出力が最適に圧縮されない場合でも、できるだけ早く完了する必要があります。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮は実行されません。 |
| [CompressionLevel](xref:System.IO.Compression.CompressionLevel) | 圧縮が完了するまでに時間がかかる場合でも、応答は最適に圧縮される必要があります。 |

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddResponseCompression();

    services.Configure<GzipCompressionProviderOptions>(options => 
    {
        options.Level = CompressionLevel.Fastest;
    });
}
```

### <a name="custom-providers"></a>カスタムプロバイダー

を使用して、 <xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider>カスタム圧縮実装を作成します。 は<xref:Microsoft.AspNetCore.ResponseCompression.ICompressionProvider.EncodingName*> 、この`ICompressionProvider`によって生成されるコンテンツエンコーディングを表します。 ミドルウェアは、この情報を使用して、要求の`Accept-Encoding`ヘッダーに指定されている一覧に基づいてプロバイダーを選択します。

サンプルアプリを使用して、クライアントは`Accept-Encoding: mycustomcompression`ヘッダーを使用して要求を送信します。 ミドルウェアはカスタム圧縮実装を使用して、応答を`Content-Encoding: mycustomcompression`ヘッダーと共に返します。 カスタム圧縮実装を機能させるには、クライアントがカスタムエンコードを圧縮解除できる必要があります。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/3.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=7)]

[!code-csharp[](response-compression/samples/2.x/SampleApp/CustomCompressionProvider.cs?name=snippet1)]

::: moniker-end

`Accept-Encoding: mycustomcompression`ヘッダーを使用してサンプルアプリに要求を送信し、応答ヘッダーを確認します。 応答`Vary`に`Content-Encoding`は、ヘッダーとヘッダーが存在します。 このサンプルでは、応答本文 (表示されません) は圧縮されません。 サンプルの`CustomCompressionProvider`クラスには圧縮実装がありません。 ただし、このサンプルでは、このような圧縮アルゴリズムを実装する場所を示しています。

![Fiddler ウィンドウには、要求の結果がエンコーディングヘッダーと mycustomcompression の値と共に表示されます。 Vary ヘッダーと Content-type ヘッダーが応答に追加されます。](response-compression/_static/request-custom-compression.png)

## <a name="mime-types"></a>MIME タイプ

ミドルウェアは、圧縮のために既定の MIME の種類のセットを指定します。

* `application/javascript`
* `application/json`
* `application/xml`
* `text/css`
* `text/html`
* `text/json`
* `text/plain`
* `text/xml`

応答の圧縮ミドルウェアオプションを使用して、MIME の種類を置換または追加します。 などの`text/*`ワイルドカードの MIME 型はサポートされていないことに注意してください。 このサンプルアプリでは、の MIME `image/svg+xml`の種類を追加し、ASP.NET Core バナーイメージ (*横断幕*) を圧縮して提供します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](response-compression/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](response-compression/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=8-10)]

::: moniker-end

## <a name="compression-with-secure-protocol"></a>セキュリティで保護されたプロトコルを使用した圧縮

セキュリティで保護された接続での圧縮`EnableForHttps`された応答は、既定では無効になっているオプションを使用して制御できます。 動的に生成されたページで圧縮を使用すると、[犯罪](https://wikipedia.org/wiki/CRIME_(security_exploit))や[侵害](https://wikipedia.org/wiki/BREACH_(security_exploit))攻撃などのセキュリティ上の問題が発生する可能性があります。

## <a name="adding-the-vary-header"></a>Vary ヘッダーを追加する

`Accept-Encoding`ヘッダーに基づいて応答を圧縮する場合、応答の複数の圧縮バージョンと圧縮されていないバージョンが存在する可能性があります。 複数のバージョンが存在し、格納`Vary`する必要があることをクライアントキャッシュとプロキシキャッシュに指示するために、ヘッダーに`Accept-Encoding`値を追加します。 ASP.NET Core 2.0 以降では、応答が圧縮さ`Vary`れるときに、ミドルウェアによってヘッダーが自動的に追加されます。

## <a name="middleware-issue-when-behind-an-nginx-reverse-proxy"></a>Nginx リバースプロキシの背後にあるミドルウェアの問題

要求が Nginx によってプロキシされ`Accept-Encoding`ている場合、ヘッダーは削除されます。 `Accept-Encoding`ヘッダーを削除すると、ミドルウェアが応答を圧縮できなくなります。 詳細については、「[NGINX:圧縮と](https://www.nginx.com/resources/admin-guide/compression-and-decompression/)圧縮解除。 この問題は、 [Nginx のパススルー圧縮 (aspnet/ \#basicmiddleware 123)](https://github.com/aspnet/BasicMiddleware/issues/123)によって追跡されます。

## <a name="working-with-iis-dynamic-compression"></a>IIS 動的圧縮の使用

アプリケーションに対して無効にするサーバーレベルで構成されたアクティブな IIS 動的圧縮モジュールがある場合は、web.config ファイルに追加してモジュールを無効にします。 詳細については、「[Disabling IIS modules](xref:host-and-deploy/iis/modules#disabling-iis-modules)」 (IIS モジュールの無効化) を参照してください。

## <a name="troubleshooting"></a>トラブルシューティング

[Fiddler](https://www.telerik.com/fiddler)、[消火バグ](https://getfirebug.com/)、 [Postman](https://www.getpostman.com/)などのツールを使用します。これに`Accept-Encoding`より、要求ヘッダーを設定し、応答ヘッダー、サイズ、本文を調べることができます。 既定では、応答圧縮ミドルウェアは、次の条件を満たす応答を圧縮します。

::: moniker range=">= aspnetcore-2.2"

* ヘッダー `Accept-Encoding`には`br`、設定したカスタム圧縮`gzip`プロバイダー `*`に一致する値、、、またはカスタムエンコーディングが含まれています。 値は、 `identity`または品質値 (qvalue, `q`) の 0 (ゼロ) に設定することはできません。
* Mime の種類 (`Content-Type`) を設定し、 <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>で構成されている mime の種類と一致させる必要があります。
* 要求にヘッダーを`Content-Range`含めることはできません。
* 応答圧縮ミドルウェアのオプションで secure protocol (https) が構成されている場合を除き、要求では安全でないプロトコル (http) を使用する必要があります。 *セキュリティで保護されたコンテンツの圧縮を有効にする場合は、[前述](#compression-with-secure-protocol)の危険に注意してください。*

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* ヘッダー `Accept-Encoding`には`gzip`、設定したカスタム圧縮`*`プロバイダーに一致する値、、またはカスタムエンコーディングが含まれています。 値は、 `identity`または品質値 (qvalue, `q`) の 0 (ゼロ) に設定することはできません。
* Mime の種類 (`Content-Type`) を設定し、 <xref:Microsoft.AspNetCore.ResponseCompression.ResponseCompressionOptions>で構成されている mime の種類と一致させる必要があります。
* 要求にヘッダーを`Content-Range`含めることはできません。
* 応答圧縮ミドルウェアのオプションで secure protocol (https) が構成されている場合を除き、要求では安全でないプロトコル (http) を使用する必要があります。 *セキュリティで保護されたコンテンツの圧縮を有効にする場合は、[前述](#compression-with-secure-protocol)の危険に注意してください。*

::: moniker-end

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/index>
* [Mozilla Developer Network:Accept-Encoding](https://developer.mozilla.org/docs/Web/HTTP/Headers/Accept-Encoding)
* [RFC 7231 セクション 3.1.2.1:コンテンツの Codings](https://tools.ietf.org/html/rfc7231#section-3.1.2.1)
* [RFC 7230 セクション 4.2.3:Gzip コーディング](https://tools.ietf.org/html/rfc7230#section-4.2.3)
* [GZIP ファイル形式仕様バージョン4.3](https://www.ietf.org/rfc/rfc1952.txt)
