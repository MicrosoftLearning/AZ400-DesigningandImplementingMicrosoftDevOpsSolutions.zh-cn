---
lab:
  title: 使用 Azure 项目 Wiki 共享团队知识
  module: 'Module 08: Implement continuous feedback'
---

# 使用 Azure 项目 Wiki 共享团队知识

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

- 设置示例 eShopOnWeb 项目：**** 如果还没有可用于本实验室的示例 EShopOnWeb 项目，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

## 实验室概述

在本实验室中，你将在 Azure DevOps 中创建和配置 Wiki，包括管理 markdown 内容和创建 Mermaid 图。

## 目标

完成本实验室后，你将能够：

- 在 Azure 项目中创建 Wiki。
- 添加和编辑 markdown。
- 创建 Mermaid 图。

## 预计用时：45 分钟

## 说明

### 练习 1：将代码发布为 Wiki

在本练习中，你将逐步了解如何将 Azure DevOps 存储库发布为 Wiki 以及如何管理已发布的 Wiki。

> **注意**：你在 Git 存储库中维护的内容可以发布到 Azure DevOps Wiki。 例如，可以将为支持软件开发工具包、产品文档或自述文件而编写的内容直接发布到 Wiki。 你可以选择在同一个 Azure DevOps 团队项目中发布多个 Wiki。

#### 任务 1：将 Azure DevOps 存储库的一个分支发布为 Wiki

在此任务中，你需要将 Azure DevOps 存储库的一个分支发布为 Wiki。

> **注意**：如果发布的 Wiki 与产品版本相对应，则可以在发布产品的新版本时发布新的分支。

1. 在左侧垂直菜单中单击“Repos”，在“文件”窗格的上部，确保已选中“eShopOnWeb”存储库（从顶部具有 Git 图标的下拉菜单中进行选择）。************ 在分支下拉列表（在包含分支图标的“文件”顶部）中，选择“主分支”，并查看主分支的内容。
1. 在“**文件**”窗格左侧的存储库文件夹和文件层次结构列表中，展开“**src**”文件夹，并浏览到“**Web” ->“wwwroot”->“图像**”子文件夹。 在“图像”子文件夹中，找到 brand.png 项，将鼠标指针悬停在其右端以显示表示“更多”菜单的垂直省略号（三个点），单击“下载”将 brand.png 文件下载到实验室计算机本地的“下载”文件夹。

    > **注意**：在本实验室的下一个练习中，你将使用此图像。

1. 我们会将 Wiki 源文件存储在 Repos 当前文件夹结构中一个单独的文件夹中。 在“Repos”中，选择“文件”。 请注意文件夹结构顶部的 eShopOnWeb 存储库标题。**** **选择省略号（3 个点）**，选择“**新建 / 文件夹**”，并提供“**`Documents`**”作为“新建文件夹”名称的标题。 由于存储库中不允许创建空文件夹，请提供“**`READ.ME`**”作为“新文件”名。
1. 按“创建”按钮确认创建文件夹和文件。
1. READ.ME 文件将在内置视图模式下打开。 由于这是“作为代码”存储的，因此需要通过单击“提交”按钮提交更改。 在“提交”窗口中，按“提交”再次确认。
1. 在左侧的 Azure DevOps 垂直菜单中，单击“概述”，在“概述”部分，选择 Wiki，然后选择“将代码发布为 Wiki”。
1. 在“将代码发布为 Wiki”窗格上，指定以下设置，然后单击“发布” 。

    | 设置 | 值 |
    | ------- | ----- |
    | 存储库 | **eShopOnWeb** |
    | 分支 | 主 |
    | Folder | /Documents |
    | Wiki 名称 | **`eShopOnWeb (Documents)`** |

    > 注意：这将自动打开 Wiki 部分并发布编辑器，你可以在其中提供 Wiki 页面标题以及添加实际内容。 请注意，建议使用 MarkDown 格式，但使用功能区来帮助你使用一些 MarkDown 布局语法。

