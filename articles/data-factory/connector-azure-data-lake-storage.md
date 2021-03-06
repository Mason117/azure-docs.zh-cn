---
title: 复制和转换 Azure Data Lake Storage Gen2 中的数据
description: 了解如何使用 Azure 数据工厂将数据复制到 Azure Data Lake Storage Gen2，以及如何转换 Azure Data Lake Storage Gen2 中的数据。
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/13/2019
ms.openlocfilehash: accf7aef7d810d9d898945adb7835ee1a7ed9404
ms.sourcegitcommit: a5ebf5026d9967c4c4f92432698cb1f8651c03bb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/08/2019
ms.locfileid: "74929973"
---
# <a name="copy-and-transform-data-in-azure-data-lake-storage-gen2-using-azure-data-factory"></a>使用 Azure 数据工厂复制和转换 Azure Data Lake Storage Gen2 中的数据

Azure Data Lake Storage Gen2 （ADLS Gen2）是专用于[Azure Blob 存储](../storage/blobs/storage-blobs-introduction.md)中内置的大数据分析的一组功能。 可以通过使用文件系统和对象存储模式，将其用于与数据进行交互。

本文概述如何使用 Azure 数据工厂中的复制活动将数据从和复制到 Azure Data Lake Storage Gen2，并使用数据流转换 Azure Data Lake Storage Gen2 中的数据。 若要了解 Azure 数据工厂，请阅读[介绍性文章](introduction.md)。

## <a name="supported-capabilities"></a>支持的功能

以下活动支持此 Azure Data Lake Storage Gen2 连接器：

- 带有[支持的源或接收器矩阵](copy-activity-overview.md)的[复制活动](copy-activity-overview.md)
- [映射数据流](concepts-data-flow-overview.md)
- [Lookup 活动](control-flow-lookup-activity.md)
- [GetMetadata 活动](control-flow-get-metadata-activity.md)
- [删除活动](delete-activity.md)

对于复制活动，可以通过此连接器执行以下操作：

- 使用帐户密钥、服务主体或 Azure 资源身份验证的托管标识，将数据从/复制到 Azure Data Lake Storage Gen2。
- 按原样复制文件，或者用[支持的文件格式和压缩编解码器](supported-file-formats-and-compression-codecs.md)分析或生成文件。

