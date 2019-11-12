---
title: SignalR でサポートされているプラットフォームの ASP.NET Core
author: bradygaster
description: ASP.NET Core SignalR のサポートされているプラットフォームについて説明します。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/01/2019
uid: signalr/supported-platforms
ms.openlocfilehash: 1be7a307710e6e522c0088fd1ca01da11a13eda1
ms.sourcegitcommit: 77c8be22d5e88dd710f42c739748869f198865dd
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/01/2019
ms.locfileid: "73426976"
---
# <a name="aspnet-core-signalr-supported-platforms"></a>SignalR でサポートされているプラットフォームの ASP.NET Core

## <a name="server-system-requirements"></a>サーバーシステムの要件

ASP.NET Core の SignalR は、ASP.NET Core がサポートするすべてのサーバープラットフォームをサポートしています。

## <a name="javascript-client"></a>JavaScript クライアント

[JavaScript クライアント](https://www.npmjs.com/package/@aspnet/signalr)は、nodejs 8 以降のバージョンと次のブラウザーで実行されます。

| ブラウザー                         | Version         |
| ------------------------------- | --------------- |
| Microsoft Edge                  | 現在の&dagger; |
| Mozilla Firefox                 | 現在の&dagger; |
| Google Chrome。Android を含む | 現在の&dagger; |
| SafariiOS を含む            | 現在の&dagger; |
| Microsoft Internet Explorer     | 11              |

*現在*の &dagger;は、ブラウザーの最新バージョンを参照します。

## <a name="net-client"></a>.NET クライアント

[.Net クライアント](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR/)は、ASP.NET Core によってサポートされる任意のプラットフォームで実行されます。 たとえば、xamarin[開発者は、SignalR](https://github.com/aspnet/Announcements/issues/305)を使用して android アプリをビルドするために、xamarin 8.4.0.1 以降と11.14.0.4 を使用した ios アプリを使用できます。

サーバーで IIS が実行されている場合、WebSocket トランスポートでは Windows Server 2012 以降に IIS 8.0 以降が必要です。 その他のトランスポートはすべてのプラットフォームでサポートされています。

## <a name="java-client"></a>Java クライアント

[Java クライアント](https://search.maven.org/artifact/com.microsoft.aspnet/signalr)は、java 8 以降のバージョンをサポートしています。

## <a name="unsupported-clients"></a>サポートされていないクライアント

次のクライアントは使用できますが、試験的または非公式です。 現時点ではサポートされておらず、そうでない場合もあります。

* [C++client](https://github.com/aspnet/SignalR/tree/master/clients/cpp)

* [Swift クライアント](https://github.com/moozzyk/SignalR-Client-Swift)
