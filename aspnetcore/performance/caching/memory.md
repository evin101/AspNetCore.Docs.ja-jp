---
title: ASP.NET Core 内のメモリ内のキャッシュ
author: rick-anderson
description: ASP.NET Core でメモリにデータをキャッシュする方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 11/2/2019
uid: performance/caching/memory
ms.openlocfilehash: 1114d154ed1af09958df63ae718712177bbf6db0
ms.sourcegitcommit: 09f4a5ded39cc8204576fe801d760bd8b611f3aa
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/05/2019
ms.locfileid: "73611444"
---
# <a name="cache-in-memory-in-aspnet-core"></a>ASP.NET Core 内のメモリ内のキャッシュ

::: moniker range=">= aspnetcore-3.0"

[Rick Anderson](https://twitter.com/RickAndMSFT)、 [John Luo](https://github.com/JunTaoLuo)、および[Steve Smith](https://ardalis.com/)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/3.0sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="caching-basics"></a>キャッシュの基本

キャッシュを使用すると、コンテンツを生成するために必要な作業量を減らすことで、アプリのパフォーマンスとスケーラビリティを大幅に向上させることができます。 キャッシュは、頻繁に変更され **、** 生成に負荷がかかるデータで最適に機能します。 キャッシュを使用すると、ソースよりもはるかに高速に返されるデータのコピーが作成されます。 アプリは、キャッシュされたデータに依存し**ない**ように作成およびテストする必要があります。

ASP.NET Core は、いくつかの異なるキャッシュをサポートしています。 最も単純なキャッシュは、 [IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache)に基づいています。 `IMemoryCache` は、web サーバーのメモリに格納されているキャッシュを表します。 サーバーファーム (複数のサーバー) で実行されているアプリでは、メモリ内キャッシュを使用するときにセッションが固定されていることを確認する必要があります。 固定セッションでは、クライアントからの後続の要求がすべて同じサーバーに送られるようにします。 たとえば、Azure Web apps では、[アプリケーション要求ルーティング](https://www.iis.net/learn/extensions/planning-for-arr)処理 (ARR) を使用して、後続のすべての要求を同じサーバーにルーティングします。

Web ファームの固定されていないセッションでは、キャッシュ整合性の問題を回避するために[分散キャッシュ](distributed.md)が必要です。 アプリによっては、分散キャッシュがメモリ内キャッシュよりも高いスケールアウトをサポートする場合があります。 分散キャッシュを使用すると、キャッシュメモリが外部プロセスにオフロードされます。

メモリ内キャッシュには、任意のオブジェクトを格納できます。 分散キャッシュインターフェイスは `byte[]`に制限されています。 メモリ内および分散キャッシュストアは、キーと値のペアとしてキャッシュ項目を格納します。

## <a name="systemruntimecachingmemorycache"></a>System.string. キャッシュ/MemoryCache

<xref:System.Runtime.Caching> / <xref:System.Runtime.Caching.MemoryCache> ([NuGet パッケージ](https://www.nuget.org/packages/System.Runtime.Caching/)) は、次の方法で使用できます。

* .NET Standard 2.0 以降。
* .NET Standard 2.0 以降を対象とするすべての[.net 実装](/dotnet/standard/net-standard#net-implementation-support)。 たとえば、2.0 以降の ASP.NET Core ます。
* .NET Framework 4.5 以降。

この記事で説明されている / `IMemoryCache` (この記事で説明しています) は、`MemoryCache` に統合[し](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/)た方がよいため `System.Runtime.Caching` / の ASP.NET Core よりも推奨されます。 たとえば、`IMemoryCache` は ASP.NET Core[依存関係の挿入](xref:fundamentals/dependency-injection)とネイティブで動作します。

ASP.NET 4.x から ASP.NET Core にコードを移植するときに、互換性ブリッジとして `MemoryCache` / `System.Runtime.Caching` を使用します。

## <a name="cache-guidelines"></a>キャッシュのガイドライン

* コードには常に、データをフェッチするためのフォールバックオプションがあり、使用可能なキャッシュ値に依存し**ない**ようにする必要があります。
* キャッシュは、不足しているリソース (メモリ) を使用します。 キャッシュ拡張の制限:
  * 外部**入力をキャッシュ**キーとして使用しないでください。
  * キャッシュの拡張を制限するには、有効期限を使用します。
  * [キャッシュサイズを制限するには、SetSize、Size、および SizeLimit を使用](#use-setsize-size-and-sizelimit-to-limit-cache-size)します。 ASP.NET Core ランタイムでは、メモリ負荷に基づいてキャッシュサイズが制限**されません**。 キャッシュサイズを制限するのは開発者だけです。

## <a name="use-imemorycache"></a>IMemoryCache を使用する

> [!WARNING]
> [依存関係の挿入](xref:fundamentals/dependency-injection)から*共有*メモリキャッシュを使用して `SetSize`、`Size`、または `SizeLimit` を呼び出してキャッシュサイズを制限すると、アプリでエラーが発生する可能性があります。 キャッシュにサイズ制限が設定されている場合、すべてのエントリは追加時にサイズを指定する必要があります。 これにより、開発者は共有キャッシュを使用する内容を完全に制御できない場合があるため、問題が発生する可能性があります。 たとえば、Entity Framework Core は共有キャッシュを使用し、サイズを指定しません。 アプリがキャッシュサイズの制限を設定し、EF Core を使用する場合、アプリは `InvalidOperationException` をスローします。
> `SetSize`、`Size`、または `SizeLimit` を使用してキャッシュを制限する場合は、キャッシュ用のキャッシュシングルトンを作成します。 詳細と例については、「 [SetSize、サイズ、および SizeLimit を使用してキャッシュサイズを制限する](#use-setsize-size-and-sizelimit-to-limit-cache-size)」を参照してください。
> 共有キャッシュは、他のフレームワークまたはライブラリによって共有されます。 たとえば、EF Core は共有キャッシュを使用し、サイズを指定しません。 

インメモリキャッシュは、[依存関係の挿入](xref:fundamentals/dependency-injection)を使用してアプリから参照される*サービス*です。 コンストラクターで `IMemoryCache` インスタンスを要求します。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ctor)]

次のコードでは、 [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__)を使用して、時間がキャッシュ内にあるかどうかを確認します。 時間がキャッシュされていない場合は、新しいエントリが作成され、が[設定](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_)されたキャッシュに追加されます。 `CacheKeys` クラスは、ダウンロードサンプルに含まれています。

[!code-csharp[](memory/3.0sample/WebCacheSample/CacheKeys.cs)]

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet1)]

現在の時刻とキャッシュされた時間が表示されます。

[!code-cshtml[](memory/3.0sample/WebCacheSample/Views/Home/Cache.cshtml)]

キャッシュされた `DateTime` の値は、タイムアウト期間内に要求があってもキャッシュに残ります。

次のコードでは、 [Getorcreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__)と[Getorcreateasync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___)を使用してデータをキャッシュしています。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

次のコードは、 [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_)を呼び出して、キャッシュされた時間を取得します。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_gct)]

次のコードは、絶対有効期限があるキャッシュされた項目を取得または作成します。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet99)]

