---
title: ASP.NET Core のカスタム承認ポリシープロバイダー
author: mjrousos
description: ASP.NET Core アプリでカスタム IAuthorizationPolicyProvider を使用して、承認ポリシーを動的に生成する方法について説明します。
ms.author: riande
ms.custom: mvc
ms.date: 04/15/2019
uid: security/authorization/iauthorizationpolicyprovider
ms.openlocfilehash: b11f7f94e1042e65d5f2ff8ab0fe9ea7838bebeb
ms.sourcegitcommit: b1e480e1736b0fe0e4d8dce4a4cf5c8e47fc2101
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/19/2019
ms.locfileid: "71108057"
---
# <a name="custom-authorization-policy-providers-using-iauthorizationpolicyprovider-in-aspnet-core"></a>ASP.NET Core で IAuthorizationPolicyProvider を使用するカスタム承認ポリシープロバイダー 

作成者: [Mike Rousos](https://github.com/mjrousos)

通常、[ポリシーベースの承認](xref:security/authorization/policies)を使用する場合、ポリシーは`AuthorizationOptions.AddPolicy` 、承認サービス構成の一部としてを呼び出すことによって登録されます。 場合によっては、すべての承認ポリシーをこの方法で登録することができない (または望ましくない) ことがあります。 そのような場合は、カスタム`IAuthorizationPolicyProvider`を使用して、承認ポリシーの提供方法を制御できます。

カスタム[IAuthorizationPolicyProvider](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider)が役に立つ可能性のあるシナリオの例を次に示します。

* 外部サービスを使用してポリシーの評価を提供する。
* 多数のポリシー (部屋番号や年齢など) を使用すると、個々の承認ポリシーを1つの`AuthorizationOptions.AddPolicy`呼び出しで追加することは意味がありません。
* 外部データソース (データベースなど) の情報に基づいて実行時にポリシーを作成するか、別のメカニズムを使用して承認要件を動的に決定します。

[AspNetCore GitHub リポジトリ](https://github.com/aspnet/AspNetCore)から[サンプルコードを表示またはダウンロード](https://github.com/aspnet/AspNetCore/tree/release/2.2/src/Security/samples/CustomPolicyProvider)します。 Aspnet/AspNetCore リポジトリ ZIP ファイルをダウンロードします。 ファイルを解凍します。 *Src/Security/samples/CustomPolicyProvider*プロジェクトフォルダーに移動します。

## <a name="customize-policy-retrieval"></a>ポリシーの取得のカスタマイズ

ASP.NET Core アプリは、インターフェイスの実装`IAuthorizationPolicyProvider`を使用して承認ポリシーを取得します。 既定では、 [Defaultauthorizationpolicyprovider](/dotnet/api/microsoft.aspnetcore.authorization.defaultauthorizationpolicyprovider)が登録され、使用されます。 `DefaultAuthorizationPolicyProvider`呼び出しで指定`AuthorizationOptions`されたからポリシーを返します。 `IServiceCollection.AddAuthorization`

この動作をカスタマイズするには、アプリ`IAuthorizationPolicyProvider`の[依存関係挿入](xref:fundamentals/dependency-injection)コンテナーに別の実装を登録します。 

インターフェイス`IAuthorizationPolicyProvider`には、次の2つの api が含まれます。

* [Getpolicyasync](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider.getpolicyasync#Microsoft_AspNetCore_Authorization_IAuthorizationPolicyProvider_GetPolicyAsync_System_String_)は、指定された名前の承認ポリシーを返します。
* [GetDefaultPolicyAsync](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationpolicyprovider.getdefaultpolicyasync)では、既定の承認ポリシー (ポリシーが`[Authorize]`指定されていない属性に使用されるポリシー) が返されます。 

これら2つの Api を実装することで、承認ポリシーの提供方法をカスタマイズできます。

## <a name="parameterized-authorize-attribute-example"></a>パラメーター化された承認属性の例

`IAuthorizationPolicyProvider`が便利なシナリオの1つは`[Authorize]` 、パラメーターに依存する要件を持つカスタム属性を有効にすることです。 たとえば、[ポリシーベースの承認](xref:security/authorization/policies)ドキュメントでは、age ベース ("AtLeast21") ポリシーがサンプルとして使用されていました。 アプリ内のさまざまなコントローラーアクションを*異なる*年齢のユーザーが使用できるようにする必要がある場合は、さまざまな年齢ベースのポリシーを作成すると便利です。 アプリケーションが必要`AuthorizationOptions`とするさまざまな有効期間ベースのポリシーをすべて登録する代わりに、カスタム`IAuthorizationPolicyProvider`のを使用してポリシーを動的に生成できます。 ポリシーを簡単に使用できるようにするには、のような`[MinimumAgeAuthorize(20)]`カスタム承認属性を使用してアクションに注釈を設定します。

## <a name="custom-authorization-attributes"></a>カスタム承認属性

承認ポリシーは、名前によって識別されます。 前に`MinimumAgeAuthorizeAttribute`説明したカスタムは、対応する承認ポリシーを取得するために使用できる文字列に引数をマップする必要があります。 これを行うには、から`AuthorizeAttribute` `Age`派生させ、プロパティを`AuthorizeAttribute.Policy`ラップする必要があります。

```csharp
internal class MinimumAgeAuthorizeAttribute : AuthorizeAttribute
{
    const string POLICY_PREFIX = "MinimumAge";

    public MinimumAgeAuthorizeAttribute(int age) => Age = age;

    // Get or set the Age property by manipulating the underlying Policy property
    public int Age
    {
        get
        {
            if (int.TryParse(Policy.Substring(POLICY_PREFIX.Length), out var age))
            {
                return age;
            }
            return default(int);
        }
        set
        {
            Policy = $"{POLICY_PREFIX}{value.ToString()}";
        }
    }
}
```

この属性の型に`Policy`は、ハードコーディングされたプレフィックス (`"MinimumAge"`) と、コンストラクターを介して渡される整数に基づく文字列があります。

パラメーターとして整数を受け取る点を除いて、 `Authorize`他の属性と同じようにアクションに適用できます。

```csharp
[MinimumAgeAuthorize(10)]
public IActionResult RequiresMinimumAge10()
```

## <a name="custom-iauthorizationpolicyprovider"></a>カスタム IAuthorizationPolicyProvider

カスタム`MinimumAgeAuthorizeAttribute`を使用すると、必要最小限の期間に承認ポリシーを簡単に要求できます。 解決すべき次の問題は、すべての異なる年齢に対して承認ポリシーを使用できるようにすることです。 ここでは、 `IAuthorizationPolicyProvider`が役に立ちます。

を使用`MinimumAgeAuthorizationAttribute`する場合、承認ポリシー名はパターン`"MinimumAge" + Age`に従います。その`IAuthorizationPolicyProvider`ため、カスタムでは、次の方法で承認ポリシーを生成する必要があります。

* ポリシー名から age を解析しています。
* を`AuthorizationPolicyBuilder`使用して新しいを作成する`AuthorizationPolicy`
* 次の例では、cookie を使用してユーザーが認証されることを前提としています。 は`AuthorizationPolicyBuilder` 、少なくとも1つの認証スキーム名を使用して構築するか、常に成功する必要があります。 そうしないと、ユーザーにチャレンジを提供する方法に関する情報はなく、例外がスローされます。
* の`AuthorizationPolicyBuilder.AddRequirements`年齢に基づいてポリシーに要件を追加します。 他のシナリオでは、 `RequireClaim` `RequireRole`代わりに、、また`RequireUserName`はを使用することもできます。

```csharp
internal class MinimumAgePolicyProvider : IAuthorizationPolicyProvider
{
    const string POLICY_PREFIX = "MinimumAge";

    // Policies are looked up by string name, so expect 'parameters' (like age)
    // to be embedded in the policy names. This is abstracted away from developers
    // by the more strongly-typed attributes derived from AuthorizeAttribute
    // (like [MinimumAgeAuthorize()] in this sample)
    public Task<AuthorizationPolicy> GetPolicyAsync(string policyName)
    {
        if (policyName.StartsWith(POLICY_PREFIX, StringComparison.OrdinalIgnoreCase) &&
            int.TryParse(policyName.Substring(POLICY_PREFIX.Length), out var age))
        {
            var policy = new AuthorizationPolicyBuilder(CookieAuthenticationDefaults.AuthenticationScheme);
            policy.AddRequirements(new MinimumAgeRequirement(age));
            return Task.FromResult(policy.Build());
        }

        return Task.FromResult<AuthorizationPolicy>(null);
    }
}
```

## <a name="multiple-authorization-policy-providers"></a>複数の承認ポリシープロバイダー

カスタム`IAuthorizationPolicyProvider`実装を使用する場合、ASP.NET Core はの`IAuthorizationPolicyProvider`インスタンスを1つだけ使用することに注意してください。 使用されるすべてのポリシー名に対してカスタムプロバイダーが承認ポリシーを提供できない場合は、バックアッププロバイダーに戻される必要があります。 

たとえば、カスタムの年齢ポリシーと従来のロールベースのポリシーの取得の両方を必要とするアプリケーションについて考えてみます。 このようなアプリでは、次のようなカスタム承認ポリシープロバイダーを使用できます。

* ポリシー名の解析を試みます。 
* ポリシー名に age が含まれて`DefaultAuthorizationPolicyProvider`いない場合は、別のポリシープロバイダー (など) を呼び出します。

上の`IAuthorizationPolicyProvider`例の実装は、コンストラクターでフォールバックポリシー `DefaultAuthorizationPolicyProvider`プロバイダーを作成することによって、を使用するように更新できます (ポリシー名が予期された "minimumage" + age のパターンと一致しない場合に使用されます)。

```csharp
private DefaultAuthorizationPolicyProvider FallbackPolicyProvider { get; }

public MinimumAgePolicyProvider(IOptions<AuthorizationOptions> options)
{
    // ASP.NET Core only uses one authorization policy provider, so if the custom implementation
    // doesn't handle all policies it should fall back to an alternate provider.
    FallbackPolicyProvider = new DefaultAuthorizationPolicyProvider(options);
}
```

次に`FallbackPolicyProvider` 、メソッドを更新して、nullを返すのではなく、を使用するようにすること`GetPolicyAsync`ができます。

```csharp
...
return FallbackPolicyProvider.GetPolicyAsync(policyName);
```

## <a name="default-policy"></a>既定のポリシー

名前付き承認ポリシーを提供するだけでなく`IAuthorizationPolicyProvider` 、カスタムで`GetDefaultPolicyAsync`を実装して、ポリシー `[Authorize]`名が指定されていない属性に対して承認ポリシーを提供する必要があります。

多くの場合、この承認属性には認証されたユーザーのみが必要であるため、の呼び出し`RequireAuthenticatedUser`で必要なポリシーを作成できます。

```csharp
public Task<AuthorizationPolicy> GetDefaultPolicyAsync() => 
    Task.FromResult(new AuthorizationPolicyBuilder(CookieAuthenticationDefaults.AuthenticationScheme).RequireAuthenticatedUser().Build());
```

カスタム`IAuthorizationPolicyProvider`のすべての側面と同様に、必要に応じてこれをカスタマイズできます。 場合によっては、フォールバック`IAuthorizationPolicyProvider`から既定のポリシーを取得することが望ましい場合があります。

## <a name="required-policy"></a>必須のポリシー

カスタム`IAuthorizationPolicyProvider`をに実装`GetRequiredPolicyAsync`する必要があります。必要に応じて、常に必須のポリシーを指定することもできます。 が`GetRequiredPolicyAsync` null 以外のポリシーを返す場合、そのポリシーは、要求された他の (名前付きまたは既定の) ポリシーと結合されます。

必要なポリシーが必要ない場合、プロバイダーは、null を返すか、フォールバックプロバイダーに遅延することができます。

```csharp
public Task<AuthorizationPolicy> GetRequiredPolicyAsync() => 
    Task.FromResult<AuthorizationPolicy>(null);
```

## <a name="use-a-custom-iauthorizationpolicyprovider"></a>カスタム IAuthorizationPolicyProvider を使用する

からカスタムポリシーを使用する`IAuthorizationPolicyProvider`には、次のことを行う必要があります。

* ポリシーベースの`AuthorizationHandler`承認シナリオと同様に、適切な型を依存関係の挿入 ([ポリシーベースの承認](xref:security/authorization/policies#authorization-handlers)で説明) に登録します。
* アプリの依存`IAuthorizationPolicyProvider`関係挿入サービスコレクション`Startup.ConfigureServices`() にカスタム型を登録して、既定のポリシープロバイダーを置き換えます。

```csharp
services.AddSingleton<IAuthorizationPolicyProvider, MinimumAgePolicyProvider>();
```

完全なカスタム`IAuthorizationPolicyProvider`サンプルについては、 [aspnet/authsamples GitHub リポジトリ](https://github.com/aspnet/AspNetCore/tree/release/2.2/src/Security/samples/CustomPolicyProvider)を参照してください。
