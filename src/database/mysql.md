# 事务

## 隔离级别
- 读未提交（Read uncommitted）：一个事务可以读取另一个事务未提交的数据。
- 读已提交（Read committed）：一个事务只能读取另一个事务已经提交的数据。
- 可重复读（Repeatable read）：一个事务在执行过程中，同样的查询语句，每次返回的结果都是一样的。
- 串行化（Serializable）：事务串行执行，事务只能一个
_引发的问题_
  不可重复读的重点是事务内读取同一行数据的结果不一致，而幻读的重点是事务内多次查询得到的**结果集**不一致。

## 事务的实现方式
锁 ，无锁
1. 表锁，行锁
   乐观锁：update A set Name=lisi,version=version+1 where ID=#{id} and version=#{version}
   悲观锁
   Gap
   共享锁（S）：SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE
   排他锁（X）：SELECT * FROM table_name WHERE ... FOR UPDATE

2. MVCC 非锁定读
MVCC 当前读，快照读（加锁）
- 在InnoDB中，MVCC实现主要通过隐藏列（DATA_TRX_ID和DATA_ROLL_PTR）和undo log实现。当事务回滚时，undo log用于将数据恢复到修改前的样子；当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，则可以通过undo log读取之前的版本数据，以此实现非锁定读。
- Repeatable read配合gap锁不会出现幻读
```sql
Select * from emp where empid > 100 for update;
//InnoDB不仅会对符合条件的empid值为101的记录加锁，也会对empid大于101（这些记录并不存在）的“间隙”加锁
```

3. 锁定读

```sql
SELECT … FOR UPDATE
SELECT … LOCK IN SHARE MODE
```
4. AUTO-INC Locking
如果多个线程同时尝试插入记录，那么该计数器就会存在**竞态**，因此需要锁来确保自增过程的原子性，这种锁被称为AUTO-INC Locking 。
MySQL 5.1.22版本引入了一种轻量级互斥量的自增长实现机制，说人话就是采用CAS+自旋替代原有的互斥锁实现。
- 对于可以预先知道插入行数的插入操作而言，都采用CAS+自旋的方式实现，注意: 如果可以知道某次批量插入操作需要插入的行数，那么可以一次性申请n个自增长id ，然后事务自己后面再慢慢进行分配
- 对于无法提前获知插入行数的批量插入操作来说，采用互斥锁的方式实现，因为此过程可能会比较漫长，如果采用CAS+自旋可能会导致长时间的空自旋，浪费CPU资源
   

# 索引
## 数据结构

## 非聚簇索引
非聚簇索引不一定回表查询吗(覆盖索引)
```sql
 SELECT name FROM table WHERE name='guang19';
```
- 覆盖索引即需要查询的字段正好是索引的字段，那么直接根据该索引，就可以查到数据了，而无需回表查询。

# SQL

## explain
### Type
system：如果表使用的引擎对于表行数统计是精确的（如：MyISAM），且表中只有一行记录的情况下，访问方法是 system ，是 const 的一种特例。
const：表中最多只有一行匹配的记录，一次查询就可以找到，常用于使用主键或唯一索引的所有字段作为查询条件。
eq_ref：当连表查询时，前一张表的行在当前这张表中只有一行与之对应。是除了 system 与 const 之外最好的 join 方式，常用于使用主键或唯一索引的所有字段作为连表条件。
ref：使用普通索引作为查询条件，查询结果可能找到多个符合条件的行。
index_merge：当查询条件使用了多个索引时，表示开启了 Index Merge 优化，此时执行计划中的 key 列列出了使用到的索引。range：对索引列进行范围查询，执行计划中的 key 列表示哪个索引被使用了。
index：查询遍历了整棵索引树，与 ALL 类似，只不过扫描的是索引，而索引一般在内存中，速度更快。
ALL：全表扫描
### Key

### Extra（重要）
这列包含了 MySQL 解析查询的额外信息，通过这些信息，可以更准确的理解 MySQL 到底是如何执行查询的。常见的值如下：Using filesort：在排序时使用了外部的索引排序，没有用到表内索引进行排序。
Using temporary：MySQL 需要创建临时表来存储查询的结果，常见于 ORDER BY 和 GROUP BY。
Using index：表明查询使用了覆盖索引，不用回表，查询效率非常高。
Using index condition：表示查询优化器选择使用了索引条件下推这个特性。
Using where：表明查询使用了 WHERE 子句进行条件过滤。一般在没有使用到索引的时候会出现。
Using join buffer (Block Nested Loop)：连

表查询的方式，表示当被驱动表的没有使用索引的时候，MySQL 会先将驱动表读出来放到 join buffer 中，再遍历被驱动表与驱动表进行查询。

## Sql优化
1. 避免使用SELECT*
2. 分页优化
3. 尽量避免多表做join
4. 建议不要使用外键与级联
5. 选择合适的字段类型
6. 尽量用UNION ALL代替UNON
7. 批量操作
8. Show Profle分析SQL执行性能
9. 优化慢SQL
10. ”正确使用索引
```
11. 选择合适的字段创建索引
12. 被频繁更新的字段应该慎重建立…
13. 尽可能的考虑建立联合索引而不…
14. 注意避免冗余索引
15. 考虑在字符串类型的字段上使用…
16. 避免索引失效
17. 别除长期未使用的索引
```
！[sql](https://oss.javaguide.cn/javamianshizhibei/javamianshizhibei-sql-optimization.png)