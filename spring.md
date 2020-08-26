#### IOC和DI

IOC是一种理念，DI是实现，spring主要用工厂模式和放射来实现

#### Bean实例化方式

类构造器；静态工厂；实例工程

#### bean的作用域

https://www.cnblogs.com/amunamuna/p/10959796.html

#### 获取bean的方式

1. 通过BeanFactory（在getBean时候创建类实例）或者ApplicationContext（在加载配置文件的时候创建单例模式的bean）
   区别：前者不支持AOP，web等功能，是懒汉模式；后者饿汉模式，支持统一资源访问，国际化消息，事件传播
2. 注解，在类上写上@Component通用型的@Service@Controller@Respository（Dao类）之一，参数填bean名字

#### bean的生命周期

粗分 

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

<img src="https://upload-images.jianshu.io/upload_images/4558491-dc3eebbd1d6c65f4.png" alt="img" style="zoom:50%;" />

postProcessBeforeInstantiation：替换原本的bean，AOP实现的关键

postProcessAfterInstantiation：可以看到该方法在属性赋值方法内，但是在真正执行赋值操作之前。其返回值为boolean，返回false时可以阻断属性赋值阶段

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gi13vpg1nqj30vi0pyb29.jpg" alt="image-20200823221823468" style="zoom: 50%;" />

#### **获得bean实例**

1. 通过配置文件构建工厂，在通过工厂的getbean方法获得实例，关闭工厂（可自定义工厂）

   ![image-20200609170347881](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqpqtrnj31e204kq5i.jpg)

2. beand的作用域

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqre3ejj30df0530un.jpg" alt="image-20200609170449254" style="zoom: 67%;" />

#### 属性注入

1. 设值注入（通过setter方法反射注入的）
2. 构造注入，需要构造函数\<constructor-arg  name=或者index=0,1..  value=\>，存在对象间依赖关系
3. p命名空间注入，需先引入, p:name=  p:car-ref=
4. spel  <property name="price" value=# {productInfo.calculatePrice ()}, 可以调用函数注入，中括号里可以填“xiaoming”或者引用对象car（不加引号）
5. javaConfig，通过Configuration注解， @bean方法return new



#### 注解

`<context:component-scan base-package="com.imooc.ioc.review"/>`是开启包扫描

`<context:annotation"/>`只能使用属性注解

#### AOP

AOP采取横向抽取机制（代理机制）,取代了传统纵向继承体系重复性代码(性能监视、事务管理、安全检查、缓存)

Spring AOP使用纯Java实现,不需要专门的编译过程和类加载器,在**运行期通过代理方式向目标类织入增强代码**

**jointpoint**是被拦截到的点

**pointcut**拦截的定义

**advise**通知/z增强



对于实现接口的类，可以用JDK动态代理，实现InvocationHandler的invoke方法,是针对接口生成子类，接口中不能有final方法，否则不起作用

没实现接口的类，用cglib代理（采用字节码，为一个类创建子类）



spring中 **advisor代表切面对目标类所有方法增强**；**pointcutAdvisor指定拦截那些目标方法**；环绕通知是aopalliance包的



##### 手动设置代理对象

ProxyFactoryBean的属性配置：

target代理目标对象，

proxyInterfaces代理要实现的接口（多个用list），

proxyTargetClass:true时使用cglib对类代理，否则是jdk代理，

interceptorNames需要织入目标的advice

singleton是否返回单例模式代理

optimize true时强制使用cglib代理

**对目标类所有方法都产生了代理**,**测试是用代理类的bean**

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqukv9aj31eo0ew7af.jpg" alt="image-20200609214835156" style="zoom:67%;" />

使用带有切点的切面，常用DefaultPointAdvisior

![image-20200609214954053](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqvs4ebj310o07d79x.jpg)

常用DefaultPointAdvisior,可**实现任意切点和通知的组合**，在工厂里可添加拦截列表

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqxpdthj31hc0piai8.jpg" alt="image-20200610135709983" style="zoom: 33%;" />

##### 自动代理

用的原类的bean

**BeanNameAutoProxyCreator**根据bean名称创建代理

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnar34y5xj30q20ciwwd.jpg" alt="image-20200610141457744" style="zoom:50%;" />

**DefaultAdvisorAutoProxyCreator**根据Advisor本身包含信息创建代理

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnar2o7jdj30rg0baka7.jpg" alt="image-20200610142723793" style="zoom: 50%;" />

**AnnotationAwareAspectJAutoProxyCreator** 基于Bean中的AspectJ注解自动代理

##### AspectJ

![image-20200610143235669](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnar7ofqaj30ae00sglq.jpg)

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnar5glvaj30qo0cwn01.jpg" alt="image-20200610144428923" style="zoom: 33%;" />

最上面两个注解都写，或者第二个在xml里注册

![image-20200610144527495](/Users/kangkang/Library/Application Support/typora-user-images/image-20200610144527495.png)

最终通知@After：无论是否有异常都会通知

后置通知@Afterrunning(value="excution(..)",returning="result")：可以获得函数返回值

单独写切点

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnardogljj3192056t9z.jpg" alt="image-20200610145348377" style="zoom: 33%;" />

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnarfoh71j30ow04k3zv.jpg" alt="image-20200610145402188" style="zoom:50%;" />

mypoint()可以有多个，多个切点可以用||隔开



xml方式，多个切点是在execution中||(or)  ！ &&(会报错)

![image-20200610151017507](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnari33kqj31oa07wacu.jpg)

