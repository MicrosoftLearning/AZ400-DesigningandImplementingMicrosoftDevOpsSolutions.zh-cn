---
lab:
  title: 使用 Azure 项目 Wiki 共享团队知识
  module: 'Module 09: Implement continuous feedback'
---

# <a name="sharing-team-knowledge-using-azure-project-wikis"></a>使用 Azure 项目 Wiki 共享团队知识

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

## <a name="lab-overview"></a>实验室概述

在本实验室中，你将在 Azure DevOps 中创建和配置 Wiki，包括管理 markdown 内容和创建 Mermaid 图。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 在 Azure 项目中创建 Wiki。
- 添加和编辑 markdown。
- 创建 Mermaid 图。

## <a name="estimated-timing-45-minutes"></a>预计用时：45 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，包括预先配置的 Tailwind Traders 团队项目，它基于 Azure DevOps 演示生成器模板和在 Microsoft Teams 中创建的团队。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 Tailwind Traders 模板生成新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的“新建项目名称”文本框中，键入“使用 Azure 项目 Wiki 共享团队知识”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表中，选择“Tailwind Traders”模板并单击“选择模板”。 
1. 返回“新建项目”页面，如果提示安装缺失的扩展，请选中“ARM 输出”下方的复选框，并单击“创建项目”。 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 Azure DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

### <a name="exercise-1-publish-code-as-a-wiki"></a>练习 1：将代码发布为 Wiki

在本练习中，你将逐步了解如何将 Azure DevOps 存储库发布为 Wiki 以及如何管理已发布的 Wiki。

> **注意**：你在 Git 存储库中维护的内容可以发布到 Azure DevOps Wiki。 例如，可以将为支持软件开发工具包、产品文档或自述文件而编写的内容直接发布到 Wiki。 你可以选择在同一个 Azure DevOps 团队项目中发布多个 Wiki。

#### <a name="task-1-publish-a-branch-of-an-azure-devops-repo-as-wiki"></a>任务 1：将 Azure DevOps 存储库的一个分支发布为 Wiki

在此任务中，你需要将 Azure DevOps 存储库的一个分支发布为 Wiki。

> **注意**：如果发布的 Wiki 与产品版本相对应，则可以在发布产品的新版本时发布新的分支。

1. 在左侧的垂直菜单中，单击“Repos”，在“文件”窗格的上部，确保已选中“TailwindTraders-Website”存储库（从顶部包含 Git 图标的下拉菜单中进行选择）  。 在分支下拉列表（在包含分支图标的“文件”顶部）中，选择“主分支”，并查看主分支的内容。
1. 在“文件”窗格的左侧，在存储库文件夹和文件层次结构列表中，展开“Documents”文件夹及其“Images”子文件夹，在“Images”子文件夹中，找到“Website.png”条目，将鼠标指针悬停在其右侧，以显示表示“更多”菜单的垂直省略号（三个点），单击“更多”，然后在下拉菜单中单击“下载”，将“Website.png”文件下载到实验室计算机上的本地“Downloads”文件夹中         。

    >**注意**：在本实验室的下一个练习中，你将使用此图像。

1. 在左侧的垂直菜单中，单击“概述”，在“概述”部分，选择 Wiki，然后选择“将代码发布为 Wiki”  。
1. 在“将代码发布为 Wiki”窗格上，指定以下设置，然后单击“发布” 。

    | 设置 | 值 |
    | ------- | ----- |
    | 存储库 | TailwindTraders-Website |
    | 分支 | 主 |
    | Folder | /Documents |
    | Wiki 名称 | Tailwind Traders (Documents) |

    >**注意**：这将自动显示 GitHubAction.md 文件的内容。

1. 查看 GitHubActions 文件的内容，并注意 Wiki 的整体结构，使其与基础存储库的结构匹配。

#### <a name="task-2-manage-content-of-a-published-wiki"></a>任务 2：管理已发布的 Wiki 的内容

在此任务中，你将管理在上一个任务中发布的 Wiki 的内容。

1. 在左侧的垂直菜单中，单击“Repos”，确保“文件”窗格上部的下拉菜单显示“TailwindTraders-Website”存储库和“主”分支，在存储库文件夹层次结构中选择“文档”文件夹，单击右上角的“+ 新建”，然后单击下拉菜单中的“文件”      。
1. 在“新建文件”面板的“新文件名”中，在“/Documents/”前缀后面键入 .order，然后单击“创建”    。
1. 在 .order 窗格的“内容”选项卡上，键入以下内容并单击“提交”  。

    ```text
    GitHubActions
    Images
    ```

1. 在“提交”窗格中，单击“提交”。
1. 在左侧的垂直菜单中，单击“概述”，在“概述”部分中，选择“Wiki”，验证窗格上部是否显示“Tailwind Traders (Documents)”，并查看 Wiki 内容的顺序   。

    >**注意**：Wiki 内容的顺序应与“.order”文件中列出的文件和文件夹的顺序一致。

1. 在左侧的垂直菜单中，单击“Repos”，确保“文件”窗格上部的下拉菜单显示“TailwindTraders-Website”存储库和“主”分支，在文件列表中，选择“文档”下的“GitHubActions.md”，然后在“GitHubActions.md”窗格中单击“编辑”       。
1. 在“GitHubActions.md”窗格上的 `#GitHub Actions` 标头正下方，添加以下引用 Documents 文件夹中的某个图像的 markdown 元素 ：

    ```
    ![Tailwind Traders Website](Images/Website.png)
    ```

