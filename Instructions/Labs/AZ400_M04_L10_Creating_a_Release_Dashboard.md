---
lab:
  title: 创建发布仪表板
  module: 'Module 04: Design and implement a release strategy'
---

# 创建发布仪表板

# 学生实验室手册

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证是否有 Microsoft 帐户或具有 Azure 订阅中所有者角色的 Azure AD 帐户。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)。

## 实验室概述

在本实验室中，你将逐步创建一个发布仪表板，并使用 REST API 检索 Azure DevOps 发布数据，这些数据可提供给自定义应用程序或仪表板。

本实验室使用 Azure DevOps 初学者资源，该资源自动创建一个 Azure DevOps 项目，用于生成应用程序并将其部署到 Azure 中。

## 目标

完成本实验室后，你将能够：

- 创建发布仪表板。
- 使用 REST API 查询发布信息。

## 预计用时：45 分钟

## 说明

### 练习 1：创建发布仪表板

在本练习中，你将在 Azure DevOps 组织中创建发布仪表板。

#### 任务 1：创建 Azure DevOps Starter 资源

在此任务中，你将在 Azure 订阅中创建 Azure DevOps Starter 资源。 这将在 Azure DevOps 组织中自动创建相应的项目。

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后使用具有 Azure 订阅中所有者或参与者角色的用户帐户登录，你将在本实验室中使用该帐户****。
1. 在 Azure 门户中，搜索并选择“DevOps Starter”**** 资源类型，然后在“DevOps Starter”**** 边栏选项卡上单击“+ 创建”****。
1. 在“DevOps Starter”**** 边栏选项卡的“使用新应用程序重新开始”**** 窗格上，选择“.NET”**** 磁贴，然后在“使用 GitHub 设置 DevOps Starter”**** 旁的上方，更改设置，单击“此处”****，依次选择“Azure DevOps”****、“完成”**** 和“下一步: 框架 >”****。
1. 在“DevOps Starter”**** 边栏选项卡的“选择应用程序框架”**** 窗格上，选择“ASP<nolink>.NET Core”**** 磁贴，将“添加数据库”**** 滑块移到“打开”**** 位置，然后单击“下一步: 服务 >”****。
1. 在“DevOps Starter”**** 边栏选项卡的“选择部署应用程序的 Azure 服务”**** 窗格上，确保已选中“Windows Web 应用”**** 磁贴，然后单击“下一步: 创建 >”****。
1. 在“DevOps Starter”**** 边栏选项卡上的“即将完成”**** 窗格上，指定以下设置：

    | 设置 | 值 |
    | ------- | ----- |
    | 项目名称 | **创建发布仪表板** |
    | Azure DevOps 组织 | 将在此实验室中使用的 Azure DevOps 组织的名称 |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | Web 应用名称 | 长度介于 2 至 60 个字符的任何全局唯一字符串（由字母、数字和连字符组成，以字母或数字开头和结尾） |
    | 位置 | 要在其中部署 Azure Web 应用和 Azure SQL 数据库的 Azure 区域的名称 |

1. 在“DevOps Starter”**** 边栏选项卡上的“即将完成”**** 窗格上，单击“其他设置”****。
1. 在“其他设置”**** 窗格上，指定以下设置，然后单击“确定”****。

    | 设置 | 值 |
    | ------- | ----- |
    | 资源组 | **az400m10l02-rg** |
    | 定价层 | F1 免费**** |
    | Application Insights 位置 | 为 Azure Web 应用的位置选择的同一 Azure 区域的名称 |
    | 服务器名称 | 长度介于 3 至 63 个字符之间的全局唯一字符串，由字母、数字和连字符组成，以字母或数字开头和结尾 |
    | 输入用户名 | **dbadmin** |
    | 位置 | 为 Azure Web 应用的位置选择的同一 Azure 区域的名称 |
    | 数据库名称 | **az400m10l02-db** |

1. 返回到“DevOps Starter”**** 边栏选项卡，在“即将完成”窗格上****，单击“完成”****，然后单击“查看 + 创建”****。

    > 备注：请等待部署完成。 预配 **DevOps Starter** 资源大约需要 2 分钟。