after里也可以写 pointcut



如果不用aspect用advisor，需要实现org.aopalliance.intercept.MethodInvocation(环绕，或者before等)接口的invoke方法。  aspect相当于把所有通知聚集在一起，然后选；advisor是单独的一个通知



##### Spring AOP 和 AspectJ AOP 有什么区别?

Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。 Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。


#### 数据库

jdbc template-->spring template

jdbc事务-->spring编程式事务-->sprin声明式事务（aop）

Spring的声明式事务处理是建立在AOP的基础之上的。其本质是对方法前后进行拦截,然后在目标方法开始之前创建或者加入一个事务,在执行完目标方法之后根据执行情况提交或者回滚事务。会产生代理类。不需要改动业务。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnarmq357j30um07ek7d.jpg" alt="image-20200610170309927" style="zoom:50%;" />

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnarpafbrj30s005s11n.jpg" alt="image-20200610170317429" style="zoom:50%;" />

或者合并

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnartjvlhj313c0ayqts.jpg" alt="image-20200610170448793" style="zoom:50%;" />

基于tx命名空间的声明式事务管理

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnarvjyd1j313c0ayqts.jpg" alt="image-20200610170645481" style="zoom: 50%;" />

或者用注解，不需要配置advice和aop，参数比较多

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnarz94kmj30jq0aedvk.jpg" alt="image-20200610172708061" style="zoom:50%;" />

#### 设计模式

1. 控制翻转依赖注入

2. 工厂设计模式

   1. `BeanFactory` ：延迟注入(使用到某个 bean 的时候才会注入),相比于`BeanFactory` 来说会占用更少的内存，程序启动速度更快。
   2. `ApplicationContext` ：容器启动的时候，不管你用没用到，一次性创建所有 bean 。BeanFactory 仅提供了最基本的依赖注入支持，ApplicationContext 扩展了 BeanFactory ,除了有BeanFactory的功能还有额外更多功能，所以一般开发人员使用ApplicationContext会更多

3. 单例模式：对于频繁使用的对象，可以省略创建对象所花费的时间；由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。**Spring 通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式**

4. 代理模式：AOP：jdk动态代理或者cglib代理

5. [模板模式](https://blog.csdn.net/qq_34337272/article/details/90487768)：定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤的实现方式。Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是 Spring 并没有使用这种方式，而是使用Callback 模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性

6. 观察者模式：

   Spring 的事件流程总结
   定义一个事件: 实现一个继承自 ApplicationEvent，并且写相应的构造函数；
   定义一个事件监听者：实现 ApplicationListener 接口，重写 onApplicationEvent() 方法；
   使用事件发布者发布消息: 可以通过 ApplicationEventPublisher 的 publishEvent() 方法发布消息。

7. 适配器模式：Spring预定义的通知要通过对应的适配器，适配成 `MethodInterceptor`接口(方法拦截器)类型的对象（如：`MethodBeforeAdviceInterceptor` 负责适配 `MethodBeforeAdvice`）

   在Spring MVC中，DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由HandlerAdapter 适配器处理。HandlerAdapter 作为期望接口，具体的适配器实现类用于对目标类进行适配，Controller 作为需要适配的类。

8. 装饰者模式：装饰者模式可以动态地给对象添加一些额外的属性或行为

9. 策略模式解决的问题：同一类型业务的类，有很多公用的流程和方法，只是在核心方法上略有区别，为了降低代码的冗余度，单独把不同的方法抽象成一个接口，各自业务类实现自己的核心方法

10. 责任链模式解决的问题：把一套流程拆分成不同的Handler，使用的时候根据业务场景拼装，可以非常灵活和低耦合的实现特定的业务流程。

11. [事务传播行为](https://blog.csdn.net/weixin_39625809/article/details/80707695)

    <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyhhdo1mxj312s0n6gy7.jpg" alt="image-20200821155220848" style="zoom: 50%;" />

    [例](https://www.jb51.net/article/167169.htm)

    ```java
    @Transactional(propagation = Propagation.REQUIRED)
    function A(){ // 有事务
     insert one data
     functionB(); // 代码正常
     functionC(); // 代码抛异常
    }
    1. C加入A的事务，失败，A中全部回滚
    2. C失败不影响B的实行结果
    3. A没事务，会抛出异常
    4. 仅有B执行成功， A会回滚
    5. BC都会执行，A不会因为C异常回滚
    6. A有事务执行BC，则抛出异常
    7. 事务嵌套运行
    ```

    

#### 循环依赖

同样对于循环依赖的场景，**构造器注入**和**prototype类型的属性注入**都会初始化Bean**失败**。因为@Service默认是单例的，所以**单例的属性注入是可以成功的**

```java
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

/**
bean 的获取过程：先从一级获取，失败再从二级、三级里面获取

创建中状态：是指对象已经 new 出来了但是所有的属性均为 null 等待被 init
*/
```

- A 创建过程中需要 B，于是 **A 将自己放到三级缓里面** ，去实例化 B

- B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了！

- - **然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A**
  - B 顺利初始化完毕，**将自己放到一级缓存里面**（此时B里面的A依然是创建中状态）

- 然后回来接着创建 A，此时 B 已经创建结束，直接从一级缓存里面拿到 B ，然后完成创建，**并将自己放到一级缓存里面**

- 如此一来便解决了循环依赖的问题

[为什么是三级不是二级](https://my.oschina.net/u/4340310/blog/4332450)  二级不能满足AOP，不然注入的是原始对象，不是代理对象

