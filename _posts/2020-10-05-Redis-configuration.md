---
layout: post
title: Redis配置
categories: [Redis]
description: Redis配置
keywords: Redis
---
# Redis配置

## 进程相关

- ``daemonize no`` : 是否开启守护进程
- ``pidfile /var/run/redis.pid``

## 网络相关

- ``port 6379``
- ``bind 127.0.0.1``
- ``timeout 300``：当客户端闲置多长**秒**后关闭连接，如果指定为 0 ，表示关闭该功能
- `` maxclients 128`` :设置同一时间最大客户端连接数，默认无限制，Redis 可以同时打开的客户端连接数为 Redis 进程可以打开的最大文件描述符数，如果设置 maxclients 0，表示不作限制。当客户端连接数到达限制时，Redis 会关闭新的连接并向客户端返回 max number of clients reached 错误信息
- ``glueoutputbuf yes`` :设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
- ``client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>``
  - client-output-buffer-limit normal 0 0 0 
  -  client-output-buffer-limit replica 256mb 64mb 60
  -  client-output-buffer-limit pubsub 32mb 8mb 60
    - normal -> normal clients including MONITOR clients
      replica  -> replica clients
      pubsub -> clients subscribed to at least one pubsub channel or pattern
  - 客户机输出缓冲区限制可用于强制断开由于某些原因没有足够快地从服务器读取数据的客户机（一个常见的原因是发布/订阅客户机不能像发布者生成消息那样快地使用消息）。
  - 一旦达到硬限制，或者达到软限制并保持达到指定秒数，客户端将立即断开连接
  - 默认情况下，普通客户端不受限制，因为它们不会在没有请求的情况下（以推送方式）接收数据，而是在请求之后才接收数据，因此只有异步客户端可能会创建这样一种情况，即请求数据的速度快于它的读取速度。
  - 相反，pubsub和replica客户端有一个默认限制，因为订阅服务器和副本以推送方式接收数据。

## 安全相关

- ``requirepass foobared`` : 设置 Redis 连接密码，如果配置了连接密码，客户端在连接 Redis 时需要通过 AUTH <password> 命令提供密码，默认关闭

## 日志相关

- ``loglevel notice``:指定日志记录级别，Redis 总共支持四个级别：debug、verbose、notice、warning，默认为 notice
- ``logfile stdout``:日志记录方式，默认为标准输出，如果配置 Redis 为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发送给 /dev/null

## 数据库相关

- ``databases 16``:设置数据库的数量，默认数据库为0，可以使用SELECT 命令在连接上指定数据库id

## 淘汰策略相关

- ``maxmemory <bytes>`` :指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会先尝试清除已到期或即将到期的 Key，当此方法处理 后，仍然到达最大内存设置，将无法再进行写入操作，但仍然可以进行读取操作。Redis 新的 vm 机制，会把 Key 存放内存，Value 会存放在 swap 区
- ``maxmemory-policy noeviction`` : MAXMEMORY策略：达到MAXMEMORY时Redis将如何选择要删除的内容。您可以从以下行为中选择一个
  - volatile-lru : 使用近似的LRU淘汰，仅限具有过期集的key。
  - allkeys-lru : 使用近似的LRU淘汰所有key。
  - volatile-lfu : 使用近似的LFU淘汰，仅限具有过期集的key。
  - allkeys-lfu : 使用近似的LFU淘汰所有key。
  - volatile-random : 删除具有过期集的随机key。
  - allkeys-random : 从所有key中随机抽取key来删除。
  - volatile-ttl : 删除最接近过期时间的key（次要TTL）
  - noeviction : 不要淘汰任何内容，只要在写操作时返回一个错误。
- ``maxmemory-samples 5`` : 每次回收时检查的采样数量

## 主从相关配置

- ``slaveof <masterip> <masterport>`` : 设置当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动从 master 进行数据同步或``replicaof``
- ``masterauth <master-password>`` :当 master 服务设置了密码保护时，slave服务连接 master 的密码
- ``masteruser <username>`` : 但是，如果您使用的是Redis acl（用于Redis version 6或更高版本），并且默认用户无法运行PSYNC命令和/或复制所需的其他命令，那么这还不够。在这种情况下，最好配置一个用于复制的特殊用户，并按如下方式指定主用户配置
- ``replica-serve-stale-data yes`` : 当复制副本失去与主服务器的连接，或复制仍在进行时，复制副本可以以两种不同的方式操作：
  - 如果replica-serve-stale-data设置为“yes”（默认值），则复制副本仍将响应客户端请求，数据可能已过期，或者如果这是第一次同步，则数据集可能为空。
  - 如果replica serve stale data设置为'no'，则除了INFO、replicaOF、AUTH、PING、SHUTDOWN、REPLCONF、ROLE、CONFIG、SUBSCRIBE、UNSUBSCRIBE、PSUBSCRIBE、PUNSUBSCRIBE、PUBLISH、PUBSUB、COMMAND、POST、HOST:和LATENCY以外，所有类型的命令都将返回错误“SYNC with master in progress”。
