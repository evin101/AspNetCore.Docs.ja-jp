---
title: IIS を使用した Windows での ASP.NET Core のホスト
author: guardrex
description: Windows Server インターネット インフォメーション サービス (IIS) での ASP.NET Core アプリをホストする方法を説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/26/2019
uid: host-and-deploy/iis/index
ms.openlocfilehash: 179ab4c97426c9d3cb8ed069d2059d767d755533
ms.sourcegitcommit: 16cf016035f0c9acf3ff0ad874c56f82e013d415
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/29/2019
ms.locfileid: "73034263"
---
# <a name="host-aspnet-core-on-windows-with-iis"></a>IIS を使用した Windows での ASP.NET Core のホスト

作成者: [Luke Latham](https://github.com/guardrex)

ASP.NET Core アプリを IIS サーバーに発行する手順のチュートリアルについては、<xref:tutorials/publish-to-iis> をご覧ください。

[.NET Core ホスティング バンドルのインストール](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>サポートされるオペレーティング システム

次のオペレーティング システムがサポートされています。

* Windows 7 以降
* Windows Server 2008 R2 以降

[HTTP.sys サーバー](xref:fundamentals/servers/httpsys) (旧称 WebListener) は、IIS が含まれるリバース プロキシ構成では動作しません。 [Kestrel サーバー](xref:fundamentals/servers/kestrel)を使用します。

Azure でのホスティングの情報については、「<xref:host-and-deploy/azure-apps/index>」を参照してください。

トラブルシューティング ガイダンスについては、「<xref:test/troubleshoot>」を参照してください。

## <a name="supported-platforms"></a>サポートされているプラットフォーム

32 ビット (x86) または 64 ビット (x64) デプロイ用に発行されるアプリがサポートされています。 アプリが次の条件を満たさない限り、32 ビット (x86) .NET Core SDK を使う 32 ビット アプリをデプロイします:

* 64 ビット アプリ用の大規模な仮想メモリ アドレス空間が必要。
* 大規模な IIS スタック サイズが必要。
* 64 ビットのネイティブの依存関係がある。

64 ビット (x64) .NET Core SDK を使って 64 ビット アプリを発行します。 ホスト システム上に 64 ビット ランタイムが存在している必要があります。

::: moniker range=">= aspnetcore-2.2"

## <a name="hosting-models"></a>ホスティング モデル

### <a name="in-process-hosting-model"></a>インプロセス ホスティング モデル

インプロセス ホスティング モデルを使用する場合、ASP.NET Core アプリはその IIS ワーカー プロセスと同じプロセスで実行されます。 インプロセス ホスティングは、パフォーマンスの点でアウトプロセス ホスティングよりも優れています。要求がループバック アダプター (発信されたネットワーク トラフィックを同じコンピューターに送り返すネットワーク インターフェイス) を介してプロキシされることがないからです。 IIS では [Windows プロセス アクティブ化サービス (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) を使用してプロセス管理が処理されます。

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module):

* アプリの初期化を実行します。
  * [CoreCLR](/dotnet/standard/glossary#coreclr) を読み込みます。
  * `Program.Main`.
* IIS ネイティブ要求の有効期間を処理します。

.NET Framework をターゲットにする ASP.NET Core アプリではインプロセス ホスティング モデルはサポートされていません。

次の図は、IIS (ASP.NET Core モジュール) とインプロセスでホストされるアプリとの間のリレーションシップを示しています。

![インプロセス ホスティング シナリオの ASP.NET Core モジュール](index/_static/ancm-inprocess.png)

Web からカーネル モードの HTTP.sys ドライバーに要求が届きます。 このドライバーによって、Web サイトの構成ポート (通常は 80 (HTTP) または 443 (HTTPS)) 上で IIS へのネイティブ要求がルーティングされます。 ASP.NET Core モジュールは、ネイティブ要求を受け取り、それを IIS HTTP サーバー (`IISHttpServer`) に渡します。 IIS HTTP サーバーは IIS 用のインプロセス サーバーの実装であり、要求がネイティブからマネージドに変換されます。

IIS HTTP サーバーによって要求が処理された後、その要求は ASP.NET Core ミドルウェア パイプラインにプッシュされます。 ミドルウェア パイプラインは要求を処理し、`HttpContext` インスタンスとしてアプリのロジックに渡します。 アプリの応答は、IIS の HTTP サーバーを経由して IIS に戻されます。 IIS は、要求を開始したクライアントに応答を送信します。

インプロセス ホスティングは既存のアプリではオプトインになっていますが、[dotnet new](/dotnet/core/tools/dotnet-new) テンプレートは既定ではすべての IIS および IIS Express シナリオにおいてインプロセス ホスティング モデルに設定されています。

`CreateDefaultBuilder` では、<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> メソッドを呼び出し、[CoreCLR](/dotnet/standard/glossary#coreclr) を起動して IIS ワーカー プロセス (*w3wp.exe* または *iisexpress.exe*) 内のアプリをホストすることで、<xref:Microsoft.AspNetCore.Hosting.Server.IServer> インスタンスを追加します。 パフォーマンス テストは、.NET Core アプリのインプロセス ホスティングでは、アプリのアウトプロセス ホスティングや [Kestrel](xref:fundamentals/servers/kestrel) サーバーへのプロキシ要求に比べ、スループットの要求が大幅に高くなることを示しています。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

> [!NOTE]
> 単一ファイルの実行可能ファイルとして公開されたアプリは、インプロセス ホスティング モデルで読み込むことができません。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="out-of-process-hosting-model"></a>アウト プロセス ホスティング モデル

ASP.NET Core アプリは IIS ワーカー プロセスとは独立したプロセスで実行されるため、プロセス管理は ASP.NET Core モジュールによって処理されます。 モジュールでは、最初の要求が届いたときに ASP.NET Core アプリのプロセスが開始され、プロセスがシャットダウンまたはクラッシュした場合はアプリが再起動されます。 この動作は、インプロセスで実行されるアプリであり、[WAS (Windows プロセス アクティブ化サービス)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) によって管理されるアプリと基本的に同じです。

次の図は、IIS (ASP.NET Core モジュール) とアウトプロセスでホストされるアプリとの間のリレーションシップを示しています。

![アウト プロセス ホスティングのシナリオの ASP.NET Core モジュール](index/_static/ancm-outofprocess.png)

要求は、Web からカーネル モードの HTTP.sys ドライバーに到着します。 ドライバーは、Web サイトの構成ポート (通常は 80 (HTTP) または 443 (HTTPS)) で IIS への要求をルーティングします。 モジュールでは、アプリのランダムなポート (ポート 80 または 443 ではありません) で Kestrel に要求が転送されます。

モジュールによってスタートアップ時に環境変数を介してポートが指定されると、<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> 拡張機能によって `http://localhost:{PORT}` をリッスンするようにサーバーが構成されます。 追加のチェックが実行され、モジュールから発生していない要求は拒否されます。 モジュールでは HTTPS 転送がサポートされていないため、要求は HTTPS を介して IIS によって受信された場合でも、HTTP を介して転送されます。

Kestrel によってモジュールから要求が取り込まれた後、その要求は ASP.NET Core ミドルウェア パイプラインにプッシュされます。 ミドルウェア パイプラインは要求を処理し、`HttpContext` インスタンスとしてアプリのロジックに渡します。 IIS 統合によって追加されたミドルウェアでは、カーネルへの要求の転送を考慮して、スキーム、リモート IP、およびパスベースが更新されます。 アプリの応答が IIS に戻され、IIS はその応答を、要求を開始した HTTP クライアントに返します。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ASP.NET Core には、既定のクロスプラットフォーム HTTP サーバーである [Kestrel サーバー](xref:fundamentals/servers/kestrel)が付属しています。

[IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) または [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) を使用すると、アプリは [Kestrel サーバー](xref:fundamentals/servers/index#kestrel)を使用して IIS ワーカー プロセスとは異なるプロセス内で実行されます ("*プロセス外*")。

ASP.NET Core アプリは IIS ワーカー プロセスとは独立したプロセスで実行されるため、プロセス管理はモジュールによって処理されます。 モジュールでは、最初の要求が届いたときに ASP.NET Core アプリのプロセスが開始され、プロセスがシャットダウンまたはクラッシュした場合はアプリが再起動されます。 この動作は、インプロセスで実行されるアプリであり、[WAS (Windows プロセス アクティブ化サービス)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) によって管理されるアプリと基本的に同じです。

次の図は、IIS (ASP.NET Core モジュール) とアウトプロセスでホストされるアプリとの間のリレーションシップを示しています。

![ASP.NET Core モジュール](index/_static/ancm-outofprocess.png)

要求は、Web からカーネル モードの HTTP.sys ドライバーに到着します。 ドライバーは、Web サイトの構成ポート (通常は 80 (HTTP) または 443 (HTTPS)) で IIS への要求をルーティングします。 モジュールでは、アプリのランダムなポート (ポート 80 または 443 ではありません) で Kestrel に要求が転送されます。

モジュールによってスタートアップ時に環境変数を介してポートが指定されると、[IIS 統合ミドルウェア](xref:host-and-deploy/iis/index#enable-the-iisintegration-components)によって `http://localhost:{port}` をリッスンするようにサーバーが構成されます。 追加のチェックが実行され、モジュールから発生していない要求は拒否されます。 モジュールでは HTTPS 転送がサポートされていないため、要求は HTTPS を介して IIS によって受信された場合でも、HTTP を介して転送されます。

Kestrel によってモジュールから要求が取り込まれた後、その要求は ASP.NET Core ミドルウェア パイプラインにプッシュされます。 ミドルウェア パイプラインは要求を処理し、`HttpContext` インスタンスとしてアプリのロジックに渡します。 IIS 統合によって追加されたミドルウェアでは、カーネルへの要求の転送を考慮して、スキーム、リモート IP、およびパスベースが更新されます。 アプリの応答が IIS に戻され、IIS はその応答を、要求を開始した HTTP クライアントに返します。

`CreateDefaultBuilder` は [Kestrel](xref:fundamentals/servers/kestrel) サーバーを Web サーバーとして構成し、ベース パスとポートを [ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)に構成することで、IIS 統合を有効にします。

ASP.NET Core モジュールでは、バックエンド プロセスに割り当てる動的なポートが生成されます。 `CreateDefaultBuilder` では <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> メソッドが呼び出されます。 `UseIISIntegration` では、localhost の IP アドレス (`127.0.0.1`) で動的なポートをリッスンするように Kestrel が構成されます。 動的なポートが 1234 である場合、Kestrel は `127.0.0.1:1234` でリッスンします。 この構成によって、以下から提供されるその他の URL 構成が置き換えられます。

* `UseUrls`
* [Kestrel の Listen API](xref:fundamentals/servers/kestrel#endpoint-configuration)
* [構成](xref:fundamentals/configuration/index) (または[コマンド ラインの --urls オプション](xref:fundamentals/host/web-host#override-configuration))

モジュールを使用する場合、`UseUrls` または Kestrel の `Listen` API を呼び出す必要はありません。 `UseUrls` または `Listen` が呼び出されると、IIS なしでアプリを実行しているときのみ、Kestrel で指定したポートがリッスンされます。

::: moniker-end

ASP.NET Core モジュール構成のガイダンスについては、「<xref:host-and-deploy/aspnet-core-module>」を参照してください。

ホスティングの詳細については、「[Hosting in ASP.NET Core](xref:fundamentals/index#host)」(ASP.NET Core でのホスティング) を参照してください。

## <a name="application-configuration"></a>アプリケーション構成

### <a name="enable-the-iisintegration-components"></a>IISIntegration コンポーネントを有効にする

一般的な *Program.cs* では、<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> を呼び出して、IIS との統合を有効にするホストの設定を開始します。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

### <a name="iis-options"></a>IIS のオプション

::: moniker range=">= aspnetcore-2.2"

**インプロセス ホスティング モデル**

IIS サーバーのオプションを構成するには、<xref:Microsoft.AspNetCore.Builder.IISServerOptions> 用のサービス構成を <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*> に含めます。 次の例では、AutomaticAuthentication が無効になります。

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

| オプション                         | 既定値 | 設定 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | `true` の場合、IIS サーバーが [Windows 認証](xref:security/authentication/windowsauth)によって認証された `HttpContext.User` を設定します。 `false` の場合、サーバーは `HttpContext.User` の ID を提供するだけで、`AuthenticationScheme` によって明示的に要求されたときにチャレンジに応答します。 `AutomaticAuthentication` を機能させるためには、IIS で Windows 認証を有効にする必要があります。 詳細については、[Windows 認証](xref:security/authentication/windowsauth)に関する記事を参照してください。 |
| `AuthenticationDisplayName`    | `null`  | ログイン ページでユーザーに表示名が表示されるように設定します。 |
| `AllowSynchronousIO`           | `false` | `HttpContext.Request` および `HttpContext.Response` に対して同期 IO が許可されるか。 |
| `MaxRequestBodySize`           | `30000000`  | `HttpRequest` の最大要求本文サイズを取得または設定します。 IIS 自体に、`IISServerOptions` に設定された `MaxRequestBodySize` の前に処理される上限 `maxAllowedContentLength` があることに注意してください。 `MaxRequestBodySize` を変更しても、`maxAllowedContentLength` に影響はありません。 `maxAllowedContentLength`を引き上げるには、*web.config* 内にエントリを追加して `maxAllowedContentLength` をより高い値に設定します。 詳細については、「[Configuration (構成)](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/#configuration)」を参照してください。 |

**アウトプロセス ホスティング モデル**

::: moniker-end

::: moniker range="< aspnetcore-3.0"

| オプション                         | 既定値 | 設定 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | `true` の場合、IIS サーバーが [Windows 認証](xref:security/authentication/windowsauth)によって認証された `HttpContext.User` を設定します。 `false` の場合、サーバーは `HttpContext.User` の ID を提供するだけで、`AuthenticationScheme` によって明示的に要求されたときにチャレンジに応答します。 `AutomaticAuthentication` を機能させるためには、IIS で Windows 認証を有効にする必要があります。 詳細については、[Windows 認証](xref:security/authentication/windowsauth)に関する記事を参照してください。 |
| `AuthenticationDisplayName`    | `null`  | ログイン ページでユーザーに表示名が表示されるように設定します。 |

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

**アウトプロセス ホスティング モデル**

::: moniker-end

IIS オプションを構成するには、<xref:Microsoft.AspNetCore.Builder.IISOptions> 用のサービス構成を <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*> に含めます。 次の例では、アプリが `HttpContext.Connection.ClientCertificate` を設定できません。

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| オプション                         | 既定値 | 設定 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | `true` の場合、[IIS 統合ミドルウェア](#enable-the-iisintegration-components)によって、[Windows 認証](xref:security/authentication/windowsauth)で認証された `HttpContext.User` が設定されます。 `false` の場合、ミドルウェアは `HttpContext.User` の ID を提供するだけで、`AuthenticationScheme` によって明示的に要求されたときに課題に応答します。 `AutomaticAuthentication` を機能させるためには、IIS で Windows 認証を有効にする必要があります。 詳細については、「[Windows 認証](xref:security/authentication/windowsauth)」のトピックを参照してください。 |
| `AuthenticationDisplayName`    | `null`  | ログイン ページでユーザーに表示名が表示されるように設定します。 |
| `ForwardClientCertificate`     | `true`  | `true` の場合、`MS-ASPNETCORE-CLIENTCERT` 要求ヘッダーが指定されていると、`HttpContext.Connection.ClientCertificate` が設定されます。 |

### <a name="proxy-server-and-load-balancer-scenarios"></a>プロキシ サーバーとロード バランサーのシナリオ

要求が発生したスキーム (HTTP/HTTPS) とリモート IP アドレスを転送するために、Forwarded Headers Middleware を構成する [IIS 統合ミドルウェア](#enable-the-iisintegration-components)と、ASP.NET Core モジュールが構成されます。 追加のプロキシ サーバーとロード バランサーの背後でホストされているアプリでは、追加の構成が必要になる場合があります。 詳細については、「[プロキシ サーバーとロード バランサーを使用するために ASP.NET Core を構成する](xref:host-and-deploy/proxy-load-balancer)」を参照してください。

### <a name="webconfig-file"></a>web.config ファイル

*web.config* ファイルでは、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)を設定します。 *web.config* ファイルの作成、変換、および公開は、プロジェクトの公開時に、MSBuild ターゲット (`_TransformWebConfig`) によって処理されます。 このターゲットは、Web SDK ターゲット (`Microsoft.NET.Sdk.Web`) に存在します。 SDK は、プロジェクト ファイルの先頭で設定されています。

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

プロジェクト内に *web.config* ファイルが存在しない場合、そのファイルは ASP.NET Core モジュールを構成するための適切な *processPath* と *arguments* を使用して作成され、[発行された出力](xref:host-and-deploy/directory-structure)に移行されます。

プロジェクトに *web.config* ファイルが含まれていない場合、そのファイルは ASP.NET Core モジュールを構成するための正しい *processPath* と *arguments* を使用して作成され、発行された出力に移行されます。 変換によりファイル内の IIS 構成の設定が変わることはありません。

*web.config* ファイルは、アクティブな IIS モジュールを制御する追加の IIS 構成設定を提供する可能性があります。 ASP.NET Core アプリを使用して要求を処理できる IIS モジュールの詳細については、[IIS モジュール](xref:host-and-deploy/iis/modules)のトピックを参照してください。

Web SDK によって *web.config* ファイルが変換されないようにするため、 **\<IsTransformWebConfigDisabled>** プロパティをプロジェクト ファイルで使用します。

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

Web SDK ファイルの変換を無効にすると、 *processPath*と*引数*開発者によって手動で設定する必要があります。 詳細については、<xref:host-and-deploy/aspnet-core-module> を参照してください。

### <a name="webconfig-file-location"></a>web.config ファイルの場所

[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)を正しく設定するためには、*web.config* ファイルが、展開されるアプリの[コンテンツ ルート](xref:fundamentals/index#content-root) パス (通常はアプリ ベース パス) に存在する必要があります。 これは、IIS に提供される web サイトの物理パスと同じ場所です。 *Web.config* ファイルは、Web 配置を使用して複数のアプリの発行を有効にするため、アプリのルートで必要です。

アプリの物理パスには、 *\<assembly>.runtimeconfig.json*、 *\<assembly>.xml* (XML ドキュメントのコメント)、 *\<assembly>.deps.json* などの機密性の高いファイルが存在します。 *web.config* ファイルが存在し、サイトは通常どおり起動した場合、IIS は、これらの機密性の高いファイルが要求された場合にファイルを提供しません。 *web.config*ファイルが存在しないか、不適切な名前が付けられているか、または通常の起動用にサイトを構成できない場合、IIS が機密性の高いファイルを公開する可能性があります。

***web.config*ファイルは、展開環境に常に存在し、適切な名前が付けられ、通常の起動用にサイトを構成できる必要があります。*web.config* ファイルを運用環境の展開から削除しないでください。**

### <a name="transform-webconfig"></a>web.config を変換する

発行時に *web.config* を変換する必要がある場合 (たとえば、構成、プロファイル、環境に基づいて環境変数を設定する場合) は、<xref:host-and-deploy/iis/transform-webconfig> を参照してください。

## <a name="iis-configuration"></a>IIS 構成

**Windows Server オペレーティング システム**

**Web サーバー (IIS)** サーバーの役割を有効にし、役割のサービスを設定します。

1. **[管理]** メニューから**役割と機能の追加**ウィザードを使用するか、**サーバー マネージャー**にあるリンクを使用します。 **[サーバーの役割]** のステップで、 **[Web サーバー (IIS)]** チェック ボックスをオンにします。

   ![[サーバーの役割の選択] のステップで Web サーバー IIS の役割を選択します。](index/_static/server-roles-ws2016.png)

1. **[機能]** のステップの後に、 **[役割サービスの]** のステップで Web サーバー (IIS) を読み込みます。 希望する IIS の役割サービスを選択するか、既定の役割サービスをそのまま使用します。

   ![[役割サービスの選択] のステップで既定の役割サービスを選択します。](index/_static/role-services-ws2016.png)

   **Windows 認証 (任意)**  
   Windows 認証を有効にするには、 **[Web サーバー]**  >  **[セキュリティ]** ノードを展開します。 **[Windows 認証]** 機能を選択します。 詳細については、「[Windows Authentication \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/)」および「[Configure Windows authentication](xref:security/authentication/windowsauth)」(Windows 認証の構成) を参照してください。

   **WebSocket (省略可能)**  
   WebSockets は、ASP.NET Core 1.1 以降でサポートされています。 WebSocket を有効にするには、 **[Web サーバー]**  >  **[アプリケーション開発]** ノードを展開します。 **[WebSocket プロトコル]** 機能を選択します。 詳細については、[WebSockets](xref:fundamentals/websockets) に関するページを参照してください。

1. **[確認]** のステップを経て Web サーバーの役割とサービスをインストールします。 **Web サーバー (IIS)** の役割をインストールした後、サーバーと IIS の再起動は必要ありません。

**Windows デスクトップ オペレーティング システム**

**[IIS 管理コンソール]** と **[World Wide Web サービス]** を有効にします。

1. **[コントロール パネル]**  >  **[プログラム]**  >  **[プログラムと機能]**  >  **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。

1. **インターネット インフォメーション サービス (IIS)** ノードを開きます。 **[Web 管理ツール]** ノードを開きます。

1. **[IIS 管理コンソール]** チェック ボックスをオンにします。

1. **[World Wide Web サービス]** チェック ボックスをオンにします。

1. **[World Wide Web サービス]** の既定の機能をそのまま使用するか、IIS 機能をカスタマイズします。

   **Windows 認証 (任意)**  
   Windows 認証を有効にするには、 **[World Wide Web サービス]**  >  **[セキュリティ]** ノードを展開します。 **[Windows 認証]** 機能を選択します。 詳細については、「[Windows Authentication \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/)」および「[Configure Windows authentication](xref:security/authentication/windowsauth)」(Windows 認証の構成) を参照してください。

   **WebSocket (省略可能)**  
   WebSockets は、ASP.NET Core 1.1 以降でサポートされています。 WebSocket を有効にするには、 **[World Wide Web サービス]**  >  **[アプリケーション開発機能]** ノードを展開します。 **[WebSocket プロトコル]** 機能を選択します。 詳細については、[WebSockets](xref:fundamentals/websockets) に関するページを参照してください。

1. IIS のインストールに再起動が必要な場合は、システムを再起動します。

![[Windows の機能] で [IIS 管理コンソール] と [World Wide Web サービス] を選択します。](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>.NET Core ホスティング バンドルのインストール

ホスティング システムに *.NET Core ホスティング バンドル*をインストールします。 このバンドルをインストールすることで、.NET Core ランタイム、.NET Core ライブラリ、[ASP.NET Core モジュール](xref:host-and-deploy/aspnet-core-module)がインストールされます。 このモジュールでは、ASP.NET Core アプリが IIS の背後で実行できるようになります。

> [!IMPORTANT]
> ホスティング バンドルが IIS の前にインストールされている場合、バンドルのインストールを修復する必要があります。 IIS をインストールした後に、ホスティング バンドル インストーラーをもう一度実行します。
>
> .NET Core の 64 ビット (x64) バージョンをインストールした後、ホスティング バンドルをインストールした場合は、SDK が表示されない可能性があります ([.NET Core SDK が検出されない](xref:test/troubleshoot#no-net-core-sdks-were-detected))。 この問題を解決するには、<xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle> を参照してください。

### <a name="direct-download-current-version"></a>直接ダウンロード (現在のバージョン)

次のリンクを使用してインストーラーをダウンロードします。

[現在の .NET Core ホスティング バンドルのインストーラー (直接ダウンロード)](https://www.microsoft.com/net/permalink/dotnetcore-current-windows-runtime-bundle-installer)

### <a name="earlier-versions-of-the-installer"></a>以前のバージョンのインストーラー

以前のバージョンのインストーラーを入手するには:

1. [.NET のダウンロードのアーカイブ](https://www.microsoft.com/net/download/archives)に移動します。
1. **[.NET Core]** の下で、.NET Core のバージョンを選択します。
1. **[Run apps - Runtime]\(アプリの実行 - ランタイム\)** 列から、目的の .NET Core ランタイム バージョンの行を探します。
1. **[Runtime & Hosting Bundle]\(ランタイム & ホスティング バンドル\)** のリンクを使用してインストーラーをダウンロードします。

> [!WARNING]
> 一部のインストーラーには、有効期限 (EOL) に達して Microsoft によるサポートが終了した、リリース バージョンが含まれています。 詳細については、[サポート ポリシー](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)をご覧ください。

### <a name="install-the-hosting-bundle"></a>ホスティング バンドルをインストールする

1. サーバーでインストーラーを実行します。 管理者のコマンド シェルからインストーラーを実行する場合、次のパラメーターを使用できます。

   * `OPT_NO_ANCM=1` &ndash; ASP.NET Core モジュールのインストールをスキップします。
   * `OPT_NO_RUNTIME=1` &ndash; .NET Core ランタイムのインストールをスキップします。
   * `OPT_NO_SHAREDFX=1` &ndash; ASP.NET Shared Framework (ASP.NET ランタイム) のインストールをスキップします。
   * `OPT_NO_X86=1` &ndash; x86 ランタイムのインストールをスキップします。 32 ビット アプリをホストしない場合は、このパラメーターを使用します。 今後、32 ビットと 64 ビットのアプリ両方をホストする可能性がある場合は、このパラメーターを使用せずに、両方のランタイムをインストールします。
   * `OPT_NO_SHARED_CONFIG_CHECK=1` &ndash; 共有構成 (*applicationHost.config*) が IIS のインストールと同じマシン上にある場合、IIS 共有構成を使うためのチェックを無効にします。 "*ASP.NET Core 2.2 以降の Hosting Bundler インストーラーに対してのみ使用できます。* " 詳細については、<xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration> を参照してください。
1. システムを再起動するか、コマンド シェルから **net stop was /y** に続けて **net start w3svc** を実行します。 IIS を再起動すると、インストーラーによって行われたシステム パスへの変更 (環境変数) が取得されます。

ホスティング バンドルをインストールするときに、IIS 内の個々のサイトを手動で停止する必要はありません。 ホストされているアプリ (IIS サイト) は、IIS の再起動時に再起動します。 アプリは、最初の要求 ([Application Initialization モジュール](#application-initialization-module-and-idle-timeout)からの要求など) を受信すると再起動します。

ASP.NET Core では、共有フレームワーク パッケージの修正プログラムのリリースに対してロールフォワード動作が採用されています。 IIS によってホストされているアプリが IIS で再起動された場合、そのアプリで最初の要求を受け取ったときに、各自の参照されているパッケージに対する最新の修正プログラムのリリースが読み込まれます。 IIS が再起動されない場合は、アプリのワーカー プロセスがリサイクルされてアプリで最初の要求を受信したときに、アプリが再起動され、ロールフォワード動作が実行されます。

> [!NOTE]
> IIS 共有構成の詳細については、「[ASP.NET Core Module with IIS Shared Configuration](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration)」 (IIS 共有構成の ASP.NET Core モジュール) を参照してください。

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>Visual Studio で発行する場合の Web 配置のインストール

[Web 配置](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)を使用してサーバーにアプリを展開する場合、Web 配置の最新バージョンをサーバーにインストールします。 Web 配置をインストールするには、[Web Platform Installer (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) を使用するか、[Microsoft ダウンロード センター](https://www.microsoft.com/download/details.aspx?id=43717)からインストーラーを取得します。 WebPI を使用することをお勧めします。 WebPI は、スタンドアロンのセットアップとホスティング プロバイダー向けの構成を提供します。

## <a name="create-the-iis-site"></a>IIS サイトを作成する

1. ホスト システムで、アプリの公開フォルダーとファイルを格納するフォルダーを作成します。 以下の手順では、フォルダーのパスはアプリへの物理パスとして IIS に提供されます。 アプリの配置フォルダーとファイル レイアウトについて詳しくは、<xref:host-and-deploy/directory-structure> をご覧ください。

1. IIS マネージャーの **[接続]** パネルで、サーバーのノードを開きます。 **[サイト]** フォルダーを右クリックします。 コンテキスト メニューで **[Web サイトの追加]** を選択します。

1. **[サイト名]** を指定し、 **[物理パス]** にはアプリの配置フォルダーを設定します。 **[バインド]** の構成を指定して **[OK]** を選択し、Web サイトを作成します。

   ![[Web サイトの追加] のステップでサイト名、物理パス、ホスト名を指定します。](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > 最上位のワイルドカードのバインド ( `http://*:80/` と `http://+:80` ) は使用しては **いけません** 。 最上位のワイルドカードのバインドは、セキュリティの脆弱性に対してアプリを切り開くことができます。 これは、強力と脆弱の両方のワイルドカードに適用されます。 ワイルドカードではなく、明示的なホスト名を使用します。 全体の親ドメインを制御する場合、サブドメイン ワイルドカード バインド (たとえば、`*.mysub.com`) にこのセキュリティ リスクはありません (脆弱である `*.com` とは対照的)。 詳細については、[rfc7230 セクション-5.4](https://tools.ietf.org/html/rfc7230#section-5.4) を参照してください。

1. サーバーのノードでは、 **[アプリケーション プール]** を選択します。

1. サイトのアプリケーション プールを右クリックし、コンテキスト メニューから **[基本設定]** を選択します。

1. **[アプリケーション プールの編集]** ウィンドウで、 **[.NET CLR バージョン]** を **[マネージド コードなし]** に設定します。

   ![.NET CLR バージョンとして [マネージド コードなし] を設定します。](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core は、別個のプロセスで実行され、ランタイムを管理します。 ASP.NET Core はデスクトップ CLR (.NET CLR) の読み込みに依存しません&mdash;.NET Core 用の Core 共通言語ランタイム (CoreCLR) が起動され、ワーカー プロセスでアプリがホストされます。 **[.NET CLR バージョン]** の **[マネージド コードなし]** への設定は省略可能ですが、推奨されます。

1. *ASP.NET Core 2.2 以降*:[インプロセス ホスティング モデル](#in-process-hosting-model)を使用する 64 ビット (x64) の[自己完結型展開](/dotnet/core/deploying/#self-contained-deployments-scd)の場合、32 ビット (x86) プロセス用のアプリケーション プールを無効にします。

   IIS マネージャー > **[アプリケーション プール]** の **[操作]** サイドバーで、 **[アプリケーション プールの既定値の設定]** または **[詳細設定]** を選択します。 **[32 ビット アプリケーションの有効化]** を探し、値を `False` に設定します。 この設定は[アウトプロセス ホスティング](xref:host-and-deploy/aspnet-core-module#out-of-process-hosting-model)で展開されたアプリには影響しません。

1. プロセス モデル ID に適切なアクセス許可があることを確認します。

   アプリ プールの既定の ID ( **[プロセス モデル]**  >  **[ID]** ) を **ApplicationPoolIdentity** から別の ID に変更した場合は、アプリのフォルダー、データベース、その他の必要なリソースにアクセスするために要求されるアクセス許可が新しい ID に設定されていることを確認します。 たとえば、アプリケーション プールには、アプリがファイルの読み取りおよび書き込みを行うフォルダーへの読み取りおよび書き込みアクセスが必要です。

**Windows 認証の構成 (任意)**  
詳細については、「[Windows 認証を構成する](xref:security/authentication/windowsauth)」を参照してください。

## <a name="deploy-the-app"></a>アプリを配置する

「[IIS サイトを作成する](#create-the-iis-site)」セクションで確立された IIS の **[物理パス]** フォルダーにアプリを配置します。 お勧めの配置方法は [Web 配置](/iis/publish/using-web-deploy/introduction-to-web-deploy)ですが、プロジェクトの *publish* フォルダーからホスティング システムの配置フォルダーにアプリを移動するためのオプションはいくつか存在します。

### <a name="web-deploy-with-visual-studio"></a>Visual Studio からのアプリの展開

Web 配置に使用する発行プロファイルの作成方法については、「[Visual Studio publish profiles for ASP.NET Core app deployment](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)」 (ASP.NET Core アプリ展開用の Visual Studio の発行プロファイル) のトピックを参照してください。 ホスティング プロバイダーから発行プロファイルまたはそれを作成するためのサポートが提供されている場合は、そのプロファイルをダウンロードし、Visual Studio の **[発行]** ダイアログを使用してインポートします。

![[発行] ダイアログ ページ](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>Visual Studio 外部での Web 配置

Visual Studio の外部で、コマンド ラインから [Web 配置](/iis/publish/using-web-deploy/introduction-to-web-deploy)を使用することもできます。 詳細については、[Web 配置ツール](/iis/publish/using-web-deploy/use-the-web-deployment-tool)に関するページを参照してください。

### <a name="alternatives-to-web-deploy"></a>Web 配置の代替手段

手動コピー、[Xcopy](/windows-server/administration/windows-commands/xcopy)、[Robocopy](/windows-server/administration/windows-commands/robocopy)、または [PowerShell](/powershell/) など、いくつかあるうちの任意の方法を使って、ホスティング システムにアプリを移動します。

IIS への ASP.NET Core の展開の詳細については、「[IIS 管理者用の展開リソース](#deployment-resources-for-iis-administrators)」を参照してください。

## <a name="browse-the-website"></a>Web サイトを閲覧する

ホスティング システムにアプリを配置したら、アプリのパブリック エンドポイントの 1 つに要求を行います。

次の例では、サイトは IIS の**ホスト名** `www.mysite.com` に**ポート** `80` でバインドされています。 `http://www.mysite.com` に対して要求が行われます。

![Microsoft Edge ブラウザーに IIS のスタートアップ ページが読み込まれています。](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>ロックされた展開ファイル

アプリが実行中は、展開フォルダー内のファイルがロックされます。 展開中にロックされたファイルを上書きすることはできません。 展開でロックされているファイルをリリースするには、次の**いずれか**の方法を使用してアプリ プールを停止します。

* Web 配置を使用し、プロジェクト ファイル内の `Microsoft.NET.Sdk.Web` を参照して停止します。 *app_offline.htm* ファイルは、Web アプリのディレクトリのルートに配置されます。 ファイルが存在する場合、ASP.NET Core モジュールはアプリを正常にシャットダウンし、展開中に *app_offline.htm* ファイルを提供します。 詳細については、「[ASP.NET Core モジュール構成リファレンス](xref:host-and-deploy/aspnet-core-module#app_offlinehtm)」を参照してください。
* サーバー上の IIS マネージャーで手動によりアプリ プールを停止します。
* PowerShell を使用して *app_offline.html* をドロップします (PowerShell 5 以降が必要)。

  ```PowerShell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm

  ```

## <a name="data-protection"></a>データの保護

[ASP.NET Core データ保護スタック](xref:security/data-protection/introduction)は、認証で使用されるミドルウェアを含め、いくつかの [ASP.NET Core](xref:fundamentals/middleware/index) ミドルウェアによって使用されます。 データ保護 API がユーザーのコードから呼び出されない場合でも、配置スクリプトを使用するかユーザー コードで、永続的な暗号化[キー ストア](xref:security/data-protection/implementation/key-management)を作成するようにデータ保護を構成する必要があります。 データ保護を構成しない場合、既定でキーはメモリ内に保持され、アプリが再起動すると破棄されます。

キーリングがメモリに格納されている場合、アプリを再起動すると次のことが行われます。

* すべての cookie ベースの認証トークンは無効になります。 
* ユーザーは、次回の要求時に再度サインインする必要があります。 
* キーリングで保護されているデータは、いずれも復号化できなくなります。 これには、[CSRF トークン](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)と [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata) が含まれます。

キーリングを保持するために IIS でのデータ保護を構成するには、次の**いずれか**の方法を使用する必要があります。

* **データ保護のレジストリ キーを作成する**

  ASP.NET Core アプリで使用されるデータ保護キーは、アプリ外部のレジストリに格納されます。 特定のアプリのキーを保持するには、アプリ プール用のレジストリ キーを作成する必要があります。

  非 Web ファーム IIS のスタンドアロン インストールの場合、ASP.NET Core アプリで使用するアプリ プールごとに、[データ保護の PowerShell スクリプト Provision-AutoGenKeys.ps1](https://github.com/aspnet/AspNetCore/blob/master/src/DataProtection/Provision-AutoGenKeys.ps1) を使用できます。 このスクリプトは、HKLM レジストリにレジストリ キーを作成します。そのキーは、アプリのアプリ プールのワーカー プロセス アカウントのみがアクセスできます。 キーは保存時に、DPAPI を使用して、コンピューター全体に適用するキーによって暗号化されます。

  Web ファームのシナリオでは、UNC パスを使用してデータ保護キー リングを格納するようにアプリを構成できます。 既定では、データ保護キーは暗号化されません。 ネットワーク共有のファイルのアクセス許可が、アプリを実行する Windows アカウントに限定されていることを確認します。 保存時のキーを保護するために、X509 証明書を使用することができます。 ユーザーが証明書をアップロードできるメカニズムを検討している場合、ユーザーの信頼できる証明書ストアに証明書を配置し、ユーザーのアプリが実行されるすべてのコンピューターで証明書を利用できるようにします。 詳細については、「[ASP.NET Core データ保護の構成](xref:security/data-protection/configuration/overview)」を参照してください。

* **ユーザー プロファイルを読み込むための IIS アプリケーション プールを構成する**

  この設定は、アプリ プールの **[詳細設定]** の **[プロセス モデル]** セクションにあります。 **[ユーザー プロファイルの読み込み]** を `True` に設定します。 `True` に設定すると、キーはユーザー プロファイル ディレクトリに格納され、ユーザー アカウントに固有のキーと共にデータ保護 API を使って保護されます。 キーは *%LOCALAPPDATA%/ASP.NET/DataProtection-Keys* フォルダーに保存されます。

  アプリ プールの [setProfileEnvironment 属性](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)も有効にする必要があります。 `setProfileEnvironment` の既定値は `true`です。 一部のシナリオ (たとえば、Windows OS) では、`setProfileEnvironment` は `false` に設定されます。 キーが期待どおりにユーザー プロファイル ディレクトリに格納されていない場合:

  1. *%windir%/system32/inetsrv/config* フォルダーに移動します。
  1. *applicationHost.config* ファイルを開きます。
  1. `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 要素を探します。
  1. `setProfileEnvironment` 属性 (その規定値は `true` です) が存在しないことを確認するか、属性の値を明示的に `true` に設定します。

* **ファイル システムをキー リング ストアとして使用する**

  [ファイル システムをキー リング ストアとして使用](xref:security/data-protection/configuration/overview)するようにアプリ コードを調整します。 X509 証明書を使用してキー リングを保護し、その証明書が信頼された証明書であることを確認します。 証明書が自己署名されている場合は、信頼されたルート ストアにその証明書を配置します。

  Web ファームで IIS を使用する場合:

  * すべてのコンピューターがアクセスできるファイル共有を使用します。
  * X509 証明書を各コンピューターに配置します。 [コード内にデータ保護](xref:security/data-protection/configuration/overview)を構成します。

* **コンピューター全体に適用するデータ保護ポリシーを設定する**

  データ保護システムでは、データ保護 API を使用するすべてのアプリに対して、[コンピューター全体に適用する既定のポリシー](xref:security/data-protection/configuration/machine-wide-policy)を設定するためのサポートは限定的です。 詳細については、「<xref:security/data-protection/introduction>」を参照してください。

## <a name="virtual-directories"></a>仮想ディレクトリ

ASP.NET Core アプリでは [IIS 仮想ディレクトリ](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories)はサポートされません。 アプリは[サブアプリケーション](#sub-applications)としてホスティングできます。

## <a name="sub-applications"></a>サブアプリケーション

ASP.NET Core アプリは [IIS サブアプリケーション (サブアプリ)](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications) としてホスティングできます。 サブアプリのパスは、ルート アプリの URL の一部になります。

::: moniker range="< aspnetcore-2.2"

サブアプリに、ハンドラーとして ASP.NET Core モジュールを含めることはできません。 このモジュールをサブアプリの *web.config* ファイルにハンドラーとして追加すると、サブアプリを閲覧しようとすると、エラーのある構成ファイルを参照する *500.19 内部サーバー エラー* が返されます。

次の例は、ASP.NET Core サブアプリの発行済み *web.config* ファイルを示しています。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

ASP.NET Core アプリの下に ASP.NET Core 以外のサブアプリをホスティングする場合は、サブアプリの *web.config* ファイルにある継承されたハンドラーを明示的に削除します。

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <remove name="aspNetCore" />
    </handlers>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

::: moniker-end

サブアプリ内にある静的資産のリンクでは、チルダとスラッシュの表記 (`~/`) を使う必要があります。 チルダとスラッシュの表記により[タグ ヘルパー](xref:mvc/views/tag-helpers/intro)がトリガーされ、作成される相対リンクにサブアプリのパスベースが付加されます。 `/subapp_path` にあるサブアプリの場合、`src="~/image.png"` を使ってリンクされる画像は `src="/subapp_path/image.png"` として作成されます。 ルート アプリの静的ファイル ミドルウェアでは、静的ファイル要求は処理されません。 この要求は、サブアプリの静的ファイル ミドルウェアによって処理されます。

静的資産の `src` 属性が絶対パス (たとえば `src="/image.png"`) に設定されている場合、リンクはサブアプリのパスベースなしで作成されます。 ルート アプリの静的ファイル ミドルウェアでは、ルート アプリの [Web ルート](xref:fundamentals/index#web-root)から資産を提供しようとしますが、ルート アプリから静的資産を利用できる場合を除いて *404 - Not Found* 応答が返されます。

ある ASP.NET Core アプリを別の ASP.NET Core アプリの下でサブアプリとしてホスティングするには:

1. サブアプリ用のアプリ プールを確立します。 デスクトップ CLR (.NET CLR) ではなく、.NET Core 用の Core 共通言語ランタイム (CoreCLR) が起動されてワーカー プロセスでアプリがホストされるため、 **[.NET CLR バージョン]** を **[マネージド コードなし]** に設定します。

1. ルート サイトを IIS マネージャーに追加し、サブアプリをルート サイトの下のフォルダー内に置きます。

1. IIS マネージャーでサブアプリのフォルダーを右クリックし、 **[アプリケーションへの変換]** を選択します。

1. **[アプリケーションの追加]** ダイアログ ボックスで、 **[アプリケーション プール]** に対して **[選択]** ボタンを使い、作成したアプリケーション プールをサブアプリ用に割り当てます。 **[OK]** を選択します。

サブアプリに対して個別のアプリ プールを割り当てることは、インプロセス ホスティング モデルを使用する場合必須となります。

インプロセス ホスティング モデルと ASP.NET Core モジュールの構成について詳しくは、<xref:host-and-deploy/aspnet-core-module> をご覧ください。

## <a name="configuration-of-iis-with-webconfig"></a>web.config による IIS の構成

IIS の構成は ASP.NET Core モジュールを使用した ASP.NET Core アプリで機能する IIS シナリオの *web.config* の `<system.webServer>` セクションによる影響を受けます。 たとえば、IIS の構成は動的な圧縮で機能します。 IIS が動的な圧縮を使用するようにサーバー レベルで構成されている場合、アプリの *web.config* ファイルの `<urlCompression>` 要素を使用すると、それを ASP.NET Core アプリに対して無効にすることができます。

詳細については、次のトピックを参照してください。

* [\<system.webServer> の構成リファレンス](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

分離されたアプリ プール (IIS 10.0 以降でサポートされています) で実行する個別アプリに対して環境変数を設定するには、IIS のリファレンス ドキュメントで、[環境変数\<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) のトピックにある *AppCmd.exe コマンド*のセクションを参照してください。

## <a name="configuration-sections-of-webconfig"></a>web.config の構成のセクション

*web.config* の ASP.NET 4.x アプリの構成セクションは、ASP.NET Core アプリの構成では使用されません。

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

ASP.NET Core アプリは、他の構成プロバイダーを使用して構成されます。 詳細については、[構成](xref:fundamentals/configuration/index)に関するページを参照してください。

## <a name="application-pools"></a>アプリケーション プール

::: moniker range=">= aspnetcore-2.2"

アプリケーション プールの分離は、ホスティング モデルによって決定されます。

* インプロセス ホスティング &ndash; 別のアプリケーション プールでアプリケーションを実行する必要があります。
* アウトプロセス ホスティング &ndash; アプリケーションをそれぞれ専用のアプリケーションプールで実行して、アプリケーションを相互に分離することをお勧めします。

IIS の **[Web サイトの追加]** ダイアログの既定では、アプリケーションごとに 1 つのアプリケーション プールです。 **[サイト名]** を指定すると、入力したテキストが自動的に **[アプリケーション プール]** テキストボックスに設定されます。 サイトが追加されるときに、そのサイト名を使用して新しいアプリ プールが作成されます。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

サーバーで複数の Web サイトをホストする場合は、アプリをそれぞれ専用のアプリ プールで実行して、アプリを相互に分離することをお勧めします。 IIS の **[Web サイトの追加]** ダイアログはこの構成の既定です。 **[サイト名]** を指定すると、入力したテキストが自動的に **[アプリケーション プール]** テキストボックスに設定されます。 サイトが追加されるときに、そのサイト名を使用して新しいアプリ プールが作成されます。

::: moniker-end

## <a name="application-pool-identity"></a>アプリケーション プール ID

アプリ プール ID アカウントを使用すると、ドメインやローカル アカウントを作成して管理する必要なく、一意のアカウントでアプリを実行できます。 IIS 8.0 以降の IIS 管理者ワーカー プロセス (WAS) は、新しいアプリ プールの名前で仮想アカウントを作成し、既定によってアプリ プールのワーカー プロセスをこのアカウントで実行します。 IIS 管理コンソールにあるアプリ プールの **[詳細設定]** で、**ID** が **ApplicationPoolIdentity** を使用するように設定されていることを確認します。

![アプリケーション プールの [詳細設定] ダイアログ](index/_static/apppool-identity.png)

IIS 管理プロセスは、Windows セキュリティ システムでのアプリ プールの名前を使用して、セキュリティで保護された識別子を作成します。 この ID を使用してリソースを保護することができます。 ただし、この ID は実際のユーザー アカウントではなく、Windows ユーザーの管理コンソールに表示されません。

アプリに対する IIS ワーカー プロセスのアクセス許可を昇格させる必要がある場合は、アプリを含むディレクトリのアクセス制御リスト (ACL) を変更します。

1. エクスプローラーを開き、そのディレクトリに移動します。

1. そのディレクトリを右クリックし、 **[プロパティ]** を選択します。

1. **[セキュリティ]** タブの **[編集]** ボタンを選択し、 **[追加]** ボタンをクリックします。

1. **[場所]** ボタンを選択し、システムが選択されていることを確認します。

1. **[選択するオブジェクト名を入力してください]** 領域に、「**IIS AppPool\\<app_pool_name>** 」と入力します。 **[名前の確認]** を選択します。 *DefaultAppPool* で、**IIS AppPool\DefaultAppPool** を使用して名前を確認します。 **[名前の確認]** ボタンが選択されているときには、**DefaultAppPool** の値が、オブジェクト名領域に表示されます。 オブジェクト名の領域に直接アプリケーション プール名を入力することはできません。 オブジェクト名を確認するときには、**IIS AppPool\\<app_pool_name>** の形式を使用します。

   ![アプリ フォルダーのユーザーまたはグループのダイアログを選択します。[名前の確認] を選択する前に、オブジェクト名領域で "DefaultAppPool" のアプリ プール名が "IIS AppPool\" に適用されます。](index/_static/select-users-or-groups-1.png)

1. **[OK]** を選択します。

   ![アプリ フォルダーのユーザーまたはグループのダイアログを選択します。[名前の確認] を選択した後に、オブジェクト名 "DefaultAppPool" がオブジェクト名領域に表示されます。](index/_static/select-users-or-groups-2.png)

1. 読み取り &amp; 実行アクセス許可は、既定で付与される必要があります。 必要に応じて、追加のアクセス許可を提供します。

**ICACLS** ツールを使用してコマンド プロンプトでアクセス許可を付与することもできます。 たとえば、*DefaultAppPool* を使用する場合、次のコマンドを使用します。

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

詳細については、「[icacls](/windows-server/administration/windows-commands/icacls)」のトピックを参照してください。

## <a name="http2-support"></a>HTTP/2 のサポート

::: moniker range=">= aspnetcore-2.2"

[HTTP/2](https://httpwg.org/specs/rfc7540.html) は、次の IIS 展開シナリオでの ASP.NET Core でサポートされます。

* インプロセス
  * Windows Server 2016/Windows 10 以降、IIS 10 以降
  * TLS 1.2 以降の接続
* アウトプロセス
  * Windows Server 2016/Windows 10 以降、IIS 10 以降
  * 一般向けのエッジ サーバーでは HTTP/2 を使用しますが、[Kestrel サーバー](xref:fundamentals/servers/kestrel)へのリバース プロキシ接続では HTTP/1.1 を使用します。
  * TLS 1.2 以降の接続

インプロセスの展開の場合、HTTP/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) によって `HTTP/2` が報告されます。 アウトプロセスの展開の場合、HTTP/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) によって `HTTP/1.1` が報告されます。

インプロセスおよびアウトプロセス ホスティング モデルの詳細については、<xref:host-and-deploy/aspnet-core-module> をご覧ください。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

次の基本要件を満たすアウトプロセスの展開には、[HTTP/2](https://httpwg.org/specs/rfc7540.html) がサポートされています。

* Windows Server 2016/Windows 10 以降、IIS 10 以降
* 一般向けのエッジ サーバーでは HTTP/2 を使用しますが、[Kestrel サーバー](xref:fundamentals/servers/kestrel)へのリバース プロキシ接続では HTTP/1.1 を使用します。
* ターゲット フレームワーク:HTTP/2 接続は IIS によって完全に処理されるため、アウトプロセスの展開には適用できません。
* TLS 1.2 以降の接続

Http/2 接続が確立されると、[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) が `HTTP/1.1` を報告します。

::: moniker-end

HTTP/2 は既定で有効になっています。 HTTP/2 接続が確立されない場合、接続は HTTP/1.1 にフォールバックします。 IIS 展開での HTTP/2 構成の詳細については、[IIS での HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis) に関するページを参照してください。

## <a name="cors-preflight-requests"></a>CORS プレフライト要求

"*このセクションは、.NET Framework をターゲットにした ASP.NET Core アプリにのみ適用されます。* "

.NET Framework をターゲットにした ASP.NET Core アプリの場合、IIS では既定で OPTIONS 要求がアプリに渡されません。 OPTIONS 要求を渡すように *web.config* でアプリの IIS のハンドラーを構成する方法については、[ASP.NET Web API 2 でのクロスオリジン要求の有効化:CORS のしくみ](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)に関する記事をご覧ください。

::: moniker range=">= aspnetcore-2.2"

## <a name="application-initialization-module-and-idle-timeout"></a>Application Initialization モジュールとアイドル タイムアウト

IIS 内で ASP.NET Core モジュール バージョン 2 によってホストされている場合:

* [Application Initialization モジュール](#application-initialization-module) &ndash; アプリのホストされている[インプロセス](#in-process-hosting-model)または[アウトプロセス](#out-of-process-hosting-model)は、ワーカー プロセスの再起動時またはサーバーの再起動時に自動的に起動するように構成できます。
* [アイドル タイムアウト](#idle-timeout) &ndash; アプリのホストされている[インプロセス](#in-process-hosting-model)は、非アクティブ期間中にタイムアウトしないように構成できます。

### <a name="application-initialization-module"></a>Application Initialization モジュール

"*アプリのホストされているインプロセスとアウトプロセスに適用されます。* "

[IIS Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization) は、アプリ プールが開始するときまたはリサイクルされるときに、アプリに HTTP 要求を送信する IIS 機能です。 要求によってアプリの起動がトリガーされます。 既定では、IIS ではアプリのルート URL (`/`) に対して要求が発行され、アプリが初期化されます (構成の詳細については[その他の技術情報](#application-initialization-module-and-idle-timeout-additional-resources)を参照)。

IIS Application Initialization 役割の機能が有効になっていることを確認します。

Windows 7 以降のデスクトップ システム上で、IIS をローカルで使う場合:

1. **[コントロール パネル]** > **[プログラム]** > **[プログラムと機能]** > **[Windows の機能の有効化または無効化]** (画面の左側) に移動します。
1. **[インターネット インフォメーション サービス]** > **[World Wide Web サービス]** > **[アプリケーション開発機能]** を開きます。
1. **[Application Initialization]** のチェック ボックスをオンにします。

Windows Server 2008 R2 以降の場合:

1. **[役割と機能の追加ウィザード]** を開きます。
1. **[役割サービスの選択]** パネルで、 **[アプリケーション開発]** ノードを開きます。
1. **[Application Initialization]** のチェック ボックスをオンにします。

次の方法のいずれかを使って、サイトの Application Initialization モジュールを有効にします。

* IIS マネージャーを使う場合:

  1. **[接続]** パネルの **[アプリケーション プール]** を選択します。
  1. 一覧でアプリのアプリ プールを右クリックして、 **[詳細設定]** を選択します。
  1. 既定の **[開始モード]** は **[オンデマンド]** です。 **[開始モード]** を **[常時実行]** に設定します。 **[OK]** を選択します。
  1. **[接続]** パネルの **[サイト]** ノードを開きます。
  1. アプリを右クリックして、 **[Web サイトの管理]** > **[詳細設定]** の順に選択します。
  1. 既定の **[有効化されたプリロード]** 設定は **[False]** です。 **[有効化されたプリロード]** を **True** に設定します。 **[OK]** を選択します。

* *web.config* を使う場合、`doAppInitAfterRestart` を `true` に設定した `<applicationInitialization>` 要素を、アプリの *web.config* ファイル内の `<system.webServer>` 要素に追加します。

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a>アイドル タイムアウト

"*アプリのホストされているインプロセスにのみ適用されます。* "

アプリがアイドル状態にならないようにするには、IIS マネージャーを使ってアプリ プールのアイドル タイムアウトを設定します。

1. **[接続]** パネルの **[アプリケーション プール]** を選択します。
1. 一覧でアプリのアプリ プールを右クリックして、 **[詳細設定]** を選択します。
1. 既定の **[アイドル状態のタイムアウト (分)]** は **[20]** 分です。 **[アイドル状態のタイムアウト (分)]** を **[0]** (ゼロ) に設定します。 **[OK]** を選択します。
1. ワーカー プロセスをリサイクルします。

アプリのホストされている[アウトプロセス](#out-of-process-hosting-model)がタイムアウトにならないようにするには、次の方法のいずれかを使います。

* 実行させ続けるために、外部サービスからアプリに ping を送信します。
* アプリでバックグラウンド サービスのみをホストしている場合は、IIS ホスティングを回避し、[ASP.NET Core アプリをホストするための Windows サービス](xref:host-and-deploy/windows-service)を使います。

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>Application Initialization モジュールとアイドル タイムアウトに関するその他の技術情報

* [IIS 8.0 Application Initialization](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)
* [Application Initialization \<applicationInitialization>](/iis/configuration/system.webserver/applicationinitialization/)。
* [アプリケーション プールのプロセス モデルの設定 \<processModel>](/iis/configuration/system.applicationhost/applicationpools/add/processmodel)。

::: moniker-end

## <a name="deployment-resources-for-iis-administrators"></a>IIS 管理者用の展開リソース

IIS ドキュメントでの IIS の詳細について説明します。  
[IIS ドキュメント ](/iis)

.NET Core アプリの展開モデルについて説明します。  
[.NET Core アプリケーションの展開](/dotnet/core/deploying/)

構成のガイダンスなど、ASP.NET Core モジュールについて説明します。  
<xref:host-and-deploy/aspnet-core-module>

発行された ASP.NET Core アプリのディレクトリ構造について説明します。  
[ディレクトリの構造](xref:host-and-deploy/directory-structure)

ASP.NET Core アプリに対してアクティブおよび非アクティブな IIS モジュールについて、さらに IIS モジュールの管理方法についてを把握します。  
[IIS モジュール](xref:host-and-deploy/iis/modules)

ASP.NET Core アプリの IIS 展開に関する問題を診断する方法について説明します。  
[トラブルシューティング](xref:test/troubleshoot-azure-iis)

IIS で ASP.NET Core アプリをホストする場合の一般的なエラーを識別します。  
[Azure App Service と IIS の一般的なエラーのリファレンス](xref:host-and-deploy/azure-iis-errors-reference)

## <a name="additional-resources"></a>その他の技術情報

* <xref:test/troubleshoot>
* [ASP.NET Core の概要](xref:index)
* [Microsoft IIS 公式サイト](https://www.iis.net/)
* [Windows Server テクニカル コンテンツ ライブラリ](/windows-server/windows-server)
* [IIS での HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)
* <xref:host-and-deploy/iis/transform-webconfig>
