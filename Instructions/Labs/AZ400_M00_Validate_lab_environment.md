---
lab:
  title: 验证实验室环境
  module: 'Module 0: Welcome'
---

# 验证实验室环境

## 学生实验室手册

## 创建 Azure DevOps 组织的说明（只需执行此操作一次）

### 如果没有 Azure 订阅，请从此处开始

1. 从讲师或其他来源获取新的“Azure Pass 促销代码”。
1. 使用专用浏览器会话从以下链接获取新的“个人 Microsoft 帐户(MSA)”：[https://account.microsoft.com](https://account.microsoft.com)。
1. 使用相同的浏览器会话，转到 [https://www.microsoftazurepass.com](https://www.microsoftazurepass.com)，然后使用 Microsoft 帐户 (MSA) 兑换 Azure Pass。 有关详细信息，请参阅[兑换 Microsoft Azure Pass](https://www.microsoftazurepass.com/Home/HowTo?Length=5)。 按照说明进行兑换。

### 如果已有 Azure 订阅，请从此处开始

1. 打开浏览器并导航到 [https://portal.azure.com](https://portal.azure.com)，然后在 Azure 门户屏幕顶部搜索“Azure DevOps”。 在结果页面中，单击“Azure DevOps 组织”。
1. 接下来，单击标记为“我的 Azure DevOps 组织”的链接或者直接导航到 [https://aex.dev.azure.com](https://aex.dev.azure.com)。
1. 在“我们需要更多详细信息”页上，选择“继续” 。
1. 在左侧下拉框中，选择“默认目录”，而不是“Microsoft 帐户”。
1. 如果系统提示“我们需要更多详细信息”，请提供你的姓名、电子邮件地址以及位置，然后单击“继续”。
1. 回到 [https://aex.dev.azure.com](https://aex.dev.azure.com) 并选择“默认目录”，单击蓝色按钮“创建新组织” 。
1. 单击“继续”即表示接受“服务条款”。
1. 如果系统提示“即将完成”，请将 Azure DevOps 组织名称保留为默认值（该名称需要是全局唯一名称），并在列表中选择靠近你的托管位置。
1. 在 Azure DevOps 中打开新创建的组织后，单击左下角的“组织设置” 。
1. 在“组织设置”屏幕上单击“计费”（打开此屏幕需要几秒钟时间） 。
1. 单击屏幕右侧的“设置计费”，选择“Azure Pass - 赞助”订阅，单击“保存”以将该订阅与组织链接起来  。
1. 当屏幕顶部显示链接的 Azure 订阅 ID 时，将 MS 托管 CI/CD 的付费并行作业数量从 0 更改为 1  。 然后单击底部的“保存”按钮。
1. 在“组织设置”中，转到“管道”部分并单击“设置”  。
1. 将“禁用经典生成管道的创建”和“禁用经典发布管道的创建”的开关切换到“关闭”

    > 注意：如果将“禁用经典发布管道的创建”开关设置为“开”，会隐藏经典发布管道创建选项，例如 DevOps 项目“管道”部分中的“发布”菜单。

1. 在“组织设置”中，转到“安全性”部分并单击“策略”  。
1. 将“允许公共项目”的开关切换为“开启”

    > 注意：某些实验室中使用的扩展可能需要公共项目才能使用免费版本。

1. 至少等待 3 个小时再使用 CI/CD 功能，以便在后端反映新的设置。 否则，仍会看到消息“未购买或授予托管并行”。

## 创建 Azure DevOps 示例项目的说明（只需执行此操作一次）

### 练习 0：配置实验室先决条件

> 注意：在继续执行这些步骤之前，请确保已完成创建 Azure DevOps 组织的步骤。

在本练习中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)。

#### 任务 1：创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 为项目提供以下设置：
    - 名称：eShopOnWeb
    - 可见性：专用
    - 高级：版本控制：Git
    - 高级：工作项流程：Scrum

1. 单击“创建”。

    ![创建项目](images/create-project.png)

#### 任务 2：导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“Repos”>“文件”，然后单击“导入存储库”。 选择“导入”  。 在“导入 Git 存储库”窗口中，粘贴以下 URL <https://github.com/MicrosoftLearning/eShopOnWeb.git> 并单击“导入”：

    ![导入存储库](images/import-repo.png)

1. 存储库按以下方式组织：
    - .ado 文件夹包含 Azure DevOps YAML 管道。
    - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
    - infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
    - .github 文件夹容器 YAML GitHub 工作流定义。
    - **src** 文件夹包含用于实验室方案的 .NET 8 网站。

现在，你已完成必要的先决条件步骤，可以继续练习本 AZ-400 课程的其他实验室。
