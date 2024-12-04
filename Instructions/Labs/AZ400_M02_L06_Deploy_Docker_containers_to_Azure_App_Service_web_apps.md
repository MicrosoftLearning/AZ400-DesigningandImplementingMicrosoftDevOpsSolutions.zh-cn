---
lab:
  title: 将 Docker 容器部署到 Azure 应用服务 Web 应用
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# 将 Docker 容器部署到 Azure 应用服务 Web 应用

## 学生实验室手册

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

## 预计用时：30 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb，然后在“工作项进程”下拉列表中选择“Scrum”。 单击“创建”。

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“Repos > 文件”、“导入”。 在“导入 Git 存储库”窗口中，粘贴以下 URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> 并单击“导入”：

1. 存储库按以下方式组织：
    - .ado 文件夹包含 Azure DevOps YAML 管道。
    - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
    - infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
    - .github 文件夹容器 YAML GitHub 工作流定义。
    - src 文件夹包含用于实验室方案的 .NET 8 网站。****

#### 任务 3：（如果已完成，请跳过此任务）将主分支设置为默认分支

1. 转到“Repos”>“分支”。
1. 将鼠标指针悬停在主分支上，然后单击列右侧的省略号。
1. 单击“设置为默认分支”。

### 练习 1：管理服务连接

在本练习中，你将使用 Azure 订阅配置服务连接，然后导入并运行 CI 管道。

#### 任务 1：（如果已完成，请跳过此任务）管理服务连接

可以创建从 Azure Pipelines 到外部和远程服务的连接，以便在作业中执行任务。

在此任务中，你将使用 Azure CLI 创建服务主体，这将允许 Azure DevOps：

- 在 Azure 订阅上部署资源。
- 向 Azure 容器注册表推送 Docker 映像。
- 添加角色分配，允许 Azure 应用服务从 Azure 容器注册表拉取 Docker 映像。

> **注意**：如果你已有一个服务主体，则可以直接进行下一个任务。

你需要一个服务主体从 Azure Pipelines 部署 Azure 资源。

从管道定义内部连接到 Azure 订阅或从项目设置页面（自动选项）新建服务连接时，Azure Pipeline 会自动创建服务主体。 你也可以从门户或使用 Azure CLI 手动创建服务主体，然后在项目中重复使用。

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
1. 在 Azure 门户中，单击页面顶部搜索文本框右侧的 Cloud Shell 图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以检索 Azure 订阅 ID 属性的值：

    ```bash
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > **注意**：将两个值都复制到文本文件中。 稍后将在本实验室用到它们。

1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以创建服务主体：

    ```bash
    az ad sp create-for-rbac --name sp-az400-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > **注意**：此命令将生成 JSON 输出。 将输出复制到文本文件中。 本实验室中稍后会用到它。

1. 接下来，从实验室计算机启动 Web 浏览器，导航到 Azure DevOps eShopOnWeb 项目。 单击“项目设置 > 服务连接”（在“管道”下）和“新建服务连接”。
1. 在“新建服务连接”边栏选项卡上，选择“Azure 资源管理器”和“下一步”（可能需要向下滚动）。
1. 选择“服务主体(手动)”并单击“下一步”。
1. 使用在前面的步骤中收集的信息填写空字段：
    - 订阅 ID 和名称。
    - 服务主体 ID (appId)、服务主体密钥（密码）和租户 ID（租户）。
    - 在“服务连接名称”中，键入 azure-connection。 需要 Azure DevOps 服务连接来与 Azure 订阅通信时，将在 YAML 管道中引用此名称。

1. 单击“验证并保存”。

### 练习 2：导入并运行 CI 管道

在本练习中，你将导入并运行 CI 管道。

#### 任务 1：导入并运行 CI 管道

1. 转到“管道 > 管道”
1. 单击“新管道”按钮（如果之前没有创建其他管道，则单击“创建管道”）********
1. 选择“Azure Repos Git (YAML)”
1. 选择 eShopOnWeb 存储库
1. 选择**现有 Azure Pipelines YAML 文件**
1. 选择主分支和 /.ado/eshoponweb-ci-docker.yml 文件，然后单击“继续”************
1. 在 YAML 管道定义中，自定义：
   - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
   - rg-az400-container-NAME，其中包含将由管道创建的资源组名称（它可能也是现有资源组）。

