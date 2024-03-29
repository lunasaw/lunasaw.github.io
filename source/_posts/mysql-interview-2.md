---
title: mysql面试题
date: 2021-04-25 15:11:34
banner_img: /img/basic/mysql-binner.jpg
index_img: /img/basic/mysql-index.jpg
tags: 
 - mysql
categories:
 - basic-component
 - mysql
---

# Mysql 面试题

## 为什么用自增列作为主键

#### 1. 如果定义了主键Primary Key，那么InnoDB会选择主键作为聚簇索引

- 如果没有显示定义主键，则InnoDB会选择第一个不包含有NULL值的唯一索引作为主键索引。
- 如果也没有这样的唯一索引，则InnoDB会选择内置6字节长度的RowId作为隐含的聚簇索引（RowId随着行记录的写入而主键递增）这个RowId不像Oracle的RowId 那样可引用，它是隐含的。

#### 2. 数据记录本身 存于主索引（一课B+Tree）的叶子结点，这就要求同一叶子结点内（大小为一个内存页或者磁盘页）的各条数据记录按主键顺序存放

- 因此每当有一条新纪录存入，Mysql会根据其主键将其插入适当的节点和位置，如果页面达到装载因子（InnoDB默认为15/16），则开辟一个新的页（节点）

#### 3. 如果表使用自增主键，那么每次插入新纪录，记录就会顺序的添加到当前索引结点的后续位置，当一页写满，就会自动开辟一个新的页

#### 4. 如果使用非自增主键（如身份证号或者学号等），由于每次插入主键的值近似于随机，因此新纪录都要被查到现有索引页中的中间的某个位置

- 此时Mysql不得不为了将新纪录插到合适的位置而移动数据，甚至目标页面可能已经被写回磁盘上而从缓存中清掉，此时又要从磁盘读回来，这增加了很多开销
- 同时频繁移动，分页造成大量的碎片，得到不够紧凑的结构，后续不得不通过Optimize table来重建表并优化填充页面

## 为什么使用数据索引能提高效率

#### 数据索引的存储是有序的

#### 在有序的情况下，通过索引查询一个数据是无需遍历记录的

#### 在极端情况下，数据索引的查询效率为二分查询效率，趋近于log2(N)

## B+树和hash索引的区别

<p class="note note-primary">B+数是一个平衡的多叉树，从根节点到每个叶子结点的高度差不超过1，而且同层级的结点之间有指针相互链接，是有序的</p>

