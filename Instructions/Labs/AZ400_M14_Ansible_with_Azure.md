---
lab:
    title: '实验室 14a： Ansible 与 Azure'
    module: '模块 14：通过 Azure 提供的第三方基础结构即代码工具'
---

# 实验室 14a： Ansible 与 Azure
# 学生实验室手册

## 实验室概述

在本实验室中，我们将使用 Ansible 部署、配置和管理 Azure 资源。 

Ansible 是声明性配置管理软件。它依赖于托管计算机的预期配置描述，这种描述通常采用 playbook 形式。Ansible 自动应用该配置并维护它持续运行，从而解决任何潜在差异。Playbook 使用 YAML 进行格式设置。

与大多数其他配置管理工具（例如 Puppet 或 Chef）不同，Ansible 是无代理的，这意味着它无需在托管计算机上安装任何软件。Ansible 使用 SSH 来管理 Linux 服务器，使用 Powershell Remoting 来管理 Windows 服务器和客户端。

为了与资源而非操作系统（例如可通过 Azure 资源管理器访问的 Azure 资源）进行交互，Ansible 支持称为模块的扩展。因此可使用 Python 高效编写 Ansible，并将模块作为 Python 库实现。Ansible 依赖于 [GitHub 托管模块](https://github.com/ansible-collections/azure)来管理 Azure 资源。

Ansible 要求在指定的主机清单中指定托管资源。对于某些系统（包括 Azure），Ansible 支持动态清单，以便在运行时动态生成主机清单。

本实验室将包括以下概要步骤：

- 在 Azure VM 上安装和配置 Ansible
- 下载 Ansible 配置和示例 playbook 文件
- 在 Azure AD 中创建和配置托管标识
- 配置 Azure AD 凭据和 SSH 以用于 Ansible
- 使用 Ansible playbook 部署 Azure VM
- 使用 Ansible playbook 配置 Azure VM

## 目标

完成本实验室后，你将能够：

- 在 Azure VM 上安装和配置 Ansible
- 下载 Ansible 配置和示例 playbook 文件
- 创建和配置 Azure Active Directory 托管标识
- 配置 Azure AD 凭据和 SSH 以用于 Ansible
- 使用 Ansible playbook 部署 Azure VM
- 使用 Ansible playbook 配置 Azure VM

## 实验室持续时间

-   预计用时：**90 分钟**

## 说明

### 准备工作

#### 登录到实验室虚拟机

确保已使用以下凭据登录到 Windows 10 计算机：
    
-   用户名：**Student**
-   密码：**Pa55w.rd**

#### 查看本实验室所需的应用程序

确定你将在本实验室中使用的应用程序：
    
-   Microsoft Edge

#### 准备 Azure 订阅

-   标识现有的 Azure 订阅或创建一个新的 Azure 订阅。
-   验证你拥有 Microsoft 帐户或 Azure AD 帐户，该帐户在 Azure 订阅中具有所有者角色并且在与 Azure 订阅关联的 Azure AD 租户中具有全局管理员角色。有关详细信息，请参阅[使用 Azure 门户列出 Azure 角色分配](https://docs.microsoft.com/zh-cn/azure/role-based-access-control/role-assignments-list-portal)和[在 Azure Active Directory 中查看和分配管理员角色](https://docs.microsoft.com/zh-cn/azure/active-directory/roles/manage-roles-portal#view-my-roles)。

### 练习 1：使用 Ansible 部署、配置和管理 Azure VM

在本练习中，你将使用 Ansible 部署、配置和管理 Azure VM。

#### 任务 1：预配充当 Ansible 控制节点的 Azure VM

在此任务中，你将使用 Azure CLI 部署一个 Azure Vm，并将它配置为负责管理 Ansible 环境的 Ansible 控制节点。

>**备注**： 你将使用配置为 Ansible 控制节点的 Azure VM 来执行 Ansible 管理任务，包括你在本实验室之前的任务中执行的操作。 

1.  在 Azure 门户的工具栏中，单击搜索文本框右侧的 **“Cloud Shell”** 图标。 

    >**备注**： 或者，可以直接通过导航到 [https://shell.azure.com](https://shell.azure.com) 来访问 Cloud Shell。

1.  如果提示选择 **“Bash”** 或 **“PowerShell”**，请选择 **“Bash”**。 

    >**备注**：如果这是第一次启动 **Cloud Shell**，并显示消息 **“未装载任何存储”**，请选择你将在本实验室中使用的订阅，然后选择 **“创建存储”**。 

1.  在 Cloud Shell 窗格中的 Bash 会话中，运行以下命令来指定 Azure 区域的名称，该区域将托管你在本实验室中部署的资源（将 `<Azure_region>` 占位符替换为你要在其中部署资源的 Azure 区域的名称）。请确保名称中不包含任何空格，例如 `westeurope`）：

    ```bash
    LOCATION=<Azure_region>
    ```

1.  在 Cloud Shell 窗格中的 Bash 会话中运行以下命令，创建将托管你在本实验室中部署的 Azure VM 的资源组：

    ```bash
    RG1NAME=az400m14l03rg
    az group create --name $RG1NAME --location $LOCATION
    RG2NAME=az400m14l03arg
    az group create --name $RG2NAME --location $LOCATION
    ```

1.  运行以下命令，将运行 Ubuntu 的 Azure VM 部署到你在上一步中创建的资源组中：

    ```bash
    VM1NAME=az400m1403vm1
    az vm create \
    --resource-group $RG1NAME \
    --name $VM1NAME \
    --image UbuntuLTS \
    --authentication-type password \
    --admin-username azureuser \
    --admin-password Pa55w.rd1234
    ```

    >**备注**：请等待部署完成，再继续下一步。该过程大约需要 2 分钟。

    >**备注**： 预配完成后，在基于 JSON 的输出中，标识该输出中包含的 **“publicIpAddress”** 属性的值。 

1.  运行以下命令，使用 SSH 连接到新部署的 Azure VM：

    ```bash
    PIP=$(az vm show --show-details --resource-group $RG1NAME --name $VM1NAME --query publicIps --output tsv)
    ssh azureuser@$PIP
    ```

1.  出现确认继续操作的提示时，键入 **“是”** 并按 **Enter** 键，然后在提示提供密码时，键入 **Pa55w.rd1234**。


#### 任务 2：在 Azure VM 上安装和配置 Ansible

在此任务中，你将在上一个任务中部署的 Azure VM 上安装和配置 Ansible。

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令，以更新高级打包工具 (apt) 包列表，在其中包含最新版本和包详细信息：

    ```bash
    sudo apt-get update
    ```

1.  运行以下命令，安装 Ansible 和所需的 Azure 模块（**确保逐行单独运行命令**；每当系统提示确认时，都请键入 **“y”** 并按 **Enter** 键）：

    ```bash
    sudo apt install python3-pip
    sudo -H pip3 install --upgrade pip
    sudo -H pip3 install ansible[azure]
    sudo apt-add-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible
    sudo ansible-galaxy collection install azure.azcollection
    curl -O https://raw.githubusercontent.com/ansible-collections/azure/dev/requirements-azure.txt
    sudo pip3 install -r requirements-azure.txt
    rm requirements-azure.txt
    ```

    >**备注**： 忽略任何警告。如果遇到任何错误，请重新运行命令。

1.  运行以下命令来安装 dnspython 包，使 Ansible playbook 可在部署之前验证 DNS 名称：

    ```bash
    sudo -H pip3 install dnspython
    ```

1.  在 Cloud Shell 窗格的 Bash 会话中，转到新部署的 Azure VM 的 SSH 会话，运行以下命令来安装 JSON 分析工具 **jq** （出现提示时，请输入 **“y”** 并按 **Enter** 键）：

    ```bash
    sudo apt install jq
    ```

1.  运行以下命令来安装 Azure CLI：

    ```bash
    curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
    ```

#### 任务 3：下载 Ansible 配置和示例 playbook 文件

在此任务中，你将从 GitHub 中下载 Ansible 配置存储库和示例实验室文件。 

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以确保已安装 **git**：

    ```bash
    sudo apt install git
    ```

1.  运行以下命令，从 GitHub 中克隆 PartsUnlimitedMRP 存储库：

    ```bash
    git clone https://github.com/Microsoft/PartsUnlimitedMRP.git
    ```

    >**备注**： 此存储库包含用于创建各种资源的 playbook，我们将在实验室中使用其中一些 playbook。


#### 任务 4：创建和配置 Azure Active Directory 托管标识

在此任务中，你将生成一个 Azure AD 托管标识来帮助执行 Ansible 的非交互式身份验证（这是访问 Azure 资源的必需操作）。你还要将托管标识分配到你在上一任务中创建的资源组上的“参与者”角色。

1.  在 Cloud Shell 窗格的 Bash 会话中，在新部署的 Azure VM 的 SSH 会话中，运行以下命令以登录与你的 Azure 订阅关联的 Azure AD 租户：

    ```bash
    az login
    ```

    >**备注**： 如果命令失败，请重新运行 Azure CLI 的安装。

1.  记下上一命令的输出中显示的代码，并切换到实验室计算机。从实验室计算机，在显示 Azure 门户的浏览器窗口中打开新的标签页，导航到 [Microsoft 设备登录页面](https://microsoft.com/devicelogin)，出现提示时输入代码，然后选择 **“下一步”**。

1.  在出现提示时，使用在本实验室中所用的凭据登录，然后关闭浏览器标签页。

1.  切换回到 Cloud Shell 窗格中的 Bash 会话。在配置为 Ansible 控制节点的 Azure VM 的 SSH 会议中，运行以下命令来生成系统分配的托管标识：


    ```bash
    RG1NAME=az400m14l03rg
    VM1NAME=az400m1403vm1
    az vm identity assign --resource-group $RG1NAME --name $VM1NAME
    ```

1.  运行以下命令来确定订阅的值：

    ```bash
    SUBSCRIPTIONID=$(az account show --query id --output tsv)
    ```

1.  运行以下命令，检索内置的“Azure 基于角色的访问控制参与者”角色的 **ID** 属性的值： 

    ```bash
    CONTRIBUTORID=$(az role definition list --name "Contributor" --query "[].id" --output tsv)
    ```

1.  运行以下命令，分配你先前在本实验室中创建的资源组上的“参与者”角色。 

    ```bash
    MIID=$(az resource list --name $VM1NAME --query [*].identity.principalId --out tsv)

    RG2NAME=az400m14l03arg
    az role assignment create --assignee "$MIID" \
    --role "$CONTRIBUTORID" \
    --scope /subscriptions/$SUBSCRIPTIONID/resourceGroups/$RG2NAME
    ```

#### 任务 5：配置 SSH 来与 Ansible 一起使用

在此任务中，你将配置 SSH 来与 Ansible 一起使用。

1.  在 Cloud Shell 窗格的 Bash 会话中，转到新部署的 Azure VM 的 SSH 会话，运行以下命令来生成密钥对（出现提示时，按 **Enter** 键三次来接受文件位置的默认值且不设置密码）：

    ```bash
    ssh-keygen -t rsa
    ```

1.  运行以下命令，授予对托管私钥的 **.ssh** 文件夹的读取、写入和执行权限：

    ```bash
    chmod 755 ~/.ssh
    ```

1.  运行以下命令，创建并设置对 **authorized_keys** 文件的读取和写入权限。

    ```bash
    touch ~/.ssh/authorized_keys
    chmod 644 ~/.ssh/authorized_keys
    ```

    >**备注**： 通过提供此文件中包含的密钥，你无需提供密码即可访问此文件。

1.  运行以下命令，将密码添加到 **authorized_keys** 文件中：

    ```bash
    ssh-copy-id azureuser@127.0.0.1
    ```

1. 在出现提示时，键入 **“是”**，然后为 **azureuser** 用户帐户（之前在本实验室中部署第三个Azure VM 时指定的帐户）输入密码 **Pa55w.rd1234**。 
1.  运行以下命令，验证系统是否未提示输入密码：

    ```bash
    ssh 127.0.0.1
    ```

1.  键入 **“退出”** 并按 **Enter** 键，终止你刚才建立的环回连接。 

>**备注**： 建立无密码 SSH 身份验证是设置 Ansible 环境的一个关键步骤。 


#### 任务 6：使用 Ansible playbook 创建 Web 服务器 Azure VM

在此任务中，你将使用 Ansible playbook 创建托管 Web 服务器的 Azure VM。 

>**备注**： 在控制 Azure VM 中启动并运行 Ansible 后，我们现在可以部署第一个 playbook，以便创建和配置托管 Azure VM。在部署示例 playbook 之前，你需要将包含在其内容中的公共 SSH 密钥替换为在上一个任务中生成的密钥。 

1.  在 Cloud Shell 窗格的 Bash 会话中，转到配置为 Ansible 控制节点的 Azure VM 的 SSH 会话，运行以下命令来标识你在上一任务中生成且存储在本地的公钥：

    ```bash
    cat ~/.ssh/id_rsa.pub
    ```

1.  记录输出，包括输出字符串结尾的用户名。 
1.  运行以下命令，在代码文本编辑器中打开“**new_vm_web.yml**”文件：

    ```bash
    code ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml
    ```

1.  在代码编辑器中，如果需要，将 `dnsname: '{{ vmname }}.westeurope.cloudapp.azure.com'` 条目中的区域名称更改为要在其中进行部署的 Azure 区域的名称。 

    >**备注**： 请确保此区域与你在 **az400m14l03rg** 资源组中创建的 Azure 区域相匹配。

1.  在代码编辑器中，将 `vm_size` 条目的值从 `Standard_A0` 更改为 `Standard_DS1_v2`。
1.  在代码编辑器中，找到文件末尾的 SSH 字符串，在 `key_data` 条目中，删除现有键值，并将其替换为在本任务前面部分记录的键值。 

    >**备注**： 确保文件中包含的 `admin_username` 条目的值与用于登录托管 Ansible 控制系统的 Azure VM 的用户名 (**azureuser**)一致。`Ssh_public_keys` 部分的 `path` 条目必须使用相同的用户名。

1.  在代码编辑器界面中，单击右上方的“**...**”，然后选择“**保存**”。

    >**备注**： 接下来，你将在本实验室开头时创建的资源组中部署一个 Azure VM：使用以下值进行部署： 

    | 设置 | 值 |
    | --- | --- |
    | 资源组 | **az400m14l03arg** |
    | 虚拟网络 | **az400m1403aVNET** |
    | 子网 | **az400m1403aSubnet** |

    >**备注**： 可在 playbook 中定义这些变量，也可在通过包含 `--extra-vars` 选项来调用 `ansible-playbook` 命令时，在运行时输入这些变量。对于 VM 名称，仅使用小写字母和数字（不使用连字符、下划线符号或大写字符）且长度不超过 15 个字符，同时确保它是全局唯一的，因为该名称用于为与相应的 Azure VM 关联的公共 IP 地址生成存储帐户和 DNS 名称。 

1.  运行以下命令，创建你将在其中使用 Ansible playbook 部署 Azure VM 的虚拟网络及其子网。

    ```bash
    RG1NAME=az400m14l03arg
    LOCATION=$(az group show --resource-group $RG1NAME --query location --output tsv)
    RG2NAME=az400m14l03arg
    VNETNAME=az400m1403aVNET
    SUBNETNAME=az400m1403aSubnet
    az network vnet create \
    --name $VNETNAME \
    --resource-group $RG2NAME \
    --location $LOCATION \
    --address-prefixes 192.168.0.0/16 \
    --subnet-name $SUBNETNAME \
    --subnet-prefix 192.168.1.0/24
    ```

1.  运行以下命令，部署用于预配 Azure VM 的示例 Ansible playbook（确保将 `<VM_name>` 替换为你选择的唯一 VM 名称）：

    ```bash
    sudo ansible-playbook ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/new_vm_web.yml --extra-vars "vmname=<VM_name> resgrp=az400m14l03arg vnet=az400m1403aVNET subnet=az400m1403aSubnet"
    ```

    >**备注**： 忽略与 ip_configuration 设置相关的弃用警告。

    >**备注**： 如果输入现有或无效的 VM 名称，则可能会遇到以下错误：

    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "The storage account named storageaccountname is already taken.- Reason.already_exists"}`. 要解决此问题，请为 Azure VM 使用其他名称，因为你使用的名称不是全局唯一的。
    - `fatal: [localhost]: FAILED! => {"changed": false, "failed": true, "msg": "Error creating or updating your-vm-name - Azure Error: InvalidDomainNameLabel\nMessage: The domain name label for your VM is invalid.It must conform to the following regular expression: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$.”}`。要解决此问题，请按照要求的命名约定，为 Azure VM 使用其他名称。 

    >**备注**： 等待部署完成。该操作需要约 3 分钟。 

1.  运行以下命令，创建一个名为“**myazure_rm.yml**”的新文件，并在代码文本编辑器中打开它：

    ```bash
    code ./myazure_rm.yml
    ```

1.  在代码编辑器界面中，粘贴以下内容：

    ```bash
    plugin: azure_rm
    include_vm_resource_groups:
    - az400m14l03arg
    auth_source: msi

    keyed_groups:
    - prefix: tag
      key: tags
    ```

1.  在代码编辑器界面中，单击右上方的“**...**”，然后选择“**保存**”。
1.  回到 Cloud Shell 窗格的 Bash 会话，在与配置为 Ansible 控制节点的 Azure VM 的 SSH 会话中，运行以下命令来执行 ping 测试，验证动态清单文件是否包含新部署的 Azure VM：

    ```bash
    sudo ansible --user azureuser --private-key=/home/azureuser/.ssh/id_rsa all -m ping -i ./myazure_rm.yml
    ```

1.  当系统提示你是否要继续连接时，请键入 **“是”** 并按 **Enter** 键。

    >**备注**： 输出应如下所示：

    ```bash
    az400m1403vm2_5444 | SUCCESS => {
        "ansible_facts": {
            "discovered_interpreter_python": "/usr/bin/python"
        },
        "changed": false,
        "ping": "pong"
    }
    ```

    >**备注**： 第一次运行该命令时，你需要确认目标 VM 的真实性，方法是键入 **“是”**，然后按 **Enter** 键。 


#### 任务 7：使用 Ansible playbook 配置 Azure VM

在此任务中，你将运行另一个 Ansible playbook，这次是配置新部署的 Azure VM。你将使用一个 playbook，它会安装软件包 http 并从 GitHub 存储库下载 HTML 页面。完成此操作后，你将有一个功能完善的 Web 服务器。

>**备注**： 我们将使用示例 playbook **“~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml”**。我们将使用变量 **vmname** 来修改 playbook 的主机参数，该参数用于定义 playbook 将面向的具体主机（包含在动态清单脚本返回的主机中）。 

1.  在 Cloud Shell 窗格的 Bash 会话中，转到配置为 Ansible 控制节点的 Azure VM 的 SSH 会话，运行以下命令来确定新部署的 Azure VM 的公共 IP 地址（**务必将`<VM_name>`占位符替换为你分配给新预配的 Azure VM 的名称**）：

    ```bash
    RGNAME='az400m14l03arg'
    VMNAME='<VM_name>'
    PIP=$(az vm show --show-details --resource-group $RGNAME --name $VMNAME --query publicIps --output tsv)
    ```

1.  运行以下命令，验证新部署的 Azure VM 当前是否未在运行任何 Web 服务（其中，`<IP_address>`占位符表示分配给你在上一任务中预配的 Azure VM 的网络适配器的公共 IP 地址）：

    ```bash
    curl http://$PIP
    ```

    >**备注**：验证响应的格式是否为 `curl: (7) Failed to connect to 52.186.157.26 port 80: Connection refused`。

1.  运行以下命令，使用 Ansible playbook 安装 HTTP 服务（其中，`<VM_name>`占位符表示你在上一任务中预配的 VM 的名称）： 

    ```bash
    sudo ansible-playbook --user azureuser --private-key=/home/azureuser/.ssh/id_rsa -i ./myazure_rm.yml ~/PartsUnlimitedMRP/Labfiles/AZ-400T05-ImplemntgAppInfra/Labfiles/ansible/httpd.yml --extra-vars "vmname=<VM_name>*"
    ```

    >**备注**： 请务必在 Azure VM 名称后面追加尾随星号 (**\***)。

    >**备注**： 请等待安装完成。所需时间应该不超过一分钟。 

1.  安装完成后，请运行以下命令，验证新部署的 Azure VM 当前是否正在运行 Web 服务（其中，`<IP_address>` 占位符表示分配给你在上一任务中预配的 Azure VM 的网络适配器的公共 IP 地址）：

    ```bash
    curl http://$PIP
    ```

    >**备注**： 输出应具有以下内容： 

    ```html
     <!DOCTYPE html>
     <html lang="en">
         <head>
             <meta charset="utf-8">
             <title>Hello World</title>
         </head>
         <body>
             <h1>Hello World</h1>
             <p>
                 <br>This is a test page
                 <br>This is a test page
                 <br>This is a test page
             </p>
         </body>
     </html>
    ```

### 练习 2：删除 Azure 实验室资源

在本练习中，你将删除在本实验室中预配的 Azure 资源，避免产生意外费用。 

>**备注**： 请记得删除不再使用的所有新创建的 Azure 资源。删除未使用的资源，确保不产生意外费用。

#### 任务 1：删除 Azure 实验室资源

在此任务中，你将使用 Azure Cloud Shell 删除在本实验室中预配的 Azure 资源，避免产生不必要的费用。 

1.  在 Azure 门户中，在 **Cloud Shell** 窗格中打开 **Bash** Shell 会话。
1.  运行以下命令，列出在本模块各实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].name" --output tsv
    ```

1.  运行以下命令，删除在本模块各个实验室中创建的所有资源组：

    ```sh
    az group list --query "[?starts_with(name,'az400m14l03')].[name]" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
    ```

    >**备注**：该命令以异步方式执行（由 --nowait 参数决定），因此，虽然你随后可在同一 Bash 会话中立即运行另一个 Azure CLI 命令，但实际上要花几分钟才能删除资源组。

## 回顾

在本实验室中，你已了解如何使用 Ansible 部署、配置和管理 Azure 资源。 

