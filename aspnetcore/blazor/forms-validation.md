---
title: Blazor フォームと検証の ASP.NET Core
author: guardrex
description: Blazor でフォームとフィールドの検証シナリオを使用する方法について説明します。
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 11/04/2019
uid: blazor/forms-validation
ms.openlocfilehash: 6dcc36c5133367493b476655dbdf73b75db9d168
ms.sourcegitcommit: a7bbe3890befead19440075b05b9674351f98872
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/10/2019
ms.locfileid: "73905732"
---
# <a name="aspnet-core-blazor-forms-and-validation"></a>Blazor フォームと検証の ASP.NET Core

作成者: [Daniel Roth](https://github.com/danroth27)、[Luke Latham](https://github.com/guardrex)

[データ注釈](xref:mvc/models/validation)を使用して、Blazor でフォームおよび検証をサポートしています。

次の `ExampleModel` 型は、データ注釈を使用して検証ロジックを定義します。

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

フォームは、`EditForm` コンポーネントを使用して定義されます。 次のフォームは、一般的な要素、コンポーネント、および Razor コードを示しています。

```csharp
<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

* フォームは、`ExampleModel` の種類で定義されている検証を使用して、`name` フィールドのユーザー入力を検証します。 モデルはコンポーネントの `@code` ブロック内に作成され、プライベートフィールド (`exampleModel`) に保持されます。 フィールドは、`<EditForm>` 要素の `Model` 属性に割り当てられます。
* `DataAnnotationsValidator` コンポーネントは、データ注釈を使用して検証サポートをアタッチします。
* `ValidationSummary` コンポーネントは、検証メッセージの概要を示します。
* フォームが正常に送信されると (検証に合格)、`HandleValidSubmit` がトリガーされます。

ユーザー入力の受信と検証には、一連の組み込みの入力コンポーネントを使用できます。 入力は、変更されたときとフォームが送信されたときに検証されます。 次の表に、使用できる入力コンポーネントを示します。

| 入力コンポーネント | &hellip; としてレンダリング       |
| --------------- | ------------------------- |
| `InputText`     | `<input>`                 |
| `InputTextArea` | `<textarea>`              |
| `InputSelect`   | `<select>`                |
| `InputNumber`   | `<input type="number">`   |
| `InputCheckbox` | `<input type="checkbox">` |
| `InputDate`     | `<input type="date">`     |

すべての入力コンポーネント (`EditForm`を含む) では、任意の属性がサポートされます。 コンポーネントパラメーターに一致しない属性は、表示される HTML 要素に追加されます。

入力コンポーネントは、編集時に検証を行い、フィールドの状態を反映するように CSS クラスを変更するための既定の動作を提供します。 一部のコンポーネントには、役に立つ解析ロジックが含まれています。 たとえば、`InputDate` と `InputNumber` では、解析不能な値を検証エラーとして登録することによって適切に処理します。 Null 値を受け入れることができる型では、対象フィールドの null 値の許容もサポートされます (たとえば、`int?`)。

次の `Starship` 型では、以前の `ExampleModel`よりも大きなプロパティとデータ注釈を使用して検証ロジックを定義しています。

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

前の例では、データ注釈が存在しないため、`Description` は省略可能です。

次のフォームでは、`Starship` モデルで定義されている検証を使用してユーザー入力を検証します。

```cshtml
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label for="identifier">Identifier: </label>
        <InputText id="identifier" @bind-Value="starship.Identifier" />
    </p>
    <p>
        <label for="description">Description (optional): </label>
        <InputTextArea id="description" @bind-Value="starship.Description" />
    </p>
    <p>
        <label for="classification">Primary Classification: </label>
        <InputSelect id="classification" @bind-Value="starship.Classification">
            <option value="">Select classification ...</option>
            <option value="Exploration">Exploration</option>
            <option value="Diplomacy">Diplomacy</option>
            <option value="Defense">Defense</option>
        </InputSelect>
    </p>
    <p>
        <label for="accommodation">Maximum Accommodation: </label>
        <InputNumber id="accommodation" 
            @bind-Value="starship.MaximumAccommodation" />
    </p>
    <p>
        <label for="valid">Engineering Approval: </label>
        <InputCheckbox id="valid" @bind-Value="starship.IsValidatedDesign" />
    </p>
    <p>
        <label for="productionDate">Production Date: </label>
        <InputDate id="productionDate" @bind-Value="starship.ProductionDate" />
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship();

    private void HandleValidSubmit()
    {
        Console.WriteLine("OnValidSubmit");
    }
}
```

`EditForm` は、変更されたフィールドと現在の検証メッセージを含む、編集プロセスに関するメタデータを追跡する[カスケード値](xref:blazor/components#cascading-values-and-parameters)として `EditContext` を作成します。 `EditForm` は、有効かつ無効な送信 (`OnValidSubmit`、`OnInvalidSubmit`) にも便利なイベントを提供します。 または、`OnSubmit` を使用して、カスタム検証コードで検証フィールドとチェックフィールドの値をトリガーします。

## <a name="inputtext-based-on-the-input-event"></a>入力イベントに基づく InputText

`change` イベントではなく、`input` イベントを使用するカスタムコンポーネントを作成するには、`InputText` コンポーネントを使用します。

次のマークアップを使用してコンポーネントを作成し、`InputText` が使用される場合と同様に、コンポーネントを使用します。

```cshtml
@inherits InputText

