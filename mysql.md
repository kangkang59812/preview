# mysql执行计划

max_connections是MySQL最大并发连接数，默认值是151

#### 1、执行计划中包含的信息

|    Column     |                    Meaning                     |
| :-----------: | :--------------------------------------------: |
|      id       |            The `SELECT` identifier             |
|  select_type  |               The `SELECT` type                |
|     table     |          The table for the output row          |
|  partitions   |            The matching partitions             |
|     type      |                 The join type                  |
| possible_keys |         The possible indexes to choose         |
|      key      |           The index actually chosen            |
|    key_len    |          The length of the chosen key          |
|      ref      |       The columns compared to the index        |
|     rows      |        Estimate of rows to be examined         |
|   filtered    | Percentage of rows filtered by table condition |
|     extra     |             Additional information             |

**id**

select查询的序列号，包含一组数字，表示查询中执行select子句或者操作表的顺序

id号分为三种情况：

​		1、如果id相同，那么执行顺序从上到下

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal;
```

​		2、如果id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

```sql
explain select * from emp e where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

​		3、id相同和不同的，同时存在：相同的可以认为是一组，从上往下顺序执行，在所有组中，id值越大，优先级越高，越先执行

```sql
explain select * from emp e join dept d on e.deptno = d.deptno join salgrade sg on e.sal between sg.losal and sg.hisal where e.deptno in (select d.deptno from dept d where d.dname = 'SALES');
```

**select_type**

主要用来分辨查询的类型，是普通查询还是联合查询还是子查询

| `select_type` Value  |                           Meaning                            |
| :------------------: | :----------------------------------------------------------: |
|        SIMPLE        |        Simple SELECT (not using UNION or subqueries)         |
|       PRIMARY        |                       Outermost SELECT                       |
|        UNION         |         Second or later SELECT statement in a UNION          |
|   DEPENDENT UNION    | Second or later SELECT statement in a UNION, dependent on outer query |
|     UNION RESULT     |                      Result of a UNION.                      |
|       SUBQUERY       |                   First SELECT in subquery                   |
|  DEPENDENT SUBQUERY  |      First SELECT in subquery, dependent on outer query      |
|       DERIVED        |                        Derived table                         |
| UNCACHEABLE SUBQUERY | A subquery for which the result cannot be cached and must be re-evaluated for each row of the outer query |
|  UNCACHEABLE UNION   | The second or later select in a UNION that belongs to an uncacheable subquery (see UNCACHEABLE SUBQUERY) |

```sql
--sample:简单的查询，不包含子查询和union
explain select * from emp;

--primary:查询中若包含任何复杂的子查询，最外层查询则被标记为Primary
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--union:若第二个select出现在union之后，则被标记为union
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--dependent union:跟union类似，此处的depentent表示union或union all联合而成的结果会受外部表影响
explain select * from emp e where e.empno  in ( select empno from emp where deptno = 10 union select empno from emp where sal >2000)

--union result:从union表获取结果的select
explain select * from emp where deptno = 10 union select * from emp where sal >2000;

--subquery:在select或者where列表中包含子查询
explain select * from emp where sal > (select avg(sal) from emp) ;

--dependent subquery:subquery的子查询要受到外部表查询的影响
explain select * from emp e where e.deptno in (select distinct deptno from dept);

--DERIVED: from子句中出现的子查询，也叫做派生类，
explain select staname,ename supname from (select ename staname,mgr from emp) t join emp on t.mgr=emp.empno ;

--UNCACHEABLE SUBQUERY：表示使用子查询的结果不能被缓存
 explain select * from emp where empno = (select empno from emp where deptno=@@sort_buffer_size);
 
--uncacheable union:表示union的查询结果不能被缓存：sql语句未验证
```

**table**

对应行正在访问哪一个表，表名或者别名，可能是临时表或者union合并结果集
		1、如果是具体的表名，则表明从实际的物理表中获取数据，当然也可以是表的别名

​		2、表名是derivedN的形式，表示使用了id为N的查询产生的衍生表

​		3、当有union result的时候，表名是union n1,n2等的形式，n1,n2表示参与union的id

**type**

type显示的是访问类型，访问类型表示我是以何种方式去访问我们的数据，最容易想的是全表扫描，直接暴力的遍历一张表去寻找需要的数据，效率非常低下，访问的类型有很多，效率从最好到最坏依次是：

system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL 

一般情况下，得保证查询至少达到range级别，最好能达到ref