- ``replica-read-only yes`` :
  - 您可以将副本实例配置为接受或不接受写入。针对副本实例写入数据对于存储一些短暂的数据可能很有用（因为在副本上写入的数据在与主服务器重新同步后很容易被删除），但如果客户端由于配置错误而写入副本，则也可能会导致问题。
  - 注意：只读副本不是为暴露在internet上的不可信客户端而设计的。它只是防止实例被滥用的保护层。默认情况下，只读副本仍会导出所有管理命令，如CONFIG、DEBUG等。在一定程度上，您可以使用“rename command”来隐藏所有管理/危险命令来提高只读副本的安全性。
- ``repl-diskless-sync no`` :复制同步策略：磁盘或网络
- ``repl-diskless-sync-delay 5`` : 
  - 启用无盘复制时，可以配置服务器等待的延迟，以便生成通过套接字将RDB传输到副本的子级。
  - 这一点很重要，因为一旦传输开始，就不可能为到达的新副本提供服务，新副本将排队等待下一次RDB传输，因此服务器会等待一个延迟，以便让更多副本到达。
  - 延迟以秒为单位指定，默认为5秒。要完全禁用它，只需将其设置为0秒，传输将尽快开始。
- ``repl-diskless-load disabled`` :
  - RDB无盘加载是实验性的。由于在此设置中，复制副本不会立即在磁盘上存储RDB，因此在故障转移期间可能会导致数据丢失。RDB无盘加载+不处理I/O读取的Redis模块也可能导致Redis在与主机的初始同步阶段出现I/O错误时中止。只在你做你需要时启用。
  - Replica可以直接从socket加载它从复制链接读取的RDB，或者将RDB存储到一个文件中，并在完全从主服务器接收到该文件之后读取该文件。
  - 在许多情况下，磁盘比网络慢，存储和加载RDB文件可能会增加复制时间（甚至会增加主机的“写入时拷贝”内存和缓冲区）。
  - 然而，直接从套接字解析RDB文件可能意味着我们必须在收到完整的RDB之前刷新当前数据库的内容。因此，提供了如下选项：
    - "disabled"    - Don't use diskless load (store the rdb file to the disk first)
    - "on-empty-db" - Use diskless load only when it is completely safe.
    - "swapdb"      - 直接从套接字解析数据时，在RAM中保留当前db内容的副本。请注意，这需要足够的内存，如果你没有，将有OOM的风险。
- ``# repl-ping-replica-period 10`` :副本以预定义的时间间隔向服务器发送ping。可以使用repl_ping_replica_period选项更改此间隔。默认值为10秒。
- ``# repl-timeout 60`` : 设置下列设置的超时时间，请务必确保此值大于为repl ping replica period指定的值，否则每当主服务器和副本之间的通信量较低时都会检测到超时。
  - 副本同步时，大容量IO的传输时间
  - 从副本连接到master的超时时间（数据，ping）
  - 从master连接到slave的超时时间（REPLCONF ACK pings）
- ``repl-disable-tcp-nodelay no`` :
  - 如果选择“yes”，Redis将使用较少的TCP数据包和较少的带宽将数据发送到副本。但这可能会增加数据在副本端出现的延迟，对于Linux内核使用默认配置，最长为40毫秒。
  - 如果选择“no”，则数据出现在副本端的延迟将减少，但复制将使用更多带宽。
  - 默认情况下，我们会针对低延迟进行优化，但在流量非常大的情况下，或者当主服务器和副本之间有很多跳跃时，将其设置为“yes”可能是个好主意。
- ``repl-backlog-size 1mb``
  - 设置复制缓冲大小。backlog是一个缓冲区，当副本断开连接一段时间后，它会累积副本数据，因此，当副本需要重新连接时，通常不需要完全重新同步，但部分重新同步就足够了，只需传递断开连接时副本丢失的部分数据。
  - 复制积压工作越大，复制副本断开连接的时间就越长，以后能够执行部分重新同步的时间就越长。
  - 仅当至少有一个副本连接时才分配积压工作。
- ``repl-backlog-ttl 3600``
  - 在主服务器一段时间内不再连接副本后，积压工作将被释放。下面的选项配置从上一个复制副本断开连接开始所需的秒数，以便释放backlog缓冲区。
  - 请注意，副本永远不会在超时时释放积压工作，因为它们可能在以后被提升为主控，并且应该能够与副本正确地“部分重新同步”：因此它们应该总是累积积压工作。
  - 值为0意味着永远不会释放backlog。
