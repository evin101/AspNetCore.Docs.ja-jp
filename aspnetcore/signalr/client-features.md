---
title: SignalR クライアントの機能
author: bradygaster
description: さまざまな ASP.NET Core SignalR クライアントでサポートされている機能について説明します。
ms.author: bradyg
ms.custom: mvc
ms.date: 09/18/2019
uid: signalr/client-features
ms.openlocfilehash: 6718722cdbcfae500026fcd429ca6b9de8f0f103
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2019
ms.locfileid: "71925351"
---
# <a name="aspnet-core-signalr-client-features"></a>SignalR クライアント機能の ASP.NET Core

## <a name="feature-distribution"></a>機能の配布

次の表は、リアルタイムサポートを提供するクライアントの機能とサポートを示しています。 各機能について、この機能をサポートする*最小*バージョンが一覧表示されます。 バージョンが表示されない場合、この機能はサポートされていません。

| 機能 | .NET | JavaScript | Java |
| ---- | :-: | :-: | :-: |
| Azure SignalR サービスのサポート |1.0.0|1.0.0|1.0.0|
| [サーバーからクライアントへのストリーミング](xref:signalr/streaming)          |1.0.0|1.0.0|1.0.0|
| [クライアントとサーバー間のストリーミング](xref:signalr/streaming)          |3.0.0|3.0.0|3.0.0|
| 自動再接続 ([.net](/aspnet/core/signalr/dotnet-client?view=aspnetcore-3.0&tabs=visual-studio#handle-lost-connection)、 [JavaScript](/aspnet/core/signalr/javascript-client?view=aspnetcore-3.0#reconnect-clients))          |3.0.0|3.0.0|❌|
| WebSocket トランスポート |1.0.0|1.0.0|1.0.0|
| Server-Sent Events トランスポート |1.0.0|1.0.0|❌|
| ロングポーリングトランスポート |1.0.0|1.0.0|3.0.0|
| JSON ハブプロトコル |1.0.0|1.0.0|1.0.0|
| MessagePack ハブ プロトコル |1.0.0|1.0.0|❌|

Java クライアントでの自動再接続のサポートは[、microsoft の問題追跡ツール](https://github.com/aspnet/AspNetCore/issues/8711)で追跡されます。

## <a name="additional-resources"></a>その他の技術情報

* [ASP.NET Core の SignalR 概要](xref:tutorials/signalr)
* [サポートされているプラットフォーム](xref:signalr/supported-platforms)
* [ハブ](xref:signalr/hubs)
* [JavaScript クライアント](xref:signalr/javascript-client)
