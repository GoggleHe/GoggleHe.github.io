---
layout: wiki
title: jps命令的使用方法
categories: [Java]
description: jps命令的使用方法
keywords: Java,jps
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

## jps命令的使用方法

### 帮助文档

usage: jps [-help]
       jps [-q] [-mlvV] [<hostid>]

Definitions:
    <hostid>:      <hostname>[:<port>]

### 参数详解

-m 输出传递给main方法的参数，在嵌入式jvm上可能是null
$> jps -m
28715 Jps -m
23789 BossMain

-l 输出应用程序main class的完整package名或者应用程序的jar文件完整路径名
$> jps -l
28729 sun.tools.jps.Jps
23789 com.asiainfo.aimc.bossbi.BossMain
23651
23651 Resin -socketwait 32768 -stdout /resin/log/stdout.log -stderr /resin/log/stderr.log
23651 com.caucho.server.resin.Resin

-v 输出传递给JVM的参数
$> jps -v

-V 隐藏输出传递给JVM的参数
$> jps -V