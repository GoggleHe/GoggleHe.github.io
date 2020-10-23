---
layout: wiki
title: jmap命令详解
categories: [Java]
description: jmap命令详解
keywords: Java, jmap
---


### -heap

#### 示例

``jmap -heap <PID>`` :查看Java堆的配置信息、使用情况、

#### 输出结果

```
Attaching to process ID 23240, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09

using thread-local object allocation.
Parallel GC with 2 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 1040187392 (992.0MB)
   NewSize                  = 21495808 (20.5MB)
   MaxNewSize               = 346554368 (330.5MB)
   OldSize                  = 43515904 (41.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 25165824 (24.0MB)
   used     = 16072776 (15.328193664550781MB)
   free     = 9093048 (8.671806335449219MB)
   63.86747360229492% used
From Space:
   capacity = 6815744 (6.5MB)
   used     = 5439488 (5.1875MB)
   free     = 1376256 (1.3125MB)
   79.8076923076923% used
To Space:
   capacity = 6815744 (6.5MB)
   used     = 0 (0.0MB)
   free     = 6815744 (6.5MB)
   0.0% used
PS Old Generation
   capacity = 83886080 (80.0MB)
   used     = 42526664 (40.55658721923828MB)
   free     = 41359416 (39.44341278076172MB)
   50.69573402404785% used

18286 interned Strings occupying 1760408 bytes.
```

### -histo

#### 示例

``jmap -histo:live <PID>``

#### 结果

```
 num     #instances         #bytes  class name
----------------------------------------------
   1:         66570        7422224  [C
   2:          7080        6192088  [B
   3:         13810        1972584  [Ljava.lang.Object;
   4:          6030        1725728  [I
   5:         51569        1650208  java.util.HashMap$Node
   6:         65682        1576368  java.lang.String
   7:         10838        1210104  java.lang.Class
   8:         30107         963424  java.util.concurrent.ConcurrentHashMap$Node
   9:          7897         694936  java.lang.reflect.Method
  10:          4002         654080  [Ljava.util.HashMap$Node;
  11:         33452         535232  java.lang.Integer
  12:         24635         394160  java.lang.Object
  13:          8332         333280  java.util.LinkedHashMap$Entry
  14:           554         325424  [Ljava.util.concurrent.ConcurrentHashMap$Node;
  15:           468         307008  io.netty.util.internal.shaded.org.jctools.queues.MpscArrayQueue
  16:           243         233520  [Lio.netty.util.Recycler$DefaultHandle;
```

### -dump

#### 示例

``jmap -dump:live,format=b,file=heap.in 23240``

#### 结果

文件heap.in

### -clstats

#### 示例

``jmap -clstats 23240``

#### 结果

```
Attaching to process ID 23240, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09
class_loader	classes	bytes	parent_loader	alive?	type

<bootstrap>	2692	4869243	  null  	live	<internal>
0x00000000c37da378	1	1474	0x00000000c20880f8	dead	sun/reflect/DelegatingClassLoader@0x000000010000a028
0x00000000c36ef768	1	880	0x00000000c20880f8	dead	sun/reflect/DelegatingClassLoader@0x000000010000a028
0x00000000c2b7e6b0	1	880	0x00000000c20880f8	dead	sun/reflect/DelegatingClassLoader@0x000000010000a028
0x00000000c2b7bd48	1	1473	0x00000000c20880f8	dead	sun/reflect/DelegatingClassLoader@0x000000010000a028

total = 420	9683	15459624	    N/A    	alive=1, dead=419	    N/A    
```



### -finalizerinfo 

#### 示例

``jmap -finalizerinfo 23240``

#### 结果

```
Attaching to process ID 23240, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.201-b09
Number of objects pending for finalization: 0
```

### -F

### -J <flag>