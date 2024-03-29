# 深入理解Java虚拟机——读书笔记

## 自动内存管理机制

### Java内存区域与内存溢出异常

#### 运行时数据区域

- 程序计数器
- Java虚拟机栈
- 本地方法栈
- Java堆
- 方法区
- 运行时常量池
- 直接内存

#### Hotspot虚拟机对象探秘

##### 对象的创建

- 定位new的类的符号引用，检查该符号引用代表的类是否已加载，未加载需要执行类加载过程
- 类加载完成后可确定对象所需内存的大小
- 分配内存空间，若内存区域规整使用**指针碰撞**的方式，不规整需要使用**空闲列表**的方式
- 同步处理分配空间（一般是CAS）的操作或在Java堆中预先给每个线程分配**本地线程分配缓冲**（Thread Local Allocation Buffer，TLAB）保证分配操作原子性
- 给分配的内存空间初始化为0值（不包括对象头）
- 设置对象头：属于哪个类、元数据信息、哈希码、GC分代年龄
- 执行<init>方法，初始化对象

##### 对象的内存布局

- 对象头
  - Mark Word：存储对象运行时数据，如哈希码、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，长度为32位或64位（会根据实际情况复用空间，保证能在32或64位的长度下能保持这么多数据）
  - 类型指针
  - 数组长度（对于数组实例）
- 实例数据
- 对齐填充

##### 对象的访问定位

通过栈上引用数据访问操作堆上的具体对象，JVM规范没有定义具体访问方式，实际由虚拟机实现决定

- 句柄池：引用存储了类型数据和指向堆中维护的句柄池的地址，再由句柄池中的地址指向堆中的对象
- 直接指针访问：直接指向堆中地址，这种方式堆对象头必须维护类型数据

##### 实战：OOM

- 堆溢出
- 虚拟机栈和本地方法栈溢出
- 方法区和运行时常量池溢出
- 直接内存溢出

### 垃圾收集器与内存分配策略

- 哪些内存需要回收
- 什么时候回收
- 如何回收

#### 对象已死吗

##### 引用计数算法

- 优点：简单快速
- 缺点：有循环引用问题

##### 可达性分析算法

可作为GC Roots的对象

- 虚拟机栈引用的对象
- 本地方法栈中引用的对象
- 方法区静态属性引用的变量
- 方法区中常量引用的变量

##### 再谈引用（引用分类）

- 强引用：只要存在，就不会被回收
- 软引用：即将溢出前，会尝试回收，若内存仍不够用才会报OOM
- 弱引用：下次GC即回收
- 虚引用（幻象引用）：不影响对象的生命周期，也不能通过虚引用获得对象实例，唯一作用是GC收到系统通知

##### 生存还是死亡（什么时候对象真正被回收）

- 若GC Roots不可达，则不可达对象将被第一次标记并做筛选。判断是否有必要执行finalize方法，对象未覆写finalize方法或虚拟机已调用过视为”没有必要执行“
- 被视为“有必要执行”的对象将进入F-Queue队列，稍后执行
- 若对象的引用再finalize方法中重新与GC Roots链上对象关联，则第二标记时将被移除即将回收集合

##### 回收方法区

主要回收废弃常量和无用的类

废弃常量：

无用类

- 所有实例已被回收
- 加载该类的ClassLoader已被回收
- 该类对于的``java.lang.Class``对象没有在任何地方被引用

### 垃圾回收算法

- 标记-清除
- 复制
- 标记-整理
- 分代收集

### HotSpot算法实现

- 枚举根节点
- 安全点
- 安全区域

### 垃圾回收器

#### 垃圾回收器

- Serial
  - 复制算法
  - 单线程
- ParNew
  - 复制算法
  - 多线程
  - Serial的多线程版本
  - -XX:SurvivorRatio=x
    - 新生代中Eden区和Survivor区比率Eden : from : to = x : 1 : 1
  - -XX:PretenureSizeThreshold
    - 超过阈值则对象直接分配进老年代
  - -XX:HandlePromotionFailure
    - 1.7开始，该参数失效
- Parallel Scavenge
  - 复制算法
  - 多线程
  - 追求吞吐量
  - 相比ParNew多了自适应功能
  - -XX:MaxGCPauseMillis
    - 设置最大GC收集时间
  - -XX:GCTimeRatio
    - 设置GC时间占总时间的比率
    - 19表示1/(1+19)=5%的GC比率
  - -XX:UseAdaptiveSizePolicy
    - 开启后jvm自适应存活区比率和晋升老年代大小
