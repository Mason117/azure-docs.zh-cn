---
title: include 文件
description: include 文件
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.topic: include
ms.date: 02/11/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 661849d2d5c42db70d366e48e4138008bc18b720
ms.sourcegitcommit: 5acd8f33a5adce3f5ded20dff2a7a48a07be8672
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/24/2019
ms.locfileid: "72902114"
---
## <a name="supported-operating-systems-and-drivers"></a>支持的操作系统和驱动程序

### <a name="nvidia-tesla-cuda-drivers"></a>NVIDIA Tesla (CUDA) 驱动程序

仅下表中列出的操作系统支持适用于 NC、NCv2、NCv3、ND 和 NDv2 系列 VM 的 NVIDIA Tesla (CUDA) 驱动程序（对 NV 系列为可选项）。 在本文发布时，驱动程序下载链接是最新的。 有关最新驱动程序，请访问 [NVIDIA](https://www.nvidia.com/) 网站。

> [!TIP]
> 作为一种在 Windows Server VM 上手动安装 CUDA 驱动程序的替代方法，可以部署 Azure [数据科学虚拟机](../articles/machine-learning/data-science-virtual-machine/overview.md)映像。 用于 Windows Server 2016 的 DSVM 版本预安装 NVIDIA CUDA 驱动程序、CUDA 深度神经网络库和其他工具。


| OS | 驱动程序 |
| -------- |------------- |
| Windows Server 2016 | [398.75](https://us.download.nvidia.com/Windows/Quadro_Certified/398.75/398.75-tesla-desktop-winserver2016-international.exe) (.exe) |
| Windows Server 2012 R2 | [398.75](https://us.download.nvidia.com/Windows/Quadro_Certified/398.75/398.75-tesla-desktop-winserver2008-2012r2-64bit-international.exe) (.exe) |

### <a name="nvidia-grid-drivers"></a>NVIDIA GRID 驱动程序

Microsoft 为用作虚拟工作站或虚拟应用程序的 NV 和 NVv3 系列 Vm 重新分发 NVIDIA 网格驱动程序安装程序。 仅在 Azure NV 系列 Vm 上安装这些网格驱动程序，且仅在下表中列出的操作系统上安装。 这些驱动程序包括 Azure 中 GRID Virtual GPU Software 的许可。 无需设置 NVIDIA vGPU 软件许可证服务器。

请注意，Nvidia 扩展将始终安装 latst 驱动程序。 我们在此处提供了与旧版本相关的客户的以前版本的链接。

| OS | 驱动程序 |
| -------- |------------- |
| Windows Server 2019<br/><br/>Windows Server 2016<br/><br/>Windows 10 | [网格9.1 （431.79）](https://go.microsoft.com/fwlink/?linkid=874181) （.exe） <br/><br/> [网格9.0 （431.02）](https://download.microsoft.com/download/8/C/C/8CC88D54-EB07-44D3-8FA9-B797B173ED04/431.02_grid_win10_server2016_server2019_64bit_international.exe) （.exe）  |
| Windows Server 2012 R2<br/><br/>Windows Server 2008 R2<br/><br/>Windows 8<br/><br/>Windows 7 | [网格9.1 （431.79）](https://go.microsoft.com/fwlink/?linkid=874184) （.exe）<br/><br/> [网格9.0 （431.02）](https://download.microsoft.com/download/B/D/6/BD6D1C34-E41F-4654-B5AB-F4DA9C08AA64/431.02_grid_win7_win8_server2008R2_server2012R2_64bit_international.exe) （.exe）  |
