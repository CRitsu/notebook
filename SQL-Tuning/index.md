---
title: Oracle 数据库 SQL 性能调优实践笔记
subtitle: 亿级数据量下 Oracle 数据库 SQL 性能调优初级解决方案。
date: 2018-12-31 19:54:38 +8
tags:
  - Oracle
  - SQL
  - Database
---

目前我所在的项目正在构建一个 ETL 系统，用来帮助客户清洗数据。经过了我们大半年时间的努力，项目快要接近尾声了，目前已经向客户请求了“真实数据”，正在做最后的线上环境测试。

在使用“真实数据”的线上测试的过程中，有一些任务出现了性能问题，这篇文章主要是对这些问题的解决方法做一个笔记，不过在进入正文之前，我想稍微说一下这些问题出现的原因。

在得到“真实数据”之前，我们一直不知道这几本出问题的任务所涉及的表的数据量是多少，而在之前的测试中一直是基于最多几百条数据的情况下做的，这些数据量还体现不出来问题，但是当数据量达到千万级和亿级的时候，性能差距就很明显了。

其根本原因还是在设计阶段，没有明确和客户讨论过表的数据规模，开发阶段也没有带入对性能问题的考虑，最终导致了本地少量数据测试完全没问题，到了线上大规模数据下测试一下就暴露性能问题的结果。

### 案例和解决方案

我们先来看一遍所有的案例、解决方案，和大概的思路，至于具体的思路和方法在最后详细说明。

本次实践的难度为初级，实际用到的解决方法从易到难，有以下几种：

- 添加索引，针对表读取大于写入的场景；
- 优化有性能问题的关键字；
- 修改业务逻辑。

当我们开始调整一条 SQL 时，第一步一般是查看优化器的执行计划，大致定位出问题的原因。通常情况下，出现性能问题的原因都是由于某一段 SQL 走了全表检索，遍历了一张大表。对于这种情况，如果没有相应的索引的话，通过建立索引表可以得到明显改善。不过如果是存在索引但是没有走索引的情况下，会需要使用 `hint` 来强制优化器使用指定的索引来达到效果。

某些关键字也会影响优化器的策略，我们遇到一个案例，一段 SQL 不合时宜的使用了 `in` 关键字造成性能问题，添加索引没有效果，最终将其转化为 `inner join` 配合索引有了明显的效果。

修改业务逻辑的方法应用场景比较少，但有时会有奇效。例如在我们遇到的一个问题中，正常的业务逻辑下需要更新大量的数据，结合数据和业务逻辑分析，我们发现可以通过修改业务逻辑，在保持功能性的前提下减少大部分的更新量。这个案例在后面还会进一步说明。

除此之外还会稍微记录一下下面这些解决方案，这些方案有些是研讨中还没有运用到实际问题上去的，不过都是些确实有效果的方法：

- 使用 partition 表分区提高效率；
- 使用 `nologging` 减少 redo log 提高写入效率；
- 分割复杂 SQL 为多个简单 SQL，用中间表减少内存负担；
- 用 `merge` 代替 `update` 做数据更新提高效率；
- 压缩表，利用空闲 CPU 减少 IO 时间。

下面来看看具体的案例。

#### 添加索引

添加索引可以大幅提高查询速度，但是代价是插入和更新语句会相应变慢。索引提高查询速度的原理是“空间换速度”，当创建一个索引的时候，数据库会提取出所有数据的相应字段创建一张索引表，这样如果一个查询语句使用了该索引，那么这条语句只会对索引表进行检索，以此来提高速度。

举例说明：

如果一张表有 150 个字段，一个查询 SQL 的 `where` 条件仅包含了其中 5 个字段，此时如果没有索引的话，这条查询 SQL 会对有 150 个字段的全表进行检索。

这个情况下如果对查询 SQL 所包含的 5 个字段创建一个联合索引，那么最终的结果，这个查询 SQL 只会对有 5 个字段的索引表进行检索。相对于全表检索，肯定是索引表检索的效率更高。

创建索引表有负面影响，其一是会占用更多空间。围绕上面这个例子 🌰 来说，创建索引表会提取所有数据的相应的 5 个字段单独储存来提高速度。

