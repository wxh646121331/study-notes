# 1 MySQL的架构介绍

## 1.1 MySQL简介

- mysql内核
- sql优化工程师
- mysql服务器的优化
- 各种参数常量设定
- 查询语句优化
- 主从复制
- 软硬件升级
- 容灾备份
- sql编程

## 1.2 MySql安装

- 下载地址：

- 查看安装时创建的mysql用户和mysql组

  > cat /etc/passwd | grep mysql
  >
  > cat /etc/group | grep mysql
  >
  > mysqladmin --version

- 启、停mysql服务端

  > service mysql start
  >
  > service mysql stop
  >
  > service mysql restart


## 1.3 MySql配置文件

> cp /usr/share/mysql/my-default.cnf /etc/my.cnf

## 1.4 MySQL逻辑架构介绍

## 1.5 MySQL存储引擎

# 2. 索引优化分析

## 2.1 性能下降SQL慢

## 2.2 常见通用的join查询

## 2.3 索引简介

## 2.4 性能分析

## 2.5 索引优化

# 3. 查询截取分析

## 3.1 查询优化

## 3.2 慢查询日志

## 3.3 批量数据脚本

## 3.4 Show Profile

## 3.5 全局查询日志

# 4. MySQL锁机制

## 4.1 表锁

## 4.2 行锁

## 4.3 页锁

# 5. 主从复制

## 5.1 复制的基本原则

## 5.2 复制的最大问题

## 5.3 一主一从常见配置



# 极客时间MySQL实战

# 1 一条sql查询语句是如何执行的

## 1.1 MySQL的逻辑架构

![](./images/MySQL/MYSQL逻辑架构图.png)

- MySQL可以大体分为server层和引擎层两部分
  - server层：server层包括连接器、查询缓存、分析器、优化器、执行器等，涵盖MySQL的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如：存储过程、触发器、视图等。
  - 引擎层：存储引擎层负责数据的存储和提取。其架构模式是插件式的，支持InnoDB、MyISAM、Memory等多个存储引擎。InnoDB从MySQL 5.5.5版本开始成为了默认存储引擎。

## 1.2 连接器

- 连接器负责跟客户端建立连接、获取权限、维持和管理连接。连接命令：

  ```mysql
  mysql -h$ip -P$port -u$root -p
  ```

- 查看连接状态：

  ```mysql
  show processlist;
  ```

- 客户端如果太长时间没动静，连接器会自动将它断开。这个时间是由wait_timout控制的，默认值是8小时

- 长连接与短连接：

  - 长连接是指连接成功后，如果客户端持续有请求，则一直使用同一个连接
  - 短连接则是每次执行完很少的几次查询就断开连接，下次查询再重新建立一个
  - 建立连接的过程比较复杂，建议尽量使用长连接
  - 全部使用长连接后，MySQL占用内存可能涨得特别快，这是因为MySQL在执行过程中临时使用的内存是管理在连接对象里面的，这些资源会在连接断开的时候才释放。
    - 解决方案：
      1. 定期断开长连接
      2. MySQL 5.7或更新版本，可以在每次执行一个比较大的操作后，通过执行mysql_reset_connection来重新初始化连接资源。这个过程不需要重新连接和重新做权限验证，但是会将连接恢复到刚刚创建完时的状态。

## 1.3 查询缓存

- 执行过的查询语句及结果可以会以key-value的形式，被直接缓存在内存中。key是查询的语句，value是查询的结果。

- 查询缓存的失效非常频繁，只要有对一个表的更新，这个表上的所有的查询缓存都会被清空。所有更新压力大的数据库，查询缓存的命中率会非常低。而表态表则适合使用查询缓存

- “按需使用”查询缓存，可以将query_cache_type设置成DEMAND，这样对于默认的sql语句都不使用查询缓存，而对于你确定要使用缓存的语句，可以用SQL_CACEH显示指定

  ```mysql
  select SQL_CACHE * from T where ID=10;
  ```

- 注意：MySQL 8.0版本直接将查询缓存的整块功能删掉了

## 1.4 分析器

- 词法分析：SQL语句由多个字符串和空格组成，MySQL需要识别出里面的字符串分别是什么，代表什么
  - MySQL从“select”这个关键字识别出来，这是一个查询语句
  - 把字符串“T”识别成“表象T”
  - 把字符串“ID”识别成“列ID”
- 语法分析：根据词法分析的结果，语法分析器会根据语法规则，判断你输入的这个SQL语句是否满足MySQL语法

## 1.5 优化器

- 在表里有多个索引的时候，决定使用哪个索引；或者在一个语句有多表关键（join）的时候，决定各个表的连接顺序

## 1.6 执行器

- 开始执行的时候，要先判断一下是否有对表T的查询权限
- 如果有权限，就打开表继续执行，打开表的时候，执行器会根据表的引擎定义，去使用引擎提供的接口。如果：表T中，ID字段没有索引，执行器的执行流程是：
  - 调用InnoDB引擎接口取这个表的和一行，判断ID值是不是10，如果不是则跳过，如果是则将这行存在结果集中； 
  - 调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行； 
  - 执行器将上述遍历过程中所有满足条件的行组成的记录集任务结果集返回给客户端。
