---
title: 评审用户话语 - LUIS
titleSuffix: Azure Cognitive Services
description: 查看活动学习捕获的最谈话，以选择目的并标记最谈话的实体。接受更改、定型和发布。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/15/2019
ms.author: diberry
ms.openlocfilehash: 67953f552b5b2bcdd7d13253548227e57dab8548
ms.sourcegitcommit: 2d3740e2670ff193f3e031c1e22dcd9e072d3ad9
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2019
ms.locfileid: "74132673"
---
# <a name="how-to-improve-the-luis-app-by-reviewing-endpoint-utterances"></a>如何通过查看端点最谈话改善 LUIS 应用

查看终结点最谈话进行正确预测的过程称为[活动学习](luis-concept-review-endpoint-utterances.md)。 主动学习捕获终结点查询并选择用户不确定的终结点话语。 你审核这些话语以选择意向并标记这些真实话语的实体。 在示例话语中接受这些更改，然后对其进行训练和发布。 然后 LUIS 更准确地识别话语。

如果有很多人参与了 LUIS 的应用程序， 

[!INCLUDE [Uses preview portal](includes/uses-portal-preview.md)]

## <a name="enable-active-learning"></a>启用主动学习

若要启用主动学习，必须记录用户查询。 这是通过使用 `log=true` querystring 参数和值调用[终结点查询](luis-get-started-create-app.md#query-the-v3-api-prediction-endpoint)来实现的。

## <a name="correct-intent-predictions-to-align-utterances"></a>更正意向预测以使最谈话

每个陈述的“统一意向”列中都显示一个建议意向。 

> [!div class="mx-imgBorder"]
> [![查看 LUIS 不能确定的终结点最谈话](./media/label-suggested-utterances/review-endpoint-utterances.png)](./media/label-suggested-utterances/review-endpoint-utterances.png#lightbox)

如果同意该意向，请选择复选标记。 如果不同意建议，请从统一意向下拉列表中选择合适的意向，然后选择统一意向右侧的复选标记。 选中复选标记后，查询文本将移到该意向，并从**查看终结点最谈话**列表中删除。 

> [!TIP]
> 若要查看并更正 "**查看终结点最谈话**" 列表中的所有示例最谈话中的实体预测，这一点很重要。

## <a name="delete-utterance"></a>删除陈述

可删除评审列表中的每个陈述。 删除后，该陈述将不再出现在列表中。 即使用户从终结点中输入同一陈述，结果也是如此。 

如果不确定是否应删除查询文本，请将其移到 "无" 目的，或创建新意向（如 `miscellaneous`），然后将查询文本移到该意向。 

## <a name="disable-active-learning"></a>禁用主动学习

若要禁用主动学习，请不要记录用户查询。 这是通过使用 `log=false` querystring 参数和值设置[终结点查询](luis-get-started-create-app.md#query-the-v2-api-prediction-endpoint)来实现的，或者不使用 querystring 值，因为默认值为 false。

## <a name="next-steps"></a>后续步骤

若要测试标记建议陈述后的性能提升情况，可通过选择顶部面板中的“测试”访问测试控制台。 有关如何使用测试控制台测试应用的说明，请参阅[训练和测试应用](luis-interactive-test.md)。
