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

 