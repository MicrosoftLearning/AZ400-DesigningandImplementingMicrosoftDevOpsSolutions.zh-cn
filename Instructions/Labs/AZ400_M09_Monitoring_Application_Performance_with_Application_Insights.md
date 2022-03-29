---
lab:
  title: 实验室 21：使用 Application Insights 监视应用程序性能
  module: 'Module 09: Implement continuous feedback'
ms.openlocfilehash: f6bd76e03bef3782b5aabf447711974f11bf823a
ms.sourcegitcommit: f72fcf5ee578f465b3495f3cf789b06c530e88a4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/04/2022
ms.locfileid: "139262547"
---
# <a name="lab-21-monitoring-application-performance-with-application-insights"></a>实验室 21：使用 Application Insights 监视应用程序性能
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-overview"></a>实验室概述

Application Insights 是多个平台上面向 Web 开发人员的可扩展应用程序性能管理 (APM) 服务。 你可以使用它来监视实时 Web 应用程序。 它将自动检测性能异常，并且包含了强大的分析工具来帮助诊断问题，并帮助持续改进性能和可用性。 它适用于本地云、混合云或任何公有云中托管的各种平台（包括 .NET、Node.js 和 Java EE）中的应用。 它通过各种开发工具中的可用连接点与 DevOps 流程集成。 它还允许通过与 Visual Studio App Center 集成来监视和分析来自移动应用的遥测。

在本实验室中，你将了解如何将 Application Insights 添加到现有 Web 应用程序，以及如何通过 Azure 门户监视应用程序。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 部署 Azure 应用服务 Web 应用
- 使用 Application Insights 生成和监视 Azure Web 应用应用程序流量
- 使用 Application Insights 调查 Azure Web 应用性能
- 使用 Application Insights 跟踪 Azure Web 应用使用情况
- 使用 Application Insights 创建 Azure Web 应用警报

## <a name="lab-duration"></a>实验室时长

-   估计时间：60 分钟

## <a name="instructions"></a>说明

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

在此任务中，你将使用 Azure DevOps 演示生成器，基于 Parts Unlimited 模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **注意**：有关此站点的详细信息，请参阅 https://docs.microsoft.com/en-us/azure/devops/demo-gen 。

1.  单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1.  在“新建项目”页面上的页面的“新建项目名称”文本框中，键入“监视应用程序性能”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1.  在模板列表中，选择“PartsUnlimited”模板，然后单击“选择模板” 。
1.  返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-create-azure-resources"></a>任务 2：创建 Azure 资源

在此任务中，你将使用 Azure 门户的 Cloud Shell 创建 Azure Web 应用和 Azure SQL 数据库。

> **注意**：本实验室涉及将 Parts Unlimited 站点部署到 Azure 应用服务。 为满足此需求，你需要启动必要的基础结构。 

1.  从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。
1. 在 Azure 门户的工具栏中，单击搜索文本框右侧的“Cloud Shell”图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。
    >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1.  在 Cloud Shell 窗格中的 Bash 提示符下，运行以下命令以创建资源组（将 `<region>` 占位符替换为离你最近的 Azure 区域的名称，例如“eastus”） 。

    ```bash
    RESOURCEGROUPNAME='az400m17l01a-RG'
    LOCATION='<region>'
    az group create --name $RESOURCEGROUPNAME --location $LOCATION
    ```

1.  若要创建 Windows 应用服务计划，请运行以下命令：

    ```bash
    SERVICEPLANNAME='az400l17-sp'
    az appservice plan create --resource-group $RESOURCEGROUPNAME \
        --name $SERVICEPLANNAME --sku B3 
    ```
    > **注意**：如果 `az appservice plan create` 命令失败并显示以 `ModuleNotFoundError: No module named 'vsts_cd_manager'` 开头的错误消息，请运行以下命令，然后重新运行失败的命令。

    ```bash
    az extension remove --name appservice-kube
    az extension add --yes --source "https://aka.ms/appsvc/appservice_kube-latest-py2.py3-none-any.whl"
    ```
