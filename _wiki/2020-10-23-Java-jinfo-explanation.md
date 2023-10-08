---
layout: wiki
title: jinfo命令详解
categories: [Java]
description: jinfo命令详解
keywords: Java, jinfo
---

### -flags

输出全部JVM参数

#### 示例

``jinfo -flags 23240``

#### 结果

```
Server compiler detected.
JVM version is 25.201-b09
Non-default VM flags: -XX:CICompilerCount=2 -XX:InitialHeapSize=65011712 -XX:MaxHeapSize=1040187392 -XX:MaxNewSize=346554368 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=21495808 -XX:OldSize=43515904 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
Command line:
```

### -sysprops

输出全部系统参数

#### 示例

``jinfo -sysprops <PID>``

#### 结果

```
Attaching to process ID 23240, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09
java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 25.201-b09
sun.boot.library.path = /usr/local/jdk1.8.0_201/jre/lib/amd64
java.protocol.handler.pkgs = org.springframework.boot.loader
....
....
```

### -flag 

#### 示例

- ``jinfo -flag <name> <PID>`` 查询指定JVM参数
- ``jinfo -flag [+|-]<name> <PID>`` 动态开启或关闭JVM配置
- ``jinfo -flag <name>=<value> <PID>`` 动态设置JVM配置的值

#### 结果

```
-XX:CICompilerCount=2
```

### 默认

展示全部JVM参数和系统属性