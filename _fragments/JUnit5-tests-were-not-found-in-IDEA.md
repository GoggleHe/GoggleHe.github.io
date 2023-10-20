---
layout: fragment
title: JUnit5 tests were not found in IDEA
tags: [Java, maven, IDEA]
description: some word here
keywords: keyword1, keyword2
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

idea环境下老项目，新增模块，导入JUnit5包后，编写案例，结果提示**tests were not found**

解决方案

- 删掉项目下.iml文件，重启idea并reimport该项目maven结构后解决