1.  创建具有唯一名称的 Web 应用。

    ```bash
    WEBAPPNAME=partsunlimited$RANDOM$RANDOM
    az webapp create --resource-group $RESOURCEGROUPNAME --plan $SERVICEPLANNAME --name $WEBAPPNAME 
    ```

    > **注意**：记录 Web 应用的名称。 本实验室中稍后会用到它。

1. 现在该创建 Application Insights 实例了。

    ```bash
    az monitor app-insights component create --app $WEBAPPNAME \
        --location $LOCATION \
        --kind web --application-type web \
        --resource-group $RESOURCEGROUPNAME
    ```

    > **注意**：如果收到“该命令需要扩展应用程序见解。 是否立即安装?”的提示，请键入“是”，然后按 Enter。

1. 让我们将 Application Insights 连接到 Web 应用程序。

    ```bash
    az monitor app-insights component connect-webapp --app $WEBAPPNAME \
        --resource-group $RESOURCEGROUPNAME --web-app $WEBAPPNAME
    ```

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

### <a name="exercise-1-monitor-an-azure-app-service-web-app-by-using-azure-application-insights"></a>练习 1：使用 Azure Application Insights 监视 Azure 应用服务 Web 应用

在本练习中，你将使用 Azure DevOps 管道将 Web 应用部署到 Azure 应用服务，生成针对 Web 应用的流量，并使用 Application Insights 查看 Web 流量、调查应用程序性能、跟踪应用程序使用情况和配置警报。

#### <a name="task-1-deploy-a-web-app-to-azure-app-service-by-using-azure-devops"></a>任务 1：使用 Azure DevOps 将 Web 应用部署到 Azure 应用服务

在此任务中，你将使用 Azure DevOps 管道将 Web 应用部署到 Azure。

> **注意**：我们在本实验室中使用的示例项目包括一个持续集成生成，我们将在不进行任何修改的情况下使用该生成。 此外，还有一个持续交付发布管道，该管道需要进行一些细微更改，才能部署到你在上一任务中实现的 Azure 资源。 

1.  切换到显示 Azure DevOps 门户中“监视应用程序性能”项目的 Web 浏览器窗口，在垂直导航窗格中，选择“管道”，然后在“管道”部分中选择“发布”   。
1.  在发布管道列表中的“PartsUnlimitedE2E”窗格上，单击“编辑” 。 
1.  在“所有管道 > PartsUnlimitedE2E”窗格上，单击表示“开发”阶段的矩形，在“开发”窗格上，单击“删除”，然后在“删除阶段”对话框中，单击“确认”     。
1.  返回“所有管道 > PartsUnlimitedE2E”窗格，单击表示 QA 阶段的矩形，在“QA”窗格上，单击“删除”，然后在“删除阶段”对话框中，单击“确认”     。
1.  返回“所有管道 > PartsUnlimitedE2E”窗格，在表示“生产”阶段的矩形中，单击“1 个作业，1 项任务”链接  。
1.  在显示“生产”阶段任务列表的窗格中，单击表示“Azure 应用服务部署”任务的条目。
1.  在“Azure 应用服务部署”窗格的“Azure 订阅”下拉列表中，选择表示你在本实验室中使用的 Azure 订阅的条目，然后单击“授权”以创建相应的服务连接  。 出现提示时，使用在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色的帐户登录。
1.  在“所有管道 > PartsUnlimitedE2E”窗格的“任务”选项卡处于活动状态后，单击“管道”选项卡标头以返回管道图表  。 
1.  在图表中，单击矩形左侧表示“生产”阶段的“预部署条件”椭圆符号 。
1.  在“预部署条件”窗格的“选择触发器”部分中，选择“发布后”  。

    > **注意**：这将在项目的生成管道成功后调用发布管道。

