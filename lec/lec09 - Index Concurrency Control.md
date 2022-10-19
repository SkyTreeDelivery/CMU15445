# lec09 - Index Concurrency Control

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220906201335.png)


## Latch Modes
读模式：多个线程读取东一个数据，可以多次取得
写模式：同一时间只能单个线程访问，如果有其他线程获得了任意模式的锁，都无法获得写锁。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220906201124.png)

### 锁的实现
- 悲观锁
- 乐观锁
- 读写锁

### HashTable Index的锁
基于page的锁
基于slot的锁

### B+树并发控制
我们希望能尽可能多地允许多个线程同时读写B+Tree。
主要考虑两种情况：
- 多个线程同时修改一个node
- 一个线程在遍历B+Tree时，另一个线程在split和merge node

### Latch Crabbing
解决方案：
1. 先获得parent的latch
2. 然后获得child的latch
3. 如果parent是安全的（即后续的操作不会发生merge和split，即不是半满、能借到key或不是全满），则释放parent的latch

这样可以尽量少地使node处于被写锁控制的状态。

### Better Latching Algorithm
实际上在很多时候都不会出现split和merge，因此我们可以乐观的认为一次写操作，除了leaf node，不需要用写锁住其他inner node。

在查询路径上一路获取、释放 read latch，到达 leaf node 时，若操作不会引起 split/merge 发生，则只需要在 leaf node 上获取 write latch 然后更新数据，释放 write latch 即可；若操作会引起 split/merge 发生，则重新执行一遍，此时在查询路径上一路获取、释放 write latch，即 Latch Crabbing 原始方案

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220906202126.png)

### Horizontal Scan
横向扫描，主要问题是可能产生死锁，一般的解决方案是，如果要进行横向搜索，如果无法获得下一个node的latch，就释放该latch后自杀。

### Delayed Parent Updates
因为修改的成本较高，当leaf node要溢出时，仅标记而暂时不更新parent node，下次该节点获取到写锁时一并修改。
