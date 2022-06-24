---
lab:
  title: 实验室 09：使用发布入口控制部署
  module: 'Module 04: Design and implement a release strategy'
ms.openlocfilehash: 487310a0850c669952358c45828cf3c395459c05
ms.sourcegitcommit: d78aebd7b14277a53f152e26cea68a30b0e90d73
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/13/2022
ms.locfileid: "146276050"
---
# <a name="lab-09-controlling-deployments-using-release-gates"></a>实验室 09：使用发布入口控制部署

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

## <a name="lab-overview"></a>实验室概述

本实验室涵盖部署入口的配置，并详细介绍如何使用这些配置来控制 Azure Pipelines 的执行。 为说明其实现，你将为 Azure Web 应用配置两个环境下的发布定义。 只有当应用没有阻碍性 bug 时，才会部署到 Canary 环境，只有当 Azure Monitor 的 Application Insights 中没有活动警报时，才会标记 Canary 环境完成。

发布管道指定了在各种环境中部署应用程序时的端到端发布过程。 通过使用作业和任务，可以完全自动化到各种环境的部署。 理想情况下，你不需要同时向所有用户公开应用程序的最新更新。 最佳做法是分阶段公开更新，即向部分用户公开，监视其使用情况，并根据最初的一组用户的体验向其他用户公开。

通过审批和入口，你能够在发布中控制部署的开始和完成。 通过手动审批，你可以等待用户批准或拒绝部署。 使用发布入口，可以指定在版本提升到下一个环境之前，应用程序要满足的运行状况条件。 在任何环境部署之前或之后，将自动评估所有这些已指定的入口，直到它们通过或到了你所定义的超时时间并失败。

入口可以从部署前条件或部署后条件面板添加到发布定义的环境中。 可以在环境条件中添加多个入口，以确保发布的所有输入都符合标准。

示例：

- 部署前入口确保在将生成部署到环境之前，工作项或问题管理系统中没有处于活动状态的问题。
- 部署后入口确保在将已部署的应用发布到下一个环境之前，没有来自该应用的监视或事件管理系统的事件。

默认情况下，每个帐户包含 4 种类型的入口。

- 调用 Azure 函数：触发执行 Azure 函数并确保成功完成。
- 查询 Azure Monitor 警报：观察为活动警报配置的 Azure Monitor 预警规则。
- 调用 REST API：调用 REST API 并在返回成功响应后继续。
- 查询工作项：确保查询返回的匹配工作项数量在阈值内。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 配置发布管道。
- 配置发布入口。
- 测试发布入口。

## <a name="estimated-timing-75-minutes"></a>预计用时：75 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板预配置的 Parts Unlimited 团队项目和两个分别代表 Canary 和“生产”环境的 Azure Web 应用，你将通过 Azure Pipelines 将应用程序部署到其中 。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 ReleaseGates 模板生成一个新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的页面上的“新建项目名称”文本框中，键入“使用发布入口控制部署”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表中，在工具栏中，单击“DevOps 实验室”，选择 ReleaseGates 模板，然后单击“选择模板”  。
1. 返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-create-two-azure-web-apps"></a>任务 2：创建两个 Azure Web 应用

在此任务中，你将创建两个分别代表 Canary 和“生产”环境的 Azure Web 应用，你将通过 Azure Pipelines 将应用程序部署到其中 。

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。
1. 在 Azure 门户中，单击页面顶部搜索文本框右侧的 Cloud Shell 图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

    >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格中的 Bash 提示符下，运行以下命令以创建资源组（将 `<region>` 占位符替换为将托管两个 Azure Web 应用的 Azure 区域的名称，例如“westeurope”或“eastus”） ：

    > **注意**：可以通过运行以下命令找到可能的位置，使用 `<region>` 上的“Name”：`az account list-locations -o table`

    ```bash
    RESOURCEGROUPNAME='az400m10l01-RG'
    az group create -n $RESOURCEGROUPNAME -l '<region>'
    ```

1. 创建应用服务计划

    ```bash
    SERVICEPLANNAME='az400m01l01-sp1'
    az appservice plan create -g $RESOURCEGROUPNAME -n $SERVICEPLANNAME --sku S1
    ```

1.  使用唯一的应用名称创建两个 Web 应用。
 
     ```bash
     SUFFIX=$RANDOM$RANDOM
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n PU$SUFFIX-Canary
     az webapp create -g $RESOURCEGROUPNAME -p $SERVICEPLANNAME -n PU$SUFFIX-Prod
     ```

    > **注意**：记录 Canary Web 应用的名称。 本实验室中稍后会用到它。