- 数据库的慢查询日志中有一个rows_examined字段，表示这个语句执行过程中扫描了多少行，这个值就是在执行器每次调用引擎获取数据行的时候累加的
- 有些场景下，执行器调用一次，在引擎内部则扫描了多行，因此引擎扫描行跟跟rows_examined并不是完全相同的

# 2 日志系统：一条SQL更新语句是如何执行的？

~~~mysql
create table T(ID int primary key, c int);
insert into T values (1, 1);
insert into T values (2, 1);
update T set c=c+1 where ID=2;
~~~

## 2.1 redo log

- 问题：在MySQL里，如果每一次的更新操作都需要写进磁盘，然后磁盘也要找到对应的那条记录，然后再更新，整个过程IO成本、查找成本都很高

- 解决方案：WAL（Write-Ahead Logging）技术，它的关键点就是先写日志，再写磁盘

- 具体细节：当有一条记录需要更新的时候，InnoDB引擎就会先把记录写到redo log里面，并更新内存，这个时候更新就算完成了。同时，InnoDB引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做的

- InnoDB的redo log是固定大小的，比如可以配置成一组4个文件，每个文件的大小是1GB，总共可以记录4GB的操作，从头开始写，写到末尾就又回到开头循环写

  ![](./images/MySQL/redo log写日志.png)

  - write pos：当前记录的位置，一边写一边后移，写到3号文件末尾后就回到0号文件开头
  - check point：当前要擦除的位置，也是往后推移并且循环的，擦除记录之前要把记录更新到数据文件
  - write pos和check point之间是还空着的部分，可以用来记录新的操作，如果write pos追上check point，表示redo log满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把check point推进一下
  - crash-safe：有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为crash-safe

## 2.2 binlog

- binlog是server层的日志系统，binlog只能用来归档
- redo log与binlog的区别
  1. redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用
  2. redo log 是物理日志，记录的是“在某个数据页上做了什么修改“；binlog是逻辑日志，记录的是这个语句的原始逻辑，比如”给 ID=2 这一行的c字段加1“
  3. redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的，”追加写“是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志

## 2.3 更新语句执行流程图

![](./images/MySQL/更新语句执行流程图.png)

## 2.4 两阶段提交

- 如果不使用两阶段提交，在恢复临时库场景或者扩容的场景，可能会出现数据不一致的情况

## 2.5 思考题

- 定期全量备份的周期”取决于系统重要性，有的是一天一备，有的是一周一备“。那么在什么场景下，一天一备会比一周一备更有优势？它影响了这个数据库系统的哪个指标？
  - 好处是”一天一备“比”一周一备“的”最长恢复时间“更短，”一天一备“模式里，最坏情况只需要应用一天的binlog，而”一周一备“模式里，最坏情况下可能要应用一周的binlog
  - 系统的对应指标是RTO（恢复目标时间）
  - 当然，频繁备份是有成本的，因为更频繁全量备份需要消耗更多存储空间，所以这个RTO是成本换来的，这就需要根据业务重要性来评估了

# 3 事务隔离：为什么你改了我还看不见？

## 3.1 隔离性与隔离级别

- 多个事务同步执行时，可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，为了解决这些总是，就有了”隔离级别“的概念
- SQL事务隔离级别：
  - 读未提交：一个事务还没提交时，它做的变更就能被别的事务看到
  - 读提交：一个事务提交之后，它做的变更才会被其他事务看到
  - 可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的
  - 串行化：”写“会加”写锁“，”读“会加”读锁“，当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行

## 3.2 事务隔离的实现

- 在可重复读隔离级别下，每条记录在更新的时候都会同时记录一条回滚操作。记录上的最新值，通过回滚操作，都可以得到前一个状态的值
- 多版本并发控制（MVCC）：同一条记录在系统中可以存在多个版本
- 当系统里没有比这个回滚日志更早的read-view的时候，回滚日志就会被删除
- 尽量不要使用长事务，其中一个原因是因为：长事务意味着系统里面会存在很老的事务视图，由于这些事务随时可能访问数据库里的任何数据，所以这个事务提交之前，数据库里面它可能用到的回滚记录都必须保留，这就会导致大量占用存储空间

## 3.3 事务的启动方式

- 显式启动事务语句，begin 或 start transaction。配套的提交语句是 commit，回滚语句是 rollback

- set autocommit=0，这个命令会将这个线程的自动提交关掉，意味着如果你只执行select语句，这个事务就启动了，而且不会自动提交，这个事务持续存在直到你主动执行commit或rollback语句，或者断开连接

- 提交事务并自动启动下一个事务，commit work and chain

- 查询长事务：在information_schema库的innodb-trx这个表中，如查找持续时间超过60s的事务：

  ```mysql
  select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60;
  ```

## 3.4 思考题

- 系统里应该避免长事务，如果你是业务开发负责人同时也是数据库负责人，你会有什么方案来避免出现或者处理这种情况呢？

# 4 深入浅出索引（上）

## 4.1 索引的常见模型

- 哈希表
  - 适用于只有等值查询的场景，比如Memcached及其它一些NoSQL引擎
  - 因为哈希表不是有序的，所以哈希索引做区间查询的速度是很慢的
