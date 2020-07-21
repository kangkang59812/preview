#### 开发流程

![image-20200608202711712](/Users/kangkang/Library/Application Support/typora-user-images/image-20200608202711712.png)

##### 配置mybatis-config.xml文件，严格遵守顺序

```xml
1.properties属性
2.settings设置
3.typeAliases类型命名
4.typeHandlers类型处理器
5.objectFactory对象厂
6.plugins插件
7.environments 环境(可配置多个环境，选一个默认的)
		environment环境变量
			transactionManager 事务管理器
			dataSource数据源
8.databaseldProvider 数据库厂商标识
9.mappers映射器
```



```xml
<properties>
  <!--这里定义变量，后面可以引用-->
  <property name="driver" value="com.mysql.jdbc.Driver"></property>
</properties>

<settings>
<!--为了entity中的名字和mysql的字段名匹配（前者oneSecnd,后者one_second）-->
<setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>

<!-- 环境，可以配置多个，default：指定采用哪个环境 -->
<environments default="test">
	<environment id="test">
    <!-- 事务管理器，JDBC类型的事务管理器 -->
    <transactionManager type="JDBC" />
    <!-- 数据源，池类型的数据源 -->
    <dataSource type="POOLED">
      <property name="driver" value="${driver}" />
      <property name="url" value="${url}" />
      <property name="username" value="${username}" />
      <property name="password" value="${password}" />
    </dataSource>
  </environment>
</environments>

<mappers>
  <mapper resource="mappers/MyMappers.xml" />
  <mapper resource="mappers/UserDaoMapper.xml"/>
  <mapper resource="mappers/UserMapper.xml"/>
</mappers>
```

##### 两种实现方式

方式一：实现Dao接口，在映射xml中`<mapper namespace="UserMapper">`，实现时要传入`SqlSession sqlSession`，执行时`this.sqlSession.selectOne("UserDao.queryUserById", id);`，第一个参数是命名空间+“.”+statementId, **namespace可以随便取，调用的时候一致就行**

方式二：不用实现Dao接口，使用mybatis的动态代理，在映射xml中`<mapper namespace="com.mybatis.dao.UserMapper">`接口全路径，测试中`this.userDao=sqlSession.getMapper(UserDao.class);`

总结

使用mapper接口不用写接口实现类即可完成数据库操作，使用非常简单，也是官方所推荐的使用方法。
使用mapper接口的必须具备以几个条件:
1) Mapper的namespace必 须和mappe**r接口的全路径一致**。
2) Mapper接口的**方法名必须和sq|定义的id一 致**。
3) Mapper接 口中方法的**输入参数类型**必须和sql定义的**parameterType不一定一致**
4) Mapper接口中方法的**输出参数类型必须和sql定义的resultType一致**。

##### 映射xml文件 

