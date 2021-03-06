---
title: 管理 Azure 数据资源管理器中的群集横向缩放（扩大）以适应不断变化的需求
description: 本文介绍了基于不断变化的需求在 Azure 数据资源管理器群集中横向扩展和扩展的步骤。
author: orspod
ms.author: orspodek
ms.reviewer: gabil
ms.service: data-explorer
ms.topic: conceptual
ms.date: 12/09/2019
ms.openlocfilehash: 52a9c0a13723361bbc93362cdd9e2c73ef0372f2
ms.sourcegitcommit: b5ff5abd7a82eaf3a1df883c4247e11cdfe38c19
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/09/2019
ms.locfileid: "74942233"
---
# <a name="manage-cluster-horizontal-scaling-scale-out-in-azure-data-explorer-to-accommodate-changing-demand"></a>管理 Azure 数据资源管理器中的群集横向缩放（扩大）以适应不断变化的需求

调整好群集的大小对于 Azure 数据资源管理器的性能至关重要。 静态群集大小可能导致利用不充分或利用过度，二者都不理想。 由于群集上的需求不能使用绝对准确度预测，因此最好*缩放*群集，添加和删除容量和 CPU 资源，并改变需求。 

有两个用于缩放 Azure 数据资源管理器群集的工作流： 
* 水平缩放，也称为放大和缩小。
* [垂直缩放](manage-cluster-vertical-scaling.md)，也称为向上和向下缩放。
本文介绍水平缩放工作流。

## <a name="configure-horizontal-scaling"></a>配置水平缩放

通过使用水平缩放，你可以根据预定义的规则和计划自动缩放实例计数。 指定群集的自动缩放设置：

1. 在 Azure 门户中转到 Azure 数据资源管理器群集资源。 在 "**设置**" 下，选择 "**横向扩展**"。 

2. 在 "向**外扩展**" 窗口中，选择所需的自动缩放方法：**手动缩放**、**优化自动缩放**或**自定义自动缩放**。

### <a name="manual-scale"></a>手动缩放

在群集创建过程中，手动缩放是默认设置。 群集的静态容量不会自动更改。 使用 "**实例计数**" 栏可以选择静态容量。 群集的缩放将保留在该设置中，直到你进行了其他更改。

   ![手动缩放方法](media/manage-cluster-horizontal-scaling/manual-scale-method.png)

### <a name="optimized-autoscale-preview"></a>优化自动缩放（预览版）

建议使用自动缩放方法。 此方法优化群集性能和成本。 如果群集接近利用率的状态，它将在中进行缩放。 此操作可降低成本，但会降低性能级别。 如果群集接近利用率状态，则会将其扩展以保持最佳性能。 配置优化的自动缩放：

1. 选择**优化的自动缩放**。 

1. 选择 "最小实例计数" 和 "最大实例计数"。 群集自动缩放在这两个数字之间的范围取决于负载。

1. 选择“保存”。

   ![优化的自动缩放方法](media/manage-cluster-horizontal-scaling/optimized-autoscale-method.png)

优化的自动缩放开始工作。 现在，群集的 Azure 活动日志中会显示其操作。

#### <a name="logic-of-optimized-autoscale"></a>优化的自动缩放的逻辑 

**横向扩展**

当群集接近利用率的状态时，可以横向扩展以保持最佳性能。 向外扩展将在以下情况下发生：
* 群集实例的数量低于用户定义的最大实例数。
* 缓存利用率长达一小时。

> [!NOTE]
> Scale out 逻辑目前不考虑引入利用率和 CPU 指标。 如果这些指标对用例非常重要，则使用[自定义自动缩放](#custom-autoscale)。

**缩小**

当群集接近利用率的状态时，请将其缩小以降低成本，但保持性能。 使用多种指标来验证是否可以在群集中进行缩放。 以下规则将在执行 scale 后的7天内评估：
* 实例数超过2个，并且大于定义的最小实例数。
* 若要确保不会对资源进行任何重载，必须在执行 scale in 之前验证以下度量值： 
    * 缓存利用率不高
    * CPU 低于平均值 
    * 引入利用率低于平均值 
    * 流式引入利用率（如果使用的是流式传输）
    * 保持活动状态的事件高于定义的最小值、处理正确时间以及时间。
    * 无查询限制 
    * 失败的查询数低于定义的最小值。

> [!NOTE]
> 目前，在中，大规模规模要求先进行7天评估，然后才能实现中的优化规模。 此评估每24小时发生一次。 如果需要快速更改，请使用[手动缩放](#manual-scale)。

### <a name="custom-autoscale"></a>自定义自动缩放

使用自定义自动缩放，可以根据指定的指标动态缩放群集。 下图显示了用于配置自定义自动缩放的流和步骤。 有关更多详细信息，请参阅图形。

1. 在 "**自动缩放设置名称**" 框中，输入一个名称，如 "*横向扩展：缓存使用率*"。 

   ![缩放规则](media/manage-cluster-horizontal-scaling/custom-autoscale-method.png)

2. 对于**缩放模式**，请选择 **"基于指标缩放**"。 此模式提供动态缩放。 还可以选择 "**缩放到特定实例计数**"。

3. 选择 " **+ 添加规则**"。

4. 在右侧的 "**缩放规则**" 部分中，输入每个设置的值。

    **条件**

    | 设置 | 说明和值 |
    | --- | --- |
    | **时间聚合** | 选择聚合条件，例如“平均”。 |
    | **指标名称** | 选择要求缩放操作依赖的指标，例如“缓存使用率”。 |
    | 时间粒度统计信息 | 在 **Average**、**Minimum**、**Maximum** 和 **Sum** 之间选择。 |
    | 运算符 | 选择适当的选项，例如“大于或等于”。 |
    | **阈值** | 选择适当的值。 例如，对于缓存利用率，80% 是一个很好的起点。 |
    | **持续时间（分钟）** | 选择适当的时间值，以便系统在计算指标时进行回溯。 一开始可将默认值设置为 10 分钟。 |
    |  |  |

    **Action**

    | 设置 | 说明和值 |
    | --- | --- |
    | **操作** | 选择进行横向缩放的适当选项。 |
    | **实例计数** | 选择在符合指标条件的情况下，需要添加或删除的节点或实例的数目。 |
    | 冷却（分钟） | 选择需要在两项缩放操作之间等待的适当时间间隔。 一开始可将默认值设置为 5分钟。 |
    |  |  |

5. 选择 **添加** 。

6. 在左侧的 "**实例限制**" 部分中，输入每个设置的值。

    | 设置 | 说明和值 |
    | --- | --- |
    | **最低** | 群集在缩放时不管使用率如何都不得低于的实例数。 |
    | 最大值 | 群集在缩放时不管使用率如何都不得高于的实例数。 |
    | **默认** | 实例的默认数目。 如果读取资源指标时出现问题，则使用此设置。 |
    |  |  |

7. 选择“保存”。

现已为 Azure 数据资源管理器群集配置了横向缩放。 添加另一个规则以进行垂直缩放。 如果需要有关群集缩放问题的帮助，请在 Azure 门户中[打开支持请求](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)。

## <a name="next-steps"></a>后续步骤

* [使用指标监视 Azure 数据资源管理器性能、运行状况和使用情况](using-metrics.md)
* [管理群集垂直缩放](manage-cluster-vertical-scaling.md)以适应群集的适当大小。