![img](https://i.loli.net/2021/04/25/zQsluwtd5VMDjxa.jpg)

<p class="note note-secondary">哈希索引就是采用一定的哈希算法，把键值转换成新的哈希值，检索不需要类似B+数那样从根结点到叶子结点逐级查找，只需要一次哈希算法即可，是无序的</p>

![img](https://i.loli.net/2021/04/25/PG9aHhrgzRDIblE.jpg)

## 哈希索引的优势

#### 等值查询，哈希索引具有绝对优势（前提是没有大量重复键值，如果有大量重复键值时，效率很低，因为存在大量的哈希冲突，需要散列再散列）

## 哈希索引不适用的场景

1. 不支持范围查询
2. 不支持索引完成排序
3. 不支持联合索引的最左侧匹配规则

## 通常，B+数索引结构适用于大多数场景，像下面这种场景用哈希才更有优势

- 在HEAP表中，如果存储数据的重复度很低（基数很大），对该列数据以等职查询为主，没有范围查询，没有排序的时候特别适合哈希索引，例如

- \# 仅等值查询`select id, name from table where name='李明'; `

1. 而常用的InnoDB引擎中默认使用的是B+树索引，它会实时监控表上索引的使用情况 。
2. 如果认为建立哈希索引可以提高查询效率，则自动在内存中的“**自适应哈希索引缓冲区**”建立哈希索引（在InnDB中默认开启自适应哈希索引）。
3. 通过观察搜素模式，Mysql回利用Index Key的前缀建立哈希索引，如果一个表大部分都在缓冲池中，那么建立一个哈希索引能够加快等值查询。

**注意：在某些工作负载情况下，通过哈希索引查找带来的性能提升远大于额外的监控索引搜索情况和保持哈希表结构带来的开销**

`但在某些时候，在负载高的清空下，自适应哈希索引中添加的read/write锁也会带来竞争，比如高并发的join操作，like操作，和%通配符操作也不适用于自适应哈希索引，可能要关闭自适应哈希索引`

##  B树和B+树的区别

- B树，每个结点都存储key和value，所有结点组成这棵树，并且叶子结点为null，叶子结点不包含任何关键字信息

![img](https://i.loli.net/2021/04/25/dCzkEw31gAlVy9e.jpg)

- B+树，所有的叶子结点包含了全部关键字信息，以及指向含有这些关键字记录的指针，且叶子结点本身依关键字大小自小而大的顺序连接
- 所有非终端结点可以看成是索引部分，结点中仅含有其子树根节点中最大（或最小）关键字，（而B树的非终端结点也可能含有需要查找的有效信息）

![img](https://i.loli.net/2021/04/25/sNupSlGDfm9nFdz.jpg)

## 为什么说B树比B+树更结合实际应用操作系统的文件索引和数据库索引

1. B+树的磁盘读写代价更低

{% note success %}
B+树内部结点并没有指向关键字具体信息的指针，因此其内部结点相对B树更小。如果把所有同一内部结点的关键字存在同一块盘快中，那么盘快能容纳的关键字数量也就越多，一次性读入内存中需要查找的关键字也就越多，相对就减少了IO读写次数也就降低了

{% endnote %}

2. B+树的查询效率更加稳定

{% note primary %}

由于非终结点的并不最终指向文件内容的结点，而只是叶子结点中关键字的索引，索引任何关键字的查找必须走一条从根节点到叶子结点的路，所有关键字查询的路径长度相同，导致每一个数据的查询效率相当

{% endnote %}

## Mysql联合索引

### 联合索引是两个或多个列以上的索引

**对于联合索引：**Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部分，但只能是最左侧部分。

例如索引是key index(a,b,c),可以支持 a，ab，abc，3种组合进行查找，**但不支持bc进行查找**，当最左侧字段是常量引用时，索引就十分有效。

### 利用索引中的附加列

`可以缩小搜索范围，但使用一个具有两列的索引不同于使用两个单独的索引。`

**复合索引：** 结构与电话簿类似，人名有姓和名构成，电话簿首先按姓氏进行排序，然后按名字对相同姓氏的人进行排序。

如果您知道姓，电话簿将非常有用；如果您知道姓和名则更有用，但如果只知道名不知道姓，电话簿将没有用处。

## 什么情况下应不建或者少建索引

1. 表记录太少
2. 经常插入，删除，修改的表
3. 数据重复且分布平均的表字段，假如一个表有10万行记录，有一个字段A只有T和F两种值，且每个值的分布概率大约为50%，那么这种表A字段建索引一般不会提高数据库查询速度
4. 经常和主键一块查询但主字段索引值比较多的表字段

## 什么是表分区

是根据一定规则，将数据库中的表分别分解成多个更小的，容易管理的部分，从逻辑上看，只有一张表，但底层却是多个物理分区组成

## 表分区与分表的区别

<p class="note note-primary">表分区：如上👆，比如用户订单记录根据时间分成多个表</p>

<p class="note note-danger">分表：根据业务逻辑拆分表，使之在底层和逻辑层都是多个物理分区</p>

**区别：分区从逻辑上来讲只有一张表，而分表则将一张表分解成多个表**

## 分区表的好处

1. **存储更多数据**，分区表的数据可以分布在不同的物理设备上，从而高效的利用多个硬件设备，和单磁盘或者文件系统相比，可以存储更多数据
2. **优化查询**，在where语句中包含分区条件时候，可以只扫描一个或者多个分区表来提高查询效率，涉及sum和count语句时，也可以在多个分区上并行处理，最后汇总结果。
3. **分区表更容易维护**，例如：想批量删除大量数据，可以清除整个分区
4. **避免某些特殊的瓶颈**，例如InnoDB的单个索引的互斥访问，发生的锁竞争

## 分区表的限制因素

1. 一个表最多只能由1024个分区
2. Mysql5.1中，分区表达式必须是整数，或者返回整数的形式，Mysql5.5中提供了非整数表达式分区的支持
3. 如果分区字段中有主键或者唯一索引列，那么多有的主键和唯一索引列都必须包含进来。即：`分区字段要么不包含主键或者索引列，要么包含全部主键和索引列`
4. 分区表中无外键约束
5. Mysql的分区适用于一个表的所有数据和索引，不能只对表数据分区而不对索引分区，也不能只对索引分区而不对表分区，也不能只对表的一部分数据分区

## 如何判断当前Mysql是否支持分区

`命令：show variables like '%partition%'`

`have_partintioning 的值为YES，表示支持分区。`

## Mysql 支持的分区类型有哪些？

1. `RANGE分区`：这种模式允许将数据划分不同范围，例如可以将一个表通过年份划分成若干分区
2. `LIST分区`:  这种模式允许通过预定义的列表的值来对数据进行分割。按照List中的值分区，与RANGE的区别是，range分区的区间范围值是连续的。
3. `HASH分区`： 中模式允许通过对表的一个或多个列的Hash Key进行计算，最后通过这个Hash码不同数值对应的数据区域进行分区。例如可以建立一个对表主键进行分区的表。
4. `KEY分区`：上面Hash模式的一种延伸，这里的Hash Key是MySQL系统产生的。

## 四种隔离级别

1. Serializable（串行）可避免脏读，不可重复度，幻读
2. Repeatable read（可重复度）可避免脏读，不可重复度
3. Read committed（读已提交）可避免脏读
4. Read uncommitted（读未提交）最低级别，任何情况都无法保证

## MVVC

{% note danger %}
Mysql InnoDB 存储引擎，实现的是基于多版本的并发控制协议—MVVC（Multi-Version Concurrency Controller）

**注**：与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control

**MVCC最大的好处**： 读不加锁，读写不冲突，在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能，现阶段几乎所有的RDBMS，都支持MVCC

{% endnote %}

1. LBCC Lock-Base Concurrency Control，基于锁的并发控制
2. MVCC：Multi-Version Concurrency Control，基于多版本的并发控制协议，纯粹基于锁的并发机制并发量低，MVCC是在基于锁的并发控制上的改进，主要是在读写操作上提高了并发

## MVCC并发控制中，读操作可分为两类

1. 读快照：读取的是记录的可见版本（有可能是历史版本），不用加锁（共享读锁也不加，所以不会阻塞其他事务的写）
2. 当前读：读取的记录是最新版本，并且，当前返回的记录都会加上锁，保证其他事务不会再并发修改

## 行级别锁定的优点

1. 在许多线程访问不同的行时只存在少量的锁定冲突
2. 回滚时只有少量的更改
3. 可以长时间锁定单一的行

## 行级别锁定的缺点

1. 比页级别锁占用更多的内存
2. 在当表的大部分使用时候，比页级别锁定更慢，因为你必须获取更多的锁
3. 如果你在大部分数据经常进行GROUP BY操作，或者必须经常扫描整个表，比其他锁定明显慢很多
4. 用高级别锁定，通过支持不同类型的锁定，你可以很容易的调节应用程序，因为其锁成本小于行级锁定

## Mysql优化

1. 开启查询缓存，优化查询

2. explain你的select查询，这可以帮你分析你的查询语句或是表结构的性能瓶颈。EXPLAIN 的查询结果还会告诉你你的索引主键被如何利用的，你的数据表是如何被搜索和排序的

3. 当只要一行数据时使用limit 1，MySQL数据库引擎会在找到一条数据后停止搜索，而不是继续往后查少下一条符合记录的数据

4. 为搜索字段建索引

5. 使用 ENUM 而不是 VARCHAR。如果你有一个字段，比如“性别”，“国家”，“民族”，“状态”或“部门”，你知道这些字段的取值是有限而且固定的，那么，你应该使用 ENUM 而不是VARCHAR

6. Prepared StatementsPrepared Statements很像存储过程，是一种运行在后台的SQL语句集合，我们可以从使用 prepared statements 获得很多好处，无论是性能问题还是安全问题。

   Prepared Statements 可以检查一些你绑定好的变量，这样可以保护你的程序不会受到“SQL注入式”攻击

7. 垂直分表

8. 选择正确的存储引擎

## key 和 index的区别

###### key

数据库的物理结构，有两层意义和作用，一是约束（偏重于约束和规范数据库的结构完整性），二是索引（辅助查询使用），包括primary key，unique，foreign key

###### index

是数据库的物理结构，只是辅助查询，创建时会在另外的表空间（mysql表和innodb表空间）以一个类似目录的结构存储，索引要分类的话，分为前缀索引，全文索引等；

## Mysql中InnoDB和MyISam 的区别

1. InnoDB支持事务，MyIsam不支持

`对于InnoDB每一条sql语言都默认组装成一个事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit中，组成一个事务`

2. InnoDB支持外键，MyIsam 不支持，对于一个包含外键的InnoDB转为MyIsam会失败

3. InnoDB是聚集索引，数据文件和索引绑在一起，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询主键，然后再通过主键查询数据，因此主键不应该过大，因为主键太大，其他索引也会很大

   MyIsam是非聚簇索引，数据文件是分离的，索引保存的是数据文件的指针，主键索引和辅助索引是独立的。

4. InnoDB不保存表的具体行树，执行select (*) from table 会扫描全表，而MyIsam用一个变量保存了整个表的行树，执行上面的语句只需要读出变量即可。

5. InnoDB不支持全文索引，而myIsam支持，效率更高。

### 如何选择

1. 是否需要事务，需要则选择InnoDB，不需要可以考虑MyIsam
2. 绝大多数只是读取可以考虑MyIsam， 读写频繁用InnoDB
3. 系统崩溃后，myIsam 恢复更困难
4. MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的，如果你不知道用什么，那就用InnoDB，至少不会差。

## **数据库表创建注意事项**

**1、字段名及字段配制合理性**

- 剔除关系不密切的字段；
- 字段命名要有规则及相对应的含义（不要一部分英文，一部分拼音，还有类似a.b.c这样不明含义的字段）；
- 字段命名尽量不要使用缩写（大多数缩写都不能明确字段含义）；
- 字段不要大小写混用（想要具有可读性，多个英文单词可使用下划线形式连接）；
- 字段名不要使用保留字或者关键字；
- 保持字段名和类型的一致性；
- 慎重选择数字类型；
- 给文本字段留足余量；

**2、系统特殊字段处理及建成后建议**

- 添加删除标记（例如操作人、删除时间）；
- 建立版本机制；

**3、表结构合理性配置**

- 多型字段的处理，就是表中是否存在字段能够分解成更小独立的几部分（例如：人可以分为男人和女人）；
- 多值字段的处理，可以将表分为三张表，这样使得检索和排序更加有调理，且保证数据的完整性！

**4、其它建议**

- 对于大数据字段，独立表进行存储，以便影响性能（例如：简介字段）；
- 使用varchar类型代替char，因为varchar会动态分配长度，char指定长度是固定的；
- 给表创建主键，对于没有主键的表，在查询和索引定义上有一定的影响；
- 避免表字段运行为null，建议设置默认值（例如：int类型设置默认值为0）在索引查询上，效率立显；
- 建立索引，最好建立在唯一和非空的字段上，建立太多的索引对后期插入、更新都存在一定的影响（考虑实际情况来创建）；