---
title: 'vscode项目级配置'
author: ['kyle']
summary: '[此文为随笔随记]通过对项目下的 .vscode 目录添加配置达到项目级配置的目的，这样就不用每个环境都修改配置了'
date: '2025-07-21T14:44:55+08:00'
tags:
- vscode

keywords:
- vscode
- .vscode
---

# settings.json 配置
通过添加 ```.vscode/settings.json``` 来配置 vscode ， 比如使 Markdown 支持代码片段和提示：
```json
{
    "[markdown]": { 
        "editor.quickSuggestions": {
            "other": true,   // 允许代码片段在非注释/字符串区域触发
            "comments": false,
            "strings": false
        }
    }
}
```

# *.code-snippets 代码片段
通过添加 ```.vscode/xxx.code-snippets``` 来添加自定义代码片段：
```json
{
	"Hugo Title": {
		"prefix": "hugo",  // 触发词，如输入"time"后按Tab
		"body": [
			"---",
			"title: '标题'",
			"author: ['kyle']",
			"summary: '简单描述'",
			"date: '${CURRENT_YEAR}-${CURRENT_MONTH}-${CURRENT_DATE}T${CURRENT_HOUR}:${CURRENT_MINUTE}:${CURRENT_SECOND}+08:00'",
			"draft: true",
			"---",
			""
		],
		"description": "hugo md 开头配置"
	}
}
```