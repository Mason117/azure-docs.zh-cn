---
title: 使用 Azure Cosmos DB 中的更改源处理器库
description: 了解如何使用 Azure Cosmos DB 更改源处理器库读取更改源、更改源处理器的组件以及实现生命周期
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 12/03/2019
ms.reviewer: sngun
ms.openlocfilehash: ea2b72e926923357e9ee7dac87720292fbbf9696
ms.sourcegitcommit: 9405aad7e39efbd8fef6d0a3c8988c6bf8de94eb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2019
ms.locfileid: "74872224"
---
# <a name="change-feed-processor-in-azure-cosmos-db"></a>Azure Cosmos DB 更改源处理器 

更改源处理器是[AZURE COSMOS DB SDK V3](https://github.com/Azure/azure-cosmos-dotnet-v3)的一部分。 它简化了读取更改源的过程，并有效地在多个使用者之间分发事件处理。

更改源处理器库的主要优点是其容错行为，可确保对更改源中的所有事件进行 "至少一次" 传递。

## <a name="components-of-the-change-feed-processor"></a>更改源处理器的组件

实现更改源处理器有四个主要组件： 

1. **监视的容器：** 监视的容器是用于生成更改源的数据。 受监视容器的任何插入和更新均反映在容器的更改源中。

1. **租赁容器：** 租约容器充当状态存储，并协调处理跨多个工作线程的更改源。 租赁容器可以存储在与监视容器相同的帐户中，也可以存储在单独的帐户中。 

1. **主机：** 主机是使用更改源处理器来侦听更改的应用程序实例。 具有相同租约配置的多个实例可以并行运行，但每个实例应具有不同的**实例名称**。 

1. **委托：** 委托是定义您（开发人员）要对更改源处理器读取的每批更改执行的操作的代码。 

若要进一步了解更改源处理器的四个元素是如何协同工作的，请看下图中的一个示例。 监视的容器存储文档，并使用 "City" 作为分区键。 我们发现分区键值分布在包含项的范围内。 有两个主机实例，更改源处理器向每个实例分配不同范围的分区键值，以最大程度地提高计算分布。 每个范围都是并行读取的，它的进度与租赁容器中的其他范围分开维护。

![更改源处理器示例](./media/change-feed-processor/changefeedprocessor.png)

## <a name="implementing-the-change-feed-processor"></a>实现更改源处理器

从调用 `GetChangeFeedProcessorBuilder`的 `Container` 实例中，入口点始终是监视的容器：

[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=DefineProcessor)]

其中第一个参数是描述此处理器的目标的唯一名称，第二个参数是将处理更改的委托实现。 

委托的示例如下：

[!code-csharp[Main](~/samples-cosmosdb-dotnet-change-feed-processor/src/Program.cs?name=Delegate)]

最后，使用 `WithInstanceName` 定义此处理器实例的名称，该名称是用于维护 `WithLeaseContainer`的租约状态的容器。

调用 `Build` 将为你指定可通过调用 `StartAsync`来启动的处理器实例。

## <a name="processing-life-cycle"></a>处理生命周期

主机实例的正常生命周期如下：

1. 读取更改源。
1. 如果没有任何更改，则睡眠时间为预定义的时间长度（可在生成器中使用 `WithPollInterval` 进行自定义）并中转到 #1。
1. 如果有更改，请将其发送到**委托**。
1. 当委托**成功**处理更改后，请用最新的已处理时间点更新租约存储，并中转到 #1。

## <a name="error-handling"></a>错误处理。

更改源处理器可复原用户代码错误。 这意味着，如果委托实现具有未经处理的异常（步骤 #4），则将停止处理特定批更改的线程，并将创建一个新线程。 新线程将进行检查，该时间点是租约存储区的最新时间点，并从该处重新开始，从而有效地将同一批更改发送到委托。 此行为将继续，直到你的委托正确地处理更改，这是因为如果委托代码引发，它将会重试该批处理。

## <a name="dynamic-scaling"></a>动态缩放

如简介中所述，更改源处理器可自动将计算分布到多个实例中。 你可以使用更改源处理器部署应用程序的多个实例，并利用它，唯一的关键要求是：

1. 所有实例都应具有相同的租赁容器配置。
1. 所有实例都应具有相同的工作流名称。
1. 每个实例都需要具有不同的实例名称（`WithInstanceName`）。

如果这三个条件适用，则更改源处理器将使用相等的分布算法，将租用容器中的所有租约分散到所有正在运行的实例和并行计算。 一个租约只能在给定时间由一个实例拥有，因此，最大实例数等于租约数。

实例数可以增大和缩小，更改源处理器会根据相应的方式动态调整负载。

而且，更改源处理器可以动态调整到容器规模，因为吞吐量或存储量会增加。 当容器增长时，更改源处理器会以透明方式处理这些情况，方法是动态增加租约并在现有实例之间分配新的租约。

## <a name="change-feed-and-provisioned-throughput"></a>更改源和预配吞吐量

消耗的 RU 会产生费用，将数据移入和移出 Cosmos 容器始终会消耗 RU。 租约容器消耗的 RU 也会产生费用。

## <a name="additional-resources"></a>其他资源

* [Azure Cosmos DB SDK](sql-api-sdk-dotnet.md)
* [GitHub 上的用法示例](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [GitHub 上的其他示例](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>后续步骤

现在，可以通过以下文章继续详细了解更改源处理器：

* [更改源概述](change-feed.md)
* [如何从更改源处理器库迁移](how-to-migrate-from-change-feed-library.md)
* [使用更改源估算器](how-to-use-change-feed-estimator.md)
* [更改源处理器开始时间](how-to-configure-change-feed-start-time.md)
