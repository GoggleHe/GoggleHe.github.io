---
layout: wiki
title: top命令详解
categories: [Linux]
description: df命令帮助文档翻译
keywords: Linux, df
---


### 原文

```
Usage: df [OPTION]... [FILE]...
Show information about the file system on which each FILE resides,
or all file systems by default.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all             include dummy file systems
  -B, --block-size=SIZE  use SIZE-byte blocks
      --direct          show statistics for a file instead of mount point
      --total           produce a grand total
  -h, --human-readable  print sizes in human readable format (e.g., 1K 234M 2G)
  -H, --si              likewise, but use powers of 1000 not 1024
  -i, --inodes          list inode information instead of block usage
  -k                    like --block-size=1K
  -l, --local           limit listing to local file systems
      --no-sync         do not invoke sync before getting usage info (default)
  -P, --portability     use the POSIX output format
      --sync            invoke sync before getting usage info
  -t, --type=TYPE       limit listing to file systems of type TYPE
  -T, --print-type      print file system type
  -x, --exclude-type=TYPE   limit listing to file systems not of type TYPE
  -v                    (ignored)
      --help     display this help and exit
      --version  output version information and exit

Display values are in units of the first available SIZE from --block-size,
and the DF_BLOCK_SIZE, BLOCK_SIZE and BLOCKSIZE environment variables.
Otherwise, units default to 1024 bytes (or 512 if POSIXLY_CORRECT is set).

SIZE may be (or may be an integer optionally followed by) one of following:
KB 1000, K 1024, MB 1000*1000, M 1024*1024, and so on for G, T, P, E, Z, Y.

Report df bugs to bug-coreutils@gnu.org
GNU coreutils home page: <http://www.gnu.org/software/coreutils/>
General help using GNU software: <http://www.gnu.org/gethelp/>
For complete documentation, run: info coreutils 'df invocation'
```



### 翻译

```
Usage: df [OPTION]... [FILE]...
Show information about the file system on which each FILE resides,
or all file systems by default.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all             展示全部文件系统
  -B, --block-size=SIZE  设置区块大小
      --direct          显示文件的统计信息而不是挂载点的统计信息
      --total           生成总计栏
  -h, --human-readable  以可读的方式输出
  -H, --si              同-h,不过单位是1000而不是1024
  -i, --inodes          展示inode信息而不是区块使用信息
  -k                    设置区块大小为1k
  -l, --local           限制查询本地系统
      --no-sync         执行前不执行sync
  -P, --portability     使用POSIX输出格式
      --sync            执行前执行sync
  -t, --type=TYPE       限制指定类型的文件系统
  -T, --print-type      打印文件类型
  -x, --exclude-type=TYPE   排除指定文件类型的文件系统
  -v                    (ignored)
      --help     显示帮助文档
      --version  输出版本信息并退出


显示的值以第一个--block-size,DF_BLOCK_SIZE, BLOCK_SIZE 和 BLOCKSIZE 环境变量中的第一个可用的大小为单位。否则，单位默认为1024字节（如果设置POSIXLY_CORRECT则为512）

SIZE may be (or may be an integer optionally followed by) one of following:
KB 1000, K 1024, MB 1000*1000, M 1024*1024, and so on for G, T, P, E, Z, Y.

大小可以是（或者后面也可以是一个整数）以下其中一个：
对于G、T、P、E、Z、Y，KB 1000、K 1024、MB 1000*1000、M 1024*1024，依此类推。
```

### 附注

- inode：文件索引节点，存储文件基本属性信息，一个文件(文件夹)一个inode
- block：存放文件数据的位置

