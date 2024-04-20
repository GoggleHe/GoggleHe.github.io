---
layout: wiki
title: vmstat命令详解
categories: [Linux]
description: vmstat命令详解
keywords: Linux, vmstat
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

## vmstat命令详解

### 帮助文档

```bash
Usage:
 vmstat [options] [delay [count]]

Options:
 -a, --active           active/inactive memory 显示活跃和非活跃内存
 -f, --forks            number of forks since boot 显示从系统启动至今的fork数量
 -m, --slabs            slabinfo 显示slabinfo信息
 -n, --one-header       do not redisplay header 在周期性循环输出时，输出的头部信息仅显示一次
 -s, --stats            event counter statistics 显示内存相关统计信息及多种系统活动数量
 -d, --disk             disk statistics 显示磁盘相关统计信息
 -D, --disk-sum         summarize disk statistics 汇总硬盘相关统计信息
 -p, --partition <dev>  partition specific statistics 显示指定磁盘分区统计信息
 -S, --unit <char>      define display unit 使用指定单位显示。例如，k或K表示1000字节，m或M表示1000000字节
 -w, --wide             wide output  更宽的输出信息
 -t, --timestamp        show timestamp 显示时间列

 -h, --help     display this help and exit
 -V, --version  output version information and exit 显示vmstat的版本信息
```

### 输出字段详解

```
FIELD DESCRIPTION FOR VM MODE
Procs
r: The number of runnable processes (running or waiting for run time).
b: The number of processes in uninterruptible sleep.
﻿
Memory
swpd: the amount of virtual memory used.
free: the amount of idle memory.
buff: the amount of memory used as buffers.
cache: the amount of memory used as cache.
inact: the amount of inactive memory.  (-a option)
active: the amount of active memory.  (-a option)
﻿
Swap
si: Amount of memory swapped in from disk (/s).
so: Amount of memory swapped to disk (/s).
﻿
IO
bi: Blocks received from a block device (blocks/s).
bo: Blocks sent to a block device (blocks/s).
﻿
System
in: The number of interrupts per second, including the clock.
cs: The number of context switches per second.
﻿
CPU
These are percentages of total CPU time.
us: Time spent running non-kernel code.  (user time, including nice time)
sy: Time spent running kernel code.  (system time)
id: Time spent idle.  Prior to Linux 2.5.41, this includes IO-wait time.
wa: Time spent waiting for IO.  Prior to Linux 2.5.41, included in idle.
st: Time stolen from a virtual machine.  Prior to Linux 2.6.11, unknown.
﻿
FIELD DESCRIPTION FOR DISK MODE
Reads
total: Total reads completed successfully
merged: grouped reads (resulting in one I/O)
sectors: Sectors read successfully
ms: milliseconds spent reading
﻿
Writes
total: Total writes completed successfully
merged: grouped writes (resulting in one I/O)
sectors: Sectors written successfully
ms: milliseconds spent writing
﻿
IO
cur: I/O in progress
s: seconds spent for I/O
﻿
FIELD DESCRIPTION FOR DISK PARTITION MODE
reads: Total number of reads issued to this partition
read sectors: Total read sectors for partition
writes : Total number of writes issued to this partition
requested writes: Total number of write requests made for partition
﻿
FIELD DESCRIPTION FOR SLAB MODE
cache: Cache name
num: Number of currently active objects
total: Total number of available objects
size: Size of each object
pages: Number of pages with at least one active object
﻿
NOTES
vmstat does not require special permissions.
﻿
These reports are intended to help identify system bottlenecks.  Linux vmstat does not count itself as a running process.
﻿
All linux blocks are currently 1024 bytes.  Old kernels may report blocks as 512 bytes, 2048 bytes, or 4096 bytes.
﻿
Since procps 3.1.9, vmstat lets you choose units (k, K, m, M).  Default is K (1024 bytes) in the default mode.
﻿
vmstat uses slabinfo 1.1
﻿
FILES
/proc/meminfo
/proc/stat
/proc/*/stat
﻿
SEE ALSO
free(1), iostat(1), mpstat(1), ps(1), sar(1), top(1)
﻿
BUGS
Does not tabulate the block io per device or count the number of system calls.
﻿
AUTHORS
Written by Henry Ware ⟨al172@yfn.ysu.edu⟩.
Fabian Frédérick ⟨ffrederick@users.sourceforge.net⟩ (diskstat, slab, partitions...)
﻿
REPORTING BUGS
Please send bug reports to ⟨procps@freelists.org⟩
```

