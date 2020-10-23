---
layout: wiki
title: top命令详解
categories: [Linux]
description: top命令详解
keywords: Linux, top
---


### 格式

``top -hv | -bcHiOSs -d secs -n max -u|U user -p pid(s) -o field -w [cols]``

基于版本3.3.10

### 参数解析

- -hv：查看帮助信息和版本
- -d secs ：设定刷新速度，单位秒
- -n max ：统计次数
- -u|U user：指定用户
- -p pid(s)：指定进程id，多个用半角逗号分隔
- -o field：指定排序字段
- -w [cols]：
- -b：以批处理形式输出
- -c：展示完整命令
- -H：监控线程
- -i：忽略失效进程
- -O：打出所有字段名称
- -S：
- -s：

### 输出解析

```
top - 17:02:25 up 45 days, 19:48,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  76 total,   2 running,  74 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.1 us,  1.1 sy,  0.0 ni, 96.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882028 total,   368556 free,   394528 used,  1118944 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  1299464 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                      
 1824 root      10 -10  133888  15828  10252 S  1.1  0.8 292:43.56 AliYunDun               
30764 root      20   0  162004   2192   1552 R  1.1  0.1   0:00.02 top   
    1 root      20   0  125468   3792   2440 S  0.0  0.2   4:15.29 systemd
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.35 kthreadd
    4 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    6 root      20   0       0      0      0 S  0.0  0.0   0:16.21 ksoftirqd/0
    7 root      rt   0       0      0      0 S  0.0  0.0   0:00.00 migration/0
    8 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh  
    9 root      20   0       0      0      0 S  0.0  0.0  15:27.89 rcu_sched   
   10 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 lru-add-drain 
```

- top
  - ``- 17:02:25`` ：当前时间
  - up 系统运行时间
  - user 用户连接数量
  - load average：最近1min、5min、15min平均负载，可理解为进程的队列长度
- Tasks：进程概况
  - 总进程数
  - 运行中进程数
  - 休眠进程数
  - 终止进程数
  - 僵尸进程数
- %Cpu(s)：
  - us 用户空间占用CPU百分比
  - sy 内核空间占用CPU百分比
  - ni 用户进程空间内改变过优先级的进程占用CPU百分比
  - id 空闲CPU百分比
  - wa 等待输入输出的CPU时间百分比
  - hi 硬件CPU中断占用百分比
  - si 软中断占用百分比
  - st 虚拟机占用百分比
- KiB Mem
  - total 总内存
  - free 空闲内存
  - used 已使用内存
  - buff/cache 缓冲区内存
- KiB Swap
  - total 交换分区总量
  - free 空闲交换分区总量
  - used 已使用的交换分区总量
  - avail Mem 已被置换到内存的交换分区总量
- 信息列
  - **PID** 进程id
  - PPID 父进程id
  - RUSER Real user name
  - UID 进程所有者的用户id
  - **USER** 进程所有者的用户名
  - GROUP 进程所有者的组名
  - TTY 启动进程的终端名。不是从终端启动的进程则显示为 ?
  - **PR** 优先级
  - **NI** nice值。负值表示高优先级，正值表示低优先级
  - P 最后使用的CPU，仅在多CPU环境下有意义
  - %CPU 上次更新到现在的CPU时间占用百分比
  - TIME 进程使用的CPU时间总计，单位秒
  - TIME+ 进程使用的CPU时间总计，单位1/100秒
  - %MEM 进程使用的物理内存百分比
  - **VIRT** 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
  - SWAP 进程使用的虚拟内存中，被换出的大小，单位kb。
  - **RES** 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
  - CODE 可执行代码占用的物理内存大小，单位kb
  - DATA 可执行代码以外的部分(数据段+栈)占用的物理内存大小，单位kb
  - **SHR** 共享内存大小，单位kb
  - nFLT 页面错误次数
  - nDRT 最后一次写入到现在，被修改过的页面数
  - **S** 进程状态
    - D=不可中断的睡眠状态
    - R=运行
    - S=睡眠
    - T=跟踪/停止
    - Z=僵尸进程
  - **COMMAND** 命令名/命令行
  - WCHAN 若该进程在睡眠，则显示睡眠中的系统函数名
  - Flags 任务标志，参考 sched.h