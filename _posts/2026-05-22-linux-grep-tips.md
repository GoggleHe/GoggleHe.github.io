---
layout: post
title: Linux grep 实用技巧
categories: [Linux]
description: Linux grep 进阶技巧，覆盖多文件搜索、管道配合、精准匹配、性能优化等实战场景
keywords: Linux, grep, 正则表达式, 文本搜索, shell, 实用技巧
---

## Linux grep 实用技巧

上一篇《grep 命令详解》介绍了基本用法和常用选项，这一篇深入实际场景，看看 grep 有哪些人见人爱的骚操作。

### 一、多文件搜索与上下文

#### 显示上下文 -A、-B、-C

排查日志时，只看匹配行往往不够，需要看上下文：

```bash
# 匹配行后 5 行
grep -A 5 "ERROR" app.log

# 匹配行前 3 行
grep -B 3 "ERROR" app.log

# 前后各 3 行（最常用）
grep -C 3 "ERROR" app.log
```

#### 显示文件名 -H

搜索多个文件时，默认会显示文件名。但有时候搜索结果来自管道，可以用 `-H` 强制显示：

```bash
grep -H "main" *.java
```

`-h` 则相反，强制不显示文件名。

### 二、配合其他命令

#### grep + xargs 批量处理

找到包含特定内容的文件后，对它们做操作：

```bash
# 找到包含 TODO 的 Java 文件，统计每个文件的行数
grep -rl "TODO" --include="*.java" src/ | xargs wc -l

# 找到包含 FIXME 的文件，备份成 .bak
grep -rl "FIXME" src/ | xargs -I {} cp {} {}.bak
```

#### grep -l + while read 逐行处理

比 xargs 更灵活的方式：

```bash
grep -rl "TODO" src/ | while read file; do
    echo "处理: $file"
    sed -i 's/TODO/DONE/g' "$file"
done
```

#### grep vs find：各司其职

| 命令 | 找什么 | 典型用法 |
|------|--------|----------|
| `find` | **文件**（按名、时间、大小） | `find . -name "*.java" -mtime -7` |
| `grep` | **内容**（按文本、正则） | `grep -rn "TODO" --include="*.java" .` |

两者也经常组合：

```bash
# 找到最近 7 天修改的 Java 文件，搜索其中包含 TODO 的行
find . -name "*.java" -mtime -7 -exec grep -l "TODO" {} \;
```

### 三、更精准的匹配

#### -F 固定字符串

当搜索的字符串包含正则元字符时（比如 IP 地址、文件路径），用 `-F` 把它们当纯文本处理，无需转义：

```bash
# 不用 -F 的话，. 需要写成 \.
grep -F "192.168.1.1" access.log

# 搜索路径中的反斜杠
grep -F "C:\Users\admin" file.txt
```

#### -f 从文件批量读取模式

把多个模式写到一个文件里，用 `-f` 一次性搜索：

```bash
# patterns.txt 内容：
# ERROR
# WARN
# NullPointerException

grep -f patterns.txt app.log
```

等价于 `grep -E "ERROR|WARN|NullPointerException"`，但文件管理起来更方便。

#### -P Perl 正则

如果系统 grep 支持 `-P`，可以使用 Perl 兼容的正则，功能更强大：

```bash
# 匹配 IP 地址：\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}
grep -P "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" access.log

# 零宽断言：匹配 "error" 前面的单词
grep -P "\w+(?=\s+error)" app.log
```

> 注意：macOS 的 BSD grep 不支持 `-P`，需要安装 GNU grep。

### 四、性能与技巧

#### 限制文件类型 --include / --exclude

在大项目中搜索时，只搜需要的文件类型可以大幅提速：

```bash
# 只搜 Java 和 XML 文件
grep -rn "TODO" --include="*.java" --include="*.xml" .

# 排除测试文件
grep -rn "TODO" --exclude="*Test*" .
```

#### 跳过目录 --exclude-dir

排除 node_modules、target 等目录是最常见的优化：

```bash
grep -rn "TODO" . --exclude-dir=node_modules --exclude-dir=target --exclude-dir=.git
```

可以用别名简化：

```bash
alias gr="grep -rn --exclude-dir=node_modules --exclude-dir=target --exclude-dir=.git"
```

之后只需 `gr "TODO" .`。

#### 颜色高亮 --color

```bash
# 始终高亮匹配内容
grep --color=always "pattern" file.txt

# 大多数 Linux 发行版默认已 alias grep='grep --color=auto'
```

### 五、实战场景

#### 日志时间范围过滤

```bash
# 某一天的所有日志
grep "2026-05-20" app.log

# 时间范围：20 号到 22 号
grep -E "2026-05-2[0-2]" app.log

# 精确到小时：20 号下午 2 点到 3 点
grep -E "2026-05-20 (14|15):" app.log
```

#### 代码搜索

```bash
# 查找最近的 API 调用变动
grep -rn "RestTemplate\|WebClient\|HttpClient" --include="*.java" src/

# 查找未使用的 import（简化版）
grep -rn "import" --include="*.java" src/ | grep -v "java\.\|javax\.\|lombok\|org\.springframework"

# 查找方法定义
grep -rn "public.*void.*\|public.*String.*\|private.*void" --include="*.java" src/ | grep -E "\(.*\)"
```

#### 统计 IP 访问

从 access.log 中统计每个 IP 的访问次数：

```bash
# 提取 IP → 排序 → 去重计数 → 按次数排序
grep -oP "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" access.log \
    | sort \
    | uniq -c \
    | sort -rn \
    | head -10
```

输出示例：
```
 3421 192.168.1.100
 2156 10.0.0.1
 1890 172.16.0.1
```

#### 两文件对比——差集

找出 file1 中有但 file2 中没有的行：

```bash
grep -vFf file2.txt file1.txt
```

解释：`-f file2.txt` 把 file2 的内容作为模式，`-v` 取反，得到 file1 中不在 file2 的行。

### 六、总结

grep 的强大之处在于组合——与管道、xargs、find、正则结合起来，能解决绝大多数文本处理问题。记住这几个实用套路：

| 场景 | 命令 |
|------|------|
| 看上下文 | `-C 5` |
| 只找文件名 | `-l` |
| 批量处理结果 | `xargs` / `while read` |
| 排除目录 | `--exclude-dir` |
| 纯文本匹配 | `-F` |
| 批量模式 | `-f patterns.txt` |
| 统计数据 | `sort \| uniq -c \| sort -rn` |
