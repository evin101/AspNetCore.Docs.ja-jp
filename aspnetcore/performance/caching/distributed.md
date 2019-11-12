---
title: ASP.NET Core での分散キャッシュ
author: guardrex
description: 特にクラウドまたはサーバーファーム環境で、ASP.NET Core 分散キャッシュを使用してアプリのパフォーマンスとスケーラビリティを向上させる方法について説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/27/2019
uid: performance/caching/distributed
ms.openlocfilehash: dbcdfcd07877fabfe6d18cd4d840b5597afa1afd
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71081547"
---
# <a name="distributed-caching-in-aspnet-core"></a>ASP.NET Core での分散キャッシュ

作成者: [Luke Latham](https://github.com/guardrex)と[Steve Smith](https://ardalis.com/)

分散キャッシュは、複数のアプリサーバーによって共有されるキャッシュであり、通常、それにアクセスするアプリサーバーに外部サービスとして保持されます。 分散キャッシュを使用すると、特にアプリがクラウドサービスまたはサーバーファームでホストされている場合に、ASP.NET Core アプリのパフォーマンスとスケーラビリティを向上させることができます。

分散キャッシュには、キャッシュされたデータが個々のアプリサーバーに格納される他のキャッシュシナリオよりも、いくつかの利点があります。

キャッシュされたデータが分散されると、データは次のようになります。

* は、複数のサーバーに対する複数の要求にわたっ*て一貫し*ています。
* サーバーの再起動とアプリのデプロイが行われます。
* はローカルメモリを使用しません。

分散キャッシュの構成は実装固有です。 この記事では、SQL Server と Redis の分散キャッシュを構成する方法について説明します。 [Ncache](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html) ([GitHub の ncache](https://github.com/Alachisoft/NCache)) などのサードパーティの実装も利用できます。 どの実装が選択されているかにかかわらず、アプリは<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>インターフェイスを使用してキャッシュと対話します。

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/distributed/samples/)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

## <a name="prerequisites"></a>前提条件

::: moniker range=">= aspnetcore-3.0"

SQL Server 分散キャッシュを使用するには、パッケージ[への参照を追加します](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer)。

Redis 分散キャッシュを使用するには、 [Microsoft.Extensions.Caching.StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)パッケージへのパッケージ参照を追加します。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

SQL Server 分散キャッシュを使用するには、[メタパッケージ](xref:fundamentals/metapackage-app)を参照するか、AspNetCore[パッケージへのパッケージ](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer)参照を追加します。

Redis 分散キャッシュを使用するには、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)を参照し、 [Microsoft.Extensions.Caching.StackExchangeRedis](https://www.nuget.org/packages/Microsoft.Extensions.Caching.StackExchangeRedis)パッケージへのパッケージ参照を追加します。 Redis パッケージは`Microsoft.AspNetCore.App`パッケージに含まれていないため、プロジェクトファイルで redis パッケージを個別に参照する必要があります。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

SQL Server 分散キャッシュを使用するには、[メタパッケージ](xref:fundamentals/metapackage-app)を参照するか、AspNetCore[パッケージへのパッケージ](https://www.nuget.org/packages/Microsoft.Extensions.Caching.SqlServer)参照を追加します。

Redis 分散キャッシュを使用するには、 [AspNetCore メタパッケージ](xref:fundamentals/metapackage-app)を参照し[て、パッケージ](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Redis)参照をパッケージに追加します。 Redis パッケージは`Microsoft.AspNetCore.App`パッケージに含まれていないため、プロジェクトファイルで redis パッケージを個別に参照する必要があります。

::: moniker-end

## <a name="idistributedcache-interface"></a>IDistributedCache インターフェイス

<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>インターフェイスには、分散キャッシュ実装の項目を操作するための次のメソッドが用意されています。

* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Get*>は<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.GetAsync*> `byte[]`文字列キーを受け取り、キャッシュ内に存在する場合は、キャッシュされた項目を配列として取得します。 &ndash;
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Set*>は<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.SetAsync*> 、文字列キーを使用`byte[]`して、 &ndash;キャッシュに項目 (配列として) を追加します。
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Refresh*>は<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RefreshAsync*> 、&ndash;キーに基づいてキャッシュ内の項目を更新し、スライド式有効期限タイムアウト (存在する場合) をリセットします。
* <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.Remove*>は<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache.RemoveAsync*> 、&ndash;文字列キーに基づいてキャッシュ項目を削除します。

## <a name="establish-distributed-caching-services"></a>分散キャッシュサービスを確立する

のの<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>実装をに`Startup.ConfigureServices`登録します。 このトピックで説明するフレームワークに用意されている実装は次のとおりです。

* [分散メモリキャッシュ](#distributed-memory-cache)
* [分散 SQL Server キャッシュ](#distributed-sql-server-cache)
* [分散 Redis cache](#distributed-redis-cache)

### <a name="distributed-memory-cache"></a>分散メモリキャッシュ

分散メモリキャッシュ (<xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddDistributedMemoryCache*>) は、項目をメモリに格納するのフレームワーク提供の<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>実装です。 分散メモリキャッシュは実際の分散キャッシュではありません。 キャッシュされた項目は、アプリが実行されているサーバー上のアプリインスタンスによって格納されます。

分散メモリキャッシュは、次のような実装に役立ちます。

* 開発とテストのシナリオで。
* 運用環境で1台のサーバーが使用されていて、メモリの消費量が問題ではない場合。 分散メモリキャッシュを実装すると、キャッシュされたデータストレージが抽象化されます。 これにより、複数のノードまたはフォールトトレランスが必要になった場合に、真の分散キャッシュソリューションを実装できます。

このサンプルアプリでは、の`Startup.ConfigureServices`開発環境でアプリを実行するときに、分散メモリキャッシュを使用します。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedMemoryCache)]

::: moniker-end

### <a name="distributed-sql-server-cache"></a>分散 SQL Server キャッシュ

分散型 SQL Server キャッシュの実装<xref:Microsoft.Extensions.DependencyInjection.SqlServerCachingServicesExtensions.AddDistributedSqlServerCache*>() を使用すると、分散キャッシュで SQL Server データベースをバッキングストアとして使用できます。 SQL Server インスタンスに SQL Server キャッシュされた項目テーブルを作成するには、 `sql-cache`ツールを使用します。 このツールでは、指定した名前とスキーマを使用してテーブルが作成されます。

`sql-cache create`コマンドを実行して SQL Server にテーブルを作成します。 SQL Server インスタンス (`Data Source`)、データベース (`Initial Catalog`)、 `dbo`スキーマ (など)、テーブル名 ( `TestCache`など) を指定します。

```dotnetcli
dotnet sql-cache create "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
```

ツールが正常に実行されたことを示すメッセージがログに記録されます。

```console
Table and index were created successfully.
```

`sql-cache`ツールによって作成されたテーブルには、次のスキーマがあります。

![SqlServer キャッシュテーブル](distributed/_static/SqlServerCacheTable.png)

> [!NOTE]
> アプリでは、では<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache>なく、のインスタンスを使用してキャッシュ値を操作する必要があります。

このサンプルアプリは<xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCache> 、の`Startup.ConfigureServices`開発環境以外で実装されています。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddDistributedSqlServerCache)]

::: moniker-end

> [!NOTE]
> <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.SchemaName*> <xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.TableName*>/ 通常、 (およびオプションで、および)はソース管理の外部に格納されます(たとえば、[シークレット マネージャー](xref:security/app-secrets)またはappsettingsappsettingsに格納されます。{<xref:Microsoft.Extensions.Caching.SqlServer.SqlServerCacheOptions.ConnectionString*> *ENVIRONMENT} json*ファイル)。 接続文字列には、ソース管理システムから保持する必要がある資格情報を含めることができます。

### <a name="distributed-redis-cache"></a>分散 Redis Cache

[Redis](https://redis.io/)は、オープンソースのメモリ内データストアであり、多くの場合、分散キャッシュとして使用されます。 Redis はローカルで使用でき、Azure でホストされる ASP.NET Core アプリの[Azure Redis Cache](https://azure.microsoft.com/services/cache/)を構成できます。

::: moniker range=">= aspnetcore-3.0"

アプリでは、の<xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> `Startup.ConfigureServices`開発以外の環境で<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>インスタンス () を使用して、キャッシュの実装を構成します。

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_AddStackExchangeRedisCache)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

アプリでは、の<xref:Microsoft.Extensions.Caching.StackExchangeRedis.RedisCache> `Startup.ConfigureServices`開発以外の環境で<xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisCacheServiceCollectionExtensions.AddStackExchangeRedisCache*>インスタンス () を使用して、キャッシュの実装を構成します。

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_AddStackExchangeRedisCache)]

