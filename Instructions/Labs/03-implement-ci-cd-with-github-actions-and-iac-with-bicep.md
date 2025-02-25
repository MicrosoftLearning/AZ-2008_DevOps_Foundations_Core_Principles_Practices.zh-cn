---
lab:
  title: 使用 GitHub Actions 实现 CI/CD 以及通过 Bicep 实现 IaC
  module: Deliver with DevOps
---

# 实验室 03 - 使用 GitHub Actions 实现 CI CD 和使用 Bicep 实现 IaC

## 预计用时：40 分钟

## 场景

请记住本模块的场景：在该场景中你为一家零售行业的软件开发公司工作，该公司希望确保其在线商店应用程序新版本的发布过程高效可靠，同时将错误风险降至最低。 由于你已决定使用 GitHub 来促进应用程序生命周期管理，因此，此实验室让你有机会对 GitHub 存储库创建分支并查看，存储库包含 Web 应用的源代码、GitHub Actions 工作流和 Bicep 模板。 此外，你将能够配置目标环境，并验证基础结构即代码 (IaC) 和 CI/CD 功能。

## 目标

在此实验中，将执行以下操作：

- 为实验室准备 Azure 订阅
- 使用 GitHub Actions 和 Bicep 模板实现基础结构即代码 (IaC) 和 CI/CD

> **注意：** 在上一个实验室中，你配置了 GitHub Pages，这意味着你实际上已经实现了持续部署（可能没有意识到这一点）。 GitHub Pages（在后台）使用 GitHub Actions，以执行每次向主分支提交后的自动部署。 要验证这一点，可以导航到分支存储库“Spoon-Knife”主页上的“操作”选项卡********。 现在，你将增强此功能。 具体而言，你将通过 Bicep 模板使用自定义开发的 GitHub Actions 工作流，在不同的 Azure 区域中预配两个 Azure 应用服务 Web 应用，并将自定义 .NET Web 应用部署到这两个应用。

> **注意：** 对于此实验室和后续实验室，请使用为第一个实验室创建的同一 GitHub 帐户。

## 先决条件

