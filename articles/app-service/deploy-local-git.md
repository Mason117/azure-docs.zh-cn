---
title: 从本地 Git 存储库部署
description: 了解如何实现从本地 Git 部署到 Azure 应用服务。 从本地计算机部署代码的最简单方法之一。
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.topic: article
ms.date: 06/18/2019
ms.reviewer: dariac
ms.custom: seodec18
ms.openlocfilehash: 91d5827a08a600c19c24ac0a96a5f4e3e98e22f2
ms.sourcegitcommit: 265f1d6f3f4703daa8d0fc8a85cbd8acf0a17d30
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/02/2019
ms.locfileid: "74671768"
---
# <a name="local-git-deployment-to-azure-app-service"></a>要 Azure App Service 的本地 Git 部署

本操作方法指南演示了如何将应用部署到本地计算机上的 Git 存储库中的[Azure App Service](overview.md) 。

## <a name="prerequisites"></a>必备组件

按照本操作方法指南中的步骤操作：

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- [安装 Git](https://www.git-scm.com/downloads)。
  
- 有一个本地 Git 存储库，其中包含要部署的代码。 若要下载示例存储库，请在本地终端窗口中运行以下命令：
  
  ```bash
  git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
  ```

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-with-kudu-build-server"></a>与 Kudu 生成服务器一起部署

若要为应用程序使用 Kudu 应用服务生成服务器启用本地 Git 部署，最简单的方法是使用 Azure Cloud Shell。 

### <a name="configure-a-deployment-user"></a>配置部署用户

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="get-the-deployment-url"></a>获取部署 URL

若要获取 URL 以对现有应用启用本地 Git 部署，请在 Cloud Shell 中运行[`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source?view=azure-cli-latest#az-webapp-deployment-source-config-local-git) 。 将 \<应用名称 > 和 \<组名称 > 替换为应用及其 Azure 资源组的名称。

```azurecli-interactive
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```

或者，若要创建新的启用 Git 的应用，请使用 `--deployment-local-git` 参数在 Cloud Shell 中运行[`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create) 。 将 \<应用名称 >、\<组名称 > 和 \<计划名称 > 替换为新 Git 应用的名称、其 Azure 资源组和 Azure App Service 计划。

```azurecli-interactive
az webapp create --name <app-name> --resource-group <group-name> --plan <plan-name> --deployment-local-git
```

任一命令返回如下 URL： `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git`。 使用此 URL 在下一步中部署应用。

你还可以使用应用级凭据启用本地 Git，而不是使用此帐户级 URL。 Azure App Service 会自动为每个应用生成这些凭据。 

在 Cloud Shell 中运行以下命令，获取应用凭据。 将 \<应用名称 > 和 \<组名称 > 替换为应用的名称和 Azure 资源组名称。

```azurecli-interactive
az webapp deployment list-publishing-credentials --name <app-name> --resource-group <group-name> --query scmUri --output tsv
```

使用返回的 URL 在下一步中部署应用。

### <a name="deploy-the-web-app"></a>部署 Web 应用

1. 打开本地 Git 存储库的本地终端窗口，并添加 Azure 远程。 在下面的命令中，将 \<url > 替换为你在上一步中获取的部署用户特定的 URL 或应用特定的 URL。
   
   ```bash
   git remote add azure <url>
   ```
   
1. 通过 `git push azure master`推送到 Azure 远程。 
   
1. 在 " **Git 凭据管理器**" 窗口中，输入[部署用户密码](#configure-a-deployment-user)，而不是 Azure 登录密码。
   
1. 查看输出。 你可能会看到特定于运行时的自动化，如 MSBuild for ASP.NET、用于 node.js 的 `npm install` 和 Python `pip install`。 
   
1. 在 Azure 门户中浏览到应用，以验证是否已部署内容。

## <a name="deploy-with-azure-pipelines-builds"></a>部署 Azure Pipelines 生成

如果你的帐户具有所需的权限，则可以设置 Azure Pipelines （预览版），以便为你的应用启用本地 Git 部署。 

- 你的 Azure 帐户必须具有写入 Azure Active Directory 和创建服务的权限。 
  
- 你的 Azure 帐户必须拥有 Azure 订阅中的 "**所有者**" 角色。

- 你必须是要使用的 Azure DevOps 项目中的管理员。

使用 Azure Pipelines （预览版）为应用启用本地 Git 部署：

1. 导航到[Azure 门户](https://portal.azure.com)中的 Azure App Service 应用页面，然后在左侧菜单中选择 "**部署中心**"。
   
1. 在 "**部署中心**" 页上，选择 "**本地 Git**"，然后选择 "**继续**"。 
   
   ![选择 "本地 Git"，然后选择 "继续"](media/app-service-deploy-local-git/portal-enable.png)
   
1. 在 "**生成提供程序**" 页上，选择 " **Azure Pipelines （预览）** "，然后选择 "**继续**"。 
   
   ![选择 Azure Pipelines （预览），然后选择 "继续"。](media/app-service-deploy-local-git/pipeline-builds.png)

1. 在 "**配置**" 页上，配置新的 Azure DevOps 组织或指定现有组织，然后选择 "**继续**"。
   
   > [!NOTE]
   > 如果未列出现有的 Azure DevOps 组织，可能需要将其链接到 Azure 订阅。 有关详细信息，请参阅[定义 CD 发布管道](/azure/devops/pipelines/apps/cd/deploy-webdeploy-webapps#cd)。
   
1. 根据你的应用服务计划[定价层](https://azure.microsoft.com/pricing/details/app-service/plans/)，你可能会看到 "**部署到过渡**" 页。 选择是否[启用部署槽位](deploy-staging-slots.md)，然后选择 "**继续**"。
   
1. 在 "**摘要**" 页上，查看设置，然后选择 "**完成**"。
   
1. Azure 管道准备就绪后，从**部署中心**页面复制 Git 存储库 URL，以便在下一步中使用。 
   
   ![复制 Git 存储库 URL](media/app-service-deploy-local-git/vsts-repo-ready.png)

1. 在本地终端窗口中，将 Azure 远程计算机添加到本地 Git 存储库。 在命令中，将 \<url > 替换为你在上一步中获取的 Git 存储库的 URL。
   
   ```bash
   git remote add azure <url>
   ```
   
1. 通过 `git push azure master`推送到 Azure 远程。 
   
1. 在 " **Git 凭据管理器**" 页上，用 visualstudio.com 用户名登录。 有关其他身份验证方法，请参阅[Azure DevOps Services authentication 概述](/vsts/git/auth-overview?view=vsts)。
   
1. 部署完成后，请在 `https://<azure_devops_account>.visualstudio.com/<project_name>/_build`中查看生成进度，并在 `https://<azure_devops_account>.visualstudio.com/<project_name>/_release`上查看部署进度。
   
1. 在 Azure 门户中浏览到应用，以验证是否已部署内容。

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="troubleshoot-deployment"></a>排查部署问题

使用 Git 发布到 Azure 中的应用服务应用时，可能会看到以下常见错误消息：

|消息|原因|分辨率
---|---|---|
|`Unable to access '[siteURL]': Failed to connect to [scmAddress]`|应用未启动并正在运行。|在 Azure 门户中启动应用。 Web 应用停止后，Git 部署不可用。|
|`Couldn't resolve host 'hostname'`|"Azure" 远程地址信息不正确。|使用 `git remote -v` 命令列出所有远程和关联的 URL。 确认“azure”远程网站的 URL 正确。 如果需要，请删除此远程网站并使用正确的 URL 重新创建它。|
|`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'master'.`|在 `git push`期间未指定分支，或者未在 `.gitconfig`中设置 `push.default` 值。|再次运行 `git push`，并指定主分支： `git push azure master`。|
|`src refspec [branchname] does not match any.`|尝试推送到 "azure" 远程上的 "master" 之外的分支。|再次运行 `git push`，并指定主分支： `git push azure master`。|
|`RPC failed; result=22, HTTP code = 5xx.`|如果尝试通过 HTTPS 推送大型 Git 存储库，则可能出现此错误。|更改本地计算机上的 git 配置，使 `postBuffer` 更大。 例如：`git config --global http.postBuffer 524288000`。|
|`Error - Changes committed to remote repository but your web app not updated.`|使用_package_文件（该文件）部署了一个 node.js 应用，该文件指定了其他所需的模块。|检查此错误之前的 `npm ERR!` 错误消息，以获取有关失败的更多上下文。 下面是此错误的已知原因和相应的 `npm ERR!` 消息：<br /><br />**包文件格式错误**： `npm ERR! Couldn't read dependencies.`<br /><br />**本机模块没有适用于 Windows 的二进制分发**：<br />`npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1` <br />或 <br />`npm ERR! [modulename@version] preinstall: \make || gmake\ `|

## <a name="additional-resources"></a>其他资源

- [项目 Kudu 文档](https://github.com/projectkudu/kudu/wiki)
- [持续部署到 Azure App Service](deploy-continuous-deployment.md)
- [示例：从本地 Git 存储库创建 web 应用并部署代码（Azure CLI）](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
- [示例：从本地 Git 存储库创建 web 应用并部署代码（PowerShell）](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
