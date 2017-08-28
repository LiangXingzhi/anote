### 事务隔离级别
一、数据库事务隔离级别
数据库事务的隔离级别有4个，由低到高依次为Read uncommitted 、Read committed 、Repeatable read 、Serializable ，这四个级别可以逐个解决脏读 、不可重复读 、幻读 这几类问题。

√: 可能出现    ×: 不会出现

事务隔离级别 | 脏读 |	不可重复读 |	幻读
---|---|---|---
Read uncommitted |	√ |	√ |	√
Read committed	| ×	| √ |	√
Repeatable read	| ×	| ×	| √
Serializable	| ×	| ×	| ×
 
注意：我们讨论隔离级别的场景，主要是在多个事务并发 的情况下，因此，接下来的讲解都围绕事务并发。


二、脏读、幻读、不可重复读
1.脏读：
脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
一个事务读取了另一个未提交的并行事务写的数据。 

2.不可重复读：
是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。（即不能读到相同的数据内容）
例如，一个编辑人员两次读取同一文档，但在两次读取之间，作者重写了该文档。当编辑人员第二次读取文档时，文档已更改。原始读取不可重复。如果只有在作者全部完成编写后编辑人员才可以读取文档，则可以避免该问题。
一个事务重新读取前面读取过的数据， 发现该数据已经被另一个已提交的事务修改过。 

3.幻读:
是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象
发生了幻觉一样。
例如，一个编辑人员更改作者提交的文档，但当生产部门将其更改内容合并到该文档的主复本时，发现作者已将未编辑的新材料添加到该文档中。如果在编辑人员和生产部门完成对原始文档的处理之前，任何人都不能将新材料添加到文档中，则可以避免该问题。
一个事务重新执行一个查询，返回一套符合查询条件的行， 发现这些行因为其他最近提交的事务而发生了改变。 


封锁（Locking）

封锁是实现并发控制的一个非常重要的技术。所谓封锁就是事务T在对某个数据对象例如表、记录等操作之前，先向系统发出请求，对其加锁。加锁后事务T就对该数据对象有了一定的控制，在事务T释放它的锁之前，其它的事务不能更新此数据对象。

基本的封锁类型有两种：排它锁（Exclusive locks 简记为X锁）和共享锁（Share locks 简记为S锁）。

排它锁又称为写锁。若事务T对数据对象A加上X锁，则只允许T读取和修改A，其它任何事务都不能再对A加任何类型的锁，直到T释放A上的锁。这就保证了其它事务在T释放A上的锁之前不能再读取和修改A。

共享锁又称为读锁。若事务T对数据对象A加上S锁，则其它事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这就保证了其它事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
封锁协议

在运用X锁和S锁这两种基本封锁，对数据对象加锁时，还需要约定一些规则，例如应何时申请X锁或S锁、持锁时间、何时释放等。我们称这些规则为封锁协议（Locking Protocol）。对封锁方式规定不同的规则，就形成了各种不同的封锁协议。下面介绍三级封锁协议。三级封锁协议分别在不同程度上解决了丢失的修改、不可重复读和读"脏"数据等不一致性问题，为并发操作的正确调度提供一定的保证。下面只给出三级封锁协议的定义，不再做过多探讨。
1级封锁协议

1级封锁协议是：事务T在修改数据R之前必须先对其加X锁，直到事务结束才释放。事务结束包括正常结束（COMMIT）和非正常结束（ROLLBACK）。1级封锁协议可防止丢失修改，并保证事务T是可恢复的。在1级封锁协议中，如果仅仅是读数据不对其进行修改，是不需要加锁的，所以它不能保证可重复读和不读"脏"数据。
2级封锁协议

2级封锁协议是：1级封锁协议加上事务T在读取数据R之前必须先对其加S锁，读完后即可释放S锁。2级封锁协议除防止了丢失修改，还可进一步防止读"脏"数据。
3级封锁协议

3级封锁协议是：1级封锁协议加上事务T在读取数据R之前必须先对其加S锁，直到事务结束才释放。3级封锁协议除防止了丢失修改和不读'脏'数据外，还进一步防止了不可重复读。

事务隔离级别