- 已完成的 [实验室 01 - 使用 GitHub 进行敏捷规划和管理](01-agile-planning-management-using-github.md)
- 已完成的 [实验室 02 - 使用 GitHub 实现工作流](02-implement-manage-repositories-using-github.md)
- 需要至少有“参与者级访问权限”的 Azure 订阅。 如果没有 Azure 订阅，可以[注册免费试用版](https://azure.microsoft.com/free)。

## 练习 0：为实验室准备 Azure 订阅

> **注意：** 如果使用新的 Azure 订阅，可能无法注册一些 Azure 资源提供程序。 由于注册过程可能需要几分钟时间，因此为了尽量减少等待时间，你将在此实验室开始时注册它们。

> **注意：** 在此实验室中，你将使用 Azure Cloud Shell。 如果这是首次在订阅中使用 Azure Cloud Shell，则需要注册相应的资源提供程序。 

1. 启动 Web 浏览器，并导航到位于 `https://portal.azure.com` 的 Azure 门户。
1. 如果系统提示，请使用对你可用的 Azure 订阅具有所有者访问权限的 Microsoft Entra ID 帐户登录。
1. 在显示 Azure 门户的 Web 浏览器选项卡中，在页面顶部的搜索文本框中输入 **`Subscriptions`**，然后在结果列表中选择“**订阅**”。
1. 在“订阅”页上，选择要在此实验室中使用的订阅****。
1. 在“订阅”页上左侧的垂直菜单中，选择“资源提供程序”****。
1. 在资源提供程序列表中，搜索并选择“Microsoft.CloudShell”****。
1. 选择“Microsoft.CloudShell”资源提供程序后，在工具栏中选择“注册”********。

   > **注意：** 不要等到注册完成，而是直接转到下一个练习。 注册可能需要几分钟时间。

## 练习 1：使用 GitHub Actions 和 Bicep 模板实现基础结构即代码 (IaC) 和 CI/CD

在本练习中，你将使用 GitHub Actions 和 Bicep 模板，通过 CI/CD 对 Azure 应用服务 Web 应用实施 IaC。

> **注意：** 为了简化初始设置，你将使用现有 GitHub 存储库的分支，其中包含 .NET 应用的源代码、GitHub Actions 工作流和 Bicep 模板。

该练习由以下任务组成：

- 任务 1：创建分支并查看包含 Web 应用的源代码、GitHub Actions 工作流和 Bicep 模板的 GitHub 存储库
- 任务 2：配置目标环境
- 任务 3：验证 IaC 和 CI/CD 功能

### 任务 1：创建分支并查看包含 Web 应用的源代码、GitHub Actions 工作流和 Bicep 模板的 GitHub 存储库

1. 切换到显示 GitHub 帐户的浏览器窗口，并确保仍处于经过身份验证的状态（如果没有，请使用 GitHub 用户帐户登录）。
1. 在同一浏览器窗口中打开另一个选项卡并导航到“eShopOnWeb”存储库，该存储库托管示例网站的 .NET 版本、GitHub Actions 工作流和 Bicep 模板[](https://github.com/MicrosoftLearning/eShopOnWeb)。
1. 在“eShopOnWeb”存储库页上，选择“创建分支”********。
1. 在“创建新分支”页上，确保“所有者”下拉列表条目显示你的 GitHub 用户名，接受“存储库名称”文本框中的默认条目“eShopOnWeb”，并确认选中“仅复制主分支”复选框，然后选择“创建分支”************************。

   > **注意：** 系统会将浏览器会话自动重定向到新创建的分支存储库。

1. 在“eShopOnWeb”页面上，确认下拉列表显示当前分支“主要”********。
1. 查看存储库结构，并注意它包含以下组件：

   - 包含多个基于 YAML 的工作流的“.github/workflows”目录，其中包括一个名为“eshoponweb-cicd.yml”的工作流********
   - 包含名为“webapp.bicep”的 Bicep 模板的“infra”目录********
   - 包含源 Web 应用 .NET 代码的“src/Web”目录****

1. 打开“.github/workflows”目录，然后选择“eshoponweb-cicd.yml”文件以查看其内容********。 请注意，此 GitHub Actions 工作流包含“**buildandtest**”和“**部署**”作业，其中包括以下步骤：

   - buildandtest
      - 签出存储库
      - 为所需的 .NET 版本 SDK 准备运行程序
      - 生成、测试和发布 .NET 项目
      - 上传已发布的网站代码项目
      - 将 bicep 模板作为下一个作业的项目上传
   - 部署：
      - 下载在上一个作业中创建的已发布项目
      - 下载在上一个作业中上传的 bicep 模板
      - 使用服务主体登录到目标 Azure 订阅
      - 使用 bicep 模板部署 Azure 应用服务 Web 应用
      - 将网站项目发布到 Azure 应用服务 Web 应用

   > **注意：** 不要专注于每个作业的详细信息。 此时了解工作流的用途更为重要。

1. 请注意，预配 Azure 资源的过程面向在工作流开头定义为环境变量 `RESOURCE-GROUP` 的单个资源组。

   > **注意：** 在运行工作流之前，需要创建资源组。

1. 请注意，工作流依赖于在“设置 Azure CLI”步骤期间向目标 Azure 订阅进行身份验证的机密，如语句 `creds: ${{ secrets.AZURE_CREDENTIALS }}` 所示。

   > **注意：** 在运行工作流之前，需要设置此机密。

1. 此外，请注意，可以将此工作流配置为手动触发，如 `on: workflow_dispatch:` 语法所示。

### 任务 2：配置目标环境

> **注意：** 首先创建资源组。 你将运行工作流两次，以便在两个不同的 Azure 区域中部署网站的两个实例。

1. 切换到显示位于 `https://portal.azure.com` 的 Azure 门户的 Web 浏览器选项卡。
1. 在 Azure 门户中页面顶部的搜索文本框中，输入 **`Resource groups`**，然后在结果列表中选择“**资源组**”。
1. 在“资源组”页面上，选择“创建”********。
1. 在“**资源组**”文本框中，输入 **`rg-eshoponweb-westeurope`**。
1. 在“区域”下拉列表中，选择“（欧洲）西欧”********。
1. 选择“审阅 + 创建”，然后在“审阅 + 创建”上，选择“创建”************。
1. 在“资源组”页面上，选择“创建”********。
1. 在“**资源组**”文本框中，输入 **`rg-eshoponweb-eastus`**。
1. 在“区域”下拉列表中，选择“（美国）美国东部”********。
1. 选择“审阅 + 创建”，然后在“审阅 + 创建”上，选择“创建”************。

    > **注意：** 接下来，你将创建一个服务主体，用于从 GitHub Actions 工作流向目标 Azure 订阅进行身份验证，并将其分配给订阅中的参与者角色。

1. 在 Azure 门户中，选择搜索文本框右侧的“Cloud Shell”图标****。
1. 如有必要，请在 Cloud Shell 窗格左上角的下拉菜单中选中“Bash”****。
1. 在“Cloud Shell”窗格中的 Bash 会话中，运行以下命令，以将 Azure 订阅 ID 的值存储在变量中：

   ```cli
   SUBSCRIPTION_ID=$(az account show --query id --output tsv) 
   echo $SUBSCRIPTION_ID
   ```

1. 复制第二个命令返回的订阅 ID 值并记录它。 本练习中稍后会用到它。
1. 在“Cloud Shell”窗格中的 Bash 会话中，运行以下命令以创建 Microsoft Entra ID 服务主体，并将其分配给订阅范围内的参与者角色：

   ```cli
   az ad sp create-for-rbac --name "devopsfoundationslabsp" --role contributor --scopes /subscriptions/$SUBSCRIPTION_ID --json-auth
   ```

1. 复制命令的整个 JSON 格式输出并记录它。 稍后需要用到它。 输出的格式应类似于以下文本：

   ```json
   {
     "clientId": "cccccccc-cccc-cccc-cccc-cccccccccccc",
     "clientSecret": "xxxxx~xxxxxxxxxxxx-xxxxxxxxxxxxx_xxxxxxx",
     "subscriptionId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
     "tenantId": "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz",
     "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
     "resourceManagerEndpointUrl": "https://management.azure.com/",
     "activeDirectoryGraphResourceId": "https://graph.windows.net/",
     "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
     "galleryEndpointUrl": "https://gallery.azure.com/",
     "managementEndpointUrl": "https://management.core.windows.net/"
   }
   ```

1. 切换到显示分叉的“eShopOnWeb”GitHub 存储库页的 Web 浏览器窗口，并在工具栏中选择“设置”********。
1. 在左侧垂直菜单“安全”部分中，选择“机密和变量”，然后在下拉列表中选择“操作”************。
1. 在“**操作机密和变量**”窗格中，选择“**新建存储库机密**”。
1. 在“**操作机密/新建机密**”窗格的“**名称**”文本框中，输入 **`AZURE_CREDENTIALS`**。
1. 在“机密”文本框中，粘贴之前在此任务中记录的 JSON 格式文本，然后选择“添加机密”********。
1. 切换回 Web 浏览器，它在“Cloud Shell”窗格中显示 Bash 会话，并运行以下命令以生成要部署的第一个应用服务 Web 应用的名称：

   ```cli
   echo devops-webapp-westeurope-$RANDOM$RANDOM
   ```

1. 复制命令返回的值并记录该值。 你将在本练习后面部分用到它。
1. 在“Cloud Shell”窗格的 Bash 会话中，运行以下命令以生成要部署的第二个应用服务 Web 应用的名称：

   ```cli
   echo devops-webapp-eastus-$RANDOM$RANDOM
   ```

1. 复制命令返回的值并记录该值。 你将在本练习后面部分用到它。

### 任务 3：验证 IaC 和 CI/CD 功能

1. 切换到显示分叉的“eShopOnWeb”GitHub 存储库页的 Web 浏览器窗口，如果需要，请在“eShopOnWeb”页面上单击“代码”选项卡，然后在下拉列表中确认当前分支是“主分支”****************。
1. 在显示分叉的“eShopOnWeb”GitHub 存储库页“主分支”的 Web 浏览器窗口中，导航到“github/workflows”文件夹并选择“eshoponweb-cicd.yml”****************。
1. 在“.github/workflows/eshoponweb-cicd.yml”窗格中，选择铅笔图标以编辑工作流。****
1. 在“编辑”窗格中，将第 4 行替换为以下文本：****

   ```yaml
   on: workflow_dispatch
   ```

    >**注意**：确保使用正确的缩进。

1. 在“编辑”窗格中，将第 8 行替换为以下文本：****

   ```yaml
    RESOURCE-GROUP: rg-eshoponweb-westeurope
   ```

1. 在“编辑”窗格中，将第 11 行中的 `YOUR-SUBS-ID` 占位符替换为在本练习前面记录的 Azure 订阅 ID 的值****：
1. 在“编辑”窗格中****，将第 11 行中的 `eshoponweb-webapp-NAME` 占位符替换为在本练习前面生成的 **第一个** Azure 应用服务 Web 应用的名称。
1. 在“.github/workflows/eshoponweb-cicd.yml”窗格中，选择“提交更改”，然后再次选择“提交更改”************。
1. 在显示分叉的“eShopOnWeb”GitHub 存储库页“主分支”的 Web 浏览器窗口中，导航到“infra”文件夹并选择“webapp.bicep”****************。
1. 在“infra/webapp.bicep”窗格中，选择铅笔图标以编辑工作流。****
1. 在“编辑”窗格中，将第 2 行替换为以下文本：****

   ```yaml
   param sku string = 'S1' // The SKU of App Service Plan
   ```

1. 在“infra/webapp.bicep”窗格中，选择“提交更改”，然后再次选择“提交更改”。************
1. 在显示分叉的“eShopOnWeb”GitHub 存储库页的 Web 浏览器窗口中，选择“操作”********。
1. 如果系统提示你启用 GitHub Actions，请选择“**我了解我的工作流，请继续并启用它们**”。
    >**注意**：这是预期行为，因为默认情况下，GitHub 将在分支存储库中禁用工作流以保护你。
1. 在左侧的“所有工作流”部分中，选择“eShopOnWeb 生成和测试”********。
1. 在“eShopOnWeb 生成和测试”窗格中，选择“运行工作流”，在下拉列表中确认“分支：主”处于选中状态，然后再次选择“运行工作流”****************。

    >**注意**：这应触发工作流运行。

1. 选择“eShopOnWeb 生成和测试”工作流运行。****

    >**注意**：如果需要，请刷新页面以查看最新的工作流运行。

1. 在“eShopOnWeb 生成和测试 #1”窗格中，选择“buildandtest”********。
1. 监视工作流的进度，并验证是否已成功完成“buildandtest”作业的所有步骤****。
1. 完成此作业后，在“摘要”部分中，选择“部署”********。
1. 监视工作流的进度，并验证“**部署**”作业的所有步骤是否已成功完成。

    >**注意**：如果任何步骤失败，请从显示工作流进度的同一页中，在右上角选择“重新运行所有作业”，然后在“重新运行所有作业”窗格中，选择“重新运行作业”************。

1. 切换到显示分叉“eShopOnWeb”GitHub 存储库页的 Web 浏览器窗口，如果需要，在“eShopOnWeb”页面上的下拉列表中确认当前分支是“主分支”************。
1. 在显示分叉的“eShopOnWeb”GitHub 存储库页“主分支”的 Web 浏览器窗口中，导航到“github/workflows”文件夹并选择“eshoponweb-cicd.yml”****************。
1. 在“.github/workflows/eshoponweb-cicd.yml”窗格中，选择铅笔图标以编辑工作流。****
1. 在“编辑”窗格中，将第 8 行替换为以下文本：****

   ```yaml
    RESOURCE-GROUP: rg-eshoponweb-eastus
   ```

1. 在“**编辑**”窗格中，将第 9 行中的`location`变量替换为离你位置最近的区域。 
1. 在“编辑”窗格中****，将第 11 行中的 `eshoponweb-webapp-NAME` 占位符替换为在本练习前面生成的 **第二个** Azure 应用服务 Web 应用的名称。
1. 在“.github/workflows/eshoponweb-cicd.yml”窗格中，选择“提交更改”，然后再次选择“提交更改”************。
1. 在显示分叉的“eShopOnWeb”GitHub 存储库页的 Web 浏览器窗口中，选择“操作”********。
1. 在左侧的“所有工作流”部分中，选择“eShopOnWeb 生成和测试”********。
1. 在“eShopOnWeb 生成和测试”窗格中，选择“运行工作流”，在下拉列表中确认“分支：主”处于选中状态，然后再次选择“运行工作流”****************。

    >**注意**：这应触发工作流运行。

1. 选择“eShopOnWeb 生成和测试”工作流运行。****
1. 在“eShopOnWeb 生成和测试 #2”窗格中，选择“buildandtest”********。
1. 监视工作流的进度，并验证是否已成功完成“buildandtest”作业的所有步骤****。
1. 完成此作业后，在“摘要”部分中，选择“部署”********。
1. 监视工作流的进度，并验证“**部署**”作业的所有步骤是否已成功完成。

    >**注意**：如果任何步骤失败，请从显示工作流进度的同一页中，在右上角选择“重新运行所有作业”，然后在“重新运行所有作业”窗格中，选择“重新运行作业”************。

1. 在显示 Azure 门户的 Web 浏览器窗口中，在页面顶部的搜索文本框中，输入 **`App Services`**，并在结果列表中选择“**应用服务**”。
1. 在“应用服务”页上的应用服务列表中，选择在本练习前面创建的“devops-webapp-westeurope-”应用服务********。
1. 在“devops-webapp-westeurope-”页面上的“Essentials”部分中，验证 是否显示“默认域”值，并选择它以在新浏览器选项卡中打开 Web 应用************。
1. 在新浏览器选项卡中，验证 Web 应用是否显示且功能正常。 还可以通过相同的方式验证“美国东部”区域中的第二个 Web 应用****。
    >**注意**：保留已部署的 Azure 资源处于运行状态。 你需要在下一个实验室使用它们。