1. 在“GitHubActions.md”窗格上，单击“提交”，并在“提交”窗格上单击“提交”   。
1. 在“GitHubActions.md”窗格的“预览”选项卡上，验证是否显示了图像 。
1. 在左侧的垂直菜单中，单击“概述”，在“概述”部分中，选择“Wiki”，验证窗格上部是否显示“Tailwind Traders (Documents)”，并且“GitHubActions”窗格的内容是否包含新引用的图像    。

### <a name="exercise-2-create-and-manage-a-project-wiki"></a>练习 2：创建和管理项目 Wiki

在本练习中，你将逐步创建和管理项目 Wiki。

> **注意**：你可以独立于现有存储库创建和管理 Wiki。

#### <a name="task-1-create-a-project-wiki-including-a-mermaid-diagram-and-an-image"></a>任务 1：创建一个包含 Mermaid 图和图像的项目 Wiki

在此任务中，你将创建一个项目 Wiki，并向其中添加一个 Mermaid 图和一个图像。

1. 在实验室计算机上，在显示“使用 Azure 项目 Wiki 共享团队知识”项目的 Wiki 窗格的 Azure DevOps 门户中，在“Tailwind Traders (Documents)”Wiki 的内容处于选中状态的情况下，在窗格顶部单击“Tailwind Traders (Documents)”下拉列表标题，然后在下拉列表中选择“新建项目 Wiki”    。
1. 在“页面标题”文本框中，键入“项目设计” 。
1. 将光标放在页面的正文中，单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中单击“标题 1”。 这将自动在行首添加哈希字符 (#)。
1. 紧接在新添加的 # 字符后面，键入“身份验证和授权”，然后按 Enter 键  。
1. 单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中单击“标题 2”。 这将自动在行首添加哈希字符 (##)。
1. 紧接在新添加的 ## 字符后面，键入“Azure DevOps OAuth 2.0 授权流”，然后按 Enter 键  。

1. 复制并粘贴以下代码以在 Wiki 上插入 mermaid 图。

    ```
    ::: mermaid
    sequenceDiagram
     participant U as User
     participant A as Your app
     participant D as Azure DevOps
     U->>A: Use your app
     A->>D: Request authorization for user
     D-->>U: Request authorization
     U->>D: Grant authorization
     D-->>A: Send authorization code
     A->>D: Get access token
     D-->>A: Send access token
     A->>D: Call REST API with access token
     D-->>A: Respond to REST API
     A-->>U: Relay REST API response
    :::
    ```

    >**注意**：有关 Mermaid 语法的详细信息，请参阅[关于 Mermaid](https://mermaid-js.github.io/mermaid/#/)

1. 在编辑器窗格的右侧的预览窗格中，单击“加载关系图”并查看结果。

    >**注意**：输出应类似于流程图，该流程图说明了如何[使用 OAuth 2.0 授权对 REST API 的访问](https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops)

1. 在编辑器窗格的右上角，单击“保存”按钮旁边指向下方的脱字号，然后在下拉菜单中单击“保存修订消息” 。
1. 在“保存页面”对话框中，键入“OAuth 2.0 Mermaid 图的“身份验证和授权”部分”，然后单击“保存”  。
1. 在“项目设计”编辑器窗格上，将光标置于先前在此任务中添加的 Mermaid 元素的末尾，按 Enter 键添加一个额外的行，单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中，单击“标题 2”  。 这将自动在行首添加双哈希字符 (##)。
1. 紧接在新添加的 ## 字符后面，键入“用户界面”，然后按 Enter 键  。
1. 在“项目设计”编辑器窗格的工具栏上，单击表示“插入文件”操作的回形针图标，在“打开”对话框中，导航到“Downloads”文件夹，选择在上一个练习中下载的“Website.png”文件，然后单击“打开”     。
1. 返回“项目设计”编辑器窗格，查看预览窗格并验证图像是否正确显示。
1. 在编辑器窗格的右上角，单击“保存”按钮旁边指向下方的脱字号，然后在下拉菜单中单击“保存修订消息” 。
1. 在“保存页面”对话框中，键入“Tailwind Traders 图像的“用户界面”部分”，然后单击“保存”  。
1. 返回编辑器窗格的右上角，单击“关闭”。

#### <a name="task-2-manage-a-project-wiki"></a>任务 2：管理项目 Wiki

在此任务中，你将管理新创建的项目 Wiki。

>**注意**：首先，将最新更改恢复到 Wiki 页面。

1. 在实验室计算机上，在显示“使用 Azure 项目 Wiki 共享团队知识”项目的 Wiki 窗格的 Azure DevOps 门户中，在“项目设计”Wiki 的内容处于选中状态的情况下，单击右上角的垂直省略号，然后在下拉菜单中单击“查看修订”   。
1. 在“修订”窗格上，单击表示最新更改的条目。
1. 在结果窗格上，查看文档的先前版本与当前版本之间的比较，单击“还原”，并在出现确认提示时，再次单击“还原”，然后单击“浏览页面”  。
1. 返回“项目设计”窗格，确认更改已成功还原。

    >**注意**：现在，你将向项目 Wiki 添加另一个页面，并将其设置为 Wiki 主页。

1. 单击“项目设计”窗格左下角的“+ 新建页面” 。
1. 在页面编辑器窗格上的“页面标题”文本框中，键入“项目设计概述”，然后依次单击“保存”和“关闭”   。
1. 返回列出“项目设计”项目 Wiki 中各页面的窗格，找到“项目设计概述”条目，用鼠标指针将其选中，并将其拖放到“项目设计”页面条目上方  。
1. 验证“项目设计概述”条目是否被列为顶层页面，并且带有将其指定为 Wiki 主页的“主页”图标。

## <a name="review"></a>审阅

在本实验室中，你在 Azure DevOps 中创建和配置了 Wiki，包括管理 markdown 内容和创建 Mermaid 图。
