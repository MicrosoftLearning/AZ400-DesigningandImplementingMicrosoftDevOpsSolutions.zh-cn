---
lab:
  title: 启用动态配置和功能标志
  module: 'Module 04: Implement a secure continuous deployment using Azure Pipelines'
---

# 启用动态配置和功能标志

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://learn.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你是否拥有 Microsoft 帐户或 Microsoft Entra 帐户以及 Azure 订阅中的参与者或所有者角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal)。

## 实验室概述

[Azure 应用程序配置](https://learn.microsoft.com/azure/azure-app-configuration/overview)提供了一项服务，用于集中管理应用程序设置和功能标志。 新式程序（特别是在云中运行的程序）通常具有许多分布式组件。 在这些组件之间分布配置设置可能会导致在应用程序部署期间难以排查错误。 使用应用程序配置来存储应用程序的所有设置并在一个位置保证其访问权限的安全。

## 目标

完成本实验室后，你将能够：

- 启用动态配置。
- 管理功能标志。

## 预计用时：45 分钟

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

### 练习 1：（如果已完成，请跳过此任务）导入和运行 CI/CD 管道

在本练习中，你将导入 CI/CD 管道以生成和部署 eShopOnWeb 应用程序。 CI 管道已准备好生成应用程序并运行测试。 CD 管道会将应用程序部署到 Azure Web 应用。

#### 任务 1：导入并运行 CI 管道

让我们首先导入名为 [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml) 的 CI 管道。

1. 转到“**管道 > 管道**”。
1. 单击“创建管道”按钮（如果没有管道）或“新建管道”按钮（如果已有已创建的管道） 。
1. 选择“Azure Repos Git (YAML)”。
1. 选择“eShopOnWeb”存储库。
1. 选择“现有 Azure Pipelines YAML 文件”。
1. 选择主分支和 /.ado/eshoponweb-ci.yml 文件，然后单击“继续”。************
1. 单击“运行”按钮以运行管道。
1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道”>“管道”****，然后单击最近创建的管道。 单击省略号和“重命名/删除”选项。 将其命名为 eshoponweb-ci，然后单击“保存”。

#### 任务 2：导入并运行 CD 管道

让我们导入名为 [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml) 的 CD 管道。

1. 转到“**管道 > 管道**”。
1. 单击“新建管道”按钮。
1. 选择“Azure Repos Git (YAML)”。
1. 选择“eShopOnWeb”存储库。
1. 选择“现有 Azure Pipelines YAML 文件”。
1. 选择主分支和 /.ado/eshoponweb-cd-webapp-code.yml 文件，然后单击“继续”。************
1. 在 YAML 管道定义中，自定义：
   - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
   - az400eshop-NAME，替换 NAME 使其全局唯一。
   - AZ400-EWebShop-NAME，其中包含之前在实验室中定义的资源组名称。

1. 单击“保存并运行”，等待管道成功执行。

    > **注意**：必须单击“保存并运行”**** 两次。 如果“作业”窗口显示所需的权限消息，请从“作业”窗口中依次选择“部署”、“视图”，然后单击“允许”两次以完成管道运行。************

    > 注意：部署可能需要几分钟才能完成。

    CD 定义由以下任务构成：
    - 资源：它已准备好根据 CI 管道完成自动触发。 它还会下载 bicep 文件的存储库。
    - AzureResourceManagerTemplateDeployment：使用 bicep 模板部署 Azure Web 应用。

1. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道”>“管道”****，然后单击最近创建的管道。 单击省略号和“重命名/删除”选项。 将其命名为 eshoponweb-cd-webapp-code，然后单击“保存”。

### 练习 2：管理 Azure 应用程序配置

在本练习中，你将在 Azure 中创建应用程序配置资源，启用托管标识，然后测试完整的解决方案。

> 注意：本练习不需要任何编码技能。 网站的代码已实现 Azure 应用程序配置功能。

如果想要了解如何在应用程序中实现此功能，请查看以下教程：[在 ASP.NET Core 应用中使用动态配置](https://learn.microsoft.com/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core)和[在 Azure 应用程序配置中管理功能标志](https://learn.microsoft.com/azure/azure-app-configuration/manage-feature-flags)。

#### 任务 1：创建应用程序配置资源

1. 在 Azure 门户中，搜索“应用程序配置”服务
1. 单击“创建应用程序配置”，然后选择：
    - 你的 Azure 订阅。
    - 之前创建的资源组（应命名为 AZ400-EWebShop-NAME）。
    - 位置。
    - 唯一名称，例如 appcs-NAME-REGION。
    - 选择“免费”定价层。
1. 单击“查看 + 创建”，然后单击“创建”。
1. 创建应用程序配置服务后，转到“概述”并复制/保存“终结点”的值。

#### 任务 2：启用托管标识

1. 转到使用管道部署的 Web 应用（应命名为 az400-webapp-NAME）。
1. 在“设置”部分中，单击“标识”，在“系统分配”部分中将状态切换为“开”，单击“保存”>“是”，然后等待几秒钟，等待操作完成。
1. 返回到应用程序配置服务，单击“访问控制”，然后单击“添加角色分配”。
1. 在“角色”部分中，选择“应用程序配置数据读取者”。
1. 在“成员”部分中，选中“管理标识”，然后单击“+ 选择成员”。************ 在“托管标识”字段中，选择“应用服务(1)”，选择你的应用服务，然后单击“选择”。************
1. 单击“查看并分配”两次以完成角色分配****。

#### 任务 3：配置 Web 应用

为了确保网站访问应用程序配置，需要更新其配置。

1. 返回到 Web 应用。
1. 在“设置”部分中，单击“环境变量”。********
1. 添加两个新的应用程序设置：
    - 第一个应用设置
        - 名称：UseAppConfig
        - 值：true
    - 第二个应用设置
        - 名称：AppConfigEndpoint
        - 值：以前从应用程序配置终结点保存/复制的值。它应类似于 <https://appcs-NAME-REGION.azconfig.io>

1. 单击“应用”，然后单击“确认”并等待设置更新。********
1. 转到“概述”并单击“浏览”
1. 在此步骤中，你将看不到网站中的任何更改，因为应用程序配置不包含任何数据。 这是你将在后续任务中执行的操作。

#### 任务 4：测试配置管理

1. 在网站中的“品牌”下拉列表中选择“Visual Studio”，然后单击箭头按钮 (>)。
1. 你将看到一条消息，指出“没有与你的搜索匹配的结果”。 本实验室的目标是在不更新网站代码或重新部署代码的情况下更新该值。
1. 若要尝试此操作，请返回到应用程序配置。
1. 在“操作”部分中，选择“配置资源管理器”。
1. 单击“创建 > 键值”，然后添加：
    - 键：eShopWeb:Settings:NoResultsMessage
    - 值：键入自定义消息
1. 单击“应用”，然后返回到网站并刷新页面。
1. 应会看到新消息，而不是旧的默认值。

#### 任务 5：测试功能标志

让我们继续测试功能管理器。

1. 若要尝试此操作，请返回到应用程序配置。
1. 在“操作”部分中，选择“功能管理器”。
1. 单击“创建”，然后添加：
    - 启用功能标志：已选中
    - 功能标志名称：SalesWeekend
1. 单击“应用”，然后返回到网站并刷新页面。
1. 你应该会看到一个图像，其中包含文本“本周末所有 T 恤减价出售”。
1. 可以在应用程序配置中禁用此功能，然后会看到图像消失。

   > [!IMPORTANT]
   > 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。 请务必禁用 **eshoponweb-cd-webapp-code** 管道，否则它将在下一次运行 **eshoponweb-ci** 后重新创建已删除的资源组和关联的资源。

## 审阅

在本实验室中，你学习了如何动态启用配置和管理功能标志。
