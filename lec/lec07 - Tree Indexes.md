# lec07/08 - Tree Indexes
B树族都是**多路自平衡树**。通常一个节点就是一页的大小（一般8k或者16k），配合缓存的特性。增删改的时间复杂度都为O(logn)。

B树和B+树，B树是将每个节点都存储数据，B+树叶节点存储数据，内部节点不存数据。

B+树是完美平衡的，所有叶节点的深度都一样，每个节点都是半满的。M/2-1 ≤ keys ≤ M-- 1。每个节点都有K个key和K+1个child节点。

叶节点中会存储深度、前后节点的指针，和key/value对。value有两种，一种是存储其指向的tuple地址的指针，一种是直接存储tuple，其中外键索引只会存储地址（因为不会存两份数据）。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220826152243.png)

插入：
- 首先找到对应的leaf node，将key/value pair按照顺序插入到节点中
- 如果空间不够，则将节点分裂，并在父节点中新增一个key
- 如果父节点空间也不够，则递归分裂直到根节点。

删除：
- 找到对应节点的leaf node，删除该entry，如果节点处于半满状态，则操作结束，否则从兄弟节点中借来entry
- 如果兄弟节点借不到，则将其与兄弟节点合并，并递归删除父节点中的entry。

### Clustered Index
Clustered Index指Table存储的数据和索引一致，为了节约空间，一个table只能创建一个cluster index，一般是按照primiary key排序，有些数据库不支持clustered Index

### Compound Index
即同时针对多个字段建立索引。该索引可以被用在包含A的条件查询中，如果查询条件中不包含A，虽然该索引仍能一定程度上加速查询，但约等于权表扫描。

### B+树的设计
- Node的大小
	- 一般IO越慢，Node越大
- 延迟Merge
	- 由于Merge操作的成本较高，因此有时候选择延迟Merge
- 对变长key的处理
- 值不唯一的索引
- node内部的搜索

优化策略
- 前缀压缩：同一个Leaf Node，通常有相同的前缀
	- ![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220906194531.png)

- 后缀截断：对于inner node，只保留部分前缀就可以起到索引的作用。
- 批量插入：如果需要大批量插入key，最好的方式是排序后从上至上地构建B+树。
- 指针混合：如果已经将page存储在bufferpool中，则可以在node中存储对应page的内存指针。

### 索引的进阶使用
- 隐式索引：很多数据库会自动创建一些index，包括主键索引、唯一索引、外键索引。
- 部分索引：只针对一张表中的部分数据建立索引，通常是基于一个条件来为索引分区，例如不同月份的数据分开建索引。
- Covering Index，直接将查询所需的字段存入index。
- 表达式索引，index中的不一定是原始的column值，而是通过函数或表达式运算后的结果只

### Trie树和Radix树
Radix树就是压缩后的Trie树。Trie树将一个key拆分为一个序列，相同前缀的序列共享节点。常用于基于前缀的搜索中，例如搜索引擎，打字的自动提示等。

### 关键字搜索
虽然Tree索引在处理条件查询和range查询时表现良好，但是对于keyword查询就无能为力。例如查询一个文本中是否存在某句话。此时应该使用