1. 请等待预配过程完成，然后关闭 Cloud Shell 窗格。

#### <a name="task-3-configure-an-application-insights-resource"></a>任务 3：配置 Application Insights 资源

1. 在 Azure 门户中，使用页面顶部的“搜索资源、服务和文档”文本框搜索“Application Insights”，然后在结果列表中选择“Application Insights”  。
1. 在“Application Insights”边栏选项卡上，选择“+ 创建” 。
1. 在“Application Insights”边栏选项卡上的“基本设置”选项卡上，指定以下设置（其他设置保留默认值） ：

    | 设置 | 值 |
    | --- | --- |
    | 资源组 | az400m10l01-RG |
    | 名称 | 在上一个任务中记录的 Canary Web 应用的名称 |
    | 区域 | 之前在上一个任务中部署 Web 应用的同一 Azure 区域 |
    | 资源模式 | **经典** |

    > **注意**：请忽略弃用消息。 这是必要操作，可防止“启用持续集成 DevOps”任务失败，本实验室稍后会用到该任务。

1. 依次单击“查看 + 创建”、“创建”。 
1. 等待预配过程完成。
1. 在 Azure 门户中，导航到你在上一任务中创建的资源组“az400m10l01-RG”。
1. 在资源列表中，单击 Canary Web 应用。
1. 在 Canary Web 应用页面左侧垂直菜单中的“设置”部分，单击 Application Insights  。
1. 在“Application Insights”边栏选项卡上，单击“打开 Application Insights” 。
1. 在“更改资源”部分中，单击“选择现有资源”选项，在现有资源列表中，选择新创建的 Application Insight 资源，单击“应用”，提示确认后单击“是”   。
1. 等待更改生效，然后在“Application Insights”边栏选项卡上单击“查看 Application Insights 数据”链接。

    > **注意**：你将在此处创建监视器警报，并将在本实验室的后续部分使用。

1.  在 Application Insights 资源边栏选项卡上的“监视”部分中，单击“警报”，然后单击“创建”>“警报规则”  。
1.  在“创建警报规则”边栏选项卡上的“条件”部分，单击“添加条件”链接  。 
1.  在“选择信号”边栏选项卡上的“按信号名称搜索”文本框中，键入“失败的请求”并选择它  。 
1.  在“配置信号逻辑”边栏选项卡上的“警报逻辑”部分中，将“阈值”保留设置为“静态”，在“阈值”文本框中，键入 0，然后单击“完成”      。

    > **注意**：每当最近 5 分钟内失败的请求数大于 0 时，该规则将生成警报。

1.  返回“创建警报规则”窗格，转到“详细信息”，指定以下设置并填写信息（其他设置保留默认值），然后单击“查看 + 创建”>“创建”  ：

    | 设置 | 值 |
    | --- | --- |
    | 警报规则名称 | PartsUnlimited_FailedRequests |
    | 严重性 | **2- 警告** |
    | 自动解决警报 | 已清除 |

    > **注意**：指标警报规则最多可能需要 10 分钟才能激活。

    > **注意**：你可以根据不同的指标创建多个警报规则，例如可用性小于 99%、服务器响应时间大于 5 秒或服务器异常大于 0 个

### <a name="exercise-1-configure-the-release-pipeline"></a>练习 1：配置发布管道

在本练习中，你将配置发布管道。

#### <a name="task-1-update-release-tasks"></a>任务 1：更新发布任务

在此任务中，你将更新发布任务。

1. 在实验室计算机上，在 Azure DevOps 门户中切换到显示“使用发布入口控制部署”项目的浏览器窗口，在垂直导航窗格中，选择 Pipelines ，然后在 Pipelines 部分，单击“发布”   。
1. 在“发布”视图的 PartsUnlimited-CD 窗格中，单击“编辑”  。

    > **注意**：此管道包含两个阶段，分别叫做“Canary 环境”和“生产” 。

1. 在“管道”选项卡上的“项目”矩形中，单击 PartsUnlimited-CI 生成项目右上角的“持续部署触发器”按钮   。
1. 如果 PartsUnlimited-CI 生成的持续部署触发器处于禁用状态，请切换开关以启用它。 将所有其他设置保留为默认值，并单击右上角的 x 标记关闭“持续部署触发器”窗格 。
1. 在“Canary 环境”阶段中，单击“1 个作业, 2 个任务”标签，并查看此阶段中的任务 。

    > **注意**：Canary 环境有 2 个任务，一个是将包发布到 Azure Web 应用，另一个是在部署后启用对应用程序的持续监视。

