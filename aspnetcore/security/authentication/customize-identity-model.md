---
title: ASP.NET Core での Identity モデルのカスタマイズ
author: ajcvickers
description: この記事では、ASP.NET Core Identity の基になる Entity Framework Core のデータ モデルをカスタマイズする方法について説明します。
ms.author: avickers
ms.date: 07/01/2019
uid: security/authentication/customize_identity_model
ms.openlocfilehash: f549fdff4a416b5fadcb2b1078b051bbab8e402e
ms.sourcegitcommit: eb3e51d58dd713eefc242148f45bd9486be3a78a
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/02/2019
ms.locfileid: "67500472"
---
# <a name="identity-model-customization-in-aspnet-core"></a>ASP.NET Core での Identity モデルのカスタマイズ

作成者: [Arthur Vickers](https://github.com/ajcvickers)

ASP.NET Core Identity は、管理および ASP.NET Core アプリでユーザー アカウントを格納するためのフレームワークを提供します。 Id が、プロジェクトに追加時に**個々 のユーザー アカウント**認証メカニズムとして選択されます。 既定では、Id はなりますの Entity Framework (EF) を使用して、コア データ モデル。 この記事では、Id モデルをカスタマイズする方法について説明します。

## <a name="identity-and-ef-core-migrations"></a>Identity と EF Core のマイグレーション

Id がどのように機能するかを把握するのに役立ちますが、モデルを調べる前に、 [EF コア移行](/ef/core/managing-schemas/migrations/)を作成してデータベースを更新します。 最上位のレベルでは、手順は次のとおりです。

1. 定義または更新を[コードでデータ モデル](/ef/core/modeling/)します。
1. このモデルに変更をデータベースに適用できる変換への移行を追加します。
1. 移行が正しく、開発者の意図を表すことを確認します。
1. モデルと同期するデータベースの更新への移行を適用します。
1. 手順 1. ~ 4. をさらに、モデルを調整し、データベースの同期を維持します。

追加して、移行を適用するには、次の方法のいずれかを使用します。

* **パッケージ マネージャー コンソール**(PMC) ウィンドウの場合、Visual Studio を使用します。 詳細については、次を参照してください。 [EF Core の優れた PMC ツール](/ef/core/miscellaneous/cli/powershell)します。
* .NET Core CLI のコマンドラインを使用する場合。 詳細については、次を参照してください。 [EF Core .NET コマンド ライン ツール](/ef/core/miscellaneous/cli/dotnet)します。
* クリックすると、**適用移行**アプリの実行時にエラー ページ ボタンをします。

ASP.NET Core では、開発時のエラー ページ ハンドラーがあります。 ハンドラーは、アプリの実行時に、移行を適用できます。 通常、実稼働アプリは、移行からの SQL スクリプトを生成し、管理されたアプリとデータベースの配置の一部としてデータベースの変更をデプロイします。

Id を使用して新しいアプリが作成されると、手順 1. と上記の 2 が完了しました既に。 つまり、初期のデータ モデルが既に存在して、最初の移行がプロジェクトに追加されました。 最初の移行は、データベースに適用する必要があります。 次の方法のいずれかでは、最初の移行を適用できます。

* 実行`Update-Database`PMC でします。
* 実行`dotnet ef database update`コマンド シェルでします。
* をクリックして、**移行を適用**アプリの実行時にエラー ページにボタンをクリックします。

モデルに変更が加えられるとは、上記の手順を繰り返します。

## <a name="the-identity-model"></a>Identity モデル

### <a name="entity-types"></a>エンティティ型

Id モデルは、次のエンティティ型で構成されます。

|エンティティ型|説明                                                  |
|-----------|-------------------------------------------------------------|
|`User`     |ユーザーを表します。                                         |
|`Role`     |ロールを表します。                                           |
|`UserClaim`|ユーザーが所有するクレームを表します。                    |
|`UserToken`|ユーザーの認証トークンを表します。               |
|`UserLogin`|ユーザーをログインに関連付けます。                              |
|`RoleClaim`|ロール内のすべてのユーザーに付与されるクレームを表します。|
|`UserRole` |ユーザーとロールを関連付ける結合エンティティを指定します。               |

### <a name="entity-type-relationships"></a>エンティティ型のリレーションシップ

[エンティティ型](#entity-types)次の方法で相互に関連します。

* 各`User`多く持つことができます`UserClaims`します。
* 各`User`多く持つことができます`UserLogins`します。
* 各`User`多く持つことができます`UserTokens`します。
* 各`Role`関連付けられた多く持つことができます`RoleClaims`します。
* 各`User`関連付けられた多く持つことができます`Roles`、および各`Role`さまざまな使用に関連付けることができます`Users`します。 これは、データベース内の結合テーブルを必要とする多対多リレーションシップです。 結合テーブルがによって表される、`UserRole`エンティティ。

### <a name="default-model-configuration"></a>既定のモデルの構成

多くの id が定義*コンテキスト クラス*から継承した[DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)を構成し、モデルを使用します。 この構成を行うを使用して、 [EF Core Code First Fluent API](/ef/core/modeling/)で、 [OnModelCreating](/dotnet/api/microsoft.entityframeworkcore.dbcontext.onmodelcreating)コンテキスト クラスのメソッド。 既定の構成は次のとおりです。

```csharp
builder.Entity<TUser>(b =>
{
    // Primary key
    b.HasKey(u => u.Id);

    // Indexes for "normalized" username and email, to allow efficient lookups
    b.HasIndex(u => u.NormalizedUserName).HasName("UserNameIndex").IsUnique();
    b.HasIndex(u => u.NormalizedEmail).HasName("EmailIndex");

    // Maps to the AspNetUsers table
    b.ToTable("AspNetUsers");

    // A concurrency token for use with the optimistic concurrency checking
    b.Property(u => u.ConcurrencyStamp).IsConcurrencyToken();

    // Limit the size of columns to use efficient database types
    b.Property(u => u.UserName).HasMaxLength(256);
    b.Property(u => u.NormalizedUserName).HasMaxLength(256);
    b.Property(u => u.Email).HasMaxLength(256);
    b.Property(u => u.NormalizedEmail).HasMaxLength(256);

    // The relationships between User and other entity types
    // Note that these relationships are configured with no navigation properties

    // Each User can have many UserClaims
    b.HasMany<TUserClaim>().WithOne().HasForeignKey(uc => uc.UserId).IsRequired();

    // Each User can have many UserLogins
    b.HasMany<TUserLogin>().WithOne().HasForeignKey(ul => ul.UserId).IsRequired();

    // Each User can have many UserTokens
    b.HasMany<TUserToken>().WithOne().HasForeignKey(ut => ut.UserId).IsRequired();

    // Each User can have many entries in the UserRole join table
    b.HasMany<TUserRole>().WithOne().HasForeignKey(ur => ur.UserId).IsRequired();
});

builder.Entity<TUserClaim>(b =>
{
    // Primary key
    b.HasKey(uc => uc.Id);

    // Maps to the AspNetUserClaims table
    b.ToTable("AspNetUserClaims");
});

builder.Entity<TUserLogin>(b =>
{
    // Composite primary key consisting of the LoginProvider and the key to use
    // with that provider
    b.HasKey(l => new { l.LoginProvider, l.ProviderKey });

    // Limit the size of the composite key columns due to common DB restrictions
    b.Property(l => l.LoginProvider).HasMaxLength(128);
    b.Property(l => l.ProviderKey).HasMaxLength(128);

    // Maps to the AspNetUserLogins table
    b.ToTable("AspNetUserLogins");
});

builder.Entity<TUserToken>(b =>
{
    // Composite primary key consisting of the UserId, LoginProvider and Name
    b.HasKey(t => new { t.UserId, t.LoginProvider, t.Name });

    // Limit the size of the composite key columns due to common DB restrictions
    b.Property(t => t.LoginProvider).HasMaxLength(maxKeyLength);
    b.Property(t => t.Name).HasMaxLength(maxKeyLength);

    // Maps to the AspNetUserTokens table
    b.ToTable("AspNetUserTokens");
});

builder.Entity<TRole>(b =>
{
    // Primary key
    b.HasKey(r => r.Id);

    // Index for "normalized" role name to allow efficient lookups
    b.HasIndex(r => r.NormalizedName).HasName("RoleNameIndex").IsUnique();

    // Maps to the AspNetRoles table
    b.ToTable("AspNetRoles");

    // A concurrency token for use with the optimistic concurrency checking
    b.Property(r => r.ConcurrencyStamp).IsConcurrencyToken();

    // Limit the size of columns to use efficient database types
    b.Property(u => u.Name).HasMaxLength(256);
    b.Property(u => u.NormalizedName).HasMaxLength(256);

    // The relationships between Role and other entity types
    // Note that these relationships are configured with no navigation properties

    // Each Role can have many entries in the UserRole join table
    b.HasMany<TUserRole>().WithOne().HasForeignKey(ur => ur.RoleId).IsRequired();

    // Each Role can have many associated RoleClaims
    b.HasMany<TRoleClaim>().WithOne().HasForeignKey(rc => rc.RoleId).IsRequired();
});

builder.Entity<TRoleClaim>(b =>
{
    // Primary key
    b.HasKey(rc => rc.Id);

    // Maps to the AspNetRoleClaims table
    b.ToTable("AspNetRoleClaims");
});

builder.Entity<TUserRole>(b =>
{
    // Primary key
    b.HasKey(r => new { r.UserId, r.RoleId });

    // Maps to the AspNetUserRoles table
    b.ToTable("AspNetUserRoles");
});
```

### <a name="model-generic-types"></a>ジェネリック型のモデル

Identity は、既定値を定義します。[共通言語ランタイム](/dotnet/standard/glossary#clr)上のエンティティの種類ごとの (CLR) 型の一覧にします。 これらの型はすべてを先頭*Identity*:

* `IdentityUser`
* `IdentityRole`
* `IdentityUserClaim`
* `IdentityUserToken`
* `IdentityUserLogin`
* `IdentityRoleClaim`
* `IdentityUserRole`

これらの型を直接使用するのではなく アプリの種類の型の基本クラスとして使用できます。 `DbContext` Id によって定義されているクラスはジェネリックでは、モデルのエンティティ型の 1 つ以上の別の CLR 型を使用できるようにします。 これらのジェネリック型が許可することも、`User`主キー (PK) データの種類を変更します。

ロールについては、サポートでの Id を使用する場合、<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext>クラスを使用する必要があります。 例:

```csharp
// Uses all the built-in Identity types
// Uses `string` as the key type
public class IdentityDbContext
    : IdentityDbContext<IdentityUser, IdentityRole, string>
{
}

// Uses the built-in Identity types except with a custom User type
// Uses `string` as the key type
public class IdentityDbContext<TUser>
    : IdentityDbContext<TUser, IdentityRole, string>
        where TUser : IdentityUser
{
}

// Uses the built-in Identity types except with custom User and Role types
// The key type is defined by TKey
public class IdentityDbContext<TUser, TRole, TKey> : IdentityDbContext<
    TUser, TRole, TKey, IdentityUserClaim<TKey>, IdentityUserRole<TKey>,
    IdentityUserLogin<TKey>, IdentityRoleClaim<TKey>, IdentityUserToken<TKey>>
        where TUser : IdentityUser<TKey>
        where TRole : IdentityRole<TKey>
        where TKey : IEquatable<TKey>
{
}

// No built-in Identity types are used; all are specified by generic arguments
// The key type is defined by TKey
public abstract class IdentityDbContext<
    TUser, TRole, TKey, TUserClaim, TUserRole, TUserLogin, TRoleClaim, TUserToken>
    : IdentityUserContext<TUser, TKey, TUserClaim, TUserLogin, TUserToken>
         where TUser : IdentityUser<TKey>
         where TRole : IdentityRole<TKey>
         where TKey : IEquatable<TKey>
         where TUserClaim : IdentityUserClaim<TKey>
         where TUserRole : IdentityUserRole<TKey>
         where TUserLogin : IdentityUserLogin<TKey>
         where TRoleClaim : IdentityRoleClaim<TKey>
         where TUserToken : IdentityUserToken<TKey>
```

この場合、ロール (要求のみ) がない Id を使用することも、<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityUserContext%601>クラスを使用する必要があります。

```csharp
// Uses the built-in non-role Identity types except with a custom User type
// Uses `string` as the key type
public class IdentityUserContext<TUser>
    : IdentityUserContext<TUser, string>
        where TUser : IdentityUser
{
}

// Uses the built-in non-role Identity types except with a custom User type
// The key type is defined by TKey
public class IdentityUserContext<TUser, TKey> : IdentityUserContext<
    TUser, TKey, IdentityUserClaim<TKey>, IdentityUserLogin<TKey>,
    IdentityUserToken<TKey>>
        where TUser : IdentityUser<TKey>
        where TKey : IEquatable<TKey>
{
}

// No built-in Identity types are used; all are specified by generic arguments, with no roles
// The key type is defined by TKey
public abstract class IdentityUserContext<
    TUser, TKey, TUserClaim, TUserLogin, TUserToken> : DbContext
        where TUser : IdentityUser<TKey>
        where TKey : IEquatable<TKey>
        where TUserClaim : IdentityUserClaim<TKey>
        where TUserLogin : IdentityUserLogin<TKey>
        where TUserToken : IdentityUserToken<TKey>
{
}
```

## <a name="customize-the-model"></a>モデルのカスタマイズ

モデルのカスタマイズの開始点では、適切なコンテキストの型から派生します。 参照してください、[ジェネリック型をモデル化](#model-generic-types)セクション。 このコンテキスト型が慣例的に呼び出される`ApplicationDbContext`と ASP.NET Core テンプレートによって作成されます。

コンテキストを使用して、2 つの方法でモデルを構成します。

* エンティティとジェネリック型パラメーターのキーの種類を指定します。
* オーバーライドする`OnModelCreating`これらの型のマッピングを変更します。

オーバーライドするときに`OnModelCreating`、`base.OnModelCreating`最初に呼び出す必要があります。 オーバーライドする側の構成を次に呼び出す必要があります。 EF Core には、最後の 1 つの wins のポリシー構成を一般があります。 たとえば場合、`ToTable`エンティティ型のメソッドが 1 つのテーブルの名前の最初と呼ばれ、もう一度後で別のテーブルの名前を持つ 2 番目の呼び出し内のテーブル名が使用されます。

### <a name="custom-user-data"></a>カスタム ユーザー データ

<!--
set projNam=WebApp1
dotnet new webapp -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design 
dotnet aspnet-codegenerator identity  -dc ApplicationDbContext --useDefaultUI 
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
 -->

[カスタム ユーザー データ](xref:security/authentication/add-user-data)から継承することではサポートされて`IdentityUser`します。 この型名前を指定するが通例`ApplicationUser`:

```csharp
public class ApplicationUser : IdentityUser
{
    public string CustomTag { get; set; }
}
```

使用して、`ApplicationUser`に関しては、汎用引数として型。

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
    }
}
```

オーバーライドする必要はありません`OnModelCreating`で、`ApplicationDbContext`クラス。 EF Core のマップ、`CustomTag`慣例プロパティ。 データベースを新たに作成する更新が必要、`CustomTag`列。 列を作成するには、移行を追加および」の説明に従って、データベースを更新[Id と EF Core の移行](#identity-and-ef-core-migrations)します。

Update *Pages/Shared/_LoginPartial.cshtml*と置換`IdentityUser`で`ApplicationUser`:

```cshtml
@using Microsoft.AspNetCore.Identity
@using WebApp1.Areas.Identity.Data
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager
```

Update *Areas/Identity/IdentityHostingStartup.cs*または`Startup.ConfigureServices`と置換`IdentityUser`で`ApplicationUser`します。

```csharp
services.AddDefaultIdentity<ApplicationUser>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultUI();
```

ASP.NET Core 2.1 以降では、Identity は、Razor クラス ライブラリとして提供されます。 詳細については、「 <xref:security/authentication/scaffold-identity> 」を参照してください。 したがって、上記のコードへの呼び出しが必要です。<xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>します。 Id ファイルをプロジェクトに追加する Identity scaffolder を使用した場合への呼び出しを削除`AddDefaultUI`します。 詳細については次を参照してください:

* [Identity のスキャフォールディング](xref:security/authentication/scaffold-identity)
* [追加、ダウンロード、および Id にカスタム ユーザー データの削除](xref:security/authentication/add-user-data)

### <a name="change-the-primary-key-type"></a>主キーの型の変更

データベースが作成された後に、PK 列のデータ型への変更は多くのデータベース システムで問題が発生します。 通常、主キーを変更するには、削除して、テーブルを再作成する必要があります。 そのため、データベースが作成されたときに、最初の移行でキーの種類を指定してください。

主キーの種類を変更する次の手順に従います。

1. データベースが作成された場合、PK 変更する前に実行`Drop-Database`(PMC) または`dotnet ef database drop`(.NET Core CLI) を削除します。
2. データベースの削除を確定した後で初期の移行を削除`Remove-Migration`(PMC) または`dotnet ef migrations remove`(.NET Core CLI)。
3. 更新プログラム、`ApplicationDbContext`クラスから派生する<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext%603>します。 新しいキーの種類を指定`TKey`します。 たとえば、使用するため、`Guid`キーの種類。

    ```csharp
    public class ApplicationDbContext
        : IdentityDbContext<IdentityUser<Guid>, IdentityRole<Guid>, Guid>
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
    ```

    ::: moniker range=">= aspnetcore-2.0"

    上記のコードでは、ジェネリック クラスで<xref:Microsoft.AspNetCore.Identity.IdentityUser%601>と<xref:Microsoft.AspNetCore.Identity.IdentityRole%601>新しいキーの種類を使用するように指定する必要があります。

    ::: moniker-end

    ::: moniker range="<= aspnetcore-1.1"

    上記のコードでは、ジェネリック クラスで<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityUser%601>と<xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityRole%601>新しいキーの種類を使用するように指定する必要があります。

    ::: moniker-end

    `Startup.ConfigureServices` ジェネリック ユーザーを使用する更新する必要があります。

    ::: moniker range=">= aspnetcore-2.1"

    ```csharp
    services.AddDefaultIdentity<IdentityUser<Guid>>()
            .AddEntityFrameworkStores<ApplicationDbContext>()
            .AddDefaultTokenProviders();
    ```

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.0"

    ```csharp
    services.AddIdentity<IdentityUser<Guid>, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>()
            .AddDefaultTokenProviders();
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-1.1"

    ```csharp
    services.AddIdentity<IdentityUser<Guid>, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext, Guid>()
            .AddDefaultTokenProviders();
    ```

    ::: moniker-end

4. 場合、カスタム`ApplicationUser`クラスが使用されているから継承するクラスを更新`IdentityUser`します。 例:

    ::: moniker range="<= aspnetcore-1.1"

    [!code-csharp[](customize-identity-model/samples/1.1/MvcSampleApp/Models/ApplicationUser.cs?name=snippet_ApplicationUser&highlight=4)]

    ::: moniker-end

    ::: moniker range=">= aspnetcore-2.0"

    [!code-csharp[](customize-identity-model/samples/2.1/RazorPagesSampleApp/Data/ApplicationUser.cs?name=snippet_ApplicationUser&highlight=4)]

    ::: moniker-end

    Update`ApplicationDbContext`カスタムを参照する`ApplicationUser`クラス。

    ```csharp
    public class ApplicationDbContext
        : IdentityDbContext<ApplicationUser, IdentityRole<Guid>, Guid>
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }
    }
    ```

    Id サービスを追加するときに、カスタムのデータベース コンテキスト クラスを登録`Startup.ConfigureServices`:

    ::: moniker range=">= aspnetcore-2.1"

    ```csharp
    services.AddDefaultIdentity<ApplicationUser>()
            .AddEntityFrameworkStores<ApplicationDbContext>()
            .AddDefaultUI()
            .AddDefaultTokenProviders();
    ```

    分析することで、プライマリ キーのデータ型が推論される、 [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)オブジェクト。

    ASP.NET Core 2.1 以降では、Identity は、Razor クラス ライブラリとして提供されます。 詳細については、「 <xref:security/authentication/scaffold-identity> 」を参照してください。 したがって、上記のコードへの呼び出しが必要です。<xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>します。 Id ファイルをプロジェクトに追加する Identity scaffolder を使用した場合への呼び出しを削除`AddDefaultUI`します。

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.0"

    ```csharp
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>()
            .AddDefaultTokenProviders();
    ```

    分析することで、プライマリ キーのデータ型が推論される、 [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)オブジェクト。

    ::: moniker-end

    ::: moniker range="<= aspnetcore-1.1"

    ```csharp
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext, Guid>()
            .AddDefaultTokenProviders();
    ```

    <xref:Microsoft.Extensions.DependencyInjection.IdentityEntityFrameworkBuilderExtensions.AddEntityFrameworkStores*>メソッドは、`TKey`主キーのデータ型を表す型。

    ::: moniker-end

5. 場合、カスタム`ApplicationRole`クラスが使用されているから継承するクラスを更新`IdentityRole<TKey>`します。 例:

    [!code-csharp[](customize-identity-model/samples/2.1/RazorPagesSampleApp/Data/ApplicationRole.cs?name=snippet_ApplicationRole&highlight=4)]

    Update`ApplicationDbContext`カスタムを参照する`ApplicationRole`クラス。 たとえば、次のクラスは参照カスタム`ApplicationUser`およびカスタム`ApplicationRole`:

    ::: moniker range=">= aspnetcore-2.1"

    [!code-csharp[](customize-identity-model/samples/2.1/RazorPagesSampleApp/Data/ApplicationDbContext.cs?name=snippet_ApplicationDbContext&highlight=5-6)]

    Id サービスを追加するときに、カスタムのデータベース コンテキスト クラスを登録`Startup.ConfigureServices`:

    [!code-csharp[](customize-identity-model/samples/2.1/RazorPagesSampleApp/Startup.cs?name=snippet_ConfigureServices&highlight=13-16)]

    分析することで、プライマリ キーのデータ型が推論される、 [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)オブジェクト。

    ASP.NET Core 2.1 以降では、Identity は、Razor クラス ライブラリとして提供されます。 詳細については、「 <xref:security/authentication/scaffold-identity> 」を参照してください。 したがって、上記のコードへの呼び出しが必要です。<xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>します。 Id ファイルをプロジェクトに追加する Identity scaffolder を使用した場合への呼び出しを削除`AddDefaultUI`します。

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.0"

    [!code-csharp[](customize-identity-model/samples/2.0/RazorPagesSampleApp/Data/ApplicationDbContext.cs?name=snippet_ApplicationDbContext&highlight=5-6)]

    Id サービスを追加するときに、カスタムのデータベース コンテキスト クラスを登録`Startup.ConfigureServices`:

    [!code-csharp[](customize-identity-model/samples/2.0/RazorPagesSampleApp/Startup.cs?name=snippet_ConfigureServices&highlight=7-9)]

    分析することで、プライマリ キーのデータ型が推論される、 [DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)オブジェクト。

    ::: moniker-end

    ::: moniker range="<= aspnetcore-1.1"

    [!code-csharp[](customize-identity-model/samples/1.1/MvcSampleApp/Data/ApplicationDbContext.cs?name=snippet_ApplicationDbContext&highlight=5-6)]

    Id サービスを追加するときに、カスタムのデータベース コンテキスト クラスを登録`Startup.ConfigureServices`:

    [!code-csharp[](customize-identity-model/samples/1.1/MvcSampleApp/Startup.cs?name=snippet_ConfigureServices&highlight=7-9)]

    <xref:Microsoft.Extensions.DependencyInjection.IdentityEntityFrameworkBuilderExtensions.AddEntityFrameworkStores*>メソッドは、`TKey`主キーのデータ型を表す型。

    ::: moniker-end

### <a name="add-navigation-properties"></a>ナビゲーション プロパティの追加

リレーションシップのモデルの構成を変更すると、その他の変更よりも難しくことができます。 新しい追加のリレーションシップを作成するのではなく、既存のリレーションシップを置き換えるに注意する必要があります。 具体的には、変更されたリレーションシップでは、既存のリレーションシップとして同じ外部キー (FK) プロパティを指定する必要があります。 間のリレーションシップなど`Users`と`UserClaims`、次のように、既定では。

```csharp
builder.Entity<TUser>(b =>
{
    // Each User can have many UserClaims
    b.HasMany<TUserClaim>()
     .WithOne()
     .HasForeignKey(uc => uc.UserId)
     .IsRequired();
});
```

このリレーションシップの FK として指定された、`UserClaim.UserId`プロパティ。 `HasMany` `WithOne`はナビゲーション プロパティのないリレーションシップを作成する引数を指定しないと呼ばれます。

ナビゲーション プロパティを追加`ApplicationUser`に関連付けできる`UserClaims`ユーザーから参照できます。

```csharp
public class ApplicationUser : IdentityUser
{
    public virtual ICollection<IdentityUserClaim<string>> Claims { get; set; }
}
```

`TKey`の`IdentityUserClaim<TKey>`はユーザーの PK に指定された型です。 この場合、`TKey`は`string`既定値が使用されているためです。 **いない**の PK 型、`UserClaim`エンティティ型。

ナビゲーション プロパティが存在するようになりましたことで構成する必要があります`OnModelCreating`:

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<ApplicationUser>(b =>
        {
            // Each User can have many UserClaims
            b.HasMany(e => e.Claims)
                .WithOne()
                .HasForeignKey(uc => uc.UserId)
                .IsRequired();
        });
    }
}
```

呼び出しで指定されたナビゲーション プロパティでのみが、以前とまったく同じ関係が構成されていることに注意してください。`HasMany`します。

ナビゲーション プロパティは、データベースではなく、EF モデルにのみ存在します。 リレーションシップの外部キーが変更されていないため、この種のモデルの変更は更新するデータベースを必要としません。 これは、変更を行った後の移行を追加することで確認できます。 `Up`と`Down`メソッドは空です。

### <a name="add-all-user-navigation-properties"></a>すべてのユーザー ナビゲーション プロパティの追加

次の例では、ガイダンスとして、上のセクションを使用して、ユーザーのすべてのリレーションシップの一方向のナビゲーション プロパティを構成します。

```csharp
public class ApplicationUser : IdentityUser
{
    public virtual ICollection<IdentityUserClaim<string>> Claims { get; set; }
    public virtual ICollection<IdentityUserLogin<string>> Logins { get; set; }
    public virtual ICollection<IdentityUserToken<string>> Tokens { get; set; }
    public virtual ICollection<IdentityUserRole<string>> UserRoles { get; set; }
}
```

```csharp
public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<ApplicationUser>(b =>
        {
            // Each User can have many UserClaims
            b.HasMany(e => e.Claims)
                .WithOne()
                .HasForeignKey(uc => uc.UserId)
                .IsRequired();

            // Each User can have many UserLogins
            b.HasMany(e => e.Logins)
                .WithOne()
                .HasForeignKey(ul => ul.UserId)
                .IsRequired();

            // Each User can have many UserTokens
            b.HasMany(e => e.Tokens)
                .WithOne()
                .HasForeignKey(ut => ut.UserId)
                .IsRequired();

            // Each User can have many entries in the UserRole join table
            b.HasMany(e => e.UserRoles)
                .WithOne()
                .HasForeignKey(ur => ur.UserId)
                .IsRequired();
        });
    }
}
```

### <a name="add-user-and-role-navigation-properties"></a>ユーザーおよびロールのナビゲーション プロパティの追加

次の例では、ガイダンスとして、上のセクションを使用して、ユーザーおよびロールのすべての関係のナビゲーション プロパティを構成します。

```csharp
public class ApplicationUser : IdentityUser
{
    public virtual ICollection<IdentityUserClaim<string>> Claims { get; set; }
    public virtual ICollection<IdentityUserLogin<string>> Logins { get; set; }
    public virtual ICollection<IdentityUserToken<string>> Tokens { get; set; }
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; }
}

