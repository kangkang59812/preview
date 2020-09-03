1. 哪些对象的创建方法会调用构造函数：new操作，通过反射的newInstance**执行无参构造器**。通过readObject（相当于反序列化）和clone方法不会调用，是复制相应域

2. 接口不能有构造方法，抽象类可以有

   ​		都可以有实现方法（jdk1.8接口的default方法，具体的方法，非抽象，还可以被重写）因此也不必实现所有方法

   ​		 接口冲成员变量默认public static final, 抽象类中是friendly(包内任意访问，子类可重新定义和复制)

   ​		 都不能定义静态抽象方法,static不能与abstract连用

   ​		 接口可以有静态方法（1.8后），抽象类可以有静态方法		 

3. 继承抽象类的可以是普通类，但**必须重写抽象类中的所有抽象方法**，也可以是抽象类，**无需重写抽象类中的所有抽象方法**

4. 重载权限修饰符可以改变，只能从protected到public

5. 抽象类中的抽象方法(其前有 abstract修饰)不能用 private（不能继承）、 static（类初始化时）、 synchronized（this，类）、native(本地方法实现) 作为修饰符。 默认是abtract修饰的

6. object9大方法

   protected Object clone() 创建并返回此对象的一个副本。 
   boolean equals(Object obj) 指示某个其他对象是否与此对象“相等”。 
   protected void finalize() 当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。 
   Class<? extendsObject> getClass() 返回一个对象的运行时类。 
   int hashCode() 返回该对象的哈希码值。 
   void notify() 唤醒在此对象监视器上等待的单个线程。 
   void notifyAll() 唤醒在此对象监视器上等待的所有线程。 
   String toString() 返回该对象的字符串表示。 
   void wait() 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。 
   void wait(long timeout) 导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。 
   void wait(long timeout, int nanos) 导致当前的线程等待，直到其他线程调用此对象的 notify()

7. 启动线程的3种方式：继承thread，实现runnable接口（thread传入lambda表达式也行），线程池

8. wait让出锁资源cpu资源，yield和sleep只让出cpu

9. float型在jvm中使用科学计数法表示的，只占8位，超出位数随机，

   ```java
   float a1=123456788f;
   float a2=a1+1;
   // a1==a2 为True，因为超出了8位
   ```

10. 类的执行顺序：***静态优先，父类优先，非静态代码块优于构造函数***；父类静态代码块，子类静态代码块，**父类非静态代码块，父类构造函数，子类非静态代码块，子类构造函数**

11. error是系统级别，不可预测；exception是程序级别，要处理；顶层是throwable。

12. equals相等，hashcode相等；equals不相等，hashcode有可能相等；hashcode相等与否，equals都不一定相等

13. main内起一个线程，整个程序一共有3个线程，main，垃圾回收，起的线程

14. 三种同步方式：synchronzied，wait/notify, lock

15. String a1=a2+"aa" 这是调用StringBulider生成一个新的String对象

16. 内存泄漏场景：静态集合类（太大），各种连接，监听器（全局的，使用对象太大），不合理的作用域

17. object.clone()是浅复制，继承Cloneable接口, 引用对象的地址不变；深复制，序列化方式实现的，复制对象，要继承Serializable接口

    ```java
    public Dancer deepClone() throws IOException, ClassNotFoundException {
            //序列化，将内存中的对象序列化为字节数组
      				
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(this);
    
            //反序列化，将字节数组转回到对象，同时完成深复制的任务
            ByteArrayInputStream bis = new 	ByteArrayInputStream(bos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bis);
            return (Dancer)ois.readObject();
        }
    ```

