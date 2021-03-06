---
title: Azure Cosmos DB 全局分布 - 揭秘
description: 本文提供 Azure Cosmos DB 全局分布相关的技术详细信息
author: SnehaGunda
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 12/02/2019
ms.author: sngun
ms.reviewer: sngun
ms.openlocfilehash: a46a69476a2ad6550bc7b3a533fd09565d461db3
ms.sourcegitcommit: 9405aad7e39efbd8fef6d0a3c8988c6bf8de94eb
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/05/2019
ms.locfileid: "74872122"
---
# <a name="global-data-distribution-with-azure-cosmos-db---under-the-hood"></a>Azure Cosmos DB 全局数据分布 - 揭秘

Azure Cosmos DB 是 Azure 中的一项基本服务，因此它已部署到全球所有 Azure 区域，包括公共、主权、国防部（DoD）和政府云。 在数据中心内，我们会在大量的计算机阵列上部署和管理 Azure Cosmos DB，其中的每个服务都有专用的本地存储。 在数据中心内，Azure Cosmos DB 部署在许多群集之间，每个群集可能运行多代硬件。 群集中的计算机通常分布在10-20 的容错域中，以便在某个区域内实现高可用性。 下图显示了 Cosmos DB 全局分布系统拓扑：

![系统拓扑](./media/global-dist-under-the-hood/distributed-system-topology.png)

**Azure Cosmos DB 中的全局分布是全包式的：** 无论何时，只需单击几下鼠标或通过单个 API 调用以编程方式，就可以添加或删除与 Cosmos 数据库关联的地理区域。 Cosmos 数据库又包含一组 Cosmos 容器。 在 Cosmos DB 中，容器充当逻辑性的分布和缩放单元。 创建的集合、表和图形（在内部）只是 Cosmos 容器。 容器完全与架构无关，并为查询提供作用域。 Cosmos 容器中的数据在引入时会自动编制索引。 自动索引使用户能够在无需架构或索引管理的情况下查询数据，尤其是在全局分布式设置中。  

- 在给定区域中，容器中的数据是通过使用您提供的分区键分发的，由基础物理分区（*本地分发*）透明管理。  

- 每个物理分区也会跨地理区域进行复制（*全球分布*）。 

当使用 Cosmos DB 弹性的应用在 Cosmos 容器上缩放吞吐量或消耗更多存储时，Cosmos DB 会以透明方式跨所有区域处理分区管理操作（拆分、克隆、删除）。 无论缩放、分布或故障情况如何，Cosmos DB 都会在分布于全球任意个区域之间的容器中提供单个数据系统映像。  

如下图所示，容器中的数据分布在两个维度中-在某个区域内和跨区域，遍布世界各地：  

![物理分区](./media/global-dist-under-the-hood/distribution-of-resource-partitions.png)

物理分区由一组称为*副本集*的副本实现。 每台计算机承载数百个副本，这些副本对应于一组固定的进程中的各个物理分区，如上图所示。 对应于物理分区的副本在区域中群集与数据中心内的计算机之间进行动态定位和负载均衡。  