public class ApplicationRole : IdentityRole
{
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; }
}

public class ApplicationUserRole : IdentityUserRole<string>
{
    public virtual ApplicationUser User { get; set; }
    public virtual ApplicationRole Role { get; set; }
}
```

```csharp
public class ApplicationDbContext
    : IdentityDbContext<
        ApplicationUser, ApplicationRole, string,
        IdentityUserClaim<string>, ApplicationUserRole, IdentityUserLogin<string>,
        IdentityRoleClaim<string>, IdentityUserToken<string>>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<ApplicationUser>(b =>
        {
            // Each User can have many UserClaims
            b.HasMany(e => e.Claims)
                .WithOne()
                .HasForeignKey(uc => uc.UserId)
                .IsRequired();

            // Each User can have many UserLogins
            b.HasMany(e => e.Logins)
                .WithOne()
                .HasForeignKey(ul => ul.UserId)
                .IsRequired();

            // Each User can have many UserTokens
            b.HasMany(e => e.Tokens)
                .WithOne()
                .HasForeignKey(ut => ut.UserId)
                .IsRequired();

            // Each User can have many entries in the UserRole join table
            b.HasMany(e => e.UserRoles)
                .WithOne(e => e.User)
                .HasForeignKey(ur => ur.UserId)
                .IsRequired();
        });

        modelBuilder.Entity<ApplicationRole>(b =>
        {
            // Each Role can have many entries in the UserRole join table
            b.HasMany(e => e.UserRoles)
                .WithOne(e => e.Role)
                .HasForeignKey(ur => ur.RoleId)
                .IsRequired();
        });

    }
}
```

メモ:

* この例では、`UserRole`結合エンティティで、ユーザーからロールへの多対多リレーションシップを移動するが必要です。
* 反映するようにナビゲーション プロパティの型を変更して`ApplicationXxx`型は、の代わりに使用されているようになりました`IdentityXxx`型。
* 使用してください、`ApplicationXxx`ジェネリック`ApplicationContext`定義します。

### <a name="add-all-navigation-properties"></a>すべてのナビゲーション プロパティの追加

次の例では、ガイダンスとして、上のセクションを使用して、すべてのエンティティ型ですべての関係のナビゲーション プロパティを構成します。

```csharp
public class ApplicationUser : IdentityUser
{
    public virtual ICollection<ApplicationUserClaim> Claims { get; set; }
    public virtual ICollection<ApplicationUserLogin> Logins { get; set; }
    public virtual ICollection<ApplicationUserToken> Tokens { get; set; }
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; }
}