1. 在“所有管道”>“PartsUnlimited-CD”窗格上，确保已选择“Canary 环境”阶段 。 在“Azure 订阅”下拉列表中，选择你的 Azure 订阅，然后单击“授权” 。 如果出现提示，请使用在 Azure 订阅中具有所有者角色的用户帐户进行身份验证。
1. 在“应用服务名称”下拉列表中，选择 Canary Web 应用的名称 。

    > **注意**：你可能需要单击“刷新”按钮。

1.  在“Application Insights 资源组名称”下拉列表中，选择 az400m10l01-RG 条目 。
1.  在“Application Insights 资源名称”下拉列表中，选择 Canary Application Insights 资源的名称，该名称应与 Canary Web 应用的名称匹配  。 
1.  在 Canary 环境的代理阶段，右键单击“启用连续监视”和“禁用所选任务”   
1.  在“所有管道”>“PartsUnlimited-CD”窗格上，单击“任务”选项卡，然后在下拉列表中选择“生产”  。
1.  选中“生产”阶段后，在“Azure 订阅”下拉列表中，选择“可用 Azure 服务连接”下显示的用于“Canary 环境”阶段的 Azure 订阅，因为我们在授权订阅使用之前就已经创建了服务连接   。
1. 在“应用服务名称”下拉列表中，选择 Prod Web 应用的名称 。
1. 在“所有管道”>“PartsUnlimited-CD”窗格上，单击“保存”，然后在“保存”对话框中，单击“确定”   。
1. 在显示“使用发布入口控制部署”项目的浏览器窗口中，在垂直导航窗格的“管道”部分，单击“管道”  。
1. 在“管道”窗格上，单击代表 PartsUnlimited-CI 生成管道的条目，然后在 PartsUnlimited-CI 窗格上单击“运行管道”   。
1. 在“运行管道”窗格上，接受默认设置，然后单击“运行”以触发管道 。 等待生成管道完成。

    > **注意**：生成成功后，将自动触发发布，并将应用程序部署到两个环境中。

1. 在垂直导航窗格的 Pipelines 部分，单击“发布”，然后在 PartsUnlimited-CD 窗格中，单击代表最新发布的条目  。
1. 在“PartsUnlimited-CD”>“Release-1”边栏选项卡上，跟踪发布进度并验证两个 Web 应用的部署是否成功完成。
1. 切换到 Azure 门户界面，导航到资源组 az400m10l01-RG，在资源列表中，单击 Canary Web 应用，在 Web 应用边栏选项卡上，单击“浏览”，并验证网页是否在新的 Web 浏览器选项卡中成功加载  。
1. 关闭显示 Parts Unlimited 网站的 Web 浏览器选项卡。
1. 切换到 Azure 门户界面，导航到资源组 az400m10l01-RG，在资源列表中，单击“生产”Web 应用，在 Web 应用边栏选项卡上，单击“浏览”，并验证网页是否在新的 Web 浏览器选项卡中成功加载  。
1. 关闭显示 Parts Unlimited 网站的 Web 浏览器选项卡。

    > **注意**：现在，应用程序已经配置了 CI/CD。 在下一个练习中，我们将在发布管道中设置入口。

### <a name="exercise-2-configure-release-gates"></a>练习 2：配置发布入口

在本练习中，你将在发布管道中设置入口。

#### <a name="task-1-configure-pre-deployment-gates"></a>任务 1：配置部署前入口

在此任务中，你将配置部署前入口。

1. 切换到显示 Azure DevOps 门户的 Web 浏览器窗口，在垂直导航窗格中的 Pipelines 部分，单击“发布”，然后在 PartsUnlimited-CD 窗格中单击“编辑”   。
1. 在“所有管道”>“PartsUnlimited-CD”窗格上，在代表“Canary 环境”阶段的矩形的左边缘上，单击代表“部署前条件”的椭圆形  。
1. 在“部署前条件”窗格上，将“部署前审批”滑块设置为“启用”，然后在“审批者”文本框中，键入并选择你的 Azure DevOps 帐户名称   。
1. 在“预部署条件”窗格中，将“入口”滑块设置为“启用”，单击“+ 添加”，然后单击弹出窗口中的“查询工作项”    。
1. 在“部署前条件”窗格上“查询工作项”部分的“查询”下拉列表中，选择“共享查询”下的 Bug，将“上限阈值”的值设置为 0      。

    > **注意**：基于“上限阈值”设置的值，如果此查询返回任何活动的 bug 工作项，则发布入口将失败。

