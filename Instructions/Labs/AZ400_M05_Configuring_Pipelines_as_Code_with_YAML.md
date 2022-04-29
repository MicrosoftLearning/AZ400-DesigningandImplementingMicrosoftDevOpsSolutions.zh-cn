---
lab:
  title: 实验室 11：使用 YAML 将管道配置为代码
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
ms.openlocfilehash: 8de8c2a736004288fe37e971771b02ca7b8afd7e
ms.sourcegitcommit: 47f04db60403c98a8fd46def1794f46594bfd1f8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/27/2022
ms.locfileid: "144341227"
---
# <a name="lab-11-configuring-pipelines-as-code-with-yaml"></a>实验室 11：使用 YAML 将管道配置为代码
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-overview"></a>实验室概述

许多团队倾向于使用 YAML 定义其生成和发布管道。 通过这种方式，他们可以访问使用视觉设计器的管道功能，但标记文件可以像其他任何源文件一样进行管理。 只需将相应的文件添加到存储库的根目录即可将 YAML 生成定义添加到项目中。 Azure DevOps 还提供了适用于常用项目类型的默认模板和 YAML 设计器，以简化定义生成和发布任务的过程。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码

## <a name="lab-duration"></a>实验室时长

-   估计时间：60 分钟

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>开始之前

#### <a name="sign-in-to-the-lab-virtual-machine"></a>登录到实验室虚拟机

请确保已使用以下凭据登录到 Windows 10 虚拟机：
    
-   用户名：Student
-   密码：Pa55w.rd

#### <a name="review-applications-required-for-this-lab"></a>查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>设置 Azure DevOps 组织。 

如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

