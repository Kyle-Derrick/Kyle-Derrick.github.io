---
title: 'hugo从同级目录下插入图片（相对路径）'
author: ['kyle']
description: '通过修改 render_image.html 调整 img 生成逻辑以达到从相对路径插入图片的目的'
date: '2025-07-21T19:40:14+08:00'
tags:
- hugo

keywords:
- hugo
- 图片引用
#hidemeta: true
---

# hugo图片引入问题
> 我在使用hugo写笔记的时候遇到了插入图片出现链接失效问题，因为在hugo中 xxx.md 生成html后会生成 xxx 目录，然后再 xxx 目录下生成 index.html ，因此编写md时的图片引用路径在生成html后边的不可用了；
> 

网上查到的解决方法有两种：
1. 图片放入 /static 目录下，然后通过绝对路径引入
2. 将 xxx.md 调整为 xxx/index.md 然后使用相对路径引用图片

但是：方法一需要把所有图片放到 /static 下，需要单独维护；方法二会导致出现大量目录和 index.md 文件，目录就会非常乱。

故而想了一种办法：通过修改md生成html时图片渲染处理逻辑，以支持相对路径图片引用

> 修改 render_image.html (/layouts/_default/_markup/render_image.html)
> 
> 默认可能没有，我的是从 PaperMod 主题里复制的

```html
{{- $u := urls.Parse .Destination -}}
{{- $src := $u.String -}}
{{- if not $u.IsAbs -}}
  {{- $path := strings.TrimPrefix "./" $u.Path }}
  {{- with or (.PageInner.Resources.Get $path) (resources.Get $path) -}}
    {{- $src = .RelPermalink -}}
    {{- with $u.RawQuery -}}
      {{- $src = printf "%s?%s" $src . -}}
    {{- end -}}
    {{- with $u.Fragment -}}
      {{- $src = printf "%s#%s" $src . -}}
    {{- end -}}
  {{/* 新增逻辑：非 index.md 的本地相对路径添加 ../ */}}
  {{- else -}}
    {{- if and (not (hasPrefix $u.Path "/")) (ne .PageInner.File.LogicalName "index.md") -}}
      {{- $src = printf "../%s" $path -}}
    {{- end -}}
  {{- end -}}
  {{- 一直到这里 -}}
{{- end -}}
{{- $attributes := merge .Attributes (dict "alt" .Text "src" $src "title" (.Title | transform.HTMLEscape) "loading" "lazy") -}}
<img
  {{- range $k, $v := $attributes -}}
    {{- if $v -}}
      {{- printf " %s=%q" $k $v | safeHTMLAttr -}}
    {{- end -}}
  {{- end -}}>
{{- /**/ -}}

```

> 这样你就可以直接在 xxx.md 中直接使用 img/xxx_1.1.png 引入图片了