- ``replica-priority 100``
  - 副本优先级是Redis在INFO输出中发布的整数。Redis Sentinel使用它来选择复制副本，以便在主服务器不再正常工作时升级为主副本。
  - 优先级较低的副本被认为更适合升级，因此，例如，如果有三个优先级为10、100、25的副本，Sentinel将选择优先级为10的副本，即优先级最低的副本。
  - 但是特殊优先级为0会将副本标记为无法执行master角色，因此Redis Sentinel永远不会选择优先级为0的副本进行升级。
  - By default the priority is 100.
- ``min-replicas-to-write 3 / min-replicas-max-lag 10``
  - 如果连接的副本少于N个，并且延迟小于或等于M秒，则主服务器可能停止接受写入。
- ``replica-announce-ip 5.5.5.5 / replica-announce-port 1234`` :
  - Redis主机能够以不同的方式列出所连接副本的地址和端口。例如，“INFO replication”部分提供了这个信息，除了其他工具外，Redis Sentinel使用这些信息来发现副本实例。此信息的另一个可用位置是主控形状的“ROLE”命令的输出。
  - 复制副本通常报告的列出的IP和地址是通过以下方式获得的：
    - 通过检查复制副本用于与主服务器连接的套接字的对等地址，可以自动检测该地址。
    - 该端口在复制握手期间由复制副本通信，通常是复制副本用于侦听连接的端口。
  - 然而，当使用端口转发或网络地址转换（NAT）时，副本实际上可以通过不同的IP和端口对访问。复制副本可以使用以下两个选项向其主服务器报告一组特定的IP和端口，以便INFO和ROLE都报告这些值。

## rdb备份相关配置

- ``save <seconds> <changes>``:  指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合。**save 900 1** **save 300 10** **save 60 10000**分别表示 900 秒（15 分钟）内有 1 个更改，300 秒（5 分钟）内有 10 个更改以及 60 秒内有 10000 个更改。
- ``rdbcompression yes``: 指定存储至本地数据库时是否压缩数据，默认为 yes，Redis 采用 LZF 压缩，如果为了节省 CPU 时间，可以关闭该选项，但会导致数据库文件变的巨大
- ``dbfilename dump.rdb`` :指定本地数据库文件名，默认值为 dump.rdb
- ``dir ./`` :指定本地数据库存放目录
- ``rdb-save-incremental-fsync yes`` : 当redis保存RDB文件时，如果启用以下选项，则每生成32mb的数据将对该文件进行fsync。这对于以增量方式将文件提交到磁盘并避免较大的延迟峰值非常有用。

## aof相关配置

- ``appendonly no`` :指定是否在每次更新操作后进行日志记录，Redis 在默认情况下是异步的把数据写入磁盘，如果不开启，可能会在断电时导致一段时间内的数据丢失。因为 redis 本身同步数据文件是按上面 save 条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为 no 表示RDB备份，改为true即开启AOF
- ``appendfilename appendonly.aof``:指定更新日志文件名，默认为 appendonly.aof
- ``appendfsync everysec`` :指定更新日志条件，共有 3 个可选值：
  - **no**：表示等操作系统进行数据缓存同步到磁盘（快）
  - **always**：表示每次更新操作后手动调用 fsync() 将数据写到磁盘（慢，安全）
  - **everysec**：表示每秒同步一次（折中，默认值）
- ``aof-rewrite-incremental-fsync yes`` : 当子进程重写AOF文件时，如果启用以下选项，则每生成32mb的数据将对该文件进行fsync。这对于以增量方式将文件提交到磁盘并避免较大的延迟峰值非常有用。

## 虚拟内存相关配置

- ``vm-enabled no`` :指定是否启用虚拟内存机制，默认值为 no，简单的介绍一下，VM 机制将数据分页存放，由 Redis 将访问量较少的页即冷数据 swap 到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析 Redis 的 VM 机制）
- ``vm-swap-file /tmp/redis.swap`` :虚拟内存文件路径，默认值为 /tmp/redis.swap，不可多个 Redis 实例共享
- ``vm-max-memory 0`` :将所有大于 vm-max-memory 的数据存入虚拟内存，无论 vm-max-memory 设置多小，所有索引数据都是内存存储的(Redis 的索引数据 就是 keys)，也就是说，当 vm-max-memory 设置为 0 的时候，其实是所有 value 都存在于磁盘。默认值为 0
- ``vm-page-size 32`` :Redis swap 文件分成了很多的 page，一个对象可以保存在多个 page 上面，但一个 page 上不能被多个对象共享，vm-page-size 是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page 大小最好设置为 32 或者 64bytes；如果存储很大大对象，则可以使用更大的 page，如果不确定，就使用默认值
- ``vm-pages 134217728`` :设置 swap 文件中的 page 数量，由于页表（一种表示页面空闲或使用的 bitmap）是在放在内存中的，，在磁盘上每 8 个 pages 将消耗 1byte 的内存。
- ``vm-max-threads 4`` :设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