```sql
--all:全表扫描，一般情况下出现这样的sql语句而且数据量比较大的话那么就需要进行优化。
explain select * from emp;

--index：全索引扫描这个比all的效率要好，主要有两种情况，一种是当前的查询时覆盖索引，即我们需要的数据在索引中就可以索取，或者是使用了索引进行排序，这样就避免数据的重排序
explain  select empno from emp;

--range：表示利用索引查询的时候限制了范围，在指定范围内进行查询，这样避免了index的全索引扫描，适用的操作符： =, <>, >, >=, <, <=, IS NULL, BETWEEN, LIKE, or IN() 
explain select * from emp where empno between 7000 and 7500;

--index_subquery：利用索引来关联子查询，不再扫描全表
explain select * from emp where emp.job in (select job from t_job);

--unique_subquery:该连接类型类似与index_subquery,使用的是唯一索引
 explain select * from emp e where e.deptno in (select distinct deptno from dept);
 
--index_merge：在查询过程中需要多个索引组合使用，没有模拟出来

--ref_or_null：对于某个字段即需要关联条件，也需要null值的情况下，查询优化器会选择这种访问方式
explain select * from emp e where  e.mgr is null or e.mgr=7369;

--ref：使用了非唯一性索引进行数据的查找
 create index idx_3 on emp(deptno);
 explain select * from emp e,dept d where e.deptno =d.deptno;

--eq_ref ：使用唯一性索引进行数据查找
explain select * from emp,emp2 where emp.empno = emp2.empno;

--const：这个表至多有一个匹配行，
explain select * from emp where empno = 7369;
 
--system：表只有一行记录（等于系统表），这是const类型的特例，平时不会出现
```

 **possible_keys** 

​        显示可能应用在这张表中的索引，一个或多个，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**key**

​		实际使用的索引，如果为null，则没有使用索引，查询中若使用了覆盖索引，则该索引和查询的select字段重叠。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**key_len**

表示索引中使用的字节数，可以通过key_len计算查询中使用的索引长度，在不损失精度的情况下长度越短越好。

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**ref**

显示索引的哪一列被使用了，如果可能的话，是一个常数

```sql
explain select * from emp,dept where emp.deptno = dept.deptno and emp.deptno = 10;
```

**rows**

根据表的统计信息及索引使用情况，大致估算出找出所需记录需要读取的行数，此参数很重要，直接反应的sql找了多少数据，在完成目的的情况下越少越好

```sql
explain select * from emp;
```

**extra**

包含额外的信息。MySQL的架构分成了server层和存储引擎层（storage engine），server层通过调用存储引擎层来返回数据。    

```sql
--using filesort:说明mysql无法利用索引进行排序，只能利用排序算法进行排序，会消耗额外的位置
explain select * from emp order by sal;

--using temporary:建立临时表来保存中间结果，查询完成之后把临时表删除
explain select ename,count(*) from emp where deptno = 10 group by ename;

--using index: 表示覆盖索引即可满足查询要求，因而无需再回表

--using index & using where: 表示首先存储引擎通过索引检索将检索结果返回（仍然不需要回表），然后在Server层再通过where语句对检索结果进行过滤。

-- Using where表示MySQL将对存储引擎层提取的结果进行过滤，过滤条件字段无索引。例如：select * from test where age > 30; 所以，Using where本身其实和是否使用索引无关，它表示的是Server层对存储引擎层返回的数据所做的过滤。当然该过滤可能会回表，也可能不会回表，取决于查询字段是否被索引覆盖。

```

#### 性能调优

1. 保证从内存中读取数据。讲数据保存在内存中
2. 确定MySQL的最大连接数
3. 为临时表分配足够的内存
4. 增加线程缓存大小

#### 数据读取

1. 系统**从磁盘中读取数据到内存时是以磁盘块（block）为基本单位**，位于同一个磁盘块中的数据会被一次性读取出来。
2. innodb存储引擎中有页（Page）的概念，**页是数据库管理磁盘的最小单位**，innodb存储引擎中默认每个页的大小为16kb，每次读取磁盘时都将页载入内存中。
3. 系统一个磁盘块的大小空间往往没有16kb这么大，**因此innodb每次io操作时都会将若干个地址连续的磁盘块的数据读入内存，从而实现整页读入内存**
4. Innodb 存储引擎保证了每一页至少有两条记录，如果一页当中的记录过大，**会截取前 768 个字节存入页中**，其余的放入 BLOB Page

#### 引擎

1. InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务

2. InnoDB支持外键，而MyISAM不支持。对一个包含外键的InnoDB表转为MYISAM会失败

