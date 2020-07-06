# Mysql 事务

## 一、事务及特性

> 数据库事务是数据库执行过程中的一个逻辑单元,由一个有限的数据库操作序列构成,区别文件系统的重要特性之一.

### 事务拥有的四个特性

1. 原子性 (Atomicity)

> 事务开始后所有的操作,要么全部做完,要么都不做.如:插入 100 条数据,操作前10条失败那么后 90条不再执行,新增的10条也会被回滚.

2. 一致性 (Consistency)

> 事务将从数据库的一种状态,变化成另外一种状态,事务开始前和结束后,数据库的完整性约束没有被破坏.如:用户id 通过修改用户id事务后,用户id 变得非唯一了,则表明遭到了破坏.

3. 隔离型 (Isolation)

> 要求每个读写事务对其他事务的操作对象能相互分离,即该事务提交前对其他事务不可见.也可以理解为多个事务并发访问时,事务之间是隔离的.一个事务不应该影响其他事务的运行结果,指在兵法环境中,当不同的事务同时操作相同的数据时,每个事务都有各自的完整数据空间.由并发事务所做的修改不影响其他事务.例如一个用户更新自己的个人信息的同时,是不能看到系统管理员也在更新该用户的个人信息的.(未提交时)

4. 持久性 (Durability)

> 事务一旦提交,则结果是永久性的,即使发生宕机故障,数据库也能将数据恢复,也就是说事务完成后,事务对数据库的所有更新将保存到数据库,不能回滚,这只是从事务本身的角度来保证

注: mysql 使用 redo log 来保证事物持久性的.

## 二、事务隔离级别

sql 标准定义的四种隔离级别被ANSI和ISO/IEC 采用,美中级别对事务的处理能力会有不同的影响.

* READ UNCOMMITTED (读未提交)

> 该隔离级别的事务会读到其他未提交事务的数据,此现象称之为脏读.(也就是读不到其他事务提交后的数据的)

1. 准备两个终端,准备 test 表写入一条测试数据,并且隔离级别调整为 READ UNCOMMITTED 

```
SET @@session.transaction_isolation = 'READ-UNCOMMITTED';
```
2. 登陆 mysql 终端 开启事务,将 1 的记录的 name 更新为ccc
```
begin;
update test set name = 'ccc' where id = 1;
select * from test;--可以看到 id 为 1 的 name 为 ccc
```

3. 登陆另外一个终端,开启事务,查询 test 表中的内容

```
use test;
bigin;
select * from test; -- 可以看到 id 为 1 的 name 为 cc 
```
因为最后一步读到了 第一个终端中没有提交的事务,即产生了脏读,大部分业务场景都不允许脏读出现,但是此隔离级别下数据库并发是最好的.

* READ COMMITTED(读提交)

一个事务可以读取另一个已提交的事务,多次读取会造成读取不一样的结果,此现象称为不可重复读问题,Oracle 和SQLServer 默认隔离级别(可以读到其他事务提交的内容)

1. 准备两个终端,设置事务隔离级别为 READ COMMITTED

```
SET @@session.transaction_isolation = 'READ-COMMITTED';
```
2. 登陆终端 1 开启事务将 test 表内的 id 为 1 的 name 更新为 ‘hhh’

```
begin;
update test set name = 'hhh' where id=1;
select * from test; -- 可以看到 id 为 1 name 为 hhh
```
3. 登陆终端 2 开启事务将,查询 test 表

```
begin;
select * from test; 发现 id 为 1 的记录 name 为 ’ccc‘;
```
4. 登陆终端 1 提交事务

```
commit;
```
5. 登陆终端 2 查询

```
select * from test; 发现 id 为 1 的记录已更新为 ‘hhh’
```

在终端 2 中第一次查询没有查询到终端 1 还未提交的内容,当终端 1 提交之后,终端 2 就可以查看到 终端 1 中的提交内容,说明该事务隔离级别下 是可以读到其他事务提交的内容的.

* REPEATABLE READ(可重复读)

> 这个隔离级别是 Mysql 默认的隔离级别,在同一个事务里面,select 的结果是事务开始时间点的状态,因此同样的,select 操作读到的结果是一致的,但是会有幻读现象,Mysql 的 InnoDB 引擎可以通过 next-key locks 机制避免幻读.(事务只会读取到事务开启时读取到的内容)

1. 准备两个终端,设置一下隔离级别

```
SET @@session.transaction_isolation = 'REPEATABLE-READ';
```

2. 登陆终端 1 开启事务查询,数据为原来记录

```
begin;
select * from test;
```

3. 登陆终端 2 开启事务查询,数据为原来记录

```
begin;
select * from test;
```
4. 切换到终端 1 增加一条记录并且体积爱哦

```
insert into test(id,name,value) values (1,'a','a')
commit;
```
5.切换到终端 2 查询还是无记录

```
select * from test; -- 查询无记录
```
通过这一步可以证明,在该隔离级别下,已经读取不到别的已提交的事务,如果想看到mysql终端 1 提交的事务,在 mysql 2 将当前事务提交之后,再去查询就可以读取到 mysql 终端 1 提交的事务

6. 在 终端 2 中插入相同 id 值的数据

```
insert into test (id,name,value) values(1,'a','a')-- 主键冲突
```
该隔离级别下的问题 会出现幻读.

* SERIALIZABLE (序列化)

在该隔离级别下事务都是串行执行的,Mysql 数据库的InnoDB引擎会读操作隐式加一把读共享锁,从而避免了 脏读 不可重复读,和幻读的问题.

1. 准备两个终端,设置事务隔离级别

```
SET @@session.transaction_isolation = 'SERIALIZABLE';
```

2. 登陆终端 1,开启一个事务,并写入一条数据.

```
begin;
insert into test (id,name,value) values (15,'ooo','ooo');
```

3. 登陆终端 2 开启一个事务查询

```
begin;
select * from test;--此时会一直卡住
```
4. 切换到终端1,提交事务

```
commit;
```
一旦事务提交,mysql 终端 2 就会立刻返回 终端 1 中的提交记录,否则就会一直卡住,知道超时,其中的超时参数是由 innodb_lock_wait_timeout 控制.由于每条 select
语句都会加锁,所以隔离级别数据库并发能力最差.

隔离级别残生的问题


| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | --- | --- | --- |
| 读未提交 | 可以出现 | 可以出现 | 可以出现 |
| 读 提交 | 不允许出现 | 可以出现  | 可以出现 |
| 可重复读 | 不允许出现 | 不允许出现 |可以出现  |
| 序列化 | 不允许出现 | 不允许出现 | 不允许出现 |