1. 在 Wiki 页“**标题**”字段中，输入“`Welcome to our Online Retail Store!`”

1. 在 Wiki 页面的正文中，粘贴以下文本：

    ```text
    ##Welcome to Our Online Retail Store!
    At our online retail store, we offer a **wide range of products** to meet the **needs of our customers**. Our selection includes everything from *clothing and accessories to electronics, home decor, and more*.
    
    We pride ourselves on providing a seamless shopping experience for our customers. Our website offers the following benefits:
    1. user-friendly,
    1. and easy to navigate, 
    1. allowing you to find what you're looking for,
    1. quickly and easily. 
    
    We also offer a range of **_payment and shipping options_** to make your shopping experience as convenient as possible.
    
    ### about the team
    Our team is dedicated to providing exceptional customer service. If you have any questions or concerns, our knowledgeable and friendly support team is always available to assist you. We also offer a hassle-free return policy, so if you're not completely satisfied with your purchase, you can easily return it for a refund or exchange.
    
    ### Physical Stores
    |Location|Area|Hours|
    |--|--|--|
    | New Orleans | Home and DIY  |07.30am-09.30pm  |
    | Seattle | Gardening | 10.00am-08.30pm  |
    | New York | Furniture Specialists  | 10.00am-09.00pm |
    
    ## Our Store Qualities
    - We're committed to providing high-quality products
    - Our products are offered at affordable prices 
    - We work with reputable suppliers and manufacturers 
    - We ensure that our products meet our strict standards for quality and durability. 
    - Plus, we regularly offer sales and discounts to help you save even more.
    
    #Summary
    Thank you for choosing our online retail store for your shopping needs. We look forward to serving you!
    ```

