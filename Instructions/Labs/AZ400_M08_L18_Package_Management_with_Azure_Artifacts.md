---
lab:
  title: 使用 Azure Artifacts 进行包管理
  module: 'Module 08: Design and implement a dependency management strategy'
---

# <a name="package-management-with-azure-artifacts"></a>使用 Azure Artifacts 进行包管理

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 可从 [Visual Studio 下载页面](https://visualstudio.microsoft.com/downloads/)获得 Visual Studio 2022 Community Edition。 Visual Studio 2022 安装应包括“ASP<nolink>.NET 和 Web 开发”、“Azure 开发”和“NET Core 跨平台开发”工作负载。  

## <a name="lab-overview"></a>实验室概述

Azure Artifacts 有助于在 Azure DevOps 中发现、安装和发布 NuGet、npm 和 Maven 包。 它与其他 Azure DevOps 功能（例如生成）深度集成，使包管理成为现有工作流的无缝组成部分。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。

## <a name="estimated-timing-40-minutes"></a>预计用时：40 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板和 Visual Studio 配置预配置的 Parts Unlimited 团队项目。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器，基于 PartsUnlimited 模板生成一个新项目。

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 <https://docs.microsoft.com/en-us/azure/devops/demo-gen> 。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“新建项目”页面上的页面上的“新建项目名称”文本框中，键入“使用 Azure Artifacts 进行包管理”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1. 在“选择模板”页面上的模板列表中，单击 PartsUnlimited 模板，再单击“选择模板”  。
1. 返回“新建项目”页面，单击“创建项目” 

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-configuring-the-parts-unlimited-solution-in-visual-studio"></a>任务 2：在 Visual Studio 中配置 Parts Unlimited 解决方案

在此任务中，你将配置 Visual Studio，以便为实验室做准备。

1. 确保你正在 Azure DevOps 门户上查看“使用 Azure Artifacts 进行包管理”团队项目。

    > **注意**：可通过导航到 [https://dev.azure.com/`<your-Azure-DevOps-account-name>` /Package%20Management%20with%20Azure%20Artifacts](https://dev.azure.com/`<your-Azure-DevOps-account-name>`/Package%20Management%20with%20Azure%20Artifacts) URL 直接访问项目页，其中 `<your-Azure-DevOps-account-name>` 占位符表示帐户名称。

1. 在“使用 Azure Artifacts 进行包管理”窗格左侧的垂直菜单中，单击 Repos 。
1. 在“文件”窗格中，单击“克隆”，选择“在文件中克隆”旁边的下拉 VS Code，在下拉菜单中，选择“Visual Studio”   。
1. 如果系统提示你是否继续，请单击“打开”。
1. 如果系统出现提示，请使用用于设置 Azure DevOps 组织的用户帐户登录。
1. 在 Visual Studio 界面的 Azure DevOps 弹出窗口中，接受默认的本地路径，然后单击“克隆” 。 这会自动将项目导入 Visual Studio，并打开一个新的 Web 浏览器选项卡，其中显示“迁移报表”页面。

    > **注意**：在“查看项目和解决方案更改”对话框中，查看有关不受支持的项目类型的警告，然后单击“确定” 。

1. 关闭显示“迁移报表”页面的 Web 浏览器选项卡。
1. 将 Visual Studio 窗口保持打开状态，以便在实验室中使用。

### <a name="exercise-1-working-with-azure-artifacts"></a>练习 1：使用 Azure Artifacts

在本练习中，你将通过以下步骤了解如何使用 Azure Artifacts：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。

#### <a name="task-1-creating-and-connecting-to-a-feed"></a>任务 1：创建并连接到源

在此任务中，你将创建并连接到源。

1. 在显示 Azure DevOps 门户中项目设置的 Web 浏览器窗口中，在垂直导航窗格中，选择 Artifacts。
1. 显示 Artifacts 中心时，单击窗格顶部的“+ 创建源” 。

    > **注意**：此源将是可供组织内用户使用的 NuGet 包的集合，并且将作为对等方与公共 NuGet 源并置。 本实验室中的方案将重点放在使用 Azure Artifacts 的工作流上，因此实际的体系结构和开发决策纯粹是说明性的。  此源将包含可以在该组织中的各个项目之间共享的通用功能。

1. 在“创建新源”窗格的“名称”文本框中，键入 PartsUnlimitedShared，在“范围”部分，选择“组织”选项，将其他设置保留为默认值，然后单击“创建”     。

    > **注意**：任何想要连接到此 NuGet 源的用户都必须配置其环境。

1. 返回 Artifacts 中心，单击“连接到源” 。
1. 在“连接到源”窗格的 NuGet 部分，选择 Visual Studio，然后在 Visual Studio 窗格中复制“源”URL    。
1. 切换回 Visual Studio 窗口。
1. 在 Visual Studio 窗口中，单击“工具”菜单标题，在下拉菜单中，选择“NuGet 包管理器”，然后在级联菜单中选择“包管理器设置”  。
1. 在“选项”对话框中，单击“包源”，然后单击加号以添加新的包源 。
1. 在对话框底部的“名称”文本框中，将“包源”替换为 PartsUnlimitedShared ，然后在“源”文本框中，粘贴在 Azure DevOps 门户中复制的 URL   。
1. 单击“更新”，然后单击“确定”以完成添加 。

    > **注意**：Visual Studio 现已连接到新源。

1. 关闭并重新打开用于克隆 PartsUnlimited 存储库的其他 Visual Studio 实例，以解释项目源更新并打开 PartsUnlimited 解决方案。 在本练习的第三个任务中需要使用它。

#### <a name="task-2-creating-and-publishing-a-nuget-package"></a>任务 2：创建并发布 NuGet 包

在此任务中，你将创建并发布 NuGet 包。

1. 在用于配置新包源的 Visual Studio 窗口中，在主菜单中单击“文件”，在下拉菜单中单击“新建”，然后在级联菜单中单击“项目”  。

    > **注意**：现在，我们将创建一个共享程序集，该程序集将以 NuGet 包的形式发布，以便其他团队可以集成该程序集并保持同步，而不必直接使用项目源。

1. 在“新建项目”窗格的“最新项目模板”页面上，使用搜索文本框找到“类库 (.NET Framework)”模板，选择 C# 模板，然后单击“下一步”   。
1. 在“新建项目”窗格的“类库(.NET Framework)”页面上，指定以下设置，然后单击“创建”  ：

    | 设置 | 值 |
    | --- | --- |
    | 项目名称 | **PartsUnlimited.Shared** |
    | 位置 | 接受默认值 |
    | 解决方案 | **创建新解决方案** |
    | 解决方案名称 | **PartsUnlimited.Shared** |
    | 框架 | **.NET Framework 4.5.1** |

    > **注意**：确保不选择“.NET Standard”。

1. 在 Visual Studio 界面的“解决方案资源管理器”窗格中，右键单击 Class1.cs，在右键菜单中选择“删除”，并在出现确认提示时单击“确定”   。
1. 在 Visual Studio 界面的“解决方案资源管理器”窗格中，右键单击 PartsUnlimited.Shared 项目节点，然后选择“属性”  。
1. 在 PartsUnlimited.Shared 属性窗格中，确认目标框架设置为 NET Framework 4.5.1  。
1. 按 **Ctrl+Shift+B** 生成项目。

    > **注意**：在下一个任务中，我们将使用 NuGet.exe 直接从生成的项目中生成 NuGet 包，但需要先生成项目。

1. 切换到显示 Azure DevOps 门户的 Web 浏览器。
1. 导航到“连接到源”窗格的 NuGet 部分，然后选择 NuGet.exe  。 这将显示 NuGet.exe 窗格。
1. 在 NuGet.exe 窗格上，单击“获取工具” 。
1. 在“获取工具”窗格上，单击“下载最新的 NuGet”链接 。 这将自动打开另一个浏览器选项卡，其中显示“可用的 NuGet 发行版”页面。
1. 在“可用的 NuGet 发行版”页面上，选择 nuget.exe 版本 v5.5.1，然后将可执行文件下载到本地 Downloads 文件夹  。
1. 切换到 Visual Studio 窗口。 在“解决方案资源管理器”窗格中，右键单击 PartsUnlimited.Shared 项目节点，然后在右键菜单中选择“在文件资源管理器中打开文件夹”  。
1. 在“文件资源管理器”窗口中，将下载的 nuget.exe 文件从 Downloads 文件夹移动到包含 .csproj 文件的文件夹中  。
1. 在同一文件资源管理器窗口中，选择“文件”菜单标题，在下拉菜单中选择“打开 Windows PowerShell”，然后在级联菜单中单击“以管理员身份打开 Windows PowerShell”  。
1. 在“管理员:Windows PowerShell”窗口中，运行以下命令以从项目创建一个 .nupkg 文件。

    > **注意**：这是打包 NuGet 位以进行部署的快捷方式。 NuGet 高度可自定义。 若要了解详细信息，请参阅 [NuGet 包创建页面](https://docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflowhttps:/docs.microsoft.com/en-us/nuget/create-packages/overview-and-workflow)。

    ```
    ./nuget.exe pack ./PartsUnlimited.Shared.csproj
    ```

    > **注意**：忽略“管理员: Windows PowerShell”窗口中显示的任何警告。

    > **注意**：NuGet 根据能够从项目中识别的信息生成一个最小的包。 例如，请注意名称为 PartsUnlimited.Shared.1.0.0.nupkg。 该版本号是从程序集中检索的。

1. 切换回 Visual Studio 窗口，在“解决方案资源管理器”窗格中，展开 PartsUnlimited.Shared\Properties 节点，单击 AssemblyInfo.cs 以在窗口的中心窗格中将其打开，然后查看其内容   。

    > **注意**：AssemblyVersion 属性可指定要生成到程序集中的版本号。 每个 NuGet 版本都需要一个唯一的版本号，因此，如果我们继续使用此方法来创建包，则需要在生成之前递增版本号。

1. 切换到“管理员: Windows PowerShell”窗口，然后运行以下命令将包发布到 PartsUnlimitedShared 源：

    > **注意**：你需要提供一个 API 密钥，该密钥可以是任何非空字符串。 我们在此处使用 AzDO。 当系统出现提示时，登录到 Azure DevOps 组织。

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.0.0.nupkg
    ```

1. 切换到显示 Azure DevOps 门户的 Web 浏览器窗口，然后在垂直导航窗格中选择 Artifacts。
1. 在 Artifacts 中心窗格上，单击左上角的下拉列表，然后在源列表中选择 PartsUnlimitedShared 条目 。

    > **注意**：PartsUnlimitedShared 源应该包括新发布的 NuGet 包。

1. 单击 NuGet 包以显示其详细信息。

#### <a name="task-3-importing-a-nuget-package"></a>任务 3：导入 NuGet 包

在此任务中，你将导入 NuGet 包。

1. 切换到显示 Parts Unlimited 解决方案的 Visual Studio 窗口 。
1. 在“解决方案资源管理器”窗格中，右键单击 PartsUnlimitedWebsite 项目下的“引用”节点，然后在右键菜单中选择“管理 NuGet 包”   。 这将打开窗口中央窗格中的“NuGet: PartsUnlimitedWebsite”选项卡。
1. 在“NuGet:PartsUnlimitedWebsite”窗格中，单击“浏览”选项卡，然后在窗格右上角的“包源”下拉列表中，选择 PartsUnlimitedShared  。

    > **注意**：包列表将仅包含你刚刚添加的一个包。

1. 选择包，然后在 PartsUnlimited.Shared 窗格中，单击“安装”将其添加到项目中 。
1. 当系统出现提示时，在“预览更改”对话框中，单击“确定” 。
1. 按 Ctrl+Shift+B 生成项目，并确认生成成功完成。

    > **注意**：NuGet 包尚未添加任何值，但我们设法验证了工作流是否按预期工作。

#### <a name="task-4-updating-a-nuget-package"></a>任务 4：更新 NuGet 包

在此任务中，你将更新 NuGet 包。

1. 切换到已打开 PartsUnlimited.Shared 项目（包含 NuGet 源项目）的 Visual Studio 窗口 。
1. 在“解决方案资源管理器”窗格中，右键单击 PartsUnlimited.Shared 项目节点，在右键菜单中选择“添加”，然后在级联菜单中选择“新项”   。
1. 在“添加新项 - PartsUnlimitedShared”对话框的“Visual C# 项”列表中，确保已选择“类”模板，在对话框底部的“名称”文本框中，键入 TaxService.cs，然后单击“添加”以添加类     。

    > **注意**：我们假设将税款计算合并到此共享类中，并进行集中管理，以便其他团队可以轻松地使用 NuGet 包。

1. 在中心窗格中，在 TaxService.cs 类的代码中，将该类的现有定义替换为以下代码并保存文件：

    ```c#
    namespace PartsUnlimited.Shared
    {
        public class TaxService
        {
            static public decimal CalculateTax(decimal taxable, string postalCode)
            {
                return taxable * (decimal).1;
            }
        }
    }
    ```

    > **注意**：由于我们要更新程序集（和包），因此需要更新程序集版本。

1. 在 Visual Studio 窗口的中心窗格中，单击 AssemblyInfo.cs 选项卡以显示相应文件的内容。
1. 在 AssemblyInfo.cs 文件中，将 `[assembly: AssemblyVersion("1.0.0.0")]` 更改为 `[assembly: AssemblyVersion("1.1.0.0")]`，并保存该文件。
1. 按 **Ctrl+Shift+B** 生成项目。
1. 切换到“管理员: Windows PowerShell”窗口，然后运行以下命令以重新打包 NuGet 包。

    > **注意**：新包将具有更新后的版本号。

    ```
    ./nuget.exe pack PartsUnlimited.Shared.csproj
    ```

1. 从“管理员: Windows PowerShell”窗口运行以下命令，以发布更新后的包。

    > **注意**：更改发布的项目版本号以反映包版本更新。

    ```
    ./nuget.exe push -source "PartsUnlimitedShared" -ApiKey AzDO PartsUnlimited.Shared.1.1.0.nupkg
    ```

1. 切换到显示 Azure DevOps 门户的 Web 浏览器窗口，其中带有 PartsUnlimitedShared 1.0.0 项目窗格。
1. 在 PartsUnlimitedShared 1.0.0 项目窗格上，单击“版本”选项卡，并验证它是否包含版本 1.0.0 和 1.1.0   。
1. 切换回到显示 PartsUnlimited 项目的 Visual Studio 窗口 。
1. 在“解决方案资源管理器”窗格中，导航到并选择 PartsUnlimitedWebsite\Utils\DefaultShippingTaxCalculator.cs 。 这将自动在窗口的中心窗格中打开文件。
1. 在 DefaultShippingTaxCalculator 文件的代码中，找到第 20 行上对 CalculateTax 的调用，并将 `tax = CalculateTax(subTotal + shipping, postalCode);` 替换为 `tax = PartsUnlimited.Shared.TaxService.CalculateTax(subTotal + shipping, postalCode);`  

    > **注意**：原始代码调用了此类内部的方法，因此我们要添加到行首的代码将其从 NuGet 程序集重定向到代码。 但是，由于该项目尚未更新 NuGet 包，它仍引用 1.0.0.0，并且这些新更改不可用，因此代码无法正确生成。

1. 在“解决方案资源管理器”窗格中，右键单击“引用”节点，然后在右键菜单中，选择“管理 NuGet 包”  。

    > **注意**：NuGet 知道我们的更新，如“更新”选项卡的内容所示。

1. 在“NuGet:PartsUnlimitedWebsite”窗格中，单击“更新”选项卡，在搜索文本框中键入“PartsUnlimited.Shared”，然后在窗格右侧的“版本:   最新稳定版 1.1.0”下拉列表旁边，单击“更新”以安装新版本。

    > **注意**：可能有许多 NuGet 更新可用，但你只需要更新 PartsUnlimited.Shared。 请注意，该包可能需要一些时间才能完全可用于更新。 如果遇到错误，请稍等片刻，然后重试。

1. 当系统出现提示时，在“预览更改”对话框中，单击“确定” 。
1. 按 F5 键生成并运行站点。 验证脚本是否按预期运行。

## <a name="review"></a>审阅

在本实验室中，你通过以下步骤了解了如何使用 Azure Artifacts：

- 创建并连接到源。
- 创建并发布 NuGet 包。
- 导入 NuGet 包。
- 更新 NuGet 包。
