---
lab:
  title: 实验室 15：在 Azure 管道中实现安全性和合规性
  module: 'Module 07: Implement security and validate code bases for compliance'
ms.openlocfilehash: 4c067d30c00b9a07342df3b8390534ba10c4ecf1
ms.sourcegitcommit: 8dc3c31d31d99fd0a2bdd3fb1c7f8f2f04d39778
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/27/2022
ms.locfileid: "147422483"
---
# <a name="lab-15-implement-security-and-compliance-in-an-azure-pipeline"></a>实验室 15：在 Azure 管道中实现安全性和合规性

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

## <a name="lab-overview"></a>实验室概述

在本实验室中，你将使用 Mend Bolt 和 Azure DevOps 自动检测代码中易受攻击的开源组件、已过期库和许可证合规性问题。 你将使用 WebGoat，这是一款有意存在不安全性的网页应用程序，由 OWASP 维护，用于说明常见的 Web 应用安全问题。

[Mend](https://www.mend.io/) 是持续开源软件安全性和合规性管理的领导者。 无论编程语言、生成工具或开发环境如何，Mend 都会集成到生成流程。 它可在后台自动、持续且以静默方式工作，可在 Mend 不断更新的开源存储库权威数据库中检查开源组件的安全性、许可和质量。

Mend 提供了 Mend Bolt，这是一种轻量级的开源安全性和管理解决方案，专门开发用于与 Azure DevOps 集成。

> 注意：Mend Bolt 按项目工作，不提供实时警报功能，这需要完整的平台。 

通常将 Mend Bolt 推荐给希望在整个软件开发生命周期（从存储库到部署后阶段）以及所有项目和产品中自动化其开源管理的大型开发团队。

Azure DevOps 与 Mend Bolt 的集成将使你能够：

- 检测并修复易受攻击的开源组件。
- 为每个项目或生成过程生成全面的开源清单报告。
- 实施开源许可证合规性，包括依赖项许可证。
- 确定过时的开源库以及更新建议。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 激活 Mend Bolt。
- 运行生成管道并查看 Mend 安全性和合规性报表。

## <a name="estimated-timing-45-minutes"></a>预计用时：45 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本演练中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [Parts Unlimited MRP GitHub 存储库](https://www.github.com/microsoft/partsunlimitedmrp)。

#### <a name="task-1-create-and-configure-the-team-project"></a>任务 1：创建和配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 [WhiteSource-Bolt 模板（将重命名为 Mend Bolt）](https://azuredevopsdemogenerator.azurewebsites.net/?name=WhiteSource-Bolt&templateid=77362)生成一个新项目

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的“新建项目名称”文本框中，键入 Mend Bolt，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表的工具栏中，单击“DevOps 实验”，选择“WhiteSource Bolt”（将重命名为 Mend Bolt）模板，然后单击“选择模板”  。
1. 返回“新建项目”页面，如果系统提示你安装缺少的扩展，请选中“Mend Bolt”下方的复选框，然后单击“创建项目”  。

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

### <a name="exercise-1-implement-security-and-compliance-in-an-azure-pipeline-using-mend-bolt"></a>练习 1：使用 Mend Bolt 在 Azure 管道中实现安全性和合规性

在本演练中，利用 Mend Bolt 查找项目代码中的安全漏洞和许可证合规性问题，然后查看生成的报表。

#### <a name="task-1-activate-mend-bolt"></a>任务 1：激活 Mend Bolt

在此任务中，你将在新生成的 Azure Devops 项目中激活 Mend Bolt。

1. 实验室计算机的 Web 浏览器窗口中显示了 Azure DevOps 门户和打开的 Mend Bolt 项目，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“管道”部分和“Mend Bolt”选项（在垂直菜单栏的“部署组”选项下方）   。
1. 在“即将完成”窗格上，输入“工作电子邮件”和“公司名称”，在“国家/地区”下拉列表中，选择表示你所在国家/地区的条目，然后单击“开始使用”按钮，开始使用免费版的 Mend Bolt    。 这将自动打开新的浏览器选项卡，其中显示了“Bolt 入门”页面。
1. 切换回显示 Azure DevOps 门户的 Web 浏览器选项卡，并验证是否显示了“你正在使用免费版的 Mend Bolt”。

#### <a name="task-2-trigger-a-build"></a>任务 2：触发生成

在此任务中，你将在基于 Java 代码的 Azure DevOps 项目中触发生成。 你将使用 Mend Bolt 扩展来确定此代码中易受攻击的组件。

1. 在实验室计算机上左侧的垂直菜单栏中，导航到“管道”部分，单击“WhileSourceBolt”，再单击“运行管道”，然后在“运行管道”窗格中，单击“运行”    。
1. 在生成窗格的“摘要”选项卡上的“作业”部分，单击“阶段 1”，然后监视生成过程的进度  。

    > **注意**：此生成可能需要花费几分钟时间完成。 生成定义由以下任务构成：

    | 任务 | 使用情况 |
    | ---- | ------ |
    | ![npm](images/m07/npm.png) npm |  安装和发布生成所需的 npm 包 |
    | ![maven](images/m07/maven.png) Maven |  使用提供的 pom xml 文件生成 Java 代码 |
    | ![Mendbolt](images/m07/whitesourcebolt.png) Mend Bolt |  扫描提供的工作目录/根目录中的代码，检测安全漏洞和存在问题的开源许可证 |
    | ![copy-files](images/m07/copy-files.png) 复制文件 |  使用匹配模式将生成的 JAR 文件从源复制到目标文件夹 |
    | ![publish-build-artifacts](images/m07/publish-build-artifacts.png)发布生成项目 |  发布生成产生的项目 |

1. 生成完成后，导航回“摘要”选项卡并查看“测试和覆盖率”部分 。

#### <a name="task-3-analyze-reports"></a>任务 3：分析报表

在本任务中，你将查看 Mend Bolt 生成报表。

1. 在生成窗格上，单击“Mend Bolt 生成报表”选项卡标头，然后等待报表显示完毕。
1. 在“Mend Bolt 生成报表”选项卡上，验证 Mend Bolt 是否自动检测了软件中的开源组件，包括可传递依赖项及其相应的许可证。
1. 在“Mend Bolt 生成报表”选项卡上，查看安全性仪表板，其中显示了在生成期间发现的漏洞。

    > **注意**：报表显示了所有易受攻击的开源组件的列表，其中包含“漏洞评分”、“漏洞库”和“严重性分布”  。 可以使用所有组件及其元数据和许可引用的链接的详细视图，从而确定开源许可证分布。

1. 在“Mend Bolt 生成报表”选项卡上，向下滚动到“过时的库”部分并查看其内容 。

    > 注意：Mend Bolt 跟踪项目中过时的库，提供库详细信息、较新版本的链接以及修正建议。

## <a name="review"></a>审阅

在本实验室中，你将使用 Mend Bolt 和 Azure DevOps 自动检测代码中易受攻击的开源组件、已过期库和许可证合规性问题。
