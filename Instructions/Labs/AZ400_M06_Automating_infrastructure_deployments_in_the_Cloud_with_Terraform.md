---
lab:
  title: 实验室 15：在云中使用 Terraform 和 Azure Pipelines 自动完成基础结构部署
  module: 'Module 06: Manage infrastructure as code using Azure, DSC, and third-party tools'
ms.openlocfilehash: 8c2df320b4a4b40bcc449cf00d6a479b864f624f
ms.sourcegitcommit: 31ed72aab4ce3ac156655216d24b04ebeea6fd0b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/29/2022
ms.locfileid: "144390485"
---
# <a name="lab-15-automating-infrastructure-deployments-in-the-cloud-with-terraform-and-azure-pipelines"></a>实验室 15：在云中使用 Terraform 和 Azure Pipelines 自动完成基础结构部署
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-overview"></a>实验室概述

[Terraform](https://www.terraform.io/intro/index.html) 是一种用于构建和更改基础结构以及对基础结构进行版本控制的工具。 Terraform 可以管理现有和热门的云服务提供商以及定制的内部解决方案。

Terraform 配置文件描述了运行单个应用程序或整个数据中心所需的组件。 Terraform 会生成一个执行计划，描述其达到所需状态的方法，然后执行这一方法来构建所描述的基础结构。 随着配置的更改，Terraform 能够确定更改的内容并创建要执行的增量执行计划。 

在本实验室中，你将了解如何将 Terraform 并入 Azure Pipelines，以便实现基础结构即代码。 

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 使用 Terraform 实现基础结构即代码
- 在 Azure 中使用 Terraform 和 Azure Pipelines 自动完成基础结构部署

## <a name="lab-duration"></a>实验室时长

-   估计时间：60 分钟

## <a name="instructions"></a>Instructions

### <a name="before-you-start"></a>开始之前

#### <a name="sign-in-to-the-lab-virtual-machine"></a>登录到实验室虚拟机

请确保已使用以下凭据登录到 Windows 10 虚拟机：
    
-   用户名：Student
-   密码：Pa55w.rd

#### <a name="review-applications-required-for-this-lab"></a>查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### <a name="set-up-an-azure-devops-organization"></a>设置 Azure DevOps 组织。 

如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

#### <a name="prepare-an-azure-subscription"></a>准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括基于 Azure DevOps 演示生成器模板的预先配置的 Parts Unlimited 团队项目。 

#### <a name="task-1-configure-the-team-project"></a>任务 1：配置团队项目

在此任务中，你将使用 Azure DevOps 演示生成器基于 Terraform 模板生成一个新项目。

1.  在实验室计算机上，启动 Web 浏览器并导航到 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)。 此实用工具将对以下过程进行自动化：在你的帐户中创建预填充了实验室所需内容（工作项、存储库等）的 Azure DevOps 项目。 

    > **注意**：有关此站点的详细信息，请参阅 https://docs.microsoft.com/en-us/azure/devops/demo-gen 。

1.  单击“登录”，并使用与你的 Azure DevOps 订阅相关联的 Microsoft 帐户登录。
1.  如果需要，在“Azure DevOps 演示生成器”页面上，单击“接受”以接受访问 Azure DevOps 订阅的权限请求 。
1.  在“新建项目”页面上的“新建项目名称”文本框中，键入“使用 Terraform 自动部署基础结构”，在“选择组织”下拉列表中选择你的 Azure DevOps 组织，然后单击“选择模板”    。
1.  在模板列表中，在工具栏中，单击“DevOps 实验室”，选择 Terraform 模板，然后单击“选择模板”  。
1.  返回“新建项目”页面，如果系统提示你安装缺少的扩展，请选中“替换令牌”和“Terraform”标签下方的复选框，然后单击“创建项目”   。

    > **注意**：等待此过程完成。 这大约需要 2 分钟。 如果该过程失败，请导航到你的 DevOps 组织，删除项目并重试。

1.  在“新建项目”页面上，单击“导航到项目” 。

### <a name="exercise-1-automate-infrastructure-deployments-in-the-cloud-with-terraform-and-azure-pipelines"></a>练习 1：在云中使用 Terraform 和 Azure Pipelines 自动完成基础结构部署