18. [异常](https://blog.csdn.net/Jin_Kwok/article/details/79866465)， classnotfound : 加载类时找不到的异常，可捕获，class.forname, classloader.loadclass

       noclassdeffounderror：jvm错误，不可捕获，	编译完环境发生变化jar包变化，new对象时找不到类
  
19. JDK动态代理的invoke方法的第一个参数是什么：代理类的一个实例。代理后的对象

      class=class jdkproxy.$Proxy0 , finall类型
      
      superClass=class java.lang.reflect.Proxy
      
      interfaces: interface jdkproxy.Hello， 也就是说和原本的Myhello实现了同一个接口
      
      invocationHandler=jdkproxy.LogInvocationHandler@a09ee92
      
      cglib代理，不能代理final类型的方法
      class=class cglib.HelloConcrete\$\$EnhancerByCGLIB$$e3734e52
      
      superClass=class lh.HelloConcrete
      
      interfaces: 
      interface net.sf.cglib.proxy.Factory
      
      invocationHandler=not java proxy class
      
20. [buffer.flip()](https://blog.csdn.net/ctwy291314/article/details/82114572?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight)的作用buffer写完数据后要调用，保证读的时候的limit是实际长度

18. [异常](https://blog.csdn.net/Jin_Kwok/article/details/79866465)， classnotfound : 加载类时找不到的异常，可捕获，class.forname, classloader.loadclass

    noclassdeffounderror：jvm错误，不可捕获，	编译完环境发生变化jar包变化，new对象时找不到类

22. [单例模式为什么要用Volatile关键字](https://blog.csdn.net/qq_34412985/article/details/89969004) 双重检查，null  syn null new; volatile禁止指令重排序，防止只用一个未经初始化的对象

    <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyb6n57ymj312i0lm178.jpg" alt="image-20200821121422510" style="zoom:33%;" />

23. 类加载过程: 加载，连接（验证，准备，解析），初始化，使用，卸载

    加载：通过类名获取二进制流，将静态存储结构转化为jvm运行时结构，在堆中生成类对象，作为方法区入口

    验证：.class字节流格式，元数据，字节码，符号引用（可以关闭）

    准备：类变量分配内存，初始化为类型的默认值。如果被final修饰，会被直接初始化为设定的值

    解析：符号引用变为直接引用

    初始化：静态变量真正赋值。只有对类真正使用时才会初始化：调用静态变量、方法，new，创建子类，反射，启动类

24. 类加载器：bootstrap加载JRE_HOME/lib顶层核心类（C语言实现）；Extention ClassLoader加载JRE_HOME/lib/ext下扩展的类；Appclass Loader加载classpath下类；User ClassLoader用户自定义

    类加载器之间不能访问，类对象一般是单例的

25. 类的加载方式：jvm加载包含main的方法主类；Class.forName()方法动态加载，会默认执行初始化块（static{}）；Class.forName(name,initialize,loader)中的initialze可指定是否要执行初始化块；ClassLoader.loadClass()方法动态加载，不会执行初始化块（类还没被连接）。 jdbc只能用第一个，因为有静态代码块把自己注册到java.sql.DriverManager

26. 双亲委派：优先父类加载，避免了重复加载，避免自定义加载器加载核心类

27. [自定义类加载器](https://blog.csdn.net/qq_30242987/article/details/88571383)：继承ClassLoader，重写findClass()方法（双亲委派），defineClass(name, classData, 0, classData.length); 类名，字节数据通过文件转为byte数组

    继承ClassLoader,重写loadClass()方法（非双亲委派）

28. GC roots：虚拟机栈（帧栈中的本地变量表）中引用的对象。方法区中静态属性引用的对象。方法区中常量引用的对象。本地方法栈中 JNI 引用的对象

29. 垃圾回收方法：标记清理：直接清理标记为垃圾的对象；标记整理：将标记对象移到内存一段，清空另一端；复制：将存活对象移到另一半内存，清空这一半

30. 分代回收：

    方法区回收：废弃常量没有引用，垃圾回收；无用类（没有实例，classloader收回，类对象没有引用，没有反射访问的）

    堆回收：分代

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyfxdz29ij30yg0u0458.jpg" alt="image-20200821145829242" style="zoom:33%;" />

31. <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyg5uirg0j316q0syx5e.jpg" alt="image-20200821150639442" style="zoom: 33%;" />

32. CMS:  初始标记（会产生全局停顿）标记根可以直接关联到的对象速度快

    ​		   并发标记（和用户线程一起）主要标记过程，标记全部对象

    ​           重新标记 （会产生全局停顿   由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正

    ​           并发清除（和用户线程一起）基于标记结果，直接清理对象

33. [G1](https://www.cnblogs.com/yufengzhang/p/10571081.html)：新生代回收；老年代并发标记（堆使用达到45%）；混合回收

    新生代回收是STOP-The-World

34. [哈希冲突](https://blog.csdn.net/porsche_gt3rs/article/details/79445707)：再哈希，开放地址法(有冲突往后跳)，链地址(hashmap)，公共溢出区

35. hashmap扩容；concurrenthashmap扩容：jdk1.8是，forwardNode实例fwd，在此期间如果其他线程的有读写操作都会判断head节点是否为forwardNode节点，如果是就帮助扩容；对于get读操作，如果当前节点有数据，还没迁移完成，此时不影响读，能够正常进行。 如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时get线程会帮助扩容；对于put/remove写操作，如果当前链表已经迁移完成，那么头节点会被设置成fwd节点，此时写线程会帮助扩容，如果扩容没有完成，当前链表的头节点会被锁住，所以写线程会被阻塞，直到扩容完成

36. 线程状态

    <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyhpod777j319t0u04iv.jpg" alt="image-20200821160019272" style="zoom: 50%;" />
=======
18. 异常](https://blog.csdn.net/Jin_Kwok/article/details/79866465)， classnotfound : 加载类时找不到的异常，可捕获，class.forname, classloader.loadclass

    noclassdeffounderror：jvm错误，不可捕获，	编译完环境发生变化jar包变化，new对象时找不到类

19. [单例模式为什么要用Volatile关键字](https://blog.csdn.net/qq_34412985/article/details/89969004) 双重检查，null  syn null new; volatile禁止指令重排序，防止只用一个未经初始化的对象
37. LongAdder和AtomicLong：高并发情况下后者cas失败率高，效率低；后者是把value拆分到多个value存放到cell，分段更新，取值时累加cells。低并发时casbase相当于cas, 高并发时才会走cell

38. 多线程读不一定加锁，主要看业务是否允许读到脏数据。

39. synchronized方法异常时，如果不catch，锁会被释放，其它线程会争夺此资源

40. 执行时间短（加锁代码），线程数少，用自旋；执行时间长，线程数多，用系统锁

41. synchronized不能锁String,Integer等对象，因为jar包里可能引用了相同的对象。导致出错。也不能锁null，会报空指针异常。

42. [eden的大对象什么时候进入old区](https://www.jianshu.com/p/f7cde625d849)：- XX:PretenureSizeThreshold，默认0，不管多大现在eden中分配

43. volatile保证线程可见性，禁止指令重排序。每个线程都有变量的副本，如果变量在线程a中修改了，那么线程b读写的还是自己的副本，无法控制线程什么时候去读变量本身。**在线程a中加sleep后，可重新从原变量取值**。

    保证线程可见性：靠cpu的缓存一致性协议，MESI

    JMM模型里有8个指令完成读写操作，通过其中load和store指令相互组合成的4个内存屏障实现禁止指令重排序。

44. 使用tryLock进行尝试锁定，**不管锁定与否，方法都将继续执**行；可以根据**tryLock的返回值来判定是否锁定**。

45. lockInterruptibly可在别的线程中调用当前线程的interrupt()方法打断当前线程的等待，进入到当前线程的catch中去

46. [cyclicbarrier](https://www.jianshu.com/p/333fd8faa56e)所有线程达到await()时才会继续执行，可复用。第一个参数是线程数n，第二个是最后一个线程到达后要做的任务。参与的线程数要是小于参数设置的，会一直阻塞；大于会 对n取余的线程数会阻塞，n的整数倍线程数会通过 。场景：合并计算结果。

    phaser是多级栅栏

47. 读写锁：readWriteLock.readLock() readWriteLock.writeLock()。写锁是排它锁，读锁是共享锁。一个线程读，其他线程也可以读。

48. new Semaphore(2, true)可以设置是否为公平锁。acquire时获取不到是阻塞的。可以用来限流。

49. ```
    static Exchanger<String> exchanger = new Exchanger<>();
    不同线程调用s = exchanger.exchange(s);会阻塞
    是两个线程间通信的方式之一，当线程都把值放进exchanger后，才会唤醒
    游戏中交易装备
    ```

50. try-catch-finally

    如果try中没有异常，则顺序为try→finally，

    如果try中有异常，则顺序为try→catch→finally

    try中有return，无异常时，会执行完finally后再return，返回的是try中保存的信息（引用类型会被finally中代码改变）

    catch中带有return，同理

    finally中有return时（不建议写），try和catch中的return会失效