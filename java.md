面向对象有四个基本特性：抽象、封装、继承、多态。抽象、封装、继承是多态的基础，多态是抽象、封装、继承的表现

#### java对象在内存的大小

https://www.cnblogs.com/ulysses-you/p/10060463.html



对象头，对象实际数据，pad(HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍)

对象头：（32位系统，4+4； 64位 8+8，开启UseCompressedOops，8+4）

**markword**：哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等

**klass**：对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例

对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度，也就是一个int类型的对象，占4字节

`jdk.nashorn.internal.ir.debug.ObjectSizeCalculator`可以评估出对象的大小

```java
// 对象A： 对象头12B + 内部对象s引用 4B + 内部对象i 基础类型int 4B + 对齐 4B = 24B
// 内部对象s 对象头12B + 2个内部的int类型8B + 内部的char[]引用 4B + 对齐0B = 24B
// 内部对象str的内部对象char数组 对象头12B + 数组长度4B + 对齐0B = 16B
// 总： 对象A 24+ 内部对象s 24B + 内部对象s的内部对象char数组 16B =64B
class A {
  String s = new String();
  int i = 0;
}

// 对象B：对象头12B + 内部对象s引用 4B + 内部对象i 基础类型int 4B + 对齐 4B = 24B
// s没有被分配堆内存空间
// 总： 对象B 24B
class B {
  String s;
  int i = 0;
}
```



#### 关键字

private static是合法的，且有着其独到的用处：为静态方法提供私有静态属性。public static常用的是为该类提供对外暴露即可以被类名直接调用的静态常量。

静态的使用注意事项：
1.静态方法只能访问静态成员（包括成员变量和成员方法）
  非静态方法可以访问静态也可以访问非静态
2.静态方法中不可以定义this，super关键字
  因为静态优先于对象存在，所以静态方法中不可以出现this，super关键字
3.主函数是静态的。



#### 抽象

抽象类必须用 `abstract` 修饰，子类**必须实现抽象类中的抽象方法**，如果有**未实现的，那么子类也必须用 abstract 修饰,也称为抽象类**。**抽象类默认的权限修饰符为 `public`**，可以定义为 public 或 procted，如果定义为 private，那么子类则无法继承。抽象类不能创建对象。里面可以有非抽象类。可以有构造函数，可以有main方法

#### 接口

  1.接口中的所有属性默认为：**public static final** ；

  2.接口中的所有方法默认为：**public abstract** ；不能用final，protect, private,  static

