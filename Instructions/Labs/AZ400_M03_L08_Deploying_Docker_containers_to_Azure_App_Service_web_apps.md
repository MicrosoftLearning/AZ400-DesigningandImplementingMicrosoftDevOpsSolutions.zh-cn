---
lab:
  title: 实验室 08：将 Docker 容器部署到 Azure 应用服务 Web 应用
  module: 'Module 03: Create and manage containers using Docker and Kubernetes'
---

# <a name="lab-08-deploying-docker-containers-to-azure-app-service-web-apps"></a>实验室 08：将 Docker 容器部署到 Azure 应用服务 Web 应用

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你是否拥有 Microsoft 帐户或具有 Azure 订阅中参与者或所有者角色的 Azure AD 帐户。 有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

## <a name="lab-overview"></a>实验室概述

在本实验室中，你将学习如何使用 Azure DevOps CI/CD 管道生成自定义 Docker 映像，将其推送到 Azure 容器注册表，并将其作为容器部署到 Azure 应用服务。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 使用 Microsoft 托管的 Linux 代理生成自定义 Docker 映像。
- 向 Azure 容器注册表推送映像。
- 使用 Azure DevOps 将 Docker 映像作为容器部署到 Azure 应用服务。

## <a name="estimated-timing-60-minutes"></a>预计用时：60 分钟

## <a name="instructions"></a>说明

### <a name="exercise-1-configure-the-lab-prerequisites"></a>练习 1：配置实验室先决条件

在本练习中，你将设置实验室先决条件，包括基于 Azure DevOps 演示生成器模板的团队项目和 Azure 资源（包括 Azure 应用服务 Web 应用、Azure 容器注册表实例和 Azure SQL 数据库）。

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 Docker 模板生成一个新项目。

> **注意**：基于 Docker 模板的项目将生成容器化的 ASP.NET Core 应用并将其部署到 Azure 应用服务