- 有序数组
  - 在等值查询和范围查询场景中的性能都非常优秀
  - 但是有序数据在插入一个新记录的时候就必须得挪动后面所有的记录，成本太高
  - 有序数组索引只适用于表态存储引擎
- 二叉搜索树
  - 二叉搜索树的读写效率都比较高，但是由于索引还要写在磁盘上，为了让一个查询尽量少地读磁盘，数据库索引一般会采用N叉树
- 跳表、LSM树等

## 4.2 InnoDB的索引模型

- 在InnoDB中，表都是根据主键顺序以索引的形式存放的，这种存储方式的表称为索引组织表
- InnoDB使用了B+树索引模型，数据都是存储在B+树中的，每一个索引在InnoDB里面对应一棵B+树
- 索引类型：
  - 主键索引也被称为聚簇索引（clustered index）
    - 主键索引的叶子节点存的是整行数据
  - 非主键索引也被称为二级索引（secondary index）
    - 非主键索引的叶子节点存的是主键的值
  - 基于主键索引和普通索引的查询区别
    - 主键查询方式，只需要搜索主键B+树
    - 普通索引查询方案，则需要先搜索普通索引树，得到主键的值，再到主键索引树上搜索一次，这个过程称为回表
    - 基于非主键索引的查询需要多扫描一棵索引对，因此，我们应该尽量使用主键查询

## 4.3 索引维护

- 页分裂：如果新插入的记录所在的数据页已经满了，根据B+树的算法，这时候需要申请一个新的数据页，然后挪动部分数据过去，这个过程称为页分裂
  - 页分裂会影响性能，同时还会影响数据页利用率
- 页合并：当相邻两个数据页由于删除了数据，利用率很低之后，会将数据页做合并。合并的过程，可以认为是分裂过程的逆过程
- 自增主键：是指自增列上定义的主键，在建表语句中一般是这样定义的：NOT NULL PRIMARY KEY AUTO_INCREMENT
  - 性能方面，自增主键的插入数据模式，正符合的递增插入的场景，每次插入一条新记录，都是追回操作，都不涉及到挪动其他记录，也不会触发叶子节点的分裂
  - 从存储空间方面考量，自动主键为整型，占用4个字节或8个字节（bigint），主键长度越小，普通索引的叶子节点就越小，普通索引占用的空间也就越小
- 适合用业务字段直接做主键的场景（KV场景）
  - 只有一个索引
  - 该索引必须是唯一索引

## 4.4 思考题

- 对于以下InnoDB表T，

  ~~~mysql
  create table T(
    id int primary key, 
    k int not null, 
    name varchar(16), 
    index(k))engine=InnoDB;
  ~~~

  如果你要重建索引k，你的SQL语句可以这么写：

  ~~~mysql
  alter table T drop index k;
  alter table T add index(k);
  ~~~

  如果你要重建主键索引，也可以这么写：

  ~~~mysql
  alter table T drop primary key;
  alter table T add primary key(id);
  ~~~

  对于上面这两个重建索引的做法，如果有不合适的，为什么？更好的方法是什么？

  - 答案：重建索引k的做法是合理的，可以达到省空间的目的。但是，重建主键的过程不合理。不论是删除主键还是创建主键，都会将整个表重建。所以连着执行这两个语句的话，第一个语句就白做了。这两个语句，可以用这个语句代替：alter table T engine=InnoDB。

# 5 深入浅出索引（下）

## 5.1 覆盖索引

- 建表语句：

  ~~~mysql
  mysql> create table T5 (
  id int primary key,
  k int NOT NULL DEFAULT 0, 
  s varchar(16) NOT NULL DEFAULT '',
  index k(k))
  engine=InnoDB;
  
  insert into T5 values(100,1, 'aa'),(200,2,'bb'),(300,3,'cc'),(500,5,'ee'),(600,6,'ff'),(700,7,'gg');
  
  ~~~

- 如下查询语句

  ~~~mysql
  select * from T5 where k between 3 and 5;
  ~~~

  的执行流程为：

  1. 在k索引树上找到k=3的记录，取得id=300；
  2. 再到id索引树想到id=300对应的记录R3； 
  3. 在k索引树查到下一个值k=5，取得id=500
  4. 再回到id索引对查到id=500对应 的记录； 
  5. 在k索引树取下一个值k=6，不满足条件，循环结束。

  在这个过程中，读了k索引树的3条记录，回表了2次

- 覆盖索引

  - 如果执行的语句是

    ~~~mysql
    select id from T5 where k between 3 and 5;
    ~~~

    则不需要回表，也就是说，如果索引里覆盖了查询需求，则称为覆盖索引

  - 由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段。

  - 注意：在引擎内部使用覆盖索引在索引k上其实读了3个记录，但是对于MySQL的Server层来说，它就是找引擎拿到了两条记录，因此MySQL认为扫描行数是2。

## 5.2 最左前缀原则

- B+树这种索引结构，可以利用索引的“最左前缀”，来定位记录
- 建立联合索引的原则：
  - 如果通过调整顺序，可以少维护一个索引，那么这个顺序往往就是需要优化考虑采用的
  - 如果不得不维护两个索引，建议创建一个占用空间比较的的单字段索引

