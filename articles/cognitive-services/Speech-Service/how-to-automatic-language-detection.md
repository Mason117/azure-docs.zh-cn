---
title: 如何将自动语言检测用于语音转换到文本
titleSuffix: Azure Cognitive Services
description: 语音 SDK 支持语音到文本的自动语言检测。 使用此功能时，所提供的音频将与提供的语言列表进行比较，并确定最可能的匹配项。 然后，可以使用返回的值来选择用于语音转换到文本的语言模型。
services: cognitive-services
author: susanhu
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/26/2019
ms.author: qiohu
ms.openlocfilehash: a72f477e64c856c545801533c131c397de627c00
ms.sourcegitcommit: 598c5a280a002036b1a76aa6712f79d30110b98d
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/15/2019
ms.locfileid: "74110179"
---
# <a name="automatic-language-detection-for-speech-to-text"></a>语音到文本的自动语言检测

当与所提供的语言列表进行比较时，自动语言检测用于确定传递到语音 SDK 的音频最可能的匹配项。 然后，将使用自动语言检测返回的值选择语音转换到文本的语言模型，从而提供更准确的脚本。 若要查看可用的语言，请参阅[语言支持](language-support.md)。

在本文中，你将了解如何使用 `AutoDetectSourceLanguageConfig` 来构造 `SpeechRecognizer` 对象并检索检测到的语言。

> [!IMPORTANT]
> 此功能仅适用于适用于的语音 SDK C++和适用于 Java 的语音 sdk。

## <a name="automatic-language-detection-with-the-speech-sdk"></a>通过语音 SDK 进行自动语言检测

自动语言检测当前在每次检测中都有两种语言的服务端限制。 构造 `AudoDetectSourceLanguageConfig` 对象时，请记住此限制。 在下面的示例中，你将创建一个 `AutoDetectSourceLanguageConfig`，然后使用它来构造 `SpeechRecognizer`。

> [!TIP]
> 您还可以指定在执行语音转换到文本时要使用的自定义模型。 有关详细信息，请参阅[将自定义模型用于自动语言检测](#use-a-custom-model-for-automatic-language-detection)。

以下代码片段演示如何在应用中使用自动语言检测：

```C++
auto autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "de-DE" });
auto recognizer = SpeechRecognizer::FromConfig(speechConfig, autoDetectSourceLanguageConfig, audioConfig);
speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto autoDetectSourceLanguageResult = AutoDetectSourceLanguageResult::FromResult(speechRecognitionResult);
auto detectedLanguage = autoDetectSourceLanguageResult->Language;
```

```Java
AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig.fromLanguages(Arrays.asList("en-US", "de-DE"));
SpeechRecognizer recognizer = new SpeechRecognizer(speechConfig, autoDetectSourceLanguageConfig, audioConfig);
Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
AutoDetectSourceLanguageResult autoDetectSourceLanguageResult = AutoDetectSourceLanguageResult.fromResult(result);
String detectedLanguage = autoDetectSourceLanguageResult.getLanguage();

recognizer.close();
speechConfig.close();
autoDetectSourceLanguageConfig.close();
audioConfig.close();
result.close();
```

## <a name="use-a-custom-model-for-automatic-language-detection"></a>使用自定义模型进行自动语言检测

除了使用语音服务模型进行语言检测以外，还可以指定用于增强识别的自定义模型。 如果未提供自定义模型，该服务将使用默认语言模型。

下面的代码段演示了如何在对语音服务的调用中指定自定义模型。 如果 `en-US`检测到的语言，则使用默认模型。 如果 `fr-FR`检测到的语言，则使用自定义模型的终结点：

```C++
std::vector<std::shared_ptr<SourceLanguageConfig>> sourceLanguageConfigs;
sourceLanguageConfigs.push_back(SourceLanguageConfig::FromLanguage("en-US"));
sourceLanguageConfigs.push_back(SourceLanguageConfig::FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));
auto autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig::FromSourceLanguageConfigs(sourceLanguageConfigs);
```

```Java
List sourceLanguageConfigs = new ArrayList<SourceLanguageConfig>();
sourceLanguageConfigs.add(SourceLanguageConfig.fromLanguage("en-US"));
sourceLanguageConfigs.add(SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));
AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig = AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs(sourceLanguageConfigs);
```

## <a name="next-steps"></a>后续步骤

- [语音 SDK 参考文档](speech-sdk.md)
