---
lab:
  title: 使用 Azure Artifacts 进行包管理
  module: 'Module 07: Design and implement a dependency management strategy'
---

# 使用 Azure Artifacts 进行包管理

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

- 设置示例 eShopOnWeb 项目：**** 如果还没有可用于本实验室的示例 EShopOnWeb 项目，请按照 [AZ-400 实验室先决条件](https://microsoftlearning.github.io/AZ400-DesigningandImplementingMicrosoftDevOpsSolutions/Instructions/Labs/AZ400_M00_Validate_lab_environment.html)中的说明创建一个。

- 可从 [Visual Studio 下载页面](https://visualstudio.microsoft.com/downloads/)获得 Visual Studio 2022 Community Edition。 Visual Studio 2022 安装应包括“ASP<nolink>.NET 和 Web 开发”、“Azure 开发”和“NET Core 跨平台开发”工作负载。

- .NET Core SDK：****[下载并安装 .NET Core SDK (2.1.400+)](https://go.microsoft.com/fwlink/?linkid=2103972)

- Azure Artifacts 凭据提供程序：****[下载并安装凭据提供程序](https://go.microsoft.com/fwlink/?linkid=2099625)。

## 实验室概述

Azure Artifacts 有助于在 Azure DevOps 中发现、安装和发布 NuGet、npm 和 Maven 包。 它与其他 Azure DevOps 功能（例如生成）深度集成，使包管理成为现有工作流的无缝组成部分。

## 目标

完成本实验室后，你将能够：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。

## 预计用时：35 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，将设置实验室先决条件。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb，并将其他字段保留默认值。 单击“创建”。

   ![“创建新项目”面板的屏幕截图。](images/create-project.png)

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“**Repos > 文件**”、“**导入存储库**”。 选择“**导入**”。 在“导入 Git 存储库”窗口中，粘贴以下 URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> 并单击“导入”：

   ![“导入存储库”面板的屏幕截图。](images/import-repo.png)

1. 存储库按以下方式组织：
   - .ado 文件夹包含 Azure DevOps YAML 管道。
   - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
   - infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
   - .github 文件夹容器 YAML GitHub 工作流定义。
   - src 文件夹包含用于实验室方案的 .NET 8 网站。****

#### 任务 3：（如果已完成，请跳过此任务）将主分支设置为默认分支

1. 转到“Repos”>“分支”****。
1. 将鼠标指针悬停在主分支上，然后单击列右侧的省略号。
1. 单击“设置为默认分支”。

#### 任务 4：在 Visual Studio 中配置 eShopOnWeb 解决方案

在此任务中，你将配置 Visual Studio，以便为实验室做准备。

1. 确保你正在 Azure DevOps 门户上查看 eShopOnWeb 团队。****

   > **注意**：可通过导航到 [https://dev.azure.com/`<your-Azure-DevOps-account-name>`/EShopOnWeb](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/eShopOnWeb) URL 直接访问项目页，其中 `<your-Azure-DevOps-account-name>` 占位符表示 Azure DevOps 组织名称。

1. 在“eShopOnWeb”窗格左侧的垂直菜单中，单击“Repos”。********
1. 在“文件”窗格中，单击“克隆”，选择“在文件中克隆”旁边的下拉 VS Code，在下拉菜单中，选择“Visual Studio”   。
1. 如果系统提示你是否继续，请单击“打开”。
1. 如果系统出现提示，请使用用于设置 Azure DevOps 组织的用户帐户登录。
1. 在 Visual Studio 界面的“Azure DevOps”弹出窗口中，接受默认的本地路径 (C:\eShopOnWeb)，然后单击“克隆”。******** 这会将自动将项目导入 Visual Studio。
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

   > **注意**：此源将是可供组织内用户使用的 NuGet 包的集合，并且将作为对等方与公共 NuGet 源并置。 本实验室中的方案将重点放在使用 Azure Artifacts 的工作流上，因此实际的体系结构和开发决策纯粹是说明性的。 此源将包含可以在该组织中的各个项目之间共享的通用功能。

1. 在“**新建源**”窗格中的“**名称**”文本框中，键入 **`eShopOnWebShared`**。 在“**可见性**”部分，选择“特定人员”，然后在“**作用域**”部分选择“**Project:eShopOnWeb**”选项，保留其他设置的默认值，然后单击“**创建**”。

   > **注意**：任何想要连接到此 NuGet 源的用户都必须配置其环境。

1. 返回 Artifacts 中心，单击“连接到源” 。
1. 在“连接到源”窗格的 NuGet 部分，选择 Visual Studio，然后在 Visual Studio 窗格中复制“源”URL    。 `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json`
1. 切换回 Visual Studio 窗口。
1. 在 Visual Studio 窗口中，单击“工具”菜单标题，在下拉菜单中，选择“NuGet 包管理器”，然后在级联菜单中选择“包管理器设置”  。
1. 在“选项”对话框中，单击“包源”，然后单击加号以添加新的包源 。
1. 在对话框底部的“名称”文本框中将“包源”替换为 eShopOnWebShared，然后在“源”文本框中粘贴在 Azure DevOps 门户中复制的 URL。****************
1. 单击“更新”，然后单击“确定”以完成添加 。

   > **注意**：Visual Studio 现已连接到新源。

#### 任务 2：创建和发布内部开发的 NuGet 包

在此任务中，你将创建并发布内部开发的自定义 NuGet 包。

1. 在用于配置新包源的 Visual Studio 窗口中，在主菜单中单击“文件”，在下拉菜单中单击“新建”，然后在级联菜单中单击“项目”  。

   > **注意**：现在，我们将创建一个共享程序集，该程序集将以 NuGet 包的形式发布，以便其他团队可以集成该程序集并保持同步，而不必直接使用项目源。

1. 在“新建项目”窗格的“最新项目模板”页面上，使用搜索文本框找到“类库”模板，选择 C# 模板，然后单击“下一步”   。
1. 在“新建项目”窗格的“类库”页面上，指定以下设置，然后单击“创建”：

   | 设置       | 值                    |
   | ------------- | ------------------------ |
   | 项目名称  | eShopOnWeb.Shared****    |
   | 位置      | 接受默认值 |
   | 解决方案      | **创建新解决方案**  |
   | 解决方案名称 | eShopOnWeb.Shared****    |

   选中复选框，以便在同一目录中放置解决方案和项目。****

1. 单击 “下一步” 。 接受 .NET 8 作为框架选项****。
1. 按“创建”按钮确认项目创建。
1. 在 Visual Studio 界面的“解决方案资源管理器”窗格中，右键单击 Class1.cs，在右键菜单中选择“删除”，并在出现确认提示时单击“确定”   。
1. 按 Ctrl+Shift+B 或右键单击 EShopOnWeb.Shared 项目，然后选择“生成”以生成项目。
1. 在实验室工作站中，打开“开始”菜单，然后搜索“Windows PowerShell”。 接下来，在级联菜单中，单击“以管理员身份打开 Windows PowerShell”。
1. 在“管理员:**** Windows PowerShell”窗口中，通过执行以下命令导航到 eShopOnWeb.Shared 文件夹：

   ```powershell
   cd c:\eShopOnWeb\eShopOnWeb.Shared
   ```

   > **注意**：eShopOnWeb.Shared 文件夹是 eShopOnWeb.Shared.csproj 文件的位置。******** 如果选择了其他位置或项目名称，请改为导航到该位置。

1. 运行以下命令，从项目创建 **.nupkg** 文件（使用唯一字符串更改`XXXXXX`占位符的值）。

   ```powershell
   dotnet pack .\eShopOnWeb.Shared.csproj -p:PackageId=eShopOnWeb-XXXXX.Shared
   ```

   > **注意**：dotnet pack 命令会生成项目，并在 bin\Release 文件夹中创建 NuGet 包。******** 如果没有 **Release** 文件夹，则可以改用 **Debug** 文件夹。

   > **注意**：忽略“管理员: Windows PowerShell”窗口中显示的任何警告。

   > 注意：dotnet 包会根据其从项目中识别的信息生成一个最小的包。**** 该参数`-p:PackageId=eShopOnWeb-XXXXXX.Shared`允许使用项目中包含的名称创建具有特定名称的包。 例如，如果将字符串`12345`替换为`XXXXXX`占位符，则包的名称将为 **eShopOnWeb-12345.Shared.1.0.0.nupkg**。 该版本号是从程序集中检索的。

1. 在 PowerShell 窗口中，运行以下命令以打开 bin\Release 文件夹：****

   ```powershell
   cd .\bin\Release
   ```

1. 运行以下命令，将该包发布到 **eShopOnWebShared** 源。 将源替换为之前从 Visual Studio **源** URL `https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json` 复制的 URL 

   ```powershell
   dotnet nuget push --source "https://pkgs.dev.azure.com/Azure-DevOps-Org-Name/_packaging/eShopOnWebShared/nuget/v3/index.json" --api-key az "eShopOnWeb.Shared.1.0.0.nupkg"
   ```

   > **重要说明**：需要为操作系统安装凭据提供程序才能使用 Azure DevOps 进行身份验证。 可以在 [Azure Artifacts 凭据提供程序](https://go.microsoft.com/fwlink/?linkid=2099625)中找到安装说明。 可以通过在 PowerShell 窗口中运行以下命令进行安装：`iex "& { $(irm https://aka.ms/install-artifacts-credprovider.ps1) } -AddNetfx"`

   > **注意**：你需要提供一个 API 密钥，该密钥可以是任何非空字符串。 我们在这里使用 az。**** 当系统出现提示时，登录到 Azure DevOps 组织。

   > **注意**：如果提示未出现，或者收到警告 **：“警告：插件凭据提供程序无法获取凭据。身份验证可能需要手动操作。请考虑使用 --interactive 重新运行命令（对于 `dotnet`），或者在 MSBuild 中使用 /p:NuGetInteractive=true，或者为 NuGet 移除 -NonInteractive 开关**，您可以在命令中添加 **--interactive** 参数。

1. 等待确认包推送操作成功。
1. 切换到显示 Azure DevOps 门户的 Web 浏览器窗口，然后在垂直导航窗格中选择 Artifacts。
1. 在“项目”中心窗格上，单击左上角的下拉列表，然后在源列表中选择“eShopOnWebShared”条目。********

   > **注意**：eShopOnWebShared 源应该包括新发布的 NuGet 包。****

1. 单击 NuGet 包以显示其详细信息。

#### 任务 3：将开源的 NuGet 包导入 Azure DevOps 包源

除了开发自己的包之外，是不是也可以使用开源 NuGet (<https://www.nuget.org>) DotNet 包库呢？ 由于有几百万个包可用，总会有一些对你的应用程序有用的包。

在此任务中，我们将使用通用的“Newtonsoft.Json”示例包，但你可以对库中的其他包使用相同的方法。

1. 在上个任务用于推送新包的同一 PowerShell 窗口中，导航回 **eShopOnWeb.Shared** 文件夹 (`cd..`)，运行以下 **dotnet** 命令以安装示例包：

   ```powershell
   dotnet add package Newtonsoft.Json
   ```

1. 检查安装过程的输出。 它会显示将尝试下载的包的不同源：

   ```powershell
   Feeds used:
     https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
     https://pkgs.dev.azure.com/<AZURE_DEVOPS_ORGANIZATION>/eShopOnWeb/_packaging/eShopOnWebShared/nuget/v3/index.json
   ```

1. 接下来，它将显示有关实际安装过程本身的其他输出。

   ```powershell
   Determining projects to restore...
   Writing C:\Users\AppData\Local\Temp\tmpxnq5ql.tmp
   info : X.509 certificate chain validation will use the default trust store selected by .NET for code signing.
   info : X.509 certificate chain validation will use the default trust store selected by .NET for timestamping.
   info : Adding PackageReference for package 'Newtonsoft.Json' into project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info :   GET https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json
   info :   OK https://api.nuget.org/v3/registration5-gz-semver2/newtonsoft.json/index.json 124ms
   info : Restoring packages for c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj...
   info :   GET https://api.nuget.org/v3/vulnerabilities/index.json
   info :   OK https://api.nuget.org/v3/vulnerabilities/index.json 84ms
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json
   info :   GET https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/vulnerability.base.json 14ms
   info :   OK https://api.nuget.org/v3-vulnerabilities/2024.02.15.23.23.24/2024.02.17.11.23.35/vulnerability.update.json 30ms
   info : Package 'Newtonsoft.Json' is compatible with all the specified frameworks in project 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : PackageReference for package 'Newtonsoft.Json' version '13.0.3' added to file 'c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj'.
   info : Writing assets file to disk. Path: c:\eShopOnWeb\eShopOnWeb.Shared\obj\project.assets.json
   log  : Restored c:\eShopOnWeb\eShopOnWeb.Shared\eShopOnWeb.Shared.csproj (in 294 ms).
   ```

1. Newtonsoft.Json 包作为 Newtonsoft.Json 安装在包中。**** 在 Visual Studio 解决方案资源管理器中，导航到 eShopOnWeb.Shared 项目，展开“依赖项”，并注意“包”下的 Newtonsoft.Json。************ 单击“包”左侧的小箭头，打开文件夹和文件列表。

创建 Azure DevOps Artifacts 包源后，根据设计，它允许使用**上游源**，例如 dotnet 示例中的 nuget.org。该示例从托管包的位置引入 Newtonsoft.Json 包。 这是避免包重复并确保始终使用最新版本的常见做法。

1. 在 Azure DevOps 门户中，刷新“项目包源”页。 包列表会同时显示 eShopOnWeb.Shared 自定义开发的包和 Newtonsoft.Json 公共源代码包。********
1. 在 Visual Studio 的 eShopOnWeb.Shared 解决方案中，右键单击“eShopOnWeb.Shared”项目，然后从关联菜单中选择“管理 NuGet 包”。************
1. 在“NuGet 包管理器”窗口中，验证“包源”是否已设置为“eShopOnWebShared”。********
1. 单击“浏览”，并等待 NuGet 包列表加载完成。
1. 此列表还会同时显示 eShopOnWeb.Shared 自定义开发的包，以及 Newtonsoft.Json 公共源代码包。********

## 审阅

在本实验室中，你通过以下步骤了解了如何使用 Azure Artifacts：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入自定义开发的 NuGet 包。