## 数据结构存储优化相关

- ``hash-max-zipmap-entries 64`` / ``hash-max-zipmap-value 512``: 当hash数据结构的映射条目小于设定值且key或value的长度小于设定值时，采用更高效的数据结构zipmap进行编码
- ``activerehashing yes`` :
  - 主动每100毫秒CPU时间使用1毫秒重新缓存，以帮助重新缓存主Redis哈希表（将顶级键映射到值的表）。
  - Redis使用的哈希表实现（参见dict.c）执行惰性rehash：在Hash表中运行的操作越多，执行的rehash机会越多，因此，如果服务器空闲，则重新散列永远不会完成，哈希表将使用更多内存。
  - 如果不确定：如果您有严格的延迟要求，并且在您的环境中，Redis可以不时地以2毫秒的延迟响应查询，那么使用“activerehashing no”。
  - 如果您没有这样的硬性要求，但希望尽快释放内存，请使用“activerehashingyes”。
- ``list-max-ziplist-size -2`` :当List的大小小于指定的阈值时，使用ziplist节省内存
- ``list-compress-depth 0`` :ziplist的压缩深度，0-不压缩，1-压缩除head和tail之间所有 etc，为确保快速的push和pop，head和tail不会被压缩
- ``set-max-intset-entries 512`` :集合只有一种特殊的编码方式：当一个集合由恰好是基数为10的64位有符号整数范围内的整数组成时。为了使用这种特殊的内存节省编码，下面的配置设置设置了集合大小的限制。
  - -5: max size: 64 Kb  <-- not recommended for normal workloads
  - -4: max size: 32 Kb  <-- not recommended
  - -3: max size: 16 Kb  <-- probably not recommended
  - -2: max size: 8 Kb   <-- good
  - -1: max size: 4 Kb   <-- good
- ``zset-max-ziplist-entries 128`` / ``zset-max-ziplist-value 64`` : 与哈希和列表类似，排序集也经过特殊编码，以节省大量空间。此编码仅在排序集的长度和元素低于以下限制时使用
- ``hll-sparse-max-bytes 3000`` : 
  - HyperLogLog稀疏表示字节限制。该限制包括16字节头。当使用稀疏表示的超日志超过此限制时，它将转换为密集表示.
  - 大于16000的值是完全无用的，因为在这一点上密集表示更节省内存。
  - 建议值为~3000，以便在不减慢过多PFADD（稀疏编码为O（N））的情况下利用空间高效编码的优点。当CPU不是问题，但空间是问题，并且数据集由许多基数在0-15000范围内的超日志组成时，该值可以提升到~10000。
- ``stream-node-max-bytes 4096`` / ``stream-node-max-entries 100`` : Streams宏节点最大大小/项数。流数据结构是一个由大节点组成的基数树，其中对多个项进行编码。使用此配置，可以配置单个节点的大小（以字节为单位），以及在附加新的流条目时切换到新节点之前可能包含的最大项数。如果将以下任何设置设置为零，则会忽略该限制，因此，例如，可以通过将max bytes设置为0，将max entries设置为所需的值来设置max entires限制。

## 其他

- ``include /path/to/local.conf`` : 指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件
- ``hz 10`` :后天任务的执行频率
  - Redis调用一个内部函数来执行许多后台任务，比如超时关闭客户端的连接，清除从未被请求的过期密钥，等等。
  - 并非所有任务都以相同的频率执行，但Redis会根据指定的“hz”值检查要执行的任务。
  - 默认情况下，“hz”设置为10。当Redis空闲时，提高该值将占用更多的CPU，但同时也会使Redis在多个密钥同时过期时响应更快，超时处理可能更精确。
  - 范围在1到500之间，但是超过100的值通常不是一个好主意。大多数用户应该使用默认值10，并且只有在需要非常低延迟的环境中才将此值提高到100
- ``notify-keyspace-events "" `` : 键空间通知配置，值是一个非空字符串，枚举及含义如下
  - K     键空间事件，以 ``__keyspace@<db>__ ``前缀发布。
  - E     键事件事件，以`__keyevent@<db>__`前缀发布。
  - g     通用命令（非类型特定），如DEL，EXPIRE，RENAME等等
  - $     字符串命令
  - l     列表命令
  - s     集合命令
  - h     哈希命令
  - z     有序集合命令
  - x     过期事件（每次键到期时生成的事件）
  - e     被驱逐的事件（当一个键由于达到最大内存而被驱逐时产生的事件）
  - A     g$lshzxe的别名，因此字符串AKE表示所有的事件。
  - 字符串中应当至少存在`K`或者`E`，否则将不会传递事件，不管字符串中其余部分是什么。

  