1.  在“所有管道 > PartsUnlimitedE2E”窗格的“管道”选项卡处于活动状态后，单击“变量”选项卡标头  。
1.  在变量列表中，将 WebsiteName 变量的值设置为与你先前在本实验室中创建的 Azure 应用服务 Web 应用的名称匹配。
1.  在窗格右上角，单击“保存”，出现提示时，在“保存”对话框中再次单击“确定”  。

    > **注意**：现在，发布管道已就位，我们可以预料对主分支的任何提交都将触发生成和发布管道。

1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，在垂直导航窗格中单击“存储库”。 
1.  在“文件”窗格中，导航到并选择 PartsUnlimited-aspnet45/src/PartsUnlimitedWebsite/Web.config 文件 。

    > **注意**：此应用程序已有 Application Insights 密钥和 SQL 连接的配置设置。 

1. 在 Web.config 窗格上，查看引用 Application Insights 密钥以及 SQL 连接的行：

    ```xml
    <add key="Keys:ApplicationInsights:InstrumentationKey" value="0839cc6f-b99b-44b1-9d74-4e408b7aee29" />
    ```

    ```xml
    <connectionStrings>
       <add name="DefaultConnectionString" connectionString="Server=(localdb)\mssqllocaldb;Database=PartsUnlimitedWebsite;Integrated Security=True;" providerName="System.Data.SqlClient" />
    </connectionStrings>
    ```

    > **注意**：部署后，你将在 Azure 门户中修改这些设置的值，以表示你先前在实验室中部署的 Azure Application Insights 和 Azure SQL 数据库。

    > **注意**：现在，只需在文件末尾添加一个空行，即可触发生成和发布过程，而无需修改任何相关代码

1. 在“Web.config”窗格上，单击“编辑”，在文件末尾添加一个空行，单击“提交”，然后在“提交”窗格中再次单击“提交”    。

    > **注意**：将开始新的生成并最终导致部署到 Azure。 不要等待它完成，而是继续执行下一步。

1.  切换到显示 Azure 门户的 Web 浏览器，并导航到你先前在实验室中预配的应用服务 Web 应用。 
1.  在应用服务 Web 应用边栏选项卡上，在左侧的垂直菜单中单击，在“设置”部分中，单击“配置”选项卡 。
1. 在“应用程序设置”列表中，单击 APPINSIGHTS_INSTRUMENTATIONKEY 条目 。 （如果未看到此条目，请在“设置”下选择 Application Insights 并启用 Aplication Insights，然后选择“应用”）   
1.  在“添加/编辑应用程序设置”边栏选项卡上，复制“值”文本框中的文本，然后单击“取消”  。

    > **注意**：这是在部署应用服务 Web 应用期间添加的默认设置，已包含 Application Insights ID。 我们需要添加应用预期的新设置，赋予其不同的名称和相同的值。 这是本示例的特殊要求。

1.  在“应用程序设置”部分，单击“+ 新建应用程序设置” 。
1. 在“添加/编辑应用程序设置”边栏选项卡的“名称”文本框中，键入 Keys:ApplicationInsights:InstrumentationKey，在“值”文本框中，键入复制到剪贴板中的字符串，然后单击“确定”    。 选择“保存”。

    > **注意**：对应用程序设置和连接字符串进行更改将触发重启 Web 应用。

1.  切换回显示 Azure DevOps 门户的 Web 浏览器窗口，在垂直导航窗格中，选择“管道”，然后在“管道”部分中单击表示最近运行的生成管道的条目 。
1.  如果生成尚未完成，跟踪生成直至完成，然后在垂直导航窗格中的“管道”部分中单击“发布”，在“PartsUnlimiteE2E”窗格中单击“Release-1”，并按照发布管道进行操作直至发布完成   。
1.  切换到显示 Azure 门户的 Web 浏览器窗口，然后在“应用服务 Web 应用”边栏选项卡左侧的垂直菜单栏中单击“概述” 。 
1.  在右侧的“Essentials”部分中，单击 URL 链接 。 这将自动打开另一显示 Parts Unlimited 网站的 Web 浏览器标签页。
1.  验证 Parts Unlimited 网站是否按预期加载。 