一个副本专属于一个 Azure Cosmos DB 租户。 每个副本托管 Cosmos DB [数据库引擎](https://www.vldb.org/pvldb/vol8/p1668-shukla.pdf)的实例，该实例管理资源以及关联的索引。 Cosmos 数据库引擎在基于 atom 记录序列（ARS）的类型系统上进行操作。 该引擎对于架构的概念是不可知的，它模糊了记录的结构和实例值之间的界限。 Cosmos DB 通过在引入时为所有内容自动编制索引来有效实现架构的完全不可知性，使用户无需处理架构或索引管理，即可查询其全球分布式数据。

Cosmos 数据库引擎包含多个组件，包括实现多种协调基元、语言运行时、查询处理器以及负责事务存储和数据索引的存储和索引子系统，分别. 为了提供持久性和高可用性，数据库引擎将在 SSD 中保存其数据和索引，并相应地将其复制到副本集中的数据库引擎实例之间。 更大的租户对应于更高的吞吐量和存储规模，并具有更大或更大的副本或两者。 该系统的每个组件是完全异步的 – 永远不会出现线程阻塞，每个线程执行生存期较短的工作，可避免任何不必要的线程切换。 速率限制和反压在从许可控制到所有 I/O 路径的整个堆栈中传播。 Cosmos 数据库引擎旨在利用细化并发，并在节约系统资源内操作时提供高吞吐量。

Cosmos DB 的全局分布依赖于两个关键抽象：*副本集*和*分区集*。 副本集是用于协调的模块化 Lego 块，分区集是一个或多个地理分散式物理分区的动态叠加层。 若要了解全局分布的工作原理，需要了解这两个关键抽象。 

## <a name="replica-sets"></a>副本集

物理分区具体化为一组分散在多个容错域之间的、名为“副本集”的自我托管动态负载均衡副本。 此集统一实现复制的状态机协议，使物理分区中的数据保持高度可用、持久且一致。 副本集成员身份*N*是动态的–它基于故障、管理操作和失败副本重新生成/恢复的时间，保留*NMin*和*n 每天*之间的波动。 复制协议还会根据成员身份的变化来重新配置读取和写入仲裁的大小。 若要统一分配分配给给定物理分区的吞吐量，请使用两个思路： 

- 首先，在领导者处理写入请求的成本高于在该应用程序上应用更新的成本。 相应地，为领先者预算的系统资源比后继者更多。 

- 其次，确保给定一致性级别的读取仲裁尽可能地专门由后继者副本组成。 除非有必要，否则我们会避免访问领先者来为读取提供服务。 对于 Cosmos DB 支持的[五个一致性模型](consistency-levels.md)，我们对基于仲裁的系统中的[负载和容量之间的负载和容量](https://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)关系进行了大量看法。  

## <a name="partition-sets"></a>分区集

一组物理分区，其中每个分区都是通过 Cosmos 数据库区域配置的，它们用于管理跨所有已配置区域复制的相同密钥集。 这种较高的协调基元称为 "*分区集*"-一种地理上分散的物理分区的动态覆盖，用于管理一组给定的密钥。 当给定的物理分区（副本集）在群集范围内时，分区集可跨越群集、数据中心和地理区域，如下图所示：  

![分区集](./media/global-dist-under-the-hood/dynamic-overlay-of-resource-partitions.png)

可将分区集视为地理分散的“超副本集”，它由多个拥有相同键集的副本集构成。 类似于副本集，分区集的成员身份也是动态的–它会根据隐式物理分区管理操作而波动，以便向/从给定的分区集添加/删除新分区（例如，当你在容器上横向扩展吞吐量、在 Cosmos 数据库中添加/删除区域或发生故障时）。 由于具有分区集的每个分区都在其自己的副本集内管理分区集成员身份，因此成员身份完全分散且高度可用。 在重新配置分区集期间，还会建立物理分区之间的叠加层拓扑。 根据一致性级别、地理距离以及源和目标物理分区之间的可用网络带宽，动态选择拓扑。  

该服务允许为 Cosmos 数据库配置一个或多个写入区域，根据所做的选择，分区集将配置为接受写入正好一个或所有区域。 该系统采用两级别嵌套共识协议 – 一个级别在接受写入的物理分区副本集的副本内运行，另一个级别在分区集的级别运行，以针对分区中提交的所有写入操作提供完全有序的保证。 此多层嵌套共识协议对于实现严格的高可用性 SLA，以及实现 Cosmos DB 为客户提供的一致性模型至关重要。  

## <a name="conflict-resolution"></a>冲突解决

我们在更新传播、冲突解决和因果关系跟踪的设计灵感来源于以往 [Epidemic 算法](https://www.cs.utexas.edu/~lorenzo/corsi/cs395t/04S/notes/naor98load.pdf)和 [Bayou](https://zoo.cs.yale.edu/classes/cs422/2013/bib/terry95managing.pdf) 系统工作的启发。 尽管这些思路的核心得以留存，并为传达 Cosmos DB 的系统设计提供方便的参考框架，但我们在将其应用于 Cosmos DB 系统时，还是对其做了重大改造。 此操作是必需的，因为以前的系统不是与资源调控一起设计的，也不是 Cosmos DB 需要操作的规模，也不提供功能（例如，有限过期一致性）和Cosmos DB 向其客户提供的综合 Sla。  

前面提到，分区集分布在多个区域之间，并遵循 Cosmos DB（多主数据库）复制协议在包含给定分区集的物理分区之间复制数据。 （分区集的）每个物理分区通常接受写入到该区域本地的客户端，并为读取操作提供服务。 区域中物理分区接受的写入操作在由客户端确认之前，将以持久方式进行提交并在物理分区中保持高可用性。 这些写入是试探性的，将使用反熵通道传播到分区集中的其他物理分区。 客户端可以通过传递请求标头来请求试探性写入或提交的写入。 反熵传播（包括传播频率）是动态的，基于分区集的拓扑、物理分区的区域邻近性，以及配置的一致性级别。 在分区集中，Cosmos DB 遵循采用动态选定仲裁器分区的主要提交方案。 仲裁器的选择是动态的，在基于叠加层拓扑重新配置分区集时，它是不可或缺的一部分。 可保证提交的写入（包括多行/批处理更新）的顺序。 

我们采用编码向量时钟（包含分别对应于每个副本集和分区集共识级别的区域 ID 和逻辑时钟）进行因果关系跟踪，并采用版本向量来检测和解决更新冲突。 拓扑和对等选择算法旨在确保版本向量的存储和网络开销固定保持在最低水平。 该算法可保证严格的融合属性。  

对于配置了多个写入区域的 Cosmos 数据库，该系统提供多种灵活的自动冲突解决策略供开发人员选择，包括： 

- **最后写入-Wins （LWW）** ，默认情况下，它使用系统定义的时间戳属性（基于时间同步时钟协议）。 Cosmos DB 还允许指定其他任何自定义数字属性用于解决冲突。  
- **应用程序定义（自定义）冲突解决策略**（通过合并过程表示），适用于应用程序定义的冲突语义协调。 在服务器端检测到数据库事务造成的写入-写入冲突时，将会调用这些过程。 系统仅提供一次保证合并过程作为承诺协议一部分执行的操作。 有[几个冲突解决示例](how-to-manage-conflicts.md)可供您使用。  

## <a name="consistency-models"></a>一致性模型

无论是使用单个还是多个写入区域配置 Cosmos 数据库，都可以选择五个定义完善的一致性模型。 对于多个写入区域，以下是一致性级别的几个值得注意的方面：  

有限过期一致性保证所有读取都将在任何区域中的最新写入数到*K*个前缀或*T*秒内。 而且，具有有限过期一致性的读取保证是单调的，并且具有一致的前缀保证。 反熵协议在运行时受到速率限制，确保前缀不会执行累加，并且不需要对写入应用反压。 会话一致性保证单调读取、单调写入、读取自己的写入、写入遵循读取和一致的前缀保证（在全球范围内）。 对于配置了强一致性的数据库，多个写入区域的好处（低写入延迟、高写入可用性）不适用，因为跨区域同步复制。

[此处](consistency-levels.md)介绍了 Cosmos DB 中的五种一致性模型的语义，并在[此处](https://github.com/Azure/azure-cosmos-tla)以数学方式说明了使用高级别 TLA + 规格。

## <a name="next-steps"></a>后续步骤

接下来，请通过以下文章了解如何配置全局分布：

* [在数据库帐户中添加/删除区域](how-to-manage-database-account.md#addremove-regions-from-your-database-account)
* [如何配置多宿主客户端](how-to-manage-database-account.md#configure-multiple-write-regions)
* [如何创建自定义冲突解决策略](how-to-manage-conflicts.md#create-a-custom-conflict-resolution-policy)
