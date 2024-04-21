---
layout: wiki
title: find命令详解
categories: [Linux]
description: find命令详解
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

## find命令详解

### 帮助文档

```bash
用法: find [-H] [-L] [-P] [-Olevel] [-D help|tree|search|stat|rates|opt|exec] [path...] [expression]

默认路径为当前目录；默认表达式为 -print
表达式可能由下列成份组成：操作符、选项、测试表达式以及动作：

操作符 (优先级递减；未做任何指定时默认使用 -and):
      ( EXPR )   ! EXPR   -not EXPR   EXPR1 -a EXPR2   EXPR1 -and EXPR2
      EXPR1 -o EXPR2   EXPR1 -or EXPR2   EXPR1 , EXPR2

positional options (always true): -daystart -follow -regextype

normal options (always true, specified before other expressions):
      -depth --help -maxdepth LEVELS -mindepth LEVELS -mount -noleaf
      --version -xautofs -xdev -ignore_readdir_race -noignore_readdir_race

比较测试 (N 可以是 +N 或 -N 或 N): -amin N -anewer FILE -atime N -cmin N
      -cnewer 文件 -ctime N -empty -false -fstype 类型 -gid N -group 名称
      -ilname 匹配模式 -iname 匹配模式 -inum N -ipath 匹配模式 -iregex 匹配模式
      -links N -lname 匹配模式 -mmin N -mtime N -name 匹配模式 -newer 文件
      -nouser -nogroup -path PATTERN -perm [-/]MODE -regex PATTERN
      -readable -writable -executable
      -wholename PATTERN -size N[bcwkMG] -true -type [bcdpflsD] -uid N
      -used N -user NAME -xtype [bcdpfls]
      -context 文本


操作: -delete -print0 -printf 格式 -fprintf 文件 格式 -print
      -fprint0 文件 -fprint 文件 -ls -fls 文件 -prune -quit
      -exec 命令 ; -exec 命令 {} + -ok 命令 ;
      -execdir 命令 ; -execdir 命令 {} + -okdir 命令 ;

```



