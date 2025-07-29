---
lab:
  title: 将 Docker 容器部署到 Azure 应用服务 Web 应用
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# 将 Docker 容器部署到 Azure 应用服务 Web 应用

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://learn.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你是否拥有 Microsoft 帐户或 Microsoft Entra 帐户以及 Azure 订阅中的参与者或所有者角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)。

## 实验室概述

在本实验室中，你将学习如何使用 Azure DevOps CI/CD 管道生成自定义 Docker 映像，将其推送到 Azure 容器注册表，并将其作为容器部署到 Azure 应用服务。

## 目标

完成本实验室后，你将能够：

- 使用 Microsoft 托管的 Linux 代理生成自定义 Docker 映像。
- 向 Azure 容器注册表推送映像。
- 使用 Azure DevOps 将 Docker 映像作为容器部署到 Azure 应用服务。

## 预计用时：20 分钟

## 说明

### 练习 0：（如已完成，则跳过）配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb，然后在“工作项进程”下拉列表中选择“Scrum”。 单击“创建”。

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“Repos”>“文件存储”****、“导入”****。 在“导入 Git 存储库”窗口中，粘贴以下 URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> 并单击“导入”：

1. 存储库按以下方式组织：
    - .ado 文件夹包含 Azure DevOps YAML 管道。
    - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
    - infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
    - .github 文件夹容器 YAML GitHub 工作流定义。
    - src 文件夹包含用于实验室方案的 .NET 8 网站。****

#### 任务 3：（如果已完成，请跳过此任务）将主分支设置为默认分支

1. 转到“Repos”>“分支”****。
1. 将鼠标指针悬停在主分支上，然后单击列右侧的省略号。
1. 单击“设置为默认分支”。

### 练习 1：导入并运行 CI 管道

在本练习中，你将使用 Azure 订阅配置服务连接，然后导入并运行 CI 管道。

#### 任务 1：导入并运行 CI 管道

1. 转到“**管道 > 管道**”
1. 单击“新管道”按钮（如果之前没有创建其他管道，则单击“创建管道”）********
1. 选择“Azure Repos Git (YAML)”
1. 选择 eShopOnWeb 存储库
1. 选择**现有 Azure Pipelines YAML 文件**
1. 选择主分支和 /.ado/eshoponweb-ci-docker.yml 文件，然后单击“继续”************
1. 在 YAML 管道定义中，自定义：
   - **YOUR-SUBSCRIPTION-ID**，替换为你的 Azure 订阅 ID。
   - 将 **resourceGroup** 替换为在创建服务连接期间使用的资源组名称，例如 **AZ400-RG1**。

1. 查看管道定义。 CI 定义由以下任务构成：
    - 资源：它下载的存储库文件将用于后续任务。
    - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure 容器注册表。
    - PowerShell：从上一任务的输出中检索“ACR 登录服务器”值并创建新的参数 acrLoginServer
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- 生成**：生成 Docker 映像并创建两个标记（最新和当前 BuildID）
    - Docker - 推送：向 Azure 容器注册表推送映像

1. 单击“保存并运行”。

1. 打开管道执行。 如果看到警告消息“此管道需要访问资源的权限，然后此运行才能继续生成”，请单击“**查看**”、“**允许**”，并再次选择“**允许**”。 这将允许管道访问 Azure 订阅。

    > 注意：部署可能需要几分钟才能完成。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“**管道 > 管道**”，然后单击最近创建的管道。 单击省略号和“重命名/移动”选项。 将其命名为 eshoponweb-ci-docker，然后单击“保存”。

1. 导航到 [**Azure 门户**](https://portal.azure.com)，在最近创建的资源组中搜索 Azure 容器注册表（应命名为 **AZ400-RG1**）。 在左侧单击“服务”下的“存储库”，并确保已创建存储库 eshoponweb/web  。 单击存储库链接时，应看到两个标记（其中一个是最新的），这些是推送的映像。 如果没有看到，请检查管道的状态。

### 练习 2：导入并运行 CD 管道

在本练习中，你将使用 Azure 订阅配置服务连接，然后导入并运行 CD 管道。

#### 任务 1：导入并运行 CD 管道

在本任务中，你将导入并运行 CD 管道。

1. 转到“**管道 > 管道**”
1. 单击“新建管道”按钮
1. 选择“Azure Repos Git (YAML)”
1. 选择 eShopOnWeb 存储库
1. 选择“现有 Azure Pipelines YAML 文件”
1. 选择主分支和 /.ado/eshoponweb-cd-webapp-docker.yml 文件，然后单击“继续”************
1. 在 YAML 管道定义中，自定义：
   - **YOUR-SUBSCRIPTION-ID**，替换为你的 Azure 订阅 ID。
   - 将 **resourceGroup** 替换为在创建服务连接期间使用的资源组名称，例如 **AZ400-RG1**。
   - 将**位置**替换为要在其中部署资源的 Azure 区域。

1. 查看管道定义。 CD 定义由以下任务构成：
    - 资源：它下载的存储库文件将用于后续任务。
    - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure 应用服务。
    - AzureResourceManagerTemplateDeployment：使用 Bicep 添加角色分配

1. 单击“保存并运行”。

1. 打开管道执行。 如果看到警告消息“此管道需要访问资源的权限，然后此运行才能继续部署”，请选择“**查看**”、“**允许**”，然后再次选择“**允许**”。 这将允许管道访问 Azure 订阅。

    > **重要说明**：如果在配置时未授权管道，那么在执行期间会遇到权限错误。 常见的错误消息包括“此管道需要具有访问资源的权限”或“由于权限不足，管道运行失败”。 要解决此问题，请导航到管道运行，单击权限请求旁边的“视图”，然后单击“允许”授予对 Azure 订阅和资源的必要访问权限。********

    > 注意：部署可能需要几分钟才能完成。

    > [!IMPORTANT]
    > 如果收到错误消息“TF402455：不允许推送到此分支；必须使用拉取请求更新此分支。”，则需要取消选中在之前的实验室中启用的”要求达到最低审阅者数量”分支保护规则。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“**管道 > 管道**”，然后将鼠标悬停在最近创建的管道上方。 单击省略号和“**重命名/移动**”选项。 将其命名为 eshoponweb-cd-webapp-docker，然后单击“保存”。

    > **说明 1**：使用 /infra/webapp-docker.bicep 模板会创建应用服务计划、启用了系统分配的托管标识的 Web 应用，并引用之前推送的 Docker 映像：${acr.properties.loginServer}/eshoponweb/web:latest。********

    > 备注 2****：使用 /infra/webapp-to-acr-roleassignment.bicep 模板为具有 AcrPull 角色的 Web 应用创建新的角色分配，以便能够检索 Docker 映像。**** 这可以在第一个模板中完成，但由于角色分配可能需要一些时间来传播，因此最好单独执行这两个任务。

#### 任务 2：测试解决方案

1. 在 Azure 门户中，导航到最近创建的资源组，现在应会看到三个资源（应用服务、应用服务计划和容器注册表）。

1. 导航到应用服务，然后单击“浏览”，这会使你转到网站。

1. 确认 eShopOnWeb 应用程序成功运行。 确认后，你便成功完成了本实验室。

> [!IMPORTANT]
> 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在本实验室中，你学习了如何使用 Azure DevOps CI/CD 管道生成自定义 Docker 映像，将其推送到 Azure 容器注册表，并将其作为容器部署到 Azure 应用服务。
