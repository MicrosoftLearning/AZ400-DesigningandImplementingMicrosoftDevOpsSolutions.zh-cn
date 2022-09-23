---
lab:
  title: 实验室 12：设置和运行功能测试
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
ms.openlocfilehash: 74c12ec06dce2b6a033a17744c761b7b9af201eb
ms.sourcegitcommit: d78aebd7b14277a53f152e26cea68a30b0e90d73
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/13/2022
ms.locfileid: "146276053"
---
# <a name="lab-12-setting-up-and-running-functional-tests"></a>实验室 12：设置和运行功能测试

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

## <a name="lab-overview"></a>实验室概述

[Selenium](http://www.seleniumhq.org/) 是一个用于 Web 应用程序的可移植开源软件测试框架。 它几乎可以在所有操作系统上运行。 它支持所有现代浏览器和多种语言，包括 .NET (C#) 和 Java。

本实验室将介绍如何在 C# Web 应用程序上执行 Selenium 测试用例，这是 Azure DevOps 发布管道的一部分。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 配置自托管 Azure DevOps 代理。
- 配置发布管道。
- 触发生成和发布。
- 在 Chrome 和 Firefox 中运行测试。

## <a name="estimated-timing-60-minutes"></a>预计用时：60 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板预配置的 Parts Unlimited 团队项目和 Azure 资源。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 Selenium 模板生成一个新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关该站点的详细信息，请参阅 [什么是 Azure DevOps 服务演示生成器？](https://docs.microsoft.com/en-us/azure/devops/demo-gen)。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的页面的“新建项目名称”文本框中，键入“设置和运行功能测试”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在模板列表中，单击工具栏中的“DevOps 实验室”，选择“Selenium”模板并单击“选择模板”  。
1. 返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-create-azure-resources"></a>任务 2：创建 Azure 资源

在此任务中，你将预配运行 Windows Server 2016、SQL Express 2017、Chrome 和 Firefox 的 Azure VM。

1. 单击此处的[部署到 Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2Falmvm%2Fmaster%2Flabs%2Fvstsextend%2Fselenium%2Farmtemplate%2Fazuredeploy.json) 链接。 这会自动重定向到 Azure 门户中的“自定义部署”边栏选项卡。
1. 出现提示时，使用用户帐户登录，该帐户在你打算在本实验室中使用的 Azure 订阅中具有所有者角色，并且在该订阅关联的 Azure AD 租户中具有全局管理员角色。
1. 在“自定义部署”边栏选项卡上，选择“编辑模板” 。
1. 在“编辑模板”边栏选项卡上，找到行 `"https://raw.githubusercontent.com/microsoft/azuredevopslabs/master/labs/vstsextend/selenium/armtemplate/chrome_firefox_VSTSagent_IIS.ps1"`，将其替换为 `https://raw.githubusercontent.com/MicrosoftLearning/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/master/Allfiles/Labs/11b/chrome_firefox_VSTSagent_IIS.ps1`，然后单击“保存” 。
1. 返回“自定义部署”边栏选项卡，指定以下设置：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 az400m11l02-RG |
    | 区域 | 你想要在本实验室中用于部署 Azure 资源的 Azure 区域的名称 |
    | 虚拟机名称 | az40011bvm |

1. 依次单击“查看 + 创建”、“创建”。 

    > **注意**：等待此过程完成。 该操作需要约 15 分钟。

### <a name="exercise-1-implement-selenium-tests-by-using-a-self-hosted-azure-devops-agent"></a>练习 1：使用自托管 Azure DevOps 代理实现 Selenium 测试

在本练习中，你将使用自托管 Azure DevOps 代理实现 Selenium 测试。

#### <a name="task-1-configure-a-self-hosted-azure-devops-agent"></a>任务 1：配置自托管 Azure DevOps 代理

在此任务中，你将使用在上一练习中部署的 VM 配置自托管代理。 Selenium 要求此代理以交互模式运行，以执行 UI 测试。

1. 在显示 Azure 门户的 Web 浏览器窗口中，搜索并选择“虚拟机”，然后从“虚拟机”边栏选项卡中选择“az40011bvm”。
1. 在 az40011bvm 边栏选项卡上，选择“连接”，在下拉菜单中选择 RDP，在 az40011bvm 连接边栏选项卡的 RDP 选项卡上，选择“下载 RDP 文件”并打开下载的文件    **\|** 。
1. 系统出现提示时，请使用以下凭据登录：

    | 设置 | 值 |
    | --- | --- |
    | 用户名 | **vmadmin** |
    | 密码 | **P2ssw0rd@123** |

1. 在 az40011bvm 远程桌面会话中，打开 Chrome Web 浏览器窗口，导航到 <https://dev.azure.com> ，并登录 Azure DevOps 组织 。
1. 在 Azure DevOps 门户的左下角单击“组织设置” 。
1. 在页面左侧的垂直菜单中，在“管道”部分单击“代理池” 。
1. 在“代理池”窗格上，单击“默认” 。
1. 在“默认”窗格上，单击“新建代理” 。
1. 在“获取代理”面板上，确保选中“Windows”选项卡和“x64”部分，然后单击“下载”   。
1. 启动文件资源管理器，创建目录 C:\\AzAgent 并将位于 Downloads 文件夹中的已下载代理 zip 文件的内容提取到此目录中 。
1. 在 az40011bvm 远程桌面会话中，右键单击“启动”菜单，然后单击“命令提示符(管理员)”  。
1. 在“管理员: 命令提示符”窗口中，运行以下命令以开始安装代理二进制文件：

    ```cmd
    cd C:\AzAgent
    Config.cmd
    ```

1. 在“管理员:命令提示符”窗口中，系统提示“输入服务器 URL”时，键入 https://dev.azure.com/\<your-DevOps-organization-name\> ，其中 \<your-DevOps-organization-name\> 表示 Azure DevOps 组织的名称，然后按 Enter    。
1. 在“管理员:命令提示符”窗口中，系统提示“输入身份验证类型(按 Enter 表示 PAT)”时，按 Enter 。
1. 在“管理员:命令提示符”窗口中，系统提示“输入个人访问令牌”时，切换到 Azure DevOps 门户，关闭“获取代理”面板，在 Azure DevOps 页的右上角，单击“用户设置”图标，在下拉菜单中，单击“个人访问令牌”，然后在“个人访问令牌”窗格上，单击“+ 新建令牌”     。
1. 在“创建新的个人访问令牌”窗格上，指定以下设置，然后单击“创建”（所有其他设置保留默认值） ：

    | 设置 | 值 |
    | --- | --- |
    | 名称 | **设置和运行功能测试实验室** |
    | 作用域 | **自定义** |
    | 作用域 | 单击窗口底部的“显示所有范围”按钮 |
    | 作用域 | **代理池** - **读取和管理** |

1. 在“成功”窗格上，将个人访问令牌的值复制到剪贴板。

    > **注意**：确保复制令牌。 关闭此窗格后，将无法再检索它。

1. 在“成功”窗格中，单击“关闭” 。
1. 切换回“管理员: 命令提示符”窗口，粘贴剪贴板的内容，然后按 Enter。
1. 在“管理员:命令提示符”窗口中，系统提示“输入代理池(按 Enter 表示默认值)”时，按 Enter 。
1. 在“管理员:命令提示符”窗口中，系统提示“输入代理名称(按 Enter 表示 az40011bvm)”时，按 Enter  。
1. 在“管理员:命令提示符”窗口中，系统提示“输入工作文件夹(按 Enter 表示 _work)”时，按 Enter 。
1. 在“管理员:命令提示符”窗口中，系统提示输入“输入是否将代理作为服务运行(是/否)(按 Enter 表示否)”时，按 Enter 。
1. 在“管理员:命令提示符”窗口中，系统提示“输入是否配置自动登录并在启动时运行代理(是/否)(按 Enter 表示否)”时，按 Enter 。
1. 注册代理后，在“管理员: 命令提示符”窗口中，键入“run.cmd”并按 Enter 以启动代理 。

    > **注意**：还需安装 Dac Framework，你稍后在本实验室中部署的应用程序会用到它。

1. 在与 az40011bvm 的远程桌面会话中，启动 Web 浏览器的另一个实例，导航到 [Microsoft SQL Server 数据层应用程序框架 (18.2) 下载页](https://www.microsoft.com/en-us/download/details.aspx?id=58207&WT.mc_id=rss_alldownloads_extensions)，然后单击“下载” 。
1. 在“选择所需的下载”上，选中“EN\x64\DacFramework.msi”复选框，然后单击“下一步”  。 这将触发自动下载 DacFramework.msi 文件。
1. 下载 DacFramework.msi 文件后，使用该文件安装 Microsoft SQL Server 数据层应用程序框架并采用默认设置。

#### <a name="task-2-configure-a-release-pipeline"></a>任务 2：配置发布管道

在此任务中，你将配置发布管道。

> **注意**：Azure VM 已将代理配置为部署应用程序和运行 Selenium 测试用例。 发布定义使用要部署到目标服务器的[阶段](https://docs.microsoft.com/en-us/vsts/build-release/concepts/process/phases)。

1. 在 az40011bvm 远程桌面会话中，在显示 Azure DevOps 门户的浏览器窗口中，单击左上角的“Azure DevOps”符号  。
1. 在显示组织项目的窗格上，单击表示“设置和运行功能测试”项目的磁贴。
1. 在“设置和运行功能测试”窗格的垂直导航窗格中，选择“管道”，在“管道”部分，单击“发布”，然后在“Selenium”窗格上单击“编辑”     。
1. 在“所有管道”>“Selenium”窗格上，单击“任务”选项卡标题，然后单击下拉菜单中的“开发”  。
1. 在“开发”阶段的任务列表中，查看“IIS 部署”、“SQL 部署”和“Selenium 测试执行”部署阶段   。

- **IIS 部署阶段**：在此阶段，使用以下任务将应用程序部署到 VM：

  - **IIS Web 应用管理**：此任务在已注册代理的目标计算机上运行。 它会在本地创建一个网站和一个在端口 82 下运行的名为“PartsUnlimited”的应用程序池  [ **http://localhost:82**](http://localhost:82)
  - **IIS Web 应用部署**：此任务使用“Web 部署”将应用程序部署到 IIS 服务器。

- **数据库部署阶段**：在此阶段，我们使用 [**SQL Server 数据库部署**](https://github.com/Microsoft/vsts-tasks/blob/master/Tasks/SqlDacpacDeploymentOnMachineGroup/README.md) 任务将 [**dacpac**](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications) 文件部署到 DB 服务器。

- **Selenium 测试执行**：通过在发布过程中执行 UI 测试可以检测到意外更改。 设置基于浏览器的自动测试可提高应用程序的质量，免去了手动进行测试。 在此阶段，我们将对已部署的 Web 应用程序执行 Selenium 测试。 后续任务描述了如何使用 Selenium 在发布管道中测试网站。

  - **Visual Studio 测试平台安装程序**：[Visual Studio 测试平台安装程序](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/tool/vstest-platform-tool-installer?view=vsts)任务将从 nuget.org 或指定源获取 Microsoft 测试平台，并将其添加到工具缓存。 它满足 vstest 要求，因此无需在代理计算机上完整安装 Visual Studio 即可运行生成或发布管道中的任何后续 Visual Studio 测试任务。
  - **运行 Selenium UI 测试**：此 [任务](https://github.com/Microsoft/azure-pipelines-tasks/blob/master/Tasks/VsTestV2/README.md)使用 vstest.console.exe 在代理计算机上执行 selenium 测试用例。

1. 在“所有管道”>“Selenium”窗格上，单击“IIS 部署”阶段，然后在“代理作业”窗格上，验证是否选中默认代理池   。
1. 对“SQL 部署”和“Selenium 测试执行”阶段重复上述步骤 。 如有需要，单击“保存”以保存更改。

#### <a name="task-3-trigger-build-and-release"></a>任务 3：触发生成和发布

在此任务中，我们将触发“生成”以编译 Selenium C# 脚本和 Web 应用程序。 生成的二进制文件会被复制到自托管代理，而 Selenium 脚本将在自动发布过程中被执行。

1. 在 az40011bvm 远程桌面会话中，在显示 Azure DevOps 门户的浏览器窗口的垂直导航窗格中，单击“管道”部分的“管道”，然后在“管道”窗格上单击“Selenium”     。
1. 在“Selenium”窗格上，单击“运行管道”，然后在“运行管道”上，单击“运行”   。

    > **注意**：此生成会将测试项目发布到 Azure DevOps，在发布中将使用这些项目。

    > **注意**：成功生成后，将触发发布。

1. 在“管道运行”窗格的“作业”部分，单击“阶段 1”并监视生成进度，直至完成 。
1. 在显示 Azure DevOps 门户的浏览器窗口的垂直导航窗格中，单击“管道”部分的“发布”，单击表示发布的条目，然后在“Selenium”>“Release-1”窗格上，单击“开发”    。
1. 在“Selenium”>“Release-1”>“开发”窗格上，监视相应的部署。
1. 在“Selenium 测试执行”阶段开始时，监视 Web 浏览器测试。
1. 发布完成后，在“Selenium”>“Release-1”>“开发”窗格上，单击“测试”选项卡以分析测试结果 。 从“结果”部分的下拉列表中选择所需的筛选器，以查看测试及其状态。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m11l02-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本实验室中，你已了解如何在 C# Web 应用程序上执行 Selenium 测试用例，这是 Azure DevOps 发布管道的一部分。
