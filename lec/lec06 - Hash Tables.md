# lec06 - Hash Tables
## DBMS中的数据结构
DBMS中的数据结构用于不同的组件，包括：
- Internal Meta-Data，内部元数据，跟踪数据库系统信息与状态
- Core Data Storage，用于跟踪tuple在系统中的存储
- 临时数据结构，例如join操作会建立临时的数据结构
- Table Indexs，用于更快递查询特定的数据

设计目标：对于数据的高效管理和访问；并发控制。

## Hash Table简介
Hash索引主要关注两个部分：Hash函数和冲突处理

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291808748.png)

## 业界动态
目前比较先进的 Hash 算法是 Facebook 提出的 XXHash

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291810739.png)

不同算法的性能比较
![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291810326.png)


### 静态Hash
适合用于提前知道数据规模的情景。没有扩容和缩容操作。

常用方法有：
- **线性哈希（Linear Probe Hashing）**，最简单的Hash方法，容易导致look up 退化为$O(n)$。
- **罗宾汉哈希（Robin Hood Hashing）**，记录key距离初始slot的距离，使距离分布比较均匀。
- **布谷鸟哈希（Cuckoo Hashing）**，同时有多个Hash Table，每个Hash Table只探测$O(1)$次，每个key在一个HashTable中有唯一确定的slot。当某个key的slot在所有HashTable中都发生冲突时，尝试交换元素的位置，重新分配slot。如果发生了无限循环，则增加table，并重新分配元素。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291819895.png)

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291819035.png)


![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291819031.png)

### 动态Hash
适合数据规模动态变化的场景，支持扩容与缩容。

常见方法有：
- **链式Hash（Chained Hashing）**，简单地在每个slot后面增加bucket（page）。
- **可扩充Hash（Extendible Hashing）**，将key映射为一串二进制代码，当bucket满时split，并增加分类探测的位数，反之则合并bucket。多个slot可以指向同一个bucket。
- **线性Hash（Linear Hashing）**，通过一个指针指向下一次要split的bucket，每当出现一个overflow的slot时，就split该pointer指向的bucket，并将指针向后移动一位。维护两个hash函数，当hash1的结果落在pointer之上时，再执行一次hash2。每当pointer指向地n个slot时（此时一共有2n个slot），就重置指针。每个slot指向一个bucket链表。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291836374.png)

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291841016.png)


![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202206291841298.png)

在数据库中，B+树更常用于Table Index，因为Hash Table不支持range search，但是B+树支持Range Search。