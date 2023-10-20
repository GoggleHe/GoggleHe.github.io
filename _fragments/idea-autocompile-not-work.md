---
layout: fragment
title: IDEA中maven子模块必须必须使用maven打包命令才执行最新代码
tags: [IDEA]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

idea中maven子模块必须执行mvn clean package才会执行新代码，直接run不能执行最新逻辑

解决方案：

依次进入`setting` - `build` - `compile` 菜单，勾选`compile independent module in parallel`

