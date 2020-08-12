**Redis是一个开源的内存中的数据结构存储系统，它可以用作：数据库、缓存和消息中间件**。Redis 将数据储存在内存里面，读写数据的时候都不会受到硬盘 I/O 速度的限制，所以速度极快。**采用的是基于内存的采用的是单进程单线程模型的 KV 数据库，由C语言编写，官方提供的数据是可以达到100000+的QPS**（每秒内查询次数）

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnap3u92jj30hy0acjrs.jpg" alt="图片描述" style="zoom:50%;" />

横轴是连接数，纵轴是QPS

##### 为什么这么快：

1. 完全基于内存，绝大部分请求是纯粹的内存操作，非常快速。数据存在内存中，查找和操作的时间复杂度都是O(1)
2. 数据结构简单，对数据操作也简单
3. 采用单线程，避免了不必要的上下文切换和竞争条件，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
4. 使用多路I/O复用模型，非阻塞IO
5. 直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求



因为Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了



使用 object encoding key命令可以查看编码格式 使用 debug object key命令可以查看更多信息

select index选择0-16的数据库，默认0，互相之间不互通

1. key 有过期特性,唯一的；
2. Redis具有发布订阅功能
3. Redis具有一定的持久化的特性

用作缓存中间件,当我们处于分布式环境下想要操作同一个资源的时候，**我们可以用同一个key然后setnx。谁set成功了，谁就获得该锁，然后可以获取到资源，然后进行操作**

```c++
expire book 60 //设置过期时间,-1永久存储
setnx //只有当key不存在的时候才能设置成功
```



##### 基础数据结构

string、list、hash、set 和 zset对应java的string,list,hashmap,set,sortedset

##### 编码

Redis的embstr编码方式和raw编码方式在3.0版本之前是以**39字节为分界的**，也就是说，如果一个字符串值的长度小于等于39字节，则按照embstr进行编码，否则按照raw进行编码。而在3.2版本之后，则变成了**44字节为分界**

在 Redis 中 string 是可以修改的,是动态字符串(Simple Dynamic String 简称 SDS)他的内部结构更像是一个 ArrayList,维护一个字节数组并预分配冗余空间以减少内存的频繁分配.当字符串的长度小于 1MB时,每次扩容都是加倍现有的空间,如果字符串长度超过 1MB 时,每次扩容时只会扩展 1MB 的空间.

```c++
struct SDS{
    T capacity;        //数组容量
    T len;            //实际长度
    byte flages;    //标志位,低三位表示类型
    byte[] content;    //数组内容
}
```

capacity和len 都是泛型, 在创建字符串的时候 len 会和 capacity 一样大,没有冗余的空间,因为修改字符串的场景很少

```c++
struct RedisObject{
    int4 type;        //数据类型 5 种
    int4 encoding;    //键值内部编码格式 int 或 embstr 等等
    int24 lru;        // 当内存超限时采用LRU算法清除内存中的对象
    
    int32 refcount;    //改键值被引用的数量
    void *ptr;        //对象内容
}
```

###### int编码

当储存的值是64 位有符号整数类型的时候将会采用 **int** 编码,这时可以使用键值自增操作.Redis 在启动时会建立1w 个`redisObject`共享对象下文会讲到,值在[0,1000)之间.如果存入整数的值在[0,1000)中Redis将不会创建新的对象,而是直接指向共享对象,键值不额外占用空间

###### embstr编码

当存储的字符串长度较短时(len<=44 字节),Redis将会采用 **embstr** 编码.embstr 即embedded string 嵌入式的字符串.将SDS结构体嵌入RedisObject对象中, 使用 malloc 方法一次分配内存地址是连续的.

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnap6tdz7j30be0h2q4q.jpg" alt="image-20200605124817940" style="zoom:25%;" />

###### raw编码

当存储的字符串长度较长时(len>44 字节),Redis 将会采用 **raw** 编码,和 **embstr** 最大的区别就是 RedisObject 和 SDS 不在一起了,内存地址不再连续了.

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnap9zom7j30h30j7jtq.jpg" alt="image-20200605124906501" style="zoom:25%;" />

###### 为什么是44字节分界

Redis 默认的内存分配器jemalloc分配内存大小的单位是$2^n$次方,为了容纳一个完整的 embstr 对象,最少会分配 32 字节的空间,再长些就是 64 字节,再之后就认为这是一个大字符串不适合用 embstr 存储,而改用 raw 编码了

根据resdisobject结构 4bytes（前三个）+4bytes+8bytes+3bytes(string的前三个)=19，64-19=44

##### LRU和基于Redis的实现

