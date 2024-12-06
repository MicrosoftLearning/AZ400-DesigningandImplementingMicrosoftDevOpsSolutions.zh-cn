---
lab:
  title: 实现用于 CI/CD 的 GitHub Actions
  module: 'Module 02: Implement CI with Azure Pipelines and GitHub Actions'
---

# 实现用于 CI/CD 的 GitHub Actions

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/azure/devops/server/compatibility)。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你是否拥有 Microsoft 帐户或 Microsoft Entra 帐户以及 Azure 订阅中的参与者或所有者角色。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/azure/active-directory/roles/manage-roles-portal)。

- 如果还没有可用于此实验室的 GitHub 帐户，请按照[注册新的 GitHub 帐户](https://github.com/join)中的说明创建一个帐户。

## 实验室概述

在此实验室中，你将学习如何实现部署 Azure Web 应用的 GitHub Action 工作流。

## 目标

完成本实验室后，你将能够：

- 为 CI/CD 实现 GitHub Action 工作流。
- 解释 GitHub Action 工作流的基本特征。

## 预计用时：40 分钟

## 说明

### 练习 1：将 eShopOnWeb 导入 GitHub 存储库

在本练习中，你会将现有的 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) 存储库代码导入你自己的 GitHub 专用存储库。

存储库按以下方式组织：
- .ado 文件夹包含 Azure DevOps YAML 管道。
- 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
- infra 文件夹包含某些实验室方案中使用的 Bicep 和 ARM 基础结构即代码模板。****
- .github 文件夹容器 YAML GitHub 工作流定义。
- src 文件夹包含用于实验室方案的 .NET 8 网站。****

#### 任务 1：在 GitHub 中创建公共存储库并导入 eShopOnWeb

在此任务中，你将创建一个空的公共 GitHub 存储库，并导入现有的 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) 存储库。

