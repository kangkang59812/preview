#### 开发流程

不需要：tomcat生成war包，上传服务器

配置环境-spring initializer-配置参数（可选）-业务开发-自动构建-自动部署-运维监控

**生成的是jar包**

#### 特性

独立运行的spring项目

极简组件依赖，自动发现与装备

运行时的应用监控

分布式与云计算集成

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnapy97jmj30xo0im7fe.jpg" alt="image-20200616150202257" style="zoom: 33%;" />

#### 入口类：

入口类命名通常以*Application结尾。
入口类上增加@SpringBootApplication注解
利用SpringApplication.run()方法启动应用

##### 启动流程

①加载配置文件：application.properties②自动装配：spring-boot-starter-web(test,logging等)③加载组件：repository service controler component entity④初始化：tomcat，数据源，连接池等

#### 配置文件

.properties或者.yml，同时存在时application.properties的优先级会比application.yml高

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghnaqe7fkfj30c80ghjs0.jpg" alt="img" style="zoom: 50%;" />

通过对属性@Value("${mydifination.key1.key2.value}")得到配置文件中的值

针对开发环境和线上环境，可以启用不同的配置文件, 命名要符合application-×××.yml, 在active中启用×××

```yml
spring:
 profiles:
  active:dev
```

#### 打包

利用Maven的package生成独立运行的jar包
利用 java -jar xxx.jar 启动Spring Boot应用
Jar包可自动加载同目录的application配置文件 

生成target文件夹后，可以将yml，properties都复制过去，会优先读取这些配置文件

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghoggyzo2pj315k0mogyh.jpg" alt="image-20200812234138274" style="zoom: 33%;" />

