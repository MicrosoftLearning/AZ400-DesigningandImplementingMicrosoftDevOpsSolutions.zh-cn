---
lab:
  title: 使用 Azure Artifacts 进行包管理
  module: 'Module 08: Design and implement a dependency management strategy'
---

# 使用 Azure Artifacts 进行包管理

# 学生实验室手册

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

- 设置示例 EShopOnWeb 项目：如果还没有可用于本实验室的示例 EShopOnWeb 项目，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

- 可从 [Visual Studio 下载页面](https://visualstudio.microsoft.com/downloads/)获得 Visual Studio 2022 Community Edition。 Visual Studio 2022 安装应包括“ASP<nolink>.NET 和 Web 开发”、“Azure 开发”和“NET Core 跨平台开发”工作负载。

## 实验室概述

Azure Artifacts 有助于在 Azure DevOps 中发现、安装和发布 NuGet、npm 和 Maven 包。 它与其他 Azure DevOps 功能（例如生成）深度集成，使包管理成为现有工作流的无缝组成部分。

## 目标

完成本实验室后，你将能够：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。

## 预计用时：40 分钟

## 说明

### 练习 0：配置实验室先决条件

在此提醒，在本练习中，需要验证实验室先决条件，既要准备好 Azure DevOps 组织，又要创建 EShopOnWeb 项目。 请参阅上面的说明了解详细信息。

#### 任务 1：在 Visual Studio 中配置 EShopOnWeb 解决方案

在此任务中，你将配置 Visual Studio，以便为实验室做准备。

1. 确保你正在 Azure DevOps 门户上查看 EShopOnWeb 团队。

    > 注意：可通过导航到 [https://dev.azure.com/`<your-Azure-DevOps-account-name>` /EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb) URL 直接访问项目页，其中 `<your-Azure-DevOps-account-name>` 占位符表示 Azure DevOps 组织名称。

1. 在“EShopOnWeb”窗格左侧的垂直菜单中，单击“Repos”。
1. 在“文件”窗格中，单击“克隆”，选择“在文件中克隆”旁边的下拉 VS Code，在下拉菜单中，选择“Visual Studio”   。
1. 如果系统提示你是否继续，请单击“打开”。
1. 如果系统出现提示，请使用用于设置 Azure DevOps 组织的用户帐户登录。
1. 在 Visual Studio 界面的“Azure DevOps”弹出窗口中，接受默认的本地路径 (C:\EShopOnWeb)，然后单击“克隆”。 这会将自动将项目导入 Visual Studio。
1. 将 Visual Studio 窗口保持打开状态，以便在实验室中使用。

### 练习 1：使用 Azure Artifacts

在本练习中，你将通过以下步骤了解如何使用 Azure Artifacts：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。

#### 任务 1：创建并连接到源

在此任务中，你将创建并连接到源。

1. 在显示 Azure DevOps 门户中项目设置的 Web 浏览器窗口中，在垂直导航窗格中，选择 Artifacts。
1. 显示 Artifacts 中心时，单击窗格顶部的“+ 创建源” 。

    > **注意**：此源将是可供组织内用户使用的 NuGet 包的集合，并且将作为对等方与公共 NuGet 源并置。 本实验室中的方案将重点放在使用 Azure Artifacts 的工作流上，因此实际的体系结构和开发决策纯粹是说明性的。  此源将包含可以在该组织中的各个项目之间共享的通用功能。

1. 在“创建新源”窗格的“名称”文本框中，键入 EShopOnWebShared，在“范围”部分，选择“组织”选项，将其他设置保留为默认值，然后单击“创建”。

    > **注意**：任何想要连接到此 NuGet 源的用户都必须配置其环境。

1. 返回 Artifacts 中心，单击“连接到源” 。
1. 在“连接到源”窗格的 NuGet 部分，选择 Visual Studio，然后在 Visual Studio 窗格中复制“源”URL    。 https://pkgs.dev.azure.com/<Azure-DevOps-Org-Name>_packaging/EShopOnWebShared/nuget/v3/index.json)
1. 切换回 Visual Studio 窗口。
1. 在 Visual Studio 窗口中，单击“工具”菜单标题，在下拉菜单中，选择“NuGet 包管理器”，然后在级联菜单中选择“包管理器设置”  。
1. 在“选项”对话框中，单击“包源”，然后单击加号以添加新的包源 。
1. 在对话框底部的“名称”文本框中，将“包源”替换为 EShopOnWebShared，然后在“源”文本框中，粘贴在 Azure DevOps 门户中复制的 URL。
1. 单击“更新”，然后单击“确定”以完成添加 。

    > **注意**：Visual Studio 现已连接到新源。

#### 任务 2：创建和发布内部开发的 NuGet 包

在此任务中，你将创建并发布内部开发的自定义 NuGet 包。

1. 在用于配置新包源的 Visual Studio 窗口中，在主菜单中单击“文件”，在下拉菜单中单击“新建”，然后在级联菜单中单击“项目”  。

    > **注意**：现在，我们将创建一个共享程序集，该程序集将以 NuGet 包的形式发布，以便其他团队可以集成该程序集并保持同步，而不必直接使用项目源。

1. 在“新建项目”窗格的“最新项目模板”页面上，使用搜索文本框找到“类库”模板，选择 C# 模板，然后单击“下一步”   。
1. 在“新建项目”窗格的“类库”页面上，指定以下设置，然后单击“创建”：

    | 设置 | 值 |
    | --- | --- |
    | 项目名称 | EShopOnWeb.Shared |
    | 位置 | 接受默认值 |
    | 解决方案 | **创建新解决方案** |
    | 解决方案名称 | EShopOnWeb.Shared |
    
    使“将解决方案和项目放在同一目录中”设置保持启用状态。 

1. 单击“下一步”。 接受“.NET 6.0 (长期支持)”作为框架选项。
1. 按“创建”按钮确认项目创建。
    
1. 在 Visual Studio 界面的“解决方案资源管理器”窗格中，右键单击 Class1.cs，在右键菜单中选择“删除”，并在出现确认提示时单击“确定”   。
1. 按 Ctrl+Shift+B 或右键单击 EShopOnWeb.Shared 项目，然后选择“生成”以生成项目。

    > **注意**：在下一个任务中，我们将使用 NuGet.exe 直接从生成的项目中生成 NuGet 包，但需要先生成项目。

1. 切换到显示 Azure DevOps 门户的 Web 浏览器。
1. 导航到“连接到源”窗格的 NuGet 部分，然后选择 NuGet.exe  。 这将显示 NuGet.exe 窗格。
1. 在 NuGet.exe 窗格上，单击“获取工具” 。
1. 在“获取工具”窗格上，单击“下载最新的 NuGet”链接 。 这将自动打开另一个浏览器选项卡，其中显示“可用的 NuGet 发行版”页面。
1. 在“可用的 NuGet 分发版本”页上，选择“nuget.exe - 建议的最新 v6.x”，并将可执行文件下载到本地“EShopOnWeb.Shared Project”文件夹（如果保留默认文件夹位置，则应为 C:\EShopOnWeb\EShopOnWeb.Shared）。
1. 选择“nuget.exe”文件，然后右键单击文件，从上下文菜单中选择“属性”，打开其属性。
1. 在“属性”上下文窗口中的“常规”选项卡中，选择“安全性”部分下的“取消阻止”。 按“应用”和“确定”进行确认。
1. 在实验室工作站中，打开“开始”菜单，然后搜索“Windows PowerShell”。 接下来，在级联菜单中，单击“以管理员身份打开 Windows PowerShell”。
1. 在“管理员: Windows PowerShell”窗口中，通过执行以下命令导航到 EShopOnWeb.Shared 文件夹： 

```
cd c:\EShopOnWeb\EShopOnWeb.Shared
```
运行以下命令，从项目创建 **.nupkg** 文件。

    > **Note**: This is a shortcut to package the NuGet bits for deployment. NuGet is highly customizable. To learn more, refer to the [NuGet package creation page](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow).

    ```
    .\nuget.exe pack ./EShopOnWeb.Shared.csproj
    ```

    > **Note**: Disregard any warnings displayed in the **Administrator: Windows PowerShell** window.

    > **Note**: NuGet builds a minimal package based on the information it is able to identify from the project. For example, note that the name is **EShopOnWeb.Shared.1.0.0.nupkg**. That version number was retrieved from the assembly.

1. 成功创建包后，运行以下命令，将包发布到 EShopOnWebShared 源：

    > **注意**：你需要提供一个 API 密钥，该密钥可以是任何非空字符串。 我们在此处使用 AzDO。 当系统出现提示时，登录到 Azure DevOps 组织。

    ```
    .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO EShopOnWeb.Shared.1.0.0.nupkg
    ```
1. 等待确认包推送操作成功。
1. 切换到显示 Azure DevOps 门户的 Web 浏览器窗口，然后在垂直导航窗格中选择 Artifacts。
1. 在“项目”中心窗格上，单击左上角的下拉列表，然后在源列表中选择“EShopOnWebShared”条目。

    > 注意：EShopOnWebShared 源应该包括新发布的 NuGet 包。

1. 单击 NuGet 包以显示其详细信息。

#### 任务 3：将开源的 NuGet 包导入 Azure DevOps 包源

除了开发自己的包之外，是不是也可以使用开源 Nuget (https://www.nuget.org) DotNet 包库呢？ 由于有几百万个包可用，总会有一些对你的应用程序有用的包。

在此任务中，我们将使用通用的“Hello World”示例包，但你可以对库中的其他包使用相同的方法。

1. 在同一 PowerShell 窗口中，运行以下 nuget 命令，安装示例包：

```
.\nuget install HelloWorld -ExcludeVersion
```

1. 检查安装过程的输出。 第一行显示了它将尝试下载包的不同的源：
```
Feeds used:
  https://api.nuget.org/v3/index.json
  https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/EShopOnWebPFeed/nuget/v3/index.json
```
1. 接下来，它将显示有关实际安装过程本身的其他输出。

```
Installing package 'Helloworld' to 'C:\eShopOnWeb\EShopOnWeb.Shared'.
  GET https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json
  OK https://api.nuget.org/v3/registration5-gz-semver2/helloworld/index.json 114ms
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  GET https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json
  OK https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v3/registrations2-semver2/helloworld/index.json 698ms

Attempting to gather dependency information for package 'Helloworld.1.3.0.17' with respect to project 'C:\eShopOnWeb\EShopOnWeb.Shared', targeting 'Any,Version=v0.0'
Gathering dependency information took 21 ms
Attempting to resolve dependencies for package 'Helloworld.1.3.0.17' with DependencyBehavior 'Lowest'
Resolving dependency information took 0 ms
Resolving actions to install package 'Helloworld.1.3.0.17'
Resolved actions to install package 'Helloworld.1.3.0.17'
Retrieving package 'HelloWorld 1.3.0.17' from 'nuget.org'.
  GET https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg
  OK https://api.nuget.org/v3-flatcontainer/helloworld/1.3.0.17/helloworld.1.3.0.17.nupkg 133ms
Installed HelloWorld 1.3.0.17 from https://api.nuget.org/v3/index.json with content hash 1Pbk5sGihV5JCE5hPLC0DirUypeW8hwSzfhD0x0InqpLRSvTEas7sPCVSylJ/KBzoxbGt2Iapg72WPbEYxLX9g==.
Adding package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Added package 'HelloWorld.1.3.0.17' to folder 'C:\eShopOnWeb\EShopOnWeb.Shared'
Successfully installed 'HelloWorld 1.3.0.17' to C:\eShopOnWeb\EShopOnWeb.Shared
Executing nuget actions took 686 ms
```
1. HelloWorld 包安装在 EShopOnWeb.Shared 文件夹下的 HelloWorld 子文件夹中。 从 Visual Studio 解决方案资源管理器导航到 EShopOnWeb.Shared 项目，注意到 HelloWorld 子文件夹。 单击子文件夹左侧的小箭头，打开文件夹和文件列表。 
1. 注意到 lib 子文件夹中有一个 signature.p7s 签名文件，它证明了包的来源。 接下来，注意到 HelloWorld.nupkg 包文件本身。 

#### 任务 4：将开源的 NuGet 包上传到 Azure Artifacts
我们将此包视为一个“已批准”的包，可供我们的 DevOps 团队重复使用，方法是将其上传到之前创建的 Azure Artifacts 包源。

1. 在 PowerShell 窗口中，执行以下命令：

 ```
.\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```
    > 注意：这会导致出现错误消息：

```
Response status code does not indicate success: 409 (Conflict - 'HelloWorld 1.3.0.17' cannot be published to the feed because it exists in at least one of the feed's upstream sources. Publishing this copy would prevent you from using 'HelloWorld 1.3.0.17' from 'NuGet Gallery'. For more information, see https://go.microsoft.com/fwlink/?linkid=864880 (DevOps Activity ID: AE08BE89-C2FA-4FF7-89B7-90805C88972C)).
```

当你创建了 Azure DevOps 项目包源后，根据设计，它允许使用上游源，例如 dotnet 示例中的 nuget.org。 但是，没有任何内容会阻止 DevOps 团队创建“仅限内部”的包源。

1. 导航到 Azure DevOps 门户，浏览到“项目”，然后选择“EShopOnWebShared 源”。 
1. 单击“搜索上游源”
1. 在“转到上游包”窗口中，选择“Nuget”作为“包类型”，然后在搜索字段中输入“HelloWorld”。
1. 按“搜索”按钮进行确认。
1. 这会生成一个具有不同可用版本的所有 HelloWorld 包的列表。
1. 单击向左箭头键可返回到 EShopOnWebShared 源。   
1. 单击齿轮，打开“源设置”。 在“源设置”页中，选择“上游源”。 
1. 注意到不同开发语言的不同上游包管理器。 从列表中选择“Nuget.org”。 按“删除”按钮，然后按“保存”按钮。

1. 保存这些更改后，可以通过重新启动以下命令，从 PowerShell 窗口使用 Nuget.exe 上传 HelloWorld 包：

```
 .\nuget.exe push -source "EShopOnWebShared" -ApiKey AzDO c:\EShopOnWeb\EShopOnWeb.Shared\HelloWorld\HelloWorld.nupkg
```

> 注意：此时上传应该会成功 

```
Pushing HelloWorld.nupkg to 'https://pkgs.dev.azure.com/pdtdemoworld/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/'...
  PUT https://pkgs.dev.azure.com/<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/
MSBuild auto-detection: using msbuild version '17.5.0.10706' from 'C:\Program Files\Microsoft Visual Studio\2022\Professional\MSBuild\Current\bin'.
  Accepted https://pkgs.dev.azure.com/pdtdemoworld<AZUREDEVOPSORGANIZATION>/7dc3351f-bb0c-42ba-b3c9-43dab8e0dc49/_packaging/188ec0d5-ff93-4eb7-b9d3-293fbf759f06/nuget/v2/ 1645ms
Your package was pushed.
PS C:\eShopOnWeb\EShopOnWeb.Shared>
```

1. 在 Azure DevOps 门户中，刷新“项目包源”页。 包列表同时显示了 EShopOnWeb.Shared 自定义开发的包，以及 HelloWorld 公共源代码包。
1. 在 Visual Studio 的 EShopOnWeb.Shared 解决方案中，右键单击“EShopOnWeb.Shared”项目，然后从上下文菜单中选择“管理 Nuget 包”。 
1. 在“Nuget 包管理器”窗口中，验证“包源”是否已设置为“EShopOnWebShared”。
1. 单击“浏览”，并等待 Nuget 包列表加载完成。
1. 此列表还会同时显示 EShopOnWeb.Shared 自定义开发的包，以及 HelloWorld 公共源代码包。

## 审阅

在本实验室中，你通过以下步骤了解了如何使用 Azure Artifacts：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入自定义开发的 NuGet 包。
- 导入公共源 NuGet 包。
