---
title: 策略分配结构的详细信息
description: 描述 Azure 策略使用的策略分配定义，用于将策略定义和参数关联到资源进行评估。
ms.date: 09/23/2019
ms.topic: conceptual
ms.openlocfilehash: 9301004fe05afa77f3e73c6ec97335a17c237ce9
ms.sourcegitcommit: 653e9f61b24940561061bd65b2486e232e41ead4
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/21/2019
ms.locfileid: "74279480"
---
# <a name="azure-policy-assignment-structure"></a>Azure Policy 分配结构

策略分配由 Azure 策略用于定义在策略或计划中分配的资源。 在分配时，策略分配可以确定该组资源的参数值，这样就可以重复使用策略定义来处理相同的资源属性，同时满足不同的遵从性需求。

使用 JSON 创建策略分配。 策略分配包含的元素：

- 显示名称
- 说明
- metadata
- 强制模式
- 策略定义
- 参数

例如，以下 JSON 显示了具有动态参数的_DoNotEnforce_模式下的策略分配：

```json
{
    "properties": {
        "displayName": "Enforce resource naming rules",
        "description": "Force resource names to begin with DeptA and end with -LC",
        "metadata": {
            "assignedBy": "Cloud Center of Excellence"
        },
        "enforcementMode": "DoNotEnforce",
        "policyDefinitionId": "/subscriptions/{mySubscriptionID}/providers/Microsoft.Authorization/policyDefinitions/ResourceNaming",
        "parameters": {
            "prefix": {
                "value": "DeptA"
            },
            "suffix": {
                "value": "-LC"
            }
        }
    }
}
```

所有 Azure Policy 示例均位于 [Azure Policy 示例](../samples/index.md)中。

## <a name="display-name-and-description"></a>显示名称和说明

使用**displayName**和**description**标识策略分配，并提供用于特定资源集的上下文。 **displayName** 的最大长度为 128个字符，**description** 的最大长度为 512个字符。

## <a name="enforcement-mode"></a>强制模式

**EnforcementMode**属性使客户能够测试现有资源上的策略的结果，而无需在[Azure 活动日志](../../../azure-monitor/platform/activity-logs-overview.md)中启动策略效果或触发条目。 这种情况通常称为 "What If"，并与安全部署做法保持一致。 **enforcementMode**不同于[禁用](./effects.md#disabled)的效果，因为这种效果会阻止资源评估发生。

此属性具有以下值：

|Mode |JSON 值 |类型 |手动修正 |活动日志条目 |说明 |
|-|-|-|-|-|-|
|Enabled |默认 |字符串 |是 |是 |策略效果在创建或更新资源的过程中强制执行。 |
|已禁用 |DoNotEnforce |字符串 |是 |否 | 在创建或更新资源的过程中不会强制实施策略。 |

如果策略或计划定义中未指定**enforcementMode** ，则使用值_Default_ 。 即使将**enforcementMode**设置为_DoNotEnforce_，也可以为[deployIfNotExists](./effects.md#deployifnotexists)策略启动[修正任务](../how-to/remediate-resources.md)。

## <a name="policy-definition-id"></a>策略定义 ID

此字段必须是策略定义或计划定义的完整路径名称。
`policyDefinitionId` 是字符串而不是数组。 如果经常将多个策略分配到一起，则建议使用[方案](./definition-structure.md#initiatives)。

## <a name="parameters"></a>参数

策略分配的此段提供[策略定义或计划定义](./definition-structure.md#parameters)中定义的参数的值。
此设计使你可以重复使用不同资源的策略或计划定义，但需要检查不同的业务值或结果。

```json
"parameters": {
    "prefix": {
        "value": "DeptA"
    },
    "suffix": {
        "value": "-LC"
    }
}
```

在此示例中，以前在策略定义中定义的参数 `prefix` 和 `suffix`。 此特定策略分配将 `prefix` 设置为**DeptA** ，并将 `suffix` 为 **-LC**。 同一策略定义可与不同部门的不同参数集一起使用，从而减少策略定义的重复和复杂性，同时提供灵活性。

## <a name="next-steps"></a>后续步骤

- 了解[策略定义结构](./definition-structure.md)。
- 了解如何[以编程方式创建策略](../how-to/programmatically-create.md)。
- 了解如何[获取符合性数据](../how-to/get-compliance-data.md)。
- 了解如何[修正不符合的资源](../how-to/remediate-resources.md)。
- 参阅[使用 Azure 管理组来组织资源](../../management-groups/overview.md)，了解什么是管理组。