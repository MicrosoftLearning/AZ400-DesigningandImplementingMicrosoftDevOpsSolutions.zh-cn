---
lab:
  title: 使用 Azure Bicep 模板进行部署
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# 使用 Azure Bicep 模板进行部署

# 学生实验室手册

## 实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

- 设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。

- [Visual Studio Code](https://code.visualstudio.com/)。 此应用程序将作为本实验室先决条件安装。

## 实验室概述

在本实验室中，你将创建一个 Azure Bicep 模板，并使用 Azure Bicep 模块概念将其模块化。 然后，你会修改主部署模板以使用该模块，最后将所有资源部署到 Azure。

## 目标

完成本实验室后，你将能够：

- 了解并创建 Azure Bicep 模板。
- 为存储资源创建可重用的 Bicep 模块。
- 将链接模板上传到 Azure Blob 存储并生成 SAS 令牌。
- 修改主模板以使用该模块。
- 修改主模板以更新依赖关系。
- 使用 Azure Bicep 模板将所有资源部署到 Azure。

## 预计用时：60 分钟

## 说明

### 练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括 Visual Studio Code。

#### 任务 1：安装并配置 Git 和 Visual Studio Code

在此任务中，你将安装 Visual Studio Code。 如果你已经实现了此先决条件，则可以直接进行下一个任务。

1. 如果尚未安装 Visual Studio Code，请从实验室计算机启动 Web 浏览器，导航到 [Visual Studio Code 下载页面](https://code.visualstudio.com/)以进行下载和安装。

### 练习 1：创建和部署 Azure Bicep 模板

在本实验室中，你将创建一个 Azure Bicep 模板和一个模板模块。 然后，你将修改主部署模板以使用模板模块并更新依赖关系，最后将模板部署到 Azure。

#### 任务 1：创建 Azure Bicep 模板

在此任务中，你将使用 Visual Studio Code 创建 Azure Bicep 模板

1. 从实验室计算机中启动 Visual Studio Code，在 Visual Studio Code 中，单击“文件”顶层菜单，在下拉菜单中选择“首选项”，在级联菜单中选择“扩展”，在“搜索扩展”文本框中键入“Bicep”，选择由 Microsoft 发布的 Bicep，然后单击“安装”以安装 Azure Bicep 语言支持。
1. 在 Web 浏览器中，连接到 **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/main.bicep>** 。 单击文件的“原始”选项。 复制代码窗口的内容并将其粘贴到 Visual Studio Code 编辑器中。

   > **注意**：我们将使用某个名为“部署简单 Windows 模板 VM”的 [Azure 快速启动模板](https://azure.microsoft.com/en-us/resources/templates/)，而不是从头开始创建模板。 模板可从 GitHub 下载 - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows)。

1. 在实验室计算机上，打开文件资源管理器，创建以下用于存储模板的本地文件夹：

   - C:\\templates

1. 使用 main.bicep 模板切换回 Visual Studio Code 窗口，单击“文件”顶层菜单，在下拉菜单中，单击“另存为”，然后在新创建的本地文件夹“C:\\templates”中将模板另存为“main.bicep”。
1. 查看模板以更好地了解其结构。 模板中包含五种资源类型：

   - Microsoft.Storage/storageAccounts
   - Microsoft.Network/publicIPAddresses
   - Microsoft.Network/virtualNetworks
   - Microsoft.Network/networkInterfaces
   - Microsoft.Compute/virtualMachines

1. 在 Visual Studio Code 中，再次保存文件，但是这次选择 C:\\templates 作为目标位置，并选择 storage.bicep 作为文件名。

   > 注意：现在，我们有两个相同的 JSON 文件：C:\\templates\\main.bicep 和 C:\\templates\\。

#### 任务 2：创建用于存储资源的模板模块

在此任务中，你将修改在上一任务中保存的模板，使存储模板模块 storage.bicep 仅创建一个存储帐户，它将由第一个模板导入。 存储模板模块需要将值传递回主模板 main.bicep，并且此值将在存储模板模块的输出元素中进行定义。

1. 在 Visual Studio Code 窗口中显示的 storage.bicep 文件中，在资源部分下，移除除 storageAccounts 资源以外的所有资源元素。 此操作将导致资源部分如下所示：

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. 然后，移除所有变量定义：

   ```bicep
   var storageAccountName = 'bootdiags${uniqueString(resourceGroup().id)}'
   var nicName = 'myVMNic'
   var addressPrefix = '10.0.0.0/16'
   var subnetName = 'Subnet'
   var subnetPrefix = '10.0.0.0/24'
   var virtualNetworkName = 'MyVNET'
   var networkSecurityGroupName = 'default-NSG'
   var securityProfileJson = {
     uefiSettings: {
       secureBootEnabled: true
       vTpmEnabled: true
     }
     securityType: securityType
   }
   var extensionName = 'GuestAttestation'
   var extensionPublisher = 'Microsoft.Azure.Security.WindowsAttestation'
   var extensionVersion = '1.0'
   var maaTenantName = 'GuestAttestation'
   var maaEndpoint = substring('emptyString', 0, 0)
   ```

1. 接下来，删除除位置以外的所有参数值，并添加以下参数代码，从而生成以下结果：

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string
   ```

1. 接着，在文件末尾移除当前输出并添加名为 storageURI 输出值的新输出。 修改输出，使其如下所示。

   ```bicep
   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

1. 保存 storage.bicep 模板模块。 存储模板现在应如下所示：

   ```bicep
   @description('Location for all resources.')
   param location string = resourceGroup().location

   @description('Name for the storage account.')
   param storageAccountName string

   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }

   output storageURI string = storageAccount.properties.primaryEndpoints.blob
   ```

#### 任务 3：修改主模板以使用模板模块

在此任务中，你将修改主模板，以引用在上一个任务中创建的模板模块。

1. 在 Visual Studio Code 中，单击“文件”顶层菜单，在下拉菜单中选择“打开文件”，在“打开文件”对话框中，导航到 C:\\templates\\main.bicep 并选择该文件，然后单击“打开”。
1. 在 main.bicep 文件的资源部分，移除存储资源元素

   ```bicep
   resource storageAccount 'Microsoft.Storage/storageAccounts@2022-05-01' = {
     name: storageAccountName
     location: location
     sku: {
       name: 'Standard_LRS'
     }
     kind: 'Storage'
   }
   ```

1. 接下来，将以下代码直接添加到新删除的存储资源元素所在的位置：

   ```bicep
   module storageModule './storage.bicep' = {
     name: 'linkedTemplate'
     params: {
       location: location
       storageAccountName: storageAccountName
     }
   }
   ```

1. 我们还需要修改对虚拟机资源中存储帐户 blob URI 的引用，改用模块的输出。 找到虚拟机资源，将 diagnosticsProfile 部分替换为以下内容：

   ```bicep
   diagnosticsProfile: {
     bootDiagnostics: {
       enabled: true
       storageUri: storageModule.outputs.storageURI
     }
   }
   ```

1. 查看主模板中的以下详细信息：

   - 主模板中的一个模块用于链接到另一个模板。
   - 该模块有一个符号名称 storageModule。 此名称用于配置任何依赖项。
   - 使用模板模块时，只能使用增量部署模式。
   - 模板模块使用的是相对路径。
   - 使用参数将值从主模板传递到模板模块。

> 注意：使用 Azure ARM 模板时，会使用一个存储帐户来上传链接的模板，以方便他人使用。 使用 Azure Bicep 模块，可以选择将其上传到 Azure Bicep 模块注册表，该注册表具有公共和专用注册表选项。 有关详细信息，请参阅 [Azure Bicep 文档](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/modules#file-in-registry)。

1. 保存模板。

#### 任务 4：使用模板模块将资源部署到 Azure

> 注意：可以通过多种方式来部署模板，例如，使用本地安装的 Azure CLI，或者从 Azure Cloud Shell 或 CI/CD 管道进行部署。 在本实验室中，你将使用 Azure Cloud Shell 中的 Azure CLI。

> 注意：与 ARM 模板不同，不能使用 Azure 门户直接部署 Bicep 模板。

> 注意：若要使用 Azure Cloud Shell，需要将 main.bicep 和 storage.bicep 文件都上传到 Cloud Shell 的主目录中。

> 注意：目前，Azure CLI 不支持部署远程 Bicep 文件。 可以生成 bicep 文件以获取 ARM 模板 JSON，然后将其上传到存储帐户，然后再执行远程部署。

1. 在实验室计算机上，在显示 Azure 门户的 Web 浏览器中，单击“Cloud Shell”图标以打开 Cloud Shell。
   > 注意：如果本练习前面部分的 PowerShell 会话仍处于活动状态，请切换到 Bash（下一步）。
1. 在“Cloud Shell”窗格中，单击“PowerShell”，在下拉菜单中，单击“Bash”，然后在出现提示时，单击“确认”  。
1. 在 Cloud Shell 窗格中，单击“上传/下载文件”图标，然后在下拉菜单中单击“上传” 。
1. 在“打开”对话框中，导航到 C:\\templates\\main.bicep，选中该文件，然后单击“打开”。
1. 按照相同的步骤上传 C:\\templates\\storage.bicep 文件。
1. 在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以使用新上传的模板进行部署：

   ```bash
   az deployment group what-if --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. 当系统提示为“adminUsername”输入值时，键入“Student”并按 Enter 键 。
1. 当系统提示为“adminPassword”输入值时，键入“Pa55w.rd1234”并按 Enter 键 。 （不会显示键入密码）
1. 查看此命令的结果，它会验证部署，并告知模板中是否有任何错误。 这非常有用，尤其是在部署具有许多资源的模板以及在业务关键型云环境中部署时。

1. 在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以使用新上传的模板进行部署：

   ```bash
   LOCATION='<region>'
   ```
   > 注意：将区域名称替换为靠近你的位置的区域。 如果你不知道哪些位置可用，请运行 `az account list-locations -o table` 命令。
  
   ```bash
   az group create --name az400m06l15-RG --location $LOCATION
   ```

   ```bash   
   az deployment group create --name az400m06l15deployment --resource-group az400m06l15-RG --template-file main.bicep
   ```

1. 当系统提示为“adminUsername”输入值时，键入“Student”并按 Enter 键 。
1. 当系统提示为“adminPassword”输入值时，键入“Pa55w.rd1234”并按 Enter 键 。 （不会显示键入密码）

1. 如果在运行上述命令部署模板时收到错误，请尝试以下操作：

   - 如果有多个 Azure 订阅，请确保已将订阅上下文设置为部署资源组的正确上下文。
   - 确保可以通过指定的 URI 访问链接模板。

> **注意**：下一步，现在可以对主部署模板中的剩余资源定义（例如网络和虚拟机资源定义）进行模块化。

> **注意**：如果不打算使用部署的资源，应删除这些资源来避免产生相关费用。 只需删除资源组 az400m06l15-RG 即可完成此操作。

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

> **注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].name" --output tsv
   ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

   ```bash
   az group list --query "[?starts_with(name,'az400m06l15-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

   > **注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 审阅

在本实验室中，你学习了如何创建 Azure 资源管理器模板，使用链接模板对其进行模块化，修改主部署模板以调用链接模板和更新后的依赖关系，以及最终将模板部署到 Azure。