## 5.3 索引下推

- MySQL 5.6 引入索引下推优化（index condition pushdown），可以在索引遍历过程中对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数

## 5.4 思考题

- 有这么一个表，表结构定义是这样的：

  ~~~mysql
  create table geek(a int(11) not null,
                   b int(11) not null,
                   c int(11) not null,
                   d int(11) not null,
                   primary key (a, b),
                   key c(c),
                   key ca(c, a),
                   key cb(c, b)
                   )engine=InnoDB;
  ~~~

  由于历史原因，这个表需要a、b做联合主键，业务里面还有这样的两个语句：

  ~~~mysql
  select * from geek where c=N order by a limit 1;
  select * from geek where c=N order by b limit 1;
  ~~~

  为了这两个查询，ca，cb这两个查询是否都是必要的？

- 答案：cb是必要的，ca不需要，ca索引与c索引记录的数据是一样的

# 6 全局锁和表锁：给表加个字段怎么有这么我阻碍？

## 6.1 全局锁

- 全局锁就表对整个数据库实例加锁。加全局锁的命令是

  ~~~mysql
  flush tables with read lock;
  ~~~

  - 全局锁的典型使用场景是，做全库逻辑备份
  - 对整库做备份，FTWRL能确保不会有其他线程对数据库做更新，数据库会完全处理只读状态，存在的风险有：
    - 如果在评为上备份，那么在备份期间都不能执行更新，业务基本上就得停摆
    - 如果在从库上备份，那么备份期间从库不能执行主库同步过来的binlog，会导致主从延迟
  - 全库更新的另一种方法是使用官方自带的逻辑备份工具mysqldump，使用参数-single-transaction，开启可重复读事务
    - single-transaction方法只适用于所有的表使用事务引擎的库
  - 全库只读，为什么不使用set global readonly=true的方式呢？
    - 有些系统中，readonly的值会被用来做其他逻辑，比如用来判断一个库是主库还是备库
    - 在异常处理机制上，如果执行FTWRL传令之后由于客户端发生异常断开，那么MySQL会自动释放这个全局锁，整个库回到可以正常更新的状态。而将整个库设置成readonly后，如果客户端发生异常，则数据库会一直保持readonly状态，这样会导致整个库上时间处于不可写状态，风险较高

- 业务的更新不只是增删改数据（DML），还有可能是加字段等修改表结构的操作（DDL），不论是哪种方法，一个库被全局锁上以后，你要对里面任务一个表做加字段操作，都是会被锁住的

## 6.2 表级锁

- MySQL里表级别锁有两种：

  - 表锁
  - 元数据锁（meta data lock, MDL）

- 表锁：

  - 加表锁的语法：lock tables … read/write
  - 释放锁：unlock tables主动释放或在客户端断开的时候自动释放
  - lock tables语法除了会限制别的线程的读写外，也限定了本线程接下来的操作对象
    - 例如：如果线程A中执行了lock tables t1 read, t2 write; 这个语句，则其他线程写t1、读写t2的语句都会被阻塞。同时线程A在执行unlock tables之前，也只能执行读t1、读写t2的操作，不能访问其他表

- 元数据锁（metadata lock, MDL）

  - MDL不需要显示式使用，在访问一个表的时候会被自动加上。

  - MDL的作用是，保证读写的正确性。

  - MySQL 5.5 版本中引入了MDL，当对一个表做增删改查操作的时候，加MDL读锁；当要对表做结构变更操作的时候，加MDL写锁

    - 读锁之间不互斥，可以有多个线程同时对一张表增删改查
    - 读写锁之间、写锁之间是互斥的，用来保证变更表结构操作的安全性。因此，如果有两个线程要同时给一个表加字段，其中一个要等另一个执行完才能开始执行。

  - 如何安全地给小表加字段？

    - 首先要解决长事务，事务不提交，就会一直占着MDL锁。在MySQL的information_schema库的innodb_tx表中，可以查到当前执行中的事务。如果要做DDL变更的表刚好有长事务在执行，要考虑先暂停DDL，或者kill掉这个长事务

    - 如果要变更的表是一个热点表，比较理想的机制是，在alter table语句里面设定等待时间，如果在这个指定的等待时间里面能够拿到MDL写锁最好，拿不到也不要阻塞后面的业务语句，先放弃，之后再通过重试命令重复这个过程

      ~~~mysql
      alter table tb_name nowait add column ...
      alter table tb_name wait n add column ...
      ~~~

## 6.3 思考题

- 备份一般都会在备库上执行，在用-single-transaction方法做逻辑备份的过程中，如果主库上的一个小表做了一个DDL，比如给一个表上加了一列。这时候，从备库上会看到什么现象呢？

- 答案：

  ~~~
  Q1:SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  Q2:START TRANSACTION  WITH CONSISTENT SNAPSHOT；
  /* other tables */
  Q3:SAVEPOINT sp;
  /* 时刻 1 */
  Q4:show create table `t1`;
  /* 时刻 2 */
  Q5:SELECT * FROM `t1`;
  /* 时刻 3 */
  Q6:ROLLBACK TO SAVEPOINT sp;
  /* 时刻 4 */
  /* other tables */
  ~~~

  