public class ApplicationRole : IdentityRole
{
    public virtual ICollection<ApplicationUserRole> UserRoles { get; set; }
    public virtual ICollection<ApplicationRoleClaim> RoleClaims { get; set; }
}

public class ApplicationUserRole : IdentityUserRole<string>
{
    public virtual ApplicationUser User { get; set; }
    public virtual ApplicationRole Role { get; set; }
}

public class ApplicationUserClaim : IdentityUserClaim<string>
{
    public virtual ApplicationUser User { get; set; }
}

public class ApplicationUserLogin : IdentityUserLogin<string>
{
    public virtual ApplicationUser User { get; set; }
}

public class ApplicationRoleClaim : IdentityRoleClaim<string>
{
    public virtual ApplicationRole Role { get; set; }
}

public class ApplicationUserToken : IdentityUserToken<string>
{
    public virtual ApplicationUser User { get; set; }
}
```

```csharp
public class ApplicationDbContext
    : IdentityDbContext<
        ApplicationUser, ApplicationRole, string,
        ApplicationUserClaim, ApplicationUserRole, ApplicationUserLogin,
        ApplicationRoleClaim, ApplicationUserToken>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        modelBuilder.Entity<ApplicationUser>(b =>
        {
            // Each User can have many UserClaims
            b.HasMany(e => e.Claims)
                .WithOne(e => e.User)
                .HasForeignKey(uc => uc.UserId)
                .IsRequired();

            // Each User can have many UserLogins
            b.HasMany(e => e.Logins)
                .WithOne(e => e.User)
                .HasForeignKey(ul => ul.UserId)
                .IsRequired();

            // Each User can have many UserTokens
            b.HasMany(e => e.Tokens)
                .WithOne(e => e.User)
                .HasForeignKey(ut => ut.UserId)
                .IsRequired();

            // Each User can have many entries in the UserRole join table
            b.HasMany(e => e.UserRoles)
                .WithOne(e => e.User)
                .HasForeignKey(ur => ur.UserId)
                .IsRequired();
        });

        modelBuilder.Entity<ApplicationRole>(b =>
        {
            // Each Role can have many entries in the UserRole join table
            b.HasMany(e => e.UserRoles)
                .WithOne(e => e.Role)
                .HasForeignKey(ur => ur.RoleId)
                .IsRequired();

            // Each Role can have many associated RoleClaims
            b.HasMany(e => e.RoleClaims)
                .WithOne(e => e.Role)
                .HasForeignKey(rc => rc.RoleId)
                .IsRequired();
        });
    }
}
```

### <a name="use-composite-keys"></a>複合キーの使用

前のセクションでは、Id モデルで使用されるキーの型の変更が示されています。 複合キーを使用するキーの Id モデルを変更すると、サポートされていないかお勧めします。 Id を持つ複合キーを使用するには、Identity manager のコードが、モデルと対話する方法を変更する必要があります。 このカスタマイズでは、このドキュメントの範囲外です。

### <a name="change-tablecolumn-names-and-facets"></a>テーブル名/列名とファセットの変更

テーブルと列の名前を変更するには、呼び出す`base.OnModelCreating`します。 次に、既定値のいずれかを上書きする構成を追加します。 たとえば、Identity のすべてのテーブルの名前を変更します。

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.Entity<IdentityUser>(b =>
    {
        b.ToTable("MyUsers");
    });

    modelBuilder.Entity<IdentityUserClaim<string>>(b =>
    {
        b.ToTable("MyUserClaims");
    });

    modelBuilder.Entity<IdentityUserLogin<string>>(b =>
    {
        b.ToTable("MyUserLogins");
    });

    modelBuilder.Entity<IdentityUserToken<string>>(b =>
    {
        b.ToTable("MyUserTokens");
    });

    modelBuilder.Entity<IdentityRole>(b =>
    {
        b.ToTable("MyRoles");
    });

    modelBuilder.Entity<IdentityRoleClaim<string>>(b =>
    {
        b.ToTable("MyRoleClaims");
    });

    modelBuilder.Entity<IdentityUserRole<string>>(b =>
    {
        b.ToTable("MyUserRoles");
    });
}
```

