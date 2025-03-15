---
lab:
  title: 使用 YAML 将管道配置为代码
  module: 'Module 03: Design and implement a release strategy'
---

# 使用 YAML 将管道配置为代码

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/azure/devops/organizations/accounts/create-organization)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你拥有 Microsoft 帐户或 Microsoft Entra 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Microsoft Entra 租户中具有全局管理员角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal)。

## 实验室概述

许多团队倾向于使用 YAML 定义其生成和发布管道。 通过这种方式，他们可以访问使用视觉设计器的管道功能，但标记文件可以像其他任何源文件一样进行管理。 只需将相应的文件添加到存储库的根目录即可将 YAML 生成定义添加到项目中。 Azure DevOps 还提供了适用于常用项目类型的默认模板和 YAML 设计器，以简化定义生成和发布任务的过程。

## 目标

完成本实验室后，你将能够：

- 在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码。

## 预计用时：45 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，将设置实验室先决条件。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb_MultiStageYAML Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb_MultiStageYAML，并将其他字段保留默认值。 单击“创建”。

   ![“创建新项目”面板的屏幕截图。](images/create-project.png)

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb_MultiStageYAML 项目。 单击“**Repos > 文件**”、“**导入存储库**”。 选择“**导入**”。 在“导入 Git 存储库”窗口中，粘贴以下 URL https://github.com/MicrosoftLearning/eShopOnWeb.git 并单击“导入”：

   ![“导入存储库”面板的屏幕截图。](images/import-repo.png)

1. 存储库按以下方式组织：
   - .ado 文件夹包含 Azure DevOps YAML 管道。
   - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
   - infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
   - .github 文件夹容器 YAML GitHub 工作流定义。
   - src 文件夹包含用于实验室方案的 .NET 8 网站。****

1. 转到“**Repos > 分支**”。
1. 将鼠标指针悬停在主分支上，然后单击列右侧的省略号。
1. 单击“设置为默认分支”。

#### 任务 3：创建 Azure 资源

在本任务中，你将使用 Azure 门户创建 Azure Web 应用。

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
1. 在 Azure 门户的工具栏中，单击搜索文本框右侧的“Cloud Shell”图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

   > **注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

   > **备注**：若要获取区域及其别名的列表，请从 Azure Cloud Shell - Bash 运行以下命令：

   ```bash
   az account list-locations -o table
   ```

1. 在 Cloud Shell 窗格中的 Bash 提示符下，运行以下命令以创建资源组（将 `<region>` 占位符替换为离你最近的 Azure 区域的名称，例如“centralus”、“westeurope”或其他选择的区域） 。

   ```bash
   LOCATION='<region>'
   ```

   ```bash
   RESOURCEGROUPNAME='az400m03l07-RG'
   az group create --name $RESOURCEGROUPNAME --location $LOCATION
   ```

1. 若要创建 Windows 应用服务计划，请运行以下命令：

   ```bash
   SERVICEPLANNAME='az400m03l07-sp1'
   az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
   ```

1. 创建具有唯一名称的 Web 应用。

   ```bash
   WEBAPPNAME=eshoponWebYAML$RANDOM$RANDOM
   az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
   ```

   > **注意**：记录 Web 应用的名称。 本实验室中稍后会用到它。

1. 关闭 Azure Cloud Shell，但让 Azure 门户在浏览器中保持打开状态。

### 练习 1：在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码

在本练习中，你将在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码。

#### 任务 1：添加 YAML 生成定义

在此任务中，你将向现有项目添加 YAML 生成定义。

1. 导航回“管道”中心的“管道”窗格 。
1. 在“创建首个管道”窗口中，单击“创建管道” 。

   > 注意：我们将使用向导并基于项目创建新的 YAML 管道定义。

1. 在“你的代码在哪里?”窗格上，单击“Azure Repos Git (YAML)”选项 。
1. 在“选择存储库”窗格中，单击“eShopOnWeb_MultiStageYAML” 。
1. 在“配置管道”窗格上，向下滚动并选择“现有 Azure Pipelines YAML 文件” 。
1. 在“选择现有 YAML 文件”边栏选项卡中，指定以下参数：
   - 分支：main
   - 路径：.ado/eshoponweb-ci.yml