1.  在“部署前条件”窗格上，将“评估前的延迟时间”设置的值保留为“0 分钟”  。 

    > **注意**：“评估前的延迟时间”表示第一次评估添加的入口之前的时间。 如果未添加任何入口，则部署将等待一段时间，然后再继续。 要使入口函数可以初始化且稳定（它可能需要一些时间才能开始返回准确的结果），我们在评估结果之前配置一个延迟，然后将其用于确定应批准还是拒绝部署。

1. 在“部署前条件”窗格上，展开“评估选项”部分并配置以下选项 ：

    - 将“重新评估入口的间隔时间”值设置为“5 分钟” 。

    > **注意**：“重新评估入口的间隔时间”表示所有入口的每次评估时间间隔。 在每个采样间隔，新请求将同时发送到每个入口以获取新结果。 采样间隔必须大于任何已配置入口的最长典型响应时间，以便有时间接收所有响应。

    - 将“入口失败后超时时间”值设置为“8 分钟” 。

    > **注意**：“入口失败后超时时间”表示所有入口的最大评估周期。 如果在同一采样间隔内所有入口都成功之前已达到超时时间，则部署将被拒绝。 我们可以为超时指定的最小值为 6 分钟，可为采样间隔指定的最小值为 5 分钟。

    > **注意**：在这种情况下，触发发布时，入口将在 0 到 5 分钟之内验证样本 。 如果结果为“通过”，则将发送通知以供审批。 如果结果为“失败”，则发布将在 8 分钟后超时。

    > **注意**：实际上，这些值可能跨越几个小时。

1. 在“部署前条件”窗格上，选择“在成功的入口上，请求批准”单选按钮 。
1. 单击右上角的 x 标记，关闭“部署前条件”窗格 。 保存发布管道中的更改。
1. 为了使“查询工作项”入口正常工作，项目生成服务需要对 Azure Boards 查询具有“读取”权限 。
1. 在 Azure DevOps 门户的垂直导航窗格中，将鼠标指针悬停在 Boards 上，在按住 Ctrl 键的同时单击“查询”，以使用“查询”窗格打开单独的浏览器选项卡   。
1. 在 Boards 视图的“查询”窗格上，单击“全部”以获取包含所有查询的列表  。
1. 右键单击“共享查询”文件夹，然后选择“安全性…”打开“共享查询权限”窗格  。
1. 在“共享查询权限”窗格上的“搜索用户或组”字段中，键入或粘贴“使用发布入口生成服务控制部署”（[项目名称] 生成服务），然后单击找到的一个标识  。

    > **注意**：必须按照如上所述搜索“使用发布入口生成服务控制部署”用户，因为该用户尚未显示为“用户”列表的成员 。 不要将“项目集合生成服务”用户与“项目生成服务”混淆 。

1. 在“用户”列表中选择“使用发布入口生成服务控制部署”用户，然后在右侧将“读取”权限设置为“允许”   。
1. 单击右上角的 x 标记，关闭“共享查询权限”窗格 。
1. 导航回到浏览器选项卡，其中发布管道仍处于打开状态。

#### <a name="task-2-configure-post-deployment-gates"></a>任务 2：配置部署后入口

在此任务中，你将启用 Canary 环境的部署后入口。

1.  返回“所有管道”>“PartsUnlimited-CD”窗格，在代表“Canary 环境”阶段的矩形的右边缘上，单击代表“部署后条件”的椭圆形  。
1.  在“部署后条件”窗格中，将“入口”滑块设置为“启用”，单击“+ 添加”，然后单击弹出菜单中的“查询 Azure Monitor 警报”    。
1.  在“部署后条件”窗格“查询 Azure Monitor 警报”部分的“Azure 订阅”下拉列表中，选择代表与 Azure 订阅的连接的“服务连接”条目，然后在“资源组”下拉列表中选择“az400m10l01-RG”条目     。
1.  在“部署后条件”窗格上，展开“评估选项”并配置以下选项 ：

