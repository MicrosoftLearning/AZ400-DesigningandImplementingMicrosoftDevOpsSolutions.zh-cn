---
lab:
    title: '实验室 16：将多容器应用程序部署到 Azure Kubernetes 服务'
    module: '模块 16：创建和管理 Kubernetes 服务基础结构'
---

# 实验室 16：将多容器应用程序部署到 Azure Kubernetes 服务
# 学生实验室手册

## 实验室概述

[**Azure Kubernetes 服务 (AKS)**](https://azure.microsoft.com/zh-cn/services/kubernetes-service/) 是在 Azure 上使用 Kubernetes 的最快方法。 **Azure Kubernetes 服务 (AKS)** 管理托管的 Kubernetes 环境，使用户无需具备容器编排专业知识即可简单地部署和管理容器化应用程序。它还增强了容器化工作负载的敏捷性、可伸缩性和可用性。Azure DevOps 通过提供持续生成和持续部署功能，进一步简化了 AKS 操作。 

在本实验室中，你将使用 Azure DevOps 把容器化的 ASP.NET Core Web 应用程序 **MyHealthClinic** (MHC) 部署到 AKS 群集。 

## 目标

完成本实验室后，你将能够：

- 使用 Azure DevOps 演示生成器工具为 .NET Core 应用程序创建 Azure DevOps 团队项目。
- 使用 Azure CLI 创建 Azure 容器注册表 (ACR)、AKS 群集和 Azure SQL 数据库
- 使用 Azure DevOps 配置容器化应用程序和数据库部署
- 使用 Azure DevOps 管道进行生成以自动部署容器化应用程序

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

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预先配置的团队项目。 

#### 任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 **Azure Kubernetes 服务**模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **备注**：有关该站点的详细信息，请参阅 [https://docs.microsoft.com/zh-cn/azure/devops/demo-gen](https://docs.microsoft.com/zh-cn/azure/devops/demo-gen)。

1.  单击 **“登录”**，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在 **“Azure DevOps 演示生成器”** 页面上，单击 **“接受”** 以接受访问 Azure DevOps 订阅的权限请求。
1.  在 **“新建项目”** 页面上的 **“新建项目名称”** 文本框中，键入 **“将多容器应用程序部署到 AKS”**，在 **“选择组织”** 下拉列表中选择你的 Azure DevOps 组织，然后单击 **“选择模板”**。
1.  在模板列表中，在工具栏中，单击 **“DevOps 实验室”**，选择 **“Azure Kubernetes 服务”** 模板，然后单击 **“选择模板”**。
1.  返回 **“新建项目”** 页面，如果系统提示你安装缺少的扩展，请选中 **“替换令牌”** 和 **“Kubernetes 扩展”** 标签下方的复选框，然后单击 **“创建项目”**。

    > **备注**：等待此过程完成。该过程大约需要 2 分钟。如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在 **“新建项目”** 页面上，单击 **“导航到项目”**。

### 练习 1：使用 Azure DevOps 将容器化的 ASP.NET Core Web 应用程序部署到 AKS 群集

在本练习中，你将使用 Azure DevOps 将容器化的 ASP.NET Core Web 应用程序部署到 AKS 群集。

#### 任务 1：为实验室部署 Azure 资源

在此任务中，你将使用 Azure CLI 部署本实验室所需的 Azure 资源，包括：

| Azure 资源 | 描述 |
| --------------- | ----------- |
| Azure 容器注册表 | 充当 Docker 映像的专用存储 |
| AKS | 充当运行 Docker 映像的容器的业务流程协调程序 |
| Azure SQL 数据库 | 为 AKS 上运行的容器化工作负载提供持久性存储 |

1.  在实验室计算机上，启动 Web 浏览器，导航到 [**Azure 门户**](https://portal.azure.com)，然后使用在本实验室中使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。
1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 
1.  如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

    >**备注**：如果这是第一次启动 **Cloud Shell**，并显示消息 **“未装载任何存储”**，请选择你将在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以识别你将在本实验室中使用的 Azure 区域中可用的 Kubernetes 的最新版本（**将 `<Azure_region>` 占位符替换**为本实验室中使用的 Azure 区域的名称，你打算在该区域部署资源）：

    ```bash
    LOCATION='<Azure_region>'
    ```

    > **备注**：可以通过运行以下命令找到可能的位置，`<Azure_region>`：`az account list-locations -o table`，使用 **Name** 属性中没有空格的值。

    ```bash
    VERSION=$(az aks get-versions --location $LOCATION --query 'orchestrators[-1].orchestratorVersion' --output tsv); echo $VERSION
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以创建将托管 AKS 部署的资源组：

    ```bash
    RGNAME=az400m16l01a-RG
    az group create --name $RGNAME --location $LOCATION
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以使用可用的最新版本创建 AKS 群集：
    
    ```bash
    AKSNAME='az400m16aks'$RANDOM$RANDOM
    az aks create --location $LOCATION --resource-group $RGNAME --name $AKSNAME --enable-addons monitoring --kubernetes-version $VERSION --generate-ssh-keys
    ```
    
    >**备注**：等待部署完成，然后再继续执行下一个任务。AKS 部署可能需要 5 分钟左右。 

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以创建逻辑服务器，从而托管你将在本实验室中使用的 Azure SQL 数据库：
  
    ```bash
    SQLNAME='az400m16sql'$RANDOM$RANDOM
    az sql server create --location $LOCATION --resource-group $RGNAME --name $SQLNAME --admin-user sqladmin --admin-password P2ssw0rd1234
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以允许从 Azure 访问新预配的逻辑服务器：

    ```bash
    az sql server firewall-rule create --resource-group $RGNAME --server $SQLNAME --name allowAzure --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以创建将在本实验室中使用的 Azure SQL 数据库：


    ```bash
    az sql db create --resource-group $RGNAME --server $SQLNAME --name mhcdb --service-objective S0 --no-wait
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以创建将在本实验室中使用的 Azure 容器注册表：

    ```bash
    ACRNAME='az400m16acr'$RANDOM$RANDOM
    az acr create --location $LOCATION --resource-group $RGNAME --name $ACRNAME --sku Standard
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以授予 AKS 生成的托管标识，以便访问新创建的 ACR：

    ```bash
    # Retrieve the id of the service principal configured for AKS
    CLIENT_ID=$(az aks show --resource-group $RGNAME --name $AKSNAME --query "identityProfile.kubeletidentity.clientId" --output tsv)

    # Retrieve the ACR registry resource id
    ACR_ID=$(az acr show --name $ACRNAME --resource-group $RGNAME --query "id" --output tsv)

    # Create role assignment
    az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
    ```

    >**备注**：有关此分配的详细信息，请参阅[通过 Azure Kubernetes 服务使用 Azure 容器注册表进行身份验证](https://docs.microsoft.com/zh-cn/azure/container-registry/container-registry-auth-aks)

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以显示逻辑服务器的名称，该逻辑服务器托管你在此任务之前创建的 Azure SQL 数据库：

    ```bash
    echo $(az sql server list --resource-group $RGNAME --query '[].name' --output tsv)'.database.windows.net'
    ```

1.  从 Cloud Shell 窗格的 **Bash** 会话，运行以下命令以显示你在此任务之前创建的 Azure 容器注册表的登录服务器的名称：

    ```bash
    az acr show --name $ACRNAME --resource-group $RGNAME --query "loginServer" --output tsv
    ```

    >**备注**：请记录这两个值。在下一个任务中需要使用它们。

1. 关闭 Cloud Shell 窗格。

#### 任务 2：配置生成和发布管道

在此任务中，你通过将 Azure 资源（包括 AKS 群集和 Azure 容器注册表）映射到生成和发布定义，在本实验室前面部分生成的 Azure DevOps 项目中配置生成和发布管道。

1.  在实验室计算机上，切换到显示 Azure DevOps 门户的 Web 浏览器窗口，其中“**将多容器应用程序部署到 AKS**”项目处于打开状态，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“**Repos**”。

    >**备注**：需要首先修改对 Docker 映像的引用。

1.  在“**AKS**”存储库窗格中的文件列表中，选择“**docker-compose.ci.build.yml**”。
1.  在“**docker-compose.ci.build.yml**”窗格中，单击“**编辑**”，将引用目标 Docker 映像的第 **5** 行替换为 `image: az400mp/aspnetcore-build:1.0-2.0`，选择“**提交**”，然后在系统提示确认时再次单击“**提交**”。 
1.  在“**AKS**”存储库窗格的文件列表中，导航到 **src/MyHealth.Web** 文件夹，然后选择“**Dockerfile**”。
1.  在 **Dockerfile** 窗格中，单击“**编辑**”，将引用基础 Docker 映像的第 **1** 行替换为 `FROM az400mp/aspnetcore1.0:1.0.4`，选择“**提交**”，系统提示确认时再次单击“**提交**”。 
1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，其中“**将多容器应用程序部署到 AKS**”项目处于打开状态，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“**管道**”。

    >**备注**：现在，你需要修改生成管道。

1.  在“**管道**”窗格的“**全部**”选项下，单击代表“**MyHealth.AKS.build**”管道的条目，然后在“**MyHealth.AKS.build**”窗格上单击“**编辑**”。

1.  在 **MyHealth.AKS.build** 管道窗格上，确保选中“**管道**”条目，并在“**代理规范**”下拉列表中，选择“**ubuntu-18.04**”。
1.  在管道的任务列表中，单击“**替换 appsettings.json 中的令牌**”任务，在“**令牌模式**”下拉列表中选择“```__...__```”。 
1.  在管道的任务列表中，单击“**运行服务**”任务，在右侧 **Docker Compose** 窗格的“**Azure 订阅**”下拉列表中，选择代表你在本实验室中使用的 Azure 订阅的条目，然后单击“授权”以创建相应的服务连接。出现提示时，使用在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色的帐户登录。

    >**备注**：请等待授权过程完成。此步骤会创建一个 Azure 服务连接，它使用服务主体身份验证 (SPA) 定义并保护与目标 Azure 订阅的连接。 

1.  在管道的任务列表中，在 **“运行服务”** 任务处于选中状态的情况下，在右侧 **“Docker Compose”** 窗格的 **“Azure 容器注册表”** 下拉列表中，选择代表在本实验室前面部分创建的 ACR 实例的条目。
     >**备注**：根据需要**刷新**列表，并选择之前创建的 ACR。**如果未显示任何选项，请键入完整的 ACR 名称：ACRNAME.azurecr.io**。
1.  重复前面的两个步骤，以在 **“生成服务”**、**“推送服务”** 和 **“锁定服务”** 任务中配置 **“Azure 订阅”**（下次不再授权，使用创建的 **“可用 Azure 服务连接”**）和 **“Azure 容器注册表”** 设置，但在本例中，请选择新创建的服务连接，而不选择 Azure 订阅。 

    >**备注**：管道包含以下任务

    | 任务 | 用法 |
    | ----- | ----- |
    | **替换令牌** | 在 **appsettings.json** 文件和 **mhc-aks.yaml** 清单文件的数据库连接字符串中，将占位符替换为 ACR 的名称 |
    | **运行服务** |  通过拉取所需的映像（例如 aspnetcore-build:1.0-2.0）和还原 **csproj** 中引用的包来准备环境 |
    | **生成服务** |  生成 **docker-compose.yml** 文件中指定的 Docker 映像并使用 **“$(Build.BuildId)”** 和 **“latest”** 标记标记映像 |
    | **推送服务** |  将 Docker 映像 **myhealth.web** 推送到 Azure 容器注册表 |
    | **推送生成项目** |  将 **mhc-aks.yaml** 和 **myhealth.dacpac** 文件发布到 Azure DevOps 中的项目放置位置，以便可以在后续发布中使用这些文件 |

    >**备注**：**appsettings.json** 文件包含用于连接到本实验室前面创建的 Azure SQL 数据库的数据库连接字符串的详细信息。**mhc-aks.yaml** 清单文件包含将在 Azure Kubernetes 服务中部署的**部署**、**服务**和 **Pod** 的配置详细信息。有关部署清单的详细信息，请参阅 [AKS 部署和 YAML 清单](https://docs.microsoft.com/zh-cn/azure/aks/concepts-clusters-workloads#deployments-and-yaml-manifests)
1.  在“**管道变量**”列表中，使用你在上一个任务结束时记录的值（SQLPassword 为 **P2ssw0rd1234**、SQLuser 为 **sqladmin**、SQLdatabase 为 **mhcdb**）更新 **ACR** 和 **SQLserver** 变量的值，然后单击“**保存并排队**”按钮旁边的向下插入点，单击“**保存**”以保存更改，在系统再次提示时单击“**保存**”。
1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧垂直菜单栏的 **“Pipelines”** 部分，单击 **“发布”**。 
1.  在 **“管道/发布”** 窗格上，选择 **“MyHealth.AKS.Release”** 条目，然后单击 **“编辑”**。
1.  在 **“所有管道/MyHealth.AKS.Release”** 窗格上，在表示部署的**开发**阶段的矩形中，单击 **“2 个作业，3 个任务”** 链接。
1.  对于 **“DB 部署”** 作业和 **“AKS 部署”** 作业（通过单击这些名称），选择 **“代理池”** Azure Pipelines -> Windows-2019。
1.  在 **“开发”** 阶段的任务列表的 **“DB 部署”** 作业部分，选择 **“执行 Azure SQL: DacpacTask”** 任务，然后在右侧 **“Azure SQL 数据库部署”** 窗格的 **“Azure 订阅”** 下拉列表中，选择代表此任务之前创建的 Azure 服务连接的条目。
1.  在 **“开发”** 阶段的任务列表的 **“AKS 部署”** 作业部分，选择 **“在 AKS 中创建部署和服务”** 任务。 
1.  在右侧 **Kubectl** 窗格的 **“Azure 订阅”** 下拉列表中，选择代表相同 Azure 服务连接的条目，在 **“资源组”** 下拉列表中，选择 **“az400m16l01a-RG”** 条目，然后在 **“Kubernetes 群集”** 下拉列表中，选择代表在本实验室前面部署的 AKS 群集的条目。
1.  在**开发**阶段的任务列表的“**AKS 部署**”作业部分，在“**在 AKS 中创建部署和服务**”任务处于选中状态的情况下，在右侧“**Kubectl**”窗格中，向下滚动并展开“**机密**”部分，在“**Azure 订阅**”下拉列表中，选择代表相同 Azure 服务连接的条目，然后在“**Azure 容器注册表**”下拉列表中，选择代表在本实验室前面创建的 Azure 容器注册表的条目。
1.  针对 **“在 AKS 中更新映像”** 任务重复上述两个步骤。

    >**备注**：根据 **mhc-aks.yaml** 文件中指定的配置，**“在 AKS 中创建部署和服务”** 任务将在 AKS 中创建所需的部署和服务。Pod 将提取最新的 Docker 映像。

    >**备注**：**“在 AKS 中更新映像”** 任务将从指定的存储库中拉取与 BuildID 相对应的所需映像，并将该映像部署到在 AKS 中运行的 **mhc-front Pod** 中。

    >**备注**：通过在后台使用命令 `kubectl create secret`，通过 Azure DevOps 在 AKS 群集中创建一个名为 **mysecretkey** 的机密。此机密将用于授权对 Azure 容器注册表的访问，以便拉取 myhealth.web 映像。

1.  在 **“MyHealth.AKS.Release”** 发布管道的**开发**阶段的 **“任务”** 窗格上，单击 **“变量”** 选项卡。 
1.  在 **“管道变量”** 列表中，将 **ACR** 变量的值更新为在上一个任务结束时记录的 Azure 容器注册表名称。 
1.  在“**管道变量**”列表中，将 **SQLserver** 变量的值更新为在上一个任务结束时记录的逻辑服务器名称（SQLPassword 为 **P2ssw0rd1234**、SQLuser 为 **sqladmin**、SQLdatabase 为 **mhcdb**）。 
1.  在 **“所有管道/MyHealth.AKS.Release”** 窗格的右上角，单击 **“保存”**，在系统出现提示时，再次单击 **“保存”** 以保存所做的更改。

    >**备注**：在管道变量列表中，**DatabaseName** 设置为 **mhcdb**，**SQLuser** 设置为 **sqladmin**，**SQLpassword** 设置为 **P2ssw0rd1234**。如果在本实验室前面创建 Azure SQL 数据库时输入了不同的值，请相应地更新变量的值。

#### 任务 3：触发生成和发布管道

在此任务中，你将触发生成和发布管道并验证其完成过程。

1.  在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧垂直菜单栏的 **“Pipelines”** 部分，单击 **“Pipelines”**。 
1.  在“**管道**”窗格的“**全部**”选项下，选择“**MyHealth.AKS.build**”管道，在“**MyHealth.AKS.build**”窗格上，单击“**运行管道**”，然后在“**运行管道**”窗格上，单击“**运行**”。
1.  在生成管道运行窗格的 **“作业”** 部分，单击 **“阶段 1”**，并监视生成过程的进度。

    >**备注**：生成将生成 Docker 映像并将其推送到 ACR。生成完成时，可以查看生成摘要。 

1.  若要查看生成的映像，请切换到显示 Azure 门户的 Web 浏览器窗口。
1.  在 Azure 门户中，搜索并选择 **“容器注册表”** 资源类型，然后在 **“容器注册表”** 边栏选项卡上，选择在本实验室前面创建的 Azure 容器注册表。
1.  在“Azure 容器注册表”边栏选项卡上的 **“服务”** 部分，单击 **“存储库”** 并验证存储库列表是否包含 **myhealth.web** 条目。
1.  切换回显示 Azure DevOps 门户的 Web 浏览器窗口。
1.  在 Azure DevOps 门户中最左侧垂直菜单栏的 **“Pipelines”** 部分，单击 **“发布”**，然后在 **“MyHealth.AKS.Release”** 边栏选项卡上，单击最新发布，然后选择 **“进行中”** 链接以监视发布的进度。
1.  发布完成时，切换到显示 Azure 门户的 Web 浏览器窗口。
1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 
1.  从 Cloud Shell 窗格的 Bash 会话，运行以下命令以访问在本实验室前面部署的 AKS 群集：

    ```bash
    RGNAME=az400m16l01a-RG
    AKSNAME=$(az aks list --resource-group $RGNAME --query '[].name' --output tsv)
    az aks get-credentials --resource-group $RGNAME --name $AKSNAME
    ```

1.  从 Cloud Shell 窗格的 Bash 会话，运行以下命令以列出在 AKS 中运行的 Pod，这些 Pod 是使用发布管道部署的：

    ```bash
    kubectl get pods
    ```

1.  从 Cloud Shell 窗格的 Bash 会话，运行以下命令以列出负载均衡器服务，该服务提供了一个用于访问容器化应用程序的外部 IP 地址：

    ```bash
    kubectl get service mhc-front --watch
    ```

    >**备注**：我们的应用程序旨在通过提供外部连接的负载均衡器服务部署在 Pod 中。 

1.  记下命令输出的 **“外部 IP”** 列中 IP 地址的值，打开一个新的 Web 浏览器标签页，浏览到该 IP 地址，并确认 **MyHealthClinic** 应用程序是否正在运行。

    >**备注**：Kubernetes 包含一个可用于基本管理操作的 Web 仪表板。使用此仪表板，可以查看应用程序的基本运行状况状态和指标，创建并部署服务，以及编辑现有应用程序。遵循 [Microsoft Docs](https://docs.microsoft.com/zh-cn/azure/aks/kubernetes-dashboard) 以访问 Azure Kubernetes 服务 (AKS) 中的 Kubernetes Web 仪表板

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**：请记得删除不再使用的所有新创建的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m16l01a-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**：该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你学习了如何使用 Azure DevOps 把容器化的 ASP.NET Core Web 应用程序 **MyHealthClinic** (MHC) 部署到 AKS 群集。 
