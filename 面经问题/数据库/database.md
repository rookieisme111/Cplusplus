# 数据库相关的知识
mysql事务、隔离级别、MVCC的理解：https://zhuanlan.zhihu.com/p/148035779
1. **数据库的索引说一下?索引的概念？索引的数据结构？聚簇索引和非聚簇索引的区别？联合/复合索引的概念，最左前缀匹配原则，覆盖索引的概念？查询优化器的概念？**
> 索引是为了加快表中数据行检索而创建的分散的存储结构，索引是针对表来建立的，索引页面的索引项中包含逻辑指针，指向它所对应的表中数据行。
> 自己的理解：索引是一种概念，索引是影响数据行存储结构，并额外包含位置信息的数据结构，增加索引是为了加快表中数据行的检索速度。

> 索引可以分为聚集索引和非聚集索引（根据数据行的顺序是否和索引项顺序相同），稠密索引和稀疏索引（根据索引项和数据行的对应关系，一对一还是一对多）
> 一个表只能有一个聚簇索引，但可以有多个非聚簇索引。因为聚簇索引决定着表中数据行在物理存储上的顺序，即聚簇索引项决定着数据行的物理存储顺序，一个表在物理设备上只能有一种存储顺序。非聚簇索引，不影响表的物理存储顺序，只在建立了索引的列进行了逻辑上的排序。从数据结构看，聚簇索引的叶子结点是数据结点，非聚簇索引的叶子结点仍然是索引结点。

> 覆盖索引的概念：仅在覆盖索引中进行一次扫描，就能在索引的叶结点中找到所有需要的列信息。省去了回表的过程，是数据库优化的一个方向。

> 复合索引的概念：使用表中多个列建立的索引，使用时遵循最左前缀匹配原则，最左前缀匹配是由于复合索引的B+树结构决定的，假设存在一个复合索引abc,则a,ab,abc都是可以利用复合索引的，因为索引的逻辑顺序第一考虑的是最左侧的顺序，当最左的索引列相同时，再考虑次左的索引列的顺序，以此类推，如果忽略左侧索引，则右侧索引的逻辑顺序没有实际意义。就像利用姓和名的首字母查找人名，首先提供性姓的首字母，快速定位到所有以该字母起始的人名，才能继续寻找符合条件的名，如果只提供了名，则不得不进行全记录的扫描。

> 

> 数据库索引的底层数据结构有hash表和B+树。inodb的索引模型采用的是B+树。
2. **事务的四个特性**
> 1. 原子性A：事务是由一组sql操作组成的逻辑处理单元，这些操作要么全部成功执行，要么一个都不执行。
> 2. 一致性C：事务执行前后的状态要一致，即数据的一致性。其它特性都是为了一致性这个最终的目标。
> 3. 隔离性I：事务与事务之间是隔离的，但是隔离也要分等级，隔离级别越高，越安全，但是却牺牲了并发性能。
> 4. 持久性D：事务的结果在向数据库写入过程中崩溃了，之后也能在数据库中恢复。持久性是一种承诺，保证数据可以长久的在数据库中存储。
3. **事务的隔离级别，以及各级别的所解决的问题**
> 1. 读未提交：最低的隔离级别，相当于没有隔离，存在脏读的问题，即一个事务可以读取到另一个事务没有提交的数据。
> 2. 读已提交：通过排他锁，可以提升隔离级别。可以顺利的解决脏读问题，但不能预防不可重复读的问题，即一次事务两次读取同一个数据，读取到结果不一致，这是由于另一个事务已经更新了这个数据的值。
> 3. 可重复读：顺利解决了不可重复读的问题，采用的是快照（一致性视图）的方法。但会出现幻读的问题，幻读一般是由insert引起的，某一事务第一次查询id=3的数据行，发现没有记录，此时另一个事务插入了符合条件的记录，然后原事务又重复查询了一遍，发现有记录了，就好像出现幻觉一样。
> 4. 序列化：最高级别的隔离，相当于所有的事务都只能串行执行。
4. **请你解释一下数据库的脏读、幻读、不可重复读**
5. **redo/undo日志**
> https://www.jianshu.com/p/57c510f4ec28
> 
> redo/undo日志分别保证了事务的持久性和原子性，目的都是为了数据的一致性。redo日志是物理日志，记录的是数据页的物理修改，undo是逻辑日志，记录每个数据行的修改，前者提供数据恢复功能，即前滚操作，后者提供回滚操作。日志采用了预写日志的方法，在事务提交前需要先完成日志的落盘（写入磁盘），此时即便数据库崩溃，也可以从一个检查点进行前滚恢复，再把缺少上下文的事务进行回滚操作，恢复到还未执行的状态。
6. **一致性视图、快照、MVCC（多版本并发控制）**
> MVCC保存了某一时刻数据的快照，某一时刻不同事务看到相同表的数据是可能是不同的。可以解决不可重复读的问题，实现是采用了一种版本号的机制。记录一行数据的创建时间和销毁时间，查询即符合创建时间早于当前版本号，并且当前未销毁的数据行，插入和删除则会更新数据行的创建和销毁时间，更新则复制该数据行，并把当前系统版本号作为新数据行的创建时间，同时作为旧数据行的销毁时间。
7. **跳表了解吗？跳表用于redis的内存索引技术。**
8. **什么是redis**
9. **redis和数据库双写的一致性问题**
10. **redis的缓存击穿和缓存雪崩的问题**
11. **redis的五个基础的数据类型**
12. **redis的索引模型——跳表**
13. **介绍一下MVCC（多版本并发控制）的机制？读已提交和可重复读的实现区别？用于解决什么问题。**
14. **当前读和快照读**
15. **按锁的性质可以分为：排他锁和共享锁；按锁的粒度可以分为：表锁和行锁；还有间隙锁、乐观锁和悲观锁等**
16. **SQL中慢查询的定位和分析**
17. **SQL的优化方法**