#### <a name="task-2-generate-and-review-application-traffic"></a>任务 2：生成和查看应用程序流量

在此任务中，你将生成针对你在上一任务中部署的应用服务 Web 应用的流量，并查看由与 Web 应用关联的 Application Insights 资源收集的数据。

1.  在显示 Parts Unlimited 网站的 Web 浏览器窗口中，浏览其页面以生成一些流量。
1.  在 Parts Unlimited 网站上，单击“Brakes”菜单项 。
1.  在浏览器窗口顶部的 URL 文本框中，将 1 附加到 URL 字符串的末尾，然后按 Enter，有效将 CategoryId 参数设置为 11   。 

    > **注意**：这将触发服务器错误，因为该类别不存在。 请多次刷新页面以生成更多错误。

1.  返回显示 Azure 门户的 Web 浏览器标签页。
1.  在显示 Azure 门户的 Web 浏览器标签页中，在“应用服务”Web 应用边栏选项卡左侧的垂直菜单栏中的“设置”部分，单击“Application Insights”条目以显示“Application Insights”配置边栏选项卡   。

    > **注意**：此边栏选项卡包括用于将 Application Insights 与不同类型的应用集成的设置。 虽然默认体验为跟踪和监视应用生成了大量数据，但 API 为更专业的场景和自定义事件跟踪提供了支持。

1.  在“Application Insights”配置边栏选项卡上，单击“查看 Application Insights 数据”链接 。
1.  查看生成的“Application Insights”边栏选项卡，该边栏选项卡显示了其中显示所收集数据的不同特征的图表，包括生成的流量和你在此任务早期触发的失败请求。

    > **注意**：如果没有立即看到任何内容，只需等待几分钟并刷新页面，直到日志开始显示在概述部分。

#### <a name="task-3-investigate-application-performance"></a>任务 3：调查应用程序性能

在此任务中，你将使用 Application Insights 来调查应用服务 Web 应用的性能。

1.  在“Application Insights”边栏选项卡左侧垂直菜单中的“调查”部分，单击“应用程序映射”  。

    > **注意**：应用程序映射可帮助你发现分布式应用程序的所有组件的性能瓶颈或热点失败。 映射的每个节点表示应用程序组件或其依赖项，以及运行状况 KPI 和警报状态。 可从任何组件单击以获得更详细的诊断，如 Application Insights 事件。 如果你的应用使用了 Azure 服务，你还可以单击进入与这些服务相关的 Azure 诊断，例如 SQL 数据库顾问建议。

1.  在“Application Insights”边栏选项卡左侧垂直菜单中的“调查”部分，单击“智能检测”  。

    > **注意**：当 Web 应用程序中存在潜在性能问题时，智能检测会自动向你发出警告。 它会对应用发送至 Application Insights 的遥测数据执行主动分析。 如果失败率中存在骤升或者客户端或服务器性能中存在异常模式，将收到警报。 此功能不需要任何配置。 它会在应用程序发送足够的遥测时运行。 但是，由于应用刚刚才部署完毕，因此还没有任何数据。

1.  在“Application Insights”边栏选项卡左侧垂直菜单中的“调查”部分，单击“实时指标”  。

    > **注意**：实时指标流让你能够探测生产中的实时 Web 应用程序的检测信号。 你可以选择并筛选指标和性能计数器进行实时监视，且服务不会受到任何影响。 还可以检查来自示例失败请求和异常的堆栈跟踪。

1.  返回显示 Parts Unlimited 网站的 Web 浏览器，浏览其页面以生成一些流量，包括一些服务器错误。 
1.  返回显示 Azure 门户的 Web 浏览器，以在实时流量到达时查看该流量。 
1.  在“Application Insights”边栏选项卡左侧垂直菜单中的“调查”部分，单击“事务搜索”  。

    > **注意**：事务搜索提供了一个灵活的界面来定位你需要回答问题的确切遥测。 