本练习将使用 Terraform 和 Azure Pipelines 将基础结构即代码部署到 Azure

#### <a name="task-1-examine-the-terraform-configuration-files"></a>任务 1：检查 Terraform 配置文件

在本任务中，你将检查预配部署 PartsUnlimited 网站所需 Azure 资源时的 Terraform 使用情况。

1.  实验室计算机的 Web 浏览器窗口中显示了 Azure DevOps 门户，其中打开了“使用 Terraform 自动部署基础结构”项目，在 Azure DevOps 门户最左侧的垂直菜单栏中，单击“存储库” 。
1.  在“文件”窗格上，单击顶部“主”条目旁的向下脱字号，然后在分支的下拉列表中，单击表示 terraform 分支的条目  。

    > **注意**：确保你现在位于 terraform 分支，且“Terraform”文件夹在存储库的存储信息中 。 

1.  在 Terraform 存储库的文件夹层次结构中，展开“Terraform”文件夹，然后单击“webapp.tf”  。
1.  在 webapp.tf 上查看 webapp.tf 文件的内容，然后单击“编辑”  。
1.  文件应如下例所示：

    ```
     terraform {
      required_version = ">= 0.11" 
      backend "azurerm" {
      storage_account_name = "__terraformstorageaccount__"
        container_name       = "terraform"
        key                  = "terraform.tfstate"
        access_key  ="__storagekey__"
        }
    }
    provider "azurerm" {
        features {} 
      }

    resource "azurerm_resource_group" "dev" {
      name     = "PULTerraform"
      location = "West Europe"
    }

    resource "azurerm_app_service_plan" "dev" {
      name                = "__appserviceplan__"
      location            = "${azurerm_resource_group.dev.location}"
      resource_group_name = "${azurerm_resource_group.dev.name}"

      sku {
        tier = "Free"
        size = "F1"
      }
    }

    resource "azurerm_app_service" "dev" {
      name                = "__appservicename__"
      location            = "${azurerm_resource_group.dev.location}"
      resource_group_name = "${azurerm_resource_group.dev.name}"
      app_service_plan_id = "${azurerm_app_service_plan.dev.id}"

    }
    ```
1.  单击“提交”，在“提交”窗格上，再次单击“提交”  。

    > 注意：webapp.tf 是 terraform 配置文件 。 Terraform 使用的是其专属文件格式，该格式称为 HCL（Hashicorp 配置语言），类似于 YAML。

    > **注意**：在本示例中，我们要创建部署网站所需的 Azure 资源组、应用服务计划和应用服务。 我们将 Terraform 文件添加到了 Azure DevOps 项目中的源代码管理存储库，以便你使用它来部署所需的 Azure 资源。 
    
#### <a name="task-2-build-your-application-using-azure-ci-pipeline"></a>任务 2：使用 Azure CI Pipeline 生成应用程序

在本任务中，你将生成应用程序并将所需文件以项目（称为 drop）形式发布。

1.  转到 Azure DevOps 门户，在 Azure DevOps 门户左侧的垂直菜单栏中，单击“管道”。 在下方，选择“管道”。
1.  在“管道”窗格上，单击“Terraform-CI”将其选中，然后在“Terraform-CI”窗格上，单击“编辑”   。

    > **注意**：这个 CI 管道有编译 .NET Core 项目的任务。 管道中的 DotNet 任务将还原依赖项，并生成、测试生成输出，然后将其发布到 zip 文件（包），该文件可部署到 Web 应用程序。 除应用程序生成外，我们还需要发布 terraform 文件才能生成项目并使其在 CD 管道中可用。 因此，需完成“复制文件”任务，将 Terraform 文件复制到项目目录。

1.  查看 Terraform-CI 窗格的“任务”选项卡后，单击“队列”（可能首先需要单击窗格右上角的省略号以显示下拉菜单）  。
1.  在“运行管道”窗格上，单击“运行”，启动生成 。 
1.  在生成运行窗格的“摘要”选项卡上的“作业”部分，单击“代理作业 1”并监视生成流程的进度  。
1.  生成成功后，切换回生成运行窗格的“摘要”选项卡，在“相关”部分，单击“已发布 1 项，已使用 1 项”链接  。 这将显示“项目”窗格。
1.  在“项目”窗格的“已发布”选项卡上，展开“drop”文件夹，验证其中是否包含 PartsUnlimitedwebsite.zip 文件，然后展开其“Terraform”子文件夹，验证其中是否包含 webapp.tf 文件     。 

