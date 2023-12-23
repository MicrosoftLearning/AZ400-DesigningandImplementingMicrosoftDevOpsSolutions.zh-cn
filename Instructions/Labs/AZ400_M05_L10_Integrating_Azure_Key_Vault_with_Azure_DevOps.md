---
lab:
  title: 将 Azure Key Vault 与 Azure DevOps 集成
  module: 'Module 05: Implement a secure continuous deployment using Azure Pipelines'
---

# 将 Azure Key Vault 与 Azure DevOps 集成

## 学生实验室手册

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://learn.microsoft.com/azure/devops/server/compatibility)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

## 实验室概述

Azure Key Vault 可安全存储和管理敏感数据，例如密钥、密码和证书。 Azure Key Vault 支持硬件安全模块以及各种加密算法和密钥长度。 通过使用 Azure Key Vault，可以最大程序降低通过源代码泄漏敏感数据的可能性，这是开发人员经常犯的一个错误。 访问 Azure Key Vault 需要正确的身份验证和授权，从而支持对其内容进行细化的权限管理。

在本实验室中，你将了解如何使用以下步骤将 Azure 密钥保管库与 Azure Pipelines 集成：

- 创建 Azure Key Vault 以将 ACR 密码存储为机密。
- 创建 Azure 服务主体以提供对 Azure Key Vault 中的机密的访问权限。
- 配置权限以允许服务主体读取机密。
- 配置管道，以从 Azure Key Vault 检索密码并将其传递到后续任务。

## 目标

完成本实验室后，你将能够：

- 创建 Microsoft Entra 服务主体。
- 创建 Azure 密钥保管库。

## 预计用时：40 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括设置新的 Azure DevOps 项目，该项目的存储库基于 [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb)。

#### 任务 1：（如果已完成，请跳过此任务）创建和配置团队项目

在此任务中，你将创建一个 eShopOnWeb Azure DevOps 项目，供多个实验室使用。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织。 单击“新建项目”。 将项目命名为 eShopOnWeb，并将其他字段保留默认值。 单击“创建”。

    ![创建项目](images/create-project.png)

#### 任务 2：（如果已完成，请跳过此任务）导入 eShopOnWeb Git 存储库

在此任务中，你将导入将由多个实验室使用的 eShopOnWeb Git 存储库。

1. 在实验室计算机上，在浏览器窗口中打开 Azure DevOps 组织和以前创建的 eShopOnWeb 项目。 单击“Repos > 文件”、“导入”。 在“导入 Git 存储库”窗口中，粘贴以下 URL https://github.com/MicrosoftLearning/eShopOnWeb.git 并单击“导入”：

    ![导入存储库](images/import-repo.png)

2. 存储库按以下方式组织：
    - .ado 文件夹包含 Azure DevOps YAML 管道。
    - 设置 .devcontainer 文件夹容器，使用容器（在 VS Code 或 GitHub Codespaces 中本地进行）开发。
    - .azure 文件夹包含某些实验室方案中使用的 Bicep&ARM 基础结构即代码模板。
    - .github 文件夹容器 YAML GitHub 工作流定义。
    - src**** 文件夹包含用于实验室方案的 .NET 7 网站。

### 练习 1：设置 CI 管道以生成 eShopOnWeb 容器

设置 CI YAML 管道，用于：

- 创建 Azure 容器注册表以保留容器映像
- 使用 Docker Compose 生成和推送 eshoppublicapi 和 eshopwebmvc 容器映像。 将仅部署 eshopwebmvc 容器。

#### 任务 1：（如果已完成，请跳过此任务）创建服务主体

在此任务中，你将使用 Azure CLI 创建服务主体，这将允许 Azure DevOps：

- 在 Azure 订阅上部署资源。
- 对稍后创建的密钥保管库机密具有读取访问权限。

> 注意：如果你已有一个服务主体，则可以直接进行下一个任务。

你需要一个服务主体从 Azure Pipelines 部署 Azure 资源。 由于我们要检索管道中的机密，因此创建 Azure 密钥保管库时，我们需要为该服务授予权限。

从管道定义内部连接到 Azure 订阅或从项目设置页面（自动选项）新建服务连接时，Azure Pipelines 会自动创建服务主体。 你也可以从门户或使用 Azure CLI 手动创建服务主体，然后在项目中重复使用。

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Microsoft Entra 租户中具有全局管理员角色。
2. 在 Azure 门户中，单击页面顶部搜索文本框右侧的 Cloud Shell 图标。
3. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

   >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

4. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以检索 Azure 订阅 ID 和订阅名称属性的值 ：

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **注意**：将两个值都复制到文本文件中。 稍后将在本实验室用到它们。

5. 在 Bash 提示符的 Cloud Shell 窗格中，运行以下命令以创建服务主体（将 myServicePrincipalName 替换为任意由字母和数字组成的唯一字符串）和 mySubscriptionID（替换为你的 Azure subscriptionId）：

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > **注意**：此命令将生成 JSON 输出。 将输出复制到文本文件中。 本实验室中稍后会用到它。