1. 单击“继续”保存这些设置。
1. 在“查看管道 YAML”屏幕中，单击“运行”以启动生成管道过程。
1. 等待生成管道过程成功完成。 忽略有关源代码本身的任何警告，因为这些警告与本实验室练习无关。

   > **注意**：YAML 文件中的每项任务都可供查看，包括任何警告和错误。

#### 任务 2：将持续交付添加到 YAML 定义

在此任务中，你需要将持续交付添加到你在上一个任务中创建的基于 YAML 的管道定义。

> **注意**：生成和测试过程成功完成后，现在可以将交付添加到 YAML 定义。

1. 在“管道运行”窗格上，单击右上角的省略号，然后在下拉菜单中单击“编辑管道”。
1. 在显示 eShopOnWeb_MultiStageYAML/.ado/eshoponweb-ci.yml 文件内容的窗格中，导航到文件末尾（第 56 行），然后按 Enter/Return 添加新的空行。
1. 在第 57 行上，添加以下内容，定义 YAML 管道中的“发布”阶段 。

   > **注意**：可定义所需的任何阶段，以更好地组织和跟踪管道进程。

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - job: Deploy
         pool:
           vmImage: "windows-latest"
         steps:
   ```

1. 将光标置于 YAML 定义末尾的新行上。

   > **注意**：这将是添加新任务的位置。

1. 在代码窗格右侧的任务列表中，搜索并选择“Azure 应用服务部署”任务。
1. 在“Azure 应用服务部署”窗格中，指定以下设置，并单击“添加”：

   - 在“Azure 订阅”下拉列表中，选择之前在实验室中已部署 Azure 资源的 Azure 订阅，单击“授权”，然后在出现提示时，使用在 Azure 资源部署期间使用的同一用户帐户进行身份验证 。
   - 在“应用服务名称”下拉列表中，选择之前在实验室中部署的 Web 应用的名称。
   - 在“包或文件夹”文本框中，将默认值更新为 `$(Build.ArtifactStagingDirectory)/**/Web.zip`。
   - 在“应用程序和配置设置”中添加 `-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development`****

1. 单击“添加”按钮，确认“助手”窗格中的设置。

   > **注意**：这会自动将部署任务添加到 YAML 管道定义。

1. 添加到编辑器的代码片段应如下所示，它反映了 azureSubscription 和 WebappName 参数的名称：

   ```yaml
   - task: AzureRmWebAppDeployment@4
     inputs:
       ConnectionType: "AzureRM"
       azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
       appType: "webApp"
       WebAppName: "eshoponWebYAML369825031"
       packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
       AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. 验证任务是否列为“steps”任务的子任务。 如果没有，则选择添加的任务中的所有行，按两次 Tab 键以将其缩进四个空格，以便将该任务作为“steps”任务的子项列出 。

   > **注意**：在本实验室的上下文中，packageForLinux 参数具有误导性，但是对于 Windows 或 Linux，它是有效的。

   > **注意**：默认情况下，这两个阶段独立运行。 因此，第一阶段的生成输出在未进行其他更改的情况下可能不适用于第二阶段。 为了实现这些更改，我们将添加一个新任务，以在部署阶段开始时下载部署项目。

1. 将光标置于“部署”阶段的“steps” 节点下的第一行上，然后按 Enter/Return 添加一个新的空行（第 64 行）。
1. 在“任务”窗格上，搜索并选择“下载生成项目”任务 。
1. 为此任务指定以下参数：
   - 下载以下作业生成的项目：当前生成
   - 下载类型：特定项目
   - 项目名称：**从列表中选择“网站”**（或如果未自动显示在列表中，则直接**键入“`Website`”**）
   - 目标目录：$(Build.ArtifactStagingDirectory)
1. 单击“添加”。
1. 添加的代码片段应如下所示：

   ```yaml
   - task: DownloadBuildArtifacts@1
     inputs:
       buildType: "current"
       downloadType: "single"
       artifactName: "Website"
       downloadPath: "$(Build.ArtifactStagingDirectory)"
   ```

1. 如果 YAML 缩进处于禁用状态，则在编辑器中仍选中添加的任务，按两次 Tab 键以将其缩进四个空格。

   > **注意**：建议在前后也各添加一个空行，使其更易阅读。