第二个影响是会让插入和更新变慢。当没有索引表的情况下，一个插入或更新语句只需要对主表操作即可，但是如果创建了索引表，插入或更新语句在操作完主表之后，还要更新相应的索引表。

总结：索引适合一张表读取多于写入的情况。索引仅对查询 SQL 有效果，包括子查询。索引越多，对 DML 语句影响越大，合适对策略是仅创建够用的索引。

具体代码不方便展示，下面仅展示几个修改前后执行效率的例子。当然最后会用测试数据做一个重现。

**例子 🌰1**

| 输入表     |      数据量 |
| ---------- | ----------: |
| 顾客表     | 138,066,840 |
| 顾客信息表 | 145,586,869 |
| 业务表     |      16,220 |

输出数据量为：7,442。

| &nbsp;     | 运行时间 |
| ---------- | -------- |
| 创建索引前 | 2:02:27  |
| 创建索引后 | 0:01:17  |

这个例子中，虽然最终输出结果仅为 7 千行，但由于 `顾客表` 和 `顾客信息表` 都是拥有百来个字段的大表，全表检索非常吃力。优化的方法是对所有查询条件的字段创建了索引，效果很明显。

**例子 🌰2**

| 输入表 |      数据量 |
| ------ | ----------: |
| 顾客表 | 138,066,840 |
| 业务表 |  43,850,757 |
| 种类表 |          16 |

输出数据量为：43,599,931。

| &nbsp;     | 运行时间 |
| ---------- | -------- |
| 创建索引前 | 3:10:17  |
| 创建索引后 | 1:12:23  |

这个例子由于四千三百万的输出数据量的限制，仅 IO 操作都会花费不少时间，添加索引后把查询部分的时间减少了不少，最终结果是 72 分钟，还算可以接受的结果。

#### 优化有性能问题的关键字

还是之前说到的那个案例。

**例子 🌰3**

| 输入表       |      数据量 |
| ------------ | ----------: |
| 顾客信息表 1 | 145,586,869 |
| 顾客信息表 2 | 294,225,728 |
| 顾客信息表 3 | 261,309,380 |
| 顾客信息表 4 | 140,379,648 |

输出数据量为：149。但是对上面四张表做了不同程度的更新。

| &nbsp; | 运行时间 |
| ------ | -------- |
| 修改前 | 1:58:48  |
| 修改后 | 0:00:03  |

这个案例中，SQL 包含一连串的 `with as` 语句，其中一块语句的查询条件里面存在两个 `in` 关键字，但是 `in` 语句里面都是亿级的大表，造成严重的性能问题，最终通过将其改写为两个 `inner join` 得到改善。

#### 修改业务逻辑

修改业务逻辑的场景一般没有普适性，需要结合数据和业务逻辑单独分析。

**例子 🌰4**

| 输入表     |      数据量 |
| ---------- | ----------: |
| 顾客表     | 138,066,840 |
| 顾客信息表 | 145,586,869 |

输出数据量为：13,764。并且途中做了几次不同程度的更新。

| &nbsp; | 运行时间 |
| ------ | -------- |
| 修改前 | 5:05:53  |
| 修改后 | 0:01:00  |

这个例子中，在正常逻辑下，我们抽出一批数据插入到目标表中，将一个 Flag 更新为初始值 `00`，然后根据各种表关联的结果再将其中一部分数据更新为 `99`。但是实际数据测试中发现需要更新为 `99` 的数据量太大了，同时 `update` 本身又是一个 cost 很高的操作，所以整段 SQL 的执行效率奇慢无比。分析数据和业务逻辑之后，我们将 Flag 初始化更新为 `99`，然后将**不符合条件的数据**更新为 `00`，以此来减少了大量更新操作，大幅度提高了执行效率。

#### 一些其他的解决方案

上面是有具体案例的实践，下面还有一些试验过有效果的方案，但是由于一些原因没有运用到这次遇到的问题上去。都是一些有价值的方法，所以在这里也记录一下。

**使用 partition 表分区提高效率**

创建表分区虽然没有用来解决这次遇到的问题，但是实际上当初设计表的时候已经是做过分区的了。