1. 收到已预配 DevOps Starter 资源的确认后，单击“转到资源”**** 按钮。 这会将浏览器重定向到“DevOps Starter”边栏选项卡。
1. 在“DevOps Starter”边栏选项卡上，跟踪 CI/CD 管道的进度，直到它成功完成。

    > **备注**：创建相应的 Azure Web 应用和 Azure SQL 数据库可能需要大约 5 分钟。 此过程会自动创建一个 Azure DevOps 项目，该项目包括可随时部署的存储库以及生成和发布管道。 Azure 资源是作为自动触发的部署管道的一部分创建的。

#### 任务 2：创建 Azure DevOps 发布

在此任务中，将创建多个 Azure DevOps 发布，包括最终会部署失败的发布版本。

1. 在显示 Azure 门户的 Web 浏览器中，在“DevOps Starter”页的工具栏上，单击“项目主页”****。 这将自动打开另一个浏览器标签页，其中显示 Azure Devops 门户中的“创建发布仪表板”**** 项目。 如果系统提示登录，请使用 Azure DevOps 组织凭据进行身份验证。

    > **备注**：首先，将新建一个可以成功部署的版本。

1. 在 Azure DevOps 门户，从左侧垂直菜单中单击“存储库”，在存储库中的文件夹列表中，导航到 Application\\aspnet-core-dotnet-core\\Pages 文件夹，然后单击 Index.chtml 条目************。
1. 在 Index.cshtml 窗格中，单击“编辑”，在行 20 中，将 `<div class="description line-2"> Your ASP.NET Core app is up and running on Azure</div>` 替换为 `<div class="description line-2"> Your ASP.NET Core app v1.1 is up and running on Azure</div>`，单击“提交”，在“提交”窗格中，再次单击“提交”************************。 这将自动触发生成管道。
1. 在 Azure DevOps 门户左侧的垂直导航窗格中，单击“管道”****。
1. 在“管道”**** 窗格的“最近”**** 选项卡上，单击“az400m10l02-CI”**** 条目，在“az400m10l02-CI”**** 窗格的“运行”**** 选项卡上，选择最近一次运行，在该运行的“摘要”**** 窗格上，在“作业”**** 部分中单击“生成”****，然后监视作业直至其成功完成。
1. 作业完成后，在 Azure DevOps 门户左侧的垂直导航窗格中，在“管道”**** 部分中，单击“版本”****。
1. 在“az400m10l02 - CD”**** 窗格的“版本”**** 选项卡上，单击“Release-2”**** 条目，在“Release-2”**** 窗格的“管道”**** 选项卡上，单击“开发”**** 阶段，然后在“开发”**** 窗格上，单击“查看日志”****，然后监视部署的进度直至其成功完成。

    > **备注**：现在，将新建一个部署会失败的版本。 失败将由内置程序集测试引起，该测试将与新发布有关的更改视为无效。

1. 在 Azure DevOps 门户，从左侧垂直菜单中单击“存储库”，在存储库中的文件夹列表中，导航到 Application\\aspnet-core-dotnet-core\\Pages 文件夹，然后单击 Index.chtml 条目************。
1. 在“Index.cshtml”**** 窗格中，单击“编辑”****。 在第 4 行中，将 `ViewData["Title"] = "Home Page - ASP.NET Core";` 替换为 `ViewData["Title"] = "Home Page v1.2 - ASP.NET Core";`****。 单击“提交”。**** 在“提交”窗格中，再次单击“提交”********。 这将自动触发生成管道。
1. 在 Azure DevOps 门户左侧的垂直导航窗格中，单击“管道”****。
1. 在“管道”**** 窗格的“最近”**** 选项卡上，单击“az400m10l02-CI”**** 条目，在“az400m10l02-CI”**** 窗格的“运行”**** 选项卡上，选择最近一次运行，在该运行的“摘要”**** 窗格上，在“作业”**** 部分中单击“生成”****，然后监视作业直至其成功完成。
1. 作业完成后，在 Azure DevOps 门户左侧的垂直导航窗格中，在“管道”**** 部分中，单击“版本”****。
1. 在“az400m10l02 - CD”窗格的“发布”选项卡上，单击“Release-3”条目，在“Release-3”窗格的“管道”选项卡上，单击“开发”阶段，然后在“开发”窗格上，单击“查看日志”，并监视部署的进度，直到部署在“测试程序集”阶段失败************************************。