不能有构造器，不能有main方法，可以有default方法并且有实现体 [调用关系](https://www.cnblogs.com/sum-41/p/10878807.html)

#### 接口和抽象类的区别

1. 抽象类只能继承一次，但是可以实现多个接口
2. **接口和抽象类必须实现其中所有的方法**，抽象类中如果有未实现的抽象方法，那么子类也需要定义为抽象类。抽象类中可以有非抽象的方法
3. 接口中的变量必须用 public static final 修饰，并且需要给出初始值。所以实现类不能重新定义，也不能改变其值。
4. 接口中的方法默认是 public abstract，也只能是这个类型。不能是 static，接口中的方法也不允许子类覆写，抽象类中允许有static 的方法
5. 

#### 方法重载

方法重载只看名称和参数列表，其他的都不看，包括返回值类型、属性修饰符、范围限定等(如果只有这些不同，会报错已存在改方法)，这些都不影响方法重载的概念

#### 多态

不同类的对象对同一消息作出不同的响应就叫做多态

三个定义分别是**父类定义子类构建、接口定义实现类构建和抽象类定义实体类构建**



**多态存在的三个条件**

1、有继承关系　　

2、子类重写父类方法　　

3、**父类定义子类构建、接口定义实现类构建和抽象类定义实体类构建**

以下三种类型的方法是没有办法表现出多态特性的（因为不能被重写）：

1、static方法，因为被static修饰的方法是属于类的，而不是属于实例的

2、final方法，因为被final修饰的方法无法被子类重写

3、private方法和protected方法，前者是因为被private修饰的方法对子类不可见，后者是因为尽管被protected修饰的方法可以被子类见到，也可以被子类重写，但是它是无法被外部所引用的，一个不能被外部引用的方法，怎么能谈多态呢

**编译时多态，即方法的重载**，从JVM的角度来讲，这是一种静态分派

**运行时多态，即方法的重写**，从JVM的角度来讲，这是一种动态分派

#### JDK设计模式

http://www.imooc.com/article/284545

1. 观察者模式
2. 桥接模式, 把set和map桥接起来，同时map和set都可以独立扩展和变化

```java
Set<String> names = Collections.newSetFromMap(new ConcurrentHashMap<String, Boolean>());
```

3. 装饰者模式 两个Reader都继承了抽象类Reader，B..类里持有I..对象的引用。两个都是独立的装饰者 各自增强了被装饰类的功能

```java
BufferedReader br = new BufferedReader(new InputStreamReader(System.in))
```

4. 原型模式：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。          								浅拷贝实现 Cloneable，重写，深拷贝是通过实现 Serializable 读取二进制流
5. 建造者模式 ：和工厂方法区别，两者都是组装对象，而建造者模式更关心零件的装配顺序。如jdk中的StringBuilder StringBuffer的append方法
6. 工厂方法模式：spring的bean factory应该划分属于抽象工厂的设计模式
7. 适配器模式：泛型必须写在返回值或者void前面
8. 享元模式：主要为了减少创建对象的数量,减少内存占用和提高内存。享元模式尝试重用现有的同类对象，如果未找到匹配的对象，则创建新对象
9. 策略模式：封装行为或者算法，能在运行时动态改变类的行为或算法。jdk中compare() java.util.Comparator#compare()  多态
10. 适配器模式

#### 线程使用场景

##### newCachedThreadPool：

底层：**返回ThreadPoolExecutor实例**，corePoolSize为0；maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为60L；unit为TimeUnit.SECONDS；workQueue为SynchronousQueue(同步队列)，无界
通俗：当有新任务到来，则插入到SynchronousQueue中，由于SynchronousQueue是同步队列，它是一个没有容量的阻塞队列。每个插入操作必须等待另一个线程的对应移除操作。这意味着，如果主线程提交任务的速度高于线程池中处理任务的速度时，CachedThreadPool会不断创建新线程。极端情况下，CachedThreadPool会因为创建过多线程而耗尽CPU资源。**SynchronousQueue是一个不存储元素阻塞队列，每次要进行offer操作时必须等待poll操作，否则不能继续添加元素。**

 **适用：执行很多短期异步的小程序或者负载较轻的服务器**

##### newFixedThreadPool：

底层：返回ThreadPoolExecutor实例，接收参数为所设定线程数量n Thread，corePoolSize为n Thread，maximumPoolSize为n Thread；keepAliveTime为0L(不限时)；unit为：TimeUnit.MILLISECONDS；WorkQueue为：**new LinkedBlockingQueue() 无界阻塞队列**
通俗：创建可容纳固定数量线程的池子，每个线程的存活时间是无限的，当池子满了就不在添加线程了；如果池中的所有线程均在繁忙状态，对于新任务会进入阻塞队列中(无界的阻塞队列)maxiumPoolSize也就变成了一个无效的参数，并且运行中的线程池并不会拒绝任务。 太多就会OOM
**适用：执行长期的任务，性能好很多**

##### newSingleThreadExecutor:

底层：FinalizableDelegatedExecutorService包装的ThreadPoolExecutor实例，corePoolSize为1；maximumPoolSize为1；keepAliveTime为0L；unit为：TimeUnit.MILLISECONDS；workQueue为：**new LinkedBlockingQueue() 无界阻塞队列**
通俗：创建只有一个线程的线程池，且线程的存活时间是无限的；当该线程正繁忙时，对于新任务会进入阻塞队列中(无界的阻塞队列)
**适用：一个任务一个任务执行的场景**

##### NewScheduledThreadPool:

底层：创建ScheduledThreadPoolExecutor实例，corePoolSize为传递来的参数，maximumPoolSize为Integer.MAX_VALUE；keepAliveTime为0；unit为：TimeUnit.NANOSECONDS；workQueue为：new DelayedWorkQueue() 一个按超时时间升序排序的队列
通俗：创建一个固定大小的线程池，线程池内线程存活时间无限制，线程池可以支持定时及周期性任务执行，如果所有线程均处于繁忙状态，对于新任务会进入DelayedWorkQueue队列中，这是一种按照超时时间排序的队列结构
**适用：周期性执行任务的场景**


##### 为什么要用线程池：

减少了创建和销毁线程的次数，每个工作线程都可以被重复利用，可执行多个任务

可以根据系统的承受能力，调整线程池中工作线线程的数目，防止因为消耗过多的内存

#### JVM调优

JVM调优核心为调整年轻代、年老大的内存空间大小及使用GC发生器的类型等

#### Synchronized

https://www.jianshu.com/p/7f8a873d479c 非公平锁

Synchronized 在线程进入 ContentionList 时，等待的线程会先

尝试自旋获取锁，如果获取不到就进入 ContentionList，这明显对于已经进入队列的线程是不公平的，



通过monitor实现，motitor3要素：临界区，monitor对象和锁，条件

**临界区我们可以认为是对对象头 mutex 的 P 或者 V 操作，这是个临界区**

monitor object 内部会有相应的数据结构，例如列表，来保存被阻塞的线程

同时由于 monitor 机制本质上是基于 mutex 这种基本原语的，所以 monitor object 还必须维护一个基于 mutex 的锁

当一个线程需要获取 Object 的锁时，会被放入 EntrySet 中进行等待，如果该线程获取到了锁，成为当前锁的 owner。如果根据程序逻辑，一个已经获得了锁的线程缺少某些外部条件，而无法继续进行下去（例如生产者发现队列已满或者消费者发现队列为空），那么该线程可以通过调用 wait 方法将锁释放，进入 wait set 中阻塞进行等待，其它线程在这个时候有机会获得锁，去干其它的事情，从而使得之前不成立的外部条件成立，这样先前被阻塞的线程就可以重新进入 EntrySet 去竞争锁。这个外部条件在 monitor 机制中称为条件变量

**若 Synchronized 修饰的方法为非静态方法，表示此方法对应的对象为 锁对象;**

**若 Synchronized 修饰的方法为静态方法，则表示此方法对应的类对象 为锁对象。**

**注意，当一个对象被锁住时，对象里面所有用 Synchronized 修饰的 方法都将产生堵塞，而对象里非 Synchronized 修饰的方法可正常被 调用，不受锁影响**

[重要例子](https://blog.csdn.net/wbybyb/article/details/83989121)



synchronized支持重入锁，否则像下面一样：线程执行method2获得锁，执行method1又要获得锁，不支持重入，会自己锁死自己

```java
public class DeadLock{
    public synchronized void method1() {}
    public synchronized void method2() {
        this.method1();
    }
    public static void main(String[] args) {
        DeadLock deadLock = new DeadLock();
        deadLock.method2();
    }
}
```

Java 层面的线程与操作系统的原生线程有映射关系，如果要将一 个线程进行阻塞或唤起都需要操作系统的协助，这就需要从用户态切换 到内核态来执行，这种切换代价十分昂贵，很耗处理器时间，现代 JDK 中做了大量的优化： 

自旋锁，偏向锁（无竞争），轻量锁，重量锁

这三种锁使得 JDK 得以优化 Synchronized 的运行，当 **JVM 检测 到不同的竞争状况时，会自动切换到适合的锁实现，这就是锁的升级、 降级**



#### 锁

1. 可重入锁，防止自锁
2. 自旋锁，自选等待，避免用户态切换到内核态，但是cpu占用大，锁的竞争激烈不适合自旋锁
3. 偏向锁：偏斜锁可以降低无竞争开销

#### 代理

动态代理利用反射机制， java.lang 或 java.lang.reflect

**运行时动态构建代理**、动态处理代理方法调用

AccessibleObject.setAccessible(boolean flag)。它的子类也大都重写了这个方法，这里的所谓 accessible 可以理解成修饰成员的 public、protected、private，这意味着我们可以在运行时修改成员访问限制

```java
public static void main(String[] args) throws NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
 5         Student student = new Student();
 6         Field field = student.getClass().getDeclaredField("name");
 7         field.setAccessible(true);
 8         System.out.println(field);
 9         Object object = field.get(student);
10         System.out.println(object);
11     }
```

**是装饰器（Decorator）模式的应用**

JDK Proxy 的优势：

最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，可能比 cglib 更加可靠。

平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。

代码实现简单。

基于类似 cglib 框架的优势：

有的时候调用目标可能不便实现额外接口，从某种角度看，限定调用者实现接口是有些侵入性的实践，类似 cglib 动态代理就没有这种限制。只操作我们关心的类，而不必为其他相关类增加工作量。

高性能。

#### 基础数据类型与包装类

成员变量“value”，你会发现，不管是 Integer 还 Boolean 等，都被声明为**“private final”，所以，它们同样是不可变类型！**

new；或者静态工厂方法 valueOf，在调用它的时候会利用一个缓存机制，带来了明显的性能改进

Boolean，缓存了 true/false 对应实例，确切说，只会返回两个常量实例 Boolean.TRUE/FALSE。Short，同样是缓存了 -128 到 127 之间的数值。

Byte，数值有限，所以全部都被缓存。

Character，缓存范围’\u0000’ 到 ‘\u007F’。



自动装箱 / 自动拆箱是发生在什么阶段？  **编译**

自动装箱的时候，缓存机制起作用吗？ 起作用

##### 线程安全计数器

```
// 不使用private final AtomicLong counter = new AtomicLong(); 是为了减少开销
//counter.incrementAndGet();
 class CompactCounter {
    private volatile long counter; //用基本数据类型减少开销
    private static final AtomicLongFieldUpdater<CompactCounter> updater = AtomicLongFieldUpdater.newUpdater(CompactCounter.class, "counter");
    public void increase() {
        updater.incrementAndGet(this);
    }
}
```

原始数据类型需要手动保证线程安全，float,double可能在更新到一半的时候出错

可以用Atom类，

#### 集合

Vector 内部是使用对象数组来保存数据，[可以根据需要自动的增加容量](https://www.cnblogs.com/wuxiang/p/5328241.html)，当数组已满时，会创建新的数组，并拷贝原有数组数据；当扩容因子大于0时，新数组长度为**原数组长度+扩容因子**，否子新数组长度为**原数组长度的2倍**；  如果指定的容量大于默认上述计算的则按指定的扩容。 最大容量和下面一样。

ArrayList**默认扩容1.5倍**（如果指定的容量大于原来1.5倍按指定的，还有一个上限容量INTMAX-8, -8是为了防止一些VM有额外的header字段）,  最大容量2^31-1-8（8是为了head里的内容Some VMs reserve some header words in an array）

如果指定初始容量为0，第五次开始才是1.5倍，之前按照规则都是+1 old+old>>1

默认初始容量是10

![image-20200718170022795](/Users/kangkang/Library/Application Support/typora-user-images/image-20200718170022795.png)

Collections.synchronizedCollection(Collection<E> c)
该方法可以将一个Collection包装为同步（线程安全）的List。但是通过Iterator、Spliterator或Stream遍历这个新List时，需要在外部做好同步

##### 排序

原始数据类型，[双轴快速排序](http://hg.openjdk.java.net/jdk/jdk/file/26ac622a4cab/src/java.base/share/classes/java/util/DualPivotQuicksort.java)（Dual-Pivot QuickSort），

对象数据类型，[TimSort](http://hg.openjdk.java.net/jdk/jdk/file/26ac622a4cab/src/java.base/share/classes/java/util/TimSort.java), 本质是归并和二分插入

并行排序算法，底层实现基于 fork-join 框架，大数据时效果明显



java1.9有变长参数`List<String> simpleList = List.of("Hello","world");`

##### 哈希

~~Hashtable 是早期 Java 类库提供的一个哈希表实现，本身是同步的，不支持 null 键和值，由于同步导致的性能开销，所以已经很少被推荐使用~~

Hashmap不是同步的，支持 null 键和值等。初始数组大小16，负载因子0.75，每次扩容2倍，即2的幂，原因在于计算数组索引时用hash&(length-1)操作①效率高②长度都是偶数，-1位奇数，地位全是1，结果取决于h，分布更均匀，如果length-1是偶数，那么&完后最低位永远为0[,结果总是偶数](https://www.jianshu.com/p/dde9b12343c1)

最大容量为2^30,因为再扩容就超了

TreeMap 则是基于红黑树的一种提供顺序访问的 Map，和 HashMap 不同，它的 get、put、remove 之类操作都是 O（log(n)）的时间复杂度，具体顺序可以由指定的 Comparator 来决定，或者根据键的自然顺序来判断

##### 线程安全容器

![image-20200719004256620](/Users/kangkang/Library/Application Support/typora-user-images/image-20200719004256620.png)

早期的concurrentHashMap是分段锁

get volatile

put 调用unsafe获得相应segment，然后获取在入锁，重复扫描检测冲突。（扩容是在一个segment上扩容）

线程安全的类：Vector,StringBuffer,Properties,

#### Servlet

web.xml配置servlet；继承HttpServlet实现service，init，destory

在第一次访问servlet时实例化对象；单例多线程

##### 生命周期

Tomcat装载xml(不会实例化)，创建(构造函数)，初始化(servlet资源初始化)，提供服务(Service)，destory

#### JDBC

PreparedStatement是预编译的SQL语句，效率高于Statement
PreparedStatement支持?操作符，相对于Statement更加灵活
PreparedStatement可以防止SQL注入，安全性高于Statement

ps.setString（1，name）从1开始的