1.  在“事务搜索”边栏选项卡上，单击“查看最近 24 小时内的所有数据” 。

    > **注意**：结果包括所有遥测数据，可按多种属性对这些数据进行筛选。

1.  在“事务搜索”边栏选项卡上，在边栏选项卡中间，结果列表的正上方，单击“分组的结果” 。 

    > **注意**：这些结果根据常见属性进行了分组。

1.  在“事务搜索”边栏选项卡上，在边栏选项卡中间，结果列表的正上方，单击“结果”以返回原始视图，列出所有结果 。
1.  在“事务搜索”边栏选项卡顶部，单击“事件类型 = 选择的所有项”，在下拉列表中，清除“全选”复选框，然后在事件类型列表中选择“异常”复选框   。

    > **注意**：应显示一些表示你之前生成的错误的异常。 

1.  在结果列表中，单击其中一项异常。 这将显示“端到端事务详细信息”边栏选项卡，其中提供了请求上下文中异常的完整时间线视图。 
1.  在“端到端事务详细信息”边栏选项卡底部，单击“查看所有遥测” 。

    > **注意**：“遥测”视图提供了相同的数据，但数据格式不同。 在“端到端事务详细信息”边栏选项卡右侧，还可以查看异常本身的详细信息，例如其属性和调用堆栈。

1.  关闭“端到端事务详细信息”边栏选项卡，返回“事务搜索”边栏选项卡，在左侧的垂直菜单中，在“调查”部分中单击“可用性”   。

    > **注意**：将 Web 应用或网站部署到任何服务器后，可以设置测试，以监视其可用性和响应性。 Application Insights 将来自全球各地的 Web 请求定期发送到应用程序。 如果应用程序无响应或响应慢，它会提醒你。 

1.  在“可用性”边栏选项卡的工具栏中，单击“+ 添加测试” 。
1.  在“创建测试”边栏选项卡的“测试名称”文本框中，键入“主页”，将 URL 设置为应用服务 Web 应用的根目录，然后单击“创建”    。

    > **注意**：测试不会立即运行，因此不会有任何数据。 如果稍后再返回查看，应可看到已更新可用性数据，反映针对实时站点的测试。 现在不要等待更新。

1.  在“可用性”边栏选项卡左侧垂直菜单中的“调查”部分，单击“失败”  。

    > **注意**：“失败”视图将所有异常报表聚合到单个仪表板中。 在这里，你可以根据依赖项或异常等筛选条件轻松定位相关数据。 

1.  在“失败”边栏选项卡右上角的“排名前 3 的响应代码”列表中，单击表示 500 错误数的链接  。

    > **注意**：这将显示一个与此 HTTP 响应代码匹配的异常列表。 选择建议的异常将显示与你先前查看的异常视图相同的异常视图。

1.  在“失败”边栏选项卡左侧垂直菜单中的“调查”部分，单击“性能”  。

    > **注意**：“性能”视图提供了一个仪表板，该仪表板根据收集的遥测数据简化了应用程序性能的详细信息。

1.  在“性能”边栏选项卡左侧垂直菜单中的“监视”部分，单击“指标”  。

    > **注意**：Application Insights 中的指标是从应用程序遥测功能发送的度量值和事件计数。 它们可帮助检测性能问题，观察应用程序的用法趋势。 标准指标的范围很广泛，也可以创建自己的自定义指标和事件。 

1.  在“指标”边栏选项卡的筛选器部分中，单击“选择指标”，然后在下拉列表中选择“服务器请求”  。

    > **注意**：还可以使用拆分对数据进行分段。 

1.  在新显示的图表顶部，单击“应用拆分”，在生成的筛选器中的“选择值”下拉列表中，选择“操作名称”  。 

    > **注意**：这将根据服务器请求引用的页面（由图表中的不同颜色表示）拆分服务器请求。

#### <a name="task-4-track-application-usage"></a>任务 4：跟踪应用程序使用情况

