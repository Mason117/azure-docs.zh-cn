---
title: 跟踪团队数据科学流程项目的进度
description: 数据科学组管理员、团队主管和项目主管如何可以跟踪数据科学项目的进度。
author: marktab
manager: cgronlun
editor: cgronlun
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 09/26/2019
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 28bd3e558294c000ba65a1c60fe227bbae7e82dd
ms.sourcegitcommit: 824e3d971490b0272e06f2b8b3fe98bbf7bfcb7f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/10/2019
ms.locfileid: "72244053"
---
# <a name="track-the-progress-of-data-science-projects"></a>跟踪数据科学项目的进度

数据科学组管理员、团队主管和项目主管可以跟踪其项目的进度，如完成了哪些工作、工作人员工作以及剩余的工作。 

## <a name="azure-devops-dashboards"></a>Azure DevOps 仪表板

如果你使用的是 Azure DevOps，则可以构建仪表板来跟踪与给定 Agile 项目关联的活动和工作项。 有关仪表板的详细信息，请参阅[仪表板、报表和小组件](/azure/devops/report/dashboards/)。

有关如何在 Azure DevOps 中创建和自定义仪表板和小组件的说明，请参阅以下快速入门：

- [添加和管理仪表板](/azure/devops/report/dashboards/dashboards)
- [将小组件添加到仪表板](/azure/devops/report/dashboards/add-widget-to-dashboard)

## <a name="example-dashboard"></a>示例仪表板

下面是一个简单的示例仪表板，用于跟踪 Agile 数据科学项目的冲刺（sprint）活动，包括与关联的存储库的提交数。 

- **倒计时**磁贴显示当前冲刺（sprint）中保留的天数。 

- 这两个**代码磁贴**显示过去七天内两个项目存储库中的提交数。 

- "**用于 TDSP 客户的工作项" 项目**显示所有工作项及其状态的查询结果。 

- **累积流关系图**（CFD）显示已关闭和活动的工作项的数目。

- **燃尽图**显示仍在冲刺（sprint）中剩余时间完成的工作。

- **燃耗图**显示了已完成的工作，与冲刺（sprint）中的总工作量相比较。

![仪表板](./media/track-progress/dashboard.png)

## <a name="next-steps"></a>后续步骤

[执行团队数据科学过程的演练](walkthroughs.md)列出了演示特定方案过程中的所有步骤的演练，其中包含链接和缩略图说明。 链接的方案说明了如何将云和本地工具和服务合并到工作流或管道中，以创建智能应用程序。 
