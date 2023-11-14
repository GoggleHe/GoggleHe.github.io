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

idea环境下老项目，执行单元测试，提示**tests were not found**

- 老项目结构未更新导致
  - 删掉项目下.iml文件，重启idea并reimport该项目maven结构后解决
- 测试编写错误
  - 方法级别必须为public或默认
  - 返回值必须为void
  - @Test注解修饰不能有入参
  - @ParameterizedTest修饰必须有入参