3. **InnoDB是聚簇索引**，使用B+Tree作为索引结构，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。.frm文件存储表结构，字段长度等, 对于数据库a,表b来说，data\a中会产生1个或者2个文件

   如果采用独立表存储模式，data\a中还会产生b.ibd文件（存储数据信息和索引信息）

   如果采用共存储模式的，数据信息和索引信息都存储在ibdata1中

   如果采用分区存储，data\a中还会有一个b.par文件（用来存储分区信息）

   **MyISAM是非聚簇索引**，使用B+Tree作为索引结构，索引和数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

#### 范式

1. 要求数据库表的每一列都是不可分割的原子数据项
2. 确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）
3. 需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关，即不能是传递函数依赖



#### 索引

https://segmentfault.com/q/1010000004197413

##### 索引覆盖

[索引覆盖](https://www.cnblogs.com/myseries/p/11265849.html)，explain中extra字段**using index。表明通过检索索引就能查到数据，无需回表，即通过索引树就能查到数据**。

优化器会在索引存在的情况下，通过符合RANGE范围的条数和总数的比例来选择是使用索引还是进行全表遍历

<img src="/Users/kangkang/Library/Application Support/typora-user-images/image-20200712154316840.png" alt="image-20200712154316840" style="zoom: 25%;" />



<img src="/Users/kangkang/Library/Application Support/typora-user-images/image-20200630185155640.png" alt="image-20200630185155640" style="zoom:33%;" />

```mysql
create table user (
    id int primary key,
    name varchar(20),
    sex varchar(5),
    index(name)
)engine=innodb;

-- 只在name上建立索引
-- 能够命中name索引，索引叶子节点存储了主键id，通过name的索引树即可获取id和name，无需回表，符合索引覆盖，效率较高
select id,name from user where name='shenjian';　

-- 能够命中name索引，索引叶子节点存储了主键id，但sex字段必须回表查询才能获取到，不符合索引覆盖，需要再次通过id值扫码聚集索引获取sex字段，效率会降低。
-- 回表：回表查询，先定位主键值，再定位行记录，它的性能较扫一遍索引树更低
select id,name,sex from user where name='shenjian';　

-- 如果把(name)单列索引升级为联合索引(name, sex)就不同了。
id  、 id name 、 id name sex 三种方式查询都能命中索引覆盖，无需回表

-- 1. 索引覆盖可以优化：全表count,name上有索引可以提高效率
select count(name) from user;
-- 不会用索引，但是type是index，extra是using index

-- 2. 如果有回表，可以通过升级索引为联合索引
-- 3. 分页查询

```

##### using where

有索引参加但是最终不需要回表查询，也是using where

如果使用了过滤，没有索引参加，那就是using where

Using where本身其实和是否使用索引无关，它表示的是**Server层对存储引擎层返回的数据所做的过滤**。当然该过滤可能会回表，也可能不会回表，取决于查询字段是否被索引覆盖

##### using index condition

首先mysql server和storage engine是两个组件，server负责sql的parse，执行； storage engine去真正的做数据/index的读取/写入。以前是这样：server命令storage engine按index key把相应的数据从数据表读出，传给server，**然后server来按where条件（index filter和table filter）做选择**。

而在MySQL 5.6加入ICP后，Index Filter与Table Filter分离，**Index Filter下降到InnoDB的索引层面进行过滤**，如果不符合条件则无须读数据表，减少了回表与返回MySQL Server层的记录交互开销，节省了disk IO，提高了SQL的执行效率 

**过滤完回表了**，extra是using index condition

仅适用于二级索引，一般发生在查询字段无法被二级索引覆盖的场景，该场景下往往需要回表。通过ICP，可以减少存储引擎返回的行记录，从而减少了IO操作

##### using where & using index

##### 索引where条件提取规则

https://www.cnblogs.com/Terry-Wu/p/9273177.html

Index Key(Fist key & Last Key)在范围之内

Index Filter 范围之内的索引过滤

Table Filter 回表读取了完整记录，并且满足以上要求，再次过滤

##### ICP

a. 当关闭ICP时，index仅仅是data access的一种访问方式，存储引擎通过索引回表获取的数据会传递到MySQL Server层进行where条件过滤，也就是做index filter和table filter。

b. 当打开ICP时，**如果部分where条件能使用索引中的字段，MySQL Server会把这部分下推到引擎层**，**可以利用index filter的where条件在存储引擎层进行数据过滤**，而非将所有通过index access的结果传递到MySQL server层进行where过滤。

优化效果：ICP能减少引擎层访问基表的次数和MySQL Server访问存储引擎的次数，减少io次数，提高查询语句性能。



##### 索引类型

主键创建的是聚集索引，其它索引创建的是非聚集索引（可能要回表）

- **UNIQUE**(**唯一索引**)：不可以出现相同的值，可以有NULL值；在有重复数据列上添加该索引会报错
- **INDEX**(**普通索引**)：允许出现相同的索引内容；
- **PROMARY KEY**(**主键索引**)：不允许出现相同的值，不允许为null；
- **fulltext index**(**全文索引**)：可以针对值中的某个单词，但效率确实不敢恭维；
- **组合索引**：实质上是将多个字段建到一个索引里，列值的组合必须唯一；

https://www.cnblogs.com/xujunkai/p/12492817.html

```mysql
create index idx_test03_c1234 on test03(c1,c2,c3,c4);
```

1. 顺序无关，内部会自己优化

2. ```mysql
   c1='a1' and c2='a2' and c3>'a3' and c4='a4' c1-c2用索引查找，c3用索引排序，c4不用索引
   
   c1='a1' and c2='a2' and c4>'a4' and c3='a3'  c1-c3用索引查找，c4用索引排序
   
   c1 = '1' and c2 = '1' and c4 = '1'，这种情况因为c3没有在条件中，所以只会用到c1,c2索引
   
   c1 = '1' and c2 > '1' and c3 = '1' 使用范围条件的时候，也会使用到该处的索引，但后面的索引都不会用到
   
   
   ```

   在使用order by关键字的时候，如果待排序的内容不能由所使用的索引直接完成排序的话，那么mysql有可能就要进行文件排序

3. ```
   group by c1 | order by c1，由于没有where的铺垫，不使用任何索引
   
   where c1 = '1' group | order by c2，使用c1,c2索引
   
   where c1 = '1' group | order by c3，只使用c1索引
   
   where c1 > '1' group | order by c2，前面也说了，范围搜索会断掉连接，所以也只会使用c1索引
   
   关于顺序：
   
   where c1 = '1' group | order by c2,c3，使用c1,c2,c3索引
   
   where c1 = '1' group | order by c3,c2，只使用c1索引
   ```

   
#### 复制

<<<<<<< HEAD
#### 事务

并发控制的单位

https://www.cnblogs.com/wyaokai/p/10921323.html

#### MVCC

多版本并发控制

https://blog.csdn.net/whoamiyang/article/details/51901888
=======
1、主服务器（master）将数据更改的操作记录写到二进制redo日志（binlog）中

2、从服务器（slave）将主服务器中的二进制日志复制到自己的中继日志（relay log）中

3、从服务器重做中继日志中的日志，把更改应用到自己的数据库中，保证数据的最终一致性

复制的原理实际上就是一个完全备份加上二进制日志的备份还原，为了保证从服务器的性能，从服务器开启了两个线程：一个是I/O线程，负责接收主服务器的二进制日志，并将其复制到自己的中继日志中；另外一个是sql线程，负责读取中继日志中的记录并执行sql以达到主从数据库的一致性。

##### 异步复制

为了保证主服务器的性能和响应及时性，mysql默认的复制是异步的，而并不是实时进行的。主服务器在执行完客户端提交的事务后，立即将结果返回给客户端，而并不会关心从服务器是否已经接收并处理。这就带来主从服务器上的执行时延，如果主服务器的压力很大，那么可能导致只从服务器之间存在较大的时延，一旦主服务器宕机了，此时强行提升从服务器作为主服务器，会导致数据的缺失

##### 完全同步复制

为了解决异步复制可能导致的主从服务器数据不一致问题，mysql5.5版本开始支持完全同步复制策略。该策略下，当主服务器执行完一个事务后，从服务器也执行成功时才返回给客户端。因为需要等待所有的从服务器执行完事务才能将结果返回给客户端，所以主从复制的性能会下降很多

##### 半同步复制

半同步复制是介于异步复制和完全同步复制之间的一种折中方案。当主服务器执行完一个事务时，只需要有一台从服务器执行完这个事务便可以返回，这样和完全同步复制相比性能就会有一个大大的提升；同时，半同步复制能保证至少有一个从服务器与主服务器中的数据完全一致，即使主服务器宕机，也可以直接将这个从服务器提升为主服务器
>>>>>>> fb7828e1b2c6144b5eabac6f816193ba9d07fad4

#### ACID

原子性，一致性，隔离性，持久性

#### 分布式事务

#### redolog和binlog

- redo log是属于innoDB层面，binlog属于MySQL Server层面的，这样在数据库用别的存储引擎时可以达到一致性的要求。
- redo log是物理日志，记录该数据页更新的内容；binlog是逻辑日志，记录的是这个更新语句的原始逻辑
- redo log是循环写，日志空间大小固定；binlog是追加写，是指一份写到一定大小的时候会更换下一个文件，不会覆盖。
- binlog可以作为恢复数据使用，主从复制搭建，redo log作为异常宕机或者介质故障后的数据恢复使用。