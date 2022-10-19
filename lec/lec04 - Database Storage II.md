# lec04 - Database Storage II

## Data Representation
sql中定义的数据类型及其标准。

字符串一般有三种定义方式，这里是第一种，即在数据bytes中描述了字符串的长度。


![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160124.png)

如果浮点数的精度不能满足需求，可以使用numeric和decimal来表示精确小数。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160400.png)

每个page能存储的数据是有限的，如果tuple过大，则会使用overflow page来鵆数据。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160543.png)

也可以使用blob类型字段来连接到数据库之外的文件，但是数据库不会负责文件的安全性。
## System Catalogs
数据库中会有一个保存元数据的数据库

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160814.png)

postgresql中可以通过`information_schema.tables`来查询已有的数据table

```sql
SELECT * FROM information_schema.tables
```

## OLTP & OLAP & HTAP
OLTP和OLAP是数据库应用的两大场景。

OLTP (On-line Transaction Processing)，指少量多次的简单读/写sql，

OLAP (On-line Analytical Processing)，指低频率大数据量分析。

HTAP(ybrid Transaction + Analytical Processing)，同时执行OLTP和OLAP的数据库

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511160949.png)

## 数据存储模型
N-ARY STORAGE MODEL(NSM)，适用于OLTP，每个tuple的所有数据连续存储。因为每个page的数据会被一次性加载到内存，即使只需要其中一小部分，所以理论上NSM不适合OLAP。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511161649.png)


DECOMPOSITION STORAGE MODEL(DSM)，适用于OLAP，一个relation的所有tuple的每个数据连续存储。对于针对大量tuples的少量字段的聚合分析，可以有效减少IO的数量。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511161639.png)


这一点和Matlab中强调的列优先计算很相似。