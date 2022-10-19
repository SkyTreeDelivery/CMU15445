# lec05 - Buffer Pools
DBMS会在内存中申请一块内存区域，即BufferPool，用于存储从硬盘中读取的page。bufferpool中的每个page被称为frame。

同时维护一个PageTable，其中存放bufferpool的page的元数据，包括每个page的实际位置（即对bufferpool中page的引用）、是否被写过（Dirty Flage）和被引用的次数（Pin Counter）等。

当page table指向的page被引用是（例如被执行引擎引用），会记录引用数，表示该page正在被使用，空间不够时不应被移除。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202205231041490.png)

如果page table中不包含被请求的page，dbms被申请一个latch，表示该frame被占用，其他人不要再使用

frame，然后从disk中把对应的page读取到bufferpool中，最后释放latch。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/202205231046302.png)

## Multiple buffer pools
DBMS会同时维护多个Buffer Pools，目的是减少每个Buffer Pool的并发锁数量，同时提高数据的locality。
- 基于hash将page分配到不同的Buffer Pools实例上
- 每个数据库分配一个Buffer Pools
- 每中page分配一个buffer Pools

一些优化策略：
- **prefetching**：基于查询策略提前将page缓存
- **scan sharing**：不同的查询使用彼此查询出的数据
- **buffer pool bypass**：对于一些大批量的数据查询，可能很多page只会使用一次，如果将其存入buffer pool会污染缓冲池，即导致buffer pool为了存储不常用的page而移除了其他更常用的page。因此单独分配一块内存，只用于本次查询。
- **os page cache**：绝大多数的数据库都不使用os缓存，除了postgresql，这是因为postgresql是主要为了学术目的研发的数据库，为了简化开发难度专注于数据库本身，直接使用了os缓存。

## Buffer Replacement Policies
缓存替换策略用于在buffer pool空间满时溢出不常用的page。

常用的缓存替换策略有：
- LRU：每个page记录上一次被访问的时间戳，每次都移除时间戳最早的page
- Clock：Clock和LRU很相似，但是不需要记录时间戳，也不需要LRU有序。每个page有一个referenct bit，从来记录是否在轮询期间被访问。
- LRU-K：LRU和Clock都容易被sequential flooding现象影响，即最近访问的page可能是最不常用的page，因此出现了LRU-K。LRU-K在LRU的基础上，记录了最近K次访问的时间戳，用于推断下一次被访问的时间。**K=1即有比较好的效果**。普通的LRU算法相当于LRU-1，一般采用LRU-2，平衡缓存的sequential flooding现象和有效性。