---
lab:
  title: 实验室 6：将外部源代码管理与 Azure Pipelines 集成
  module: 'Module 03: Implement CI with Azure Pipelines and GitHub Actions'
ms.openlocfilehash: af9b06398bffd5ede41513f4423ce875a018d214
ms.sourcegitcommit: 73179152f51e48ada9641c4a6a33ea941606c469
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2022
ms.locfileid: "146774604"
---
# <a name="lab-06-integrating-external-source-control-with-azure-pipelines"></a>实验室 6：将外部源代码管理与 Azure Pipelines 集成

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 如果尚无可用于本实验室的 GitHub 帐户，请按照[注册新 GitHub 帐户](https://docs.github.com/get-started/signing-up-for-github/signing-up-for-a-new-github-account)中的说明操作。

## <a name="lab-overview"></a>实验室概述

通过推出 Azure DevOps，Microsoft 为开发人员提供了一种新的持续集成/持续交付 (CI/CD) 服务（称为 Azure Pipelines），可用于进行持续生成、测试以及部署到任何平台或云。 它具有适用于 Linux、macOS 和 Windows 的云托管代理；支持本地容器的强大工作流；并且可灵活部署到 Kubernetes、VM 和无服务器环境。

Azure Pipelines 为每个 GitHub 开源项目免费提供无限的 CI/CD 分钟数和 10 个并行作业。 所有开源项目都在付费客户使用的相同基础结构上运行。 这意味着你将获得同样快速的性能和高质量的服务。 许多顶级开源项目已经在使用 Azure Pipelines 实现 CI/CD，例如 Atom、CPython、Pipenv、Tox、Visual Studio Code 和 TypeScript，并且此类项目每天都在增加。

在本实验室中，你会发现使用 GitHub 项目设置 Azure Pipelines 是多么容易，以及如何可以立竿见影地看到好处。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 从 GitHub 市场安装 Azure Pipelines。
- 将 GitHub 项目与 Azure 管道集成。
- 通过管道跟踪拉取请求。

## <a name="estimated-timing-60-minutes"></a>预计用时：60 分钟

## <a name="instructions"></a>说明

### <a name="exercise-1-getting-started-with-azure-pipelines"></a>练习 1：Azure Pipelines 入门

在本练习中，你将使用来自市场的新 Azure Pipelines 集成将 GitHub 项目与 Azure DevOps 集成。

#### <a name="task-1-forking-a-github-repo-and-installing-azure-pipelines"></a>任务 1：创建 GitHub 存储库分支并安装 Azure Pipelines

在此任务中，你将创建一个 GitHub 存储库分支，并在 GitHub 帐户中安装 Azure Pipelines。

1. 在实验室计算机上，启动 Web 浏览器，导航到 [GitHub actionsdemos/计算器站点](https://github.com/actionsdemos/calculator)，如果尚未登录 GitHub，请立即登录。

    > **注意**：这是我们将在本实验中创建分支和使用的基线项目。

1. 在“actionsdemos/计算器站点”页面上，单击“创建分支”，以在自己的 GitHub 帐户中创建存储库的分支 。 如果出现提示，请选择要在其中创建存储库分支的帐户。
1. 在显示创建分支后的存储库的页面上，单击顶部菜单中的“市场”。
    > **注意**：GitHub 市场提供了来自 Microsoft 和第三方的各种工具，可帮助扩展项目工作流。
2. 在“搜索应用和操作”中，键入“Azure Pipelines”，按 Enter 键，然后在结果列表中单击“Azure Pipelines”   。
3. 在“Azure Pipelines”页面上，单击“阅读全文”，然后通读 Azure Pipelines 的优势 。

    > **注意**：所有人都可以将 Azure Pipelines 产品/服务免费用于公共存储库，如果使用专用存储库，则可以免费用于单个生成队列。

4. 在“Azure Pipelines”页面上，单击“免费安装” 。 如果拥有多个 GitHub 帐户，请从“切换计费帐户”下拉列表中选择要在其中创建计算器分支的帐户 。
5. 在“查看订单”页面上，单击“完成订单并开始安装” 。
6. 在“安装 Azure Pipelines”页面上，使用默认选项“所有存储库”，然后单击“安装”  。

    > **注意**：可以选择指定要包含的存储库，但出于本实验室的目的，只需包括所有存储库即可。 请注意，Azure DevOps 需要列出的权限集才能实现其服务。

7.  如果出现提示，请使用 GitHub 密码进行身份验证以继续操作。
8.  出现提示时，在“设置 Azure Pipelines 项目”页上，首先单击“切换目录”并确保选中“默认目录” 。 然后，在“选择你的 Azure DevOps 组织”下拉列表中，选择你的 Azure DevOps 帐户，并单击“创建新项目” 。
9.  出现提示时，在“设置 Azure Pipelines 项目”页面上的“项目名称”文本框中，键入“将外部源代码管理与 Azure Pipelines 集成”，将“项目可见性”设置为“私有”，然后单击“继续”     。
10. 在“Microsoft 希望 Azure Pipelines 获得的权限”页面上，单击“授权 Azure Pipelines” 。

#### <a name="task-2-configuring-your-azure-pipelines-project"></a>任务 2：配置 Azure Pipelines 项目

在此任务中，你将基于在上一个任务中创建的 GitHub 存储库的分支配置 Azure Pipelines 项目。

> **注意**：你现在位于 Azure DevOps 站点上，并且需要设置 Azure Pipelines 项目。

1. 在 Azure DevOps 门户中，单击左侧导航面板中的“管道”。

1. 单击右上方的“创建管道”按钮。

1. 在 Azure DevOps 门户中“管道”视图的“选择存储库”窗格上，选择在上一个任务中创建的 GitHub 计算器存储库的分支 。

    > **注意**：Azure Pipelines 将分析你的项目，以尝试确定任何现有模板是否适合。 在这种情况下，推荐使用针对 Node.js 的模板，它完全符合我们的需求。 虽然推荐的模板是适用于本实验室的最佳模板，但还是建议了一些替代模板。
1. 在“配置管道”上，选择“Node.js” 。

    > **注意**：生成管道定义为 YAML，这是一种非常适合用于定义此类进程的标记语法，因为使用它你可以像存储库中的任何其他文件一样管理管道的配置。 这是一个非常简单的模板，用于标识要从中拉取 VM 以进行生成的池、安装 Node.js 以进行生成的进程以及实际的生成本身。

1. 在“查看管道 YAML”上，单击“保存并运行”以保存管道并为新生成排队 。
1. 在“保存并运行”窗格上，接受默认设置，然后单击“保存并运行” 。

    > **注意**：在本实验室中，你可以将此新文件直接提交到主分支。

    > **注意**：需要一些时间才能完成管道。 在此期间，它将配置生成代理，从 GitHub 拉取源，然后根据管道定义生成它。

1. 在“生成作业”窗格的“摘要”选项卡上，确认生成已成功完成。

#### <a name="task-3-modifying-a-yaml-build-pipeline-definition"></a>任务 3：修改 YAML 生成管道定义

在此任务中，你将在 GitHub 存储库分支中修改 YAML 生成定义，并跟踪由修改触发的生成进程。

> **注意**：虽然使用默认管道是一个不错的开始，但它并不能完成我们希望自动完成的所有工作。 例如，如果它也运行我们的测试来确认更改不会产生 bug，那就太好了。 让我们回到 GitHub，我们可以在 GitHub 中手动编辑 YAML。

1. 在“生成作业”窗格的“摘要”选项卡上，右键单击“存储库和版本”标签旁代表 GitHub 项目存储库的条目，该存储库托管了本实验室前面部分创建的分支，然后选择“在新选项卡中打开链接”  。此操作将打开一个新的浏览器标签页，显示包含分支内容的 GitHub 页面。

    > **注意**：由于本实验室将涉及在 GitHub 和 Azure DevOps 之间来回切换，因此使它们都打开一个浏览器标签页会更容易。

1. 在显示分支内容的 GitHub 页面上，找到代表文件 azure-pipelines.yml 的条目并单击它。 此操作将自动打开该文件并显示其内容。
1. 在“master/calculator/azure-pipelines.yml”页面上，在显示文件内容的窗格的右上角，单击铅笔形状的“编辑此文件”图标 。

    > **注意**：我们的项目已包含使用 Mocha 编写的测试，因此我们只需在管道中执行它们即可。

1. 要添加测试运行，请在 `npm run build` 命令正下方添加 `npm test` 命令，并使用相同的缩进。 此外，将 `displayName` 条目更新为 `'npm install, build, and test'` 以清楚地指示生成的每个任务正在执行的操作：

    ```
      npm test
    displayName: 'npm install, build, and test'
    ```

1. 滚动到该页面底部，将默认提交消息替换为“添加 npm 测试”，然后单击“提交更改” 。

    > **注意**：同样，考虑到这是实验室环境，将此更改直接提交到主分支也是可以接受的。

1. 切换回显示“Azure DevOps”门户的浏览器标签页，并使用痕迹导航导航到“管道”视图的“管道”窗格  。
1. 验证由更新触发的新生成是否已显示在“最近运行的管道”列表中的“最近”选项卡上 。 单击管道对应的条目，在“运行”选项卡上，选择最新的运行，然后在“作业”部分中，单击“作业”条目  。
1. 在显示作业详细信息的窗格上，单击作业的各个任务，然后逐步完成。

#### <a name="task-4-proposing-a-change-via-github-pull-request"></a>任务 4：通过 GitHub 拉取请求提出更改

在此任务中，你将提出一个无效的更改并查看由拉取请求触发的生成结果。

> **注意**：此管道设置的一大好处是，我们现在有了质量门，每次有人提交更改时，它都会自动运行。 这样可以更轻松地管理可能具有任何数量不同质量级别的贡献的项目。

1. 切换回显示 GitHub 页面的浏览器标签页，该页面显示 azure-pipelines.yml 文件的内容，导航回列出创建分支后的存储库内容的页面，然后单击“转到文件” 。
1. 在“calculator/”提示下，键入“arithmeticController.js”，然后在结果列表中单击“api/controllers/arithmeticController.js”  。 这将自动将浏览器会话重定向到“master/calculator/api/controllers/arithmeticController.js”页面，其中显示该文件的内容。

    > **注意**：该控制器包含该应用的核心功能。 但是，“添加”操作的代码尚不完全清楚。 将自己当作具有良好意图但缺乏 JavaScript 经验的人。 他们可能会认为这是一个机会，可以通过清理代码来帮助解决问题，从而使其变得更好。

1. 在“master/calculator/api/controllers/arithmeticController.js”页面上，在显示文件内容的窗格的右上角，单击铅笔形状的“编辑此文件”图标 。
1. 将 `'add':      function(a,b) { return +a + +b },` 行更改为 `'add':      function(a,b) { return a + b },`。

    > **注意**：这是一个错误的更改，将会产生无效的结果。

1. 滚动到页面底部，将默认提交消息替换为“修改添加函数”，选择“新建分支”，将其名称设置为“additional-cleanup”，然后单击“提出文件更改”   。
1. 在“打开拉取请求”页面上，单击“打开拉取请求”以启动将未经测试的更改纳入生产代码的过程 。

    > **注意**：Azure DevOps 将检测更改并启动生成管道。 这将导致检查失败，从而触发对 GitHub UI 的更新。

    > **注意**：回到最初的“项目所有者”的心态。

1. 在“修改添加函数 #1”拉取请求页面上的“所有检查均已失败”部分中，单击“详细信息”以了解详细信息 。
1. 查看“注释”部分，并单击其正下方的链接“查看有关 Azure Pipelines 的更多详细信息” 。 此操作将打开一个新的浏览器标签页，其中显示 Azure DevOps 门户中作业运行失败的信息。
1. 在 Azure 门户的失败作业窗格中，单击“作业”条目以显示其详细信息。
1. 在作业任务列表中，单击“npm 安装、生成和测试”任务以查看其输出。
1. 找到列出失败测试的部分。

    > **注意**：测试失败的原因可能无法马上得知，但是通过我们积累的在管道方面的所有历史记录，可以很容易地确定新拉取请求中的某些内容就是原因。 下一步将弄清楚为什么“21 + 21”得到“2121”而不是预期的“42”。

1. 关闭显示 Azure DevOps 门户中作业运行失败的选项卡。

#### <a name="task-5-using-the-broken-pull-request-to-improve-the-project"></a>任务 5：使用中断的拉取请求来改进项目

在此任务中，你将纠正在上一个任务中创建的拉取请求中引入的无效更改。

> **注意**：回到最初的“项目所有者”的心态。

1. 返回显示“修改添加函数 #1”GitHub 页面的浏览器标签页，然后返回列出创建分支后的存储库内容的主页面。
1. 在包含存储库文件列表的窗格顶部，单击“拉取请求”，然后单击代表最新拉取请求的条目。
1. 在“修改添加函数 #1”GitHub 页面上，单击“文件已更改”选项卡并查看其内容 。

    > **注意**：做出这些更改的人似乎没有意识到，必须在每个变量之前加上加号才能强制将这些变量转换为其数字表示形式。 删除它们后，JavaScript 会将中间的加号解释为字符串串联运算符，这就说明了为什么失败的测试中会出现 21 + 21 = 2121。

1. 在“修改添加函数 #1”GitHub 页面上，单击“查看更改”按钮正下方的省略号，然后在下拉菜单中单击“编辑文件”  。
1. 通过在“a”和“b”变量前面添加加号来还原原来的更改，得到 `'add':      function(a,b) { return +a + +b },` 。 此外，对前一行添加注释，说明 `// Using + operator to type cast variables as integers in order to prevent string concatenation`。
1. 滚动到页面底部，将默认提交消息替换为“修复添加函数”，确保选择了“直接提交到 additional-cleanup 分支”选项，然后单击“提交更改”  。
1. 在“修改添加函数 #1”GitHub 页面上，选择“对话”选项卡 。

    > **注意**：Azure DevOps 将再次检测更改并启动生成管道。 等待所有检查通过。

1. 通过所有检查后，单击“合并拉取请求”，然后单击“确认合并” 。

#### <a name="task-6-adding-a-build-status-badge"></a>任务 6：添加生成状态徽章

在此任务中，你将向 GitHub 存储库添加生成状态徽章。

> **注意**：质量项目的一个重要徽章是其生成状态徽章。 如果有人发现带有徽章的项目，并且该徽章表明该项目当前处于成功生成状态，这表明该项目得到了有效维护。

1. 切换回显示“Azure DevOps”门户的浏览器标签页，并使用痕迹导航导航到“管道”视图的“管道”窗格的“最近”选项卡   。
1. 在“最近运行的管道”列表中的“最近”选项卡上，单击本练习中使用的管道对应的条目 。
1. 在“管道”窗格上，单击右上角的省略号，然后在下拉列表中选择“状态徽章”。

    > **注意**：“状态徽章”UI 提供了一种快速且简便的方法来引用生成状态。 通常，你希望在自己的仪表板中使用提供的 URL，或者可以使用 Markdown 片段将状态徽章添加到 Wiki 页面等位置。

1. 在“状态徽章”窗格上，单击“示例 Markdown”的“复制到剪贴板”按钮  。
1. 切换回显示 GitHub 页面的浏览器标签页，该页面中显示创建分支后的存储库的内容，如果需要，单击“<> 代码”选项卡。
1. 在 repo 文件列表中，单击 README<nolink>.md，然后在 master/calculator/README.md 页面上，在显示文件内容的窗格的右上角，单击铅笔形状的“编辑此文件”图标  。
1. 在第 6 行上方再加一行，并将剪贴板的内容粘贴到其中。
1. 滚动到该页面底部，将默认提交消息替换为“添加 Azure Pipelines 状态徽章”，然后单击“提交更改” 。

    > **注意**：现在，项目的标题页面上有了动态的生成状态徽章，可让每个人知道你正在有效地管理项目。

## <a name="review"></a>审阅

在本实验室中，你使用来自市场的新 Azure Pipelines 集成将 GitHub 项目与 Azure DevOps 进行了集成。