これらの例では、既定の Id 型を使用します。 などのアプリの種類を使用して場合`ApplicationUser`、既定値の型ではなく、その型を構成します。

次の例では、一部の列名を変更します。

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.Entity<IdentityUser>(b =>
    {
        b.Property(e => e.Email).HasColumnName("EMail");
    });

    modelBuilder.Entity<IdentityUserClaim<string>>(b =>
    {
        b.Property(e => e.ClaimType).HasColumnName("CType");
        b.Property(e => e.ClaimValue).HasColumnName("CValue");
    });
}
```

特定のデータベース列のいくつかの種類を構成することができます*ファセット*(たとえば、最大`string`許容長)。 次の例では、いくつかの列の最大長を設定する`string`モデル内のプロパティ。

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.Entity<IdentityUser>(b =>
    {
        b.Property(u => u.UserName).HasMaxLength(128);
        b.Property(u => u.NormalizedUserName).HasMaxLength(128);
        b.Property(u => u.Email).HasMaxLength(128);
        b.Property(u => u.NormalizedEmail).HasMaxLength(128);
    });

    modelBuilder.Entity<IdentityUserToken<string>>(b =>
    {
        b.Property(t => t.LoginProvider).HasMaxLength(128);
        b.Property(t => t.Name).HasMaxLength(128);
    });
}
```

