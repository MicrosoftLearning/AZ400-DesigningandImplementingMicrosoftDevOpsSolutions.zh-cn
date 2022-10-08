---
lab:
  title: 实验室 14：使用 Azure 资源管理器模板进行 Azure 部署
  module: 'Module 06: Manage infrastructure as code using Azure and DSC'
---

# <a name="lab-14-azure-deployments-using-azure-resource-manager-templates"></a>实验室 14：使用 Azure 资源管理器模板进行 Azure 部署

# <a name="student-lab-manual"></a>学生实验室手册

## <a name="lab-requirements"></a>实验室要求

- 本实验室需要使用 Microsoft Edge 或[支持 Azure DevOps 的浏览器](https://docs.microsoft.com/en-us/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers)。

-               设置 Azure DevOps 组织：如果还没有可用于本实验室的 Azure DevOps 组织，请按照[创建组织或项目集合](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/create-organization?view=azure-devops)中的说明创建一个。

- 标识现有的 Azure 订阅或创建一个新的 Azure 订阅。

- 验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。

- [Visual Studio Code](https://code.visualstudio.com/)。 此应用程序将作为本实验室先决条件安装。

## <a name="lab-overview"></a>实验室概述

在本实验室中，你将创建一个 Azure 资源管理器 (ARM) 模板，并使用链接模板概念将其模块化。 然后，你将修改主部署模板以调用链接的模板和更新的依赖关系，最后将模板部署到 Azure。

## <a name="objectives"></a>目标

完成本实验室后，你将能够：

- 创建资源管理器模板。
- 创建用于存储资源的链接模板。
- 将链接模板上传到 Azure Blob 存储并生成 SAS 令牌。
- 修改主模板以调用链接模板。
- 修改主模板以更新依赖关系。
- 使用链接模板将资源部署到 Azure。

## <a name="estimated-timing-60-minutes"></a>预计用时：60 分钟

## <a name="instructions"></a>说明

### <a name="exercise-0-configure-the-lab-prerequisites"></a>练习 0：配置实验室先决条件

在本练习中，你将设置实验室先决条件，其中包括 Visual Studio Code。

#### <a name="task-1-install-and-configure-git-and-visual-studio-code"></a>任务 1：安装并配置 Git 和 Visual Studio Code

在此任务中，你将安装 Visual Studio Code。 如果你已经实现了此先决条件，则可以直接进行下一个任务。

1. 如果尚未安装 Visual Studio Code，请从实验室计算机启动 Web 浏览器，导航到 [Visual Studio Code 下载页面](https://code.visualstudio.com/)以进行下载和安装。

### <a name="exercise-1-author-and-deploy-azure-resource-manager-templates"></a>练习 1：创作和部署 Azure 资源管理器模板

在本实验室中，你将创建 Azure 资源管理器模板，并使用链接的模板将其模块化。 然后，你将修改主部署模板以调用链接的模板和更新的依赖关系，最后将模板部署到 Azure。

#### <a name="task-1-create-resource-manager-template"></a>任务 1：创建资源管理器模板

在此任务中，你将使用 Visual Studio Code 创建资源管理器模板

1. 从实验室计算机中启动 Visual Studio Code，在 Visual Studio Code 中，单击“文件”顶层菜单，在下拉菜单中，选择“首选项”，在级联菜单中选择“扩展”，在“搜索扩展”文本框中，键入“Azure 资源管理器 (ARM) 工具”，选择相应的搜索结果，然后单击“安装”以安装 Azure 资源管理器工具     
1. 在 Web 浏览器中，连接到 **<https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows/azuredeploy.json>** 。 单击文件的“原始”选项。 复制代码窗口的内容并将其粘贴到 Visual Studio Code 编辑器中。

    > **注意**：我们将使用某个名为“部署简单 Windows 模板 VM”的 [Azure 快速启动模板](https://azure.microsoft.com/en-us/resources/templates/)，而不是从头开始创建模板。 模板可从 GitHub 下载 - [vm-simple-windows](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.compute/vm-simple-windows)。

1. 在实验室计算机上，打开文件资源管理器并创建以下用于存储模板的本地文件夹：

    - C:\\templates
    - C:\\templates\\storage

1. 使用 azuredeploy.json 模板切换回 Visual Studio Code 窗口，单击“文件”顶层菜单，在下拉菜单中，单击“另存为”，然后在新创建的本地文件夹“C:\\templates”中将模板另存为“azuredeploy.json”   。
1. 查看模板以更好地了解其结构。 模板中包含五种资源类型：

    - Microsoft.Storage/storageAccounts
    - Microsoft.Network/publicIPAddresses
    - Microsoft.Network/virtualNetworks
    - Microsoft.Network/networkInterfaces
    - Microsoft.Compute/virtualMachines

1. 在 Visual Studio Code 中，再次保存文件，但是这次选择 C:\\templates\\storage 作为目标位置，并选择 storage.json 作为文件名  。

    > **注意**：现在，我们有两个相同的 JSON 文件：C:\\templates\\azuredeploy.json and C:\\templates\\storage\\storage.json 。

#### <a name="task-2-create-a-linked-template-for-storage-resources"></a>任务 2：创建用于存储资源的链接模板

在此任务中，你将修改在上一任务中保存的模板，使链接存储模板 storage.json 仅创建一个存储帐户，而其执行将由第一个模板调用。 链接存储模板需要将值传递回主模板 azuredeploy.json，并且此值将在链接存储模板的输出元素中定义。

1. 在 Visual Studio Code 窗口中显示的 storage.json 文件中的资源部分下，删除除 storageAccounts 资源以外的所有资源元素  。 此操作将导致资源部分如下所示：

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2021-04-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage"
      }
    ],
    ```

1. 将 storageAccount 的名称元素从变量重命名为参数

    ```json
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[parameters('storageAccountName')]",
        "location": "[parameters('location')]",
        "apiVersion": "2021-04-01",
        "sku": {
           "name": "Standard_LRS"
        },
        "kind": "Storage",
        "properties": {}
      }
    ],
    ```

1. 接下来，删除整个变量部分和所有变量定义：

    ```json
    "variables": {
      "storageAccountName": "[concat('bootdiags', uniquestring(resourceGroup().id))]",
      "nicName": "myVMNic",
      "addressPrefix": "10.0.0.0/16",
      "subnetName": "Subnet",
      "subnetPrefix": "10.0.0.0/24",
      "virtualNetworkName": "MyVNET",
      "networkSecurityGroupName": "default-NSG"
    },
    ```

1. 接下来，删除除位置以外的所有参数值，并添加以下参数代码，从而生成以下结果：

    ```json
    "parameters": {
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for all resources."
        }
      },
        "storageAccountName":{
        "type": "string",
        "metadata": {
          "description": "Azure Storage account name."
        }
      }
    },
    ```

1. 接下来，更新输出部分以定义 storageURI 输出值。 storageUri 值是主模板中的虚拟机资源定义所需的。 请将该值作为输出值传回主模板。 修改输出，使其如下所示。

    ```json
    "outputs": {
      "storageUri": {
        "type": "string",
        "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
      }
    }
    ```

1. 最后，确保架构版本为 2019-04-01（如果在 VS Code 中显示，则忽略警告/错误）：

    ```json
        {
          "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
            "storageAccountName":{
              "type": "string",
              "metadata": {
                "description": "Azure Storage account name."
              }
    ```

1. 保存 storage.json 模板。 链接存储模板现在应如下所示：

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "metadata": {
        "_generator": {
          "name": "bicep",
          "version": "0.4.1.14562",
          "templateHash": "8381960602397537918"
        }
      },
      "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        },
        "storageAccountName": {
          "type": "string",
          "metadata": {
            "description": "Azure Storage account name."
          }
        }
    
      },
      "functions": [],
      "variables": {
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2021-04-01",
          "name": "[parameters('storageAccountName')]",
          "location": "[parameters('location')]",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage"
        }
      ],
      "outputs": {
        "storageUri": {
          "type": "string",
          "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
        }
      }
    }
    ```

#### <a name="task-3-upload-linked-template-to-azure-blob-storage-and-generate-sas-token"></a>任务 3：将链接模板上传到 Azure Blob 存储并生成 SAS 令牌

在此任务中，你需要将在上一任务中创建的链接模板上传到 Azure Blob 存储并生成 SAS 令牌，以便在后续部署期间提供对该模板的访问权限。

> **注意**：链接到某个模板时，Azure 资源管理器服务必须能够通过 http 或 https 访问该模板。 为此，请将链接存储模板 storage.json 上传到 Azure 中的 blob 存储。 然后，你将生成一个数字签名 URL，该 URL 提供对该相应 blob 的有限访问权限。 你将使用 Azure Cloud Shell 中的 Azure CLI 执行这些步骤。 或者，可以通过 Azure 门户手动创建 blob 容器，上传文件并生成 URL，或使用实验室计算机上安装的 Azure CLI 或 Azure PowerShell 模块。

1. 在实验室计算机上，启动 Web 浏览器，导航到 [Azure 门户](https://portal.azure.com)，然后使用至少具有将在本实验室中所使用的 Azure 订阅中的参与者角色的用户帐户登录。

1. 在 Azure 门户的工具栏中，单击搜索文本框右侧的“Cloud Shell”图标。

    > **注意**：或者，可以直接导航到 [Azure Cloud Shell](http://shell.azure.com)。

1. 如果系统提示选择“Bash”或“PowerShell”，请选择“PowerShell”  。

    >**注意**：如果这是第一次启动 Cloud Shell，并看到“未装载任何存储”消息，请选择在本实验室中使用的订阅，然后选择“创建存储”  。

1. 在 Cloud Shell 窗格的 PowerShell 会话中，运行以下命令以创建 blob 存储容器，上传你在上一个任务中创建的模板文件，并生成一个 SAS 令牌，你将在主模板中引用该令牌以访问链接模板。
1. 首先，复制并粘贴以下代码行以设置要部署到的 Azure 区域的值。 命令将等待你的输入，如提示符中所示。

    ```powershell
    # Provide the name of the closest Azure region in which you can provision Azure VMs
    $location = Read-Host -Prompt 'Enter the name of Azure region (i.e. centralus)'
    ```

1. 其次，将以下代码复制并粘贴到同一个 Cloud Shell 会话中以创建 blob 存储容器：

    ```powershell
    # This is a random string used to assign the name to the Azure storage account
    $suffix = Get-Random
    $resourceGroupName = 'az400m13l01-RG'
    $storageAccountName = 'az400m13blob' + $suffix

    # The name of the Blob container to be created
    $containerName = 'linktempblobcntr' 

    # A file name used for downloading and uploading the linked template
    $fileName = 'storage.json' 
    
    # Create a resource group
    New-AzResourceGroup -Name $resourceGroupName -Location $location 
    
    # Create a storage account
    $storageAccount = New-AzStorageAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $storageAccountName `
      -Location $location `
      -SkuName 'Standard_LRS'
    
    $context = $storageAccount.Context
    
    # Create a container
    New-AzureStorageContainer -Name $containerName -Context $context
    ```

1. 在 Cloud Shell 窗格中，单击“上传/下载文件”图标，然后在下拉菜单中单击“上传” 。 在“打开”对话框中，导航到 C:\\templates\\storage\\storage.json 并选择该文件，然后单击“打开”  。

      ```powershell
      # Upload the linked template
      Set-AzureStorageBlobContent `
        -Container $containerName `
        -File "$home/$fileName" `
        -Blob $fileName `
        -Context $context

      # Generate a SAS token. We set an expiry time of 24 hours, but you could have shorter values for increased security.
      $templateURI = New-AzureStorageBlobSASToken `
        -Context $context `
        -Container $containerName `
        -Blob $fileName `
        -Permission r `
        -ExpiryTime (Get-Date).AddHours(24.0) `
        -FullUri

      "Resource Group Name: $resourceGroupName"
      "Linked template URI with SAS token: $templateURI"
      ```

  >**注意**：确保记录脚本生成的最终输出。 稍后将在本实验室中用到它。
  
  >**注意**：输出值应如下所示：

  ```
      Resource Group Name: az400m13l01-RG
      Linked template URI with SAS token: https://az400m13blob1677205310.blob.core.windows.net/linktempblobcntr/storage.json?sv=2018-03-28&sr=b&sig=B4hDLt9rFaWHZXToJlMwMjejAQGT7x0INdDR9bHBQnI%3D&se=2020-11-23T21%3A54%3A53Z&sp=r
  ```

  >**注意**：对于需要更高安全级别的场景，你可以在主模板部署过程中动态生成 SAS 令牌，并为 SAS 令牌指定更短的有效期。

1. 关闭 Cloud Shell 窗格。

#### <a name="task-4-modify-the-main-template-to-call-the-linked-template"></a>任务 4：修改主模板以调用链接模板

在此任务中，你将修改主模板，以引用在上一个任务中上传到 Azure Blob 存储的链接模板。

> **注意**：为了负责处理通过模块化所有存储元素对模板结构所做的更改，我们现在需要修改主模板以调用新的存储资源定义。

1. 在 Visual Studio Code 中，单击“文件”顶层菜单，在下拉菜单中选择“打开文件”，在“打开文件”对话框中，导航到 C:\\templates\\azuredeploy.json 并选择该文件，然后单击“打开”   。
1. 在 azuredeploy.json 文件的资源部分，删除存储资源元素

    ```json
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "apiVersion": "2021-04-01",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage"
    },
    ```

1. 接下来，将以下代码直接添加到新删除的存储资源元素所在的位置：

    > **注意**：确保将 `<linked_template_URI_with_SAS_token>` 占位符替换为你在上一个任务结束时记录的实际值。

    ```json
    {
      "name": "linkedTemplate",
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2018-05-01",
      "properties": {
          "mode": "Incremental",
          "templateLink": {
              "uri":"<linked_template_URI_with_SAS_token>"
          },
          "parameters": {
              "storageAccountName":{"value": "[variables('storageAccountName')]"},
              "location":{"value": "[parameters('location')]"}
          }
       }
    },
    ```

1. 查看主模板中的以下详细信息：

    - 主模板中的 Microsoft.Resources/deployments 资源用于链接到另一个模板。
    - 部署资源的名称为 linkedTemplate。 此名称用于配置依赖项。
    - 调用链接的模板时，只能使用增量部署模式。
    - templateLink/uri 包含链接模板的 URI。
    - 请使用参数将值从主模板传递给链接模板。

1. 保存模板。

#### <a name="task-5-modify-main-template-to-update-dependencies"></a>任务 5：修改主模板以更新依赖关系

在此任务中，你将修改主模板以处理需要更新的剩余依赖关系。

> **注意**：由于存储帐户是在链接存储模板中定义的，因此需要更新 Microsoft.Compute/virtualMachines 资源定义。

1. 在虚拟机元素的资源部分，通过替换以下内容来更新 dependsOn 元素：

    ```json
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
      "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]"
    ]
    ```

    替换为

    ```json
    "dependsOn": [
      
      "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]",
      "linkedTemplate"
    ]
      ```

1. 在 Microsoft.Compute/virtualMachines 元素下的资源部分中，通过替换以下内容，重新配置 properties/diagnosticsProfile/bootDiagnostics/storageUri 元素，以反映在链接存储模板中定义的输出值 ：

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
      }
    ```

    替换为

    ```json
    "diagnosticsProfile": {
      "bootDiagnostics": {
        "enabled": true,
        "storageUri": "[reference('linkedtemplate').outputs.storageUri.value]"
      }
    ```

1. 保存更新后的主部署模板。

#### <a name="task-6-deploy-resources-to-azure-by-using-linked-templates"></a>任务 6：通过使用链接模板将资源部署到 Azure

> **注意**：可以通过多种方式来部署模板，例如，直接从 Azure 门户进行部署、使用本地安装的 Azure CLI 或 PowerShell 进行部署或者通过 Azure Cloud Shell 进行部署。 在本实验室中，你将使用 Azure Cloud Shell 中的 Azure CLI。  

> **注意**：若要使用 Azure Cloud Shell，需将主部署模板 azuredeploy.json 上传到 Cloud Shell 的主目录中， 另外，也可以将其上传到 Azure Blob 存储，就像上传链接模板一样，并使用其 URI（而不是本地文件系统路径）来引用它。

1. 在实验室计算机上，在显示 Azure 门户的 Web 浏览器中，单击“Cloud Shell”图标以打开 Cloud Shell。
    > **注意**：如果本练习前面部分的 PowerShell 会话仍处于活动状态，则可以使用该会话，而无需切换到 Bash（下一步）。 以下步骤在 Cloud Shell 的 PowerShell 和 Bash 会话中均可运行。 如果要打开新的 Cloud Shell 会话，请按照说明进行操作。
1. 在“Cloud Shell”窗格中，单击“PowerShell”，在下拉菜单中，单击“Bash”，然后在出现提示时，单击“确认”  。
1. 在 Cloud Shell 窗格中，单击“上传/下载文件”图标，然后在下拉菜单中单击“上传” 。
1. 在“打开”对话框中，导航到 C:\\templates\\azuredeploy.json 并选择该文件，然后单击“打开”  。
1. 在 Cloud Shell 窗格的 Bash 会话中，运行以下命令以使用新上传的模板进行部署：

    ```bash
    az deployment group create --name az400m13l01deployment --resource-group az400m13l01-RG --template-file azuredeploy.json
    ```

1. 当系统提示为“adminUsername”输入值时，键入“Student”并按 Enter 键 。
1. 当系统提示为“adminPassword”输入值时，键入“Pa55w.rd1234”并按 Enter 键 。 （不会显示键入密码）

1. 如果在运行上述命令部署模板时收到错误，请尝试以下操作：

    - 如果有多个 Azure 订阅，请确保已将订阅上下文设置为部署资源组的正确上下文。
    - 确保可以通过指定的 URI 访问链接模板。

> **注意**：下一步，现在可以对主部署模板中的剩余资源定义（例如网络和虚拟机资源定义）进行模块化。

> **注意**：如果不打算使用部署的资源，应删除这些资源来避免产生相关费用。 只需删除资源组 az400m13l01-RG 即可完成此操作。

### <a name="exercise-2-remove-the-azure-lab-resources"></a>练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。

>**注意**：记得删除所有不再使用的新建 Azure 资源。 删除未使用的资源可确保不会出现意外费用。

#### <a name="task-1-remove-the-azure-lab-resources"></a>任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。

1. 在 Azure 门户中，在 Cloud Shell 窗格中打开 Bash Shell 会话 。
1. 运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].name" --output tsv
    ```

1. 通过运行以下命令，删除在此模块的实验室中创建的所有资源组：

    ```bash
    az group list --query "[?starts_with(name,'az400m13l01-RG')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**注意**：该命令以异步方式执行（由 --nowait 参数确定），因此，尽管可立即在同一 Bash 会话中运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## <a name="review"></a>审阅

在本实验室中，你学习了如何创建 Azure 资源管理器模板，使用链接模板对其进行模块化，修改主部署模板以调用链接模板和更新后的依赖关系，以及最终将模板部署到 Azure。
