---
lab:
  title: 实验室 03：在 Azure Repos 中使用 Git 进行版本控制
  module: 'Module 02: Development for enterprise DevOps'
ms.openlocfilehash: d0d9805311da5ccc640fa4d4d8821b9e5a682cec
ms.sourcegitcommit: d78aebd7b14277a53f152e26cea68a30b0e90d73
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/13/2022
ms.locfileid: "146276067"
---
# <a name="lab-03-version-controlling-with-git-in-azure-repos"></a>实验室 03：在 Azure Repos 中使用 Git 进行版本控制

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- [适用于 Windows 的 Git 下载页面](https://gitforwindows.org/)。 此应用程序将作为本实验室先决条件安装。

- [Visual Studio Code](https://code.visualstudio.com/)。 此应用程序将作为本实验室先决条件安装。

## <a name="lab-overview"></a>实验室概述

Azure DevOps 支持两种类型的版本控制：Git 和 Team Foundation 版本控制 (TFVC)。 下面简要概述了两个版本的控制系统：

- **Team Foundation 版本控制 (TFVC)** ：TFVC 是一个集中式版本控制系统。 通常，团队成员的开发计算机上的每个文件只有一个版本。 历史数据仅在服务器上维护。 分支是基于路径的，并且在服务器上创建。

- **Git**：Git 是一种分布式版本控制系统。 Git 存储库可存在于本地（开发人员的计算机上）。 每个开发人员在其开发计算机上拥有源存储库的副本。 开发人员可在其开发计算机上提交每个变更集并执行版本控制操作（如历史记录和比较），无需网络连接。

Git 是新项目的默认版本控制提供程序。 除非需要 TFVC 中的集中式版本控制功能，否则应在项目中使用 Git 进行版本控制。

在本实验室中，你将了解如何在 Azure DevOps 中使用分支和存储库。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 在 Azure Repos 中使用分支。
- 在 Azure Repos 中使用存储库。

## <a name="estimated-timing-30-minutes"></a>预计用时：30 分钟

## <a name="instructions"></a>说明

### <a name="note"></a>注意

如果你已完成“实验室 02 **：在 Azure Repos 中使用 Git 进行版本控制”，并且已经完成了“练习 0** **：配置实验室先决条件”和“练习 1** **：克隆现有的存储库”中的步骤，可以跳到“练习 2** **：从 Azure DevOps 管理分支”** 。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板和 Visual Studio Code 配置预配置的 Parts Unlimited 团队项目。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 Parts Unlimited 模板生成一个新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“创建新项目”页面上的“新建项目名称”文本框中，键入“在 Azure Repos 中使用 Git 进行版本控制”，在“选择组织”下拉列表中，选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表中，找到“PartsUnlimited”模板，然后单击“选择模板” 。
1. 返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-install-and-configure-git-and-visual-studio-code"></a>任务 2：安装并配置 Git 和 Visual Studio Code

在此任务中，你将安装并配置 Git 和 Visual Studio Code，包括配置 Git 凭据帮助器，以安全地存储用于与 Azure DevOps 进行通信的 Git 凭据。 如果你已实现这些先决条件，则可直接继续执行下一个任务。

1. 如果尚未安装 Git 2.29.2 或更高版本，请启动 Web 浏览器，导航到[适用于 Windows 的 Git 下载页面](https://gitforwindows.org/)以进行下载和安装。
1. 如果尚未安装 Visual Studio Code，请从 Web 浏览器窗口导航到 [Visual Studio Code 下载页面](https://code.visualstudio.com/)以进行下载和安装。
1. 如果尚未安装 Visual Studio C# 扩展，请从 Web 浏览器窗口导航到 [C# 扩展安装页面](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)并安装该扩展。
1. 在实验室计算机上，打开“Visual Studio Code”。
1. 在 Visual Studio Code 界面中，从主菜单中选择“终端”\|“新建终端”，以打开“终端”窗格 。
1. 通过检查“终端”窗格右上角的下拉列表是否显示“1: powershell”，确保当前终端正在运行 PowerShell  

    > **注意**：若要将当前终端 shell 更改为 PowerShell，请单击“终端”窗格右上角的下拉列表，然后单击“选择默认 Shell”  。 在“Visual Studio Code”窗口的顶部，选择首选终端 shell“Windows PowerShell”，然后单击下拉列表右侧的加号，以使用所选的默认 shell 打开新的终端。

1. 在“终端”窗格中，运行以下命令来配置凭据帮助器。

    ```git
    git config --global credential.helper wincred
    ```

1. 在“终端”窗格中运行以下命令，为 Git 提交配置用户名和电子邮件（将括号中的占位符替换为首选用户名和电子邮件）：

    ```git
    git config --global user.name "<John Doe>"
    git config --global user.email <johndoe@example.com>
    ```

### <a name="exercise-1-clone-an-existing-repository"></a>练习 1：克隆现有存储库

在本练习中，你将使用 Visual Studio Code 来克隆在上一练习中预配的 Git 存储库。

#### <a name="task-1-clone-an-existing-repository"></a>任务 1：克隆现有存储库

在此任务中，你将逐步完成使用 Visual Studio Code 克隆 Git 存储库的过程。

1. 如有需要，启动 Web 浏览器，导航到 Azure DevOps 组织，然后打开上一练习中生成的“在 Azure Repos 中使用 Git 进行版本控制”项目。

    > **注意**：或者，你可以通过导航到 [https://dev.azure.com/`<your-Azure-DevOps-account-name>` /Version%20Controlling%20with%20Git%20in%20Azure%20Repos](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Version%20Controlling%20with%20Git%20in%20Azure%20Repos) URL 直接访问项目页面，其中 `<your-Azure-DevOps-account-name>` 占位符代表你的帐户名称。

1. 在 DevOps 界面的垂直导航窗格中，选择“存储库”图标。
1. 在“PartsUnlimited”窗格的右上角，单击“克隆” 。

    > **注意**：获取 Git 存储库的本地副本的过程称为“克隆”。 每个主流开发工具都支持此功能，并且将能够连接到 Azure Repos 来获取可使用的最新资源。 导航到“Repos”中心。

1. 在“克隆存储库”面板上，在已选择 HTTPS 命令行选项的情况下，单击存储库克隆 URL 旁边的“复制到剪贴板”按钮  。

    > **注意**：可以将此 URL 与任何与 Git 兼容的工具一起使用，以获取代码库的副本。

1. 关闭“克隆存储库”面板。
1. 切换到在实验室计算机上运行的 Visual Studio Code。
1. 单击“查看”菜单标题，然后在下拉菜单中单击“命令面板” 。

    > **注意**：命令面板提供了一种轻松便捷的方式来访问各种任务，包括作为第三方扩展实现的任务。 可使用键盘快捷方式 Ctrl+Shift+P 来打开它。

1. 在命令面板提示符下，运行“Git:**Clone”命令**。

    > **注意**：要查看所有相关命令，可以从键入 Git 开始。

1. 在“提供存储库 URL 或选择存储库源”文本框中，粘贴之前在此任务中复制的存储库克隆 URL，并按 Enter 键 。
1. 在“选择文件夹”对话框中，导航到 C: 驱动器，创建名为 Git 的新文件夹并选择它，然后单击“选择存储库位置”  。
1. 根据提示登录到 Azure DevOps 帐户。
1. 克隆进程完成后，如果出现提示，在 Visual Studio Code 中单击“打开”，以打开克隆的存储库。

    > **注意**：你可能会收到的有关项目加载问题的警告，可以忽略它们。 解决方案可能不适合生成，但我们将专注于使用 Git，因此不需要生成项目。

### <a name="exercise-2-manage-branches-from-azure-devops"></a>练习 2：从 Azure DevOps 管理分支

在本练习中，你将借助 Azure DevOps 使用分支。 除了 Visual Studio Code 中提供的功能外，还可直接从 Azure DevOps 门户管理存储库分支。

#### <a name="task-1-create-a-new-branch"></a>任务 1：创建新分支

在此任务中，你将使用 Azure DevOps 创建分支，并使用 Visual Studio Code 提取它。

1. 切换到显示 Azure DevOps 组织以及在上一个练习中生成的“在 Azure Repos 中使用 Git 进行版本控制”项目的 Web 浏览器。

    > **注意**：或者，你可以通过导航到 [<https://dev.azure.com/>`<your-Azure-DevOps-account-name>` /Version%20Controlling%20with%20Git%20in%20Azure%20Repos) URL 直接访问项目页面，其中 `<your-Azure-DevOps-account-name>` 占位符代表你的帐户名称。

1. 在 Web 浏览器窗口中，导航到项目的“存储库”窗格，并选择“分支” 。
1. 在“分支”窗格上，单击“新建分支” 。
1. 在“创建分支”面板的“名称”文本框中，键入“版本”，确保“主分支”显示在“基于”下拉列表中，在“要链接的工作项”下拉列表中，选择一个或多个可用的工作项，然后单击“创建”      。
1. 切换到“Visual Studio Code”窗口。
1. 按 Ctrl+Shift+P 打开命令面板 。
1. 在命令面板提示符下，开始键入“Git **:Fetch”，并选择出现的“Git** **:Fetch”** 。 此命令将更新本地快照中的原始分支。
1. 在 Visual Studio Code 窗口的左下角，再次单击“master”条目。
1. 在分支列表中选择“原始/版本”。 这会创建一个名为“版本”的新本地分支，并将其签出。

#### <a name="task-2-delete-and-restore-a-branch"></a>任务 2：删除和还原分支

在此任务中，你将使用 Azure DevOps 门户删除和还原在上一个任务中创建的分支。

1. 切换到在 Azure DevOps 门户中显示“分支”窗格的“我的”选项卡的 Web 浏览器 。
1. 在“分支”窗格的“我的”选项卡上，将鼠标指针悬停在“版本”分支条目上方，以显示右侧的省略号  。
1. 单击省略号，在弹出菜单中选择“删除分支”，并在出现确认提示时单击“删除” 。
1. 在“分支”窗格的“我的”选项卡上，选择“全部”选项卡  。
1. 在“分支”窗格的“全部”选项卡上，在“搜索分支名称”文本框中，键入“版本”   。
1. 查看“已删除的分支”部分，此部分包含表示新创建的分支的条目。
1. 在“已删除的分支”部分，将鼠标指针悬停在“版本”分支条目上方，以显示右侧的省略号 。
1. 单击弹出菜单中的省略号，选择“还原分支”。

    > **注意**：只要你知道某个已删除分支的确切名称，就可以使用此功能还原该分支。

#### <a name="task-3-lock-and-unlock-a-branch"></a>任务 3：锁定和解锁分支

在此任务中，你将使用 Azure DevOps 门户锁定和解锁主分支。

锁定是防止出现可能与重要合并相冲突的新更改或防止使分支处于只读状态的理想方法。 或者，如果你只想确保分支中的更改在合并之前经过了检查，可以使用分支策略并拉取请求，而不是锁定。

锁定不会阻止克隆存储库或将在分支中进行的更新提取到你的本地存储库。 如果锁定分支，请与团队分享锁定原因，并确保他们知道解锁分支后应使用分支执行什么操作。

1. 切换到在 Azure DevOps 门户中显示“分支”窗格的“我的”选项卡的 Web 浏览器 。
1. 在“分支”窗格的“我的”选项卡上，将鼠标指针悬停在主分支条目上方，以显示右侧的省略号  。
1. 单击弹出菜单中的省略号，选择“锁定”。
1. 在“分支”窗格的“我的”选项卡上，将鼠标指针悬停在主分支条目上方，以显示右侧的省略号  。
1. 单击弹出菜单中的省略号，选择“解锁”。

#### <a name="task-4-tag-a-release"></a>任务 4：标记版本

在此任务中，你将使用 Azure DevOps 门户在 Azure DevOps Repos 中标记版本。

产品团队已决定，当前版本的网站应发布为 v1.1.0-beta。

1. 在 Azure DevOps 门户的垂直导航窗格中，在“Repos”部分中，选择“标记” 。
1. 在“标记”窗格中，单击“新建标记” 。
1. 在“创建标记”面板中，在“名称”文本框中键入“v1.1.0-beta”，在“基于”下拉列表中，保留“主分支”条目处于选中状态，在“说明”文本框中，键入“Beta 版本 v1.1.0”，然后单击“创建”       。

    > **注意**：你现在已标记此版本中的项目。 你可以基于各种理由对提交进行标记，并且通过 Azure DevOps，可灵活地编辑和删除它们以及管理其权限。

### <a name="exercise-3-manage-repositories"></a>练习 3：管理存储库

在本练习中，你将使用 Azure DevOps 门户在 Azure DevOps Repos 中创建和删除 Git 存储库。

可在团队项目中创建 Git 存储库，以管理项目的源代码。 每个 Git 存储库都有其自己的一组权限和分支，以使它自身与项目中的其他工作隔离。

#### <a name="task-1-create-a-new-repo-from-azure-devops"></a>任务 1：从 Azure DevOps 创建新存储库

在此任务中，你将使用 Azure DevOps 门户在 Azure DevOps Repos 中创建 Git 存储库。

1. 在显示 Azure DevOps 门户的 Web 浏览器中，在垂直导航窗格中，单击左上角的加号（位于项目名称右侧），然后在级联菜单中单击“新建存储库”。
1. 在“创建存储库”窗格的“存储库类型”中，保留默认的 Git 条目，在“存储库名称”文本框中，键入“新建存储库”，将其他设置保留为默认值，然后单击“创建”     。

    > **注意**：可选择创建名为“README.md”的文件。 这将是默认的 markdown 文件，当有人使用 Web 浏览器导航到存储库根目录时，会显示此文件。 此外，可以使用 .gitignore 文件预配置存储库。 此文件根据命名模式和/或路径指定要在源代码管理中忽略的文件。 有许多可用模板，其中包括根据要创建的项目类型而忽略的常见模式和路径。

    > **注意**：现在，存储库可供使用。 现在可使用 Visual Studio Code 或任何其他与 git 兼容的工具来克隆它。

#### <a name="task-2-delete-and-rename-git-repos"></a>任务 2：删除和重命名 Git 存储库

在此任务中，你将使用 Azure DevOps 门户在 Azure DevOps Repos 中删除 Git 存储库。

有时，你需要重命名或删除存储库，这非常简单。

1. 在显示 Azure DevOps 门户的 Web 浏览器中，在垂直导航窗格底部，单击“项目设置”。
1. 在“项目设置”垂直导航窗格中，向下滚动到“Repos”部分，并单击“存储库”  。
1. 在“所有存储库”窗格的“存储库”选项卡上，将鼠标指针悬停在“新建存储库”分支条目上方，以显示右侧的省略号  。
1. 单击省略号，在弹出菜单中选择“删除分支”，并在出现确认提示时单击“删除”   。

## <a name="review"></a>审阅

在本实验室中，你使用了 Azure DevOps 门户来管理分支和存储库。