### <a name="map-to-a-different-schema"></a>別のスキーマへのマップ

スキーマは、データベース プロバイダーにわたる動作が異なることができます。 既定値は、SQL Server のすべてのテーブルを作成する、 *dbo*スキーマ。 別のスキーマ内のテーブルを作成できます。 例えば:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    base.OnModelCreating(modelBuilder);

    modelBuilder.HasDefaultSchema("notdbo");
}
```

::: moniker range=">= aspnetcore-2.1"

### <a name="lazy-loading"></a>遅延読み込み

このセクションでは、Id モデルで遅延読み込みプロキシのサポートが追加されます。 ナビゲーション プロパティが読み込まれている確認せずに使用することができるので、遅延読み込みの使用をお勧めします。

エンティティの種類にできるに適したいくつかの方法で遅延読み込み」の説明に従って、 [EF Core ドキュメント](/ef/core/querying/related-data#lazy-loading)します。 わかりやすくするために、必要があります、遅延読み込みプロキシを使用します。

* インストール、 [Microsoft.EntityFrameworkCore.Proxies](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Proxies/)パッケージ。
* 呼び出し<xref:Microsoft.EntityFrameworkCore.ProxiesExtensions.UseLazyLoadingProxies*>内[AddDbContext\<TContext >](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext)します。
* パブリック エンティティ型`public virtual`ナビゲーション プロパティ。

次の例では、通話`UseLazyLoadingProxies`で`Startup.ConfigureServices`:

```csharp
services
    .AddDbContext<ApplicationDbContext>(
        b => b.UseSqlServer(connectionString)
              .UseLazyLoadingProxies())
    .AddDefaultIdentity<ApplicationUser>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

エンティティ型にナビゲーション プロパティを追加する方法のガイダンスについては、前の例を参照してください。

## <a name="additional-resources"></a>その他の技術情報

* <xref:security/authentication/scaffold-identity>

::: moniker-end
