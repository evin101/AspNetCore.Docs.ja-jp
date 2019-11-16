---
title: C# を使用した gRPC サービス
author: juntaoluo
description: で gRPC サービスをC#作成する際の基本的な概念について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 07/03/2019
uid: grpc/basics
ms.openlocfilehash: 8d99d036fd4b00fc4568e67ea5225dc006dea4b1
ms.sourcegitcommit: 73e255e846e414821b8cc20ffa3aec946735cd4e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/03/2019
ms.locfileid: "71925183"
---
# <a name="grpc-services-with-c"></a>gRPC サービスと C\#

このドキュメントでは、でC# [gPRC](https://grpc.io/docs/guides/)アプリを作成するために必要な概念の概要を説明します。 ここで説明するトピックは、 [C コア](https://grpc.io/blog/grpc-stacks)ベースと ASP.NET Core ベースの gRPC アプリの両方に適用されます。

## <a name="proto-file"></a>プロトコルファイル

gRPC では、API 開発に対してコントラクト優先のアプローチが使われます。 プロトコルバッファー (protobuf) は、既定ではインターフェイスデザイン言語 (IDL) として使用されます。 *@No__t の-1*ファイルには次のものが含まれます。

* gRPC サービスの定義。
* クライアントとサーバーの間で送信されるメッセージ。

Protobuf ファイルの構文の詳細については、[公式のドキュメント (protobuf)](https://developers.google.com/protocol-buffers/docs/proto3)を参照してください。

たとえば、「 [gRPC サービスの](xref:tutorials/grpc/grpc-start)使用」で使用されている*あいさつ*ファイルについて考えてみます。

* サービスを`Greeter`定義します。
* サービス`Greeter`は呼び出しを`SayHello`定義します。
* `SayHello`メッセージを送信し、メッセージ`HelloReply`を受信します。 `HelloRequest`

[!code-protobuf[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Protos/greet.proto)]

## <a name="add-a-proto-file-to-a-c-app"></a>プロトコルファイルを C\#アプリに追加する

*@No__t-1*ファイルは、次のように @no__t 項目グループに追加することによってプロジェクトに含められます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=2&range=7-9)]

## <a name="c-tooling-support-for-proto-files"></a>.proto ファイルに対する C# ツール サポート

ファイルからC# *アセットを生成するには、ツールパッケージ [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) が必要\*です。* 生成されたアセット (ファイル):

* は、プロジェクトがビルドされるたびに、必要に応じて生成されます。
* プロジェクトに追加されていないか、ソース管理にチェックインされていません。
* は、 *obj*ディレクトリに含まれるビルド成果物です。

このパッケージは、サーバープロジェクトとクライアントプロジェクトの両方で必要です。 メタパッケージ`Grpc.AspNetCore`には、へ`Grpc.Tools`の参照が含まれています。 サーバープロジェクトは、 `Grpc.AspNetCore` Visual Studio のパッケージマネージャーを使用するか、プロジェクト`<PackageReference>`ファイルにを追加することによって追加できます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=1&range=12)]

クライアントプロジェクトは、gRPC クライアントを使用するために必要な他のパッケージと一緒に直接参照`Grpc.Tools`する必要があります。 ツールパッケージは実行時に必須ではないため、依存関係`PrivateAssets="All"`は次のようにマークされます。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/GrpcGreeterClient.csproj?highlight=3&range=9-11)]

## <a name="generated-c-assets"></a>生成C#されたアセット

ツールパッケージは、含まC#れている *\** ファイルで定義されているメッセージを表す型を生成します。

サーバー側アセットの場合、抽象サービスの基本型が生成されます。 基本型には、 *proto*ファイルに含まれるすべての gRPC 呼び出しの定義が含まれています。 この基本型から派生し、gRPC 呼び出しのロジックを実装する具象サービス実装を作成します。 では、前に説明した例で`GreeterBase`は、仮想`SayHello`メソッドを含む抽象型が生成されます。 `greet.proto` 具象実装`GreeterService`は、メソッドをオーバーライドし、gRPC 呼び出しを処理するロジックを実装します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/Services/GreeterService.cs?name=snippet)]

クライアント側の資産の場合は、具象的なクライアントの種類が生成されます。 *Proto*ファイル内の gRPC 呼び出しは、具象型のメソッドに変換されます。これは、を呼び出すことができます。 前に説明した例では、具象`GreeterClient`型が生成されています。 `greet.proto` を`GreeterClient.SayHelloAsync`呼び出して、サーバーに対する gRPC 呼び出しを開始します。

[!code-csharp[](~/tutorials/grpc/grpc-start/sample/GrpcGreeterClient/Program.cs?name=snippet)]

既定では、サーバーとクライアントの資産は、@no__t 2 つの項目グループに含まれる *@no__t*の各ファイルに対して生成されます。 サーバープロジェクト`GrpcServices`でサーバー資産のみが生成されるようにするには、属性を`Server`に設定します。

[!code-xml[](~/tutorials/grpc/grpc-start/sample/GrpcGreeter/GrpcGreeter.csproj?highlight=2&range=7-9)]

同様に、クライアントプロジェクトでは`Client` 、属性はに設定されます。

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="additional-resources"></a>その他の技術情報

* <xref:grpc/index>
* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/aspnetcore>
* <xref:grpc/client>