- 将“重新评估入口的间隔时间”值设置为“5 分钟” 。
- 将“入口失败后超时时间”值设置为“8 分钟” 。
- 选择“在成功的入口上，请求批准”选项。

    > **注意**：采样间隔和超时一起作用，确保入口以适当的间隔调用函数，如果在超时周期同一采样间隔内未成功执行函数，则将拒绝部署。

1. 单击右上角的 x 标记，关闭“部署后条件”窗格 。
1. 返回 PartsUnlimited-CD 窗格，单击“保存”，然后在“保存”对话框中，单击“确定”   。

### <a name="exercise-3-test-release-gates"></a>练习 3：测试发布入口

在本练习中，你将通过更新应用程序来测试发布入口，这将触发部署。

#### <a name="task-1-update-and-deploy-application-after-adding-release-gates"></a>任务 1：添加发布入口后更新和部署应用程序

在此任务中，你将在启用发布入口的情况下跟踪发布过程。

1. 在显示 Azure DevOps 门户的浏览器窗口的垂直导航窗格中，选择“发布”。
1. 单击“创建发布”，然后在“创建新发布”窗格中，单击“创建”  。
1. 在 Azure DevOps 门户中，在垂直导航窗格的“管道”部分，单击“发布” 。
1. 在“发布”选项卡上，单击 PartsUnlimited-CD/Release-2 条目并查看部署到 Canary 环境的进度  。
1. 在代表“Canary 环境”阶段的矩形的左边缘上，单击代表“预部署条件”的椭圆，这时可能会被标记为“评估入口”或“部署前入口失败”   。
1. 在“Canary 环境”窗格上，请注意“查询工作项”入口已失败 。

    > **注意**：这表明存在活动的工作项。 需要关闭这些工作项后才能继续。 下一次采样时间将在 5 分钟后。

1. 打开一个新的浏览器选项卡，导航到 Azure DevOps 门户，在垂直导航窗格中，选择 Boards，然后在 Boards 部分选择“查询”  。
1. 在 Boards 视图的“查询”窗格上，单击“所有”选项卡  。
1. 在“查询”窗格“所有”选项卡上的“共享查询”部分，单击 Bug 条目，然后在“查询”>“共享查询”>“Bug”窗格中，单击“运行查询”     。
1. 验证查询是否返回名为“Canary 环境中的磁盘空间不足”的工作项，并且该工作项的状态为“新建” 。

    > **注意**：我们假设基础结构团队已解决磁盘空间问题。

1. 单击“Canary 环境中的磁盘空间不足”条目。
1. 在“Canary 环境中的磁盘空间不足”窗格左上角的“状态”标签旁边，单击“新建”，然后在下拉列表中，单击“关闭”，再单击“保存”    。
1. 切换回“Canary 环境”窗格，并等待通过第二次评估。

    > **注意**：如果第二次评估已失败，则将鼠标指针悬停在代表“Canary 环境”阶段的矩形上以显示“重新部署”选项，单击“重新部署”，然后在“Canary 环境”边栏选项卡上单击“部署”并监视部署前入口的处理状态    。

    > **注意**：评估成功后，你将看到部署前审批请求。

1. 单击“审批者”，然后单击“批准”将部署排入 Canary 环境的队列 

    > **注意**：成功部署到 Canary 环境后，我们将看到部署后入口正在运行，该入口使用 Application Insights 来检测是否存在针对新部署应用程序的失败请求。

1. 要触发失败的请求，请切换到显示 Azure 门户的 Web 浏览器窗口，导航到 Canary Azure Web 应用边栏选项卡，然后单击“浏览” 。 这将打开一个新的浏览器选项卡，其中显示 PartsUnlimited 网站。
1. 在 PartsUnlimited 网站上，单击“更多”。

    > **注意**：网站的此部分故意配置错误，以便触发失败的请求。

1. 返回到 PartsUnlimited 网站的主页，再次单击“更多”，然后重复此步骤几次。
1. 验证 Application Insights 是否检测到失败的请求，操作方法：导航到 Canary Web 应用页面的 Application Insights 边栏选项卡，然后在 Application Insights 边栏选项卡上单击“警报”，并验证该页面是否列出了一个或多个 Sev 2 警报  。

    > **注意**：由于异常触发了警报，因此“查询 Azure Monitor”入口将失败。 反过来，这将阻止部署到“生产”环境。

### <a name="exercise-4-remove-the-azure-lab-resources"></a>练习 4：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m10l01-RG')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m10l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本实验室中，你配置了发布管道，然后配置并测试了发布入口。