1. 单击“保存并运行”，等待管道成功执行。

    > 注意：部署可能需要几分钟才能完成。

    CI 定义由以下任务构成：
    - 资源：它下载的存储库文件将用于后续任务。
    - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure 容器注册表。
    - PowerShell：从上一任务的输出中检索“ACR 登录服务器”值并创建新的参数 acrLoginServer
    - [**Docker**](https://learn.microsoft.com/azure/devops/pipelines/tasks/reference/docker-v0?view=azure-pipelines) **- 生成**：生成 Docker 映像并创建两个标记（最新和当前 BuildID）
    - Docker - 推送：向 Azure 容器注册表推送映像

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道 > 管道”，然后单击最近创建的管道。 单击省略号和“重命名/移动”选项。 将其命名为 eshoponweb-ci-docker，然后单击“保存”。

1. 导航到 [Azure 门户](https://portal.azure.com)，在最近创建的资源组中搜索 Azure 容器注册表（它应名为 rg-az400-container-NAME）。 在左侧单击“服务”下的“存储库”，并确保已创建存储库 eshoponweb/web  。 单击存储库链接时，应看到两个标记（其中一个是最新的），这些是推送的映像。 如果没有看到，请检查管道的状态。

### 练习 3：导入并运行 CD 管道

在本练习中，你将使用 Azure 订阅配置服务连接，然后导入并运行 CD 管道。

#### 任务 1：添加新角色分配

在本任务中，你将添加新角色分配以允许 Azure 应用服务从 Azure 容器注册表拉取 Docker 映像。

1. 导航到 [Azure 门户](https://portal.azure.com)。
1. 在 Azure 门户中，单击页面顶部搜索文本框右侧的 Cloud Shell 图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。
1. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以检索 Azure 订阅 ID 属性的值：

    ```sh
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionId
    spId=$(az ad sp list --display-name sp-az400-azdo --query "[].id" --output tsv)
    echo $spId
    roleName=$(az role definition list --name "User Access Administrator" --query "[0].name" --output tsv)
    echo $roleName
    ```

1. 获取服务主体 ID 和角色名称后，通过运行此命令创建角色分配（将 rg-az400-container-NAME 替换为资源组名称）

    ```sh
    az role assignment create --assignee $spId --role $roleName --scope /subscriptions/$subscriptionId/resourceGroups/<rg-az400-container-NAME>
    ```

现在应会看到 JSON 输出，用于确认命令运行是否成功。

#### 任务 2：导入并运行 CD 管道

在本任务中，你将导入并运行 CD 管道。

1. 转到“管道 > 管道”
1. 单击“新建管道”按钮
1. 选择“Azure Repos Git (YAML)”
1. 选择 eShopOnWeb 存储库
1. 选择“现有 Azure Pipelines YAML 文件”
1. 选择主分支和 /.ado/eshoponweb-cd-webapp-docker.yml 文件，然后单击“继续”************
1. 在 YAML 管道定义中，自定义：
   - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
   - rg-az400-container-NAME，其中包含之前在实验室中定义的资源组名称。

1. 单击“保存并运行”，等待管道成功执行。

    > 注意：部署可能需要几分钟才能完成。

    > 重要说明：如果收到错误消息“TF402455: 不允许推送到此分支；必须使用拉取请求更新此分支。”需要取消选中在前面的实验室中启用的”要求达到最小审阅者数量”分支保护规则。

    CD 定义由以下任务构成：
    - 资源：它下载的存储库文件将用于后续任务。
    - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure 应用服务。
    - AzureResourceManagerTemplateDeployment：使用 Bicep 添加角色分配

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道”>“管道”，然后将鼠标指针悬停在最近创建的管道上方。 单击省略号和“重命名/移动”选项。 将其命名为 eshoponweb-cd-webapp-docker，然后单击“保存”。

    > **说明 1**：使用 /infra/webapp-docker.bicep 模板会创建应用服务计划、启用了系统分配的托管标识的 Web 应用，并引用之前推送的 Docker 映像：${acr.properties.loginServer}/eshoponweb/web:latest。********

    > 备注 2****：使用 /infra/webapp-to-acr-roleassignment.bicep 模板为具有 AcrPull 角色的 Web 应用创建新的角色分配，以便能够检索 Docker 映像。**** 这可以在第一个模板中完成，但由于角色分配可能需要一些时间来传播，因此最好单独执行这两个任务。

#### 任务 3：测试解决方案

1. 在 Azure 门户中，导航到最近创建的资源组，现在应会看到三个资源（应用服务、应用服务计划和容器注册表）。

1. 导航到应用服务，然后单击“浏览”，这会使你转到网站。

恭喜！ 在本练习中，你使用自定义 Docker 映像部署了网站。

### 练习 4：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'rg-az400-container-')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 审阅

在本实验室中，你学习了如何使用 Azure DevOps CI/CD 管道生成自定义 Docker 映像，将其推送到 Azure 容器注册表，并将其作为容器部署到 Azure 应用服务。
