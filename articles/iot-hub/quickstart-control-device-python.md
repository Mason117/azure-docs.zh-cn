---
title: 从 Azure IoT 中心控制设备快速入门 (Python) | Microsoft Docs
description: 在本快速入门中，会运行两个示例 Python 应用程序。 一个为后端应用程序，可远程控制连接到中心的设备。 另一个应用程序可模拟连接到中心的可受远程控制的设备。
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: quickstart
ms.custom: mvc
ms.date: 06/21/2019
ms.openlocfilehash: b36e5d88c67a4aabf530aa8d945c17870e9c126b
ms.sourcegitcommit: 8bd85510aee664d40614655d0ff714f61e6cd328
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/06/2019
ms.locfileid: "74892645"
---
# <a name="quickstart-control-a-device-connected-to-an-iot-hub-python"></a>快速入门：控制连接到 IoT 中心的设备 (Python)

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

IoT 中心是一项 Azure 服务，使你可以从云管理 IoT 设备，并将大量设备遥测引入云以进行存储或处理。 在本快速入门中，会使用直接方法来控制连接到 IoT 中心的模拟设备  。 可使用直接方法远程更改连接到 IoT 中心的设备的行为。

本快速入门使用两个预先编写的 Python 应用程序：

* 从后端应用程序调用的可响应直接方法的模拟设备应用程序。 为了接收直接方法调用，此应用程序会连接到 IoT 中心上特定于设备的终结点。

* 后端应用程序，可在模拟设备上调用直接方法。 为了在设备上调用直接方法，此应用程序会连接到 IoT 中心上的服务端终结点。

