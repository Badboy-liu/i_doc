# mysql

## mysql有三层    

### navicat 客户端       service 服务端       engine  存储引擎



### 1.客户端是给用户连接mysql的一个软件

##### 根据mysql的连接协议mysql的服务端



### 2.服务端是mysql的服务器，监听3306端口

##### 用于和其他设备交互（本地设备和其他电脑）



### 3.存储引擎是mysql写入和读入字符串的一个服务，

#####  不同的存储引擎，读写方式是不一样的







### sql  

##### select * from 表名  where   having         group by     order by



##### update from 表名  where 

##### delete from 表名 where

##### insert into 表名  (字段)



#### 事务

| 隔离级别         | 解释           | 脏读       | 不可重复读 | 幻读       |
| ---------------- | -------------- | ---------- | ---------- | ---------- |
| READ UNCOMMITTED | 读未提交的事务 | 可能发生   | 可能发生   | 可能发生   |
| READ COMMITTED   | 读已提交的事务 | 不可能发生 | 可能发生   | 可能发生   |
| REPEATABLE READ  | 可重复读       | 不可能发生 | 不可能发生 | 可能发生   |
| SERIALIZABLE     | 可串行化       | 不可能发生 | 不可能发生 | 不可能发生 |



### mysql的日志



##### Redolog记录的是数据库事务操作中产生的变化，记录修改后的值；

##### Undolog记录事务操作前的数据值；用于回滚数据，更新之前的数据，旧版本数据

##### Binlog是MySQL数据库用于记录所有对数据库表的更改（包括数据插入、更新和删除）的二进制日志文件。







### mysql的四个事务隔离级别

##### 1.读未提交(读readview最后一个版本的数据)

##### 2.读已提交(读取事务id小于等于自己本身的readview)

##### 3.可重复读(读取的是最后一次生成的readview)

##### 4.可串行化(一个事务只应许一个事务读写)



### mvcc

##### mysql每条数据上有事务时会有版本链

##### 事务后的数据记录事务前的数据，后产生版本链，用于事务隔离查询

![mvcc](https://github.com/Badboy-liu/i_doc/blob/main/mysql/1693e86e2a3fffa3~tplv-t2oaga2asx-jj-mark_1512_0_0_0_q75.webp)

##### 你当时事务id和事务隔离级别去查询版本链条中你可以读取到哪个

###### 1.读未提交，直接读取mvcc最新事务版本的数据

###### 2.读已提交，读取正在跑的事务id最小的，然后去mvcc找比现在正在跑的事务id还小一个的事务，说明是事务已经提交的最大版本

###### 3.可重复读，是每次数据更改后只生成一次readview视图，多个事务只能读取同一个readview已提交的最大版本事务的数据

###### 4.可串行化，意味着在同一时间只有一个事务可以读写数据



### explain 

#### 查看sql的执行计划，用于查询更新优化

##### explain select * from 表名

##### explain  update from 表名  where 

##### explain  delete from 表名 where

##### explain  insert into 表名  (字段)

##### explain  format=”json“  可以查询更详细的信息



#### 优化链路

##### SHOW VARIABLES LIKE 'optimizer_trace';

##### SET OPTIMIZER_TRACE="enabled=on",END_MARKERS_IN_JSON=on;

##### SET optimizer_trace_offset=-30,optimizer_trace_limit=30;

##### select * from user where role_id = 10

##### SELECT * FROM information_schema.OPTIMIZER_TRACE;



### 索引

#### 就是字典目录，可以让你快速找到数据

##### 每多一个索引就是多了一个目录

##### btree字典目录  多叉树

##### hash字典目录



#### 索引类型

##### 唯一索引 

###### 1.可用于防止数据重复，名称和手机号的检测或者唯一值的检测

###### 2.查询很快

##### 普通索引

###### 1.用于常用字段的查询

##### 全文索引

###### 没用过

##### 主键索引

###### id查询很快，锁主键，唯一



###### SELECT * FROM hero FORCE INDEX(idx_name) WHERE name <= 'c曹操' LOCK IN SHARE MODE;

###### FORCE INDEX(idx_name) 强制使用索引





### 锁

#### mysql 8.0

##### select * from performance_schema.data_lock_waits; 查看等待锁的事务

##### select * from performance_schema.data_locks;          查看正在锁的事务

##### select * from information_schema.innodb_trx;           查看未提交的事务

#### mysql 5.7

##### select * from information_schema.innodb_locks;

##### select * from information_schema.innodb_lock_waits;

##### select * from information_schema.innodb_trx;





#### 查询binlog日志

##### SHOW BINLOG EVENTS   [IN 'log_name']   [FROM pos]   [LIMIT [offset,] row_count]

##### SHOW BINLOG EVENTS;

##### **SHOW** BINLOG EVENTS **IN** 'xiaohaizi-bin.000004';

##### C:\Program Files\MySQL\MySQL Server 8.0\bin\mysqlbinlog.exe DESKTOP-G67PUE5-bin.000026





#### Buffer Pool

##### Buffer Pool是内存数据，也称为缓存（中文名是`缓冲池`）

##### show variables like “%buffer%”

##### mysql把数据从硬盘加载到内存放到Buffer Pool

##### 往后我们查询的都是Buffer Pool的数据而不是硬盘的



#### flush链表

##### 当你执行update\delete\insert的时候其实更新的是内存的数据，然后更新索引，主键和其他索引

##### 当事务完成，会把这条改变的数据防到要刷盘的链表里面



#### lru链表

##### 1.Buffer Pool满了需要淘汰不常用的数据怎么办，此时lru链表解决这个问题

##### 最开始把每次从硬盘查询的到的数据放到链表头部，这样常用的数据都在最前面，可是有问题



##### 2.问题是，如果这一次查询量很多就会把常用的数据挤到最后面并且淘汰掉

##### 所有分了两个阶段，一开始把数据缓存到冷数据区，访问量到一定次数放到热数据区，这样就解决了这个问题

![1693e86e2a3fffa3~tplv-t2oaga2asx-jj-mark_1512_0_0_0_q75](https://raw.githubusercontent.com/Badboy-liu/i_doc/main/mysql\1693e86e2a3fffa3~tplv-t2oaga2asx-jj-mark_1512_0_0_0_q75.webp)



#### 为什么limit 大页数很慢？

##### 因为limit是在server层做的

##### server根据查询语句选择是否使用索引，然后去查询存储引擎，存储引擎根据链表一条一条返回给server

##### 而limit 1000,10   说明server要过滤10000次后选择10条