```xml
<mapper namespace="com.mybatis.dao.UserMapper">
    <!--
       1.#{},预编译的方式preparedstatement，使用占位符替换，防止sql注入，一个参数的时候，任意参数名可以接收
       2.${},普通的Statement，字符串直接拼接，不可以防止sql注入，一个参数的时候，必须使用${value}接收参数
     -->
  <!--  statement id要和Dao接口中的方法名相同 -->
  <!--  resultType是方法返回类型 -->
    <select id="queryUserByTableName" resultType="com.mybatis.pojo.User">
        select * from ${tableName}
    </select>

    <select id="login" resultType="com.mybatis.pojo.User">
        select * from tb_user where user_name = #{userName} and password = #{password}
    </select>

    <!-- statement，内容：sql语句。
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       resultType：sql语句查询结果集的封装类型，使用动态代理之后和方法的返回类型一致；resultMap：二选一
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <select id="queryUserById" resultType="com.mybatis.pojo.User">
        select * from tb_user where id = #{id}
    </select>
    <select id="queryUserAll" resultType="com.mybatis.pojo.User">
        select * from tb_user
    </select>
    <!-- 新增的Statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
       useGeneratedKeys:开启主键回写
       keyColumn：指定数据库的主键
       keyProperty：主键对应的pojo属性名
     -->
    <insert id="insertUser" useGeneratedKeys="true" keyColumn="id" keyProperty="id"
            parameterType="com.mybatis.pojo.User">
        INSERT INTO tb_user (
        id,
        user_name,
        password,
        name,
        age,
        sex,
        birthday,
        created,
        updated
        )
        VALUES
        (
        #{id},
        #{userName},
        #{password},
        #{name},
        #{age},
        #{sex},
        #{birthday},
        NOW(),
        NOW()
        );
    </insert>
    <!--
       更新的statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <update id="updateUser" parameterType="com.mybatis.pojo.User">
        UPDATE tb_user
        <trim prefix="set" suffixOverrides=",">
            <if test="userName!=null">user_name = #{userName},</if>
            <if test="password!=null">password = #{password},</if>
            <if test="name!=null">name = #{name},</if>
            <if test="age!=null">age = #{age},</if>
            <if test="sex!=null">sex = #{sex},</if>
            <if test="birthday!=null">birthday = #{birthday},</if>
            updated = now(),
        </trim>
        WHERE
        (id = #{id});
    </update>
    <!--
       删除的statement
       id：唯一标识，随便写，在同一个命名空间下保持唯一，使用动态代理之后要求和方法名保持一致
       parameterType：参数的类型，使用动态代理之后和方法的参数类型一致
     -->
    <delete id="deleteUserById" parameterType="java.lang.String">
        delete from tb_user where id=#{id}
    </delete>
</mapper>
```

##### 测试：

```java
// 指定配置文件
String resource = "mybatis-config.xml";
// 读取配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);
// 构建sqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
// 获取sqlSession
SqlSession sqlSession = sqlSessionFactory.openSession(true); //自动commit

// 1. 映射文件的命名空间（namespace）必须是mapper接口的全路径
// 2. 映射文件的statement的id必须和mapper接口的方法名保持一致
// 3. Statement的resultType必须和mapper接口方法的返回类型一致
// 4. statement的parameterType必须和mapper接口方法的参数类型一致（不一定）
this.userMapper = sqlSession.getMapper(UserMapper.class); //使用接口

this.userDao=new UserDaoImpl(sqlSession); //用实现类的对象
```

#### 数据库中的字段名和entity中的属性名不同：

```java
//1. 在映射文件xml里，用别名查询，别名和entity里的一样
<!--使用别名-->
    <select id="queryUserById" resultType="com.zpc.mybatis.pojo.User">
      select
       tuser.id as id,
       tuser.user_name as userName,  //数据库里是user_name，entity里是userName
       tuser.password as password,
       tuser.name as name,
       tuser.age as age,
       tuser.birthday as birthday,
       tuser.sex as sex,
       tuser.created as created,
       tuser.updated as updated
       from
       tb_user tuser
       where tuser.id = #{id};
   </select>
//2. 在settings里设置，自动在user_name和userName这样两种形式转换
  <setting name="mapUnderscoreToCamelCase" value="true"/>
//3. ResultMap,可以解决：名字不匹配；高级查询一对一，一对多，多对多
```

#### properties属性

```xml
<!-- 可以从config.properties读取url和driver这样的信息，灵活-->
<properties resource="org/mybatis/example/config.properties" url="...">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
<!--
如果属性在不只一个地方进行了配置，那么 MyBatis 将按照下面的顺序来加载：
1）在 properties 元素体内指定的属性首先被读取。
2）然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
3）最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。
SqlSessionFactory build(InputStream inputStream, String environment, Properties properties)
SqlSessionFactory build(Configuration config)

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性。-->

```

#### settings

