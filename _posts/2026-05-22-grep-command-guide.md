---
layout: post
title: grep 命令详解
categories: [Linux]
description: Linux grep 命令全面入门，从基本搜索到正则表达式，覆盖常用选项与实战组合
keywords: Linux, grep, 正则表达式, 文本搜索, shell
---

## grep 命令详解

### 一、grep 是什么

grep 是 **G**lobal search **R**egular **E**xpression and **P**rint 的缩写，Linux 下最强大的文本搜索工具之一。它能在文件内容或命令输出中，按指定的模式（普通字符串或正则表达式）查找匹配的行并打印出来，堪称文本处理的"瑞士军刀"。

### 二、基本用法

```
grep [选项] 模式 [文件...]
```

最简单的形式——在文件中搜索字符串：

```bash
grep "hello" file.txt
```

通过管道接收输入：

```bash
cat file.txt | grep "hello"
```

更常见的管道用法：

```bash
# 在命令输出中过滤
ps aux | grep java
dmesg | grep error
history | grep ssh
```

### 三、常用选项详解

#### 3.1 忽略大小写 -i

```bash
grep -i "warning" app.log
# 匹配 Warning、WARNING、warning 等
```

#### 3.2 反向匹配 -v

打印不包含模式的行：

```bash
grep -v "#" config.conf
# 排除注释行，只看有效配置
```

经典组合——排除 grep 进程自身：

```bash
ps aux | grep java | grep -v grep
```

#### 3.3 显示行号 -n

```bash
grep -n "error" app.log
# 输出：45:error connecting to database
```

#### 3.4 计数 -c

只统计匹配行数，不输出内容：

```bash
grep -c "ERROR" app.log
# 输出：23
```

#### 3.5 递归搜索 -r / -R

```bash
grep -r "TODO" src/
# 递归搜索 src 目录下的所有文件

grep -rn "TODO" src/
# 加行号，最常用的代码搜索组合

grep -r "TODO" --include="*.java" src/
# 只搜索 .java 文件

grep -r "TODO" --exclude="*test*" src/
# 排除测试文件
```

#### 3.6 只显示文件名 -l / -L

```bash
grep -l "main" *.java
# 输出包含 main 的 Java 文件列表

grep -L "main" *.java
# 输出不包含 main 的文件
```

#### 3.7 精确匹配单词 -w

只匹配完整的单词，而不是子串：

```bash
grep -w "cat" file.txt
# 匹配 "cat" 但不匹配 "catalog"、"category"
```

#### 3.8 扩展正则 -E

使用扩展正则表达式，无需转义 `+`、`?`、`|`、`()` 等元字符：

```bash
grep -E "error|warning|fatal" app.log
# 匹配多个关键词

grep -E "^[0-9]{3}" file.txt
# 匹配以三位数字开头的行
```

`grep -E` 等价于 `egrep`。

#### 3.9 只输出匹配部分 -o

只打印匹配到的文本本身，而不是整行：

```bash
grep -o "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" access.log
# 只提取 IP 地址
```

### 四、正则表达式入门

grep 支持两种正则模式：**基础正则**（BRE）和 **扩展正则**（ERE，`-E` 启用）。

#### 常用元字符

| 元字符 | 含义 | 示例 |
|--------|------|------|
| `^` | 行首 | `^root` 匹配以 root 开头的行 |
| `$` | 行尾 | `error$` 匹配以 error 结尾的行 |
| `.` | 任意单个字符 | `c.t` 匹配 cat、cut、cot |
| `*` | 前一个字符重复 0 次或多次 | `ab*c` 匹配 ac、abc、abbc |
| `\+` | 前一个字符重复 1 次或多次（需要 `-E`） | `ab\+c` 匹配 abc、abbc |
| `\{n,m\}` | 重复 n 到 m 次 | `[0-9]\{3\}` 匹配三位数字 |
| `[...]` | 字符集 | `[aeiou]` 匹配任意元音 |
| `[^...]` | 排除字符集 | `[^0-9]` 匹配非数字 |
| `\|` | 或（需要 `-E`） | `error\|fatal` |

#### 几个例子

```bash
# 空行
grep "^$" file.txt

# 非空行
grep -v "^$" file.txt

# 以 # 开头的注释行
grep "^#" config.conf

# 5 到 6 位数字
grep -E "[0-9]{5,6}" file.txt

# 邮箱地址
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt
```

### 五、实用组合

#### 日志排查

```bash
# 查找错误
grep -in "exception" app.log

# 查看错误前后 5 行上下文
grep -in -C 5 "NullPointerException" app.log

# 统计各错误级别数量
grep -c "ERROR" app.log
grep -c "WARN" app.log
grep -c "INFO" app.log
```

#### 进程过滤

```bash
# 查找 Java 进程（排除自身）
ps aux | grep java | grep -v grep

# 或更简洁的方式
pgrep java
```

#### 代码搜索

```bash
# 项目中所有 TODO
grep -rn "TODO" --include="*.java" --include="*.xml" .

# 不搜索 target 和 node_modules
grep -rn "TODO" . --exclude-dir=target --exclude-dir=node_modules
```

### 六、总结

grep 是 Linux 下使用频率最高的命令之一，熟练掌握 grep 能极大地提升日常工作效率。核心要记住的套路：

- `-i` 忽略大小写
- `-v` 反向匹配
- `-n` 显示行号
- `-r` 递归搜索
- `-E` 扩展正则
- `-C n` 上下文

配合正则表达式，grep 能解决绝大多数文本搜索场景。下一篇《grep 实用技巧》将进一步介绍更高级的用法和实战场景。