#### <a name="prepare-an-azure-subscription"></a>准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，包括基于 Azure DevOps 演示生成器模板和 Azure 资源（包括 Azure Web 应用和 Azure SQL 数据库）的预配置的 Parts Unlimited 团队项目。 

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 PartsUnlimited-YAML 模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **注意**：有关该站点的详细信息，请参阅 [什么是 Azure DevOps 服务演示生成器？](https://docs.microsoft.com/en-us/azure/devops/demo-gen)。

1.  单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1.  在“新建项目”页面上的“新建项目名称”文本框中，键入“使用 YAML 将管道配置为代码”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1.  在模板列表中，单击工具栏中的“常规”，选择“PartsUnlimited-YAML”模板并单击“选择模板”  。
1.  返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-create-azure-resources"></a>任务 2：创建 Azure 资源

在此任务中，你将使用 Azure 门户创建 Azure Web 应用和 Azure SQL 数据库。

1.  从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。
1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的“Cloud Shell”图标。
1.  如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

    >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。 

1.  在 Cloud Shell 窗格中的 Bash 提示符下，运行以下命令以创建资源组（将 `<region>` 占位符替换为离你最近的 Azure 区域的名称，例如“eastus”） 。
    
    ```bash
    LOCATION='<region>'
    ```
    ```bash
    RESOURCEGROUPNAME='az400m11l01-RG'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1.  若要创建 Windows 应用服务计划，请运行以下命令：

    ```bash
    SERVICEPLANNAME='az400l11a-sp1'
    az appservice plan create --resource-group $RESOURCEGROUPNAME --name $SERVICEPLANNAME --sku B3
    ```
        

1.  创建具有唯一名称的 Web 应用。

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME
    ```

    > **注意**：记录 Web 应用的名称。 本实验室中稍后会用到它。

1.  接下来，创建一个 Azure SQL Server。

    ```bash
    USERNAME="Student"
    SQLSERVERPASSWORD="Pa55w.rd1234"
    SERVERNAME="partsunlimitedserver$RANDOM"

    az sql server create --name $SERVERNAME --resource-group $RESOURCEGROUPNAME \
    --location $LOCATION --admin-user $USERNAME --admin-password $SQLSERVERPASSWORD
    ```

1.  Web 应用需要能够访问 SQL Server，因此我们需要在 SQL Server 防火墙规则中允许访问 Azure 资源。

    ```bash
    STARTIP="0.0.0.0"
    ENDIP="0.0.0.0"
    az sql server firewall-rule create --server $SERVERNAME --resource-group $RESOURCEGROUPNAME \
    --name AllowAzureResources --start-ip-address $STARTIP --end-ip-address $ENDIP
    ```

1.  现在，在该服务器中创建数据库。

    ```bash
    az sql db create --server $SERVERNAME --resource-group $RESOURCEGROUPNAME --name PartsUnlimited \
    --service-objective S0
    ```

1.  创建的 Web 应用需要其配置中的数据库连接字符串，因此，请运行以下命令来准备并将其添加到 Web 应用的应用设置中。

    ```bash
    CONNSTRING=$(az sql db show-connection-string --name PartsUnlimited --server $SERVERNAME \
    --client ado.net --output tsv)

    CONNSTRING=${CONNSTRING//<username>/$USERNAME}
    CONNSTRING=${CONNSTRING//<password>/$SQLSERVERPASSWORD}

    az webapp config connection-string set --name $WEBAPPNAME --resource-group $RESOURCEGROUPNAME \
    -t SQLAzure --settings "DefaultConnectionString=$CONNSTRING" 
    ```

### <a name="exercise-1-configure-cicd-pipelines-as-code-with-yaml-in-azure-devops"></a>练习 1：在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码

在本练习中，你将在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码。

#### <a name="task-1-delete-the-existing-pipeline"></a>任务 1：删除现有管道

在此任务中，你将删除现有管道。

1.  在实验室计算机上，切换到 Azure DevOps 门户中显示“使用 YAML 将管道配置为代码”项目的浏览器窗口，然后在垂直导航窗格中，选择“管道” 。

    > **注意**：在配置 YAML 管道之前，需要禁用现有的生成管道。

1.  在“管道”窗格上，选择“PartsUnlimited”条目 。 
1.  在“PartsUnlimited”边栏选项卡的右上角单击垂直省略号，然后在下拉菜单中选择“删除” 。
1.  写入“PartsUnlimited”，并单击“删除” 。
1.  在垂直导航窗格中，选择“Repos”>“文件”。 确保你位于“主”分支（“文件”窗口顶部的下拉菜单），在 azure-pipelines.yml 文件中，单击垂直省略号，然后在下拉菜单中选择“删除”   。 通过单击“提交”（保留默认选项）提交对主分支的更改。

#### <a name="task-2-add-a-yaml-build-definition"></a>任务 2：添加 YAML 生成定义

在此任务中，你将向现有项目添加 YAML 生成定义。

1.  导航回“管道”中心的“管道”窗格 。 
1.  在“创建首个管道”窗口中，单击“创建管道” 。 

    > **注意**：我们将使用向导并基于项目自动创建 YAML 定义。

1.  在“你的代码在哪里?”窗格上，单击“Azure Repos Git (YAML)”选项 。
1.  在“选择存储库”窗格上，单击“PartsUnlimited” 。
1.  在配置管道窗格中，单击 ASP.NET 以使用此模板作为管道的起点 **<nolink>** 。 这将打开“查看你的管道 YAML”窗格。

    > **注意**：管道定义将在存储库根目录中保存为名为“azure-pipelines.yml”的文件。 该文件将包含构建和测试典型 ASP<nolink>.NET 解决方案所需的步骤。 还可以根据需要自定义该生成。 在这种情况下，将更新池以强制使用运行 Windows 2019 的 VM。

1.  请确保 `trigger` 为 master。

    > **注意**：在 Repos 中查看存储库是否具有主分支，组织可以为新存储库选择默认分支名称 ：[更改默认分支](https://docs.microsoft.com/en-us/azure/devops/repos/git/change-default-branch?view=azure-devops#choosing-a-name)。 

1.  在“查看管道 YAML”窗格上，在第 10 行中，将 `vmImage: 'windows-latest'` 替换为 `vmImage: 'windows-2019'` 。
1.  删除 VSTest@2 任务：
    ```yaml
    - task: VSTest@2
      inputs:
        platform: '$(buildPlatform)'
        configuration: '$(buildConfiguration)'
    ```
1.  在“查看你的管道 YAML”窗格上，单击“保存并运行” 。
1.  在“保存并运行”窗格上，接受默认设置，然后单击“保存并运行” 。
1.  在“管道运行”窗格的“作业”部分，单击“作业”，监视其进度并验证它是否成功完成 。 

    > **注意**：YAML 文件中的每项任务都可供查看，包括任何警告和错误。

#### <a name="task-3-add-continuous-delivery-to-the-yaml-definition"></a>任务 3：将持续交付添加到 YAML 定义

在此任务中，你需要将持续交付添加到你在上一个任务中创建的基于 YAML 的管道定义。

> **注意**：生成和测试过程成功完成后，现在可以将交付添加到 YAML 定义。 

1.  在“管道运行”窗格上，单击右上角的省略号，然后在下拉菜单中单击“编辑管道”。
1.  在显示“azure-pipelines.yaml”文件内容的窗格上，在第 8 行的 `trigger` 部分后面，添加以下内容，以定义 YAML 管道中的“生成”阶段 `trigger`。 

    > **注意**：可定义所需的任何阶段，以更好地组织和跟踪管道进程。

    ```yaml
    stages:
    - stage: Build
      jobs:
      - job: Build
    ```

1.  选择 YAML 文件的其余内容，并按两次 Tab 以将其缩进四个空格（它应与 ```job: Build``` 具有相同的标识）。 

    > **注意**：这样，以 `pool` 部分开始的所有内容都成为 `job: Build` 的一部分。 

1.  在文件底部，添加以下配置以定义第二阶段。

    ```yaml
    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
          vmImage: 'windows-2019'
        steps:
    ```

1.  将光标置于 YAML 定义末尾的新行上。 

    > **注意**：这将是添加新任务的位置。

1.  在代码窗格右侧的任务列表中，搜索并选择“Azure 应用服务部署”任务。
1.  在“Azure 应用服务部署”窗格中，指定以下设置，并单击“添加”：

    - 在“Azure 订阅”下拉列表中，选择之前在实验室中已部署 Azure 资源的 Azure 订阅，单击“授权”，然后在出现提示时，使用在 Azure 资源部署期间使用的同一用户帐户进行身份验证 。
    - 在“应用服务名称”下拉列表中，选择之前在实验室中部署的 Web 应用的名称。 
    - 在“包或文件夹”文本框中，键入 `$(System.ArtifactsDirectory)/drop/*.zip`。 

    > **注意**：这会自动将部署任务添加到 YAML 管道定义。

1.  在编辑器中仍选中添加的任务，按两次 Tab 键以将其缩进四个空格，以便将该任务作为“steps”任务的子项列出 。

    > **注意**：在本实验室的上下文中，packageForLinux 参数具有误导性，但是对于 Windows 或 Linux，它是有效的。 

    > **注意**：默认情况下，这两个阶段独立运行。 因此，第一阶段的生成输出在未进行其他更改的情况下可能不适用于第二阶段。 为了实现这些更改，我们将使用一个任务在生成阶段结束时发布生成输出，使用另一个任务在部署阶段开始时下载该生成输出。 

1.  将光标置于生成阶段末尾的空行上以添加另一任务。 （在 `task: VSBuild@1` 正下方）
1.  在“任务”窗格上，搜索并选择“发布生成项目”任务 。 
1.  在“发布生成项目”窗格上，接受默认设置，并单击“添加”。 

    > **注意**：这会将可下载的生成项目发布到别名“drop”下的某个位置。

1.  在编辑器中仍选中添加的任务，按两次 Tab 键以将其缩进四个空格（或按 Tab，直至任务与上述的缩进相同） 。

    > **注意**：建议在前后各添加一个空行，使其更易阅读。

1.  将光标置于“部署”阶段的“steps”节点下方的第一行上 。
1.  在“任务”窗格上，搜索并选择“下载生成项目”任务 。
1.  单击“添加”。
1.  在编辑器中仍选中添加的任务，按两次 Tab 键以将其缩进四个空格。 

    > **注意**：建议在前后也各添加一个空行，使其更易阅读。

1.  向下载任务添加一个属性，指定 `drop` 的 `artifactName`（确保间距一致）：

    ```
    artifactName: 'drop'
    ```

1.  单击“保存”，在“保存”窗格上，再次单击“保存”，以直接将更改提交到主分支  。

    > **注意**：这将自动触发新的生成。

1.  该管道将类似于此示例（在最后一个任务中引用自己的订阅和 Web 应用）：
    
     > **注意**：需要正确设置缩进才能使其有效，复制并粘贴可能会修改它。

    ```
    trigger:
    - master

    stages:
    - stage: Build
      jobs:
      - job: Build
        pool:
            vmImage: 'windows-2019'

        variables:
            solution: '**/*.sln'
            buildPlatform: 'Any CPU'
            buildConfiguration: 'Release'

        steps:
        - task: NuGetToolInstaller@1

        - task: NuGetCommand@2
          inputs:
            restoreSolution: '$(solution)'

        - task: VSBuild@1
          inputs:
            solution: '$(solution)'
            msbuildArgs: '/p:DeployOnBuild=true /p:WebPublishMethod=Package /p:PackageAsSingleFile=true /p:SkipInvalidConfigurations=true /p:PackageLocation="$(build.artifactStagingDirectory)"'
            platform: '$(buildPlatform)'
            configuration: '$(buildConfiguration)'

        - task: PublishBuildArtifacts@1
          inputs:
            PathtoPublish: '$(Build.ArtifactStagingDirectory)'
            ArtifactName: 'drop'
            publishLocation: 'Container'

    - stage: Deploy
      jobs:
      - job: Deploy
        pool:
            vmImage: 'windows-2019'
        steps:
        - task: DownloadBuildArtifacts@0
          inputs:
            buildType: 'current'
            downloadType: 'single'
            downloadPath: '$(System.ArtifactsDirectory)'
            artifactName: 'drop'
        - task: AzureRmWebAppDeployment@4
          inputs:
            ConnectionType: 'AzureRM'
            azureSubscription: 'YOUR-AZURE-SUBSCRIPTION'
            appType: 'webApp'
            WebAppName: 'YOUR-WEBAPP-NAME'
            packageForLinux: '$(System.ArtifactsDirectory)/drop/*.zip'
    ```

1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，从垂直导航窗格中选择“管道”。
1.  在“管道”窗格上，单击表示新配置的管道的条目。
1. 单击最近的运行（自动启动）。
1.  在“摘要”窗格上，监视管道运行的进度。
1.  如果你看到一条消息，内容为“此管道需要访问资源的权限，然后此运行才能继续部署”，请单击“查看”，在“等待查看”对话框中，单击“允许”，然后在“允许访问?”窗格中，再次单击“允许”    。
1.  在“摘要”窗格底部，单击“部署”阶段以查看该部署的详细信息 。

    > **注意**：任务完成后，应用将部署到 Azure Web 应用。

#### <a name="task-4-review-the-deployed-site"></a>任务 4：查看已部署的站点

1.  切换回显示 Azure 门户的 Web 浏览器窗口，导航到显示 Azure Web 应用的属性的边栏选项卡。
1.  在 Azure Web 应用边栏选项卡上，单击“概述”，然后在“概述”边栏选项卡上，单击“浏览”以在新的 Web 浏览器选项卡中打开站点 。
1.  验证已部署的站点是否按预期在新浏览器选项卡中加载。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].name" --output tsv
    ```

1.  通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m11l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本实验室中，你已在 Azure DevOps 中使用 YAML 将 CI/CD 管道配置为代码。
