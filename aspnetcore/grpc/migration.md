---
title: gRPC サービスの C-core から ASP.NET Core への移行
author: juntaoluo
description: 既存の C コアベースの gRPC アプリを移動して ASP.NET Core スタック上で実行する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/25/2019
uid: grpc/migration
ms.openlocfilehash: c4c07808540c9af370bfa253e8154a8a19f0f3de
ms.sourcegitcommit: 897d4abff58505dae86b2947c5fe3d1b80d927f3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/06/2019
ms.locfileid: "73634063"
---
# <a name="migrating-grpc-services-from-c-core-to-aspnet-core"></a>GRPC サービスの C-core から ASP.NET Core への移行

作成者: [John Luo](https://github.com/juntaoluo)

基になるスタックが実装されているため、 [C コアベースの gRPC](https://grpc.io/blog/grpc-stacks)アプリと ASP.NET Core ベースのアプリでは、すべての機能が同じ方法で動作するわけではありません。 このドキュメントでは、2つのスタック間の移行に関する主な違いについて説明します。

## <a name="grpc-service-implementation-lifetime"></a>gRPC サービスの実装の有効期間

ASP.NET Core スタックでは、既定で gRPC サービスが[スコープ有効期間](xref:fundamentals/dependency-injection#service-lifetimes)で作成されます。 これに対し、gRPC C-core は、既定では、[単一の有効期間](xref:fundamentals/dependency-injection#service-lifetimes)を持つサービスにバインドされます。

スコープ内の有効期間を使用すると、サービス実装は、スコープの有効期間を持つ他のサービスを解決できます。 たとえば、スコープが指定された有効期間は、コンストラクターの挿入によって DI コンテナーから `DbContext` を解決することもできます。 スコープ有効期間の使用:

* サービス実装の新しいインスタンスは、要求ごとに作成されます。
* 実装型のインスタンスメンバーを介して、要求間で状態を共有することはできません。
* 共有状態は、単一のサービスに DI コンテナーに格納することを想定しています。 格納されている共有状態は、gRPC サービス実装のコンストラクターで解決されます。

サービスの有効期間の詳細については、「<xref:fundamentals/dependency-injection#service-lifetimes>」を参照してください。

### <a name="add-a-singleton-service"></a>シングルトンサービスを追加する

gRPC C コア実装から ASP.NET Core への移行を容易にするために、サービス実装のサービスの有効期間をスコープからシングルトンに変更することができます。 これには、サービス実装のインスタンスを DI コンテナーに追加する必要があります。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc();
    services.AddSingleton(new GreeterService());
}
```

ただし、シングルトンの有効期間を持つサービス実装では、コンストラクターの挿入によってスコープ付きサービスを解決できなくなりました。

## <a name="configure-grpc-services-options"></a>GRPC サービスのオプションの構成

C コアベースのアプリでは、`grpc.max_receive_message_length` や `grpc.max_send_message_length` などの設定は、[サーバーインスタンスの構築](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server__ctor_System_Collections_Generic_IEnumerable_Grpc_Core_ChannelOption__)時に `ChannelOption` で構成されます。

ASP.NET Core では、gRPC は `GrpcServiceOptions` の種類を使用して構成を提供します。 たとえば、gRPC サービスの最大受信メッセージサイズは、`AddGrpc` を使用して構成できます。 次の例では、既定の `MaxReceiveMessageSize` を 4 MB から 16 MB に変更します。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddGrpc(options =>
    {
        options.MaxReceiveMessageSize = 16 * 1024 * 1024; // 16 MB
    });
}
```

構成の詳細については、「<xref:grpc/configuration>」を参照してください。

## <a name="logging"></a>ログの記録

C コアベースのアプリは、デバッグの目的で[logger を構成](https://grpc.io/grpc/csharp/api/Grpc.Core.GrpcEnvironment.html?q=size#Grpc_Core_GrpcEnvironment_SetLogger_Grpc_Core_Logging_ILogger_)するために `GrpcEnvironment` に依存しています。 ASP.NET Core スタックは、[ログ API](xref:fundamentals/logging/index)を介してこの機能を提供します。 たとえば、logger は、コンストラクターインジェクションを使用して gRPC サービスに追加できます。

```csharp
public class GreeterService : Greeter.GreeterBase
{
    public GreeterService(ILogger<GreeterService> logger)
    {
    }
}
```

## <a name="https"></a>HTTPS

C コアベースのアプリは、 [Server. Ports プロパティ](https://grpc.io/grpc/csharp/api/Grpc.Core.Server.html#Grpc_Core_Server_Ports)を使用して HTTPS を構成します。 同様の概念は、ASP.NET Core でサーバーを構成するために使用されます。 たとえば、Kestrel は、この機能に[エンドポイント構成](xref:fundamentals/servers/kestrel#endpoint-configuration)を使用します。

## <a name="grpc-interceptors-vs-middleware"></a>gRPC インターセプターとミドルウェア

ASP.NET Core[ミドルウェア](xref:fundamentals/middleware/index)は、C コアベースの gRPC アプリのインターセプターと比較して同様の機能を提供します。 ASP.NET Core ミドルウェアとインターセプターは概念的に似ています。 両方とも：

* は、gRPC 要求を処理するパイプラインを構築するために使用されます。
* パイプライン内の次のコンポーネントの前または後に作業を実行することを許可します。
* `HttpContext`へのアクセスを提供します。
  * ミドルウェアでは、`HttpContext` はパラメーターです。
  * インターセプターでは、`ServerCallContext.GetHttpContext` 拡張メソッドで `ServerCallContext` パラメーターを使用して `HttpContext` にアクセスできます。 この機能は、ASP.NET Core で実行されているインターセプターに固有の機能であることに注意してください。

gRPC インターセプターと ASP.NET Core ミドルウェアの違い:

* インターセプター
  * [ServerCallContext](https://grpc.io/grpc/csharp/api/Grpc.Core.ServerCallContext.html)を使用して、抽象化の gRPC レイヤーを操作します。
  * 次へのアクセスを提供します。
    * 呼び出しに送信された逆シリアル化されたメッセージ。
    * シリアル化される前に、呼び出しから返されるメッセージ。
* ミドルウェア
  * gRPC インターセプターの前に実行されます。
  * は、基になる HTTP/2 メッセージを操作します。
  * は、要求ストリームと応答ストリームからのみバイトにアクセスできます。

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:tutorials/grpc/grpc-start>
