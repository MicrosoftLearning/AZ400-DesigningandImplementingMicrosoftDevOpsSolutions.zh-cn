---
lab:
  title: 实验室 00：验证实验室环境
  module: 'Module 0: Welcome'
ms.openlocfilehash: d5886a54ef4531d68bfa5da28ba2542ddb0cd463
ms.sourcegitcommit: 08d43004a29343fed5b35bf6819ba38b074201f3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/09/2022
ms.locfileid: "139687272"
---
# <a name="lab-00-validate-lab-environment"></a>实验室 00：验证实验室环境
# <a name="student-lab-manual"></a>学生实验室手册

## <a name="instructions"></a>说明

1. 从讲师或其他来源获取新的 Azure Pass 促销代码（有效期为 30 天）。
2. 使用专用浏览器会话从 [Microsoft 帐户](https://account.microsoft.com)获取新的 Microsoft 帐户 (MSA) 或使用现有帐户。
3. 使用相同的浏览器会话，转到 [Microsoft Azure Pass](https://www.microsoftazurepass.com)，然后使用 Microsoft 帐户 (MSA) 兑换 Azure Pass。 有关详细信息，请参阅[兑换 Microsoft Azure Pass](https://www.microsoftazurepass.com/Home/HowTo?Length=5)。 按照说明进行兑换。 
4. 使用相同的浏览器会话，转到 [Microsoft Azure](https://portal.azure.com)，然后在门户屏幕顶部搜索“Azure DevOps”。 在结果页面中，单击“Azure DevOps 组织”。 
5. 接下来，单击标记为“我的 Azure DevOps 组织”的链接或者直接导航到[我的信息](https://aex.dev.azure.com)。
6. 在“我们需要更多详细信息”页上，选择“继续” 。
7. 在左侧下拉框中，选择“默认目录”，而不是“Microsoft 帐户”。
8. 如果系统提示“我们需要更多详细信息”，请提供你的姓名、电子邮件地址以及位置，然后单击“继续”。
9. 回到[我的信息](https://aex.dev.azure.com)并选择默认目录，单击蓝色按钮“创建新组织” 。
10. 单击“继续”即表示接受“服务条款”。
11. 如果系统提示“即将完成”，请将 Azure DevOps 组织名称保留为默认值（该名称需要是全局唯一名称），并在列表中选择靠近你的托管位置。
12. 在 Azure DevOps 中打开新创建的组织后，单击左下角的“组织设置” 。
13. 在“组织设置”屏幕上单击“计费”（打开此屏幕需要几秒钟时间） 。
14. 单击屏幕右侧的“设置计费”，选择“Azure Pass - 赞助”订阅，单击“保存”以将该订阅与组织链接起来  。
15. 当屏幕顶部显示链接的 Azure 订阅 ID 时，将 MS 托管 CI/CD 的付费并行作业数量从 0 更改为 1  。 然后单击底部的“保存”按钮。 
16. 至少等待 3 个小时再使用 CI/CD 功能，以便在后端反映新的设置。 否则，仍会显示消息“此代理未运行，因为已达到最大请求数…”。
17. 在“组织设置”中，转到“安全性”->“策略”
18. 启用“通过 OAuth 进行第三方应用程序访问”
19. 启用“允许公共项目”
20. 可选操作：可以通过创建和触发生成管道来验证新设置是否已成功应用。 为此，请与讲师交谈或使用 [Azure DevOps 演示生成器](https://azuredevopsdemogenerator.azurewebsites.net)在新创建的组织中创建一个演示项目并启用计费。