尽管数据库理论对并发一致性问题提供了完善的解决机制，但让程序员自己去控制如何加锁以及加锁、解锁的时机显然是很困难的事情。索性绝大多数数据库以及开发工具都提供了事务隔离级别，让用户以一种更轻松的方式处理并发一致性问题。常见的事务隔离级别包括：ReadUnCommitted、ReadCommitted、RepeatableRead和Serializable四种。不同的隔离级别下对数据库的访问方式以及数据库的返回结果有可能是不同的。我们将通过几个实验深入了解事务隔离级别以及SQL Server在后台是如何将它们转换成锁的。
 Serializable

Serializable隔离级别是最高的事务隔离级别，在此隔离级别下，不会出现读脏数据、不可重复读和幻影读的问题。在详细说明为什么之前首先让我们看看什么是幻影读。

所谓幻影读是指：事务1按一定条件从数据库中读取某些数据记录后，事务2插入了一些符合事务1检索条件的新记录，当事务1再次按相同条件读取数据时，发现多了一些记录。
#### 1.1 准备环境
~~~mysql
mysql -u root -proot;

create database test CHARACTER SET=utf8 COLLATE=utf8_bin;
use test;
# drop table t_account;
create table t_account (id VARCHAR(20) primary key, name varchar(20) not null UNIQUE, balance BIGINT);
insert into t_account(id, name, balance) values('1', 'zhao', 100);
insert into t_account(id, name, balance) values('2', 'qian', 200);
insert into t_account(id, name, balance) values('3', 'sun', 300);
insert into t_account(id, name, balance) values('4', 'li', 400);


### 开2个session，分别设置session_name;
### session 1
set @session_name = 'Session1';
### session 2
set @session_name = 'Session2';
~~~

#### 1.2 查看当前默认事务隔离级别
SET session TRANSACTION ISOLATION LEVEL Serializable;（参数可以为：Read uncommitted，Read committed，Repeatable，Serializable）
~~~mysql
select @@tx_isolation;
+-----------------+
| @@tx_isolation  |
+-----------------+
| REPEATABLE-READ |
+-----------------+
1 row in set (0.00 sec)
~~~

#### 1.3 Read uncommitted VS Read uncommitted
+ 1.3.1 dirty read

时间|session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ UNCOMMITTED; | set session transaction isolation level READ UNCOMMITTED; | 设置隔离级别
T3 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1修改zhao的账户，为他存入1000元
T4 | | select balance from t_account where id = '1'; | balance = 1100|事务2读取事务1没有提交的记录，即产生脏读
T5 | rollback; |  | 事务1回滚
T6 | | select balance from t_account where id = '1'; | balance = 100 事务2 读取的数据已经不存在，是事务1中的临时脏数据

+ 1.3.2 non-repeatable read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ UNCOMMITTED; | set session transaction isolation level READ UNCOMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = '1'; | 事务2读取 zhao的账户，balance = 100 
T4 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1为zhao存入1000元
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = '1'; | balance = 1100 事务2 读取zhao的数据已经不是100，在一个事务里边读取同一个记录，得到不同值，不可重复读

+ 1.3.3 phantom read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ UNCOMMITTED; | set session transaction isolation level READ UNCOMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额
T4 | insert into t_account(id, name, balance) values('5', 'zhou', 500); | | 事务1添加一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500, 添加时产生的幻读

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ UNCOMMITTED; | set session transaction isolation level READ UNCOMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500
T4 | delete from t_account where id = '5'; | | 事务1添删除一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = null, 删除时产生的幻读


#### 1.4 Read committed VS Read committed

+ 1.4.1 dirty read

时间|session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ COMMITTED; | set session transaction isolation level READ COMMITTED; | 设置隔离级别
T3 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1修改zhao的账户，为他存入1000元
T4 | | select balance from t_account where id = '1'; | balance = 100|事务2读取事务2开始之前已提交的记录
T5 | rollback; |  | 事务1回滚
T6 | | select balance from t_account where id = '1'; | balance = 100 事务2 读取的数据没有变化，不存在脏读问题

+ 1.4.2 non-repeatable read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ COMMITTED; | set session transaction isolation level READ COMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = '1'; | 事务2读取 zhao的账户，balance = 100 
T4 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1为zhao存入1000元
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = '1'; | balance = 1100 事务2 读取zhao的数据已经不是100，在一个事务里边读取同一个记录，得到不同值，不可重复读