> **注意**：Application Insights 提供了一组广泛的功能来跟踪应用程序使用情况。 

1.  在“指标”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“用户”  。

    > **注意**：虽然应用程序的用户还不多，但有一些可用数据。 

1.  在“用户”边栏选项卡上的主图下，单击“查看更多见解” 。 这将显示额外的数据，向下扩展边栏选项卡。
1.  向下滚动以查看有关地理位置、操作系统和浏览器的详细信息。

    > **注意**：还可以深入了解特定于用户的数据，以更好地了解特定于用户的使用模式。

1.  在“用户”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“事件”  。
1.  在“事件”边栏选项卡上的主图下，单击“查看更多见解” 。

    > **注意**：显示内容将包括目前基于站点使用情况引发的一系列内置事件。 可以通过编程方式添加包含自定义数据的自定义事件，以满足你的需要。

1.  在“事件”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“漏斗图”  。

    > **注意**：了解客户体验对你的业务而言至关重要。 如果应用程序的使用涉及多个步骤，则你需要了解大多数客户是否遵循了预期的流程。 Web 应用程序中通过一系列步骤完成的进度被称为“漏斗图”。 Azure Application Insights 漏斗图可用于深入了解你的用户，并监视分步转换率。

1.  在“漏斗图”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“用户流”  。

    > **注意**：用户流工具从你指定的初始页面视图、自定义事件或异常启动。 基于此给定的初始事件，用户流显示相应用户会话期间发生在其之前和之后的事件。 不同粗细的线显示用户遵循每条路径的次数。 “会话启动”节点表示会话的开始。 “会话结束”节点显示有多少用户在上一个节点之后没有生成页面视图或自定义事件，这可能与离开你的站点的用户相对应。

1.  在“用户流”边栏选项卡上，单击“选择事件”，在“编辑”窗格的“初始事件”下拉列表中的“页面视图”部分，选择“主页 - Parts Unlimited”条目，然后单击“创建图”      。

1.  在“用户流”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“保留期”  。

    > **注意**：Application Insights 中的“留存情况”功能可以帮助分析有多少用户回归到应用，以及他们以何频率执行特定的任务或达成目标。 例如，如果运行游戏网站，则可以会在输掉游戏后回归到网站的用户数与在获胜后回归的用户数进行比较。 此信息有助于改进用户体验和业务策略。

    > **注意**：“用户保留期分析”体验已转换为 Azure 工作簿

1.  在“保留期”边栏选项卡上，单击“保留期分析工作簿”，查看“整体保留期”图表，然后关闭边栏选项卡  。
1.  在“保留期”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“影响”  。

    > **注意**：“影响”分析网站属性（如加载时间）如何影响应用各个部件的转换率。 更准确地说，它可以发现页面视图的任何维度、自定义事件或请求对页面视图或自定义事件造成的影响。

    > **注意**：“影响分析”体验已转换为 Azure 工作簿

1.  在“影响”边栏选项卡上，单击“影响分析工作簿”，在“影响”边栏选项卡上的“选择的事件”下拉列表中的“页面视图”部分，选择“主页 - Parts Unlimited”，在“影响事件”中，选择“浏览产品 - Parts Unlimited”，查看结果，然后关闭边栏选项卡       。
1.  在“影响”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“队列”  。

    > **注意**：队列是具有某种共性的用户、会话、事件或操作集。 在 Application Insights 中，队列由分析查询定义。 如果你要反复分析特定的用户或事件集，队列可让你更灵活地准确表达所需的集。 队列的使用方式类似于筛选器，但队列的定义基于自定义分析查询，因此它们更具适应性且更复杂。 与筛选器不同，队列可以保存，因此可供其他团队成员重复使用。

1.  在“队列”边栏选项卡左侧垂直菜单中的“使用情况”部分，单击“更多”  。

    > **注意**：此边栏选项卡包括供查看的各种报表和模板。

