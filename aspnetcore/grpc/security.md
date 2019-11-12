---
title: GRPC での ASP.NET Core のセキュリティに関する考慮事項
author: jamesnk
description: GRPC for ASP.NET Core のセキュリティに関する考慮事項について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/07/2019
uid: grpc/security
ms.openlocfilehash: f84bec0ef485b701b2be36384a2e49b9b28e473d
ms.sourcegitcommit: 8b36f75b8931ae3f656e2a8e63572080adc78513
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70310367"
---
# <a name="security-considerations-in-grpc-for-aspnet-core"></a>GRPC での ASP.NET Core のセキュリティに関する考慮事項

作成者: [James Newton-King](https://twitter.com/jamesnk)

この記事では、.NET Core を使用して gRPC をセキュリティで保護する方法について説明します。

## <a name="transport-security"></a>トランスポートセキュリティ

gRPC メッセージは、HTTP/2 を使用して送受信されます。 次のことをお勧めします。

* [トランスポート層セキュリティ (TLS)](https://tools.ietf.org/html/rfc5246)を使用して、運用環境の grpc アプリでメッセージをセキュリティで保護します。
* gRPC サービスは、セキュリティで保護されたポートをリッスンし、応答するだけです。

TLS は Kestrel で構成されます。 Kestrel エンドポイントの構成の詳細については、「 [kestrel エンドポイントの構成](xref:fundamentals/servers/kestrel#endpoint-configuration)」を参照してください。

## <a name="exceptions"></a>例外

例外メッセージは、通常、クライアントに公開されない機密データと見なされます。 既定では、gRPC は gRPC サービスによってスローされた例外の詳細をクライアントに送信しません。 代わりに、エラーが発生したことを示す一般的なメッセージをクライアントが受信します。 クライアントへの例外メッセージの配信は、 [EnableDetailedErrors](xref:grpc/configuration#configure-services-options)を使用して (開発やテストなどで) オーバーライドできます。 例外メッセージを実稼働アプリでクライアントに公開することはできません。

## <a name="message-size-limits"></a>メッセージサイズの制限

GRPC クライアントおよびサービスへの着信メッセージは、メモリに読み込まれます。 メッセージサイズの制限は、gRPC が過剰なリソースを消費しないようにするためのメカニズムです。

gRPC では、メッセージごとのサイズ制限を使用して、受信メッセージと送信メッセージを管理します。 既定では、gRPC は受信メッセージを 4 MB に制限します。 送信メッセージに制限はありません。

サーバーでは、次のように`AddGrpc`して、アプリケーションのすべてのサービスに対して grpc メッセージ制限を構成できます。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options =>
    {
        options.MaxReceiveMessageSize = 1 * 1024 * 1024; // 1 MB
        options.MaxSendMessageSize = 1 * 1024 * 1024; // 1 MB
    });
}
```

また、を使用し`AddServiceOptions<TService>`て、個々のサービスに対して制限を構成することもできます。 メッセージサイズの制限の構成の詳細については、「 [Grpc の構成](xref:grpc/configuration)」を参照してください。

## <a name="client-certificate-validation"></a>クライアント証明書の検証

[クライアント証明書](https://tools.ietf.org/html/rfc5246#section-7.4.4)は、接続が確立されると最初に検証されます。 既定では、Kestrel は接続のクライアント証明書の追加の検証を実行しません。

クライアント証明書によって保護されている gRPC サービスでは、 [AspNetCore](xref:security/authentication/certauth)パッケージを使用することをお勧めします。 証明書認証 ASP.NET Core は、次のようなクライアント証明書に対して追加の検証を実行します。

* 証明書に有効な拡張キー使用法 (EKU) がある
* 有効期間内
* 証明書の失効を確認する