スライド式有効期限付きのキャッシュされた項目セットは、古くなっている可能性があります。 スライディング有効期間よりも頻繁にアクセスされる場合、項目は期限切れになりません。 スライド式有効期限と絶対有効期限を組み合わせて、その絶対有効期限が過ぎると項目が期限切れになることを保証します。 絶対有効期限は、項目をキャッシュできる期間を上限として設定し、スライドしている有効期限の間隔内に項目が要求されていない場合は、前に期限切れになることを許可します。 絶対有効期限とスライド式有効期限の両方を指定した場合、有効期限は論理的に論理和になります。 スライド式有効期限間隔*または*絶対有効期限が経過すると、項目はキャッシュから削除されます。

次のコードは、スライディング*と*絶対の両方の有効期限を持つキャッシュされた項目を取得または作成します。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet9)]

上記のコードでは、データが絶対時間より長くキャッシュされないことが保証されています。

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*>、<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*>、<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.Get*> は、<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions> クラスの拡張メソッドです。 これらのメソッドにより、<xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> の機能が拡張されます。

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

次の例を次に示します。

* スライド式有効期限を設定します。 このキャッシュされた項目にアクセスする要求は、スライド式有効期限をリセットします。
* キャッシュの優先度を[NeverRemove](xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove)に設定します。
* エントリがキャッシュから削除された後に呼び出される[PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate)を設定します。 コールバックは、キャッシュから項目を削除するコードとは異なるスレッドで実行されます。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>SetSize、サイズ、および SizeLimit を使用してキャッシュサイズを制限する