HashMap存储key,value， value是指向双向链表中key的指针  [java实现](https://zhuanlan.zhihu.com/p/34133067)

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnapay9mzj30mt0i0dhz.jpg" alt="image-20200605122133884" style="zoom:33%;" />

save(key, value)，首先在 HashMap 找到 Key 对应的节点，如果节点存在，更新节点的值，并把这个节点移动队头。如果不存在，需要构造新的节点，并且尝试把节点塞到队头，如果LRU空间不足，则通过 tail 淘汰掉队尾的节点，同时在 HashMap 中移除 Key

get(key)，通过 HashMap 找到 LRU 链表节点，把节点插入到队头，返回缓存的值

**需要额外的存储存放 next 和 prev 指针，牺牲比较大的存储空间，显然是不划算的。所以Redis采用了一个近似的做法，就是随机取出若干个key，然后按照访问时间排序后，淘汰掉最不经常使用的**

淘汰策略：`volatile-lru` 设置了过期时间的key参与近似的lru淘汰策略；所有的key均参与近似的lru淘汰策略

Redis会基于server.maxmemory_samples配置选取固定数目的key，然后比较它们的lru访问时间，然后淘汰最近最久没有访问的key，maxmemory_samples的值越大，Redis的近似LRU算法就越接近于严格LRU算法，但是相应消耗也变高，对性能有一定影响，样本值默认为5。

##### List

可以用于消息队列。list 的底层不是简单的 LinkedList，而是 ziplist（压缩列表）或 quicklist（快速列表）

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnapff24oj30ti047wfb.jpg" alt="image-20200605123038897" style="zoom:33%;" />

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnapd55zyj30rs088gnd.jpg" alt="image-20200605123053166" style="zoom:33%;" />

(以前数据少的时候用压缩的，现在全是quicklist)压缩列表使用一块连续的内存，元素间紧挨着存储没有空隙. 每个元素中包括前一个元素的长度、当前元素的编码和内容. 可能会造成后面的元素雪崩式的更改 prevlen，即 **联级更新**。因此，list 中存储的元素不应该太多、太大

有了 ztail_offset 就可以快速的定位到最后一个节点,这样就可以倒序遍历了.也就是说 ziplist支持双向遍历.

在 quicklist 中每个 ziplist 默认大小是 8kb,超出这个字节就会增加一个 ziplist.这个默认大小是可配置的,由`list-max-ziplist-size`决定

```c++
struct entry{
    int<var> prevlen;            //前一个 entry 的长度
    int<var> encoding;            //元素类型编码
    optional byte[] content;    //元素内容
}
```

##### Hash

内部实现与 Java 中的 HashMap 一致（数组 + 链表）。字典内部有两个 HashTable，Rehash（扩容或缩容）时用于存储新旧两份数据.
扩容时机：元素个数等于数组长度（如果 Redis 正在做 bgsave(持久化) 时,可能不会去扩容,因为要减少内存页的过多分离(Copy On Write).但是如果 hashtable 已经非常满了,元素的个数达到了数组长度的 5 倍时,Redis 会强制扩容）
扩容方式：2 倍数组长度
缩容时机：元素个数少于数组长度 10%

Redis 中的 Hash和 Java的 HashMap 更加相似,都是数组+链表的结构.当发生 hash 碰撞时将会把元素追加到链表上.值得**注意的是在 Redis 的 Hash 中 value 只能是字符串**

HashMap 扩容是个很耗时的操作,需要去申请新的数组,为了追求高性能,Redis 采用了`渐进式 rehash` 策略.这也是 hash 中最重要的部分

在扩容的时候 rehash 策略会保留新旧两个 hashtable 结构,查询时也会同时查询两个 **hashtable.Redis会将旧 hashtable 中的内容一点一点的迁移到新的 hashtable 中,**当迁移完成时,就会用新的 hashtable 取代之前的.当 hashtable 移除了最后一个元素之后,这个数据结构将会被删除(客户端没有操作时做)



##### Set和Zset

set 是没有排序的字符串集合，不允许出现重复元素，内部结构与 hash 字典一致，只是 value 为 null

zset结构，多级索引

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnapkhbwej30od08wq9j.jpg" alt="image-20200605132825918" style="zoom:50%;" />



##### 两种持久化方式

RDF 一段时间内生成指定时间点生成数据集快照 (snapshot)效率低，影响性能和数据的浪费

AOF 增量日志只是把**改变**的添加到某一个磁盘里面，当下次Redis启动的时候就加载这些改动的部分

您也可以关闭持久化功能，将Redis作为一个高效的网络的缓存数据功能使用。

##### 事务

multi：开启事务 exec：执行事务

##### spring中properties文件中的参数

name：表示你的连接池的名称也就是你要访问连接池的地址
auth：是连接池管理权属性，Container表示容器管理
type：是对象的类型
driverClassName：是数据库驱动的名称
url：是数据库的地址
username：是登陆数据库的用户名
password：是登陆数据库的密码
maxIdle，最大空闲数，始终保留在池中的最大连接数，如果启用，将定期检查限制连接，超出此属性设定的值且空闲时间超过minEvictableIdleTimeMillis的连接则释放。设为0表示无限制。
MaxActive，连接池的最大数据库连接数。设为0表示无限制。
maxWait ，最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。

##### 实现高可用

主从复制：主服务器要什么改变都复制到从服务器，数据量会非常大，每一个从服务器数据量和主服务器一样

集群：Redis Cluster中有一个16384长度的槽的概念，他们的编号为0、1、2、3……16382、16383。这个槽是一个虚拟的槽，并不是真正存在的。正常工作的时候，Redis Cluster中的每个Master节点都会负责一部分的槽，当有某个key被映射到某个Master负责的槽，那么这个Master负责为这个key提供服务，至于哪个Master节点负责哪个槽，这是可以由用户指定的，也可以在初始化的时候自动生成（redis-trib.rb脚本）。这里值得一提的是，在Redis Cluster中，只有Master才拥有槽的所有权，如果是某个Master的slave，这个slave只负责槽的使用，但是没有所有权。有一个客户端请求了某个key发现不在某个节点上，该节点会找到这个key所在的节点，然后返回给客户端，让客户端重新发起请求。

##### key过期策略

- **惰性删除**：当读/写一个已经过期的key时，会触发惰性删除策略，直接删除掉这个过期key（无法保证冷数据被及时删掉）
- **定期删除**：Redis会定期主动淘汰一批已过期的key（随机抽取一批key检查）
- **内存淘汰机制**：当前已用内存超过maxmemory限定时，触发主动清理策略

##### redis 内存淘汰机制：

noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错。
allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（常用）。
allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key。
volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。
volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。
volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。

#### 持久化

##### RDB
```bash
# 时间策略
save 900 1 # 900s内如果有1条是写入命令，就触发产生一次快照
save 300 10
save 60 10000

# 文件名称
dbfilename dump.rdb

# 文件保存路径
dir /home/work/app/redis/data/

# 如果持久化出错，主进程是否停止写入
stop-writes-on-bgsave-error yes # 当备份进程出错时，主进程就停止接受新的写入操作

# 是否压缩
rdbcompression yes # Redis本身就属于CPU密集型服务器，再开启压缩会带来更多的CPU消耗，相比硬盘成本，CPU更值钱

# 导入时是否检查
rdbchecksum yes

# 禁用RDB配置
save ""

```
##### AOF

```bash
# 是否开启aof
appendonly yes

# 文件名称
appendfilename "appendonly.aof"

# 同步方式
# always：把每个写命令都立即同步到aof，很慢，但是很安全
# everysec：每秒同步一次，是折中方案
# no：redis不处理交给OS来处理，非常快，但是也最不安全
appendfsync everysec

# aof重写期间是否同步
no-appendfsync-on-rewrite no

# 重写触发配置
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 加载aof时如果有错如何处理, 如果该配置启用，在加载时发现aof尾部不正确时，会向客户端写入一个log，但是会继续执行，如果设置为 no ，发现错误就会停止，必须修复后才能重新加载
aof-load-truncated yes

# 文件重写策略
aof-rewrite-incremental-fsync yes
```
启动时会先检查AOF文件是否存在，如果不存在就尝试加载RDB。那么为什么会优先加载AOF呢？因为AOF保存的数据更完整，通过上面的分析我们知道AOF基本上最多损失1s的数据
RDB持久化与AOF持久化可以同时存在，配合使用

#### 集群

[codis和cluster](https://www.cnblogs.com/pingyeaa/p/11294773.html)

#### 缓存穿透，雪崩，击穿

缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时从DB查询，失去缓存意义

解决办法：布隆过滤器，做完hashcode后一定不存在的数据用bitmap拦截；返回空值，并写入缓存设置过期时间  (两者可以结合）

针对key异常多、请求重复率比较低的数据，我们就没有必要进行缓存，使用第二种方案直接过滤掉。

而对于空数据的key有限的，重复率比较高的，我们则可以采用第一种方式进行缓存。


缓存雪崩是指在我们设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到DB，DB瞬时压力过重雪崩（多个key）
解决办法：事前：redis集群，保证高可用；  事中：本地缓存，限流降级；  事后：持久化


缓存击穿是key在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮（单个）

解决办法：设置互斥锁，

```java
public String get(key) {
      String value = redis.get(key);
      if (value == null) { //代表缓存值过期
          //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
		  if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
               value = db.get(key);
                      redis.set(key, value, expire_secs);
                      redis.del(key_mutex);
              } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
                      sleep(50);
                      get(key);  //重试
              }
          } else {
              return value;      
          }
 }
```
#### 分布式锁

和上面代码类似，但是如果setnx和设置过期时间不是原子性的，就可能出现死锁
可以用lua脚本(包含setnx和expire两条指令)