# 7 行锁功过：怎么减少行锁对性能的影响？

- 行锁是在引擎层由各个引擎自己实现的。并不是所有的引擎都支持行锁，MyISAM引擎就不支持行锁

## 7.1 两阶段锁

- 两阶段锁协议：在InnoDB事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放
- 如果事务中需要锁多个行，要把最可能造成锁冲突、最可以影响并发度的锁尽量往后放

## 7.2 死锁和死锁检测

- 死锁：当并发系统中不同线程出现循环资源依赖，涉及的线程都在等待别的线程释放资源时，就会导致这几个线程都进入无限等待的状态
- 出现死锁后，有两种策略：
  - 直接进入等待，直到超时。这个超时时间可以通过参数innodb_lock_wait_timeout来设置
  - 发起死锁检测，发现死锁后，主动回滚链条中的某一个事务，让其他事务得以继续执行。将参数innodb_deadlock_detect设置成on，表示开启死锁检测。死锁检测会耗费大量的CUP

## 7.3 思考题

- 如果要删除一个表里面的前10000行数据，有以下3种方法可以做到：
  - 直接执行 delete from T limit 10000;
  - 在一个连接中循环执行20次delete from T limit 500;
  - 在20个连接中同时执行delete from T limit 500
- 答案：第一种方法一次性更新10000条数据，会产生大事务，根据两阶段锁协议，事务提交的时候才会释放锁，导致数据会被长时间锁住。第三种方法可能出现资源抢占，导致死锁。所以选择第二种方法

  

# 8 事务到底是隔离的还是不隔离的

## 8.1 事务的启动时机

- begin/start transaction命令并不是一个事务的起点，在执行到它们之后的第一个InnoDB表的语句，事务才真正启动。
- 如果要马上启动一个事务，可以使用start transaction with consistent snapshot命令

## 8.2 两个“视图”概念

- view，是一个用查询语句定义的虚拟表，在调用的时候执行查询语句并生成结果，创建视图的语法是：create view…，它的查询方法与表一样
- InnoDB在实现MVCC时用到的一致性视图，即consistent read view，用于支持RC（Read Committed，读提交）和RR（Repeatable Read，可重复读）隔离级别的实现。它没有物理结构，作用是事务执行期间用来定义“我能看到什么数据”。

## 8.3 “快照”在MVCC里是怎么工作的？

- 在可重复读隔离级别下，事务在启动的时候就“拍了个快照”，这个快照是基于整库的

- InnoDB里面每个事务有一个唯一的事务ID，叫作transaction id。它是在事务开始的时候向InnoDB的事务系统申请的，是按申请顺序严格递增的

- InnoDB的行数据有多个版本，每个数据版本有自己的row trx_id

- InnoDB为每个事务构造了一个数组，用来保存这个事务启动瞬间，当前正在“活跃”的所有事务ID。当前系统里面已经创建过的事务ID的最大值加1记为高水位，视图数组和高水位就组成了当前事务的一致性视图（read-view）

- 对于一个事务视图来说，除了自己的更新总是可见以外，有三种情况：

  - 版本未提交，不可见
  - 版本已提交，但是是在视图创建后提交的，不可见
  - 版本已提交，而且是在视图创建前提交的，可见

- 更新数据都是先读后写的，而这个读，只能读当前的值，称为“当前读”

- 除了update语句外，select语句如果加锁，也是当前读

  ~~~mysql
  select * from t where id=1 lock in share mode; //读锁(S锁，共享锁)
  select * from t where id=1 for update; //写锁(X锁，排他锁)
  ~~~

- 读提交的逻辑与可重复读的逻辑区别：

  - 在可重复读隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；
  - 在读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图

# 9 普通索引和唯一索引，应该怎么选择？

## 9.1 查询过程

~~~mysql
select id from T where k=5;
~~~

- 对于普通索引来说，查找到满足条件的第一个记录后，需要查找下一个记录，直到碰到第一个不满足k=5条件的记录
- 对于唯一索引来说，由于索引定义了唯一性，查找到第一个满足条件的记录后，就会停止继续检索
- InnoDB的数据是按数据页为单位来读写的，也就是说，当需要读一条记录的时候，并不是将这个记录本身从磁盘读出来，而是以页为单位，将其整体读入内存，在InnoDB里，每个数据页的大小默认是16KB。
- 因此，对于查询过程，普通索引和唯一索引的性能差异微乎其微

## 9.2 更新过程

- change buffer
  
  - 当需要更新一个数据页时，如果这个数据页已经在内存中，则直接更新。如果这个数据页不在内存中，在不影响数据一致性的前提下，InnoDB会将这些更新操作缓存在change buffer中，这样就不需要从磁盘中读入数据页了。当下次查询需要使用这个数据页的时候，将数据页读入内存，然后执行change buffer中与这个页相关的操作
  
- merge
  
  - 将change buffer中的操作应用到原数据页，生成最新结果的过程，称作merge
  - 除了访问这个数据页会触发merge外，系统有后台线程会定期merge。在数据库正常关闭的过程中，也会执行merge操作
  
- 使用 change buffer的优点

  - 如果能够将更新操作先记录在change buffer，减少读磁盘，语句的执行速度会得到明显的提升。而且，数据读入内存是需要占用buffer pool的，所有这种方式还能够避免占用内存，提高内存利用率