#### 任务 3：创建 Azure DevOps 发布仪表板

在此任务中，将创建仪表板并将其添加到与发布相关的小组件。

1. 在 Azure DevOps 门户左侧的垂直菜单中，单击“概述”****，在“概述”**** 部分中，单击“仪表板”****，然后单击“添加小组件”****。
1. 在“添加小组件”**** 窗格上，向下滚动小组件列表，选择“部署状态”**** 条目，然后单击“添加”****。
1. 使用上一步中所述的过程添加“发布运行状况详细信息”****、“发布运行状况概述”**** 和“发布管道概述”**** 小组件。
    > **备注**：从市场“团队项目运行状况”[](https://marketplace.visualstudio.com/items?itemName=ms-devlabs.TeamProjectHealth)安装“发布运行状况详细信息”**** 以及“发布运行状况概述”****
1. 使用鼠标将“发布管道概述”**** 拖到“部署状态”**** 小组件的右侧，以避免需要垂直滚动浏览仪表板，然后单击“完成编辑”****。
1. 返回仪表板窗格，在表示“部署状态”**** 小组件的矩形中，单击“配置小组件”****。
1. 在“配置”**** 窗格上，指定以下设置（其他设置保留默认值），然后单击“保存”****。

    | 设置 | “值” |
    | ------- | ----- |
    | 生成管道 | **az400m10l02 - CI** |
    | 链接的发布管道 | **az400m10l02 - CD; az400m10l02 - CD\dev** |

1. 返回仪表板窗格，将鼠标悬停在表示“发布运行状况概述”**** 小组件的矩形右上角，以显示表示“更多操作”**** 菜单的省略号，单击它，然后在下拉菜单中单击“配置”****。  
1. 在“配置”**** 窗格上，指定以下设置（其他设置保留默认值），然后单击“保存”****。

    | 设置 | “值” |
    | ------- | ----- |
    | 选择发布定义 | **az400m10l02 - CD** |

1. 返回仪表板窗格，将鼠标悬停在表示“发布运行状况详细信息”**** 小组件的矩形右上角，以显示表示“更多操作”**** 菜单的省略号，单击它，然后在下拉菜单中单击“配置”****。  
1. 在“配置”**** 窗格上，指定以下设置（其他设置保留默认值），然后单击“保存”****。

    | 设置 | “值” |
    | ------- | ----- |
    | 定义 | **az400m10l02 - CD** |

1. 返回仪表板窗格，将鼠标悬停在表示“发布管道概述”**** 小组件的矩形右上角，以显示表示“更多操作”**** 菜单的省略号，单击它，然后在下拉菜单中单击“配置”****。  
1. 在“配置”**** 窗格上，指定以下设置（其他设置保留默认值），然后单击“保存”****。

    | 设置 | “值” |
    | ------- | ----- |
    | 发布管道 | **az400m10l02 - CD** |

1. 返回仪表板窗格，单击“刷新”**** 以更新小组件显示的内容。

    > **备注**：通过小组件上的链接可直接导航到 Azure DevOps 门户中的相应窗格。

### 练习 2：通过 REST API 查询发布信息

在本练习中，你将使用 Postman 通过 REST API 查询发布信息。

#### 任务 1：生成 Azure DevOps 个人访问令牌

在此任务中，你将生成 Azure DevOps 个人访问令牌，用于对此练习下一任务中要安装的 Postman 应用进行身份验证。

1. 在实验室计算机上显示 Azure DevOps 门户的 Web 浏览器窗口中，单击 Azure DevOps 页面的右上角的“用户设置”图标，在下拉菜单中单击“个人访问令牌”，在“个人访问令牌”窗格中，单击“+ 新建令牌”   。
1. 在“新建个人访问令牌”窗格上，单击“显示所有范围”链接，指定以下设置，然后单击“创建”（将其他设置全部保留为默认值）  ：

    | 设置 | 值 |
    | --- | --- |
    | 名称 | **创建“发布仪表板”实验室** |
    | 范围 | **版本** |
    | 权限 | **读取** |
    | 作用域 | **生成** |
    | 权限 | **读取** |

1. 在“成功”窗格上，将个人访问令牌的值复制到剪贴板。

    > **注意**：确保记下令牌的值。 关闭此窗格后，将无法再检索它。

1. 在“成功”窗格中，单击“关闭” 。

#### 任务 2：使用 Postman 通过 REST API 查询发布信息

在此任务中，你将使用 Postman 通过 REST API 查询发布信息。

1. 在实验室计算机上，启动 Web 浏览器并导航到“Postman 下载页面”[](https://www.postman.com/downloads/)，单击“下载应用”**** 按钮，在下拉菜单中，单击“Windows 64 位”****，然后单击下载的文件并运行安装。 安装完成后，Postman 桌面应用将自动启动。
1. 在“创建 Postman 帐户”**** 窗格中，提供电子邮件地址、用户名和密码，然后单击“创建免费帐户”****。

    > **备注**：将收到 Postman 的电子邮件，用于激活 Postman 帐户以完成帐户预配过程。

1. 登录后，在 Postman 桌面应用窗口的左上角单击“+ 新建”****，在“新建”**** 窗格上单击“请求”****，在“保存请求”**** 窗格的“请求名称”**** 文本框中，键入“Get-Releases”****，单击“+ 创建集合”****，在“命名集合”**** 文本框中，键入“Azure DevOps az400m10l02 查询”****，单击右侧的复选标记， 然后单击“保存到 Azure DevOps az400m10l02 查询”按钮****。
1. 打开另一个 Web 浏览器窗口并导航到[“版本 - 列表”**** Microsoft 文档页](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/releases/list?view=azure-devops-rest-6.0)并查看其内容。
1. 切换回 Postman 桌面应用，在应用窗口右上角的“启动板”窗格中，单击“授权”**** 选项卡标题，在“类型”**** 下拉列表中，选择“基本身份验证”**** 条目，然后在“密码”**** 文本框中粘贴在上一任务中复制的令牌的值（请勿设置“用户名”**** 文本框的值）。
1. 在应用窗口右上角部分的启动板窗格中，确保“获取”显示在下拉列表中，在“输入请求 URL”文本框中，键入以下内容并单击“发送”（将 `<organization_name>` 的值替换为 Azure DevOps 组织的名称），以列出所有发布************：

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/releases?api-version=6.0
    ```

1. 查看应用窗口右下角的“正文”**** 选项卡上列出的输出，并验证它是否包含在此实验室上一练习中创建的发布列表。
1. 切换到显示 Microsoft Docs 内容的 Web 浏览器窗口，然后导航到[“部署 - 列表”**** Microsoft Docs 页](https://docs.microsoft.com/en-us/rest/api/azure/devops/release/deployments/list?view=azure-devops-rest-6.0)并查看其内容。
1. 在应用窗口右上角部分的启动板窗格中，确保“获取”显示在下拉列表中，在“输入请求 URL”文本框中，键入以下内容并单击“发送”（将 `<organization_name>` 的值替换为 Azure DevOps 组织的名称），以列出所有部署************：

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?api-version=6.0
    ```

1. 查看应用窗口右下角的“正文”**** 选项卡上列出的输出，并验证它是否包含在此实验室上一练习中启动的部署列表。
1. 在应用窗口右上角部分的启动板窗格中，确保“获取”显示在下拉列表中，在“输入请求 URL”文本框中，键入以下内容并单击“发送”（将 `<organization_name>` 的值替换为 Azure DevOps 组织的名称），以列出所有部署************：

    ```url
    https://vsrm.dev.azure.com/<organization_name>/Creating%20a%20Release%20Dashboard/_apis/release/deployments?DeploymentStatus=failed&api-version=6.0
    ```

1. 查看应用窗口右下角的“正文”**** 选项卡上列出的输出，并验证它是否仅包含在此实验室上一练习中启动的失败部署。

### 练习 3：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m10l02-rg')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 审阅

在本实验室中，你学习了如何创建和配置发布仪表板，以及如何使用 REST API 检索 Azure DevOps 发布数据。