> [!IMPORTANT]
> 在本文中，后端应用程序使用 Python V1 服务客户端，设备应用程序使用 Python V2 设备客户端。 V1 服务客户端位于 Azure IoT Python SDK GitHub 存储库的 [v1-deprecated 分支](https://github.com/Azure/azure-iot-sdk-python/tree/v1-deprecated)中。 V1 服务客户端 *azure-iothub-service-client* 的 Pip 包具有严格的特定于平台的要求 -- 包括开发计算机上安装的 Python 版本。 **先决条件**部分注明了这些要求。
>

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果还没有 Azure 订阅，可以在开始前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="prerequisites"></a>先决条件

运行以下命令将用于 Azure CLI 的 Microsoft Azure IoT 扩展添加到 Cloud Shell 实例。 IOT 扩展会将 IoT 中心、IoT Edge 和 IoT 设备预配服务 (DPS) 特定的命令添加到 Azure CLI。

```azurecli-interactive
az extension add --name azure-cli-iot-ext
```

如果尚未进行此操作，请从 https://github.com/Azure-Samples/azure-iot-samples-python/archive/master.zip 下载示例 Python 项目并提取 ZIP 存档。

**对于 Windows**，必须满足以下先决条件才能安装 V1 IoT 中心服务客户端 Pip 包：

* 确保已安装 [Python 版本 **3.6.x**](https://www.python.org/downloads/)。

* 确保已安装 [Microsoft Visual C++ Redistributable for Visual Studio](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)。

**对于非 Windows 平台**，请参阅 V1 SDK 文档中的 [Python Pip 包分发表](https://github.com/Azure/azure-iot-sdk-python/blob/v1-deprecated/doc/python-devbox-setup.md#python-pip-package-distribution-table)。 确保已在开发计算机上安装了为平台指定的 Python 3.x 版本并满足任何相关要求。 安装 Python 3.x 而不是 2.7 可在 V2 设备客户端（本快速入门也使用此客户端）中启用异步操作。

## <a name="create-an-iot-hub"></a>创建 IoT 中心

如果已完成上一[快速入门：将遥测数据从设备发送到 IoT 中心](quickstart-send-telemetry-python.md)，则可以跳过此步骤。

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>注册设备

如果已完成上一[快速入门：将遥测数据从设备发送到 IoT 中心](quickstart-send-telemetry-python.md)，则可以跳过此步骤。

必须先将设备注册到 IoT 中心，然后该设备才能进行连接。 在本快速入门中，将使用 Azure Cloud Shell 来注册模拟设备。

1. 在 Azure Cloud Shell 中运行以下命令，以创建设备标识。

    **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    **MyPythonDevice**：这是所注册的设备的名称。 建议使用 **MyPythonDevice**，如图所示。 如果为设备选择不同名称，则可能还需要在本文中从头至尾使用该名称，并在运行示例应用程序之前在其中更新设备名称。

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyPythonDevice
    ```

2. 在 Azure Cloud Shell 中运行以下命令，以获取刚注册设备的_设备连接字符串_：

    **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyPythonDevice --output table
    ```

    记下如下所示的设备连接字符串：

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    稍后会在快速入门中用到此值。

3. 还需一个服务连接字符串，以便后端应用程序能够连接到 IoT 中心并检索消息  。 以下命令检索 IoT 中心的服务连接字符串：

    **YourIoTHubName**：将下面的占位符替换为你为 IoT 中心选择的名称。

    ```azurecli-interactive
    az iot hub show-connection-string \
      --policy-name service \
      --name {YourIoTHubName} \
      --output table
    ```

    记下如下所示的服务连接字符串：

   `HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

    稍后会在快速入门中用到此值。 此服务连接字符串与你在上一步中记录的设备连接字符串不同。

## <a name="listen-for-direct-method-calls"></a>侦听直接方法调用

模拟设备应用程序会连接到 IoT 中心上特定于设备的终结点，发送模拟遥测数据，并侦听中心的直接方法调用。 在本快速入门中，中心的直接方法调用告知设备对其发送遥测的间隔进行更改。 执行直接方法后，模拟设备会将确认发送回中心。

1. 在本地终端窗口中，导航到示例 Python 项目的根文件夹。 然后导航到 **iot-hub\Quickstarts\simulated-device-2** 文件夹。

1. 在所选文本编辑器中打开 SimulatedDevice.py 文件  。

    将 `CONNECTION_STRING` 变量的值替换为之前记下的设备连接字符串。 然后将更改保存到 **SimulatedDevice.py**。

1. 在本地终端窗口中，运行以下命令，为模拟设备应用程序安装所需的库：

    ```cmd/sh
    pip install azure-iot-device
    ```

1. 在本地终端窗口中，运行以下命令，以便运行模拟设备应用程序：

    ```cmd/sh
    python SimulatedDevice.py
    ```

    以下屏幕截图显示了模拟设备应用程序将遥测数据发送到 IoT 中心后的输出：

    ![运行模拟设备](./media/quickstart-control-device-python/SimulatedDevice-1.png)

## <a name="call-the-direct-method"></a>调用直接方法

后端应用程序会连接到 IoT 中心上的服务端终结点。 应用程序通过 IoT 中心对设备进行直接方法调用，并侦听确认。 IoT 中心后端应用程序通常在云中运行。

1. 在其他本地终端窗口中，导航到示例 Python 项目的根文件夹。 然后导航到 iot-hub\Quickstarts\back-end-application 文件夹  。

1. 在所选文本编辑器中打开 BackEndApplication.py 文件  。

    将 `CONNECTION_STRING` 变量的值替换为以前记下的服务连接字符串。 然后将更改保存到 **BackEndApplication.py**。

1. 在本地终端窗口中，运行以下命令，为模拟设备应用程序安装所需的库：

    ```cmd/sh
    pip install azure-iothub-service-client future
    ```

1. 在本地终端窗口中，运行以下命令，以便运行终端应用程序：

    ```cmd/sh
    python BackEndApplication.py
    ```

    以下屏幕截图显示了应用程序对设备进行直接方法调用并接收确认后的输出：

    ![运行后端应用程序](./media/quickstart-control-device-python/BackEndApplication.png)

    运行后端应用程序后，在运行模拟设备的控制台窗口中会出现一条消息，且其发送消息的速率也会发生变化：

    ![模拟客户端的变化](./media/quickstart-control-device-python/SimulatedDevice-2.png)

    > [!NOTE]
    > 如果在导入 *iothub_service_client* 时出现错误，请确保已安装在[先决条件](#prerequisites)中为平台指定的相同 Python 版本和任何其他相关项目。 如果在验证先决条件之后仍出现错误，可能需要为平台生成服务客户端。 若要了解如何为平台生成 SDK，请参阅 V1 SDK 文档中的 [devbox 安装说明](https://github.com/Azure/azure-iot-sdk-python/blob/v1-deprecated/doc/python-devbox-setup.md)。
    >

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>后续步骤

在本快速入门中，已从后端应用程序调用了设备上的直接方法，并在模拟设备应用程序中响应了直接方法调用。

若要了解如何将设备到云的消息路由到云中的不同目标，请继续学习下一教程。

> [!div class="nextstepaction"]
> [教程：将遥测路由到不同的终结点进行处理](tutorial-routing.md)