1. 在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。

    > **注意**：有关此站点的详细信息，请参阅 [https://docs.microsoft.com/en-us/azure/devops/demo-gen](https://docs.microsoft.com/en-us/azure/devops/demo-gen)。

1. 单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1. 如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1. 在“创建新项目”页面上的“新建项目名称”文本框中，键入“将 Docker 容器部署到 Azure 应用服务 Web 应用”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”。
1. 在模板列表中，在工具栏中，单击“DevOps 实验室”，选择“DevOps 实验室”标头，单击 Docker 模板，然后单击“选择模板”。
1. 返回“创建新项目”页面，如果提示安装缺失的扩展，请选中“Docker 集成”标签下方的复选框，并单击“创建项目”。

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1. 在“新建项目”页面上，单击“导航到项目” 。

#### <a name="task-2-create-azure-resources"></a>任务 2：创建 Azure 资源

在此任务中，你将使用 Azure Cloud Shell 创建本实验室中所需的 Azure 资源：

- Azure 容器注册表
- 用于容器的 Azure Web 应用
- Azure SQL Database

1. 从实验室计算机启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，并使用用户帐户登录，该帐户在本实验室中将使用的 Azure 订阅中具有所有者角色，并在与此订阅关联的 Azure AD 租户中具有全局管理员角色。
1. 在显示 Azure 门户的 Web 浏览器的工具栏中，单击搜索文本框右侧的“Cloud Shell”图标。
1. 如果系统提示选择“Bash”或“PowerShell”，请选择“Bash”。

    >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格中的 Bash 会话中，运行以下命令，创建代表你将用于在本实验室中部署资源的 Azure 区域的变量、包含这些资源的资源组以及这些资源的名称，包括 Azure 容器注册表实例、Azure 应用服务计划名称、Azure Web 应用名称、Azure SQL 数据库逻辑服务器名称和 Azure SQL 数据库名称：

1. 在“Cloud Shell”窗格中的 Bash 会话中，运行以下命令以创建资源组，该资源组将托管你在本实验室中部署的 Azure 资源（将 `<Azure_region>` 占位符替换为 Azure 区域的名称，例如“eastus”，表示你计划在其中部署这些资源）：

    ```bash
    LOCATION='<Azure_region>'
    ```

    >**注意**：可通过运行 `az account list-locations -o table` 来识别 Azure 区域名称

1. 运行以下命令，创建代表 Azure 资源名称的变量，包括 Azure 容器注册表实例、Azure 应用服务计划名称、Azure Web 应用名称、Azure SQL 数据库逻辑服务器名称和 Azure SQL 数据库名称：

    ```bash
    RG_NAME='az400m1501a-RG'
    ACR_NAME=az400m151acr$RANDOM$RANDOM
    APP_SVC_PLAN='az400m1501a-app-svc-plan'
    WEB_APP_NAME=az400m151web$RANDOM$RANDOM
    SQLDB_SRV_NAME=az400m15sqlsrv$RANDOM$RANDOM
    SQLDB_NAME='az400m15sqldb'
    ```

    >**注意**：可通过运行 `az account list-locations -o table` 来识别 Azure 区域名称

1. 运行以下命令，创建本实验室所需的所有 Azure 资源：

    ```bash
    az group create --name $RG_NAME --location $LOCATION
    az acr create --name $ACR_NAME --resource-group $RG_NAME --location $LOCATION --sku Standard --admin-enabled true
    az appservice plan create --name 'az400m1501a-app-svc-plan' --location $LOCATION --resource-group $RG_NAME --is-linux
    az webapp create --name $WEB_APP_NAME --resource-group $RG_NAME --plan $APP_SVC_PLAN --deployment-container-image-name elnably/dockerimagetest
    IMAGE_NAME=myhealth.web
    az webapp config container set --name $WEB_APP_NAME --resource-group $RG_NAME --docker-custom-image-name $IMAGE_NAME --docker-registry-server-url $ACR_NAME.azurecr.io/$IMAGE_NAME:latest --docker-registry-server-url https://$ACR_NAME.azurecr.io
    az sql server create --name $SQLDB_SRV_NAME --resource-group $RG_NAME --location $LOCATION --admin-user sqladmin --admin-password Pa55w.rd1234
    az sql db create --name $SQLDB_NAME --resource-group $RG_NAME --server $SQLDB_SRV_NAME --service-objective S0 --no-wait 
    az sql server firewall-rule create --name AllowAllAzure --resource-group $RG_NAME --server $SQLDB_SRV_NAME --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
    ```

    >**注意**：等待预配过程完成。 这可能需要大约 5 分钟。

1. 运行以下命令，配置新创建的 Azure Web 应用的连接字符串（分别用 Azure SQL 数据库逻辑服务器及其数据库实例的名称值替换 $SQLDB_SRV_NAME 和 $SQLDB_NAME 占位符）：

    ```bash
    CONNECTION_STRING="Data Source=tcp:$SQLDB_SRV_NAME.database.windows.net,1433;Initial Catalog=$SQLDB_NAME;User Id=sqladmin;Password=Pa55w.rd1234;"
    az webapp config connection-string set --name $WEB_APP_NAME --resource-group $RG_NAME --connection-string-type SQLAzure --settings defaultConnection="$CONNECTION_STRING"
    ```

1. 在显示 Azure 门户的 Web 浏览器中，关闭“Cloud Shell”窗格，导航到“资源组”边栏选项卡，然后在“资源组”边栏选项卡上，选择“az400m1501a-RG”条目。
1. 在“az400m1501a-RG”资源组边栏选项卡上，查看其资源列表。

    >**注意**：请记录逻辑 Azure SQL 数据库服务器的名称。 本实验室中稍后会用到它。

1. 在“az400m1501a-RG”资源组边栏选项卡上的资源列表中，单击代表容器注册表实例的条目。
1. 在容器注册表边栏选项卡上的左侧垂直菜单中，在“设置”部分中，单击“访问密钥”。
1. 在容器注册表实例的“访问密钥”边栏选项卡上，标识注册表名称、登录服务器、管理员用户和密码条目的值。

    >**注意**：请记录“注册表名称”和“登录服务器”的值（注册表名称和管理员用户名应匹配） 。 稍后将在本实验室用到它们。

### <a name="exercise-2-deploy-a-docker-container-to-azure-app-service-web-app-using-azure-devops"></a>练习 2：使用 Azure DevOps 将 Docker 容器部署到 Azure 应用服务 Web 应用

在此练习中，你将使用 Azure DevOps 将 Docker 容器部署到 Azure 应用服务 Web 应用。

#### <a name="task-1-configure-continuous-integration-ci-and-continuous-delivery-cd"></a>任务 1：配置持续集成(CI) 和持续交付 (CD)

在此任务中，你将使用上一个练习中生成的 Azure DevOps 项目，以实现 CI/CD 管道，该管道可生成 Docker 容器并将其部署到 Azure 应用服务 Web 应用。

1. 在实验室计算机上，切换到显示 Azure DevOps 门户的 Web 浏览器窗口，其中“将 Docker 容器部署到 Azure 应用服务 Web 应用”项目处于打开状态，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“Repos” 。

    >**注意**：需要首先修改对 Docker 映像的引用。

1. 在 Docker 存储库窗格的文件列表中，选择 docker-compos.ci.build.yml 。
1. 在 docker-compos.ci.build.yml 窗格中，单击“编辑”，将引用目标 Docker 映像的第 5 行替换为 `image: az400mp/aspnetcore-build:1.0-2.0`，选择“提交”，系统提示确认时再次单击“提交”    。
1. 在 Docker 存储库窗格的文件列表中，导航到 src/MyHealth.Web 文件夹，然后选择“Dockerfile”  。
1. 在 Dockerfile 窗格中，单击“编辑”，将引用基础 Docker 映像的第 1 行替换为 `FROM az400mp/aspnetcore1.0:1.0.4`，选择“提交”，系统提示确认时再次单击“提交”    。
1. 在显示 Azure DevOps 门户的 Web 浏览器窗口中，“将 Docker 容器部署到 Azure 应用服务 Web 应用”项目处于打开状态，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“管道” 。

    >**注意**：现在，你需要修改生成管道。

1. 在“管道”窗格上，单击代表 MHCDocker.build 管道的条目，然后在“MHCDocker.build”窗格上单击“编辑”   。

    >**注意**：生成管道包含以下任务

    | 任务 | 使用情况 |
    | ----- | ----- |
    | **运行服务** | 通过还原所需的包来准备生成环境 |
    | **生成服务** | 生成 myhealth.web 映像 |
    | **推送服务** | 将标记有 $(Build.BuildId) 的 myhealth.web 映像推送到容器注册表 |
    | **发布项目** | 允许通过 Azure DevOps 工件共享 dacpac 进行数据库部署 |

1. 在 MHCDocker.build 管道窗格上，确保选中“管道”条目，并在“代理规范”下拉列表中，选择“ubuntu-18.04”   。
1. 在“MHCDocker.build”管道窗格上的管道任务列表中，单击“运行服务”任务，在右侧的“Docker Compose”窗格中的“Azure 订阅”下拉列表中，选择代表你正在次实验室中使用的 Azure 订阅的条目，然后单击“授权”以创建相应的服务连接    。 出现提示时，使用在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色的帐户登录。

    >**注意**：此步骤会创建一个 Azure 服务连接，它使用服务主体身份验证 (SPA) 定义并保护与目标 Azure 订阅的连接。

1. 在管道的任务列表中，选择“运行服务”任务，然后在右侧的“Docker Compose”窗格的“Azure 容器注册表”下拉列表中，选择代表你此前在本实验室创建的 ACR 实例的条目（请根据需要刷新列表，或键入登录服务器的名称）   。
1. 重复前面的两个步骤，在“生成服务”和“推送服务”任务中配置 Azure 订阅和 Azure 容器注册表设置，但此时不是选择你的 Azure 订阅，而是选择新创建的服务连接。
1. 在同一“MHCDocker.build”管道窗格上，在窗格顶部，单击“保存并排队”按钮旁边的朝下脱字号，单击“保存”以保存更改，并在再次提示时单击“保存”。

    >**注意**：接下来，你将修改发布管道。

1. 在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧垂直菜单栏的“管道”部分，单击“发布”。
1. 在“管道/发布”窗格上，确保选择 MHCDocker.release 条目，然后单击“编辑”。
1. 在“所有管道/MHCDocker.release”窗格上，在表示部署的“开发”阶段的矩形中，单击“2 项作业，2 项任务”链接。

    >**注意**：发布管道包含以下任务

    | 任务 | 使用情况 |
    | ----- | ----- |
    | **执行 Azure SQL: DacpacTask** | 将 dacpac 工件（包括目标架构和数据）部署到 Azure SQL 数据库 |
    | **Azure 应用服务部署** | 从指定容器注册表中拉取在生成阶段生成的 docker 映像，并将该映像部署到 Azure 应用服务 Web 应用 |

1. 在管道的任务列表中，单击“执行 Azure SQL: DacpacTask ”任务，在右侧的“Azure SQL 数据库部署”窗格上的“Azure 订阅”下拉列表中，选择代表你在此任务前面部分创建的 Azure 服务连接的条目 。
1. 在管道的任务列表中，单击“Azure 应用服务部署”任务，在右侧的“Azure 应用服务部署”窗格上的“Azure 订阅”下拉列表中，选择代表你在此任务中前面部分创建的 Azure 服务连接的条目。然后在“应用服务名称”下拉列表中，选择代表你在本实验室前面部分部署的 Azure 应用服务 Web 应用的条目。

    >**注意**：接下来，你将需要配置部署所需的代理池信息。

1. 选择数据库部署作业，在右侧的“代理作业”窗格的“代理池”下拉列表中，选择“Azure Pipelines”，然后，在“代理规范”下拉列表中，选择 windows-2019     。
1. 选择“Web 应用程序部署”作业在右侧的“代理作业”窗格中的“代理池”下拉列表中，选择“Azure Pipelines”然后在“代理规范”下拉列表中选择“ubuntu-18.04”     。
1. 在窗格顶部，单击“变量”标头。
1. 在管道变量列表中，设置以下变量的值：

    | 变量 | 值 |
    | -------- | ----- |
    | ACR | 你在本实验室的上一个练习中记录的 Azure 容器注册表登录名，包括 azurecr.io 后缀 |
    | DatabaseName | **az400m15sqldb** |
    | 密码 | **Pa55w.rd1234** |
    | SQLadmin | **sqladmin** |
    | SQLserver | 你在本实验室的上一个练习中记录的 Azure SQL 数据库逻辑服务器名称，包括 database.windows.net 后缀 |

1. 在窗格右上角，单击“保存”按钮以保存更改，然后在再次出现提示时，单击“确定” 。

#### <a name="task-2-trigger-build-and-release-pipelines-by-using-code-commit"></a>任务 2：使用代码提交触发生成管道和发布管道

在此练习中，你将使用代码提交触发生成管道和发布管道。

1. 在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“Repos”。

    >**注意**：这将自动显示“文件”窗格。

1. 在“文件”窗格上，导航到“src/MyHealth.Web/Views/Home”文件夹，然后单击代表 Index.cshtml 文件的条目，然后单击“编辑”将其打开进行编辑。
1. 在“Index.cshtml”窗格的第 28 行，将“JOIN US”更改为“CONTACT US”，然后在窗格的右上角单击“提交”，并在提示确认时再次单击“提交”。

    >**注意**：此操作将启动源代码的自动生成。

1. 在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“管道”。
1. 在“管道”窗格上，单击代表由提交触发的管道运行的条目。
1. 在“MHCDocker.build”窗格上，单击代表管道运行的条目。
1. 在管道运行的“摘要”选项卡上，在“作业”部分中，单击“Docker”条目，然后在生成的窗格中监视各个任务的进度，直到作业成功完成。

    >**注意**：该生成将生成 Docker 映像并将其推送到 Azure 容器注册表。 生成完成后，你将能够查看其摘要。

1. 在显示 Azure DevOps 门户的 Web 浏览器窗口中，在 Azure DevOps 门户最左侧垂直菜单栏的“管道”部分，单击“发布”。
1. 在“发布”窗格上，单击代表由成功生成触发的最新发布的条目。
1. 在“MHCDocker.release”>“Release-1”窗格上，选择代表“开发”阶段的矩形。
1. 在“MHCDocker.release”>“Release-1”>“开发”窗格上，监视发布任务的进度，直到其成功完成。

    >**注意**：该发布将由生成过程生成的 Docker 映像部署到应用服务 Web 应用。 发布完成后，可以查看其摘要和日志。

1. 发布管道完成后，切换到显示 [Azure 门户](https://portal.azure.com)的 Web 浏览器窗口，并导航到你在本实验室前面部分配置的 Azure 应用服务 Web 应用的边栏选项卡。
1. 在应用服务 Web 应用上，单击代表目标 Web 应用的 URL 链接条目。

    >**注意**：这将自动打开一个新的 Web 浏览器选项卡，其中显示目标网站。

1. 验证目标 Web 应用是否显示 HealthClinic.biz 网站，包括你为触发 CI/CD 管道而应用的更改。

### <a name="exercise-3-remove-the-azure-lab-resources"></a>练习 3：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m1501')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本实验室中，你使用了 Azure DevOps CI/CD 管道生成自定义 Docker 映像，将其推送到 Azure 容器注册表，并使用 Azure DevOps 将其作为容器部署到 Azure 应用服务。
