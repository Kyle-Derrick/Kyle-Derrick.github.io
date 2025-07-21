---
title: 'hugo使用经验'
author: ['kyle']
summary: 'hugo使用过程中的一些经验记录'
date: '2025-07-21T15:42:35+08:00'
tags:
- hugo

keywords:
- hugo
#hidemeta: true
---

#  杂项记录点

## 1. 命令
* 编写时可启动 server 随时观察实际显示效果：
```sh
hugo server
```
* 编译
```sh
hugo --gc --minify --cleanDestinationDir
```

## 2. md开头的参数

1. date  
   • 文章发布日期，格式为 YYYY-MM-DDTHH:mm:ss+08:00（ISO 8601）。  
   • 用途：决定文章在时间轴中的位置，影响排序和归档路径。  
   • 示例：date: 2025-07-21T14:30:00+08:00

2. draft  
   • 布尔值（true/false），标记是否为草稿。  
   • 用途：若为 true，默认情况下 hugo 命令不会生成该文章，需添加 --buildDrafts 参数才能编译。  
   • 示例：draft: true

3. description  
   • 文章摘要或描述，通常用于 SEO 和列表页预览。  
   • 示例：description: "Hugo Front Matter 参数详解"

4. author  
   • 指定作者姓名，可在模板中调用显示。  
   • 示例：author: "Your Name"

5. image  
   • 文章封面或特色图片路径（支持本地或外部 URL）。  
   • 示例：image: "/images/featured.jpg"

6. summary  
   • 手动指定文章摘要，覆盖自动生成的摘要。  
   • 示例：summary: "本文详细解析 Hugo Front Matter 的所有参数"

7. toc  
   • 布尔值（true/false），控制是否生成目录（Table of Contents）。  
   • 示例：toc: true

8. weight  
   • 数值，控制文章在列表中的排序权重（数值越小越靠前）。  
   • 示例：weight: 10

9. hiddenFromHomePage / hiddenFromSearch  
   • 布尔值，控制文章是否在首页或搜索结果中隐藏。  
   • 示例：hiddenFromHomePage: true

10. math  
   • 布尔值，启用数学公式支持（需主题兼容 LaTeX）。  
   • 示例：math: true

12. lightgallery  
   • 布尔值，启用图片灯箱效果（依赖主题实现）。  
   • 示例：lightgallery: true

13. aliases  
   • 数组，设置文章别名（旧 URL 重定向）。  
   • 示例：  
     aliases:  
       - "/old-path/post.html"


```yaml
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
lastmod: {{ .Date }}
author: ["AInfinity"]

categories:
- category 1
- category 2

tags:
- tag 1
- tag 2

keywords:
- word 1
- word 2

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: true # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
```