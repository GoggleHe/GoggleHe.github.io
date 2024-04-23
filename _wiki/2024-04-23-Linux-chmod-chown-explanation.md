---
layout: wiki
title: chmod+chown命令详解
categories: [Linux]
description: chmod+chown命令详解
keywords: Linux, chmod, chown
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

## chmod命令详解

```bash
用法：chmod [选项]... 模式[,模式]... 文件...
　或：chmod [选项]... 八进制模式 文件...
　或：chmod [选项]... --reference=参考文件 文件...
Change the mode of each FILE to MODE.
With --reference, change the mode of each FILE to that of RFILE.

  -c, --changes          like verbose but report only when a change is made
  -f, --silent, --quiet  suppress most error messages
  -v, --verbose          output a diagnostic for every file processed
      --no-preserve-root  do not treat '/' specially (the default)
      --preserve-root    fail to operate recursively on '/'
      --reference=RFILE  use RFILE's mode instead of MODE values
  -R, --recursive        change files and directories recursively
      --help            显示此帮助信息并退出
      --version         显示版本信息并退出

Each MODE is of the form '[ugoa]*([-+=]([rwxXst]*|[ugo]))+|[-+=][0-7]+'.

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
请向<http://translationproject.org/team/zh_CN.html> 报告chmod 的翻译错误
要获取完整文档，请运行：info coreutils 'chmod invocation'

```

## chown命令详解

```bash
用法：chown [选项]... [所有者][:[组]] 文件...
　或：chown [选项]... --reference=参考文件 文件...
Change the owner and/or group of each FILE to OWNER and/or GROUP.
With --reference, change the owner and group of each FILE to those of RFILE.

  -c, --changes          like verbose but report only when a change is made
  -f, --silent, --quiet  suppress most error messages
  -v, --verbose          output a diagnostic for every file processed
      --dereference      affect the referent of each symbolic link (this is
                         the default), rather than the symbolic link itself
  -h, --no-dereference   affect symbolic links instead of any referenced file
                         (useful only on systems that can change the
                         ownership of a symlink)
      --from=当前所有者:当前所属组
                                只当每个文件的所有者和组符合选项所指定时才更改所
                                有者和组。其中一个可以省略，这时已省略的属性就不
                                需要符合原有的属性。
      --no-preserve-root  do not treat '/' specially (the default)
      --preserve-root    fail to operate recursively on '/'
      --reference=RFILE  use RFILE's owner and group rather than
                         specifying OWNER:GROUP values
  -R, --recursive        operate on files and directories recursively

The following options modify how a hierarchy is traversed when the -R
option is also specified.  If more than one is specified, only the final
one takes effect.

  -H                     if a command line argument is a symbolic link
                         to a directory, traverse it
  -L                     traverse every symbolic link to a directory
                         encountered
  -P                     do not traverse any symbolic links (default)

      --help            显示此帮助信息并退出
      --version         显示版本信息并退出

Owner is unchanged if missing.  Group is unchanged if missing, but changed
to login group if implied by a ':' following a symbolic OWNER.
OWNER and GROUP may be numeric as well as symbolic.

示例：
  chown root /u         将 /u 的属主更改为"root"。
  chown root:staff /u   和上面类似，但同时也将其属组更改为"staff"。
  chown -hR root /u     将 /u 及其子目录下所有文件的属主更改为"root"。

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
请向<http://translationproject.org/team/zh_CN.html> 报告chown 的翻译错误
要获取完整文档，请运行：info coreutils 'chown invocation'

```

