# HashMap 详解

## 什么是HashMap

* 基于Map接口实现的哈希表，提供了一系列的key-value操作，并且允许key和value为null值. 
* HashMap类除了不同步和允许null上,其他与Hashtable大致相同.  
* 此类不保证映射的顺序,特别是不保证顺序不随时间变化而变化

## 数据结构

- 一个Node类型的数组，数组长度为HashMap的容量，数组内已存在映射数量为HashMap的大小（size）

## HashMap的hash方法

- 将hashCode的高16位与低16异或得到新hash值

## 时间复杂度

- 在hash方法分配良好，没有冲突的情况下，get和put的时间复杂度为O(1)
- 若存在冲突，则节点未树化时为O(n)，树化后为O(log n)

## HashMap的resize流程

- 判断**原table**的长度是否大于0

  - 如果**原table**若大于HashMap的最大容量，直接返回**原table**，不执行resize
  - 若**原table**长度小于HashMap的最大容量且大于默认初始化的大小（16）
    - 设置**新table**的长度为**原table**的2倍
    - 设置新的扩容阈值为原扩容阈值的2倍

- 若原扩容阈值大于0

  - 设置**新table**的长度为原扩容阈值

- 其他

  - 设置**新table**的容量默认容量（16）
  - 新的下次扩容阈值为默认容量 * 默认加载因子

- 遍历**原table**中的所有的节点并放入**新table**
  - 如果节点没有子节点：重hash至**新table**
  - 如果节点为树节点：遍历树节点，判断（hash & oldCap）== 0 是否为true将节点分成两个部分
    - 为true的部分在原节点不动
    - 为false的部分在分配至至新的位置，索引为（oldIndex + oldCap）
  - 如果节点为链表，同树节点处理流程

#### 速记

- table为空时怎么处理
- 扩容后原table上节点怎么迁移到新table（重hash）
  - 单节点槽怎么重hash
  - 链表节点怎么槽重hash
  - 树节点槽怎么重hash
- 什么时候执行树化和链表化

## HashMap的putVal流程

- 方法签名

```java
/**
 * Implements Map.put and related methods
 *
 * @param hash key的hash值
 * @param key the key
 * @param value 要put的值
 * @param onlyIfAbsent 若为true, 不修改已存在的值
 * @param evict 若为true, 说明哈希桶（table）在创建模式，即调用时table为空
 * @return 原value或null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict);
```

- 
- 若哈希桶table为空，resize初始化
- 索引为hash & （n - 1）位置是否为空
  - 若为空，将node节点存放至该索引
  - 若不为空
    - 当即key已存在（key不为空、hash值相等、key的hashCode相等且equals为true时），使用新value替换旧value
    - 当key不存在时
      - 判断对应索引是否为树节点，若是则插入树节点（待研究）
      - 若对应索引节点为链表，遍历将node节点插入链表末尾，并判断链表长度是否大于8，若大于则树化，若key已存在则跳过
- 若onlyIfAbsent为true，不修改原值，默认为false

