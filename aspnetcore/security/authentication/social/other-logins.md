---
title: 外部の OAuth 認証プロバイダー
author: rick-anderson
description: ASP.NET Core アプリを使用する外部 OAuth 認証プロバイダーを検出します。
ms.author: riande
ms.custom: mvc
ms.date: 11/11/2018
uid: security/authentication/otherlogins
ms.openlocfilehash: 2bc9a11d0a46e54b4206f846d187b8c1cc954f89
ms.sourcegitcommit: 8516b586541e6ba402e57228e356639b85dfb2b9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/11/2019
ms.locfileid: "67815560"
---
# <a name="external-oauth-authentication-providers"></a>外部の OAuth 認証プロバイダー

作成者: [Rick Anderson](https://twitter.com/RickAndMSFT)、 [Pranav Rastogi](https://github.com/rustd)、および[Valeriy Novytskyy](https://github.com/01binary)

次の一覧には、ASP.NET Core アプリを使用する共通の外部 OAuth 認証プロバイダーが含まれています。 によって管理されるものなどのサード パーティの NuGet パッケージ[aspnet contrib](https://www.nuget.org/packages?q=owners%3Aaspnet-contrib+title%3AOAuth)、ASP.NET Core チームによって実装される認証プロバイダーを補完するために使用できます。

* [LinkedIn](https://www.linkedin.com/developer/apps) ([指示](https://developer.linkedin.com/docs/oauth2))

* [Instagram](https://www.instagram.com/developer/register/) ([指示](https://www.instagram.com/developer/authentication/))

* [Reddit](https://www.reddit.com/login?dest=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps) ([指示](https://github.com/reddit/reddit/wiki/OAuth2-Quick-Start-Example))

* [Github](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fapplications%2Fnew) ([指示](https://developer.github.com/v3/oauth/))

* [Yahoo](https://login.yahoo.com/config/login?src=devnet&.done=http%3A%2F%2Fdeveloper.yahoo.com%2Fapps%2Fcreate%2F) ([指示](https://developer.yahoo.com/bbauth/user.html))

* [Tumblr](https://www.tumblr.com/oauth/apps) ([指示](https://www.tumblr.com/docs/api/v2#auth))

* [Pinterest](https://www.pinterest.com/login/?next=http%3A%2F%2Fdevsite%2Fapps%2F) ([指示](https://developers.pinterest.com/docs/api/overview/?))

* [Pocket](https://getpocket.com/developer/apps/new) ([指示](https://getpocket.com/developer/docs/authentication))

* [Flickr](https://www.flickr.com/services/apps/create) ([指示](https://www.flickr.com/services/api/auth.oauth.html))

* [Dribble](https://dribbble.com/signup) ([指示](https://developer.dribbble.com/v1/oauth/))

* [Vimeo](https://vimeo.com/join) ([指示](https://developer.vimeo.com/api/authentication))

* [SoundCloud](https://soundcloud.com/you/apps/new) ([指示](https://developers.soundcloud.com/blog/we-love-oauth-2))

* [VK](https://vk.com/apps?act=manage) ([指示](https://vk.com/pages?oid=-17680044&p=Authorizing_Sites))

[!INCLUDE[Multiple authentication providers](includes/chain-auth-providers.md)]

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]