#### <a name="task-3-deploy-resources-using-terraform-iac-in-azure-cd-pipeline"></a>任务 3：在 Azure CD 管道中使用 Terraform (IaC) 部署资源

在本任务中，你将在部署管道的过程中使用 Terraform 创建 Azure 资源，然后将 PartsUnlimited 应用程序部署到 Terraform 预配的 Azure 应用服务 Web 应用。

1.  转到 Azure DevOps 门户，在 Azure DevOps 门户左侧的垂直菜单栏的“管道”部分，单击“发布”，确保 Terraform-CD 条目已选中，然后单击“编辑”   。
1.  转到“所有管道”>“Terraform-CD”窗格，在表示“开发”阶段的矩形中，单击“1 项作业，8 项任务”链接  。
1.  在“开发”阶段的任务列表中，选择“使用 Azure CLI 部署所需的 Azure 资源”任务 。 
1.  在“Azure CLI”窗格的“Azure 订阅”下拉列表中，选择表示你的 Azure 订阅的条目，然后单击“授权”以配置相应的服务连接  。 出现提示时，使用在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色的帐户登录。

    > **注意**：默认情况下，Terraform 会在本地将状态存储在名为 terraform.tfstate 的文件中。 在团队中使用 Terraform 时，使用本地文件会导致 Terraform 的使用变得复杂。 Terraform 支持远程数据存储，有助于状态共享。 在这种情况下，我们使用 Azure CLI 任务创建 Azure 存储帐户和 Blob 容器来存储 Terraform 状态。 有关 Terraform 远程状态的更多信息，请参阅 [Terraform 文档](https://www.terraform.io/docs/state/remote.html)

1.  在“开发”阶段的任务列表中，选择“Azure PowerShell”任务 。 
1.  在“Azure PowerShell”窗格上的“Azure 连接类型”下拉列表中，选择“Azure 资源管理器”，然后在“Azure 订阅”下拉列表中选择新创建的 Azure 服务连接（位于“可用 Azure 服务连接”下方）   。

    > **注意**：若要配置 Terraform [后端](https://www.terraform.io/docs/backends/)，我们需要托管 Terraform 状态的 Azure 存储帐户的访问密钥。 在这种情况下，我们使用 Azure PowerShell 任务来检索上一任务中预配的 Azure 存储帐户的访问密钥。 通过使用 `Write-Host "##vso[task.setvariable variable=storagekey]$key"`，我们将创建一个管道变量，稍后的任务可以使用该变量。

1.  在“开发”阶段的任务列表中，选择“替换 Terraform 文件中的令牌”任务 。 将任务的“令牌模式”属性更新为 `__...__` 以匹配 webapp.tf 文件中的令牌 。

    > **注意**：如果你仔细查看过 webapp.tf 文件，你应该会注意到一些以“__”为前缀和后缀的值，例如 `__terraformstorageaccount__` 。 “替换令牌”任务会将这些值替换为发布管道中定义的变量值。
     
1.  Terraform 工具安装程序任务从 Internet 或工具缓存中安装指定版本的 Terraform，然后将其预追加到 Azure Pipelines 代理（托管或专用）的 PATH 中。
1.  在“开发”阶段的任务列表中，选择并查看“安装 Terraform”任务 。 本任务将安装指定的 Terraform 版本。

    > **注意**：通过自动化运行 Terraform 时，关键通常是核心计划/应用周期，一般包括以下三个阶段：

    1.  初始化 Terraform 工作目录。
    1.  生成一个计划，以便将当前配置修改为匹配所需配置。
    1.  应用在计划阶段确定的更改。
    
    > **注意**：发布管道中的其余 Terraform 任务会实现此工作流。

1.  在“开发”阶段的任务列表中，选择“Terraform: init”任务 。 
1.  在“Terraform”窗格上的“Azure 订阅”下拉列表中，选择先前使用的 Azure 服务连接 。 
1.  在“Terraform”窗格的“容器”下拉列表中，键入“terraform”并确保将“Key”参数设置为 terraform.tfstate    。 

    > **注意**：`terraform init` 命令解析当前工作目录中的所有 *.tf 文件，并自动下载处理这些文件所需的所有提供程序。 在本示例中，该命令将下载 [Azure 提供程序](https://www.terraform.io/docs/providers/azurerm/)，因为我们在部署 Azure 资源。 有关 `terraform init` 命令的详细信息，请参阅 [Terraform 文档](https://www.terraform.io/docs/commands/init.html)

1.  在“开发”阶段的任务列表中，选择“Terraform: plan”任务 。
1.  在“Terraform”窗格上的“Azure 订阅”下拉列表中，选择先前使用的 Azure 服务连接 。 
1.  在 Terraform 窗格的“其他命令参数”文本框中，输入 `-out=tfplan` 。

    > **注意**：`terraform plan` 命令用于创建执行计划。 Terraform 确定需要执行哪些操作才能达到配置文件中指定的所需状态。 这样，你无需实际应用即可查看哪些更改处于范围内。 有关 `terraform plan` 命令的详细信息，请参阅 [Terraform 文档](https://www.terraform.io/docs/commands/plan.html)

1.  在“开发”阶段的任务列表中，选择“Terraform: apply -auto-approve”任务 。 
1.  在“Terraform”窗格上的“Azure 订阅”下拉列表中，选择先前使用的 Azure 服务连接 。 
1.  在 Terraform 窗格的“其他命令参数”文本框中，将当前条目替换为 `-auto-approve tfplan` 。

    > **注意**：此任务将运行 `terraform apply` 命令来部署资源。 默认情况下，该任务还会提示你确认并继续。 由于我们正在自动执行部署，因此该任务包含 `auto-approve` 参数，无需进行确认。 

1.  在“开发”阶段的任务列表中，选择“Azure 应用服务部署”任务 。 从下拉列表中选择“Azure 服务连接”。

    > **注意**：此任务会将 PartsUnlimited 包部署到上一步中由“Terraform: apply -auto-approve”任务预配的 Azure 应用服务。

1.  在“开发”阶段上，单击“代理作业”，在“代理池”下拉列表上，选择 ：“Azure Pipelines”>“windows-2019”。
1.  在“所有管道”>“Terraform-CD”窗格上，单击“保存”，在“保存”对话框中，单击“确定”，然后在右上角，单击“创建发布”    。 
1.  在“新建发布”窗格上的“将触发器从自动更改为手动的阶段”下拉列表中，单击“开发”，在“项目”部分的“版本”下拉列表中，为此发布选择表示项目版本的条目，然后单击“创建”     。
1.  在 Azure DevOps 门户中，导航回“Terraform-CD”窗格，然后单击表示新创建的发布的“Release-1” 。 
1.  在“Terraform-CD”>“Release-1”边栏选项卡上，单击表示“开发”阶段的矩形，在“开发”窗格上，单击“部署”，然后再次单击“部署”    。
1.  返回“Terraform-CD”>“Release-1”边栏选项卡，单击表示“开发”阶段的矩形并监视部署流程 。 
1.  发布完成后，在实验计算机上启动另一个 Web 浏览器窗口，导航到 [Azure 门户](https://portal.azure.com)，然后通过在本实验室所使用的 Azure 订阅中至少具有参与者角色的用户帐户登录。 
1.  在 Azure 门户中，搜索并选择“应用服务”资源，然后从“应用服务”边栏选项卡中导航到名称以 pulterraformweb 开头的 Web 应用  。 
1.  在 Web 应用边栏选项卡上，单击“浏览”。 这将打开另一个 Web 浏览器选项卡，其中显示了新部署的 Web 应用程序。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv
    ```

1.  通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```sh
    az group list --query '[?contains(`["terraformrg", "PULTerraform"]`, name)].name' --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。


## <a name="review"></a>审阅

在本实验室中，你了解了如何使用 Azure Pipelines 在 Azure 上通过 Terraform 自动完成可重复的部署。
