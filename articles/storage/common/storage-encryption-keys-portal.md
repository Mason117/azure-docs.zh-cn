---
title: 使用 Azure 门户来配置客户管理的密钥
titleSuffix: Azure Storage
description: 了解如何使用 Azure 门户使用 Azure 存储加密的 Azure Key Vault 来配置客户管理的密钥。 使用客户管理的密钥可以创建、轮换、禁用和撤消访问控制。
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: b1006fead92763c5c2e670527b5e232618b633e5
ms.sourcegitcommit: 8bd85510aee664d40614655d0ff714f61e6cd328
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/06/2019
ms.locfileid: "74895307"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-the-azure-portal"></a>使用 Azure 门户配置客户管理的密钥 Azure Key Vault

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

本文说明如何使用[Azure 门户](https://portal.azure.com/)配置使用客户管理的密钥的 Azure Key Vault。 若要了解如何使用 Azure 门户创建密钥保管库，请参阅[快速入门：使用 Azure 门户从 Azure Key Vault 设置和检索机密](../../key-vault/quick-create-portal.md)。

> [!IMPORTANT]
> 将客户托管的密钥用于 Azure 存储加密要求在 key vault 上设置两个属性，**软删除**并不**清除**。 默认情况下不启用这些属性。 若要启用这些属性，请使用 PowerShell 或 Azure CLI。
> 仅支持 RSA 密钥和密钥大小2048。

## <a name="enable-customer-managed-keys"></a>启用客户管理的密钥

若要在 Azure 门户中启用客户托管的密钥，请执行以下步骤：

1. 导航到存储帐户。
1. 在存储帐户的“设置”边栏选项卡上，单击“加密”。 选择“使用自己的密钥”选项，如下图所示。

    ![显示加密选项的门户屏幕截图](./media/storage-encryption-keys-portal/ssecmk1.png)

## <a name="specify-a-key"></a>指定密钥

启用客户管理的密钥后，你将有机会指定与存储帐户关联的密钥。

### <a name="specify-a-key-as-a-uri"></a>将密钥指定为 URI

若要将密钥指定为 URI，请执行以下步骤：

1. 若要在 Azure 门户中查找密钥 URI，请导航到密钥保管库，并选择 "**密钥**" 设置。 选择所需的密钥，然后单击该密钥以查看其设置。 复制提供 URI 的 "**密钥标识符**" 字段的值。

    ![显示密钥保管库密钥 URI 的屏幕截图](media/storage-encryption-keys-portal/key-uri-portal.png)

1. 在存储帐户的 "**加密**设置" 中，选择 "**输入密钥 URI** " 选项。
1. 在“密钥 URI”字段中，指定 URI。

   ![显示如何输入密钥 URI 的屏幕截图](./media/storage-encryption-keys-portal/ssecmk2.png)

### <a name="specify-a-key-from-a-key-vault"></a>从 Key Vault 指定密钥

若要指定密钥保管库中的密钥，请首先确保你有包含密钥的密钥保管库。 若要指定密钥保管库中的密钥，请执行以下步骤：

1. 选择“从 Key Vault 中选择”选项。
2. 选择包含要使用的密钥的 Key Vault。
3. 从 Key Vault 中选择密钥。

   ![显示客户托管密钥选项的屏幕截图](./media/storage-encryption-keys-portal/ssecmk3.png)

## <a name="update-the-key-version"></a>更新密钥版本

当你创建新的密钥版本时，你将需要更新存储帐户以使用新版本。 执行以下步骤：

1. 导航到你的存储帐户并显示**加密**设置。
1. 指定新密钥版本的 URI。 或者，你可以再次选择密钥保管库和密钥来更新版本。

## <a name="next-steps"></a>后续步骤

- [静态数据的 Azure 存储加密](storage-service-encryption.md)
- [什么是 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview)？