1.  在“更多 \| 库”边栏选项卡的“使用情况”部分，单击“页面浏览量分析”并查看相应边栏选项卡的内容  。

    > **注意**：此特定报表提供了有关页面视图的见解。 默认情况下，还有许多其他可用报表，你可以自定义和保存新的报表。

#### <a name="task-5-configure-web-app-alerts"></a>任务 5：配置 Web 应用警报

1.  在“更多 \| 库”边栏选项卡的左侧垂直菜单的“监视”部分，单击“警报”  。 

    > **注意**：通过警报，可以设置在 Application Insights 度量值达到指定条件时执行操作的触发器。

1.  在“警报”边栏选项卡的工具栏中，单击“+ 新建警报规则” 。
1.  在“创建警报规则”边栏选项卡上，请注意，在“范围”部分，默认情况下将选中当前 Application Insights 资源 。 
1.  在“创建警报规则”边栏选项卡的“条件”部分，单击“添加条件”  。
1.  在“配置信号逻辑”边栏选项卡上，搜索并选择“失败的请求”指标 。
1.  在“配置信号逻辑”边栏选项卡上，向下滚动到“警报逻辑”部分，确保“阈值”设置为“静态”并将“阈值的值”设置为 1     。 

    > **注意**：这将在报告第二个失败的请求时触发警报。 默认情况下，将基于过去 5 分钟内的度量值聚合每分钟评估一次条件。 

1.  在“配置信号逻辑”边栏选项卡上，单击“完成” 。

    > **注意**：创建条件后，需要为它定义一个要执行的“操作组”。 

1.  返回到“创建警报规则”边栏选项卡，在“操作”部分，单击“选择操作组”，然后在“选择要附加到此警报规则的操作组”上，单击“+ 创建操作组”   。
1.  在“创建操作组”边栏选项卡的“基本信息”选项卡上，指定以下设置并单击“下一步:   通知 >”：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 az400m17l01b-RG |
    | 操作组名称 | az400m17-action-group |
    | 显示名称 | az400m17-ag |

1.  在“创建操作组”边栏选项卡的“通知”选项卡上，在“通知类型”下拉列表中选择“电子邮件/短信消息/推送/语音”。 “电子邮件/短信消息/推送/语音”边栏选项卡随即打开。 
1.  在“电子邮件/短信消息/推送/语音”边栏选项卡上，选择“电子邮件”复选框，在“电子邮件”文本框中，键入电子邮件地址，然后单击“确定”。
1.  返回到“创建操作组”边栏选项卡的“通知”选项卡，在“名称”文本框中，键入“电子邮件”并单击“下一步:     操作 >”：
1.  在“创建操作组”边栏选项卡的“操作”选项卡上，选择“操作组”下拉列表，查看可用选项而不进行任何更改，然后单击“查看 + 创建”。
1.  在“创建操作组”边栏选项卡的“查看 + 创建”选项卡中，单击“创建”
1.  返回“创建警报规则”边栏选项卡，在“警报规则详细信息”部分的“警报规则名称”文本框中，键入“az400m17 实验室警报规则”，查看其余警报规则设置而不修改它们，然后单击“创建警报规则”。
1.  切换到显示“Parts Unlimited”网站的 Web 浏览器窗口，在“Parts Unlimited”网站上，单击“Brakes”菜单项。
1.  在浏览器窗口顶部的 URL 文本框中，将 1 附加到 URL 字符串的末尾，然后按 Enter，有效将 CategoryId 参数设置为 11   。 

    > **注意**：这将触发服务器错误，因为该类别不存在。 请多次刷新页面以生成更多错误。

1.  大约五分钟后，检查你的电子邮件帐户，验证是否收到一封指示你定义的警报已触发的电子邮件。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m17l01')].name" --output tsv
    ```

1.  通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m17l01')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本练习中，你已使用 Azure DevOps 管道将 Web 应用部署到 Azure 应用服务，生成针对 Web 应用的流量，并使用 Application Insights 查看 Web 流量、调查应用程序性能、跟踪应用程序使用情况和配置警报。
