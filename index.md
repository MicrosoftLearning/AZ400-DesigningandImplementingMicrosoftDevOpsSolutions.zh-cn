---
title: 联机托管说明
permalink: index.html
layout: home
ms.openlocfilehash: b620a0c228333f5d50f55bd3e2238ccdd6dd8472
ms.sourcegitcommit: d8f0a6eb53689d119ce8cf3a4416ad2c80cf919d
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/27/2022
ms.locfileid: "137894647"
---
# <a name="content-directory"></a>内容目录

下面列出了每个实验室练习的超链接。

## <a name="labs"></a>实验室

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 模块 | 实验室 |
| --- | --- | 
{% 表示实验室 % 中的活动}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}


