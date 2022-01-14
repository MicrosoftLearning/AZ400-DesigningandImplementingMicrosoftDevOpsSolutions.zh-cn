---
lab:
    title: '实验室 07： 将 Azure Key Vault 与 Azure DevOps 集成'
    module: '模块 7：管理应用程序配置和机密'
---

# 实验室 07： 将 Azure Key Vault 与 Azure DevOps 集成
# 学生实验室手册

## 实验室概述

Azure Key Vault 可安全存储和管理敏感数据，例如密钥、密码和证书。Azure Key Vault 支持硬件安全模块以及各种加密算法和密钥长度。通过使用 Azure Key Vault，可以最大程序降低通过源代码泄漏敏感数据的可能性，这是开发人员经常犯的一个错误。访问 Azure Key Vault 需要正确的身份验证和授权，从而支持对其内容进行细化的权限管理。

在本实验室中，你将了解如何使用以下步骤将 Azure Key Vault 与 Azure DevOps 管道集成：

- 创建 Azure Key Vault 以将 MySQL 服务器密码存储为机密。
- 创建 Azure 服务主体以提供对 Azure Key Vault 中的机密的访问权限。
- 配置权限以允许服务主体读取机密。
- 配置管道，以从 Azure Key Vault 检索密码并将其传递到后续任务。

## 目标

完成本实验室后，你将能够：

-   创建 Azure Active Directory (Azure AD) 服务主体。
-   创建 Azure 密钥保管库。 
-   通过 Azure DevOps 管道跟踪拉取请求。

## 实验室持续时间

-   预计用时：**40 分钟**

## 说明

### 准备工作

#### 登录到实验室虚拟机

请确保已使用以下凭据登录到 Windows 10 虚拟机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

#### 设置 Azure DevOps 组织

如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/zh-cn/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预配置的 Parts Unlimited 团队项目。

#### 任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 **Azure Key Vault** 模板生成新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **备注**：有关该站点的详细信息，请参阅 https://docs.microsoft.com/zh-cn/azure/devops/demo-gen。

1.  单击 **“登录”**，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在 **“Azure DevOps 演示生成器”** 页面上，单击 **“接受”** 以接受访问 Azure DevOps 订阅的权限请求。
1.  在 **“新建项目”** 页面上的 **“新建项目名称”** 文本框中，键入 **“将 Azure Key Vault 与 Azure DevOps 集成”**，在 **“选择组织”** 下拉列表中选择你的 Azure DevOps 组织，然后单击 **“选择模板”**。
1.  在 **“选择模板”** 页面上的标题菜单中，单击 **“DevOps 实验室”**，在模板列表中，单击 **“Azure Key Vault”** 模板，然后单击 **“选择模板”**。
1.  返回 **“新建项目”** 页面，选中 **“ARM 输出”** 标签下方的复选框，然后单击 **“创建项目”**

    > **备注**：等待此过程完成。该过程大约需要 2 分钟。如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在 **“新建项目”** 页面上，单击 **“导航到项目”**。

### 练习 1：将 Azure Key Vault 与 Azure DevOps 集成

- 创建 Azure 服务主体以提供对 Azure 密钥保管库中机密的访问权限。
- 创建 Azure 密钥保管库以将 MySQL 服务器密码作为机密存储。
- 配置权限以允许服务主体读取机密。
- 配置管道，以从 Azure Key Vault 检索密码并将其传递到后续任务。

#### 任务 1：创建服务主体 

在本任务中，你将使用 Azure CLI 创建服务主体。 

> **备注**：如果你已有一个服务主体，则可以直接进行下一个任务。

你需要服务主体才能将应用从 Azure Pipelines 部署到 Azure 资源。由于我们要检索管道中的机密，因此创建 Azure 密钥保管库时，我们需要为该服务授予权限。 

从管道定义内部连接到 Azure 订阅或从项目设置页面新建服务连接时，Azure Pipeline 会自动创建服务主体。你也可以从门户或使用 Azure CLI 手动创建服务主体，然后在项目中重复使用。如果要获取一组预定义的权限，建议使用现有的服务主体。

1.  从实验室计算机启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。
1.  在 Azure 门户中，单击页面顶部搜索文本框右侧的 **Cloud Shell** 图标。 
1.  如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

   >**备注**：如果这是第一次启动 **Cloud Shell**，并显示消息 **“未装载任何存储”**，请选择你将在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1.  在 **Bash** 提示符的 **Cloud Shell** 窗格中，运行以下命令以创建服务主体（将 `<service-principal-name>` 替换为任意由字母和数字组成的唯一字符串）：

    ```
    az ad sp create-for-rbac --name <service-principal-name>
    ```

    > **备注**：此命令将生成 JSON 输出。将输出复制到文本文件中。稍后将在本实验室用到它。