`MemoryCache` インスタンスでは、必要に応じてサイズ制限を指定して適用できます。 キャッシュには、エントリのサイズを測定する機構がないため、キャッシュサイズの制限には定義済みの測定単位がありません。 キャッシュサイズの制限が設定されている場合、すべてのエントリでサイズを指定する必要があります。 ASP.NET Core ランタイムでは、メモリ負荷に基づいてキャッシュサイズが制限されません。 キャッシュサイズを制限するのは開発者だけです。 指定されたサイズは、開発者が選択した単位で示されます。

(例:

* Web アプリが主に文字列をキャッシュしている場合は、各キャッシュエントリのサイズを文字列の長さにすることができます。
* アプリでは、すべてのエントリのサイズを1と指定することができ、サイズ制限はエントリの数です。

<xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> が設定されていない場合、キャッシュはバインドされずに拡張されます。 ASP.NET Core ランタイムは、システムメモリが不足しているとキャッシュをトリミングしません。 アプリは次のように設計されています。

* キャッシュの拡張を制限します。
* 使用可能なメモリが制限されている場合は、<xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> または <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> を呼び出します。

次のコードでは、[依存関係の挿入](xref:fundamentals/dependency-injection)によってアクセスできる、単位なしの固定サイズ <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> が作成されます。

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` には単位がありません。 キャッシュサイズの制限が設定されている場合、キャッシュされたエントリは、最適と思われる任意の単位でサイズを指定する必要があります。 キャッシュインスタンスのすべてのユーザーは、同じ単体システムを使用する必要があります。 キャッシュされたエントリサイズの合計が `SizeLimit` で指定された値を超えた場合、エントリはキャッシュされません。 キャッシュサイズの制限が設定されていない場合、エントリに設定されているキャッシュサイズは無視されます。

次のコードでは `MyMemoryCache` を[依存関係挿入](xref:fundamentals/dependency-injection)コンテナーに登録します。

[!code-csharp[](memory/3.0sample/RPcache/Startup.cs?name=snippet)]

`MyMemoryCache` は、このサイズ制限のキャッシュを認識し、キャッシュエントリのサイズを適切に設定する方法を理解しているコンポーネントの独立したメモリキャッシュとして作成されます。

次のコードでは `MyMemoryCache` が使用されています。

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet)]

キャッシュエントリのサイズは <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.Size> または <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.SetSize*> 拡張メソッドで設定できます。

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache. Compact

`MemoryCache.Compact` は、指定された割合のキャッシュを次の順序で削除しようとします。

* 期限切れのすべての項目。
* 優先順位別の項目。 優先順位の低い項目が最初に削除されます。
* 最近使用したオブジェクトのうち、最も古いオブジェクト。
* 最も古い絶対有効期限を持つ項目。
* 最も古いスライド式有効期限を持つ項目。

優先 <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> を持つピン留めされた項目は削除されません。 次のコードでは、キャッシュ項目を削除して `Compact` を呼び出します。

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

詳細については、 [GitHub の「Compact source](https://github.com/aspnet/Extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) 」を参照してください。

## <a name="cache-dependencies"></a>キャッシュの依存関係

次のサンプルは、依存エントリの有効期限が切れた場合に、キャッシュエントリを期限切れにする方法を示しています。 キャッシュされた項目に <xref:Microsoft.Extensions.Primitives.CancellationChangeToken> が追加されます。 `CancellationTokenSource`で `Cancel` が呼び出されると、両方のキャッシュエントリが削除されます。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ed)]

<xref:System.Threading.CancellationTokenSource> を使用すると、複数のキャッシュエントリを1つのグループとして削除できます。 上記のコードの `using` パターンでは、`using` ブロック内に作成されたキャッシュエントリによって、トリガーと有効期限の設定が継承されます。

## <a name="additional-notes"></a>補足メモ

* 有効期限はバックグラウンドでは発生しません。 期限切れの項目のキャッシュをアクティブにスキャンするタイマーはありません。 キャッシュのすべてのアクティビティ (`Get`、`Set`、`Remove`) は、期限切れの項目のバックグラウンドスキャンをトリガーできます。 `CancellationTokenSource` (<xref:System.Threading.CancellationTokenSource.CancelAfter*>) のタイマーによって、エントリも削除され、期限切れの項目のスキャンがトリガーされます。 次の例では、登録されたトークンに[CancellationTokenSource (TimeSpan)](/dotnet/api/system.threading.cancellationtokensource.-ctor)を使用します。 このトークンが起動すると、エントリが直ちに削除され、削除コールバックが発生します。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ae)]

* コールバックを使用してキャッシュ項目を再作成する場合:

  * コールバックが完了していないため、複数の要求でキャッシュされたキー値を空にすることができます。
  * これにより、複数のスレッドがキャッシュされた項目を再作成する可能性があります。

* あるキャッシュエントリを使用して別のキャッシュエントリを作成すると、子は親エントリの有効期限トークンと時間ベースの有効期限の設定をコピーします。 親エントリを手動で削除または更新しても、子の有効期限は切れません。

* キャッシュからキャッシュエントリが削除された後に起動されるコールバックを設定するには、<xref:Microsoft.Extensions.Caching.Memory.ICacheEntry.PostEvictionCallbacks> を使用します。
* ほとんどのアプリでは、`IMemoryCache` が有効になっています。 たとえば、`AddMvc`、`AddControllersWithViews`、`AddRazorPages`、`AddMvcCore().AddRazorViewEngine`、およびその他の多くの `Add{Service}` メソッドを `ConfigureServices` に呼び出すと、`IMemoryCache` が有効になります。 上記の `Add{Service}` メソッドのいずれかを呼び出さないアプリの場合、`ConfigureServices` で <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddMemoryCache*> を呼び出すことが必要になる場合があります。

## <a name="additional-resources"></a>その他の技術情報

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<!-- This is the 2.1 version -->
[Rick Anderson](https://twitter.com/RickAndMSFT)、 [John Luo](https://github.com/JunTaoLuo)、および[Steve Smith](https://ardalis.com/)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/sample)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="caching-basics"></a>キャッシュの基本

キャッシュを使用すると、コンテンツを生成するために必要な作業量を減らすことで、アプリのパフォーマンスとスケーラビリティを大幅に向上させることができます。 キャッシュは、あまり変更されないデータで最適に機能します。 キャッシュを使用すると、元のソースよりもはるかに高速に返されるデータのコピーが作成されます。 コードは、キャッシュされたデータに依存し**ない**ように記述してテストする必要があります。

ASP.NET Core は、いくつかの異なるキャッシュをサポートしています。 最も単純なキャッシュは、web サーバーのメモリに格納されているキャッシュを表す[IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache)に基づいています。 サーバーファーム (複数のサーバー) で実行されるアプリでは、メモリ内キャッシュを使用するときにセッションが固定されていることを確認する必要があります。 固定セッションでは、クライアントからの以降の要求がすべて同じサーバーに送られるようにします。 たとえば、Azure Web apps は、[アプリケーション要求ルーティング](https://www.iis.net/learn/extensions/planning-for-arr)処理 (ARR) を使用して、ユーザーエージェントからのすべての要求を同じサーバーにルーティングします。

Web ファームの固定されていないセッションでは、キャッシュ整合性の問題を回避するために[分散キャッシュ](distributed.md)が必要です。 アプリによっては、分散キャッシュがメモリ内キャッシュよりも高いスケールアウトをサポートする場合があります。 分散キャッシュを使用すると、キャッシュメモリが外部プロセスにオフロードされます。

メモリ内キャッシュには、任意のオブジェクトを格納できます。 分散キャッシュインターフェイスは `byte[]`に制限されています。 メモリ内および分散キャッシュストアは、キーと値のペアとしてキャッシュ項目を格納します。

## <a name="systemruntimecachingmemorycache"></a>System.string. キャッシュ/MemoryCache

<xref:System.Runtime.Caching> / <xref:System.Runtime.Caching.MemoryCache> ([NuGet パッケージ](https://www.nuget.org/packages/System.Runtime.Caching/)) は、次の方法で使用できます。

* .NET Standard 2.0 以降。
* .NET Standard 2.0 以降を対象とするすべての[.net 実装](/dotnet/standard/net-standard#net-implementation-support)。 たとえば、2.0 以降の ASP.NET Core ます。
* .NET Framework 4.5 以降。

この記事で説明されている / `IMemoryCache` (この記事で説明しています) は、`MemoryCache` に統合[し](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/)た方がよいため `System.Runtime.Caching` / の ASP.NET Core よりも推奨されます。 たとえば、`IMemoryCache` は ASP.NET Core[依存関係の挿入](xref:fundamentals/dependency-injection)とネイティブで動作します。

ASP.NET 4.x から ASP.NET Core にコードを移植するときに、互換性ブリッジとして `MemoryCache` / `System.Runtime.Caching` を使用します。

## <a name="cache-guidelines"></a>キャッシュのガイドライン

* コードには常に、データをフェッチするためのフォールバックオプションがあり、使用可能なキャッシュ値に依存し**ない**ようにする必要があります。
* キャッシュは、不足しているリソース (メモリ) を使用します。 キャッシュ拡張の制限:
  * 外部**入力をキャッシュ**キーとして使用しないでください。
  * キャッシュの拡張を制限するには、有効期限を使用します。
  * [キャッシュサイズを制限するには、SetSize、Size、および SizeLimit を使用](#use-setsize-size-and-sizelimit-to-limit-cache-size)します。 ASP.NET Core ランタイムでは、メモリ負荷に基づいてキャッシュサイズが制限されません。 キャッシュサイズを制限するのは開発者だけです。

## <a name="using-imemorycache"></a>IMemoryCache の使用

> [!WARNING]
> [依存関係の挿入](xref:fundamentals/dependency-injection)から*共有*メモリキャッシュを使用して `SetSize`、`Size`、または `SizeLimit` を呼び出してキャッシュサイズを制限すると、アプリでエラーが発生する可能性があります。 キャッシュにサイズ制限が設定されている場合、すべてのエントリは追加時にサイズを指定する必要があります。 これにより、開発者は共有キャッシュを使用する内容を完全に制御できない場合があるため、問題が発生する可能性があります。 たとえば、Entity Framework Core は共有キャッシュを使用し、サイズを指定しません。 アプリがキャッシュサイズの制限を設定し、EF Core を使用する場合、アプリは `InvalidOperationException` をスローします。
> `SetSize`、`Size`、または `SizeLimit` を使用してキャッシュを制限する場合は、キャッシュ用のキャッシュシングルトンを作成します。 詳細と例については、「 [SetSize、サイズ、および SizeLimit を使用してキャッシュサイズを制限する](#use-setsize-size-and-sizelimit-to-limit-cache-size)」を参照してください。

インメモリキャッシュは、[依存関係の挿入](../../fundamentals/dependency-injection.md)を使用してアプリから参照される*サービス*です。 `ConfigureServices`で `AddMemoryCache` を呼び出します。

[!code-csharp[](memory/sample/WebCache/Startup.cs?highlight=9)]

コンストラクターで `IMemoryCache` インスタンスを要求します。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor)]

`IMemoryCache` には NuGet[パッケージ](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/)AspNetCore が必要です。これは、[メタパッケージ](xref:fundamentals/metapackage-app)で入手できます。

次のコードでは、 [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__)を使用して、時間がキャッシュ内にあるかどうかを確認します。 時間がキャッシュされていない場合は、新しいエントリが作成され、が[設定](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_)されたキャッシュに追加されます。

[!code-csharp[](memory/sample/WebCache/CacheKeys.cs)]

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]

現在の時刻とキャッシュされた時間が表示されます。

[!code-cshtml[](memory/sample/WebCache/Views/Home/Cache.cshtml)]

キャッシュされた `DateTime` の値は、タイムアウト期間内に要求があってもキャッシュに残ります。 次の図は、現在の時刻と、キャッシュから取得された古い時間を示しています。

![2つの異なる時刻が表示されたインデックスビュー](memory/_static/time.png)

次のコードでは、 [Getorcreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__)と[Getorcreateasync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___)を使用してデータをキャッシュしています。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

次のコードは、 [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_)を呼び出して、キャッシュされた時間を取得します。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*>、<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*>、および[Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_)は、<xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> の機能を拡張する[cacheextensions](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions)クラスの拡張メソッドです。 他のキャッシュメソッドの詳細については、「 [IMemoryCache メソッド](/dotnet/api/microsoft.extensions.caching.memory.imemorycache)と[cacheextensions メソッド](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions)」を参照してください。

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

次の例を次に示します。

* スライド式有効期限を設定します。 このキャッシュされた項目にアクセスする要求は、スライド式有効期限をリセットします。
* キャッシュの優先度を `CacheItemPriority.NeverRemove` に設定します。
* エントリがキャッシュから削除された後に呼び出される[PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate)を設定します。 コールバックは、キャッシュから項目を削除するコードとは異なるスレッドで実行されます。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>SetSize、サイズ、および SizeLimit を使用してキャッシュサイズを制限する

`MemoryCache` インスタンスでは、必要に応じてサイズ制限を指定して適用できます。 キャッシュには、エントリのサイズを測定する機構がないため、キャッシュサイズの制限には定義済みの測定単位がありません。 キャッシュサイズの制限が設定されている場合、すべてのエントリでサイズを指定する必要があります。 ASP.NET Core ランタイムでは、メモリ負荷に基づいてキャッシュサイズが制限されません。 キャッシュサイズを制限するのは開発者だけです。 指定されたサイズは、開発者が選択した単位で示されます。

(例:

* Web アプリが主に文字列をキャッシュしている場合は、各キャッシュエントリのサイズを文字列の長さにすることができます。
* アプリでは、すべてのエントリのサイズを1と指定することができ、サイズ制限はエントリの数です。

<xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> が設定されていない場合、キャッシュはバインドされずに拡張されます。 ASP.NET Core ランタイムは、システムメモリが不足しているとキャッシュをトリミングしません。 アプリは次のように設計されています。

* キャッシュの拡張を制限します。
* 使用可能なメモリが制限されている場合は、<xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> または <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> を呼び出します。

次のコードでは、[依存関係の挿入](xref:fundamentals/dependency-injection)によってアクセスできる、単位なしの固定サイズ <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> が作成されます。

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` には単位がありません。 キャッシュサイズの制限が設定されている場合、キャッシュされたエントリは、最適と思われる任意の単位でサイズを指定する必要があります。 キャッシュインスタンスのすべてのユーザーは、同じ単体システムを使用する必要があります。 キャッシュされたエントリサイズの合計が `SizeLimit` で指定された値を超えた場合、エントリはキャッシュされません。 キャッシュサイズの制限が設定されていない場合、エントリに設定されているキャッシュサイズは無視されます。

次のコードでは `MyMemoryCache` を[依存関係挿入](xref:fundamentals/dependency-injection)コンテナーに登録します。

[!code-csharp[](memory/sample/RPcache/Startup.cs?name=snippet&highlight=5)]

`MyMemoryCache` は、このサイズ制限のキャッシュを認識し、キャッシュエントリのサイズを適切に設定する方法を理解しているコンポーネントの独立したメモリキャッシュとして作成されます。

次のコードでは `MyMemoryCache` が使用されています。

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet)]