- 什么条件下可以使用change buffer

  - 更新操作流程：

    ~~~mermaid
    graph TB
    start[更新操作]-->isInMemory{是否在内存中}
    isInMemory--Y-->isUnique{"唯一索引？"}
    isInMemory--N-->isUnique2{"唯一索引？"}
    isUnique--Y-->validateUnique[校验唯一性]
    isUnique--N-->update[更新内存]
    validateUnique-->update
    update-->e((结束))
    isUnique2--Y-->readData[数据页读入内存]
    readData-->validateUnique
    isUnique2--N-->writeChangeBuf[写change buffer]
    writeChangeBuf-->e
    
    ~~~

  - change buffer适用于写多读少的场景，比如账单、日志类

  - change buffer不适用于写入之后马上会做查询的场景，对于这种场景，change buffer不但不能减少随机访问IO的次数，反而增加了change buffer的维护代码

- change buffer 和 redo log

  - redo log主要节省的是随机写磁盘的IO消耗（转成顺序写）
  - change buffer主要节省的是随机读磁盘的IO消耗

# 10 MySQL为什么有时候会选错索引？

创建表

~~~mysql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `a` (`a`),
  KEY `b` (`b`)
) ENGINE=InnoDB
~~~

存储过程写入数据

~~~mysql
delimiter ;;
create procedure idata()
begin
  declare i int;
  set i=1;
  while(i<=100000)do
    insert into t values(i, i, i);
    set i=i+1;
  end while;
end;;
delimiter ;
call idata();
~~~

## 10.1 优化器逻辑

- 优化器会根据几个因素选择索引
  - 扫描行数
  - 是否使用临时表
  - 是否排序
- 扫描行数的判断
  - 索引区分度：一个索引上不同的值的个数，基数越大，索引的区分度越好
  - MySQL使用采样统计得到索引的基数，因而并不准确。采样统计的时候，InnoDB默认会选择N个数据页，统计这个页面上的不同值，得到一个平均值，然后乘以这个索引的页面数据，就得到了个这索引的基数。当变更的数据行数超过1/M的时候，会自动触发重新做一次索引统计
  - 在MySQL中，有两种存储索引统计的方式，可以通过设置参数innodb_stats_persistent的值来选择
    - on表示统计信息会持久化存储，默认的N是20，M是10
    - off表示统计信息只存储在内存中，默认的N是8，M是16
- 使用analyze table t命令，可以重新统计索引信息



# 11 怎么给字符串加索引？

## 11.1 前缀索引的选择

~~~mysql
CREATE TABLE `user` (
  `id` int(11) NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  `email` varchar(64) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `name` (`name`)
) ENGINE=InnoDB;

alter table user add index index_email(email);
alter table user add index index_email_6(email(6));
~~~

- 使用前缀索引，定义好长度，就可以做到既节省空间，又不用额外增加太多的查询成本

- 给字符串创建前缀索引时，如何确定前缀索引的长度呢？

  ~~~
  select count(distinct(email)) from user;
  mysql> select count(distinct left(email, 3))as l3,
      -> count(distinct left(email, 4)) as l4,
      -> count(distinct left(email, 5)) as l5
      -> from user;
  ~~~

  - 按照预设的可以接受的区分度损失比例，选择合适的索引。

## 11.2 前缀索引对覆盖索引的影响

- 使用前缀索引就用不上覆盖索引对查询性能的优化，这也是选择是否使用前缀索引时需要考虑的一个因素

## 11.3 其他方式

- 倒序存储
- 使用hash字段
- 两种方式的异同点
  - 相同点：都不支持范围查询，只能支持等值查询
  - 区别：
    - 从占用的额外空间来看，倒序存储方式在主键索引上，不会消耗额外的存储空间，而hash字段方法需要增加一个字段。当然，倒序存储方式使用4个字节的前缀长度应该是不够的，如果再长一点，这个消耗跟额外的这个hash字段也差不多抵消了
    - 从CPU消耗方面，倒序方式每次写和读的时候，都需要额外调用一次reverse函数，而hash字段的方式需要额外调用一次crc32()函数。如果只从这两个函数的计算复杂度来看的话，reverse函数额外消耗的CPU资源会更小些
    - 从查询效率上看，使用hash字段方式的查询性能相对更稳定一些，因为crc32算出来的值虽然有冲突的概率，但概率非常小，可以认为每次查询的平均扫描行数接近1。而倒序存储方式毕竟还是用的前缀索引方式，还是会增加扫描行数。

## 11.4 思考题

- 如果你在维护一个学校的学生信息数据库，学生登录名的统一格式是“学号@gmail.com”，而学号的规则 是：15位的数据，其中前三位是所有城市编号、第4到第6位是学校编号、第7位到第10位是入学年份，最后5位是顺序编号。系统登录的时候都需要学生输入登录名和密码，验证正确后才能继续使用系统，就只考虑登录验证这个行为的话，你会怎么设计这个登录名的索引呢？
- 答：使用hash索引，哈希值为第7位到第15位数字

# 12 为什么我的MySQL会“抖”一下？