```xml
<setting name="mapUnderscoreToCamelCase" value="true"/> 自动驼峰命名规则，默认False
<setting name="cacheEnabled" value="true"/> 影响所有映射器中配置的缓存的全局开关，默认True
<setting name="lazyLoadingEnabled" value="true"/> 延迟加载的全局开关，默认False(fetchType可覆盖)
用到子对象的方法或属性时才进行查询

<setting name="aggressiveLazyLoading" value="true"/> 
启用时，当延迟加载开启时访问对象中一个懒对象属性时，将完全加载这个对象的所有懒对象属性。false， 当延迟加
载时，按需加载对象属性(即访问对象中一个懒对象属性，不会加载对象中其他的懒对象属性)。默认为true

<setting name="useGeneratedKeys" value="true"/> 允许JDBC自动生成主键，默认False

延迟加载的意义在于，虽然是关联查询，但不是及时将关联的数据查询出
来，而且在需要的时候进行查询。
```

#### typeAliases

类型别名是为 Java 类型命名的一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余

```xml
<typeAliases>
    <typeAlias type="com.mybatis.pojo.User" alias="User"/>
    <!--或者开启包扫描,别名就是类名，有许多内置类名Integer等用在resultType那里-->
    <package name="com.mybatis.pojo"/>
</typeAliases>
```

#### typeHandlers

无论是 MyBatis 在预处理语句（PreparedStatement）中**设置一个参数时**，还是**从结果集中取出一个值时**， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。可以**重写类型处理器**或创建你自己的类型处理器来处理不支持的或非标准的类型

#### plugins

允许在已映射语句执行过程中的某一点进行拦截调用

```java
Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)
ParameterHandler (getParameterObject, setParameters)
ResultSetHandler (handleResultSets, handleOutputParameters)
StatementHandler (prepare, parameterize, batch, update, query)
```

#### environments

尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一。更多的是选择使用spring来管理数据源，来做到环境的分离

#### mappers

需要告诉 MyBatis 到哪里去找到 SQL 映射语句,resources是根目录, `/mymappers/User.xml`

```xml
<mappers>
    <mapper resource="mappers/MyMapper.xml"/>
    <mapper resource="mappers/UserDaoMapper.xml"/>
    <!--注解方式可以使用如下配置方式-->
  <!--接口所在的包中定义mapper.xml，并且要求xml文件和interface的名称要相同
当然也可以使用包扫描（必须使用注解方式，即在接口方法上使用注解，如@Select("select * from tb_user ")）-->
    <mapper class="com.mybatis.dao.UserMapper"/>
</mappers>
```

#### Mapper XML文件详解

##### select

id属性：当前名称空间下的statement的唯一标识。必须。要求id和mapper接口中的方法的名字一致。
resultType：将结果集映射为java的对象类型。必须（和 resultMap 二选一）
parameterType：传入参数类型。

大于号，小于号，& ，‘ ，“需要用字符实体\&lt;等

或者用\<![CDATA[ >= ]]> 包裹  >=

##### insert

id：唯一标识，随便写，在同一个命名空间下保持唯一，**使用动态代理之后要求和方法名保持一致**
parameterType：参数的类型，**使用动态代理之后和方法的参数类型一致**
useGeneratedKeys:开启主键回写？？
keyColumn：指定数据库的主键
keyProperty：主键对应的pojo属性名

##### update

##### delete

##### #{}和${}

\#{}相当于是到sql语句中带“”的，${}是不带的。${}一般用于传输数据库的表名、字段名等，传参数用#{}防止sql注入。

对在实现接口的类，用xml映射，sql语句如果有表名，用@param没用，得用\${_parameter}或者\${value},

多参数总受报错，未解决，暂时都用接口，不要用实现类。

##### ResultMap

关联嵌套https://www.cnblogs.com/kenhome/p/7764398.html

