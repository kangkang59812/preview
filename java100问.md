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

8. sleep让出锁资源cpu资源，yield和sleep只让出cpu

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
    1. [异常](https://blog.csdn.net/Jin_Kwok/article/details/79866465)， classnotfound : 加载类时找不到的异常，可捕获，class.forname, classloader.loadclass

       noclassdeffounderror：jvm错误，不可捕获，	编译完环境发生变化jar包变化，new对象时找不到类

