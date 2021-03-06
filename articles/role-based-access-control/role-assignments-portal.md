---
title: 使用 Azure RBAC 和 Azure 门户添加或删除角色分配
description: 了解如何使用 Azure 基于角色的访问控制（RBAC）和 Azure 门户为用户、组、服务主体或托管标识授予对 Azure 资源的访问权限。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: 8078f366-a2c4-4fbb-a44b-fc39fd89df81
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/25/2019
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 17a325e96e9709b60da2f23d1dc68e3300fde80c
ms.sourcegitcommit: c69c8c5c783db26c19e885f10b94d77ad625d8b4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/03/2019
ms.locfileid: "74707867"
---
# <a name="add-or-remove-role-assignments-using-azure-rbac-and-the-azure-portal"></a>使用 Azure RBAC 和 Azure 门户添加或删除角色分配

[!INCLUDE [Azure RBAC definition grant access](../../includes/role-based-access-control-definition-grant.md)] 本文介绍了如何使用 Azure 门户分配角色。

如果需要在 Azure Active Directory 中分配管理员角色，请参阅[在 Azure Active Directory 中查看和分配管理员角色](../active-directory/users-groups-roles/directory-manage-roles-portal.md)。

## <a name="prerequisites"></a>必备组件

若要添加或删除角色分配，必须具备：

- `Microsoft.Authorization/roleAssignments/write` 和 `Microsoft.Authorization/roleAssignments/delete` 权限，例如[用户访问管理员](built-in-roles.md#user-access-administrator)或[所有者](built-in-roles.md#owner)

## <a name="overview-of-access-control-iam"></a>访问控制概述（IAM）

**访问控制（IAM）** 是用于分配角色的边栏选项卡。 它也称为标识和访问管理，出现在 Azure 门户中的多个位置。 下面显示了订阅的“访问控制(IAM)”边栏选项卡的示例。

![订阅的“访问控制(IAM)”边栏选项卡](./media/role-assignments-portal/access-control-subscription.png)

若要在访问控制（IAM）边栏选项卡中最有效，在尝试分配角色时，可以回答以下三个问题：

1. **谁需要访问？**

    引用用户、组、服务主体或托管标识的用户。 这也称为*安全主体*。

1. **他们需要什么角色？**

    权限会组合到各个角色中。 可以从多个[内置角色](built-in-roles.md)的列表中进行选择，也可以使用自己的自定义角色。

1. **需要在何处进行访问？**

    其中，是指访问应用到的资源集。 其中，可以是管理组、订阅、资源组或单个资源（例如存储帐户）。 这称为*范围*。

## <a name="add-a-role-assignment"></a>添加角色分配

请按照以下步骤在不同的范围内分配角色。

1. 在 Azure 门户中，单击 "**所有服务**"，然后选择范围。 例如，可以选择“管理组”、“订阅”、“资源组”或某个资源。

1. 单击特定资源。

1. 单击“访问控制(IAM)”。

1. 单击“角色分配”选项卡以查看在此范围内的所有角色分配。

1. 单击“添加” > “添加角色分配”以打开“添加角色分配”窗格。

   如果没有分配角色的权限，则将禁用“添加角色分配”选项。

   ![添加菜单](./media/role-assignments-portal/add-menu.png)

   ![“添加角色分配”窗格](./media/role-assignments-portal/add-role-assignment.png)

1. 在“角色”下拉列表中选择一个角色，例如“虚拟机参与者”。

1. 在“选择”列表中，选择用户、组、服务主体或托管标识。 如果没有在列表中看到安全主体，则可在“选择”框中键入相应内容，以便在目录中搜索显示名称、电子邮件地址和对象标识符。

1. 单击“保存”以分配该角色。

   片刻之后，会在所选范围内为安全主体分配角色。

## <a name="assign-a-user-as-an-administrator-of-a-subscription"></a>将用户分配为订阅的管理员

要使用户成为 Azure 订阅的管理员，请在订阅范围为其分配[所有者](built-in-roles.md#owner)角色。 所有者角色向用户授予对订阅中所有资源的完全访问权限，包括向他人授予访问权限的权限。 这些步骤与任何其他角色分配相同。

1. 在 Azure 门户中，依次单击“所有服务”、“订阅”。

1. 单击要在其中添加角色分配的订阅。

1. 单击“访问控制(IAM)”。

1. 单击“角色分配”选项卡以查看此订阅的所有角色分配。

1. 单击“添加” > “添加角色分配”以打开“添加角色分配”窗格。

   如果没有分配角色的权限，则将禁用“添加角色分配”选项。

   ![添加菜单](./media/role-assignments-portal/add-menu.png)

   ![“添加角色分配”窗格](./media/role-assignments-portal/add-role-assignment.png)

1. 在“角色”下拉列表中，选择“所有者”角色。

1. 在“选择”列表中，选择用户。 如果没有在列表中看到用户，则可在“选择”框中键入相应内容，以便在目录中搜索显示名称和电子邮件地址。

1. 单击“保存”以分配该角色。

   片刻之后，会在订阅范围内为该用户分配“所有者”角色。

## <a name="remove-a-role-assignment"></a>删除角色分配

在 RBAC 中，若要删除访问权限，请删除角色分配。 按照以下步骤删除角色分配。

1. 在要删除访问权限的范围（例如管理组、订阅、资源组或资源）内打开“访问控制(IAM)”。

1. 单击“角色分配”选项卡以查看此订阅的所有角色分配。

1. 在角色分配列表中，在需删除其角色分配的安全主体旁边添加复选标记。

   ![“删除角色分配”消息](./media/role-assignments-portal/remove-role-assignment-select.png)

1. 单击“删除”。

   ![“删除角色分配”消息](./media/role-assignments-portal/remove-role-assignment.png)

1. 在显示的“删除角色分配”消息中，单击“是”。

    不能删除继承的角色分配。 如果需要删除继承的角色分配，则必须在创建角色分配的作用域上进行操作。 在“作用域”列的“(继承)”旁，有一条链接指向分配了此角色的范围。 请转到该处列出的范围以删除角色分配。

   ![“删除角色分配”消息](./media/role-assignments-portal/remove-role-assignment-inherited.png)

## <a name="next-steps"></a>后续步骤

- [使用 Azure RBAC 和 Azure 门户列出角色分配](role-assignments-list-portal.md)
- [教程：使用 RBAC 和 Azure 门户向用户授予对 Azure 资源的访问权限](quickstart-assign-role-user-portal.md)
- [ Azure 资源 RBAC 故障排除](troubleshooting.md)
- [使用 Azure 管理组来组织资源](../governance/management-groups/overview.md)