1.  在 **Bash** 提示符的 **Cloud Shell** 窗格中，运行以下命令以检索 Azure 订阅 ID 和订阅名称属性的值： 

    ```
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > **备注**：将两个值都复制到文本文件中。稍后将在本实验室用到它们。


#### 任务 2：创建 Azure 密钥保管库

在本任务中，你将使用 Azure 门户创建 Azure 密钥保管库。

对于本实验室方案，我们使用可连接到 MySQL 数据库的应用。我们打算将 MySQL 数据库的密码作为机密存储到密钥保管库中。

1.  在 Azure 门户中的 **“搜索资源、服务和文档”** 文本框中，键入 **“Key vaults”**，然后按 **Enter** 键。 
1.  在 **“密钥保管库”** 边栏选项卡上，单击 **“+ 机密”**。 
1.  在 **“创建密钥保管库”** 边栏选项卡的 **“基本”** 选项卡中，指定以下设置，然后单击 **“下一步：访问策略”**：

    | 设置 | 值 |
    | --- | --- |
    | 订阅 | 在本实验室中使用的 Azure 订阅的名称 |
    | 资源组 | 新资源组名称 **az400m07l01-RG** |
    | 密钥保管库名称 | 任何唯一有效的名称 |
    | 区域 | 靠近实验室环境位置的 Azure 区域 |
    | 定价层 | **标准** |
    | 保留已删除保管库的天数 | **7** |
    | 清除保护 | **禁用清除保护** |

1.  在 **“创建密钥保管库”** 边栏选项卡的 **“访问策略”** 选项卡上，单击 **“+ 添加访问策略”** 以设置新策略。

    > **备注**：你需要保护对密钥保管库的访问，只允许得到授权的应用程序和用户进行访问。若要从保管库访问数据，你需要提供对服务主体的读取（获取）权限，以便在管道中进行身份验证。 

1.  在 **“添加访问策略”** 边栏选项卡上，直接单击 **“选择主体”** 标签下的 **“未选择”** 链接。 
1.  在 **“主体”** 边栏选项卡上，搜索在上一练习中创建的安全主体，将其选中，然后单击 **“选择”**。 

    > **备注**： 可以按主体的名称或 ID 进行搜索。

1.  返回 **“添加访问策略”** 边栏选项卡，在 **“机密权限”** 下拉列表中，选中 **“获取”** 和 **“列出”** 权限旁的复选框，然后单击 **“添加”**。 
1.  返回 **“创建密钥保管库”** 边栏选项卡的 **“访问策略”** 选项卡，单击 **“查看 + 创建”**，然后在 **“查看 + 创建”** 边栏选项卡上，单击 **“创建”**。 

    > **备注**：等待预配 Azure 密钥保管库。所需时间应该不超过一分钟。

1.  在 **“部署完成”** 边栏选项卡上，单击 **“转到资源”**。
1.  在 Azure 密钥保管库边栏选项卡左侧的垂直菜单中的 **“设置”** 部分，单击 **“机密”**。 
1.  在 **“机密”** 边栏选项卡上，单击 **“生成/导入”**。
1.  在 **“创建机密”** 边栏选项卡上，指定以下设置并单击 **“创建”**（将其他设置保留为默认值）：

    | 设置 | 值 |
    | --- | --- |
    | 上传选项 | **手动** |
    | 名称 | **sqldbpassword** |
    | 值 | 任何有效的 MySQL 密码值 |


#### 任务 3：检查 Azure Pipeline

在本任务中，将 Azure Pipeline 配置为从 Azure 密钥保管库检索机密。

1.  在实验计算机上启动 Web 浏览器，然后导航到在上一练习中创建的 Azure DevOps 项目 **“将 Azure Key Vault 与 Azure DevOps 集成”**。
1.  在 Azure DevOps 门户的垂直导航窗格中，选择 **“管道”**，然后验证是否显示了 **“管道”** 窗格。
1.  在“**管道**”窗格中，单击表示 **SmartHotel-CouponManagement-CI** 管道的条目。单击“**编辑**”。
1.  在管道定义中，将“**管道**” > “**代理规范**”修改为“**ubuntu 18.04**”。单击“**保存并排队**” > “**保存并运行**”以触发生成。
1.  在 Azure DevOps 门户的垂直导航窗格中的 **“管道”** 部分，选择 **“版本”**。 
1.  在 **“SmartHotel-CouponManagement-CD”** 窗格上，单击右上角的 **“编辑”**。
1.  在 **“所有管道”>“SmartHotel-CouponManagement-CD”** 窗格上，选择 **“任务”** 选项卡，然后在下拉菜单中选择 **“开发”**。

    > **备注**： **“开发”** 阶段的版本定义具有 **Azure Key Vault** 任务。此任务从 Azure Key Vault 下载*机密*。你需要指向之前在实验室中创建的订阅和 Azure Key Vault 资源。

    > **备注**：你需要为管道授权才能部署到 Azure。Azure 管道可以使用新的服务主体自动创建服务连接，**但建议使用之前创建的服务连接**，因为它有权读取机密。 

1.  选择“**在代理上运行**”，并将“**代理池**”字段修改为“**Azure Pipelines**”，将代理规范修改为“**ubuntu 18.04**”。
1.  选择 **“Azure Key Vault”** 任务，然后在右侧的 **“Azure Key Vault”** 任务属性的 **“Azure 订阅”** 标签旁，单击 **“管理”**。 
这将打开另一个浏览器标签页，该选项卡显示了 Azure DevOps 门户中的 **“服务连接”** 窗格。
1.  在 **“服务连接”** 窗格上，单击 **“新建服务连接”**。 
1.  在 **“新建服务连接”** 窗格上，选择 **“Azure 资源管理器”** 选项，单击 **“下一步”**，选择 **“服务主体(手动)”**，然后再次单击 **“下一步”**。
1.  在 **“新建服务连接”** 窗格上，使用 Azure CLI 创建服务主体，然后使用复制到本练习的第一个任务内的文本文件中的信息来指定以下设置：

    - 订阅 ID：通过运行 `az account show --query id --output tsv` 获取的值
    - 订阅名称：通过运行 `az account show --query name --output tsv` 获取的值
    - 服务主体 ID：通过运行 `az ad sp create-for-rbac --name <service-principal-name>` 而生成的输出中的标记为 **appId** 的值
    - 服务主体密钥：通过运行 `az ad sp create-for-rbac --name <service-principal-name>` 而生成的输出中的标记为 **“密码”** 的值
    - 租户 ID：通过运行 `az ad sp create-for-rbac --name <service-principal-name>` 而生成的输出中的标记为 **“租户”** 的值

1.  在 **“新建服务连接”** 窗格上，单击 **“验证”** 以确定提供的信息是否有效。 
1.  收到 **“验证成功”** 响应后，在 **“服务连接名称”** 文本框中，键入 **“kv-service-connection”**，然后单击 **“验证并保存”**。
1.  切换回显示管道定义和 **Azure Key Vault** 任务的 Web 浏览器标签页。
1.  选中 **“Azure Key Vault”** 任务后，在 **“Azure Key Vault”** 窗格上，单击 **“刷新”** 按钮，在 **“Azure 订阅”** 下拉列表中，选择 **“kv-service-connection”** 条目，在 **“密钥保管库”** 下拉列表中，选择在第一个任务中创建的表示 Azure 密钥保管库的条目，然后在 **“机密筛选器”** 文本框中，键入 **“sqldbpassword”**。最后，展开 **“输出变量”** 部分，然后在 **“参考名称”** 文本框中，键入 **“sqldbpassword”**。 

    > **备注**：在运行时，Azure Pipelines 将提取最新的机密值，然后将其设置为任务变量 **$(sqldbpassword)**。通过引用该变量，这些任务可以被后续任务使用。  

1.  若要验证这一点，请选择下一个任务 **“Azure 部署”** （部署 ARM 模板），并查看 **“覆盖模板参数”** 文本框中的内容。 

    ```
    -webAppName $(webappName) -mySQLAdminLoginName "azureuser" -mySQLAdminLoginPassword $(sqldbpassword)
    ```

    > **备注**： **“覆盖模板参数”** 内容会引用 **sqldbpassword** 变量来设置 mySQL 管理员密码。这将使用在密钥保管库中指定的密码来预配 ARM 模板中定义的 MySQL 数据库。 

1.  可以通过指定任务的订阅（如果在项目中使用了新的订阅，请单击 **“授权”**）和位置来完成管道定义。对 **“Azure 应用服务部署”** 管道中的最后一项任务*重复*相同的操作（从下拉列表中的 **“可用 Azure 服务连接”** 部分选择订阅）。 

    > **备注**： 在“Azure 订阅”下拉列表中，你将看到已被授权连接到 Azure 的订阅的**可用 Azure 服务连接**。如果再次从 **“可用 Azure 订阅”** 列表选择已授权的订阅并尝试 **“授权”**，该过程会失败。

1.  在“**变量**”选项卡上，将“**resourcegroup**”变量更改为纯文本（单击锁定），并将“**az400m07l01-RG**”写入值字段中。
1.  最后，**保存**并单击 **“新建发布” > “创建”** （保留默认值）以开始部署。

1. 请确保管道成功运行，并在完成后通过在 Azure 门户中打开资源组 **az400m07l01-RG** 来查看创建的资源。打开并浏览（**“概述”->“浏览”**） **“应用服务”**，查看已发布的网站。

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**：请记得删除不再使用的所有新创建的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m07l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**：该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

#### 回顾

在本实验室中，你已使用以下步骤将 Azure Key Vault 与 Azure DevOps 管道集成：

- 创建了 Azure 密钥保管库以将 MySQL 服务器密码作为机密存储。
- 创建了 Azure 服务主体以提供对 Azure 密钥保管库中机密的访问权限。
- 配置了权限以允许服务主体读取机密。
- 配置了管道以从 Azure 密钥保管库检索密码并将其传递给后续任务。