## 12.1 相关概念

- flush : 将内存里的数据写入到磁盘的过程，称为flush，也称作“刷脏页”
- 脏页：当内存里的数据页与磁盘不一致的时候，称这个数据页为“脏页”，将内存里的数据页写入磁盘后，内存数据页就与磁盘一致了，则数据页称为“干净页”
- InnoDB用缓冲池”buffer pool“管理内存，缓冲池中的内存页有三种形态
  - 没有使用的
  - 已使用的脏页
  - 已使用的干净页

## 12.2 引发fush的原因

- redo log写满，这时系统将不能接收任何的更新操作，所以这种情况是要尽量避免的
- 系统内存不足，当需要新的数据页，而内存不够用时，就需要淘汰掉一些数据页，如果被淘汰的数据页刚好是”脏页“，就需要先将数据flush到磁盘
  - 为什么不直接将数据页淘汰，下次需要使用的时候，从磁盘加载数据到内存，再应用redo log呢？
    - 因为flush一定会写盘，这样就保证的数据页只有两种状态，这样查询效率会更高
      - 内存中有，且为最新的数据，直接返回
      - 内存中没有，磁盘中就是最新的数据，加载到内存后返回
- MySQL认为系统”空闲“的时候
- 系统正常关闭的时候，会把内存里的脏页都flush到磁盘，这样下次启动MySQL的时候，就可以直接从磁盘读数据，这样启动速度会很快

## 12.3 InnoDB刷脏页的控制策略

- 刷脏页虽然是常态，但是如果出现以下两种情况，是会明显影响性能的

  - 一个查询需要淘汰的脏页太多，会导致查询的时间明显变长
  - 日志写满，此时系统写性能迭为0，这种情况是业务不能接受的

- 脏页控制策略及相关参数

  - innodb_io_capacity : 设置磁盘能力，建议设置成磁盘的IOPS

    - 磁盘的IOPS可以使用fio工具来测试

  - innodb_max_dirty_pages_pct：脏页比例上限

  - F1(M)计算规则（M为当前脏页比例）：

    ~~~
    F1(M)
    {
      if M>=innodb_max_dirty_pages_pct then
          return 100;
      return 100*M/innodb_max_dirty_pages_pct;
    }
    ~~~

  - F2(N)（N为write point与check point之间的差值）：N越大，F2(N)越大

  - 引擎控制刷脏页的速度根据以下公式：innodb_io_capacity*(max(F1(M), F2(N)))%

- 查询脏页比例

  ~~~mysql
  select variable_value into @a from global_status where variable_name = 'INNODB_BUFFER_POOL_PAGES_DIRTY';
  select variable_value into @b from global_status where variable_name = 'INNODB_BUFFER_POOL_PAGES_TOTAL';
  select @a/@b;
  ~~~

# 13 为什么表数据删掉一半，表文件大小不变？

## 13.1 参数innodb_file_per_table

- 建议将innodb_file_per_table这个参数设置为ON，因为一个表单独存储更容易管理，而且在不需要这个表的时候，通过drop table命令，系统就会直接删除掉这个文件。而如果放在共享表空间中，即使表删掉了，空间也是不会回收的

## 13.2 数据删除流程

- delete 命令只是把记录的位置，或者数据页标记为”可复用“，但磁盘文件的大小是不会变的

- 重建表可以去掉表的的空洞：转存数据、交换表名、删除旧表

  ~~~mysql
  alter table table_name engine=InnoDB
  ~~~

## 13.3 思考题

- 假设有人碰到了一个”想要收缩表空间，结果适得其反“的情况：

  1. 一个表t文件大小为1TB
  2. 对这个表执行alter table t engine=InnoDB;
  3. 执行完后，空间不仅没变小，还稍微大了一点

  可能是什么原因呢？

- 答案

  1. 表本身已经没有什么空洞了，比如刚刚做过一次重建表
  2. 在DDL期间，如果刚好有外部的DML在执行，这期间可能会引入一些新的空洞
  3. 在重建表的时候，InnoDB不会把整张表占满，每个页急留下了1/16给后续的更新用，也就是说，其实重建表之后不是最紧凑的。

# 14 count(*)这么慢，该怎么办？

## 14.1 count(*)的实现方式

- MyISAM 引擎把一个表的总行数存在磁盘上
- InnoDB 引擎在执行count(*)的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数
  - MySQL优化方式：MySQL优化器会找到最小的索引树来遍历，在保证逻辑正确的前提下，尽量减少扫描的数据量

## 14.2 计数方案

- 用缓存系统保存计数
  - 缺点:
    - 缓存系统可能会丢失更新
    - 从缓存系统中取到的值是逻辑上不精确的（原因是两个不同的存储构成的系统，不支持分布式事务，无法拿到精确一致的视图）
- 用数据库保存计数，可以解决崩溃丢失更新和逻辑上不精确的问题

## 14.3 不同的count的用法

