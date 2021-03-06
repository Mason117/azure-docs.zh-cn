---
title: 教程：为 Azure Active Directory 的自动用户预配配置联合目录 |Microsoft Docs
description: 了解如何配置 Azure Active Directory 以自动将用户帐户预配到联合目录和取消其预配。
services: active-directory
documentationcenter: ''
author: zchia
writer: zchia
manager: beatrizd
ms.assetid: na
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2019
ms.author: zhchia
ms.openlocfilehash: bc19a64fc72fef6407416f65cdc3b60a4c4f9464
ms.sourcegitcommit: a8b638322d494739f7463db4f0ea465496c689c6
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2019
ms.locfileid: "68296072"
---
# <a name="tutorial-configure-federated-directory-for-automatic-user-provisioning"></a>教程：为自动用户预配配置联合目录

本教程的目的是演示要在联合目录中执行的步骤, 并 Azure Active Directory (Azure AD) 配置 Azure AD, 以自动将用户和/或组预配到联合目录。

> [!NOTE]
>  本教程介绍在 Azure AD 用户预配服务之上构建的连接器。 有关此服务的功能、工作原理以及常见问题的重要详细信息，请参阅[使用 Azure Active Directory 自动将用户预配到 SaaS 应用程序和取消预配](../manage-apps/user-provisioning.md)。
>
> 此连接器目前以公共预览版提供。 有关预览功能的一般 Microsoft Azure 使用条款的详细信息, 请参阅[Microsoft Azure 预览版的补充使用条款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>系统必备

本教程中概述的方案假定你已具有以下先决条件：

* Azure AD 租户。
* [联合目录](https://www.federated.directory/pricing)。
* 具有管理员权限的联合目录中的用户帐户。

## <a name="assign-users-to-federated-directory"></a>将用户分配到联合目录
Azure Active Directory 使用称为 "分配" 的概念来确定哪些用户应收到对所选应用的访问权限。 在自动用户预配的上下文中, 只同步已分配到 Azure AD 中的应用程序的用户和/或组。

在配置和启用自动用户预配之前, 应决定 Azure AD 中哪些用户和/或组需要访问联合目录。 确定后, 可按照此处的说明将这些用户和/或组分配到联合目录:

 * [向企业应用分配用户或组](../manage-apps/assign-user-or-group-access-portal.md) 
 
 ## <a name="important-tips-for-assigning-users-to-federated-directory"></a>将用户分配到联合目录的重要提示
 * 建议将单个 Azure AD 用户分配到联合目录来测试自动用户预配配置。 其他用户和/或组可以稍后分配。

* 将用户分配到联合目录时, 必须在分配对话框中选择任何特定于应用程序的有效角色 (如果可用)。 将从设置中排除具有默认访问角色的用户。
    
 ## <a name="set-up-federated-directory-for-provisioning"></a>设置联合目录以进行预配

在使用 Azure AD 为自动用户预配配置联合目录之前, 你将需要在联合目录上启用 SCIM 预配。

1. 登录到[联合目录管理控制台](https://federated.directory/of)

    ![联合目录教程](media/federated-directory-provisioning-tutorial/companyname.png)

2. 导航到 "**目录" > 用户目录**", 然后选择租户。 

    ![联合目录](media/federated-directory-provisioning-tutorial/ad-user-directories.png)

3.  若要生成永久持有者令牌, 请导航到 "**目录密钥" > "创建新密钥"。** 

    ![联合目录](media/federated-directory-provisioning-tutorial/federated01.png)

4. 创建目录密钥。 

    ![联合目录](media/federated-directory-provisioning-tutorial/federated02.png)
    

5. 复制“访问令牌”  值。 此值将在 Azure 门户中联合目录应用程序的 "预配" 选项卡的 "**机密令牌**" 字段中输入。 

    ![联合目录](media/federated-directory-provisioning-tutorial/federated03.png)
    
## <a name="add-federated-directory-from-the-gallery"></a>从库中添加联合目录

若要为 Azure AD 配置自动用户预配, 需要将 Azure AD 应用程序库中的联合目录添加到托管的 SaaS 应用程序列表。

**若要从 Azure AD 应用程序库添加联合目录, 请执行以下步骤:**

1. 在 **[Azure 门户](https://portal.azure.com)** 的左侧导航面板中, 选择 " **Azure Active Directory**"。

    ![“Azure Active Directory”按钮](common/select-azuread.png)

2. 转到“企业应用程序”，并选择“所有应用程序”。  

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

3. 若要添加新应用程序, 请选择窗格顶部的 "**新建应用程序**" 按钮。

    ![“新增应用程序”按钮](common/add-new-app.png)

4. 在搜索框中, 输入 "**联合目录**", 在结果面板中选择 "**联合目录**"。

    ![结果列表中的联合目录](common/search-new-app.png)

5. 在单独的浏览器中导航到下面突出显示的**URL** 。 

    ![联合目录](media/federated-directory-provisioning-tutorial/loginpage1.png)

6. 单击 "**登录"** 。

    ![联合目录](media/federated-directory-provisioning-tutorial/federated04.png)

7.  由于联合目录是 OpenIDConnect 应用, 因此请选择使用 Microsoft 工作帐户登录到联合目录。
    
    ![联合目录](media/federated-directory-provisioning-tutorial/loginpage3.png)
 
8. 身份验证成功后, 接受同意页面的许可提示。 然后, 该应用程序将自动添加到你的租户, 你会被重定向到联合目录帐户。

    ![联合目录添加 SCIM](media/federated-directory-provisioning-tutorial/premission.png)



## <a name="configuring-automatic-user-provisioning-to-federated-directory"></a>配置联合目录的自动用户预配 

本部分将指导你完成以下步骤: 配置 Azure AD 预配服务, 以便基于 Azure AD 中的用户和/或组分配在联合目录中创建、更新和禁用用户和/或组。

### <a name="to-configure-automatic-user-provisioning-for-federated-directory-in-azure-ad"></a>若要在 Azure AD 中配置联合目录的自动用户预配:

1. 登录到 [Azure 门户](https://portal.azure.com)。 选择 "**企业应用程序**", 并选择 "**所有应用程序**"。

    ![“企业应用程序”边栏选项卡](common/enterprise-applications.png)

2. 在应用程序列表中, 选择 "**联合目录**"。

    ![应用程序列表中的联合目录链接](common/all-applications.png)

3. 选择“预配”  选项卡。

    ![设置选项卡](common/provisioning.png)

4. 将“预配模式”  设置为“自动”  。

    ![设置选项卡](common/provisioning-automatic.png)

5. 在 "**管理员凭据**" 部分中`https://api.federated.directory/v2/` , 输入 "租户 URL"。 输入先前从 "**机密令牌**" 中的联合目录检索并保存的值。 单击 "**测试连接**" 以确保 Azure AD 可以连接到联合目录。 如果连接失败, 请确保联合目录帐户具有管理员权限, 然后重试。

    ![租户 URL + 令牌](common/provisioning-testconnection-tenanturltoken.png)

8. 在“通知电子邮件”字段中，输入应接收预配错误通知的个人或组的电子邮件地址，并选中复选框“发生故障时发送电子邮件通知”   。

    ![通知电子邮件](common/provisioning-notification-email.png)

9. 单击“保存”  。

10. 在 "**映射**" 部分下, 选择 "**将 Azure Active Directory 用户同步到联合目录**"。

    ![联合目录教程](media/federated-directory-provisioning-tutorial/user-mappings.png)
    
    
11. 在 "**属性映射**" 部分中, 查看从 Azure AD 同步到联合目录的用户属性。 选为 "**匹配**" 属性的属性用于匹配联合目录中的用户帐户以执行更新操作。 选择“保存”按钮以提交任何更改  。

    ![联合目录教程](media/federated-directory-provisioning-tutorial/user-attributes.png)
    

12. 若要配置范围筛选器，请参阅[范围筛选器教程](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md)中提供的以下说明。

13. 若要为联合目录启用 Azure AD 预配服务, 请在 "**设置**" 部分中将 "**预配状态**" 更改为 **"打开**"。

    ![设置状态切换开启](common/provisioning-toggle-on.png)

14. 通过在 "**设置**" 部分的 "**范围**" 中选择所需的值, 定义要预配到联合目录的用户和/或组。

    ![预配范围](common/provisioning-scope.png)

15. 已准备好预配时，单击“保存”  。

    ![正在保存设置配置](common/provisioning-configuration-save.png)

此操作会对“设置”部分的“范围”中定义的所有用户和/或组启动初始同步   。 初始同步执行的时间比后续同步长，只要 Azure AD 预配服务正在运行，大约每隔 40 分钟就会进行一次同步。 你可以使用 "**同步详细信息**" 部分监视进度并跟踪指向预配活动报告的链接, 该报告描述了 Azure AD 预配服务在联合目录上执行的所有操作。

有关如何读取 Azure AD 设置日志的详细信息, 请参阅[有关自动用户帐户预配的报告](../manage-apps/check-status-user-account-provisioning.md)
## <a name="additional-resources"></a>其他资源

* [管理企业应用的用户帐户预配](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>后续步骤

* [了解如何查看日志并获取有关预配活动的报告](../manage-apps/check-status-user-account-provisioning.md)
