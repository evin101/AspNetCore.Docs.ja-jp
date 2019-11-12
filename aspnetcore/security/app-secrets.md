---
title: ASP.NET Core での開発におけるアプリシークレットの安全な保存
author: rick-anderson
description: ASP.NET Core アプリの開発中に、機密情報をアプリシークレットとして格納および取得する方法について説明します。
ms.author: scaddie
ms.custom: mvc
ms.date: 03/13/2019
uid: security/app-secrets
ms.openlocfilehash: 0203a5737caf1af809b739d9e266a6971cd1523b
ms.sourcegitcommit: 215954a638d24124f791024c66fd4fb9109fd380
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71080710"
---
# <a name="safe-storage-of-app-secrets-in-development-in-aspnet-core"></a>ASP.NET Core での開発におけるアプリシークレットの安全な保存

[Rick Anderson](https://twitter.com/RickAndMSFT)、 [Daniel Roth](https://github.com/danroth27)、 [Scott Addie](https://github.com/scottaddie)

[サンプル コードを表示またはダウンロード](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/security/app-secrets/samples)します ([ダウンロード方法](xref:index#how-to-download-a-sample))。

このドキュメントでは、ASP.NET Core アプリの開発時に機密データを格納および取得する方法について説明します。 パスワードやその他の機密データをソースコードに格納しないでください。 運用環境のシークレットは、開発またはテストには使用しないでください。 [Azure Key Vault 構成プロバイダー](xref:security/key-vault-configuration)により、Azure テストと運用のシークレットを格納し、保護することが可能です。

## <a name="environment-variables"></a>環境変数

環境変数は、コードまたはローカル構成ファイルにアプリシークレットを格納しないようにするために使用されます。 環境変数は、以前に指定したすべての構成ソースの構成値を上書きします。

::: moniker range="<= aspnetcore-1.1"

コンストラクターでを呼び出し<xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*>て、環境変数の値の読み取りを構成します。 `Startup`

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=8)]

::: moniker-end

**個々のユーザーアカウント**のセキュリティが有効になっている ASP.NET Core web アプリを考えてみましょう。 既定のデータベース接続文字列は、というキー `DefaultConnection`を持つプロジェクトの*appsettings*ファイルに含まれています。 既定の接続文字列は LocalDB 用であり、ユーザーモードで実行され、パスワードは必要ありません。 アプリの展開時に`DefaultConnection` 、キー値は環境変数の値でオーバーライドできます。 環境変数には、機密性の高い資格情報を持つ完全な接続文字列が格納される場合があります。

> [!WARNING]
> 環境変数は通常、暗号化されていないプレーンテキストで格納されます。 コンピューターまたはプロセスが侵害された場合、信頼されていない相手から環境変数にアクセスできます。 ユーザーシークレットが漏えいしないようにするための追加の手段が必要な場合があります。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

## <a name="secret-manager"></a>シークレットマネージャー

Secret Manager ツールは、ASP.NET Core プロジェクトの開発中に機微なデータを保存します。 このコンテキストでは、機密データの一部がアプリシークレットになります。 アプリシークレットは、プロジェクトツリーとは別の場所に格納されます。 アプリシークレットは、特定のプロジェクトに関連付けられているか、複数のプロジェクト間で共有されています。 アプリシークレットがソース管理にチェックインされていません。

> [!WARNING]
> シークレットマネージャーツールは、保存されているシークレットを暗号化せず、信頼されたストアとして扱わないでください。 開発目的でのみ使用されます。 キーと値は、ユーザープロファイルディレクトリの JSON 構成ファイルに格納されます。

## <a name="how-the-secret-manager-tool-works"></a>Secret Manager ツールの動作

Secret Manager ツールは、値の格納場所や方法などの実装の詳細を抽象化します。 このツールは、実装の詳細を把握していなくても使用できます。 値は、ローカルコンピューター上のシステムで保護されたユーザープロファイルフォルダー内の JSON 構成ファイルに格納されます。

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

ファイルシステムのパス:

`%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`

# <a name="linux--macostablinuxmacos"></a>[Linux/macOS](#tab/linux+macos)

ファイルシステムのパス:

`~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

---

上記のファイルパスで、を`<user_secrets_id>` *.csproj*ファイル`UserSecretsId`で指定された値に置き換えます。

シークレットマネージャーツールで保存したデータの場所または形式に依存するコードは記述しないでください。 これらの実装の詳細は変更される可能性があります。 たとえば、シークレット値は暗号化されませんが、将来の可能性があります。

::: moniker range="<= aspnetcore-2.0"

## <a name="install-the-secret-manager-tool"></a>シークレットマネージャーツールをインストールする

Secret Manager ツールは、.NET Core SDK 2.1.300 以降の .NET Core CLI にバンドルされています。 2\.1.300 の前にバージョンを .NET Core SDK には、ツールのインストールが必要です。

> [!TIP]
> コマンド`dotnet --version`シェルからを実行して、インストールされている .NET Core SDK のバージョン番号を確認します。

使用されている .NET Core SDK にツールが含まれている場合は、次のような警告が表示されます。

```console
The tool 'Microsoft.Extensions.SecretManager.Tools' is now included in the .NET Core SDK. Information on resolving this warning is available at (https://aka.ms/dotnetclitools-in-box).
```

ASP.NET Core プロジェクトに[SecretManager](https://www.nuget.org/packages/Microsoft.Extensions.SecretManager.Tools/) NuGet パッケージをインストールします。 例:

[!code-xml[](app-secrets/samples/1.x/UserSecrets/UserSecrets.csproj?name=snippet_CsprojFile&highlight=15-16)]

コマンドシェルで次のコマンドを実行して、ツールのインストールを検証します。

```dotnetcli
dotnet user-secrets -h
```

Secret Manager ツールには、使用法、オプション、およびコマンドヘルプのサンプルが表示されます。

```console
Usage: dotnet user-secrets [options] [command]

Options:
  -?|-h|--help                        Show help information
  --version                           Show version information
  -v|--verbose                        Show verbose output
  -p|--project <PROJECT>              Path to project. Defaults to searching the current directory.
  -c|--configuration <CONFIGURATION>  The project configuration to use. Defaults to 'Debug'.
  --id                                The user secret ID to use.

Commands:
  clear   Deletes all the application secrets
  list    Lists all the application secrets
  remove  Removes the specified user secret
  set     Sets the user secret to the specified value

Use "dotnet user-secrets [command] --help" for more information about a command.
```

> [!NOTE]
> *.Csproj*ファイルの`DotNetCliToolReference`要素で定義されているツールを実行するには、 *.csproj*ファイルと同じディレクトリに存在する必要があります。

::: moniker-end

## <a name="enable-secret-storage"></a>シークレットストレージを有効にする

Secret Manager ツールは、ユーザープロファイルに格納されているプロジェクト固有の構成設定を操作します。

::: moniker range=">= aspnetcore-3.0"

Secret Manager ツールには .NET Core SDK `init` 3.0.100 以降のコマンドが含まれています。 ユーザーシークレットを使用するには、プロジェクトディレクトリで次のコマンドを実行します。

```dotnetcli
dotnet user-secrets init
```

上のコマンドは、 `UserSecretsId` *.csproj*ファイルの`PropertyGroup`内に要素を追加します。 既定では、の`UserSecretsId`内部テキストは GUID です。 内部テキストは任意ですが、プロジェクトに固有です。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

ユーザーシークレットを使用するには`UserSecretsId` 、 *.csproj*ファイル`PropertyGroup`の内に要素を定義します。 の`UserSecretsId`内部テキストは任意ですが、プロジェクトに固有です。 通常、 `UserSecretsId`開発者はの GUID を生成します。

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](app-secrets/samples/2.x/UserSecrets/UserSecrets.csproj?name=snippet_PropertyGroup&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](app-secrets/samples/1.x/UserSecrets/UserSecrets.csproj?name=snippet_PropertyGroup&highlight=3)]

::: moniker-end

> [!TIP]
> Visual Studio で、ソリューションエクスプローラーでプロジェクトを右クリックし、コンテキストメニューから **[ユーザーシークレットの管理]** を選択します。 このジェスチャは、 `UserSecretsId` GUID が設定された要素を .csproj ファイルに追加し*ます*。

## <a name="set-a-secret"></a>シークレットを設定する

キーとその値で構成されるアプリシークレットを定義します。 シークレットは、プロジェクトの`UserSecretsId`値に関連付けられます。 たとえば、 *.csproj*ファイルが存在するディレクトリから次のコマンドを実行します。

```dotnetcli
dotnet user-secrets set "Movies:ServiceApiKey" "12345"
```

前の例では、コロンは、 `Movies`が`ServiceApiKey`プロパティを持つオブジェクトリテラルであることを示しています。

シークレットマネージャーツールは、他のディレクトリからも使用できます。 .Csproj ファイル`--project`が存在するファイルシステムパスを指定するには、オプションを使用します。 例:

```dotnetcli
dotnet user-secrets set "Movies:ServiceApiKey" "12345" --project "C:\apps\WebApp1\src\WebApp1"
```

### <a name="json-structure-flattening-in-visual-studio"></a>Visual Studio での JSON 構造のフラット化

Visual Studio の **[ユーザーシークレットの管理]** ジェスチャは、テキストエディターで*シークレットの json*ファイルを開きます。 *シークレット*の内容を、格納されるキーと値のペアで置き換えます。 例:

```json
{
  "Movies": {
    "ConnectionString": "Server=(localdb)\\mssqllocaldb;Database=Movie-1;Trusted_Connection=True;MultipleActiveResultSets=true",
    "ServiceApiKey": "12345"
  }
}
```

JSON 構造体は、または`dotnet user-secrets remove` `dotnet user-secrets set`を使用した変更後にフラット化されます。 たとえば、を実行`dotnet user-secrets remove "Movies:ConnectionString"`すると`Movies` 、オブジェクトリテラルが折りたたまれます。 変更後のファイルは次のようになります。

```json
{
  "Movies:ServiceApiKey": "12345"
}
```

## <a name="set-multiple-secrets"></a>複数のシークレットを設定する

シークレットのバッチは、JSON を`set`コマンドにパイプすることによって設定できます。 次の例では、*入力した json*ファイルの内容を`set`コマンドにパイプ処理します。

# <a name="windowstabwindows"></a>[Windows](#tab/windows)

コマンドシェルを開き、次のコマンドを実行します。

  ```dotnetcli
  type .\input.json | dotnet user-secrets set
  ```

# <a name="linux--macostablinuxmacos"></a>[Linux/macOS](#tab/linux+macos)

コマンドシェルを開き、次のコマンドを実行します。

  ```dotnetcli
  cat ./input.json | dotnet user-secrets set
  ```

---

## <a name="access-a-secret"></a>シークレットにアクセスします。

[ASP.NET Core 構成 API](xref:fundamentals/configuration/index)は、シークレットマネージャーのシークレットへのアクセスを提供します。

::: moniker range=">= aspnetcore-2.0 <= aspnetcore-2.2"

プロジェクトが .NET Framework 対象である場合は、 [Microsoft. Extensions. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) NuGet パッケージをインストールします。

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

ASP.NET Core 2.0 以降では、プロジェクトがを呼び出し<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>て、事前に構成された既定値でホストの新しいインスタンスを初期化すると、開発モードでユーザーシークレットの構成ソースが自動的に追加されます。 `CreateDefaultBuilder`が<xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*>の場合<xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.EnvironmentName> 、 <xref:Microsoft.AspNetCore.Hosting.EnvironmentName.Development>を呼び出します。

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Program.cs?name=snippet_CreateWebHostBuilder&highlight=2)]

が`CreateDefaultBuilder`呼び出されない場合は、 `Startup`コンストラクターでを呼び出す<xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*>ことによって、ユーザーシークレット構成ソースを明示的に追加します。 次`AddUserSecrets`の例に示すように、アプリが開発環境で実行されている場合にのみ、を呼び出します。

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=12)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[Microsoft. extension. Configuration. UserSecrets](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.UserSecrets) NuGet パッケージをインストールします。

<xref:Microsoft.Extensions.Configuration.UserSecretsConfigurationExtensions.AddUserSecrets*>コンストラクターで、の呼び出しを使用してユーザーシークレットの構成ソースを追加します。 `Startup`

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupConstructor&highlight=12)]

::: moniker-end

ユーザーシークレットは API を使用し`Configuration`て取得できます。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup.cs?name=snippet_StartupClass&highlight=14)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup.cs?name=snippet_StartupClass&highlight=26)]

::: moniker-end

## <a name="map-secrets-to-a-poco"></a>シークレットを POCO にマップする

オブジェクトリテラル全体を POCO (プロパティを持つ単純な .NET クラス) にマップすると、関連するプロパティを集計するのに役立ちます。

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

前のシークレットを POCO にマップするには、 `Configuration` API の[オブジェクトグラフバインド](xref:fundamentals/configuration/index#bind-to-an-object-graph)機能を使用します。 次のコードは、カスタム`MovieSettings` POCO にバインドし、 `ServiceApiKey`プロパティ値にアクセスします。

::: moniker range=">= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup3.cs?name=snippet_BindToObjectGraph)]

::: moniker-end

::: moniker range="= aspnetcore-1.0"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup3.cs?name=snippet_BindToObjectGraph)]

::: moniker-end

`MovieSettings`と`Movies:ConnectionString` シークレットは、のそれぞれのプロパティにマップされます。`Movies:ServiceApiKey`

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Models/MovieSettings.cs?name=snippet_MovieSettingsClass)]

## <a name="string-replacement-with-secrets"></a>シークレットを使用した文字列の置換

プレーンテキストでのパスワードの保存は安全ではありません。 たとえば、 *appsettings*に格納されているデータベース接続文字列には、指定されたユーザーのパスワードを含めることができます。

[!code-json[](app-secrets/samples/2.x/UserSecrets/appsettings-unsecure.json?highlight=3)]

より安全な方法は、パスワードをシークレットとして保存することです。 例えば:

```dotnetcli
dotnet user-secrets set "DbPassword" "pass123"
```

Appsettings の`Password`接続文字列からキーと値のペアを削除します。 例:

[!code-json[](app-secrets/samples/2.x/UserSecrets/appsettings.json?highlight=3)]

シークレットの値は、 <xref:System.Data.SqlClient.SqlConnectionStringBuilder>オブジェクトの<xref:System.Data.SqlClient.SqlConnectionStringBuilder.Password*>プロパティに設定して、接続文字列を完成させることができます。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](app-secrets/samples/2.x/UserSecrets/Startup2.cs?name=snippet_StartupClass&highlight=14-17)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](app-secrets/samples/1.x/UserSecrets/Startup2.cs?name=snippet_StartupClass&highlight=26-29)]

::: moniker-end

## <a name="list-the-secrets"></a>シークレットを一覧表示する

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

*.Csproj*ファイルが存在するディレクトリから、次のコマンドを実行します。

```dotnetcli
dotnet user-secrets list
```

次の出力が表示されます。

```console
Movies:ConnectionString = Server=(localdb)\mssqllocaldb;Database=Movie-1;Trusted_Connection=True;MultipleActiveResultSets=true
Movies:ServiceApiKey = 12345
```

前の例では、キー名のコロンは、 *json*内のオブジェクト階層を表しています。

## <a name="remove-a-single-secret"></a>1つのシークレットを削除する

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

*.Csproj*ファイルが存在するディレクトリから、次のコマンドを実行します。

```dotnetcli
dotnet user-secrets remove "Movies:ConnectionString"
```

アプリの*シークレットの json*ファイルは、 `MoviesConnectionString`キーに関連付けられているキーと値のペアを削除するように変更されました。

```json
{
  "Movies": {
    "ServiceApiKey": "12345"
  }
}
```

を`dotnet user-secrets list`実行すると、次のメッセージが表示されます。

```console
Movies:ServiceApiKey = 12345
```

## <a name="remove-all-secrets"></a>すべてのシークレットを削除する

[!INCLUDE[secrets.json file](~/includes/app-secrets/secrets-json-file-and-text.md)]

*.Csproj*ファイルが存在するディレクトリから、次のコマンドを実行します。

```dotnetcli
dotnet user-secrets clear
```

アプリのすべてのユーザーシークレットが、*シークレットの json*ファイルから削除されています。

```json
{}
```

を`dotnet user-secrets list`実行すると、次のメッセージが表示されます。

```console
No secrets configured for this application.
```

## <a name="additional-resources"></a>その他の技術情報

* <xref:fundamentals/configuration/index>
* <xref:security/key-vault-configuration>