- count(*)、count(主键 id)、和count(1)都表示返回满足条件的结果集的总和
- count(字段)表示返回满足条件的数据行里面，参数”字段“不为NULL的总个数
- count(主键 id)：InnoDB引擎会遍历整张表，把每一行的id值都取出来，返回给server层。server层拿到id后，判断是不可以为空的，就按行累加
- count(1)：InnoDB引擎遍历整张表，但不取值。server层对于返回的每一行，放一个数字”1“进去，判断是不可能为空的，按行累加
- count(字段)：
  - 如果这个字段定义为not null的话，一行行地从记录里面读出这个字段，判断不能为null，按行累加
  - 如果这个字段定义允许为null，那么执行的时候，判断到有可以是null，还要把值取出来再判断一下，不是null才累加
- count(*)：不会把全部字段取出来，而是专门做了优化，不取值。count(\*)肯定不是null，按行累加
- 效率排序：count(字段)<count(主键 id)<count(1)≈count(*)

# 15 日志和索引相关问题

- MySQL崩溃恢复时的判断规则
  - 如果redo log里的事务的完整的，也就是已经有commit 标识，直接提交
  - 如果redo log里只有完整的prepare标识，则判断对应事务的binlog是否存在并完整
    - 如果是，则提交事务
    - 否则，回滚事务
- 如何判断binlog是否完整？
  - MySQL的binlog是有完整格式的
    - statement格式的binlog，最后会有COMMIT
    - row格式的binlog，最后会有XID EVENT
    - 另外，参数binlog_checksum可以用来验证binlog内容的正确性
- redo log和bin log是怎么关联起来的？
  - 它们有一个共同的字段XID，崩溃恢复的时候，会按顺序扫描redo log
    - 如果扫描到既有prepare，也有commit标记的redo log，直接提交
    - 如果扫描到只有prepare但没有commit的redo log，则拿XID去找bin log对应的事务
- 

# 附录

## 常用命令

| 命令                              | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| mysql -h\$ip -P\$port -u\$user -p | 连接数据库                                                   |
| show processlist;                 | 查看连接状态，等价于select * from information_schema.processlist; |
| show databases;                   | 查询所有schema                                               |
| use database_name;                | 切换scheme                                                   |
| show tables;                      | 查询当前数据库下所有的表                                     |
| desc table_name;                  | 查看表结构                                                   |
| show variables like '%tx%';       | 查看参数配置                                                 |
| show full columns from tb_nam;    | 查看表结构详细                                               |
| show create table tb_name;        | 查看建表语句                                                 |
| select * from mysql.proc limit 1; | 存储过程                                                     |
| show procedure status;            | 查看存储过程                                                 |
| show create procedure proc_name;  | 查看存储过程创建代码                                         |
| show create function func_name;   | 查看函数创建代码                                             |
| show table status;                | 查看表状态                                                   |

## 常用参数

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| innodb_flush_log_at_trx_commit | 设置成 1 的时候表示每次事务的 redo log 都直接持久化到磁盘，建议设置成 1，这样可以保证 MySQL 异常重启之后数据不丢失 |
| sync_binlog                    | 设置成1表示每次事务的 binlog 都持久化到磁盘，建议设置成 1，可以保证 MySQL 异常重启之后 binlog 不丢失 |
| innodb_lock_wait_timeout       | 锁等待超时时间                                               |
| innodb_deadlock_detect         | 是否开启主动死锁检测                                         |
| innodb_change_buffer_max_size  | change buffer占用buffer pool的百分比                         |
| slow_query_log                 | 慢查询日志开关                                               |
| slow_query_log_file            | 查查询日志存储路径                                           |
| long_query_time                | 慢查询阈值，默认为10秒                                       |
| innodb_stats_persistent        | 采样统计存储方式                                             |
| innodb_io_capacity             | 磁盘能力                                                     |
| innodb_max_dirty_pages_pct     | 脏页比例上限                                                 |
| innodb_flush_neighbors         | 刷脏页开启”连坐“机制                                         |
| innodb_file_per_table          | 控制表数据的存放，ON表示每个InnoDB表数据存储在一个以.idb为后缀的文件中，OFF表示表数据放在系统共享表空间中，也就是跟数据字典放在一起，从MySQL 5.6.6 版本开始，默认值是ON |
| binlog_checksum                | 验证binlog内容的正确性                                       |

## 常用函数

| 函数                          | 说明           |
| ----------------------------- | -------------- |
| str_to_date(date, '%Y-%m-%d') | 字段串转Date   |
| date_format(date, '%Y-%m-%d') | Date转字符串   |
| timediff(date1, date2)        | 取时间差       |
| time_to_sec(datediff)         | 将时间差转成秒 |

## 时间格式

| 格式 | 说明                              |
| ---- | --------------------------------- |
| %Y   | 4位的年份                         |
| %y   | 2位的年份                         |
| %m   | 月，格式为01…...12                |
| %c   | 月，格式为1……12                   |
| %d   | 月份中的天数，格式为00……31        |
| %e   | 月份中的天数，格式为0……31         |
| %H   | 小时，格式为00……23                |
| %k   | 小时，格式为0……23                 |
| %h   | 小时，格式为01……12                |
| %l   | 小时，格式为1……12                 |
| %i   | 分，格式为00……59                  |
| %r   | 时间，格式为12小时hh:mm:ss [A/P]M |
| %T   | 时间，格式为24小时hh:mm:ss        |
| %S   | 秒，格式为00……59                  |
| %s   | 秒                                |

