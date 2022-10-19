# lec03 - Database Storage I
数据库基于page管理数据。A page is a fixed-size block of data。
![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511155226.png)

不同的DBMS使用不同的方式管理page。
![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511155520.png)

A heap file is an unordered collection of pages where tuples that are stored in random order.

Two ways to represent a heap file:
→ Linked List
→ Page Directory（常用）

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511155709.png)

page常用的结构如下图所示。这和内存中的栈和堆有些类似，都是从两端开始，可以充分利用page的空间。
![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511155742.png)

tuple的结构如下图所示。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160025.png)


数据库会使用record id来跟踪每个tuple的位置。不同数据库的record id略有不同。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511155912.png)
