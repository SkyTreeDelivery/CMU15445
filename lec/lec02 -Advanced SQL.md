# lec02 - Advanced SQL
## SQL标准的发展。
![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511151339.png)

## SQL的分类
- Data Manipulation Language (DML)
- Data Definition Language (DDL)
- Data Control Language (DCL)

Also includes:
- View definition
- Integrity & Referential Constraints
- Transactions

不同的数据库对于统一功能的实现有很多差异

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220508161814.png)

不同数据库对于字符串的支持

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220508170051.png)

## 聚合函数与having
where语句先于聚合函数执行，因此需要使用having。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511151719.png)

## window function
窗口函数类似于聚合函数，但是其结果只是附加在原始数据上，而不是进行分组输出。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511151826.png)

如果想只用窗口函数的结果，需要在子查询中先执行窗口函数。这是因为在执行where语句时，窗口函数还未执行。

![](https://zhang113751picgo.oss-cn-hangzhou.aliyuncs.com/img/20220511151807.png)

## common table expressions
可以基于CTE来简化sql嵌套。

可以用CTE实现递归。