6. 接下来，从实验室计算机启动 Web 浏览器，导航到 Azure DevOps eShopOnWeb 项目。 单击“项目设置 > 服务连接”（在“管道”下）和“新建服务连接”。

    ![新建服务连接](images/new-service-connection.png)

7. 在“新建服务连接”边栏选项卡上，选择“Azure 资源管理器”和“下一步”（可能需要向下滚动）。

8. 选择“服务主体(手动)”并单击“下一步”。

9. 使用在前面的步骤中收集的信息填写空字段：
    - 订阅 ID 和名称。
    - 服务主体 ID (appId)、服务主体密钥（密码）和租户 ID（租户）。
    - 在“服务连接名称”中，键入 azure subs。 需要 Azure DevOps 服务连接来与 Azure 订阅通信时，将在 YAML 管道中引用此名称。

    ![Azure 服务连接](images/azure-service-connection.png)

10. 单击“验证并保存”。

#### 任务 2：设置和运行 CI 管道

在此任务中，你将导入现有的 CI YAML 管道定义、修改并运行它。 它将创建新的 Azure 容器注册表 (ACR) 并生成/发布 eShopOnWeb 容器映像。

1. 从实验室计算机启动 Web 浏览器，导航到 Azure DevOps eShopOnWeb 项目。 转到“管道 > 管道”，然后单击“创建管道”（或“新建管道”）。

2. 在“你的代码在哪里?”窗口中，选择“Azure Repos Git (YAML)”并选择“eShopOnWeb”存储库。

3. 在“配置”部分，选择“现有 Azure Pipelines YAML 文件”。 提供以下路径 /.ado/eshoponweb-ci-dockercompose.yml，然后单击“继续”。

    ![选择“管道”](images/select-ci-container-compose.png)

4. 在 YAML 管道定义中，通过替换 AZ400-EWebShop-NAME 中的 NAME 来自定义资源组名称，并将 YOUR-SUBSCRIPTION-ID 替换为你自己的 Azure subscriptionId。

5. 单击“保存并运行”，等待管道成功执行。

    > **注意**：此生成可能需要花费几分钟时间完成。 生成定义由以下任务构成：
    - AzureResourceManagerTemplateDeployment 使用 bicep 部署 Azure 容器注册表。
    - PowerShell 任务获取 bicep 输出（ACR 登录服务器）并创建管道变量。
    - DockerCompose 任务生成 eShopOnWeb 的容器映像并将其推送到 Azure 容器注册表。

6. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道 > 管道”，然后单击最近创建的管道。 单击省略号和“重命名/删除”选项。 将其命名为 eshoponweb-ci-dockercompose，然后单击“保存”。

7. 执行完成后，在 Azure 门户中打开以前定义的资源组，应找到 Azure 容器注册表 (ACR)，其中包含创建的容器映像 eshoppublicapi 和 eshopwebmvc。 仅在部署阶段使用 eshopwebmvc。

    ![ACR 中的容器映像](images/azure-container-registry.png)

8. 单击“访问密钥”并复制密码值，该值将用于以下任务，因为我们会在 Azure 密钥保管库中将其保留为机密。

    ![ACR 密码](images/acr-password.png)

#### 任务 2：创建 Azure 密钥保管库

在本任务中，你将使用 Azure 门户创建 Azure 密钥保管库。

对于本实验室场景，我们将有一个 Azure 容器实例 (ACI)，用于拉取并运行存储在 Azure 容器注册表 (ACR) 中的容器映像。 我们打算将 ACR 的密码作为机密存储到密钥保管库中。

1. 在 Azure 门户中的“搜索资源、服务和文档”文本框中，键入“密钥保管库”，然后按 Enter 键。
2. 选择“密钥保管库”边栏选项卡，单击“创建 > 密钥保管库”。
3. 在“创建密钥保管库”边栏选项卡的“基本信息”选项卡中，指定以下设置，然后单击“下一步”：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 你在此实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组 AZ400-EWebShop-NAME 的名称 |
    | 密钥保管库名称 | 任何唯一的有效名称，如 ewebshop-kv-NAME（替换 NAME） |
    | 区域 | 靠近实验室环境位置的 Azure 区域 |
    | 定价层 | **标准** |
    | 保留已删除保管库的天数 | **7** |
    | 清除保护 | **禁用清除保护** |

4. 在“创建密钥保管库”边栏选项卡的“访问配置”选项卡上，选择“保管库访问策略”，然后在“访问策略”部分中，单击“+ 创建”以设置新策略    。

    > **注意**：你需要保护对密钥保管库的访问，只允许得到授权的应用程序和用户进行访问。 若要从保管库访问数据，你需要提供对先前创建的服务主体的读取（获取/列出）权限，以便在管道中进行身份验证。 

    1. 在“权限”边栏选项卡上“机密权限”的下方，选中“获取”和“列出”权限。 单击“下一步”。
    2. 在“主体”边栏选项卡上，使用给定的“ID”或“名称”搜索以前创建的服务主体，并从列表中选择它。 单击“下一步”、“下一步”、“创建”（访问策略）  。
    3. 在“查看 + 创建”边栏选项卡上，单击“创建”。

