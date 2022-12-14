---
lab:
  title: 使用 Azure Pipelines 启用持续集成
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
---

# <a name="enabling-continuous-integration-with-azure-pipelines"></a>使用 Azure Pipelines 启用持续集成

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

## <a name="lab-overview"></a>实验室概述

本实验室将教授你如何使用 Azure Pipelines 中的生成和发布功能为应用程序配置持续集成 (CI) 和持续部署 (CD)。 此可编写脚本的 CI/CD 系统是基于 Web 的跨平台系统，同时还提供了用于可视化复杂工作流的新式界面。 虽然我们不会在本实验室中演示所有跨平台的可能性，但需要指出的是，你还可以针对 iOS、Android、Java（使用 Ant，、Maven 或 Gradle）和 Linux 进行生成。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 根据模板创建基本生成管道。
- 跟踪和审查生成。
- 调用持续集成生成。

## <a name="estimated-timing-45-minutes"></a>预计用时：45 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预配置的 Parts Unlimited 团队项目。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 Parts Unlimited 模板生成一个新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的“新建项目名称”文本框中，键入“使用 Azure Pipelines 启用持续集成”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表中，找到“PartsUnlimited”模板，然后单击“选择模板” 。
1. 返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

### <a name="exercise-1-introduction-to-azure-devops-build"></a>练习 1：Azure DevOps 生成简介

在本练习中，你将根据模板创建基本的生成管道，跟踪和审查新的生成作业，并触发持续集成生成。

#### <a name="task-1-creating-a-basic-build-pipeline-from-a-template"></a>任务 1：从模板中创建基本生成管道

在此任务中，你将使用预定义的模板创建和配置生成管道。

1. 在显示你的 Azure DevOps 组织以及上一练习中生成的“使用 Azure Pipelines 启用持续集成”项目的 Web 浏览器中，选择垂直导航窗格中的“管道”部分，并确保显示“管道”视图  。

    > **注意**：你也可以通过导航到 [<https://dev.azure.com/>`<your-Azure-DevOps-account-name>` /Enabling%20Continuous%20Integration%20with%20Azure%20Pipelines) URL 直接访问项目页面，其中 `<your-Azure-DevOps-account-name>` 占位符代表你的帐户名称。

1. 在“管道”窗格上，将鼠标指针悬停在表示现有 PartsUnlimitedE2E 管道的条目上，以显示右侧的省略号 。
1. 单击该省略号，然后在下拉菜单中单击“编辑”。

    > **注意**：为了避免稍后在实验室中触发两个管道，请先针对模板创建的管道禁用 CI 触发器。

1. 在“PartsUnlimitedE2E”窗格的“任务”选项卡上，单击“触发器”选项卡，清除“启用持续集成”复选框，然后依次单击“保存并排队”、“保存”     。
1. 若要新建管道，请通过在 Azure DevOps 门户的垂直导航窗格中选择“管道”，导航回“管道”视图 。
1. 返回“管道”窗格，单击“新建管道”以创建新的生成管道 。

    > **注意**：生成管道的默认选项包含使用 YAML。 在本实验室中，你将使用经典编辑器。

1. 在“代码在哪里>?” 窗格中，单击页面底部的“使用经典编辑器”链接。

    > **注意**：首先需要配置源存储库。 所有主要的平台都可用，但这里我们只需要默认选项。 此生成将使用 PartsUnlimited 存储库的主分支 。

1. 确保已选择“Azure Repos Git”选项以及 PartsUnlimited 存储库和主分支条目，然后单击“继续”   。
1. 在“选择模板”窗格的“搜索”文本框中，键入“ASP.NET”，在结果列表中，选择“ASP.NET”模板，然后单击“应用”将此模板应用到生成定义    。

    > **注意**：请注意，有许多选项应涵盖我们所有的主流场景。 出于此处的目的，我们将仅使用基线 ASP.NET 模板来生成项目。 此生成管道的过程很容易遵循。 获取源后，Azure DevOps 将使用 NuGet 还原任何依赖包。 然后，将生成并测试该项目。 然后将结果发布到配置的目标。

1. 在“任务”选项卡上，查找“测试程序集”任务，右键单击并禁用所选任务  。
1. 选择“变量”选项卡并查看其内容。

    > **注意**：在此处可配置在生成期间要使用的特殊参数，例如配置或平台。

1. 选择“触发器”选项卡，然后选中“启用持续集成”复选框 。

    > **注意**：每当提交源更改时，都将自动调用生成。 借助触发器，你可以按计划（其他生成完成或对源进行更改时）自动调用生成。

1. 选择“选项”选项卡并查看其内容。

    > **注意**：此部分包括与生成工作流相关的各种选项。 请注意，通常将在任务本身的配置视图中针对特定的生成任务配置选项。

1. 选择 **“历史记录”** 选项卡。

    > **注意**：此时，该选项卡不包含任何条目，但它将显示你对生成定义进行更改的历史记录。

1. 选择“保存并排队”选项卡标题，然后在下拉菜单中选择“保存并排队”条目以保存新生成并将其排入队列 。

    > **注意**：可通过“项目设置” > “设置” > “保留策略”定义管道项目的保留时间  。 通过这些设置，你可以配置保留哪些管道运行以及保留多长时间。

1. 在“运行管道”窗格中，接受默认选项，然后单击“保存并运行” 。 随即将自动显示管道运行作业的“摘要”选项卡以及“已排队”状态 。

#### <a name="task-2-tracking-and-reviewing-a-build"></a>任务 2：跟踪和审查版本

在此任务中，你将跟踪并审查新的生成作业。

> **注意**：生成开始后，你将能够跟踪每个任务的控制台输出。

1. 在管道运行作业的“摘要”选项卡上的“作业”部分中，单击“代理作业 1”  。 随即将显示该作业的详细信息窗格。

    > **注意**：如果要查看以前的任务，可以滚动右窗格以查看其日志。

2. 成功完成生成后，在作业详细信息窗格上，单击左向箭头以返回到“摘要”视图。

    > **注意**：“摘要”视图提供有关生成的概述详细信息，包括有关提交、测试和项目的详细信息。

#### <a name="task-3-invoking-a-continuous-integration-build"></a>任务 3：调用持续集成版本

在此任务中，你将触发持续集成生成。

   > **注意**：在本练习的第一个任务中，你配置了生成以支持持续集成。 在此任务中，你将测试其功能。

1. 在显示 Azure DevOps 门户中的项目设置的 Web 浏览器窗口中，选择垂直导航窗格中的“存储库”部分，并确保显示“文件”视图 。
1. 在中间窗格中，导航到文件 PartsUnlimited-aspnet45/src/PartsUnlimitedWebsite/Views/Home/Index.cshtml，并选择它。
1. 在“Index.cshtml”窗格上，单击“编辑” 。
1. 在“Index.cshtml”窗格上，通过将 `ViewBag.Title = "Home Page";` 行更改为 `ViewBag.Title = "Lab Project Home Page";` 进行次要更新，然后单击“提交” 。
1. 在“提交”窗格上，接受默认的提交详细信息，然后单击“提交” 。

    > **注意**：这将自动触发生成。

1. 在垂直导航窗格中，选择“管道”部分，并确保显示“管道”视图 。
1. 在“管道”窗格中，确认其包含代表由你的更改所触发的新生成（请注意，其编号末尾包含 .2）的条目 。
1. 单击生成条目以显示其详细信息，并验证其是否已成功完成。

## <a name="review"></a>审阅

在本实验室中，你使用 Azure DevOps 门户根据模板创建了基本生成管道，跟踪和审查了生成，并调用了持续集成生成。
