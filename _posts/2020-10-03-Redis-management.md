---
layout: post
title: Redis管理
categories: [Redis]
description: Redis管理
keywords: Redis
---

# Redis管理

## Redis配置

- 配置redis.conf文件：keyword argument1 argument2 ... argumentN

  - ``slaveof 127.0.0.1 6380``
  - ``requirepass "hello world"``

- 通过命令行传参

  - ``./redis-server --port 6380 --slaveof 127.0.0.1 6379``

- 运行时配置更改：[CONFIG SET](http://www.redis.cn/commands/config-set.html)、[CONFIG REWRITE](http://www.redis.cn/commands/config-rewrite.html)

  - 并非所有的配置指令都支持这种使用方式，但是大部分是支持的。
  - 需要确保的是在通过[CONFIG SET](http://www.redis.cn/commands/config-set.html)命令进行的设置的同时，也需在 redis.conf文件中进行了相应的更改。（防止重启配置丢失）

- 配置Redis成为一个缓存

  - ```
    maxmemory 2mb
    maxmemory-policy allkeys-lru
    ```

## 复制

### 主从

#### 同步机制

- 当一个 master 实例和一个 slave 实例连接正常时， master 会发送一连串的命令流来保持对 slave 的更新，以便于将自身数据集的改变复制给 slave ， ：包括客户端的写入、key 的过期或被逐出等等。
- 当 master 和 slave 之间的连接断开之后，因为网络问题、或者是主从意识到连接超时， slave 重新连接上 master 并会尝试进行部分重同步：这意味着它会尝试只获取在断开连接期间内丢失的命令流。
- 当无法进行部分重同步时， slave 会请求进行全量重同步。这会涉及到一个更复杂的过程，例如 master 需要创建所有数据的快照，将之发送给 slave ，之后在数据集更改时持续发送命令流到 slave 。

#### 主从注意事项

- Redis 使用异步复制，slave 和 master 之间异步地确认处理的数据量
- 一个 master 可以拥有多个 slave
- slave 可以接受其他 slave 的连接。除了多个 slave 可以连接到同一个 master 之外， slave 之间也可以像层叠状的结构（cascading-like structure）连接到其他 slave 。自 Redis 4.0 起，所有的 sub-slave 将会从 master 收到完全一样的复制流。
- Redis 复制在 master 侧是非阻塞的。这意味着 master 在一个或多个 slave 进行初次同步或者是部分重同步时，可以继续处理查询请求。
- 复制在 slave 侧大部分也是非阻塞的。当 slave 进行初次同步时，它可以使用旧数据集处理查询请求，假设你在 redis.conf 中配置了让 Redis 这样做的话。否则，你可以配置如果复制流断开， Redis slave 会返回一个 error 给客户端。但是，在初次同步之后，旧数据集必须被删除，同时加载新的数据集。 slave 在这个短暂的时间窗口内（如果数据集很大，会持续较长时间），会阻塞到来的连接请求。自 Redis 4.0 开始，可以配置 Redis 使删除旧数据集的操作在另一个不同的线程中进行，但是，加载新数据集的操作依然需要在主线程中进行并且会阻塞 slave 。
- 复制既可以被用在可伸缩性，以便只读查询可以有多个 slave 进行（例如 O(N) 复杂度的慢操作可以被下放到 slave ），或者仅用于数据安全。
- 可以使用复制来避免 master 将全部数据集写入磁盘造成的开销：一种典型的技术是配置你的 master Redis.conf 以避免对磁盘进行持久化，然后连接一个 slave ，其配置为不定期保存或是启用 AOF。但是，这个设置必须小心处理，因为重新启动的 master 程序将从一个空数据集开始：如果一个 slave 试图与它同步，那么这个 slave 也会被清空。

#### 当 master 关闭持久化时，复制的安全性

- 在使用 Redis 复制功能时的设置中，强烈建议在 master 和在 slave 中启用持久化。
- 当不可能启用时，例如由于非常慢的磁盘性能而导致的延迟问题，**应该配置实例来避免重置后自动重启**。
- 当 Redis Sentinel 被用于高可用并且 master 关闭持久化，这时如果允许自动重启进程也是很危险的。例如， master 可以重启的足够快以致于 Sentinel 没有探测到故障，因此上述的故障模式也会发生。

#### Redis 复制功能是如何工作的

- 每一个 Redis master 都有一个 replication ID ：这是一个较大的伪随机字符串，标记了一个给定的数据集。
- 每个 master 也持有一个偏移量，master 将自己产生的复制流发送给 slave 时，发送多少个字节的数据，自身的偏移量就会增加多少，目的是当有新的操作修改自己的数据集时，它可以以此更新 slave 的状态。
- 制偏移量即使在没有一个 slave 连接到 master 时，也会自增，所以基本上每一对给定的``Replication ID, offset``都会标识一个 master 数据集的确切版本。
- 当 slave 连接到 master 时，它们使用 PSYNC 命令来发送它们记录的旧的 master replication ID 和它们至今为止处理的偏移量。通过这种方式， master 能够仅发送 slave 所需的增量部分。
- 但是如果 master 的缓冲区中没有足够的命令积压缓冲记录，或者如果 slave 引用了不再知道的历史记录（replication ID），则会转而进行一个全量重同步：在这种情况下， slave 会得到一个完整的数据集副本，从头开始。以下是全量同步的细节
  - master 开启一个后台保存进程，以便于生产一个 RDB 文件。
  - 同时它开始缓冲所有从客户端接收到的新的写入命令。
  - 当后台保存完成时， master 将数据集文件传输给 slave， slave将之保存在磁盘上，然后加载文件到内存。
  - 再然后 master 会发送所有缓冲的命令发给 slave。这个过程以指令流的形式完成并且和 Redis 协议本身的格式相同。

#### 无需磁盘参与的复制

- 正常情况下，一个全量重同步要求在磁盘上创建一个 RDB 文件，然后将它从磁盘加载进内存，然后 slave以此进行数据同步。
- 如果磁盘性能很低的话，这对 master 是一个压力很大的操作。Redis 2.8.18 是第一个支持无磁盘复制的版本。在此设置中，子进程直接发送 RDB 文件给 slave，无需使用磁盘作为中间储存介质。

#### 只读性质的 slave

A —> B —-> C : 即使节点 B 是可写的，C 也不会看到 B 的写入，而是将拥有和 master 实例 A 相同的数据集。

#### 允许只写入 N 个附加的副本

##### 概念

- 配置成当至少有N个副本写入成功，master才写成功

##### 原理

- Redis slave 每秒钟都会 ping master，确认已处理的复制流的数量。
- Redis master 会记得上一次从每个 slave 都收到 ping 的时间。
- 用户可以配置一个最小的 slave 数量，使得它滞后 <= 最大秒数。
- 如果至少有 N 个 slave ，并且滞后小于 M 秒，则写入将被接受。

##### 配置

	- 如果条件不满足，master 将会回复一个 error 并且写入将不被接受。
	- min-slaves-to-write <slave 数量>
- min-slaves-max-lag <秒数>

#### Redis 复制如何处理 key 的过期

- slave 不会让 key 过期，而是等待 master 让 key 过期。当一个 master 让一个 key 到期（或由于 LRU 算法将之驱逐）时，它会合成一个 DEL 命令并传输到所有的 slave。
- 但是，由于这是 master 驱动的 key 过期行为，master 无法及时提供 DEL 命令，所以有时候 slave 的内存中仍然可能存在在逻辑上已经过期的 key 。为了处理这个问题，slave 使用它的逻辑时钟以报告只有在不违反数据集的一致性的读取操作（从主机的新命令到达）中才存在 key。用这种方法，slave 避免报告逻辑过期的 key 仍然存在。在实际应用中，使用 slave 程序进行缩放的 HTML 碎片缓存，将避免返回已经比期望的时间更早的数据项。
- 在Lua脚本执行期间，不执行任何 key 过期操作。当一个Lua脚本运行时，从概念上讲，master 中的时间是被冻结的，这样脚本运行的时候，一个给定的键要么存在要么不存在。这可以防止 key 在脚本中间过期，保证将相同的脚本发送到 slave ，从而在二者的数据集中产生相同的效果。
- 一旦一个 slave 被提升为一个 master ，它将开始独立地过期 key，而不需要任何旧 master 的帮助。

#### INFO 和 ROLE 命令

- 有两个 Redis 命令可以提供有关主从实例当前复制参数的很多信息。一个是INFO。如果使用复制参数像 INFO replication 调用该命令，则只显示与复制相关的信息。
- 另一个更加 computer-friendly 的命令是 ROLE，它提供 master 和 slave 的复制状态以及它们的复制偏移量，连接的 slaves 列表等等。

#### 重新启动和故障转移后的部分重同步

- 从 Redis 4.0 开始，当一个实例在故障转移后被提升为 master 时，它仍然能够与旧 master 的 slaves 进行部分重同步。
- slave 会记住旧 master 的旧 replication ID 和复制偏移量，因此即使询问旧的 replication ID，其也可以将部分复制缓冲提供给连接的 slave 。
- 但是，升级的 slave 的新 replication ID 将不同，因为它构成了数据集的不同历史记录。例如，master 可以返回可用，并且可以在一段时间内继续接受写入命令，因此在被提升的 slave 中使用相同的 replication ID 将违反一对复制标识和偏移对只能标识单一数据集的规则。
- 外，slave 在关机并重新启动后，能够在 RDB 文件中存储所需信息，以便与 master 进行重同步。这在升级的情况下很有用。当需要时，最好使用 SHUTDOWN 命令来执行 slave 的保存和退出操作。

## 持久化

### 不同级别的持久化方式

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储.
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.
- 如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 你也可以同时开启两种持久化方式, 在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.

### RDB的优点和缺点

#### 优点

- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.
- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.
- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能.
- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.

#### 缺点

- 如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.
- RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.

### AOF 优点和缺点

#### 优点

- 使用AOF 会让你的Redis更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.
- AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。

#### 缺点

- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。

### 如何选择使用哪种持久化方式？

- 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能
- 如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化

### 快照

- 在默认情况下， Redis 将数据库快照保存在名字为 dump.rdb的二进制文件中
- 可以对 Redis 进行设置， 让它在“ N 秒内数据集至少有 M 个改动”这一条件被满足时， 自动保存一次数据集
- 你也可以通过调用 SAVE或者 BGSAVE ， 手动让 Redis 进行数据集保存操作。
- 这种持久化方式被称为快照 snapshotting.

### 工作方式

当 Redis 需要保存 dump.rdb 文件时， 服务器执行以下操作:

- Redis 调用forks. 同时拥有父进程和子进程。
- 子进程将数据集写入到一个临时 RDB 文件中。
- 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

### 只追加操作的文件（Append-only file，AOF）

- 配置：``appendonly yes``

### 日志重写

- 问题：如果你对一个计数器调用了 100 次 INCR ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100 条记录（entry）。然而在实际上， 只使用一条 SET 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。
- 解决：可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。执行 BGREWRITEAOF 命令， Redis 将生成一个新的 AOF 文件， 这个文件包含重建当前数据集所需的最少命令。Redis 2.2 需要自己手动执行 BGREWRITEAOF 命令； Redis 2.4 则可以自动触发 AOF 重写， 具体信息请查看 2.4 的示例配置文件

### AOF数据同步到磁盘的方式

- 每次有新命令追加到 AOF 文件时就执行一次 fsync ：非常慢，也非常安全
- 每秒 fsync 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
- 从不 fsync ：将数据交给操作系统来处理。更快，也更不安全的选择。
- 推荐（并且也是默认）的措施为每秒 fsync 一次， 这种 fsync 策略可以兼顾速度和安全性。

### 如果AOF文件损坏了怎么办？

- 为现有的 AOF 文件创建一个备份。

- 使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复:

  $ redis-check-aof –fix

- （可选）使用 diff -u 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。

- 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。

### AOF重写的工作原理

AOF 重写和 RDB 创建快照一样，都巧妙地利用了写时复制机制:

- Redis 执行 fork() ，现在同时拥有父进程和子进程。
- 子进程开始将新 AOF 文件的内容写入到临时文件。
- 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾,这样样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。
- 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
- 搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

### 怎样从RDB方式切换为AOF方式

- 为最新的 dump.rdb 文件创建一个备份。
- 将备份放到一个安全的地方。
- 执行以下两条命令:
- redis-cli config set appendonly yes
- redis-cli config set save “”
- 确保写命令会被正确地追加到 AOF 文件的末尾。
- 执行的第一条命令开启了 AOF 功能： Redis 会阻塞直到初始 AOF 文件创建完成为止， 之后 Redis 会继续处理命令请求， 并开始将写入命令追加到 AOF 文件末尾。
- 执行的第二条命令用于关闭 RDB 功能。 这一步是可选的， 如果你愿意的话， 也可以同时使用 RDB 和 AOF 这两种持久化功能。

### AOF和RDB之间的相互作用

- 在版本号大于等于 2.4 的 Redis 中， BGSAVE 执行的过程中， 不可以执行 BGREWRITEAOF 。 反过来说， 在 BGREWRITEAOF 执行的过程中， 也不可以执行 BGSAVE。这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。
- 如果 BGSAVE 正在执行， 并且用户显示地调用 BGREWRITEAOF 命令， 那么服务器将向用户回复一个 OK 状态， 并告知用户， BGREWRITEAOF 已经被预定执行： 一旦 BGSAVE 执行完毕， BGREWRITEAOF 就会正式开始。 当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

### 备份redis数据

- RDB 文件一旦被创建， 就不会进行任何修改。
-  当服务器要创建一个新的 RDB 文件时， 它先将文件的内容保存在一个临时文件里面， 当临时文件写入完毕时， 程序才使用 rename(2) 原子地用临时文件替换原来的 RDB 文件。
- 这也就是说， 无论何时， 复制 RDB 文件都是绝对安全的。
- 创建一个定期任务（cron job）， 每小时将一个 RDB 文件备份到一个文件夹， 并且每天将一个 RDB 文件备份到另一个文件夹。
- 确保快照的备份都带有相应的日期和时间信息， 每次执行定期任务脚本时， 使用 find 命令来删除过期的快照： 比如说， 你可以保留最近 48 小时内的每小时快照， 还可以保留最近一两个月的每日快照。
- 至少每天一次， 将 RDB 备份到你的数据中心之外， 或者至少是备份到你运行 Redis 服务器的物理机器之外。

### 容灾备份

- Redis 的容灾备份基本上就是对数据进行备份， 并将这些备份传送到多个不同的外部数据中心

## Redis 的管理

### Redis安装提示

- 我们建议使用linux部署Redis。Redis也在osx，FreeBSD，OpenBSD上经过测试，但Linux经过所有主要的压力测试，并且最多产品部署。 确保设置Linux内核overcommit memory setting为1。向/etc/sysctl.conf添加vm.overcommit_memory = 1然后重启，或者运行命令sysctl vm.overcommit_memory=1以便立即生效。
- 确保禁用Linux内核特性transparent huge pages，它对内存使用和延迟有非常大的负面影响。通过命令echo never > sys/kernel/mm/transparent_hugepage/enabled来完成。
- 确保你的系统设置了一些swap（我们建议和内存一样大）。如果linux没有swap并且你的redis实例突然消耗了太多内存，或者Redis由于内存溢出会宕掉，或者Linux内核OOM Killer会杀掉Redis进程。
- 设置一个明确的maxmemory参数来限制你的实例，以便确保实例会报告错误而不是当接近系统内存限制时失败
- 如果你对一个写频繁的应用使用redis，当向磁盘保存RDB文件或者改写AOF日志时，**redis可能会用正常使用内存2倍的内存**。额外使用的内存和保存期间写修改的内存页数量成比例，因此经常和这期间改动的键的数量成比例。确保相应的设置内存的大小。
- 当在daemontools下运行时，使用daemonize no
- 即使你禁用持久化，如果你使用复制，redis会执行rdb保存，除非你使用新的无磁盘复制特性，这个特性目前还是实验性的。
- 如果你使用复制，确保要么你的master激活了持久化，要么它不会在当掉后自动重启。slave是master的完整备份，因此如果master通过一个空数据集重启，slave也会被清掉。

### 在EC2上运行redis

- 使用基于HVM的实例，不是基于PV的实例。
- 不要使用老的实例家族。例如，使用HVM的m3.medium代替PV的m1.medium。
- Redis持久化使用EC2 EBS卷需要小心处理，因为有时EBS卷有高延迟特性。
- 如果当slave和master同步时有问题，你可能想尝试新的**无磁盘复制**特性。

### 无需停机升级或重启 Redis 实例

- 为你的 Redis 实例配置新的从实例。为此你需要一台额外的服务器，或者你的服务器有足够的内存以便同时运行两个 Redis 实例。
- 如果你用单一的服务器，需确保从实例用与主实例不同的端口启动，否则从实例根本不会启动成功。
- 等待复制机制的初始同步完成（检查从实例的日志文件）。
- 用 INFO 命令确认在主从实例中有同样数量的键值。用 redis-cli 检查从实例工作正常并响应你的命令。 使用config set slave-read-only no，允许写slave。
- 配置你所有的客户端以便使用新的实例（即从实例）
- 当你确认主实例不再接收到任何请求（你可以用 [MONITOR](http://www.redis.cn/commands/monitor.html) 命令监视到），使用 **SLAVEOF NO ONE** 命令切换从实例为主实例，然后关闭原先的主实例。

## Redis安全性

## Redis信号处理

### SIGTERM信号处理

- SIGTERM信号会让Redis安全的关闭。Redis收到信号时并不立即退出，而是开启一个定时任务，这个任务就类似执行一次SHUTDOWN命令的。 这个定时关闭任务会在当前执行命令终止后立即施行，因此通常有有0.1秒或更少时间延迟。
- 发送SIGTERM信号关闭Redis：pkill redis-server、kill -15 redis-pid
- 如果存在正在执行RDB文件保存或者AOF重写的子进程，子进程被终止。
- 如果AOF功能是开启的，Redis会通过系统调用fsync将AOF缓冲区数据强制输出到硬盘。
- 如果Redis配置了使用RDB文件进行持久化，那么此时就会进行同步保存。由于保存时同步的，那也就不需要额外的内存。
- 如果Server是守护进程，PID文件会被移除。
- 如果Unix域的Socket是可用的，它也会被移除。
- Server退出，退出码为0.
- 万一RDB文件保存失败，Shutdown失败，Server则会继续运行以保证没有数据丢失。自从Redis2.6.11之后，Redis不会再次主动Shutdown，除非它接收到了另一个SIGTERM信号或者另外一个SHUTDOWN命令

### SIGSEGV,SIGBUS,SIGFPF和SIGILL信号的处理

- Redis接收到以下几种信号时会崩溃：
  - SIGSEGV
  - SIGBUS
  - SIGFPE
  - SIGILL
- 如果以上信号被捕获，Redis会终止所有正在进行的操作，并进行以下操作：
  - 包括调用栈信息，寄存器信息，以及clients信息会以bug报告的形式写入日志文件。
  - 自从Redis2.8（当前为开发版本）之后，Redis会在系统崩溃时进行一个快速的内存检测以保证系统的可靠性。
  - 如果Server是守护进程，PID文件会被移除。
  - 最后server会取消自己对当前所接收信号的信号处理器的注册，并重新把这个信号发给自己，这是为了保证一些默认的操作被执行，比如把Redis的核心Dump到文件系统。

### 子进程被终止时会发生什么

- 当一个正在进行AOF重写的子进程被信号终止，Redis会把它当成一个错误并丢弃这个AOF文件(可能是部分或者完全损坏)。AOF重写过程会在以后被重新触发。
- 当一个正在执行RDB文件保存的子进程被终止Redis会把它当做一个严重的错误，因为AOF重写只会导致AOF文件冗余，但是RDB文件保存失败会导致Redis不可用。
- 如果一个正在保存RDB文件的子进程被信号终止或者自身出现了错误(非0退出码)，Redis会进入一种特殊的错误状态，不允许任何写操作
  - Redis会继续回复所有的读请求。
  - Redis会回复给所有的写请求一个MISCONFIG错误。

### 不触发错误状态的情况下终止RDB文件的保存

- 自从Redis2.6.10之后就可以使用信号SIGUSR1，这个信号会被特殊处理：它会像其他信号一样终止子进程，但是父进程不会检测到这个严重的错误，照常接收所有的用户写请求。

## Redis如何处理客户端连接

### 客户端的连接的建立

当一个新的客户端连接被接受执行以下操作：

- 当Redis使用非阻塞I/O复用，客户端socket设置为非阻塞状态。
- socket TCP_NODELAY属性被设置确保在连接中我们不会延迟。
- 一个 可读的文件事件被创建，因而当新的数据可以被访问时，Reids可以更快接收客户端在socket上的查询

当客户端初始化后，Redis检查我们是否还在它可以同时处理的客户端的数量限制范围内(这个是使用 maxclients 配置指令配置的，请参阅本文档的下一节获取更多的信息。

如果它因为当前已经接受了最大数量的客户端，无法接受当前的客户端，Redis将尝试发送一个错误给客户端以便让其意识到这种情况，并且立即关闭连接。即使连接被Redis立即关闭，错误信息也会返回给客户端，因为新的socket输出缓冲区一般情况下都足够放下错误信息，因而客户端内核将处理连接错误。

### 客户端按照什么顺序被处理

该顺序是由客户端socket文件描述符的数字大小及核心报告客户端事件的顺序决定的，因此顺序可以看成不确定的

Redis给客户端提供服务时会做以下两件事

- 每次它从客户端socket读取新东西的时候它只执行一次 read() 系统调用，以确保当我们有多台客户端连接时，并且有一些要求高客户端以非常快的速率发送查询时，其它客户端不会因此而受到惩罚和经历一个糟糕的延时。（译者注：意思就是不读取完整个socket的消息，而是每个socket轮流读一次）
- 当系统调用执行完，当前缓冲中的命令不管有多少都会被顺序处理。

### 最大数量的客户端

- 在Redis 2.4中，同时处理的最大客户端数量的限制是硬编码的。
- 在Redis 2.6中这个限制是动态的：默认情况下为10000个客户端，除非在redis.conf中配置了maxclients配置项。

## 高可用性

### Sentinel的功能和作用

- **监控（Monitoring**）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

### 启动一个sentinel服务

- ``redis-sentinel``
- ``redis-server /path/to/sentinel.conf --sentinel``

### 配置 Sentinel

最小配置（sentinel.conf ）：配置格式：sentinel <选项的名字> <主服务器的名字> <选项的值>

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

- 第一行配置指示 Sentinel 去监视一个名为 mymaster 的主服务器， 这个主服务器的 IP 地址为 127.0.0.1 ， 端口号为 6379 ， 而将这个主服务器判断为失效至少需要 2 个 Sentinel 同意 （只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行）。

- down-after-milliseconds 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。

  - 如果服务器在给定的毫秒数之内， 没有返回 Sentinel 发送的 PING 命令的回复， 或者返回一个错误， 那么 Sentinel 将这个服务器标记为主观下线（subjectively down，简称 SDOWN ）。
  - 不过只有一个 Sentinel 将服务器标记为主观下线并不一定会引起服务器的自动故障迁移： 只有在足够数量的 Sentinel 都将一个服务器标记为主观下线之后， 服务器才会被标记为客观下线（objectively down， 简称 ODOWN ）， 这时自动故障迁移才会执行。

- parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。

  - 如果从服务器被设置为允许使用过期数据集（参见对 redis.conf 文件中对 slave-serve-stale-data 选项的说明）， 那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求， 因为尽管复制过程的绝大部分步骤都不会阻塞从服务器， 但从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求： 如果全部从服务器一起对新的主服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。

    你可以通过将这个值设为 1 来保证每次只有一个从服务器处于不能处理命令请求的状态。

### 主观下线和客观下线

- 主观下线（Subjectively Down， 简称 SDOWN）指的是单个 Sentinel 实例对服务器做出的下线判断。
- 客观下线（Objectively Down， 简称 ODOWN）指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。）
- 如果一个服务器没有在 master-down-after-milliseconds 选项所指定的时间内， 对向它发送 PING 命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线
  - 返回 +PONG 。
  - 返回 -LOADING 错误。
  - 返回 -MASTERDOWN 错误。
- 注意， 一个服务器必须在 master-down-after-milliseconds 毫秒内， 一直返回无效回复才会被 Sentinel 标记为主观下线。举个例子， 如果 master-down-after-milliseconds 选项的值为 30000 毫秒（30 秒）， 那么只要服务器能在每 29 秒之内返回至少一次有效回复， 这个服务器就仍然会被认为是处于正常状态的。
-  如果 Sentinel 在给定的时间范围内， 从其他 Sentinel 那里接收到了足够数量的主服务器下线报告， 那么 Sentinel 就会将主服务器的状态从主观下线改变为客观下线。 如果之后其他 Sentinel 不再报告主服务器已下线， 那么客观下线状态就会被移除。
- 客观下线条件**只适用于主服务器**： 对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。
- 只要一个 Sentinel 发现某个主服务器进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

### 每个 Sentinel 都需要定期执行的任务

- 每个 Sentinel 以每秒钟一次的频率向它所知的主服务器、从服务器以及其他 Sentinel 实例发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 那么这个实例会被 Sentinel 标记为主观下线。 一个有效回复可以是： +PONG 、 -LOADING 或者 -MASTERDOWN 。
- 如果一个主服务器被标记为主观下线， 那么正在监视这个主服务器的所有 Sentinel 要以每秒一次的频率确认主服务器的确进入了主观下线状态。
- 如果一个主服务器被标记为主观下线， 并且有足够数量的 Sentinel （至少要达到配置文件指定的数量）在指定的时间范围内同意这一判断， 那么这个主服务器被标记为客观下线。
- 在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有主服务器和从服务器发送 INFO 命令。 当一个主服务器被 Sentinel 标记为客观下线时， Sentinel 向下线主服务器的所有从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
- 当没有足够数量的 Sentinel 同意主服务器已经下线， 主服务器的客观下线状态就会被移除。 当主服务器重新向 Sentinel 的 PING 命令返回有效回复时， 主服务器的主观下线状态就会被移除。

### 自动发现 Sentinel 和从服务器

- 一个 Sentinel 可以与其他多个 Sentinel 进行连接， 各个 Sentinel 之间可以互相检查对方的可用性， 并进行信息交换。
- 你无须为运行的每个 Sentinel 分别设置其他 Sentinel 的地址， 因为 Sentinel 可以通过发布与订阅功能来自动发现正在监视相同主服务器的其他 Sentinel ， 这一功能是通过向频道 **sentinel**:hello 发送信息来实现的。
- 每个 Sentinel 会以每两秒一次的频率， 通过发布与订阅功能， 向被它监视的所有主服务器和从服务器的 **sentinel**:hello 频道发送一条信息， 信息中包含了 Sentinel 的 IP 地址、端口号和运行 ID （runid）。
- 每个 Sentinel 都订阅了被它监视的所有主服务器和从服务器的 **sentinel**:hello 频道， 查找之前未出现过的 sentinel （looking for unknown sentinels）。 当一个 Sentinel 发现一个新的 Sentinel 时， 它会将新的 Sentinel 添加到一个列表中， 这个列表保存了 Sentinel 已知的， 监视同一个主服务器的所有其他 Sentinel 。
- Sentinel 发送的信息中还包括完整的主服务器当前配置（configuration）。 如果一个 Sentinel 包含的主服务器配置比另一个 Sentinel 发送的配置要旧， 那么这个 Sentinel 会立即升级到新配置上。
- 在将一个新 Sentinel 添加到监视主服务器的列表上面之前， Sentinel 会先检查列表中是否已经包含了和要添加的 Sentinel 拥有相同运行 ID 或者相同地址（包括 IP 地址和端口号）的 Sentinel ， 如果是的话， Sentinel 会先移除列表中已有的那些拥有相同运行 ID 或者相同地址的 Sentinel ， 然后再添加新 Sentinel 。

### Sentinel API

有两种方式可以和 Sentinel 进行通讯：

- 第一种方法是通过直接发送命令来查询被监视 Redis 服务器的当前状态， 以及 Sentinel 所知道的关于其他 Sentinel 的信息， 诸如此类。
- 另一种方法是使用发布与订阅功能， 通过接收 Sentinel 发送的通知： 当执行故障转移操作， 或者某个被监视的服务器被判断为主观下线或者客观下线时， Sentinel 就会发送相应的信息。

### Sentinel 命令

以下列出的是 Sentinel 接受的命令：

- [PING](http://www.redis.cn/commands/ping.html) ：返回 PONG 。
- SENTINEL masters ：列出所有被监视的主服务器，以及这些主服务器的当前状态。
- SENTINEL slaves ：列出给定主服务器的所有从服务器，以及这些从服务器的当前状态。
- SENTINEL get-master-addr-by-name ： 返回给定名字的主服务器的 IP 地址和端口号。 如果这个主服务器正在执行故障转移操作， 或者针对这个主服务器的故障转移操作已经完成， 那么这个命令返回新的主服务器的 IP 地址和端口号。
- SENTINEL reset ： 重置所有名字和给定模式 pattern 相匹配的主服务器。 pattern 参数是一个 Glob 风格的模式。 重置操作清楚主服务器目前的所有状态， 包括正在执行中的故障转移， 并移除目前已经发现和关联的， 主服务器的所有从服务器和 Sentinel 。
- SENTINEL failover ： 当主服务器失效时， 在不询问其他 Sentinel 意见的情况下， 强制开始一次自动故障迁移 （不过发起故障转移的 Sentinel 会向其他 Sentinel 发送一个新的配置，其他 Sentinel 会根据这个配置进行相应的更新）。

### 发布与订阅信息

- 客户端可以将 Sentinel 看作是一个只提供了订阅功能的 Redis 服务器： 你不可以使用 [PUBLISH](http://www.redis.cn/commands/publish.html) 命令向这个服务器发送信息， 但你可以用 [SUBSCRIBE](http://www.redis.cn/commands/subscribe.html) 命令或者 [PSUBSCRIBE](http://www.redis.cn/commands/psubscribe.html) 命令， 通过订阅给定的频道来获取相应的事件提醒。

### 脑裂问题成因与解决

成因：哨兵与Redis节点网络波动，导致哨兵认为master下线，并重新选举新的master，而master实际未宕机，仍与客户端连接，此时集群中存在两个master，都写入数据会导致数据差异。

解决：配置如下项，当master不能同时满足时，master将不能写入数据，避免了数据分别写入多个master的问题

```
# 如果有少于 N 个从节点连接且滞后时间小于或等于 M 秒，主节点可以停止接受写入操作。
# 这些 N 个从节点需要处于“在线”状态。
# 滞后时间必须小于或等于指定值，以秒为单位，是从上次接收到的从节点ping的时间计算出来的，
# 通常每秒发送一次。
# 此选项不能保证 N 个副本会接受写入操作，但会限制在不足够的从节点可用时丢失写入的暴露窗口# 到指定的秒数。
# 例如，要求至少有 3 个滞后时间小于或等于 10 秒的从节点，请使用：
min-slaves-to-write 3
min-slaves-max-lag 10
```