<input 
    @attributes="AdditionalAttributes" 
    class="@CssClass" 
    value="@CurrentValue" 
    @oninput="EventCallback.Factory.CreateBinder<string>(
        this, __value => CurrentValueAsString = __value, CurrentValueAsString)" />
```

## <a name="validation-support"></a>検証のサポート

`DataAnnotationsValidator` コンポーネントは、データ注釈を使用した検証サポートをカスケード `EditContext`にアタッチします。 データ注釈を使用した検証のサポートを有効にするには、この明示的なジェスチャが必要です。 データ注釈とは異なる検証システムを使用するには、`DataAnnotationsValidator` をカスタム実装に置き換えます。 ASP.NET Core の実装は、参照ソース: [Data注釈](https://github.com/aspnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs)を検証するために、 [adddata注釈](https://github.com/aspnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs)を使用して/ます。

`ValidationSummary` コンポーネントは、検証の[概要タグヘルパー](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper)に似たすべての検証メッセージを要約します。

`ValidationMessage` コンポーネントには、[検証メッセージタグヘルパー](xref:mvc/views/working-with-forms#the-validation-message-tag-helper)に似た、特定のフィールドの検証メッセージが表示されます。 `For` 属性と、モデルプロパティに名前を付けるラムダ式を使用して、検証用のフィールドを指定します。

```cshtml
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

`ValidationMessage` コンポーネントと `ValidationSummary` コンポーネントでは、任意の属性がサポートされています。 コンポーネントパラメーターと一致しない属性は、生成された `<div>` 要素または `<ul>` 要素に追加されます。

::: moniker range=">= aspnetcore-3.1"

**Blazor パッケージ (AspNetCore の検証)**

[Microsoft.AspNetCore.Blazor.DataAnnotations.Validation](https://www.nuget.org/packages/Microsoft.AspNetCore.Blazor.DataAnnotations.Validation)は、`DataAnnotationsValidator` コンポーネントを使用して、検証エクスペリエンスのギャップを埋めるパッケージです。 パッケージは現在*試験段階*です。今後のリリースでは、これらのシナリオを ASP.NET Core framework に追加する予定です。

`DataAnnotationsValidator` コンポーネントでは、検証モデルの複合プロパティのサブプロパティは検証されません。 コレクション型プロパティの項目は検証されません。 これらの型を検証するために、`Microsoft.AspNetCore.Blazor.DataAnnotations.Validation` パッケージでは、`ObjectGraphDataAnnotationsValidator` コンポーネントと連動する `ValidateComplexType` 検証属性が導入されています。 これらの型の使用例については、 [aspnet/Samples GitHub リポジトリの Blazor 検証サンプル](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/Validation)を参照してください。

<xref:System.ComponentModel.DataAnnotations.CompareAttribute> は、`DataAnnotationsValidator` コンポーネントでは適切に機能しません。 `Microsoft.AspNetCore.Blazor.DataAnnotations.Validation` パッケージでは、これらの制限を回避する追加の検証属性 `ComparePropertyAttribute`が導入されています。 Blazor アプリでは、`ComparePropertyAttribute` は `CompareAttribute`の直接置換です。 詳細については、「 [OnValidSubmit EditForm では Compareattribute が無視されました (aspnet/AspNetCore \#10643)](https://github.com/aspnet/AspNetCore/issues/10643#issuecomment-543909748)」を参照してください。

::: moniker-end

::: moniker range="< aspnetcore-3.1"

### <a name="validation-of-complex-or-collection-type-properties"></a>複合型またはコレクション型のプロパティの検証

モデルのプロパティに適用される検証属性は、フォームが送信されるときに検証されます。 ただし、モデルのコレクションまたは複合データ型のプロパティは、`DataAnnotationsValidator` コンポーネントによるフォームの送信時に検証されません。 このシナリオで入れ子になった検証属性を使用するには、カスタム検証コンポーネントを使用します。 例については、 [aspnet/Samples GitHub リポジトリの Blazor 検証サンプル](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/Validation)を参照してください。

::: moniker-end