```xml
<resultMap id="唯一的标识" type="映射的pojo对象" extends="继承的resultMap的id">
  <id column="表的主键字段，或者可以为查询语句中的别名字段" jdbcType="字段类型" property="映射pojo对象的主键属性" />
  <result column="表的一个字段（可以为任意表的一个字段）" jdbcType="字段类型" property="映射到pojo对象的一个属性（须为type定义的pojo对象中的一个属性）"/>
  
  完成子对象的映射
  <association property="pojo的一个对象属性" javaType="pojo关联的pojo对象">
    <id column="关联pojo对象对应表的主键字段" jdbcType="字段类型" property="关联pojo对象的主席属性"/>
    <result  column="任意表的字段" jdbcType="字段类型" property="关联pojo对象的属性"/>
  </association>
  
  <!-- 集合中的property须为oftype定义的pojo对象的属性-->
  <collection property="pojo的集合属性" ofType="集合中的pojo对象">
    <id column="集合中pojo对象对应的表的主键字段" jdbcType="字段类型" property="集合中pojo对象的主键属性" />
    <result column="可以为任意表的字段" jdbcType="字段类型" property="集合中的pojo对象的属性" />  
  </collection>
  
  子查询
  <association property="" javaType="" select="子查询的id" column="子查询条件">
    <id column="关联pojo对象对应表的主键字段" jdbcType="字段类型" property="关联pojo对象的主席属性"/>
    <result  column="任意表的字段" jdbcType="字段类型" property="关联pojo对象的属性"/>
  </association>
</resultMap>
```

##### sql片段

```xml
<sql id=””></sql>
<include refId=”” />

<select id="queryUserById" resultMap="userResultMap">
	select <include refid="commonSql"></include> from tb_user where id = #{id}
</select>

<select id="queryUsersLikeUserName" resultType="User">
		select <include refid="CommonSQL.commonSql"></include> from tb_user where user_name like "%"#{userName}"%"
</select>

<!--也可以把sql定义在其他的mapper文件里
在mapper里引用它-->
<mappers>
		<mapper resource="CommonSQL.xml"/>
		<!-- 开启mapper接口的包扫描，基于class的配置方式 -->
		<package name="com.zpc.mybatis.mapper"/>
</mappers>
```

##### like

```xml
<!--用bind或者concat-->
<select id="queryUsersLikeUserName" resultType="User">
  	<bind name="pattern" value="'%' + keyword + '%'" />
		select <include refid="CommonSQL.commonSql"></include> from tb_user where user_name like #{pattern}
</select>
<!--或者把#{pattern}换成concat("%",#{keyword},"%")
或者'%${name}%'-->
```

##### if

```xml
<select id="queryUserList" resultType="com.mybatis.pojo.User">
    select * from tb_user WHERE sex=1
    <if test="name!=null and name.trim()!=''">  name是传入的参数，方法里要@param()指定
      and name like '%${name}%'
    </if>
</select>
```

##### choose-when

```xml
<select id="queryUserListByNameOrAge" resultType="com.mybatis.pojo.User">
    select * from tb_user WHERE sex=1
    <!--
    1.一旦有条件成立的when，后续的when则不会执行
    2.当所有的when都不执行时,才会执行otherwise
    -->
    <choose>
        <when test="name!=null and name.trim()!=''">
            and name like '%${name}%'
        </when>
        <when test="age!=null">
            and age = #{age}
        </when>
        <otherwise>
            and name='鹏程'
        </otherwise>
    </choose>
</select>
```

##### where

```xml
<select id="queryUserListByNameAndAge" resultType="com.mybatis.pojo.User">
    select * from tb_user
    <!--如果多出一个and，会自动去除，如果缺少and或者多出多个and则会报错-->
    <where>
        <if test="name!=null and name.trim()!=''">
            and name like '%${name}%'
        </if>
        <if test="age!=null">
            and age = #{age}
        </if>
    </where>
</select>
```

set

```xml
<update id="updateUser" parameterType="com.mybatis.pojo.User">
    UPDATE tb_user
    <trim prefix="set" suffixOverrides=",">  去掉多余的,
        <if test="userName!=null">user_name = #{userName},</if>
        <if test="password!=null">password = #{password},</if>
        <if test="name!=null">name = #{name},</if>
        <if test="age!=null">age = #{age},</if>
        <if test="sex!=null">sex = #{sex},</if>
        <if test="birthday!=null">birthday = #{birthday},</if>
        updated = now(),
    </trim>
    WHERE
    (id = #{id});
</update>
```

