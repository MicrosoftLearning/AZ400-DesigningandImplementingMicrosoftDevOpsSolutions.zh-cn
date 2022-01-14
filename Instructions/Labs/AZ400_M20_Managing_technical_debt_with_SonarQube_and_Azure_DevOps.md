---
lab:
    title: '实验室 20： 使用 SonarCloud 和 Azure DevOps 管理技术债务'
    module: '模块 20：验证代码库是否合规'
---

# 实验室 20： 使用 SonarCloud 和 Azure DevOps 管理技术债务
# 学生实验室手册

## 实验室概述

在 Azure DevOps 的上下文中，术语 *“技术债务”* 表示达成战术目标的次优手段，这会对在软件开发和部署领域中达到战略目标的能力产生负面影响。技术债务会导致代码难以理解、容易出错、更改耗时且难以验证，因而影响工作效率。如果不进行适当的监督和管理，技术债务会不断累积，从长远来看会严重影响软件的整体质量和开发团队的工作效率。

[SonarCloud](https://sonarcloud.io/){:target="\_blank"} 是基于云的代码质量和安全服务。SonarCloud 的主要功能包括：

- 支持 23 种编程和脚本语言，包括 Java、JS、C#、C/C++、Objective-C、TypeScript、Python、ABAP、PLSQL 和 T-SQL。
- 提供成千上万条规则，用于根据强大的静态代码分析器来查找难以发现的 bug 和质量问题。
- 可与常用的 CI 服务（包括 Travis、Azure DevOps、BitBucket 和 AppVeyor）进行基于云的集成。
- 可进行深入的代码分析，以了解分支中的所有源文件和拉取请求，并帮助实现绿色 Quality Gate 以及促进生成。
- 加速且可缩放。

在本实验室中，你将了解如何将 Azure DevOps Services 与 SonarCloud 集成。

> **备注**：运行此实验室之前，请确保能够运行 Azure DevOps 管道。由于 2021 年 2 月对公共项目进行了更改，因此需要请求才能访问管道：https://devblogs.microsoft.com/devops/change-in-azure-pipelines-grant-for-public-projects/

## 目标

完成本实验室后，你将能够：

- 设置 Azure DevOps 项目和 CI 生成从而与 SonarCloud 集成
- 分析 SonarCloud 报表
- 将静态分析集成到 Azure DevOps 拉取请求过程中

## 实验室持续时间

-   预计用时：**60 分钟**

## 说明

### 准备工作

#### 登录到实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### 设置 Azure DevOps 组织。 

如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/zh-cn/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

### 练习 0：配置实验室先决条件

在此练习中，你将设置实验室先决条件，其中包括基于 [Sonar 扫描示例存储库](https://github.com/SonarSource/sonar-scanning-examples.git)的团队项目。

#### 任务 1：创建团队项目

在此任务中，你将基于 [Sonar 扫描示例存储库](https://github.com/SonarSource/sonar-scanning-examples.git)创建新的 Azure DevOps 项目。

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure DevOps 门户**](https://dev.azure.com)，然后登录到 Azure DevOps 组织。
1.  单击 **Azure DevOps 门户**右上角的 **“+ 新建项目”**。 
1.  在 **“新建项目”** 窗格上的 **“项目名称”** 文本框中，键入 **“SonarExamples”**，在 **“可见性”** 部分，依次单击 **“公共”** 和 **“创建”**。 
   
    > **备注**：除非要通过 SonarCloud 注册付费计划，否则请确保将 Azure DevOps 项目设置为公共。如果要注册付费计划，则可以创建专用项目。

1.  转到 **“SonarExamples”** 窗格，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击 **“Repos”**，在 **“SonarExamples 为空。请添加一些代码!”** 窗格的 **“导入存储库”** 部分，单击 **“导入”**。
1.  在 **“导入 Git 存储库”** 窗格上，确保 **Git** 在 **“存储库类型”** 下拉列表中显示，在 **“克隆 URL”** 中，键入 **“https://github.com/SonarSource/sonar-scanning-examples.git”** 然后单击 **“导入”**。 

    > **备注**： 扫描示例存储库包含大量生成系统和语言的示例项目，包括 C# 与 MSBuild 中，以及使用 Java 的 Maven 和 Gradle。

#### 任务 2：生成 Azure DevOps 个人访问令牌

在此任务中，你将生成 Azure DevOps 个人访问令牌，用于对此练习下一任务中要安装的 Postman 应用进行身份验证。

1.   在实验室计算机上显示 Azure DevOps 门户的 Web 浏览器窗口中，单击 Azure DevOps 页面的右上角的 **“用户设置”** 图标，在下拉菜单中单击 **“个人访问令牌”**，在 **“个人访问令牌”** 窗格中，单击 **“+ 新建令牌”**。
1.   在 **“新建个人访问令牌”** 窗格上，单击 **“显示所有范围”** 链接，指定以下设置，然后单击 **“创建”** （将其他设置全部保留为默认值）：

     | 设置 | 值 |
     | --- | --- |
     | 名称 | **使用 SonarCloud 和 Azure DevOps 管理技术债务的实验室** |
     | 范围 | **自定义** |
     | 范围 | **代码** |
     | 权限 | **读写** |

1.   在 **“成功”** 窗格上，将个人访问令牌的值复制到剪贴板。

     > **备注**： 确保记下令牌的值。关闭此窗格后，将无法再检索它。 

1.   在 **“成功”** 窗格中，单击 **“关闭”**。

#### 任务 3：安装和配置 SonarCloud Azure DevOps 扩展

在此任务中，你将在 Azure DevOps 项目中安装和配置 SonarCloud Azure DevOps 扩展。

1.  在实验室计算机上，启动 Web 浏览器，导航到 Visual Studio 市场中的 [SonarCloud 扩展页](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud)，单击 **“免费获取”**，确保 Azure DevOps 组织的名称出现在 **“选择 Azure Devops 组织”** 下拉列表中，然后单击 **“安装”**。
1.  安装完成后，单击 **“前往组织”**。这会将浏览器重定向到显示了组织主页的 Azure DevOps 门户。 

    > **备注**： 如果你没有从市场中安装扩展的相应权限，系统会向帐户管理员发送请求，以请求其批准安装。

    > **备注**： SonarCloud 扩展包含生成任务、生成模板和自定义仪表板小组件。

1.  在 Web 浏览器窗口中，导航到 **SonarCloud 主页** [https://sonarcloud.io/](https://sonarcloud.io/)。
1.  在 SonarCloud 主页面上，单击 **“登录”**。
1.  在 **“登录或注册 SonarCloud”** 上，单击 **“使用 Azure DevOps”**。 
1. 系统提示“**是否允许此应用访问你的信息**”时，请单击“**是**”。如果出现提示，请选择“**代表组织同意**”和“**接受**”。

    > **备注**： 在 SonarCloud 中，你将创建组织，并在其中新建项目。在 SonarCloud 中设置的组织和项目将与在 Azure DevOps 中设置的组织和项目一样。

1.  在“**欢迎使用 SonarCloud**”页面上，单击“**从 Azure 导入组织**”。
1.  在 **“创建组织”** 页面上的 **“Azure DevOps 组织名称”** 文本框中，键入你的 Azure DevOps 组织的名称，在 **“个人访问令牌”** 文本框中，粘贴在上一练习中记下的令牌的值，然后单击 **“继续”**。 
1.  在“**导入组织详细信息**”部分的“**密钥**”文本框中，键入用于指定组织的字符串，然后单击“**继续**”。

    > **备注**： 此密钥在 SonarCloud 系统中必须唯一。确保 **“密钥”** 文本框的右侧显示了绿色对勾。这表示密钥符合唯一性前提条件。

1.  在 **“选择计划”** 部分，选择要在本实验中使用的计划，然后单击 **“创建组织”**。

    > **备注**： 现在，你已创建了与 Azure DevOps 组织完全相同的 SonarCloud 组织。

    > **备注**： 接下来，在新创建的组织中，你将创建 SonarCloud 项目，该项目与 Azure DevOps 项目 **SonarExamples** 完全相同。 

1.  在 **“全部设置完毕! 组织现已准备就绪”** 页面上，单击 **“分析新项目”**。
1.  在 **“分析项目 - 选择存储库”** 页的 Azure DevOps Projects 列表中，选中 **“SonarExamples/SonarExamples”** 条目旁的复选框，然后单击 **“设置”**。
1.  在 **“分析项目”** 页面上，单击 **“使用 Azure DevOps 管道”** 磁贴。 
1.  在 **“使用 Azure Pipelines 分析”** 页的 **“安装扩展”** 部分，单击 **“继续”**。

    > **备注**： 如果已安装扩展，可忽略创建扩展部分。 

1.  在 **“使用 Azure Pipelines 分析”** 页的 **“配置 Azure Pipelines”** 部分，单击 **“NET”**。这将显示 **“准备分析配置”**、 **“运行代码分析”** 和 **“发布 Quality Gate 结果”** 所需执行的一系列步骤。 

    > **备注**： 查看实现这些目标的步骤列表。你将在后续任务中实现它们。

    > **备注**： 记下设置 SonarCloud 服务终结点所需的令牌值以及项**目密钥**和**项目名称的值**。


### 练习 1：设置与 SonarCloud 集成的 Azure DevOps 管道

在本练习中，你将设置与 SonarCloud 集成的 Azure DevOps 管道。

> **备注**： 我们将设置与 SonarCloud 集成的新生成管道，用于分析 **SonarExamples** 代码。 

#### 任务 1：开始创建项目生成管道

在此任务中，你将开始为项目创建生成管道。

1.  切换到在 Azure DevOps 门户中显示 **“SonarExamples”** 窗格的 Web 浏览器窗口，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击 **“管道”**，然后单击 **“创建管道”**。
1.  在 **“你的代码在哪里?”** 窗格上，查看可用选项。

    > **备注**： 你有两种选项：可以使用 **YAML 编辑**器或**经典编辑器**来配置管道。如果使用经典编辑器，则可使用上述 SonarCloud 扩展中安装的预定义模板。如果使用 YAML 编辑器，则需要使用单独提供的 YAML 文件。我们将逐步介绍这两种选项。 

    > **备注**： 如果打算使用经典编辑器，请跳过下一任务。

#### 任务 2： 使用 YAML 编辑器创建管道

在此任务中，你将使用 YAML 编辑器创建管道。

> **备注**： 继续配置 YAML 管道前，你将先为 SonarCloud 创建服务连接。

1.  打开另一个浏览器标签页，导航到 Azure DevOps 门户中的 **SonarExamples** 主页。
1.  在显示 Azure DevOps 门户中的 **“SonarExamples”** 窗格的 Web 浏览器窗口中，在左下角单击 **“项目设置”**。
1.  在 **“项目设置”** 窗格的垂直菜单栏中的 **“管道”** 部分，依次单击 **“服务连接”** 和 **“创建服务连接”**。
1.  在 **“新建服务连接”** 窗格上，选择 **“SonarCloud”** 选项，然后单击 **“下一步”**。
1.  在 **“新建 SonarCloud 服务连接”** 窗格的 **“SonarCloud 令牌”** 文本框中，粘贴在上一任务中记下的令牌值，在 **“服务连接名称”** 文本框中，键入 **“SC”** 并单击 **“验证并保存”**。 
1.  切换回显示了 **“你的代码在哪里?”** 窗格的 Web 浏览器标签页。如果已关闭此标签页，请返回在 Azure DevOps 门户中的 **“SonarExamples”** 窗格，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击 **“管道”**，然后单击 **“创建管道”**。
1.  在 **“你的代码在哪里?”** 窗格上，单击 **“Azure Repos Git”**。
1.  在 **“选择存储库”** 窗格中，单击 **“SonarExamples”**。 
1.  在 **“配置管道”** 窗格上，单击 **“NET 桌面”** YAML 模板。

    > **备注**： 这将自动显示打开了模板 YAML 文件的 YAML 编辑器。若要正确对其进行配置，需对其进行调整（或替换），确保与以下文件匹配：

    ```yaml
    trigger:
    - master

    pool:
      vmImage: 'windows-latest'

    variables:
      buildConfiguration: 'Release'
      buildPlatform: 'any cpu'

    steps:
    - task: NuGetToolInstaller@0
      displayName: 'Use NuGet 4.4.1'
      inputs:
        versionSpec: 4.4.1

    - task: NuGetCommand@2
      displayName: 'NuGet restore'
      inputs:
        restoreSolution: 'SomeConsoleApplication.sln'

    - task: SonarCloudPrepare@1
      displayName: 'Prepare analysis configuration'
      inputs:
        SonarCloud: 'SC'
        organization: 'myorga'
        scannerMode: 'MSBuild'
        projectKey: 'dotnet-framework-on-azdo'
        projectName: 'Sample .NET Framework project with Azure DevOps'


    - task: VSBuild@1
      displayName: 'Build solution **\*.sln'
      inputs:
        solution: 'SomeConsoleApplication.sln'
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: VSTest@2
      displayName: 'VsTest - testAssemblies'
      inputs:
        testAssemblyVer2: |
         **\$(BuildConfiguration)\*Test*.dll
         !**\obj\**
        codeCoverageEnabled: true
        platform: '$(BuildPlatform)'
        configuration: '$(BuildConfiguration)'

    - task: SonarCloudAnalyze@1
      displayName: 'Run SonarCloud analysis'

    - task: SonarCloudPublish@1
      displayName: 'Publish results on build summary'
    ```

    > **备注**： 你可以从 [SonarSource GitHub 存储库](https://github.com/SonarSource/sonar-scanner-vsts/blob/master/yaml-pipeline-templates/net-desktop-sonarcloud.yml)下载 **net-desktop-sonarcloud.yml** 文件。

    > **备注**： 需遵循本任务中的其余步骤来修改 YAML 管道。 

1. 在 **NuGetCommand@2** 任务中，将 `restoreSolution: 'SomeConsoleApplication.sln'` 替换为 `restoreSolution: '**\SomeConsoleApplication.sln'` 以说明解决方案不在存储库的根目录中。
1.  在 **VSBuild@1** 任务中，将 `solution: 'SomeConsoleApplication.sln'` 替换为 `solution: 'SomeConsoleApplication.sln'`，以说明解决方案不在存储库的根目录中。
1.  在 **SonarCloudPrepare@1** 任务中，将 `organization: 'myorga'` 条目中的 `myorga` 占位符的值替换为 SonarCloud 组织的名称。
1.  在 **SonarCloudPrepare@1** 任务中，将 `projectKey: 'dotnet-framework-on-azdo'` 条目中的 `dotnet-framework-on-azdo` 占位符的值替换为 SonarCloud 项目密钥的名称。
1.  在 **SonarCloudPrepare@1** 任务中，将 `projectName: `Sample .NET Framework project with Azure DevOps` 条目中的 `Sample .NET Framework project with Azure DevOps` 占位符的值替换为 SonarCloud 项目 (`SonarExamples`) 的名称。
1.  在 **“查看管道 YAML”** 窗格上，单击 **“保存并运行”**，然后在 **“保存并运行”** 窗格中，单击 **“保存并运行”**。

    > **备注**：如果在 YAML 编辑器中完成了此任务，请跳过下一个任务。

1. 转到 “Azure 管道”>“管道”，单击 **“Sonarexample”** 管道，打开最新的运行。你将看到它在队列中，取消等待中的运行，单击 **“是”**。现在，单击 **“运行新的”** > **“运行”** 以触发一次新的运行（这次管道将为私密项目分配适当的代理）。

#### 任务 3：使用经典编辑器创建管道

在此任务中，你将使用经典编辑器创建管道。

1.  在 **“你的代码在哪里?”** 窗格上，单击 **“使用经典编辑器”**。
1.  在 **“选择源”** 窗格上，确保选中 **“Azure Repos Git”** 选项，**“存储库”** 下拉列表中显示 **“SonarExamples”** 条目，并且 **“手动和计划生成的默认分支”** 中显示主分支，然后单击 **“继续”**。

    > **备注**： 先前安装的 SonarCloud 扩展为 Maven、Gradle、.NET Core 和 .NET 桌面应用程序提供了启用了 SonarCloud 的自定义生成模板。这些模板基于标准 Azure DevOps 模板，但具有其他特定于分析的任务和一些预配置的设置。

1.  在 **“选择模板”** 窗格上，向下滚动到 **“其他”** 部分，然后依次单击 **“具有 SonarCloud 的 NET 桌面”** 模板条目和 **“应用”**。

    > **备注**： 该模板包含所有必需的任务和大多数必需的设置。你需要提供其余设置。

1.  在生成管道定义的 **“任务”** 选项卡上，确保选中 **“管道”** 条目，在右侧的 **“代理池”** 下拉列表中，选择 **“Azure Pipelines”** 条目，然后在 **“代理规范”** 下拉列表中，选择 **“vs2017-win2016”** 条目。
1.  在管道任务列表中，选择 **“准备分析 SonarCloud”** 任务，然后单击 **“+ 新建”**。
1.  在 **“新建服务连接”** 窗格的 **“SonarCloud 令牌”** 文本框中，粘贴从本实验室的先前部分中记下的令牌值，单击 **“验证”** 对其进行验证，在 **“服务连接名称”** 文本框中，键入 **“SC”** 并单击 **“验证并保存”**。 
1.  回到 **“准备分析配置”** 窗格中的 **“组织”** 下拉列表中，选择 SonarCloud 组织的名称。 
1.  在 **“准备分析配置”** 窗格的 **“项目密钥”** 文本框中，键入从本实验室先前部分中记下的项目密钥的名称。
1.  在 **“准备分析配置”** 窗格的 **“项目名称”** 文本框中，键入从本实验室的先前部分中记下的项目的名称 (`SonarExamples`)。
1.  可以根据需要禁用“发布 Quality Gate 结果”，在管道任务列表中，选择 **“发布 Quality Gate 结果”** 任务，在 **“发布 Quality Gate 结果”** 窗格中，展开 **“控制选项”** 部分，然后清除 **“已启用”** 复选框。 

    > **备注**： 如果要使用预部署入口和发布管道，则必须执行本任务。

    > **备注**： 如果已启用本步骤，则 **“生成摘要”** 页的 **“扩展”** 选项卡中将显示分析结果的摘要。但这将导致 SonarCloud 上的处理完成后才能完成生成。

1.  在生成管道编辑器窗格上，单击 **“保存并排队”**，在下拉菜单中，单击 **“保存并排队”**，然后在 **“运行管道”** 窗格上，单击 **“保存并运行”**，然后等待生成完成。

#### 任务 4：检查管道结果

在此任务中，你将检查管道结果。

1.  在生成运行窗格（之前创建的 YAML 或经典窗格）上，查看 **“摘要”** 选项卡的内容，然后单击 **“扩展”** 选项卡标头。

    > **备注**： 如果启用了 **“发布 Quality Gate 结果”** 任务，则 **“扩展”** 选项卡包含 SonarCloud 分析报表的摘要。

1.  在 **“扩展”** 选项卡上，单击 **“详细 SonarCloud 报表”**。这将自动打开新的浏览器标签页，其中显示了 SonarCloud 项目报表页。

    > **备注**： 另外，你可以浏览 SonarCloud 项目。 

1.  验证报表是否不包含 Quality Gate 结果，如果不包含，则说明原因。

    > **备注**： 为了能够看到 Quality Gate 结果，在运行第一个报表后，我们需要设置 **“新建代码定义”**。这样，后续的管道运行将包含 Quality Gate 结果。

1.  在 SonarCloud 项目的 **“概述”** 选项卡上，单击 **“设置新建代码定义”**。 
1.  在 SonarCloud 项目的 **“管理”** 选项卡上，单击 **“上一版本”**。
1.  切换到 Web 浏览器窗口，该窗口显示了 Azure DevOps 门户中的 **SonarExamples** 项目窗格和最新的生成运行，然后单击 **“运行新建”**，在 **“运行管道”** 窗格中，单击 **“运行”**。
1.  在生成运行窗格上，查看 **“摘要”** 选项卡的内容，然后单击 **“扩展”** 选项卡标头。
1.  在 **“扩展”** 选项卡上，单击 **“详细 SonarCloud 报表”**。这将自动打开新的浏览器标签页，其中显示了 SonarCloud 项目报表页。
1.  验证报表现在是否包含 Quality Gate 结果。

    > **备注**： 我们现已在 SonarCloud 上创建了新组织，并配置了 Azure DevOps 生成，用于执行分析和将生成结果推送到 SonarCloud。

### 练习 2： 分析 SonarCloud 报表

在此练习中，你将分析 SonarCloud 报表。

#### 任务 1：分析 SonarCloud 报表

在本任务中，你将分析 SonarCloud 报表。

1.  在 SonarCloud 项目的 **“概述”** 选项卡的 **“可靠性措施”** 部分，你会发现该处有一个 bug 条目。

    > **备注**： 该页面还包含其他指标，例如 **“代码异味”**、 **“覆盖率”**、 **“复制”** 和 **“大小”** （代码行）。下表简要说明了每个术语。

    | 术语 | 描述 |
    | --- | --- |
    | **Bug** | 表示代码错误的问题。如果尚未发生该问题，则该问题将会并且可能会在最糟糕的时刻发生。需修复该问题 |
    | **漏洞** | 与安全性相关的问题，该问题对于攻击者而言是潜在后门 |
    | **代码异味** | 代码中与可维护性相关的问题。如果不修复该问题，则意味着即使做最乐观的估计，维护人员进行后续更改时也会更艰难。而在最坏的情况下，维护人员会因代码的状态困惑，导致进行更改时引入更多错误 |
    | **覆盖率** | 表示单元测试等测试所验证代码的百分比。为有效防止 bug，应进行这些测试，并覆盖大部分代码 |
    | **复制** | 复制图标显示复制了源代码的哪些部分 |
    | **大小** | 给出项目中的代码行计数，包括语句、函数、类、文件和目录的数量 |

    > **备注**： Bug 计数旁显示的字母表示**可靠性等级**。特别是，字母 **C** 表示此代码中至少存在 1 个重大 bug。有关可靠性等级的详细信息，请参阅 [SonarQube 文档](https://docs.sonarqube.org/display/SONAR/Metric+Definitions#MetricDefinitions-Reliability)。还可以在其中找到有关[规则类型](https://docs.sonarqube.org/latest/user-guide/rules/)的详细信息和查看[严重性](https://docs.sonarqube.org/latest/user-guide/issues/)。

1.  单击表示 **bug** 计数的数字。这会自动显示 **“问题”** 选项卡的内容。 
1.  在 **“问题”** 选项卡的右侧，单击表示 bug 的大矩形，显示相应的代码。 

    > **备注**： 查看 **Program.cs** 文件的第 9 行中的错误详细信息，其中包含的建议说明了以下内容： **更改此条件，确保其不总是评估为“true”；某些后续代码永不执行。**

1.  将鼠标悬停在代码和行号之间的垂直线上，了解代码覆盖率的缺额。

    > **备注**： 示例项目非常小，没有历史数据。但数千个 [SonarCloud 公共项目](https://sonarcloud.io/explore/projects)具有更值得研究且更实际的结果。

### 练习 3：实现与 SonarCloud 的 Azure DevOps 拉取请求集成

在此练习中，你将在 Azure DevOps 和 SonarCloud 之间设置拉取请求集成。

> **备注**： 为配置 SonarCloud 分析来分析 Azure DevOps 拉取请求中包含的代码，需要执行以下任务：

- 将 Azure DevOps 个人访问令牌添加到 SonarCloud 项目，该令牌可授权其对拉取请求的访问权限。
- 配置 Azure DevOps 分支策略，该策略控制拉动请求触发的生成

#### 任务 1：创建 Azure DevOps 个人访问令牌，以与 SonarCloud 进行拉取请求集成

在此任务中，你将查看个人访问令牌要求，以实现与 SonarCloud 项目的 Azure DevOps 拉取请求集成。

1.  切换到 Web 浏览器窗口，其中显示了 Azure DevOps 门户中的 **SonarExamples** 项目。
1.  再次执行本实验室先前部分所述的步骤，生成具有 **“代码”** 范围和 **SonarExamples** 项目**读写**权限的个人访问令牌。 

    > **备注**： 另外，你可以再次使用在本实验室先前部分中生成的个人访问令牌。

    > **备注**： SonarCloud 中对拉取请求的注释将添加到创建了个人访问令牌的用户的安全性上下文。建议为此创建单独的“机器人”Azure DevOps 用户，以清楚地标识源自 SonarCloud 的注释。

#### 任务 2： 配置 SonarCloud 中的拉取请求集成

在此任务中，你将通过为 SonarCloud 项目分配 Azure DevOps 个人访问令牌来配置 SonarCloud 中的拉取请求集成。

1.  切换到 Web 浏览器窗口，其中显示了 SonarCloud 门户中的 **SonarExamples** 项目。 
1.  在项目的仪表板页面上，单击 **“管理”** 选项卡的标头，然后在下拉菜单中，单击 **“常规设置”**。
1.  在 **“常规设置”** 页面，单击 **“拉取请求”**。
1.  在 **“拉取请求”** 设置的 **“常规”** 部分的 **“提供程序”** 下拉列表中，选择 **“Azure DevOps Services”**，然后单击 **“保存”**。
1.  在 **“拉取请求”** 设置的 **“与 Azure DevOps Services 集成”** 部分的 **“个人访问令牌”** 文本框中，粘贴先前生成的 Azure DevOps 个人访问令牌，然后单击 **“保存”**

#### 任务 3：为与 SonarCloud 集成配置分支策略

在此任务中，你将为与 SonarCloud 集成配置 Azure DevOps 分支策略。

1.  切换到 Web 浏览器窗口，其中显示了 Azure DevOps 门户中的 **SonarExamples** 项目。
1.  在 Azure DevOps 门户最左侧的垂直菜单栏中，单击 **“Repos”**，然后在 **“存储库”** 部分，单击 **“分支”**。 
1.  在“**分支**”窗格的分支列表中，将鼠标悬停在**主**分支条目的右边缘上，这样会显示表示“**更多选项**”菜单的垂直省略号字符，单击省略号，然后在弹出的菜单中，单击“**分支策略**”。
1.  在**主**窗格上，单击 **“生成验证”** 部分右侧的 **“+”**。
1.  在“**添加生成策略**”窗格的“**生成管道**”下拉列表中，选择在此实验室先前部分中创建的管道，然后在“**显示名称**”文本框中，键入“**SonarCloud 分析**”并单击“**保存**”。

    > **备注**： Azure DevOps 现已配置为创建针对**主**分支的任意拉取请求时会触发 SonarCloud 分析。

#### 任务 4：验证拉取请求集成

在此任务中，你将通过创建拉取请求和查看生成的结果来验证 Azure DevOps 和 SonarCloud 之间的拉取请求集成。

> **备注**： 你将更改存储库中的文件，并创建用于触发 SonarCloud 分析的请求。

1.  在 Azure DevOps 门户左侧的垂直菜单栏中，单击 **“Repos”**。这将显示 **“文件”** 窗格。 
1.  在中央窗格中的文件夹层次结构中，导航到 **sonarqube-scanner-msbuild\\CSharpProject\\SomeConsoleApplication** 文件夹中的 **Program.cs** 文件，然后单击“**编辑**”。
1.  在 **“Program.cs”** 窗格上，将以下空方法添加到 `public static bool AlwaysReturnsTrue()` 行正上方的代码中。 

    ```csharp
    public void Unused(){

    }
    ```

1.  在 **“Program.cs”** 窗格中，单击 **“提交”**。
1.  在 **“提交”** 窗格上的 **“分支名称”** 文本框中，键入 **“branch1”**，选中 **“创建拉取请求”** 复选框，然后单击 **“提交”**。
1.  在“**新建拉取请求**”窗格上，选择“**创建**”。 
1.  在 **“更新的 Program.cs”** 窗格的 **“概述”** 选项卡上，监视生成流程的进度，直至完成。 
1.  在 **“更新的 Program.cs”** 窗格的 **“概述”** 选项卡上，请注意，虽然成功完成了必需的检查，但非必需检查失败，然后单击 **“查看 3 项检查”** 链接。
1.  在 **“检查”** 窗格上，查看 SonarCloud 检查的结果，然后关闭窗格。

    > **备注**： 结果表明，分析生成已成功完成，但 PR 中的新代码未能通过代码质量检查。发现的新问题的注释已推送到 PR。

1.  回到 **“更新的 Program.cs”** 窗格的 **“概述”** 选项卡，向下滚动到包含注释的部分，并查看 SonarCloud 中关于新添加的类的注释。 

    > **备注**： 报告的问题仅包含与拉取请求相对应的代码更改。忽略了 **Program.cs** 和其他文件中之前就已存在的问题。

#### 任务 4：阻止对未通过代码质量检查进行响应的拉取请求

在此任务中，你将配置如何阻止对未通过代码质量检查进行响应的拉取请求。 

> **备注**： 此时，即使未通过代码质量检查，仍可完成提取请求并提交相应的更改。除非通过相关的代码质量检查，否则你需修改 Azure DevOps 配置，以阻止提交。

1.  在显示 **“更新的 Programs.cs”** 窗格的 Azure DevOps 门户中，单击左下角的 **“项目设置”**。
1.  在 **“项目设置”** 垂直菜单的 **“Repos”** 部分，单击 **“存储库”**。
1.  在 **“所有存储库”** 窗格中，单击 **“SonarExamples”**。
1.  在 **“SonarExamples”** 窗格中，单击 **“策略”** 选项卡标头。
1.  在 **“策略”** 列表上，向下滚动到分支列表，然后单击表示**主**分支的条目。
1.  在**主**窗格上，向下滚动至 **“状态检查”** 部分并单击 **“+”**。
1.  在 **“添加状态策略”** 窗格的 **“待检查的状态”** 下拉列表中，选择 **“SonarCloud/quality gate”** 条目，确保将 **“策略要求”** 选项设置为 **“必需”**，然后单击 **“保存”**

    > **备注**： 此时，用户只有在代码质量检查成功后才能合并拉取请求。因此，需要在相应的 SonarCloud 项目中修复 SonarCloud 识别的所有问题，或将其标记为 **“已确认”** 或 **“已解决”**。

## 回顾

在本实验室中，你了解了如何将 Azure DevOps Services 与 SonarCloud 集成。
