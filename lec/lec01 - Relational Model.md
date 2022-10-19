# lec01 - Relational Model
## 基本概念
relation和tuple的概念：
- relation：对应table
- tuples：对应rows，一个tuple就是一个row。值通常是原子的，即不可分的，数值的，即课比较和排序。

在SQL规范中，自增id是通过sequence实现的，postgresql遵循了sql规范，而MySQL和SQLite则使用AUTO_INCREMENT关键字。

## 数据库的类型
- Relational => 大部分 DBMS 属于关系型，也是本课讨论的重点
- Key/Value
- Graph
- Document
- Column-family
- Array/Matrix

## Data Manipulation Languages (DML)
分为两种：
- Procedural：查询指令需要指定DBMS执行时的具体查询策略，例如关系代数。
- NonProcedural：查询命令只指定需要查询的数据，不关心DBMS背后做的事情，例如SQL。

## 关系代数
relational algebra 是基于 set algebra 提出的，从 relation 中查询和修改 tuples 的一些基本操作，它们包括：
- Select ($σ$)
- Projection ($π$)
- Union ($∪$)
- Intersection ($∩$)
- Difference ($−$)
- Product ($×$)
- Join ($⨝$)
- Rename ($ρ$)
- Assignment ($R←S$)
- Duplicate Elimination ($δ$)
- Aggregation ($γ$)
- Sorting ($τ$)
- Division ($R÷S$ )

基于关系代数的链式组合，我们可以实现查询和管理数据。

**注意**：使用 Relation Algebra 时，我们实际上指定了执行策略，如：
$$σb_id=102​(R⨝S)vs.(R⨝(σb_id=102​(S))$$
它们所做的事情都是 ”返回 R 和 S Join 后的结果中，b_id 等于 102 的 tuples“。

虽然 Relational Algebra 只是 Relational Model 的具体实现方式，但在之后的课程将会看到它对查询优化、执行的帮助。