1. 此示例文本概述了几个常见的 MarkDown 语法功能，包括标题和副标题 (##)、粗体 (* *)、斜体 (* )、如何创建表等等。

1. 完成后，按右上角的“保存”按钮。

1. 刷新浏览器，或选择任何其他 DevOps 门户选项，然后返回到 Wiki 部分。 请注意，你现在会看到 EShopOnWeb (Documents) Wiki，并看到 Wiki 的主页“欢迎使用我们的在线零售商店”。

#### 任务 2：管理已发布的 Wiki 的内容

在此任务中，你将管理在上一个任务中发布的 Wiki 的内容。

1. 在左侧的垂直菜单中单击“Repos”，确保“文件”窗格上部的下拉菜单显示“EShopOnWeb”存储库和主分支，然后在存储库文件夹层次结构中选择“Documents”文件夹，最后选择“Welcome-to-our-Online-Retail-Store!.md”文件。************************
1. 请注意在此处作为原始文本格式显示的 MarkDown 格式，你可以继续从此处编辑文件内容。

> 注意：由于 Wiki 源文件是作为源代码处理的，传统源代码管理（克隆、拉取请求、审批等）的所有做法现在也适用于 Wiki 页面。

### 练习 2：创建和管理项目 Wiki

在本练习中，你将逐步创建和管理项目 Wiki。

> **注意**：你可以独立于现有存储库创建和管理 Wiki。

#### 任务 1：创建一个包含 Mermaid 图和图像的项目 Wiki

在此任务中，你将创建一个项目 Wiki，并向其中添加一个 Mermaid 图和一个图像。

1. 在实验室计算机上显示 eShopOnweb 项目的 Wiki 窗格的 Azure DevOps 门户中，在“EShopOnWeb (Documents)”Wiki 的内容处于选中状态的情况下，在窗格顶部单击“eShopOnWeb（文件）”下拉列表标题（向下箭头图标），然后在下拉列表中选择“新建项目 Wiki”********************。
1. 在“**页标题**”文本框中，键入“**`Project Design`**”。
1. 将光标放在页面的正文中，单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中单击“标题 1”。 这将自动在行首添加哈希字符 (#)。
1. 紧接在新添加的 **#** 字符后面，键入“**`Authentication and Authorization`**”，然后按 **Enter** 键。
1. 单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中单击“标题 2”。 这将自动在行首添加哈希字符 (##)。
1. 紧接在新添加的 **##** 字符后面，键入“**`Azure DevOps OAuth 2.0 Authorization Flow`**”，然后按 **Enter** 键。
1. 复制并粘贴以下代码以在 Wiki 上插入 mermaid 图。

    ```text
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

    > **注意**：有关 Mermaid 语法的详细信息，请参阅[关于 Mermaid](https://mermaid-js.github.io/mermaid/#/)

1. 在编辑器窗格的右侧的预览窗格中，单击“加载关系图”并查看结果。

    > **注意**：输出应类似于流程图，该流程图说明了如何[使用 OAuth 2.0 授权对 REST API 的访问](https://docs.microsoft.com/azure/devops/integrate/get-started/authentication/oauth)

1. 在编辑器窗格的右上角，单击“保存”按钮旁边指向下方的脱字号，然后在下拉菜单中单击“保存修订消息” 。
1. 在“**保存页**”对话框中，键入“**`Authentication and authorization section with the OAuth 2.0 Mermaid diagram`**”，然后单击“**保存**”。
1. 在“项目设计”编辑器窗格上，将光标置于先前在此任务中添加的 Mermaid 元素的末尾，按 Enter 键添加一个额外的行，单击工具栏上表示标题设置的最左侧图标，然后在下拉列表中，单击“标题 2”  。 这将自动在行首添加双哈希字符 (##)。
1. 紧接在新添加的 **##** 字符后面，键入“**`User Interface`**”，然后按 **Enter** 键。
1. 在“项目设计”编辑器窗格的工具栏上，单击表示“插入文件”操作的回形针图标，在“打开”对话框中，导航到“下载”文件夹，选择在上一个练习中下载的“Brand.png”文件，然后单击“打开”。
1. 返回“项目设计”编辑器窗格，查看预览窗格并验证图像是否正确显示。
1. 在编辑器窗格的右上角，单击“保存”按钮旁边指向下方的脱字号，然后在下拉菜单中单击“保存修订消息” 。
1. 在“**保存页**”对话框中，键入“**`User Interface section with the eShopOnWeb image`**”，然后单击“**保存**”。
1. 返回编辑器窗格的右上角，单击“关闭”。

#### 任务 2：管理项目 Wiki

在此任务中，你将管理新创建的项目 Wiki。

> **注意**：首先，将最新更改恢复到 Wiki 页面。

1. 在实验室计算机上显示 **eShopOnWeb** 项目的 **Wiki 窗格**的 Azure DevOps 门户中，在“**项目设计**”Wiki 的内容处于选中状态的情况下，单击右上角的垂直省略号，然后在下拉菜单中单击“**查看修订**”。
1. 在“修订”窗格上，单击表示最新更改的条目。
1. 在结果窗格上，查看文档的先前版本与当前版本之间的比较，单击“还原”，并在出现确认提示时，再次单击“还原”，然后单击“浏览页面”  。
1. 返回“项目设计”窗格，确认更改已成功还原。

    > **注意**：现在，你将向项目 Wiki 添加另一个页面，并将其设置为 Wiki 主页。

1. 单击“项目设计”窗格左下角的“+ 新建页面” 。
1. 在页编辑器窗格上的“**页标题**”文本框中，键入“**`Project Design Overview`**”，然后依次单击“**保存**”和“**关闭**”。
1. 返回列出“项目设计”项目 Wiki 中各页面的窗格，找到“项目设计概述”条目，用鼠标指针将其选中，并将其拖放到“项目设计”页面条目上方  。
1. 通过在显示的窗口中按“移动”按钮来确认更改。
1. 验证“项目设计概述”条目是否被列为顶层页面，并且带有将其指定为 Wiki 主页的“主页”图标。

## 审阅

在本实验室中，你在 Azure DevOps 中创建和配置了 Wiki，包括管理 markdown 内容和创建 Mermaid 图。
