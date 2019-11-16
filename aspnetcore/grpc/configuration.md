---
title: gRPC for .NET 構成
author: jamesnk
description: .NET アプリ用に gRPC を構成する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 09/05/2019
uid: grpc/configuration
ms.openlocfilehash: 3ef92f10d914ef9fa3e13a7bdd5c863bab297f57
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2019
ms.locfileid: "71925206"
---
# <a name="grpc-for-net-configuration"></a>gRPC for .NET 構成

## <a name="configure-services-options"></a>サービスオプションの構成

gRPC サービスは`AddGrpc` *Startup.cs*ので構成されます。 次の表では、gRPC サービスを構成するためのオプションについて説明します。

| OPTION | 既定値 | 説明 |
| ------ | ------------- | ----------- |
| `MaxSendMessageSize` | `null` | サーバーから送信できる最大メッセージサイズ (バイト単位)。 構成された最大メッセージサイズを超えるメッセージを送信しようとすると、例外が発生します。 |
| `MaxReceiveMessageSize` | 4 MB | サーバーが受信できる最大メッセージサイズ (バイト単位)。 サーバーがこの制限を超えるメッセージを受信すると、例外がスローされます。 この値を大きくすると、サーバーはより大きなメッセージを受け取ることができますが、メモリの消費に悪影響を与える可能性があります。 |
| `EnableDetailedErrors` | `false` | の`true`場合、サービスメソッドで例外がスローされると、詳細な例外メッセージがクライアントに返されます。 既定値は `false` です。 を`EnableDetailedErrors`に`true`設定すると、機密情報が漏洩する可能性があります。 |
| `CompressionProviders` | gzip | メッセージの圧縮と圧縮解除に使用される圧縮プロバイダーのコレクション。 カスタム圧縮プロバイダーを作成してコレクションに追加することができます。 既定で構成されているプロバイダーは、 **gzip**圧縮をサポートしています。 |
| `ResponseCompressionAlgorithm` | `null` | サーバーから送信されたメッセージの圧縮に使用される圧縮アルゴリズム。 アルゴリズムは、の`CompressionProviders`圧縮プロバイダーと一致している必要があります。 アルゴリズムで応答を圧縮するには、クライアントは、 **grpc-accept-encoding**ヘッダーでアルゴリズムを送信することによって、アルゴリズムをサポートしていることを示す必要があります。 |
| `ResponseCompressionLevel` | `null` | サーバーから送信されたメッセージの圧縮に使用される圧縮レベルです。 |

`AddGrpc` の`Startup.ConfigureServices`呼び出しにオプションデリゲートを指定することで、すべてのサービスに対してオプションを構成できます。

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup.cs?name=snippet)]

1つのサービスのオプションは、で`AddGrpc`提供されるグローバルオプションを上書きします。また、次のものを使用し`AddServiceOptions<TService>`て構成できます。

[!code-csharp[](~/grpc/configuration/sample/GrcpService/Startup2.cs?name=snippet)]

## <a name="configure-client-options"></a>クライアントオプションを構成する

gRPC クライアントの構成がオン`GrpcChannelOptions`になっています。 次の表では、gRPC チャネルを構成するためのオプションについて説明します。

| OPTION | 既定値 | 説明 |
| ------ | ------------- | ----------- |
| `HttpClient` | 新しいインスタンス | gRPC 呼び出しを行うために使用する。`HttpClient` クライアントを設定してカスタム`HttpClientHandler`を構成することも、gRPC 呼び出し用に HTTP パイプラインにハンドラーを追加することもできます。 が指定されていない場合`HttpClient`は、チャネルに対して新しいインスタンスが作成されます。 `HttpClient` 自動的に破棄されます。 |
| `DisposeHttpClient` | `false` | `true` `GrpcChannel` と`HttpClient`が指定されている場合は、が破棄されると、インスタンスが破棄されます。 `HttpClient` |
| `LoggerFactory` | `null` | gRPC 呼び出しに関する情報をログに記録するためにクライアントによって使用される。`LoggerFactory` インスタンス`LoggerFactory`は、依存関係の挿入から解決すること`LoggerFactory.Create`も、を使用して作成することもできます。 ログ記録を構成する例に<xref:grpc/diagnostics#grpc-client-logging>ついては、「」を参照してください。 |
| `MaxSendMessageSize` | `null` | クライアントから送信できる最大メッセージサイズ (バイト単位)。 構成された最大メッセージサイズを超えるメッセージを送信しようとすると、例外が発生します。 |
| `MaxReceiveMessageSize` | 4 MB | クライアントが受信できる最大メッセージサイズ (バイト単位)。 クライアントがこの制限を超えるメッセージを受信すると、例外がスローされます。 この値を大きくすると、クライアントはより大きなメッセージを受け取ることができますが、メモリの消費に悪影響を及ぼす可能性があります。 |
| `Credentials` | `null` | `ChannelCredentials` のインスタンス。 資格情報は、認証メタデータを gRPC 呼び出しに追加するために使用されます。 |
| `CompressionProviders` | gzip | メッセージの圧縮と圧縮解除に使用される圧縮プロバイダーのコレクション。 カスタム圧縮プロバイダーを作成してコレクションに追加することができます。 既定で構成されているプロバイダーは、 **gzip**圧縮をサポートしています。 |

コード例を次に示します。

* チャネルの送信メッセージと受信メッセージの最大サイズを設定します。
* クライアントを作成します。

[!code-csharp[](~/grpc/configuration/sample/Program.cs?name=snippet&highlight=3-8)]

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/diagnostics>
* <xref:tutorials/grpc/grpc-start>