>[!IMPORTANT]
>如果在 Azure 存储防火墙设置上启用 "**允许受信任的 Microsoft 服务访问此存储帐户**" 选项，并想要使用 azure 集成运行时连接到 Data Lake Storage Gen2，则必须对 ADLS Gen2 使用[托管标识身份验证](#managed-identity)。

>[!TIP]
>如果启用分层命名空间，则在 Blob 与 Data Lake Storage Gen2 Api 之间，当前没有操作的互操作性。 如果遇到错误 "ErrorCode = FilesystemNotFound"，并出现 "指定的文件系统不存在" 的消息，则这是由通过 Blob API 创建的指定接收器文件系统引起的，而不是在其他位置 Data Lake Storage Gen2 API。 若要解决此问题，请指定一个名称不是 Blob 容器名称的新文件系统。 然后，数据工厂会在数据复制期间自动创建该文件系统。

## <a name="get-started"></a>开始体验

>[!TIP]
>有关如何使用 Data Lake Storage Gen2 连接器的演练，请参阅将[数据加载到 Azure Data Lake Storage Gen2](load-azure-data-lake-storage-gen2.md)中。

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

以下各节提供了有关用于定义特定于 Data Lake Storage Gen2 的数据工厂实体的属性的信息。

## <a name="linked-service-properties"></a>链接服务属性

Azure Data Lake Storage Gen2 连接器支持以下身份验证类型。 有关详细信息，请参阅相应的部分：

- [帐户密钥身份验证](#account-key-authentication)
- [服务主体身份验证](#service-principal-authentication)
- [Azure 资源的托管标识身份验证](#managed-identity)

>[!NOTE]
>使用 PolyBase 将数据加载到 SQL 数据仓库时，如果源 Data Lake Storage Gen2 配置了虚拟网络终结点，则必须按 PolyBase 要求使用托管标识身份验证。 请参阅 "[托管标识身份验证](#managed-identity)" 部分，了解更多配置先决条件。

### <a name="account-key-authentication"></a>帐户密钥身份验证

若要使用存储帐户密钥身份验证，需支持以下属性：

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | type 属性必须设置为 AzureBlobFS。 |是 |
| url | 具有 `https://<accountname>.dfs.core.windows.net`模式 Data Lake Storage Gen2 的终结点。 | 是 |
| accountKey | Data Lake Storage Gen2 的帐户密钥。 将此字段标记为 SecureString 以安全地将其存储在数据工厂中或[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 |是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果数据存储位于专用网络，则可以使用 Azure 集成运行时或自承载集成运行时。 如果未指定此属性，则使用默认的 Azure 集成运行时。 |No |

>[!NOTE]
>使用帐户密钥身份验证时，不支持辅助 ADLS 文件系统终结点。 可以使用其他身份验证类型。

**示例：**

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "accountkey": { 
                "type": "SecureString", 
                "value": "<accountkey>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="service-principal-authentication"></a>服务主体身份验证

若要使用服务主体身份验证，请执行以下步骤。

1. 按照[向 Azure AD 租户注册应用程序](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant)中的步骤，在 Azure Active Directory （Azure AD）中注册应用程序实体。 记下下面的值，这些值用于定义链接服务：

    - 应用程序 ID
    - 应用程序密钥
    - 租户 ID

2. 向服务主体授予适当的权限。 请参阅[文件和目录中的访问控制列表](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories)Data Lake Storage Gen2 中权限的工作方式示例

    - **作为源**：在存储资源管理器中，至少授予对所有上游文件夹和文件系统的**Execute**权限，以及对要复制的文件的**读取**权限。 或者，在访问控制（IAM）中，至少授予**存储 Blob 数据读取**者角色。
    - **作为接收器**：在存储资源管理器中，至少授予对所有上游文件夹和文件系统的**Execute**权限，以及对接收器文件夹的**写入**权限。 或者，在访问控制（IAM）中，至少授予 "**存储 Blob 数据参与者**" 角色。

>[!NOTE]
>如果使用数据工厂 UI 创作，并且未使用 IAM 中的 "存储 Blob 数据读取器/参与者" 角色设置服务主体，则在执行测试连接或浏览/导航文件夹时，选择 "测试与文件路径的连接" 或 "从指定路径浏览"，然后指定具有 Execute 权限的文件系统或路径以继续。

此链接服务支持以下属性：

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | type 属性必须设置为 AzureBlobFS。 |是 |
| url | 具有 `https://<accountname>.dfs.core.windows.net`模式 Data Lake Storage Gen2 的终结点。 | 是 |
| servicePrincipalId | 指定应用程序的客户端 ID。 | 是 |
| servicePrincipalKey | 指定应用程序的密钥。 将此字段标记为 `SecureString`，以将其安全地存储在数据工厂中。 或者，可以[引用存储在 Azure Key Vault 中的机密](store-credentials-in-key-vault.md)。 | 是 |
| tenant | 指定应用程序的租户信息（域名或租户 ID）。 通过将鼠标悬停在 Azure 门户的右上角来检索它。 | 是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果数据存储位于专用网络，则可以使用 Azure 集成运行时或自承载集成运行时。 如果未指定，则使用默认的 Azure 集成运行时。 |No |

**示例：**

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>" 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="managed-identity"></a> Azure 资源的托管标识身份验证

可将数据工厂与代表此特定数据工厂的 [Azure 资源托管标识](data-factory-service-identity.md)相关联。 可以直接使用此托管身份进行 Data Lake Storage Gen2 身份验证，类似于使用自己的服务主体。 它允许此指定的工厂访问 Data Lake Storage Gen2 的数据，并将数据复制到。

若要将托管标识用于 Azure 资源身份验证，请执行以下步骤。

1. 通过复制与工厂一起生成的**服务标识应用程序 ID**的值，[检索数据工厂托管的标识信息](data-factory-service-identity.md#retrieve-managed-identity)。

2. 向托管标识授予适当的权限。 请参阅[文件和目录中的访问控制列表](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories)Data Lake Storage Gen2 中权限的工作方式示例。

    - **作为源**：在存储资源管理器中，至少授予对所有上游文件夹和文件系统的**Execute**权限，以及对要复制的文件的**读取**权限。 或者，在访问控制（IAM）中，至少授予**存储 Blob 数据读取**者角色。
    - **作为接收器**：在存储资源管理器中，至少授予对所有上游文件夹和文件系统的**Execute**权限，以及对接收器文件夹的**写入**权限。 或者，在访问控制（IAM）中，至少授予 "**存储 Blob 数据参与者**" 角色。

>[!NOTE]
>如果使用数据工厂 UI 创作并且托管标识未使用 IAM 中的 "存储 Blob 数据读取器/参与者" 角色设置，则在执行测试连接或浏览/导航文件夹时，选择 "测试与文件路径的连接" 或 "从指定路径浏览"，然后指定具有 Execute 权限的文件系统或路径以继续。

>[!IMPORTANT]
>如果你使用 PolyBase 将 Data Lake Storage Gen2 中的数据加载到 SQL 数据仓库，则在对 Data Lake Storage Gen2 使用托管标识身份验证时，请确保你还遵循[本指南](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md#impact-of-using-vnet-service-endpoints-with-azure-storage)中的步骤1和步骤2，将 sql 数据库服务器注册到 Azure Active Directory （Azure AD）和2）将 "存储 Blob 数据参与者" 角色分配到 sql 数据库服务器;其余的是由数据工厂处理的。 如果使用 Azure 虚拟网络终结点配置了你的 Data Lake Storage Gen2，若要使用 PolyBase 从其加载数据，你必须使用 PolyBase 所需的托管标识身份验证。

此链接服务支持以下属性：

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | type 属性必须设置为 AzureBlobFS。 |是 |
| url | 具有 `https://<accountname>.dfs.core.windows.net`模式 Data Lake Storage Gen2 的终结点。 | 是 |
| connectVia | 用于连接到数据存储的[集成运行时](concepts-integration-runtime.md)。 如果数据存储位于专用网络，则可以使用 Azure 集成运行时或自承载集成运行时。 如果未指定，则使用默认的 Azure 集成运行时。 |No |

**示例：**

```json
{
    "name": "AzureDataLakeStorageGen2LinkedService",
    "properties": {
        "type": "AzureBlobFS",
        "typeProperties": {
            "url": "https://<accountname>.dfs.core.windows.net", 
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>数据集属性

有关可用于定义数据集的各节和属性的完整列表，请参阅[数据集](concepts-datasets-linked-services.md)。

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

基于格式的数据集中 `location` 设置下的 Data Lake Storage Gen2 支持以下属性：

| properties   | 描述                                                  | 需要 |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | 数据集中 `location` 下的 type 属性必须设置为**AzureBlobFSLocation**。 | 是      |
| fileSystem | Data Lake Storage Gen2 文件系统名称。                              | No       |
| folderPath | 给定文件系统下的文件夹的路径。 如果要使用通配符筛选文件夹，请跳过此设置并在 "活动源" 设置中指定它。 | No       |
| fileName   | 给定 fileSystem + folderPath 下的文件名。 如果要使用通配符来筛选文件，请跳过此设置并在 "活动源" 设置中指定它。 | No       |

**示例：**

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobFSLocation",
                "fileSystem": "filesystemname",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

### <a name="legacy-dataset-model"></a>旧数据集模型

>[!NOTE]
>对于向后兼容性，仍支持以下数据集模型。 建议使用前面部分中提到的新模型，然后 ADF 创作 UI 已经切换为生成新模型。

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | 数据集的 type 属性必须设置为 AzureBlobFSFile。 |是 |
| folderPath | Data Lake Storage Gen2 中的文件夹的路径。 如果未指定，它指向根目录。 <br/><br/>支持通配符筛选器。 允许的通配符 `*` （与零个或多个字符匹配）和 `?` （与零个或一个字符匹配）。 如果你的实际文件夹名称包含通配符或此转义字符位于内部，则使用 `^` 进行转义。 <br/><br/>示例：文件系统/文件夹/。 请参阅[文件夹和文件筛选器示例](#folder-and-file-filter-examples)中的更多示例。 |No |
| fileName | 指定 "folderPath" 下的文件的名称或通配符筛选器。 如果没有为此属性指定任何值，则数据集会指向文件夹中的所有文件。 <br/><br/>对于筛选器，允许的通配符 `*` （与零个或多个字符匹配）和 `?` （与零个或一个字符匹配）。<br/>- 示例 1：`"fileName": "*.csv"`<br/>- 示例 2：`"fileName": "???20180427.txt"`<br/>如果你的实际文件名包含通配符或此转义字符位于内部，则使用 `^` 进行转义。<br/><br/>如果没有为输出数据集指定 fileName，并且没有在活动接收器中指定**preserveHierarchy** ，则复制活动会自动生成具有以下模式的文件名： "*Data. [活动运行 ID GUID]。[GUID if FlattenHierarchy]。[格式（如果已配置）。[已配置压缩]* ，例如 "0a405f8a-93ff-4c6f-b3be-f69616f1df7a. gz"。 如果使用表名（而不是查询）从表格源复制，则名称模式为 " *[表名称]. [format]。[压缩（如果已配置）* "，例如" MyTable "。 |No |
| modifiedDatetimeStart | 基于上次修改的属性筛选文件。 如果文件的上次修改时间在 `modifiedDatetimeStart` 和 `modifiedDatetimeEnd`之间的时间范围内，则选择这些文件。 此时间应用于 UTC 时区，格式为 "2018-12-01T05：00： 00Z"。 <br/><br/> 如果要对大量文件执行文件筛选，则启用此设置会影响数据移动的总体性能。 <br/><br/> 属性可以为 NULL，这意味着不会对数据集应用任何文件属性筛选器。 如果 `modifiedDatetimeStart` 具有日期时间值，但 `modifiedDatetimeEnd` 为 NULL，则表示已选择最后修改的属性大于或等于 datetime 值的文件。 当 `modifiedDatetimeEnd` 具有日期时间值，但 `modifiedDatetimeStart` 为 NULL 时，表示已选择 "上次修改时间" 属性小于 "日期时间" 值的文件。| No |
| modifiedDatetimeEnd | 基于上次修改的属性筛选文件。 如果文件的上次修改时间在 `modifiedDatetimeStart` 和 `modifiedDatetimeEnd`之间的时间范围内，则选择这些文件。 此时间应用于 UTC 时区，格式为 "2018-12-01T05：00： 00Z"。 <br/><br/> 如果要对大量文件执行文件筛选，则启用此设置会影响数据移动的总体性能。 <br/><br/> 属性可以为 NULL，这意味着不会对数据集应用任何文件属性筛选器。 如果 `modifiedDatetimeStart` 具有日期时间值，但 `modifiedDatetimeEnd` 为 NULL，则表示已选择最后修改的属性大于或等于 datetime 值的文件。 当 `modifiedDatetimeEnd` 具有日期时间值，但 `modifiedDatetimeStart` 为 NULL 时，表示已选择 "上次修改时间" 属性小于 "日期时间" 值的文件。| No |
| format | 若要在基于文件的存储之间按原样复制文件（二进制副本），可以在输入和输出数据集定义中跳过格式节。<br/><br/>如果想要分析或生成具有特定格式的文件，则下面是支持的文件格式类型：**TextFormat**、**JsonFormat**、**AvroFormat**、**OrcFormat** 和 **ParquetFormat**。 请将 **format** 中的 **type** 属性设置为上述值之一。 有关详细信息，请参阅[文本格式](supported-file-formats-and-compression-codecs.md#text-format)、 [JSON 格式](supported-file-formats-and-compression-codecs.md#json-format)、 [Avro 格式](supported-file-formats-and-compression-codecs.md#avro-format)、 [ORC 格式](supported-file-formats-and-compression-codecs.md#orc-format)和[Parquet 格式](supported-file-formats-and-compression-codecs.md#parquet-format)部分。 |否（仅适用于二进制复制方案） |
| compression | 指定数据的压缩类型和级别。 有关详细信息，请参阅[受支持的文件格式和压缩编解码器](supported-file-formats-and-compression-codecs.md#compression-support)。<br/>支持的类型为 **GZip**、**Deflate**、**BZip2** 和 **ZipDeflate**。<br/>支持的级别为“最佳”和“最快”。 |No |

>[!TIP]
>如需复制文件夹下的所有文件，请仅指定 **folderPath**。<br>若要复制具有给定名称的单个文件，请使用带有文件名的文件夹部分和**文件名**指定**folderPath** 。<br>若要复制文件夹下的部分文件，请使用带有通配符筛选器的文件夹部分和**文件名**指定**folderPath** 。 

**示例：**

```json
{
    "name": "ADLSGen2Dataset",
    "properties": {
        "type": "AzureBlobFSFile",
        "linkedServiceName": {
            "referenceName": "<Azure Data Lake Storage Gen2 linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "myfilesystem/myfolder",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>复制活动属性

有关可用于定义活动的各节和属性的完整列表，请参阅[复制活动配置](copy-activity-overview.md#configuration)和[管道和活动](concepts-pipelines-activities.md)。 本部分列出了 Data Lake Storage Gen2 源和接收器支持的属性。

### <a name="azure-data-lake-storage-gen2-as-a-source-type"></a>作为源类型 Azure Data Lake Storage Gen2

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

基于格式的复制源中 `storeSettings` 设置下的 Data Lake Storage Gen2 支持以下属性：

| properties                 | 描述                                                  | 需要                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | `storeSettings` 下的 type 属性必须设置为**AzureBlobFSReadSetting**。 | 是                                           |
| recursive                | 指示是要从子文件夹中以递归方式读取数据，还是只从指定的文件夹中读取数据。 当 recursive 设置为 true 且接收器是基于文件的存储时，不会在接收器上复制或创建空的文件夹或子文件夹。 允许的值为 **true**（默认值）和 **false**。 | No                                            |
| wildcardFolderPath       | 在数据集中配置的给定文件系统下带有通配符的文件夹路径用于筛选源文件夹。 <br>允许的通配符 `*` （与零个或多个字符匹配）和 `?` （与零个或一个字符匹配）。 如果你的实际文件夹名称中有通配符或此转义字符，请使用 `^` 进行转义。 <br>请参阅[文件夹和文件筛选器示例](#folder-and-file-filter-examples)中的更多示例。 | No                                            |
| wildcardFileName         | 带有给定文件系统 + folderPath/wildcardFolderPath 下的通配符的文件名用于筛选源文件。 <br>允许的通配符 `*` （与零个或多个字符匹配）和 `?` （与零个或一个字符匹配）。 如果你的实际文件夹名称中有通配符或此转义字符，请使用 `^` 进行转义。 请参阅[文件夹和文件筛选器示例](#folder-and-file-filter-examples)中的更多示例。 | 如果未在数据集中指定 `fileName` 则为 Yes |
| modifiedDatetimeStart    | 基于上次修改的属性筛选文件。 如果文件的上次修改时间在 `modifiedDatetimeStart` 和 `modifiedDatetimeEnd`之间的时间范围内，则选择这些文件。 此时间应用于 UTC 时区，格式为 "2018-12-01T05：00： 00Z"。 <br> 属性可以为 NULL，这意味着不会将文件属性筛选器应用于数据集。 如果 `modifiedDatetimeStart` 具有日期时间值，但 `modifiedDatetimeEnd` 为 NULL，则表示已选择最后修改的属性大于或等于 datetime 值的文件。 当 `modifiedDatetimeEnd` 具有日期时间值，但 `modifiedDatetimeStart` 为 NULL 时，表示已选择 "上次修改时间" 属性小于 "日期时间" 值的文件。 | No                                            |
| modifiedDatetimeEnd      | 同上。                                               | No                                            |
| maxConcurrentConnections | 要同时连接到存储存储的连接数。 仅在要限制与数据存储的并发连接时指定。 | No                                            |

**示例：**

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSetting",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AzureBlobFSReadSetting",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="legacy-source-model"></a>旧源模型

>[!NOTE]
>以下复制源模型仍受支持，以便向后兼容。 建议使用前面提到的新模型，然后 ADF 创作 UI 已经切换为生成新模型。

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | 复制活动源的 type 属性必须设置为 AzureBlobFSSource。 |是 |
| recursive | 指示是要从子文件夹中以递归方式读取数据，还是只从指定的文件夹中读取数据。 当 recursive 设置为 true 且接收器是基于文件的存储时，不会在接收器上复制或创建空的文件夹或子文件夹。<br/>允许的值为 **true**（默认值）和 **false**。 | No |
| maxConcurrentConnections | 并发连接到数据存储的连接数。 仅在要限制与数据存储的并发连接时指定。 | No |

**示例：**

```json
"activities":[
    {
        "name": "CopyFromADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ADLS Gen2 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureBlobFSSource",
                "recursive": true
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-data-lake-storage-gen2-as-a-sink-type"></a>Azure Data Lake Storage Gen2 作为接收器类型

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

基于格式的复制接收器 `storeSettings` 设置下的 Data Lake Storage Gen2 支持以下属性：

| properties                 | 描述                                                  | 需要 |
| ------------------------ | ------------------------------------------------------------ | -------- |
| type                     | `storeSettings` 下的 type 属性必须设置为**AzureBlobFSWriteSetting**。 | 是      |
| copyBehavior             | 定义以基于文件的数据存储中的文件为源时的复制行为。<br/><br/>允许值包括：<br/><b>- PreserveHierarchy（默认值）</b>：保留目标文件夹中的文件层次结构。 指向源文件夹的源文件相对路径与指向目标文件夹的目标文件相对路径相同。<br/><b>- FlattenHierarchy</b>：源文件夹中的所有文件都位于目标文件夹的第一级。 目标文件具有自动生成的名称。 <br/><b>- MergeFiles</b>：将源文件夹中的所有文件合并到一个文件中。 如果指定了文件名，则合并文件的名称为指定名称。 否则，它是自动生成的文件名。 | No       |
| maxConcurrentConnections | 并发连接到数据存储的连接数。 仅在要限制与数据存储的并发连接时指定。 | No       |

**示例：**

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Parquet output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "ParquetSink",
                "storeSettings":{
                    "type": "AzureBlobFSWriteSetting",
                    "copyBehavior": "PreserveHierarchy"
                }
            }
        }
    }
]
```

#### <a name="legacy-sink-model"></a>旧接收器模型

>[!NOTE]
>下面的复制接收器模型仍受支持，以便向后兼容。 建议使用前面提到的新模型，然后 ADF 创作 UI 已经切换为生成新模型。

| properties | 描述 | 需要 |
|:--- |:--- |:--- |
| type | 复制活动接收器的 type 属性必须设置为 AzureBlobFSSink。 |是 |
| copyBehavior | 定义以基于文件的数据存储中的文件为源时的复制行为。<br/><br/>允许值包括：<br/><b>- PreserveHierarchy（默认值）</b>：保留目标文件夹中的文件层次结构。 指向源文件夹的源文件相对路径与指向目标文件夹的目标文件相对路径相同。<br/><b>- FlattenHierarchy</b>：源文件夹中的所有文件都位于目标文件夹的第一级。 目标文件具有自动生成的名称。 <br/><b>- MergeFiles</b>：将源文件夹中的所有文件合并到一个文件中。 如果指定了文件名，则合并文件的名称为指定名称。 否则，它是自动生成的文件名。 | No |
| maxConcurrentConnections | 并发连接到数据存储的连接数。 仅在要限制与数据存储的并发连接时指定。 | No |

**示例：**

```json
"activities":[
    {
        "name": "CopyToADLSGen2",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<ADLS Gen2 output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureBlobFSSink",
                "copyBehavior": "PreserveHierarchy"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>文件夹和文件筛选器示例

本部分介绍使用通配符筛选器生成文件夹路径和文件名的行为。

| folderPath | fileName | recursive | 源文件夹结构和筛选器结果（用**粗体**表示的文件已检索）|
|:--- |:--- |:--- |:--- |
| `Folder*` | （空，使用默认值） | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | （空，使用默认值） | 是 | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | false | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| `Folder*` | `*.csv` | 是 | FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="some-recursive-and-copybehavior-examples"></a>一些 recursive 和 copyBehavior 示例

本部分介绍了对于 recursive 值和 copyBehavior 值的不同组合，复制操作产生的行为。

| recursive | copyBehavior | 源文件夹结构 | 生成目标 |
|:--- |:--- |:--- |:--- |
| 是 |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用与源相同的结构创建目标 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| 是 |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用以下结构创建目标 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 的自动生成的名称<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 的自动生成的名称<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 的自动生成的名称<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 的自动生成的名称<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 的自动生成的名称 |
| 是 |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用以下结构创建目标 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File5 的内容将合并到一个文件中，且自动生成文件名。 |
| false |preserveHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用以下结构创建目标 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/>未选取具有 File3、File4 和 File5 的不会选取。 |
| false |flattenHierarchy | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用以下结构创建目标 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 的自动生成的名称<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 的自动生成的名称<br/><br/>未选取具有 File3、File4 和 File5 的不会选取。 |
| false |mergeFiles | Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 | 使用以下结构创建目标 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的内容将合并到一个文件中，且自动生成文件名。 File1 的自动生成的名称<br/><br/>未选取具有 File3、File4 和 File5 的不会选取。 |

## <a name="preserve-acls-from-data-lake-storage-gen1"></a>保留 Data Lake Storage Gen1 中的 Acl

>[!TIP]
>若要将数据从 Azure Data Lake Storage Gen1 复制到一般的 Gen2 中，请参阅[使用 Azure 数据工厂将 Azure Data Lake Storage Gen1 数据复制到 Gen2](load-azure-data-lake-storage-gen2-from-gen1.md)和最佳实践。

将 Azure Data Lake Storage Gen1 中的文件复制到 Gen2 时，可以选择保留 POSIX 访问控制列表（Acl）和数据。 有关访问控制的详细信息，请参阅 Azure Data Lake Storage Gen2 中 Azure Data Lake Storage Gen1 和[access control](../storage/blobs/data-lake-storage-access-control.md)中的[访问控制](../data-lake-store/data-lake-store-access-control.md)。

可以使用 Azure 数据工厂复制活动保存以下类型的 Acl。 您可以选择一个或多个类型：

- **ACL**：复制并保留对文件和目录的 POSIX 访问控制列表。 它将完整的现有 Acl 从源复制到接收器。 
- **所有者**：复制并保留文件和目录的拥有用户。 需要超级用户访问接收器 Data Lake Storage Gen2。
- **组**：复制并保留所属的文件和目录组。 需要超级用户对接收器 Data Lake Storage Gen2 或拥有用户的访问权限（如果拥有用户也是目标组的成员）。

如果指定从文件夹复制，则如果 `recursive` 设置为 true，则数据工厂将复制该给定文件夹的 Acl 以及该文件夹下的文件和目录。 如果指定从单个文件复制，则会复制该文件上的 Acl。

>[!IMPORTANT]
>选择保留 Acl 时，请确保为数据工厂授予足够高的权限，以便在接收器 Data Lake Storage Gen2 帐户上运行。 例如，使用帐户密钥身份验证，或将存储 Blob 数据所有者角色分配给服务主体或托管标识。

将 source 作为 Data Lake Storage Gen1 配置为具有二进制复制选项或二进制格式，并将接收器配置为带有二进制复制选项或二进制格式的 Data Lake Storage Gen2 时，可以在 "**复制数据工具设置**" 页上或在活动创作的 "**复制活动**" > "**设置**" 选项卡上找到 "**保留**" 选项。

![Data Lake Storage Gen1 Gen2 保留 ACL](./media/connector-azure-data-lake-storage/adls-gen2-preserve-acl.png)

下面是 JSON 配置的示例（请参阅 `preserve`）： 

```json
"activities":[
    {
        "name": "CopyFromGen1ToGen2",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "AzureDataLakeStoreSource",
                "recursive": true
            },
            "sink": {
                "type": "AzureBlobFSSink",
                "copyBehavior": "PreserveHierarchy"
            },
            "preserve": [
                "ACL",
                "Owner",
                "Group"
            ]
        },
        "inputs": [
            {
                "referenceName": "<Azure Data Lake Storage Gen1 input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Data Lake Storage Gen2 output dataset name>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="mapping-data-flow-properties"></a>映射数据流属性

在映射数据流功能中了解有关[源转换](data-flow-source.md)和[接收器转换](data-flow-sink.md)的详细信息。

## <a name="lookup-activity-properties"></a>查找活动属性

若要了解有关属性的详细信息，请检查[查找活动](control-flow-lookup-activity.md)。

## <a name="getmetadata-activity-properties"></a>GetMetadata 活动属性

若要了解有关属性的详细信息，请查看[GetMetadata 活动](control-flow-get-metadata-activity.md) 

## <a name="delete-activity-properties"></a>删除活动属性

若要了解有关属性的详细信息，请选中 "[删除活动](delete-activity.md)"
## <a name="next-steps"></a>后续步骤

有关数据工厂中复制活动支持作为源和接收器的数据存储的列表，请参阅[支持的数据存储](copy-activity-overview.md##supported-data-stores-and-formats)。
