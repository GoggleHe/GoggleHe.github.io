---
layout: post
title: Linux文件系统
categories: [Linux]
description: Linux文件系统
keywords: keyword1, keyword2
---

## Linux文件系统

### 文件名与磁盘

/dev/sd[a-p]：表示SATA接口的硬盘文件，[a-p]根据硬盘被侦测的顺序命名

/dev/vd[a-p]：表示虚拟接口的硬盘文件，[a-p]根据硬盘被侦测的顺序命名

`df -h`可列出磁盘文件名及其挂载点

```bash
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   17G  1.9G   16G   11% /
devtmpfs                 475M     0  475M    0% /dev
tmpfs                    487M     0  487M    0% /dev/shm
tmpfs                    487M   26M  461M    6% /run
tmpfs                    487M     0  487M    0% /sys/fs/cgroup
/dev/sda1               1014M  133M  882M   14% /boot
tmpfs                     98M     0   98M    0% /run/user/0
```

/dev/mapper/centos-root ：一个logic volume group，由多个volume组成的volume group。

volume是一个逻辑上独立的存储空间，可以是一个硬盘分区，一个软盘，一个USB等任何存储设备。volume可以包含一个或多个存储设备，是存储数据的容器，文件系统则是一种用来组织和管理文件的方法。

### 磁盘分区

#### 检查可用磁盘

`fdisk -l`：检查空磁盘

#### 分区

`fdisk <文件系统名>`

根据指令手册操作

#### 内核读取新的分区表

` partprobe /dev/sdb` ：

#### 创建文件系统(格式化)

`mkfs.xfs /dev/sdb -f` ：格式化，后缀为Linux文件系统格式

#### 挂载

修改配置文件

`vim /etc/fstab`：新增如下行，第一个 0 表示不备份 第二个 0 表示不检查

```
/dev/sdb                /my_partition           xfs     defaults        0 0
```

`mount -a` 挂载

#### 检查

`df -h` ：观察新挂载的硬盘分区是否有显示