##### for each

```xml
<select id="queryUserListByIds" resultType="com.zpc.mybatis.pojo.User">
    select * from tb_user where id in
    <foreach collection="ids" item="id" open="(" close=")" separator=",">
        #{id}
    </foreach>
</select>
输入是数组
```

#### 缓存

**一级缓存的作用域是session**，打开一个sqlsession后，执行相同的sql会从缓存中返回，**默认开启**

条件：1. 同一个session中 2. 相同的SQL和参数`sqlSession.clearCache();`可以强制清除缓存

**执行update、insert、delete的时候，会清空缓存**

二**级缓存的作用域是一个mapper的namespace** ，同一个namespace中查询sql可以从缓存中命中

在mapper中开启\<cache/\>,开启二级缓存，类必须序列化

可以再mybatis-config.xml中开启和关闭所有mapper中的缓存

```xml
< cache
eviction="FIFO" 可选LRU(默认),FIFO,SOFT,WEAK
flushInterval="60000" 60秒刷新
size="512 " 
readOnly="true"/>
```

#### 高级查询

##### 一对一查询

一个entity包含另一个entity

```xml
<!--方式1 扩展原始的entity, 用新的entity继承-->
<!-- 方式2 -->
public class Order {
    private Integer id;
    private Long userId;
    private String orderNumber;
    private Date created;
    private Date updated;
    private User user; //添加User对象，上面的属性都是order表里
}
<!-- 必须手动resultMap-->
 <resultMap id="resultMap" type="com.mybatis.pojo.Order" autoMapping="true">
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="order_number" property="orderNumber"/>
   
   			<!-- column="user_id"好像可以不要  -->
        <association property="user" column="user_id" javaType="com.mybatis.pojo.User" autoMapping="true">
						<!--内部对象User自己的映射-->
            <id column="id" property="id"/>
            <result column="user_name" property="userName"/>
        </association>
    </resultMap>
    <select id="queryOrderUserByOrderNumber" resultType="com.mybatis.pojo.Order" resultMap="resultMap">
      select * from tb_order o left join tb_user u on o.user_id=u.id where o.order_number = #{number}
    </select>
```

##### 一对多查询

```xml
<resultMap id="OrderUserDetailResultMap" type="com.mybatis.pojo.Order" autoMapping="true">
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="order_number" property="orderNumber"/>
        <association property="user" javaType="com.mybatis.pojo.User" autoMapping="true">
            <result column="user_name" property="userName"/>
        </association>
        <collection property="detailList" javaType="List" ofType="com.mybatis.pojo.OrderDetail" autoMapping="true">
            <result column="order_id" property="orderId"/>
            <result column="total_price" property="totalPrice"/>
        </collection>
    </resultMap>

```

#### jdbc参数，写在properties文件里

```properties
jdbc.initialSize=0       //初始化连接
jdbc.maxActive=30     //连接池的最大数据库连接数，设为0表示无限制
jdbc.maxIdle=20        //没有人用连接的时候，最大闲置的连接个数，设置为0时，表示没有限制。
jdbc.maxWait=1000    //超时等待时间以毫秒为单位
jdbc.removeAbandoned=true //是否自动回收超时连接
jdbc.removeAbandonedTimeout=60 //设置被遗弃的连接的超时的时间（以秒数为单位），即当一个连接被遗弃的时间超过设置的时间，则它会自动转换成可利用的连接。默认的超时时间是300秒。
jdbc.logAbandoned = true //是否在自动回收超时连接的时候打印连接的超时错误
jdbc.validationQuery=select 1 from dual //给出一条简单的sql语句进行验证
jdbc.testOnBorrow=true //在取出连接时进行有效验证

```

#### generator

自动生成xml，mapper。可以用命令行或者maven插件实现。

https://blog.csdn.net/hellozpc/article/details/80878563

##### 事务

用jdbc的事务或者spring的事务管理。