---
layout: post
title: jstat命令详解
categories: []
description: jstat命令详解
keywords: 

---


### 格式

- ``jstat -help|-options`` 查看帮助文档
- ``jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`` 使用格式

### 参数详解

- ``-<option>``枚举值
  - class：监视类加载、卸载数量、总空间以及装载所耗费的时间
  - compiler：输出即时编译器编译过的方法、耗时等信息
  - gc：监视java堆情况，包括Eden区、2个survivor区老年代、永久代等的容量，已用空间，垃圾收集时间合计等信息
  - gccapacity：监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大、最小空间
  - gccause：与-gcutil功能一样，但是会额外输出导致上一次垃圾收集产生的原因
  - gcmetacapacity：
  - gcnew：监视新生代垃圾收集情况
  - gcnewcapacity：与-gcnew基本相同，输出主要关注使用到的最大最小空间
  - gcold：监视老年代垃圾收集情况
  - gcoldcapacity：与-gcold基本相同，输出主要关注使用到的最大最小空间
  - gcutil：监视内容与-gc基本相同，但输出主要关注已使用空占总空间的百分比
  - printcompilation：输出已经被即时编译的方法
- -t 输出时间戳
- ``-h<line>``持续采样时，每隔多少行输出一次表头
- interval 采样间隔时间，单位可为ms或s，默认为ms
- count 采样次数

### 示例

![image-20201020161313872](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\image-20201020161313872.png)

输出解释

- S0、S1、E、O、M、CCS分别表示Survivor1、Survivor2、Eden、Old、Metaspace和压缩类空间的内存使用量占比，单位为百分数
- YGC、FGC表示minor gc和full gc回收次数
- YGCT、FGCT、GCT表示minor gc、full gc和总gc耗时，单位为秒

![image-20201020163729301](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\image-20201020163729301.png)

- S0C和S1C等表示各分代的容量，C表示capacity
- S0U和S1U等表示各分代的使用空间，U表示usage
- YGC、FGC表示minor gc和full gc的次数
- YGCT、FGCT、GCT表示垃圾回收使用时间，单位是秒

![image-20201020165939505](C:\Users\EDZ\AppData\Roaming\Typora\typora-user-images\image-20201020165939505.png)

- NGCMN：新生代最小容量
- NGCMX：新生代最大容量
- NGC：当前新生代容量
- S0C：第一个幸存区大小
- S1C：第二个幸存区的大小
- EC：伊甸园区的大小
- OGCMN：老年代最小容量
- OGCMX：老年代最大容量
- OGC：当前老年代大小
- OC:当前老年代大小
- MCMN:最小元数据容量
- MCMX：最大元数据容量
- MC：当前元数据空间大小
- CCSMN：最小压缩类空间大小
- CCSMX：最大压缩类空间大小
- CCSC：当前压缩类空间大小
- YGC：年轻代gc次数
- FGC：老年代GC次数