5. 返回到“创建密钥保管库”边栏选项卡，单击“查看 + 创建”>“创建”

    > 注意：等待预配 Azure 密钥保管库。 此过程应该会在 1 分钟内完成。

6. 在“部署完成”边栏选项卡上，单击“转到资源”。
7. 在“Azure 密钥保管库”(ewebshop-kv-NAME) 边栏选项卡左侧垂直菜单中的“对象”部分，单击“机密” 。
8. 在“机密”边栏选项卡上，单击“生成/导入”。
9. 在“创建机密”边栏选项卡上，指定以下设置并单击“创建”（将其他设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 上传选项 | **手动** |
    | 名称 | acr-secret |
    | 值 | 在上一任务中复制的 ACR 访问密码 |

#### 任务 3：创建连接到 Azure 密钥保管库的变量组

在此任务中，你将在 Azure DevOps 中创建一个变量组，该变量组将使用服务连接（服务主体）从密钥保管库中检索 ACR 密码机密。

1. 在实验室计算机上，启动 Web 浏览器并导航到 Azure DevOps 项目 eShopOnWeb。

2. 在 Azure DevOps 门户的垂直导航窗格中，选择“管道 > 库”。 单击“+ 变量组”。

3. 在“新建变量组”边栏选项卡上，指定以下设置：

    | 设置 | 值 |
    | --- | --- |
    | 变量组名称 | eshopweb-vg |
    | 链接 Azure 密钥保管库中的机密 | **enable** |
    | Azure 订阅 | 可用 Azure 服务连接 > Azure 订阅 |
    | 密钥保管库名称 | 你的密钥保管库名称|

4. 在“变量”下，单击“+ 添加”，然后选择 acr-secret 机密。 单击“确定”。
5. 单击“保存” 。

    ![变量组创建](images/vg-create.png)

#### 任务 4：设置 CD 管道以在 Azure 容器实例 (ACI) 中部署容器

在此任务中，你将导入一个 CD 管道，对其进行自定义并运行，以部署之前在 Azure 容器实例中创建的容器映像。

1. 从实验室计算机启动 Web 浏览器，导航到 Azure DevOps eShopOnWeb 项目。 转到“管道 > 管道”，然后单击“新建管道”。

2. 在“你的代码在哪里?”窗口中，选择“Azure Repos Git (YAML)”并选择“eShopOnWeb”存储库。

3. 在“配置”部分，选择“现有 Azure Pipelines YAML 文件”。 提供以下路径 /.ado/eshoponweb-cd-aci.yml，然后单击“继续”。

4. 在 YAML 管道定义中，自定义：

    - YOUR-SUBSCRIPTION-ID，替换为你的 Azure 订阅 ID。
    - az400eshop-NAME，替换 NAME 使其全局唯一。
    - YOUR-ACR.azurecr.io 和 ACR-USERNAME 与 ACR 登录服务器（这两者都需要 ACR 名称，可以在“ACR > 访问密钥”上查看）。
    - AZ400-EWebShop-NAME，其中包含之前在实验室中定义的资源组名称。

5. 单击“保存并运行”。
6. 打开管道并等待成功执行。

    > 重要说明：如果看到消息“此管道需要访问资源的权限，然后才能继续运行 Docker Compose to ACI”，请再次单击“查看”、“允许”和“允许”。 这是允许管道创建资源所必需的。

    > 注意：部署可能需要几分钟才能完成。 CD 定义由以下任务构成：
    - 资源：已准备好根据 CI 管道完成自动触发。 它还会下载 bicep 文件的存储库。
    - 变量（对于部署阶段）连接到变量组，以使用 Azure 密钥保管库机密 acr-secret
    - AzureResourceManagerTemplateDeployment 使用 bicep 模板部署 Azure 容器实例 (ACI)，并提供 ACR 登录参数以允许 ACI 从 Azure 容器注册表 (ACR) 下载以前创建的容器映像。

7. 管道将采用基于项目名称的名称。 让我们重命名它，以便更好地识别管道。 转到“管道 > 管道”，然后单击最近创建的管道。 单击省略号和“重命名/删除”选项。 将其命名为 eshoponweb-cd-aci，然后单击“保存”。

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，打开创建的资源组，然后单击“删除资源组”。

## 审阅

在本实验室中，你已使用以下步骤将 Azure Key Vault 与 Azure DevOps 管道集成：

- 创建了一个 Azure 服务主体，用于提供对 Azure 密钥保管库机密的访问权限，并验证从 Azure DevOps 到 Azure 的部署。
- 运行从 Git 存储库导入的 2 个 YAML 管道。
- 配置了一个管道，以使用变量组从 Azure 密钥保管库检索密码并将其用于后续任务。