1. 在实验室计算机中，启动 Web 浏览器，导航到 [GitHub 网站](https://github.com/)，使用帐户登录，然后单击“新建”创建新存储库。

    ![“创建新存储库”按钮的屏幕截图。](images/github-new.png)

1. 在“创建新存储库”页上，单击“导入存储库”链接（页面标题下方）。

    > **注意**：还可以在 <https://github.com/new/import> 直接打开导入网站

1. 在“将项目导入 GitHub”页上：

    | 字段 | 值 |
    | --- | --- |
    | 旧存储库的克隆 URL| <https://github.com/MicrosoftLearning/eShopOnWeb> |
    | 所有者 | 帐户别名 |
    | 存储库名称 | eShopOnWeb |
    | 隐私 | **公共** |

1. 单击“开始导入”并等待存储库准备就绪。

1. 在存储库页上，转到“设置”，单击“操作”>“常规”，然后选择“允许所有操作和可重用工作流”选项。 单击“保存” 。

    ![启用 GitHub Actions 选项的屏幕截图。](images/enable-actions.png)

### 练习 2：设置 GitHub 存储库和 Azure 访问

在本练习中，你将创建一个 Azure 服务主体，以授权 GitHub 从 GitHub Actions 访问 Azure 订阅。 你还将设置 GitHub 工作流，用于生成、测试网站并将其部署到 Azure。

#### 任务 1：创建 Azure 服务主体并将其另存为 GitHub 机密

在此任务中，你将创建 GitHub 用于部署所需资源的 Azure 服务主体。 作为替代方法，还可以[在 Azure 中使用 OpenID Connect](https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure) 作为无机密身份验证机制。

1. 在实验室计算机上的浏览器窗口中，打开 Azure 门户 (<https://portal.azure.com/>)。
1. 在门户中，查找“资源组”并单击它。
1. 单击“+ 创建”为练习创建新的资源组。
1. 在“创建资源组”选项卡上，为资源组提供以下名称：rg-eshoponweb-NAME（将 NAME 替换为某个独一无二的别名）********。 单击“**查看 + 创建 > 创建**”。
1. 在 Azure 门户中，打开 Cloud Shell（搜索栏旁边）。

    > **注意**：如果这是你第一次打开 Cloud Shell，则需要配置[永久性存储](https://learn.microsoft.com/azure/cloud-shell/persisting-shell-storage)

1. 确保终端在 Bash 模式下运行并执行以下命令，将 SUBSCRIPTION-ID 和 RESOURCE-GROUP 替换为自己的标识符（这两者均可在资源组的“概述”页上找到）：

    `az ad sp create-for-rbac --name GH-Action-eshoponweb --role contributor --scopes /subscriptions/SUBSCRIPTION-ID/resourceGroups/RESOURCE-GROUP --sdk-auth`

    > **注意**：请确保以单行键入或粘贴！

    > **注意**：此命令将创建对之前创建的资源组具有参与者访问权限的服务主体。 这样，我们可以确保 GitHub Actions 仅具有仅与此资源组（而不是订阅的其余部分）交互所需的权限

1. 该命令将输出 JSON 对象，你稍后会将它用作工作流的 GitHub 机密。 复制 JSON。 JSON 包含用于以 Microsoft Entra 标识（服务主体）的名称对 Azure 进行身份验证的标识符。

    ```JSON
        {
            "clientId": "<GUID>",
            "clientSecret": "<GUID>",
            "subscriptionId": "<GUID>",
            "tenantId": "<GUID>",
            (...)
        }
    ```

1. （如果已注册，则跳过此步骤）还需要运行以下命令，为稍后将部署的 **Azure 应用服务**注册资源提供程序：

   ```bash
   az provider register --namespace Microsoft.Web
   ```

1. 在浏览器窗口中，返回到 eShopOnWeb GitHub 存储库。
1. 在存储库页上，转到“设置”，单击“机密和变量”>“操作”。 单击“新建存储库机密”
    - 名称：**`AZURE_CREDENTIALS`**
    - 机密：粘贴以前复制的 JSON 对象（GitHub 能够以相同的名称保存多个机密，由 [azure/login](https://github.com/Azure/login) 操作使用）

1. 单击“添加机密”。 现在，GitHub Actions 将能够使用存储库机密来引用服务主体。

#### 任务 2：修改和执行 GitHub 工作流

在此任务中，你将修改给定的 GitHub 工作流并执行该工作流，以在自己的订阅中部署解决方案。

1. 在浏览器窗口中，返回到 eShopOnWeb GitHub 存储库。
1. 在存储库页上，转到“代码”并打开以下文件：eShopOnWeb/.github/workflows/eshoponweb-cicd.yml。 此工作流为给定的 .NET 8 网站代码定义 CI/CD 进程。
1. 取消注释 on 部分（删除“#”）。 工作流每次推送到主分支时都会触发，并提供手动触发（“workflow_dispatch”）。
1. 在 env 部分进行以下更改：
    - 替换 RESOURCE-GROUP 变量中的 NAME。 它应该是在前面步骤中创建的同一资源组。
    - （可选）可以为 LOCATION 选择离你最近的 [azure 区域](https://azure.microsoft.com/explore/global-infrastructure/geographies)。 例如，“eastus”、“eastasia”、“westus”等。
    - 替换 SUBSCRIPTION-ID 中的 YOUR-SUBS-ID。
    - 将 WEBAPP-NAME 中的 NAME 替换为一些唯一别名。 它将用于使用 Azure 应用服务创建全局唯一网站。
1. 请仔细阅读工作流，并提供注释以帮助理解。

1. 单击“开始提交”和“提交更改”保留默认值（更改主分支）。 工作流将自动执行。

#### 任务 3：查看 GitHub 工作流执行

在此任务中，你将查看 GitHub 工作流执行：

1. 在浏览器窗口中，返回到 eShopOnWeb GitHub 存储库。
1. 在存储库页上，转到“操作”，在执行之前将看到工作流设置。 单击该磁贴。

    ![正在进行中的 GitHub 工作流的屏幕截图。](images/gh-actions.png)

1. 等待工作流完成。 在“摘要”中，可以看到两个工作流作业，即从执行中保留的状态和项目。 可以单击每个作业以查看日志。

    ![成功的工作流的屏幕截图。](images/gh-action-success.png)

1. 在浏览器窗口中，返回到 Azure 门户 (<https://portal.azure.com/>)。 打开之前创建的资源组。 你将看到 GitHub Action 使用 bicep 模板创建了 Azure 应用服务计划和应用服务。 可以看到已发布的网站打开了该应用服务且正在单击“浏览”。

    ![浏览 WebApp 的屏幕截图。](images/browse-webapp.png)

#### （可选）任务 4：使用 GitHub 环境添加手动审批预部署

在此任务中，你将使用 GitHub 环境在工作流的部署作业上执行定义的操作之前请求手动批准。

1. 在存储库页上，转到“代码”并打开以下文件：eShopOnWeb/.github/workflows/eshoponweb-cicd.yml。
1. 在“deploy”作业节中，可以找到对一个名为“Development”的环境的引用。************ GitHub 使用的[环境](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)为目标添加保护规则（和机密）。

1. 在存储库页上，转到“设置”，打开“环境”，然后单击“新建环境”。
1. 将其命名为 **`Development`**，然后单击“**配置环境**”。

    > **注意**：如果“**环境**”列表中已存在名为“**开发**”的环境，则单击该环境名称打开其配置。  

1. 在“配置开发”选项卡中，选中“必需审阅者”选项和作为审阅者的 GitHub 帐户。 单击“保存保护规则”。
1. 现在，让我们测试保护规则。 在存储库页上，转到“**操作**”，单击“**eShopOnWeb 生成和测试**”工作流，然后单击“**运行工作流 > 运行工作流**”以手动执行。

    ![手动触发工作流的屏幕截图。](images/gh-manual-run.png)

1. 单击工作流的开始执行，并等待 buildandtest 作业完成。 到达 deploy 作业后，你将看到评审请求。

1. 单击“查看部署”，选中“开发”，然后单击“批准和部署”。

    ![操作审批的屏幕截图。](images/gh-approve.png)

1. 工作流将遵循 deploy 作业执行并完成。

> [!IMPORTANT]
> 请记得删除在 Azure 门户中创建的资源，以避免不必要的费用。

## 审阅

在此实验室中，你实现了部署 Azure Web 应用的 GitHub Action 工作流。