キャッシュエントリのサイズは、 [size](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryoptions.size?view=aspnetcore-2.1#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_Size)または[SetSize](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryextensions.setsize?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryExtensions_SetSize_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_System_Int64_)拡張メソッドを使用して設定できます。

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache. Compact

`MemoryCache.Compact` は、指定された割合のキャッシュを次の順序で削除しようとします。

* 期限切れのすべての項目。
* 優先順位別の項目。 優先順位の低い項目が最初に削除されます。
* 最近使用したオブジェクトのうち、最も古いオブジェクト。
* 最も古い絶対有効期限を持つ項目。
* 最も古いスライド式有効期限を持つ項目。

優先 <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> を持つピン留めされた項目は削除されません。

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

詳細については、 [GitHub の「Compact source](https://github.com/aspnet/Extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) 」を参照してください。

## <a name="cache-dependencies"></a>キャッシュの依存関係

次のサンプルは、依存エントリの有効期限が切れた場合に、キャッシュエントリを期限切れにする方法を示しています。 キャッシュされた項目に <xref:Microsoft.Extensions.Primitives.CancellationChangeToken> が追加されます。 `CancellationTokenSource`で `Cancel` が呼び出されると、両方のキャッシュエントリが削除されます。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]

`CancellationTokenSource` を使用すると、複数のキャッシュエントリを1つのグループとして削除できます。 上記のコードの `using` パターンでは、`using` ブロック内に作成されたキャッシュエントリによって、トリガーと有効期限の設定が継承されます。

## <a name="additional-notes"></a>補足メモ

* コールバックを使用してキャッシュ項目を再作成する場合:

  * コールバックが完了していないため、複数の要求でキャッシュされたキー値を空にすることができます。
  * これにより、複数のスレッドがキャッシュされた項目を再作成する可能性があります。

* あるキャッシュエントリを使用して別のキャッシュエントリを作成すると、子は親エントリの有効期限トークンと時間ベースの有効期限の設定をコピーします。 親エントリを手動で削除または更新しても、子の有効期限は切れません。

* キャッシュからキャッシュエントリが削除された後に起動されるコールバックを設定するには、 [PostEvictionCallbacks](/dotnet/api/microsoft.extensions.caching.memory.icacheentry.postevictioncallbacks#Microsoft_Extensions_Caching_Memory_ICacheEntry_PostEvictionCallbacks)を使用します。

## <a name="additional-resources"></a>その他の技術情報

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end