- Serial Old
- Parallel Old
- CMS
- G1
  - 原理
    - G1收集器仍保留分代概念，将堆分成若干相等大小的Region区，不同的region扮演不同代的角色，设置Humongou区以处理大对象。
    - G1收集器跟踪各个region垃圾堆积的价值，优先回收价值高收益大的region
  - 流程
    - 初始标记（Initial Marking）：仅仅只是标记一下GC Roots能直接关联到的对象，并且修改TAMS
      指针的值，让下一阶段用户线程并发运行时，能正确地在可用的Region中分配新对象。这个阶段需要
      停顿线程，但耗时很短，而且是借用进行Minor GC的时候同步完成的，所以G1收集器在这个阶段实际
      并没有额外的停顿。
    - 并发标记（Concurrent Marking）：从GC Root开始对堆中对象进行可达性分析，递归扫描整个堆
      里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以
      后，还要重新处理SATB记录下的在并发时有引用变动的对象。
    - 最终标记（Final Marking）：对用户线程做另一个短暂的暂停，用于处理并发阶段结束后仍遗留
      下来的最后那少量的SATB记录。
    - 筛选回收（Live Data Counting and Evacuation）：负责更新Region的统计数据，对各个Region的回
      收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个Region
      构成回收集，然后把决定回收的那一部分Region的存活对象复制到空的Region中，再清理掉整个旧
      Region的全部空间。这里的操作涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行
      完成的。

JVM垃圾回收器相关参数

|                   | 单/多线程 | 收集算法  | 是否会stw | 应用场景                                    |
| ----------------- | --------- | --------- | --------- | ------------------------------------------- |
| Serial            | 单线程    | 复制      | 会        | Client模式下新生代默认收集器                |
| Serial Old        | 单线程    | 标记-整理 | 会        | 老年代                                      |
| ParNew            | 多线程    | 复制      | 基本不会  | 新生代，关注停顿时间                        |
| Parallel Scavenge | 多线程    | 复制      | 基本不会  | 新生代 吞吐量优先，相比ParNew多了自适应功能 |
| Parallel Old      | 多线程    | 标记-整理 | 基本不会  | 老年代 吞吐量优先                           |
| CMS               | 多线程    | 并发-清除 | 部分不会  | 追求短停顿                                  |
| G1                |           |           |           |                                             |
| ZGC               |           |           |           |                                             |

##### gc相关参数

- 堆
  - ``-Xms1600m ``：JVM初始分配的内存由-Xms指定，默认是物理内存的1/64
  - ``-Xmx1600m``：JVM最大分配的内存由-Xmx指定，默认是物理内存的1/4
  - ``-XX:PermSize`` :设置非堆内存初始值，默认是物理内存的1/64
  - ``-XX:MaxPermSize``: 设置最大非堆内存的大小，默认是物理内存的1/4
  - ``-Xmn2G``：设置年轻代大小为2G
- 新生代
  - ``-XX:SurvivorRatio``：新生代中Eden区占比，如8表示Eden：from：to = 8:1:1
- ParNew
  - ``-XX：ParallelGCThreads``: 限制垃圾收集的线程数
- Parallel Scavenge
  - ``-XX：MaxGCPauseMillis``:最大垃圾收集停顿时间(大于0的毫秒数)
  - ``-XX：GCTimeRatio``:吞吐量大小，大于0小于100的整数，99表示 1/(1+99)=1%，即允许1%的GC时间
  - ``-XX：+UseAdaptiveSizePolicy``:开启后系统自适应新生代的大小（-Xmn）、Eden与Survivor区的比例（-XX：SurvivorRatio）、晋升老年代对象大小（-XX：PretenureSizeThreshold）等细节参数
- CMS
  - ``-XX：+UseConcMarkSweepGC``:开启CMS垃圾回收器
  - ``-XX:CMSInitiatingOccupancyFraction``：设置CMS触发Full GC的老年代阈值（CMS无法处理浮动垃圾，所以需要预留一部分空间处理浮动垃圾问题）
  - ``-XX:+UseCMSCompactAtFullCollection``：设置CMS在撑不住需要Full GC时进行压缩整理，默认开启(JDK9开始废弃该参数)
  - ``-XX:CMSFullBeforeCompaction``：设置执行多少次不带压缩的Full GC后，来一次带压缩的Full GC，默认0，表示每次进入Full GC都需要整理碎片
- G1
  - ``-XX：+UseG1GC``:Java 7和8之前显示开启G1收集器，Java 9之后G1时默认收集器，6以及6之前不支持G1
  - ``-XX：G1HeapRegionSize``:设置G1收集器Region区大小，，取值范围为1MB～32MB，且应为2的N次幂。超过Region区一般大小的对象将视为大对象放入Humongou区

#### 理解GC日志

### 内存分配与回收策略

#### 空间分配担保

- 进行minor gc之前，检查老年代剩余的最大可用连续空间是否大于新生代所有对象的总空间，如果大于说明此次Minor GC确保安全
- 若不大于，判断设置是否允许担保失败，若允许，判断老年代剩余最大连续可用空间是否大于历次晋升对象的平均值，若大于继续Minor GC，否则执行Full GC
- JDK 6 Update 24之后，担保失败设置参数失效，默认允许担保失败