+ 1.4.3 phantom read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ COMMITTED; | set session transaction isolation level READ COMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额
T4 | insert into t_account(id, name, balance) values('5', 'zhou', 500); | | 事务1添加一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500, 添加时产生的幻读

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level READ COMMITTED; | set session transaction isolation level READ COMMITTED; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500
T4 | delete from t_account where id = '5'; | | 事务1添删除一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = null, 删除时产生的幻读



#### 1.5 repeatable Read VS repeatable Read

+ 1.5.1 dirty read

时间|session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level REPEATABLE READ; | set session transaction isolation level REPEATABLE READ; | 设置隔离级别
T3 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1修改zhao的账户，为他存入1000元
T4 | | select balance from t_account where id = '1'; | balance = 100|事务2读取事务2开始之前已提交的记录
T5 | rollback; |  | 事务1回滚
T6 | | select balance from t_account where id = '1'; | balance = 100 事务2 读取的数据没有变化，不存在脏读问题

+ 1.5.2 repeatable read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level REPEATABLE READ; | set session transaction isolation level REPEATABLE READ; | 设置隔离级别
T3 | | select balance from t_account where id = '1'; | 事务2读取 zhao的账户，balance = 100 
T4 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1为zhao存入1000元
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = '1'; | balance = 100 事务2 读取zhao的数据仍是100，可重复读

+ 1.5.3 phantom read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level REPEATABLE READ; | set session transaction isolation level REPEATABLE READ; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额, 查不到
T4 | insert into t_account(id, name, balance) values('5', 'zhou', 500); | | 事务1添加一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500, 查不到，添加时没有产生的幻读，但是不是最新的结果

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level REPEATABLE READ; | set session transaction isolation level REPEATABLE READ; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500
T4 | delete from t_account where id = '5'; | | 事务1添删除一条id为5的记录
T5 | commit; |  | 事务1提交
T6 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500, 删除时没有产生的幻读，但是不是最新的结果


#### 1.6 serializable VS serializable


+ 1.6.1 dirty read

时间|session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level SERIALIZABLE; | set session transaction isolation level SERIALIZABLE; | 设置隔离级别
T3 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1修改zhao的账户，为他存入1000元
T4 | | select balance from t_account where id = '1'; | 事务2被阻塞，直到事务1 结束才能读到数据
T5 | rollback; |  | 事务1回滚，事务2的查询返回结果, balance = 100
T6 | | select balance from t_account where id = '1'; | balance = 100 事务2 读取的数据没有变化，不存在脏读问题

+ 1.6.2 repeatable read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level SERIALIZABLE; | set session transaction isolation level SERIALIZABLE; | 设置隔离级别
T3 | | select balance from t_account where id = '1'; | 事务2读取 zhao的账户，balance = 100 
T4 | update t_account set balance = 100 + 1000 where id = '1'; | | 事务1为zhao存入1000元, 事务1 被阻塞，直到事务2提交
T5 |  | commit; | 事务2提交，事务2提交前可重复读
T6 | commit; | | 事务1 解除阻塞，并执行成功，提交

+ 1.6.3 phantom read

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level SERIALIZABLE; | set session transaction isolation level SERIALIZABLE; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额, 查不到
T4 | insert into t_account(id, name, balance) values('5', 'zhou', 500); | | 事务1添加一条id为5的记录，被阻塞
T5 |  | commit; | 事务2提交，事务2提交前可以成功避免幻读
T6 | commit; |  | 事务1 解除阻塞，并执行成功，提交

时间| session 1 | session 2 | 备注
---|---|---|---
T1 | set autocommit = 0; | set autocommit = 0; | 关闭事务自动提交, 并提交当前事务 
T2 | set session transaction isolation level SERIALIZABLE; | set session transaction isolation level SERIALIZABLE; | 设置隔离级别
T3 | | select balance from t_account where id = 5; | 事务2查询id为5的账户余额 balance = 500
T4 | delete from t_account where id = '5'; | | 事务1添删除一条id为5的记录，被阻塞
T5 |  |  commit; | 事务2提交, 事务2提交前可以成功避免幻读
T6 | commit; |  |  事务1 解除阻塞，并执行成功，提交
