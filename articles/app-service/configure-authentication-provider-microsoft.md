---
title: 配置 Microsoft 身份验证
description: 了解如何将 Microsoft 帐户身份验证配置为应用服务应用的标识提供者。
ms.assetid: ffbc6064-edf6-474d-971c-695598fd08bf
ms.topic: article
ms.date: 08/08/2019
ms.custom: seodec18
ms.openlocfilehash: f9158a4094b7d2ec148c2cae85decb3ad959b7c3
ms.sourcegitcommit: 265f1d6f3f4703daa8d0fc8a85cbd8acf0a17d30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/02/2019
ms.locfileid: "74671923"
---
# <a name="configure-your-app-service-app-to-use-microsoft-account-login"></a>将应用服务应用配置为使用 Microsoft 帐户登录

[!INCLUDE [app-service-mobile-selector-authentication](../../includes/app-service-mobile-selector-authentication.md)]

本主题说明如何将 Azure 应用服务配置为使用 Microsoft 帐户作为身份验证提供程序。 

## <a name="register-microsoft-account"> </a>使用 Microsoft 帐户注册应用

1. 中转到 Azure 门户中的[**应用注册**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade)。 如果需要，请用 Microsoft 帐户登录。
1. 选择 "**新建注册**"，然后输入应用程序名称。
1. 在 "**重定向 uri**" 中，选择 " **Web**"，然后输入 `https://<app-domain-name>/.auth/login/microsoftaccount/callback supply the endpoint for your application`。 将 *\<的应用程序域名 >* 替换为应用的域名。  例如，`https://contoso.azurewebsites.net/.auth/login/microsoftaccount/callback` 。 请务必在 URL 中使用 HTTPS 方案。

1. 选择“注册”。
1. 复制**应用程序（客户端） ID**。 稍后需要用到此信息。
1. 在左侧窗格中，选择 "**证书" & 机密** > **新的客户端密钥**。 输入 "说明"，选择 "有效期"，然后选择 "**添加**"。
1. 复制 "**证书 & 机密**" 页上显示的值。 离开页面后，不会再次显示。

    > [!IMPORTANT]
    > 密码是一个非常重要的安全凭据。 请不要与任何人共享密码或者在客户端应用程序中分发它。

## <a name="secrets"> </a>向应用服务应用程序添加 Microsoft 帐户信息

1. 在[Azure 门户]中转到你的应用程序。
1. 选择 "**设置**" > "**身份验证/授权**"，并确保 "**应用服务身份验证** **" 已打开**。
1. 在 "**身份验证提供程序**" 下，选择 " **Microsoft 帐户**"。 粘贴你之前获取的应用程序（客户端） ID 和客户端密码。 启用应用程序所需的任何范围。
1. 选择“确定”。

   应用服务提供身份验证，但不限制对站点内容和 Api 的授权访问。 必须在应用代码中为用户授权。

1. 可有可无若要限制对 Microsoft 帐户用户的访问，请将 "**请求未经身份验证时要执行的操作**" 设置为 "**使用 Microsoft 帐户登录**"。 设置此功能时，应用要求对所有请求进行身份验证。 它还会将所有未经身份验证的请求重定向到 Microsoft 帐户进行身份验证。

   > [!CAUTION]
   > 以这种方式限制访问权限适用于对应用的所有调用，对于具有公开可用主页的应用，与在许多单页应用程序中一样，这可能并不理想。 对于此类应用程序，"**允许匿名请求（无操作）** " 可能首选，以便应用手动启动身份验证。 有关详细信息，请参阅[身份验证流](overview-authentication-authorization.md#authentication-flow)。

1. 选择“保存”。

现在，可以使用 Microsoft 帐户在应用中进行身份验证。

## <a name="related-content"></a>后续步骤

[!INCLUDE [app-service-mobile-related-content-get-started-users](../../includes/app-service-mobile-related-content-get-started-users.md)]

<!-- URLs. -->

[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Azure 门户]: https://portal.azure.com/
