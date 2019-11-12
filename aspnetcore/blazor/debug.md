---
title: デバッグ ASP.NET Core Blazor
author: guardrex
description: Blazor アプリをデバッグする方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/15/2019
uid: blazor/debug
ms.openlocfilehash: 9fc3f1d2dd7dc79d2ba3d64bff6e0f92ac2cf6dc
ms.sourcegitcommit: 35a86ce48041caaf6396b1e88b0472578ba24483
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2019
ms.locfileid: "72391183"
---
# <a name="debug-aspnet-core-blazor"></a>デバッグ ASP.NET Core Blazor

作成者: [Daniel Roth](https://github.com/danroth27)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

Chrome で Blazor Webassembly アプリをデバッグするための*早期*サポートが存在します。

デバッガーの機能は制限されています。 次のようなシナリオがあります。

* ブレークポイントを設定および削除します。
* コードを実行するか、(`F8`) コードの実行を再開 (@no__t) します。
* *[ローカル] 画面で*、型 `int`、`string`、`bool` のすべてのローカル変数の値を確認します。
* 呼び出し履歴を参照してください。これには、JavaScript から .NET への呼び出しチェーンや、.NET から JavaScript までの呼び出し履歴が含まれます。

次のこと*はできません*。

* @No__t-0、`string`、または `bool` ではないすべてのローカルの値を確認します。
* クラスのプロパティまたはフィールドの値を確認します。
* 変数の上にマウスポインターを合わせると、値が表示されます。
* コンソールで式を評価します。
* 非同期呼び出し間でステップ実行します。
* その他の一般的なデバッグシナリオを実行します。

さらにデバッグを行うシナリオの開発は、エンジニアリングチームにとって重視されています。

## <a name="prerequisites"></a>必要条件

デバッグには、次のいずれかのブラウザーが必要です。

* Google Chrome (バージョン70以降)
* Microsoft Edge preview ([エッジ開発チャネル](https://www.microsoftedgeinsider.com))

## <a name="procedure"></a>プロシージャ

1. @No__t 0 構成で Blazor Webasアプリを実行します。 @No__t-0 オプションを[dotnet run](/dotnet/core/tools/dotnet-run)コマンドに渡します。 `dotnet run --configuration Debug`.
1. ブラウザーでアプリにアクセスします。
1. 開発者ツールパネルではなく、アプリにキーボードフォーカスを置きます。 [開発者ツール] パネルは、デバッグを開始したときに閉じることができます。
1. 次の Blazor に固有のキーボードショートカットを選択します。
   * `Shift+Alt+D` (Windows/Linux)
   * macOS の `Shift+Cmd+D`
1. 画面に表示されている手順に従って、リモートデバッグが有効になっているブラウザーを再起動します。
1. 次の Blazor に固有のキーボードショートカットをもう一度選択して、デバッグセッションを開始します。
   * `Shift+Alt+D` (Windows/Linux)
   * macOS の `Shift+Cmd+D`

## <a name="enable-remote-debugging"></a>リモートデバッグを有効にする

リモートデバッグが無効になっている場合は、[デバッグ可能な**ブラウザー] タブ**のエラーページが Chrome によって生成されます。 エラーページには、デバッグポートを開いた状態で Chrome を実行し、Blazor デバッグプロキシがアプリに接続できるようにするための手順が含まれています。 *Chrome インスタンスをすべて閉じ*、指示に従って chrome を再起動します。

## <a name="debug-the-app"></a>アプリをデバッグする

リモートデバッグが有効になっている状態で Chrome を実行すると、デバッグ キーボードショートカットによって新しい デバッガー タブが開きます。しばらくすると、**ソース** タブにアプリ内の .net アセンブリの一覧が表示されます。 各アセンブリを展開し、デバッグに使用できる *.cs*/ *. razor*ソースファイルを見つけます。 ブレークポイントを設定し、アプリのタブに戻ります。ブレークポイントは、コードの実行時にヒットします。 ブレークポイントにヒットした後、コードを実行するか (`F8`) コードを正常に実行するかを、1ステップ (`F10`) にします。

Blazor は、 [Chrome DevTools プロトコル](https://chromedevtools.github.io/devtools-protocol/)を実装し、でプロトコルを補強するデバッグプロキシを提供します。NET 固有の情報。 デバッグキーボードショートカットを押すと、Blazor は、プロキシで Chrome DevTools をポイントします。 プロキシは、デバッグしようとしているブラウザーウィンドウに接続します (したがって、リモートデバッグを有効にする必要があります)。

## <a name="browser-source-maps"></a>ブラウザーソースマップ

ブラウザーソースマップを使用すると、ブラウザーはコンパイルされたファイルを元のソースファイルにマップでき、クライアント側のデバッグによく使用されます。 ただし、Blazor は現在、 C# JavaScript/wasm に直接マップされていません。 代わりに、Blazor はブラウザー内で IL を解釈するため、ソースマップは関係しません。

## <a name="troubleshooting-tip"></a>トラブルシューティングのヒント

エラーが発生した場合は、次のヒントを参考にしてください。

**[デバッガー]** タブで、ブラウザーで開発者ツールを開きます。 コンソールで `localStorage.clear()` を実行して、ブレークポイントを削除します。
