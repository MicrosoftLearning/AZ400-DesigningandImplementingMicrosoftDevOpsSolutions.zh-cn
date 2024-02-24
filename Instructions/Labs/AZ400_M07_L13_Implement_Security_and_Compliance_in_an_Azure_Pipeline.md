---
lab:
  title: 在 Azure DevOps 管道中实现安全性和合规性
  module: 'Module 07: Implement security and validate code bases for compliance'
---

# 在 Azure DevOps 管道中实现安全性和合规性

## 学生实验室手册

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://learn.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)中的说明创建一个。

## 实验室概述

在本实验室中，你将使用 Mend Bolt（以前称为 WhiteSource）自动检测代码中易受攻击的开源组件、已过期库和许可证合规性问题。 你将利用 WebGoat，这是一款有意存在不安全性的网页应用程序，其由 OWASP 维护，用于说明常见的网页应用程序安全问题。

[Mend](https://www.mend.io/) 是持续开源软件安全性和合规性管理的领导者。 无论编程语言、生成工具或开发环境如何，WhiteSource 都会集成到生成流程。 它可在后台自动、持续且以静默方式工作，可在 WhiteSource 不断更新的开源存储库权威数据库中检查开源组件的安全性、许可证和质量。

Mend 提供了 Mend Bolt，这是一种轻量级的开源安全性和管理解决方案，专用于与 Azure DevOps 和 Azure DevOps Server 集成。 请注意，Mend Bolt 适用于所有项目，且不提供实时警报功能（该功能需要完整平台），通常推荐给要在整个软件开发生命周期（从存储库阶段到部署后阶段）以及所有项目和产品中自动执行开源管理的大型开发团队。

Azure DevOps 与 Mend Bolt 的集成将使你能够：

- 检测并修复易受攻击的开源组件。
- 为每个项目或生成过程生成全面的开源清单报告。
- 实施开源许可证合规性，包括依赖项许可证。
- 确定过时的开源库以及更新建议。

## 目标

完成本实验室后，你将能够：

- 激活 Mend Bolt。
- 运行生成管道并查看 Mend 安全性和合规性报告。

## 预计用时：45 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb，并将其他字段保留默认值。 单击“创建”。

    ![创建项目](images/create-project.png)

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“Repos > 文件”、“导入”。 在“导入 Git 存储库”窗口中，粘贴以下 URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> 并单击“导入”：

    ![导入存储库](images/import-repo.png)

1. 存储库按以下方式组织：
    - .ado 文件夹包含 Azure DevOps YAML 管道。
    - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
    - **infra** 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。
    - .github 文件夹容器 YAML GitHub 工作流定义。
    - **src** 文件夹包含用于实验室方案的 .NET 8 网站。

### 练习 1：使用 Mend Bolt 在 Azure DevOps 管道中实现安全性和合规性

在本演练中，利用 Mend Bolt 查找项目代码中的安全漏洞和许可证合规性问题，然后查看生成的报表。

#### 任务 1：激活 Mend Bolt 扩展

在此任务中，你将在新生成的 Azure Devops 项目中激活 WhiteSource Bolt。

1. 在实验室计算机上，在显示已打开 eShopOnWeb 项目的 Azure DevOps 门户的 Web 浏览器窗口中，单击市场图标 >“浏览市场”。

    ![浏览市场](images/browse-marketplace.png)

1. 在市场上，搜索 Mend Bolt（以前称为 WhiteSource）并打开它。 Mend Bolt 是以前已知的 WhiteSource 工具的免费版本，该工具可扫描所有项目并检测开放源代码组件、许可证和已知漏洞。

    > 警告：请确保选择“Mend Bolt”选项（免费版本）！

1. 在“Mend Bolt (以前称为 WhiteSource)”页上，单击“免费获取”。

    ![获取 Mend Bolt](images/mend-bolt.png)

1. 在下一页上，选择所需的 Azure DevOps 组织并安装。 安装后转到组织。

1. 在 Azure DevOps 中，导航到“组织设置”，然后在“扩展”下选择“Mend”。 提供你的工作电子邮件地址（实验室个人帐户，例如，使用 <AZ400learner@outlook.com> 而不是 <student@microsoft.com>）、公司名称和其他详细信息，然后单击“创建帐户”按钮开始使用免费版本。

    ![获取 Mend 帐户](images/mend-account.png)

#### 任务 2：创建并触发生成

在此任务中，你将在 Azure DevOps 项目中创建并触发 CI 生成管道。 你将使用 Mend Bolt 扩展来确定此代码中易受攻击的 OSS 组件。

1. 在实验室计算机上，从 eShopOnWeb Azure DevOps 项目的左侧垂直菜单栏中，导航到“管道 > 管道”部分，单击“创建管道”（或“新建管道）”。

1. 在“你的代码在哪里?”窗口中，选择“Azure Repos Git (YAML)”并选择“eShopOnWeb”存储库。

1. 在“配置”部分，选择“现有 Azure Pipelines YAML 文件”。 提供以下路径 /.ado/eshoponweb-ci-mend.yml，然后单击“继续”。

    ![选择“管道”](images/select-pipeline.png)

1. 查看管道，然后单击“运行”。 成功运行需要几分钟时间。
    > **注意**：此生成可能需要花费几分钟时间完成。 生成定义由以下任务构成：
    - DotnetCLI 任务，用于还原、生成、测试和发布 dotnet 项目。
    - Whitesource 任务（仍保留旧名称），以运行 OSS 库的 Mend 工具分析。
    - 发布项目，运行此管道的代理将上传已发布的 Web 项目。

1. 在执行管道时，请重命名它以便更容易地识别它（因为项目可用于多个实验室）。 转到 Azure DevOps 项目中的“管道/管道”部分，单击正在执行的管道名称（它将获取默认名称），然后在省略号图标上查找“重命名/移动”选项。 将其重命名为 eshoponweb-ci-mend，然后单击“保存”。

    ![重命名管道](images/rename-pipeline.png)

1. 管道执行完成后，可以查看结果。 打开 eshoponweb-ci-mend 管道的最新执行。 “摘要”选项卡将显示执行日志，以及相关详细信息，例如使用的存储库版本（提交）、触发器类型、已发布的项目、测试覆盖率等。

1. 在“Mend Bolt”选项卡上，可以查看 OSS 安全分析。 它将显示有关所用清单的详细信息、发现的漏洞（以及如何解决这些漏洞），以及有关库相关许可证的有趣报告。 花费一些时间查看报告。

    ![Mend 结果](images/mend-results.png)

## 审阅

在本实验室中，你将使用 Mend Bolt 和 Azure DevOps 自动检测代码中易受攻击的开源组件、已过期库和许可证合规性问题。
