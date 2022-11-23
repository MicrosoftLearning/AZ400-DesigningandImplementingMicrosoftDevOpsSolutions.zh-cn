---
lab:
  title: 实验室 07：使用 DevOps Starter 实现 GitHub Actions
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# <a name="lab-07-implementing-github-actions-by-using-devops-starter"></a>实验室 07：使用 DevOps Starter 实现 GitHub Actions

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你是否拥有 Microsoft 帐户或具有 Azure 订阅中参与者或所有者角色的 Azure AD 帐户。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

- 如果还没有可用于此实验的 GitHub 帐户，请按照[注册新的 GitHub 帐户](https://github.com/join)中的说明创建一个帐户。

## <a name="lab-overview"></a>实验室概述

在此实验室中，你将学习如何使用 DevOps Starter 来实现部署 Azure Web 应用的 GitHub Action 工作流。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 使用 DevOps Starter 实现 GitHub Actions 工作流。
- 解释 GitHub Action 工作流的基本特征。

## <a name="estimated-timing-30-minutes"></a>预计用时：30 分钟

## <a name="instructions"></a>说明

### <a name="exercise-1-create-a-devops-starter-project"></a>练习 1：创建 DevOps Starter 项目

在本练习中，你将使用 DevOps Starter 来更好地配置许多资源，包括：

- GitHub 存储库托管：

  - 示例 .NET Core 网站的代码。
  - Azure 资源管理器模板，用于部署托管网站代码的 Azure Web 应用。
  - 构建、部署和测试网站的工作流。

- 一个 Azure Web 应用，通过使用 GitHub 工作流自动部署。

#### <a name="task-1-create-devops-starter-project"></a>任务 1：创建 DevOps Starter 项目

在这个任务中，你将创建 Azure DevOps Starter 项目，它会自动设置 GitHub 存储库，并创建和触发 GitHub 工作流来基于 GitHub 存储库的内容部署 Azure Web 应用。

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1. 在 Azure 门户中搜索并选择“DevOps Starter”资源类型，然后在“DevOps Starter”边栏选项卡上单击“+ 添加”、“+ 新建”或“+ 创建”    。
1. 在“DevOps Starter”边栏选项卡的“使用新应用程序重新开始”页面上，单击“单击此处，使用 GitHub 设置 DevOps Starter”文本中的“此处”链接   。

    > **注意**：此时会显示“DevOps Starter 设置”边栏选项卡。

1. 在“DevOps Starter 设置”边栏选项卡上确保已选中“GitHub”磁贴，然后单击“完成”  。
1. 返回“DevOps Starter”边栏选项卡，单击“下一页: 框架 >”。
1. 在“DevOps Starter”边栏选项卡的“选择应用程序框架”页面上，选择“ASP.NET Core”磁贴并单击“下一页  : 服务 >”。
1. 在“DevOps Starter”边栏选项卡的“选择 Azure 服务以部署应用程序”页面上，确保已选中“Windows Web 应用”磁贴并单击“下一页:    创建 >”。
1. 在“DevOps Starter”边栏选项卡的“选择存储库和订阅”页面上，单击”授权”  。

    > **注意**：此时会显示“授权 Azure GitHub Actions”弹出 Web 浏览器窗口。

1. 在“授权 Azure GitHub Actions”弹出窗口中查看所需权限，然后单击“授权 AzureGithubActions” 。

    > **注意**：此时会将该 Web 弹出浏览器窗口重定向到 Azure DevOps 站点，系统会提示你提供 Azure DevOps 信息。

1. 看到提示时，请在弹出 Web 浏览器窗口中单击“继续”。
1. 返回至“DevOps Starter”边栏选项卡的“选择存储库和订阅”页面，指定以下设置并单击“查看 + 创建”  ：

    | 设置 | Value |
    | ------- | ----- |
    | 组织 | GitHub 帐户名称 |
    | 存储库 | **az400m08l01** |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | Web 应用名称 | azurewebsites.net DNS 命名空间中任何有效的、全局唯一的主机名 |
    | 位置 | 可以预配 Azure Web 应用的任何 Azure 区域的名称。 **注意**：等待预配完成。 |

   > 这大约需要 1 分钟。 在“Deploy_DevOps_Project_az400m08l01\| 概述”边栏选项卡上，单击“转到资源” 。
   > 在“az400m08l01”边栏选项卡的“GitHub 工作流”磁贴上单击“授权”  。 在“GitHub 授权”边栏选项卡上再次单击“授权” 。

1. 返回至“az400m08l01”边栏选项卡，在“GitHub 工作流”磁贴上监视操作进度 。
1. **注意**：等待 GitHub 工作流的构建、部署和功能测试作业完成。
1. 这大约需要 5 分钟。
1. 任务 2：查看创建 DevOps Starter 项目的结果

    > 在此任务中，你将查看创建 DevOps Starter 项目的结果。 在显示 Azure 门户的 Web 浏览器窗口中的“az400m08l01”边栏选项卡上，查看“GitHub 工作流”部分，并验证构建、部署和功能测试作业是否成功完成    。

#### <a name="task-2-review-the-results-of-creating-the-devops-starter-project"></a>在“az400m08l01”边栏选项卡上，查看“Azure 资源”部分，并验证是否包含一个应用服务 Web 应用实例和相应的Application Insights资源 。

在“az400m08l01”边栏选项卡的顶部，请注意在上一个任务中创建的指向工作流文件和 GitHub 存储库的链接 。

1. 在“az400m08l01”边栏选项卡的顶部，单击指向 GitHub 存储库的链接。
1. 在 GitHub 存储库页面上，注意三个带有以下标签的文件夹：
1. **.github\workflows** - 包含 YAML 格式的工作流文件
1. **应用程序** - 包含示例网站的代码
1. **ArmTemplates** - 包含工作流用于预配 Azure 资源的 Azure 资源管理器模板

    - 在 GitHub 存储库页面，单击“.github/workflows”，然后单击 devops-starter-workflow.yml 条目  。
    - 在显示 devops-starter-workflow.yml 内容的 GitHub 存储库页面上检查其内容，并注意它包含构建、部署和功能测试作业定义   。
    - 在 GitHub 存储库页面的工具栏中单击“操作”。

1. 在 GitHub 存储库页面的“操作”选项卡上，在“所有工作流”部分中，单击表示最近运行的工作流的条目 。
1. 在工作流运行页面上，查看工作流状态，以及注释和工件的列表 。
1. 在 GitHub 存储库页面上的工具栏中单击“设置”，然后在“设置”选项卡上单击“机密”  。
1. 在“操作机密”窗格上，请注意 AZURE_CREDENTIALS 条目，它表示访问目标 Azure 订阅所需的凭据 。
1. 导航至 az400m08l01/Application/aspnet-core-dotnet-core/Pages/Index.cshtml GitHub 存储库页面，并单击右上角的铅笔图标切换到编辑模式。
1. 将行 19 更改为 `<div class="description line-1"> GitHub Workflow has been successfully updated</div>`。
1. 向下滚动到页面底部并单击“提交更改”。
1. 在 GitHub 存储库页面的工具栏中单击“操作”。
1. 在“所有工作流”部分，单击“Update Index.cshtml”条目 。
1. 在“devops-starter-workflow.yml”部分，监视部署进度并验证部署是否成功完成。
1. **注意**：注意：如果使用 azure/CLI@1 的操作失败，请将以下更改提交到 devops-starter-workflow.yml 文件（更改默认的 azure cli 版本）并验证其是否成功完成 ：
1. 切换到在 Azure 门户中显示 DevOps Starter 边栏选项卡的浏览器窗口，并单击“Application 终结点”条目旁边的“浏览”链接 。
1. 在新打开的 Web 浏览器窗口中，验证更新后的文本（表示在 GitHub 存储库中提交的更改）是否显示在 Web 应用主页上。
     > 练习 2：删除 Azure 实验室资源
     > <!-- {% raw %}) -->
     > ```
     >     - name: Deploy ARM Template
     >       uses: azure/CLI@v1
     >       continue-on-error: false
     >       with:
     >         azcliversion: 2.29.2
     >         inlineScript: |
     >           az group create --name "${{ env.RESOURCEGROUPNAME }}" --location "${{ env.LOCATION }}"
     >           az deployment group create --resource-group "${{ env.RESOURCEGROUPNAME }}" --template-file ./ArmTemplates/windows-webapp-template.json --parameters webAppName="${{ env.AZURE_WEBAPP_NAME }}" hostingPlanName="${{ env.HOSTINGPLANNAME }}" appInsightsLocation="${{ env.APPINSIGHTLOCATION }}" sku="${{ env.SKU }}"
     > ```
     > <!-- {% endraw %}) -->

1. 在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。
1. **注意**：记得删除所有不再使用的新建 Azure 资源。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>删除未使用的资源可确保不会出现意外费用。

任务 1：删除 Azure 实验室资源

>在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。

#### <a name="task-1-remove-the-azure-lab-resources"></a>运行以下命令，列出在本模块各实验室中创建的所有资源组：

通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

1. **注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。
1. 审阅

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].name" --output tsv
    ```

1. 在此实验室中，你使用 DevOps Starter 实现了部署 Azure Web 应用的 GitHub Action 工作流。

    ```sh
    az group list --query "[?starts_with(name,'az400m08l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    ><bpt id="p1">**</bpt>Note<ept id="p1">**</ept>: The command executes asynchronously (as determined by the --nowait parameter), so while you will be able to run another Azure CLI command immediately afterwards within the same Bash session, it will take a few minutes before the resource groups are actually removed.

## <a name="review"></a>Review

In this lab, you implemented a GitHub Action workflow that deploys an Azure web app by using DevOps Starter.