::: moniker-end

::: moniker range="< aspnetcore-2.2"

アプリは、 <xref:Microsoft.Extensions.Caching.Redis.RedisCache>インスタンス (<xref:Microsoft.Extensions.DependencyInjection.RedisCacheServiceCollectionExtensions.AddDistributedRedisCache*>) を使用してキャッシュの実装を構成します。

```csharp
services.AddDistributedRedisCache(options =>
{
    options.Configuration = "localhost";
    options.InstanceName = "SampleInstance";
});
```

::: moniker-end

Redis をローカルコンピューターにインストールするには、次のようにします。

* [Chocolatey Redis パッケージ](https://chocolatey.org/packages/redis-64/)をインストールします。
* コマンド`redis-server`プロンプトからを実行します。

## <a name="use-the-distributed-cache"></a>分散キャッシュを使用する

<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>インターフェイスを使用するには、アプリの<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>任意のコンストラクターからのインスタンスを要求します。 インスタンスは、[依存関係の挿入 (DI)](xref:fundamentals/dependency-injection)によって提供されます。

::: moniker range=">= aspnetcore-3.0"

サンプルアプリが起動すると<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> 、がに`Startup.Configure`挿入されます。 現在の時刻はを使用<xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>してキャッシュされます[(詳細については、「汎用ホスト:IHostApplicationLifetime](xref:fundamentals/host/generic-host#ihostapplicationlifetime)):

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

サンプルアプリが起動すると<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache> 、がに`Startup.Configure`挿入されます。 現在の時刻はを使用<xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime>してキャッシュされます[(詳細については、「Web Host:IApplicationLifetime インターフェイス](xref:fundamentals/host/web-host#iapplicationlifetime-interface)):

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Startup.cs?name=snippet_Configure&highlight=10)]

::: moniker-end

サンプルアプリは、 <xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>インデックスページ`IndexModel`で使用するために、に挿入されます。

インデックスページが読み込まれるたびに、キャッシュがで`OnGetAsync`キャッシュされた時間についてチェックされます。 キャッシュされた時間が経過していない場合は、時間が表示されます。 キャッシュされた時間が最後にアクセスされてから20秒経過した場合 (このページが最後に読み込まれたとき)、ページにはキャッシュされた*時間*が表示されます。

[キャッシュされた時間の**リセット**] ボタンを選択して、キャッシュされた時刻を現在の時刻に直ちに更新します。 このボタンをクリック`OnPostResetCachedTime`すると、ハンドラーメソッドがトリガーされます。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](distributed/samples/3.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](distributed/samples/2.x/DistCacheSample/Pages/Index.cshtml.cs?name=snippet_IndexModel&highlight=7,14-20,25-29)]

::: moniker-end

> [!NOTE]
> インスタンスに対して<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>シングルトンまたはスコープ設定された有効期間を使用する必要はありません (少なくとも組み込み実装の場合)。
>
> また、DI を使用<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>する代わりに、必要に応じてインスタンスを作成することもできますが、コードでインスタンスを作成すると、コードのテストが難しくなり、[明示的な依存関係の原則](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)に違反する可能性があります。

## <a name="recommendations"></a>推奨事項

アプリに最適なの<xref:Microsoft.Extensions.Caching.Distributed.IDistributedCache>実装を決定する際には、次の点を考慮してください。

* 既存のインフラストラクチャ
* パフォーマンス要件
* コスト
* チームエクスペリエンス

キャッシュソリューションは、通常、キャッシュされたデータを高速に取得するためにインメモリストレージに依存しますが、メモリは限られたリソースであり、拡張にはコストがかかります。 一般的に使用されるデータのみをキャッシュに格納します。

一般に、Redis cache は、SQL Server キャッシュよりも高いスループットを実現し、待機時間が短くなります。 ただし、通常、ベンチマークはキャッシュ戦略のパフォーマンス特性を判断するために必要です。

SQL Server が分散キャッシュバッキングストアとして使用されている場合、キャッシュに同じデータベースを使用すると、アプリの通常のデータストレージと取得が両方のパフォーマンスに悪影響を与える可能性があります。 分散キャッシュバッキングストアには専用の SQL Server インスタンスを使用することをお勧めします。

## <a name="additional-resources"></a>その他の技術情報

* [Azure での Redis Cache](/azure/azure-cache-for-redis/)
* [Azure での SQL Database](/azure/sql-database/)
* [Web ファームでの NCache 用の IDistributedCache Provider の ASP.NET Core](http://www.alachisoft.com/ncache/aspnet-core-idistributedcache-ncache.html)([GitHub の Ncache](https://github.com/Alachisoft/NCache))
* <xref:performance/caching/memory>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>
* <xref:host-and-deploy/web-farm>
