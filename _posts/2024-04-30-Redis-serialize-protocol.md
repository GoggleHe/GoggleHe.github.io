---
layout: post
title: Redis通信协议
categories: [Redis]
description: Redis通信协议
keywords: Redis
---

# Redis通信协议

## 介绍

RESP(Redis serialization protocol) 

## RESP 2

RESP 协议使用 `\r\n` 作为分隔符

### 支持的类型

**1. 整数**

以 `:` 作为开头，格式为: `:[<+|->]<value>\r\n`

- `:1234\r\n` 用来表示整数 1234
- `:-567\r\n` 用来表示整数 -567

**2. 简单字符串**

以 `+` 作为开头，格式为: `+Simple String\r\n`

比如返回简单字符串 "OK"，使用 RESP2 表示即为:

```text
+OK\r\n
```

**3. 复杂字符串(Bulk strings)**

以 `$` 作为开头，格式为: `$<length>\r\n<data>\r\n`

比如字符串 "hello" 在 RESP2 的表示即为:

```text
$5\r\n
hello\r\n
```

其中，`$5\r\n` 表示字符串 "hello" 长度为 5

**4. 简单错误(Simple Error)**

以 `-` 作为开头，格式为: `-Error message\r\n`

例如 `-ERR unknown command 'asdf'\r\n` 表示返回命令不存在在的错误。格式类似简单字符串，无法表示带有二进制的错误信息。

**5. 数组**

以 `*` 作为开头，格式为: `*<number-of-elements>\r\n<element-1>...<element-n>`

例如，数组 `[1, "foo"]` 在 RESP2 的表示为:

```text
*2\r\n
+1\r\n
$3\r\nfoo\r\n
```

`*2\r\n` 表示该数组元素个数为 2，紧接着是数组元素的内容，分别是整数 1 和字符串 "foo"

**6. 空(null)**

在 RESP2 里面 null 有两种表示: `$-1\r\n` 表示空字符串的 null，`*-1\r\n` 表示数组的 null

### **RESP2 存在的问题**

首先，缺少准确的数据类型，如 SET 和 Map 类型导致只能通过数组来表示，所以客户端解析时需要知道当前是什么命令才能将返回解析成正确的数据类型。比如：

- 如果是 HGETALL 命令，则将响应结果解析为 Map 类型
- 如果是 SMEMBERS 命令，则将响应结果解析为 Set 类型
- 如果是 LRANGE 命令，则将响应结果解析为 Array 类型

如果能够返回精确的类型，那么客户端在解析的时候则可不需要关心当前命令，根据返回数据类型即可对应进行转换。类似的，目前浮点类型使用字符串表示，而布尔类型整数 0|1 来表示。

其次，null 有多种形式 `$-1\r\n` 和 `*-1\r\n`，同样客户端也需要按照命令来解释。

最后，错误信息只能是简单文本，无法携带二进制内容。

## **RESP3**

### 新类型

**1. 布尔类型**

以 `#` 作为开头, 格式为: `#<t|f>\r\n` ，其中 `t` 表示 true, `f` 表示 false

例如 `#t\r\n` 则表示返回值为 true

**2. 空(null)**

以 `_` 开头，格式固定为: `_\r\n`

**3. 浮点数**

以 `,` 作为开头，格式为: `,[<+|->]<integral>[.<fractional>][<E|e>[sign]<exponent>]\r\n`

例如 `,+1.23\r\n` 表示浮点数 1.23，`,-4.5\r\n` 表示浮点数 -4.5

而在 RESP2 则只能使用字符串 `$4\r\n1.23\r\n` 来表示浮点数 1.23

**4. 大数类型(Big Numbers)** 以 `(` 作为开头，格式为: `([+|-]<number>\r\n`

例如大数 `3492890328409238509324850943850943825024385` 在 RESP3 里面表示即为:

```text
(3492890328409238509324850943850943825024385\r\n
```

注意，如果客户端不支持 Big Number 类型则应该转换为字符串类型。

**5. 复杂错误类型(Bulk errors)**

以 `!` 作为开头，格式为: `!<length>\r\n<error>\r\n`

形式上和复杂字符串是一致，只是开头字符不一样。例如:

```text
!21\r\n
SYNTAX invalid syntax\r\n
```

**6. 字典(Maps)**

以 `%` 作为开头，格式为: `%<number-of-entries>\r\n<key-1><value-1>...<key-n><value-n>`

例如 `{ "first": 1, "second": 2 }` 用 RESP3 表示即为:

```text
%2\r\n
+first\r\n
:1\r\n           
+second\r\n                 
:2\r\n
```

**7. 集合(Sets)**

以 `~` 作为开头，格式为: `~<number-of-elements>\r\n<element-1>...<element-n>`

例如集合 `[3, 10, 12]` 使用 RESP3 表示即为:

```bash
~2\r\n
+3\r\n
+10\r\n
+12\r\n
```

**8. 推送(Push)**

以 `>` 作为开头，格式为: `><number-of-elements>\r\n<element-1>...<element-n>`

这个命令的主要作用是 Redis Server 主动推送数据给客户端，Client-side Caching 实现需要依赖 Push 协议来将更新数据推送给你到客户端。

**9. Verbatim strings**

以 `=` 作为开头，格式为: `=<length>\r\n<encoding>:<data>\r\n` ，其中 encoding 使用固定 3 个字符表示数据的编码方式，紧接着 `:` 用来分割编码和内容。比如是纯文本:

```bash
=15\r\n
txt:Some string\r\n
```

## Redis 请求/响应

对于 Redis 服务来说，请求(request)统一都使用字符串数组(bulk strings array)，而像整数 / 错误 / 简单字符串 / null 这些只会在返回响应(response)里面使用。

以 `SET hello hulk` 命令为例，变成 Redis 请求这是:

```text
第一个元素是字符串，长度 3       第三个元素是字符串，长度 4
      |                            |
      v                            v
*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$4\r\nhulk\r\n
 ^                  ^
 |                  ｜

数组长度 3          第二个元素是字符串，长度 5
```

从协议解析来看：

1. 第一个字符遇到 `*` 则可知当前是一个数组，接着解析数组长度直到分隔符(`\r\n`)，这里数组长度是 `3`，接着读取数组元素；
2. 开始解析数组第一个元素，看到 `$` 则开始进入解析字符串长度直到分隔符， 读取到一个元素长度为 `3`， 接着读取到内容 `SET\r\n` ，长度符合预期则继续解析下一个元素，否则抛错，以同样的方式继续解析数组后面的两个元素
3. 接着 Redis 开始处理请求，请求如果正确被处理则返回: `+OK\r\n`， 出错则是: `-ERR {message}\r\n`。

上面也提到 Redis 的请求(Request) 统一使用字符串数组的格式，第一个参数是命令，其他为命令参数。所以，请求的数组元素一定是字符串，只有 Client 解析响应内容的时候才会有多种类型。

## RESP2 和 RESP3 兼容

向下兼容，新特性需要RESP3