1. 单击“保存”，在“保存”窗格上，再次单击“保存”，以直接将更改提交到主分支  。

   > 注意：由于原始 CI-YAML 未配置为自动触发新的生成，因此必须手动启动此生成。

1. 在 Azure DevOps 左侧菜单中，导航到“管道”，然后再次选择“管道”。
1. 打开“eShopOnWeb_MultiStageYAML”**** 管道，然后单击“运行管道”****。
1. 从显示的窗格中确认“运行”。
1. 请注意，此时出现了两个不同的阶段，即“生成 .Net Core 解决方案”和“部署到 Azure Web 应用”。
1. 等待管道启动，并等待它成功完成生成阶段。
1. 一旦部署阶段需要启动，系统会提示你“需要权限”，并显示一个橙色条：

   ```text
   This pipeline needs permission to access a resource before this run can continue to Deploy to an Azure Web App
   ```

1. 单击“查看”
1. 在“正在等待审阅”窗格中，单击“允许”。
1. 验证“允许弹出”窗口中的消息，然后单击“允许”进行确认。
1. 这将启动部署阶段。 等待该过程成功完成。

   > 注意：如果由于 YAML 管道语法问题导致部署失败，请参考以下内容：

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   # trigger:
   # - main

   resources:
    repositories:
      - repository: self
        trigger: none

   stages:
   - stage: Build
     displayName: Build .Net Core Solution
     jobs:
     - job: Build
       pool:
         vmImage: ubuntu-latest
       steps:
       - task: DotNetCoreCLI@2
         displayName: Restore
         inputs:
           command: 'restore'
           projects: '**/*.sln'
           feedsToUse: 'select'

       - task: DotNetCoreCLI@2
         displayName: Build
         inputs:
           command: 'build'
           projects: '**/*.sln'

       - task: DotNetCoreCLI@2
         displayName: Test
         inputs:
           command: 'test'
           projects: 'tests/UnitTests/*.csproj'

       - task: DotNetCoreCLI@2
         displayName: Publish
         inputs:
           command: 'publish'
           publishWebProjects: true
           arguments: '-o $(Build.ArtifactStagingDirectory)'

       - task: PublishBuildArtifacts@1
         displayName: Publish Artifacts ADO - Website
         inputs:
           pathToPublish: '$(Build.ArtifactStagingDirectory)'
           artifactName: Website

       - task: PublishBuildArtifacts@1
         displayName: Publish Artifacts ADO - Bicep
         inputs:
           PathtoPublish: '$(Build.SourcesDirectory)/infra/webapp.bicep'
           ArtifactName: 'Bicep'
           publishLocation: 'Container'

    - stage: Deploy
      displayName: Deploy to an Azure Web App
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-latest'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            artifactName: 'Website'
            downloadPath: '$(Build.ArtifactStagingDirectory)'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)'
            appType: 'webApp'
            WebAppName: 'eshoponWebYAML369825031'
            packageForLinux: '$(Build.ArtifactStagingDirectory)/**/Web.zip'
            AppSettings: '-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development'
   ```

#### 任务 3：查看已部署的站点

1. 切换回显示 Azure 门户的 Web 浏览器窗口，导航到显示 Azure Web 应用的属性的边栏选项卡。
1. 在 Azure Web 应用边栏选项卡上，单击“概述”，然后在“概述”边栏选项卡上，单击“浏览”以在新的 Web 浏览器选项卡中打开站点 。
1. 验证新浏览器选项卡中的已部署站点是否按预期加载，显示 eShopOnWeb 电子商务网站。

### 练习 2：在 Azure DevOps 中使用 YAML 为 CI/CD 管道配置环境设置

在本练习中，你将向 Azure DevOps 中基于 YAML 的管道添加审批。

#### 任务 1：设置管道环境

YAML 管道即代码并不像 Azure DevOps 经典发布管道那样具有发布/质量门。 但是，可以使用环境为 YAML 管道即代码配置一些相似的内容。 在此任务中，你将使用此机制为生成阶段配置审批。

1. 在 Azure DevOps 项目 eShopOnWeb_MultiStageYAML 中****，导航到“管道”****。
1. 在左侧的“管道菜单”下，选择“环境”。
1. 单击“创建环境”。
1. 在“**新建环境**”窗格中，为环境添加一个名称，称为 **`approvals`**。
1. 在“资源”下，选择“无”。
1. 单击“创建”按钮确认设置。
1. 创建环境后，从新的**审批**环境中选择“**审批和检查**”选项卡。
1. 在“添加你的第一个检查”中，选择“审批”。
1. 将 Azure DevOps 用户帐户名称添加到“approvers”字段。

   > 注意：在现实场景中，这将显示为处理此项目的 DevOps 团队的名称。

1. 单击“创建”按钮确认已定义的审批设置。
1. 最后，我们需要将必需的“environment: approvals”设置添加到部署阶段的 YAML 管道代码中。 为此，导航到“Repos”，浏览到“.ado”文件夹，然后选择“eshoponweb-ci.yml”管道即代码文件。
1. 在“内容”视图中，单击“编辑”按钮切换到编辑模式。
1. 导航到“部署作业”的起始位置（第 60 行的 -job: Deploy）
1. 在下方添加一个新的空行，并添加以下代码片段：

   ```yaml
   environment: approvals
   ```

   生成的代码片段应如下所示：

   ```yaml
   jobs:
     - job: Deploy
       environment: approvals
       pool:
         vmImage: "windows-latest"
   ```

1. 由于环境是部署阶段的特定设置，因此“jobs”不能使用它。 我们必须对当前的作业定义进行一些额外的更改。
1. 在第 60 行，将“- job: Deploy”重命名为“- deployment: Deploy”
1. 接下来，在第 **63** 行 (vmImage: windows-latest) 下，添加一个新的空行。
1. 粘贴以下 YAML 代码片段：

   ```yaml
   strategy:
     runOnce:
       deploy:
   ```

1. 选择其余的代码片段（第 67 行一直到末尾），并使用 Tab 键固定 YAML 的缩进 。

   生成的 YAML 代码片段现在应如下所示，它反映了部署阶段：

   ```yaml
   - stage: Deploy
     displayName: Deploy to an Azure Web App
     jobs:
       - deployment: Deploy
         environment: approvals
         pool:
           vmImage: "windows-latest"
         strategy:
           runOnce:
             deploy:
               steps:
                 - task: DownloadBuildArtifacts@1
                   inputs:
                     buildType: "current"
                     downloadType: "single"
                     artifactName: "Website"
                     downloadPath: "$(Build.ArtifactStagingDirectory)"
                 - task: AzureRmWebAppDeployment@4
                   inputs:
                     ConnectionType: "AzureRM"
                     azureSubscription: "AZURE SUBSCRIPTION HERE (b999999abc-1234-987a-a1e0-27fb2ea7f9f4)"
                     appType: "webApp"
                     WebAppName: "eshoponWebYAML369825031"
                     packageForLinux: "$(Build.ArtifactStagingDirectory)/**/Web.zip"
                     AppSettings: "-UseOnlyInMemoryDatabase true -ASPNETCORE_ENVIRONMENT Development"
   ```

1. 单击“提交”，然后在出现的“提交”窗格中再次单击“提交”，确认对代码 YAML 文件的更改。
1. 导航到左侧的“Azure DevOps 项目”菜单，选择“管道”，接着选择“管道”，然后注意前面使用的“EshopOnWeb_MultiStageYAML”管道。
1. 打开管道。
1. 单击“运行管道”以触发新的管道运行；单击“运行”进行确认。
1. 与之前一样，生成阶段按预期启动。 等待启动过程成功完成。
1. 接下来，由于我们已经为部署阶段配置了“environment:approvals”，它将在启动之前请求审批确认。
1. 从“管道”视图中可以看到这一点，其中显示“正在等待(已通过 0/1 项检查)”。 同时，还会显示一条通知消息，指示“在此运行可以继续部署到 Azure Web 应用之前，审批需要评审”。
1. 单击此消息旁边的“查看”按钮。
1. 在显示的“部署到 Azure Web 应用的检查和手动验证”窗格中，单击“审批等待”消息。
1. 单击“批准”。
1. 这样，部署阶段就可以启动并成功部署 Azure Web 应用源代码了。

   > 注意：虽然本示例仅使用了审批，但也需要了解，Azure Monitor、REST API 等其他检查也可以采用类似的方式

   > [!IMPORTANT]
   > 请记住删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在本实验室中，你已在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码。
