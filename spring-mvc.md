#### 核心组件

1. DispatcherServlet : 前置控制器
2. Handler : 处理器,完成具体业务逻辑
3. HandlerMapping : 将请求分发到不同的handler
4. HandlerInterceptor : 处理器拦截器，一个接口
5. HandlerExecutionChain : 处理器执行链，至少包含一个handler和一个默认的 handlerInterceptor
6. HandlerAdapter : Handler在处理业务前，对表单数据进行一系列操作（验证，类型转换，封装到bean等） 处理器适配器
7. ModelAndView :装载模型数据和视图信息(逻辑视图)
8. ViewResolver :视图解析器

#### 流程

<img src="/Users/kangkang/Library/Application Support/typora-user-images/image-20200616171740050.png" alt="image-20200616171740050" style="zoom: 33%;" />

1. 客户端请求被DispatcherServlet接收
2. DispatcherServlet将请求映射到Handler
3. 生成Handler以及HandlerInterceptor
4. 返回HandlerExecutionChain ( Handler+ HandlerInterceptor ) 
5. DispatcherServlet通过HandlerAdapter执行Handler
6. 并返回一个ModelAndView
7. DispatcherServlet通过ViewResolver进行解析，将**逻辑视图**转为**物理视图**
8. 将**模型数据**填充到View中 ,响应给客户端 

```java
@Override
public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
    ModelAndView modelAndView=new ModelAndView();
    //添加模型数据
    modelAndView.addObject("name","tom");
    //添加逻辑视图
    modelAndView.setViewName("show ");
    return modelAndView;
}
```

#### Handler

在解析modelAndView时，是把放入的对象放进了request中。

##### 数据绑定

将HTTP请求中的参数绑定到Handler业务方法的形参, 会将HTTP请求中的 参数进行封装。

 HandlerAdapter-->HttpMessageConverter-->Handler

@ResponseBody可以直接将方法的返回值给客户端，不需要跳转jsp页面

绑定集合时，不能直接传入Lists,set,map等类型，要用**包装类**，包装类的属性是List，set, map类型

**set绑定需要实例化，并在在无参构造函数中添加两个空对象**

```java
public class CourseList {
    private List<Course> courses;

    public List<Course> getCourses() {
        return courses;
    }

    public void setCourses(List<Course> courses) {
        this.courses = courses;
    }
}

public class CourseSet {
    private Set<Course> courses = new HashSet<Course>();
    public void setCourses(Set<Course> courses) {
        this.courses = courses;
    }
    public Set<Course> getCourses() {
        return courses;
    }
    public  CourseSet(){
        courses.add(new Course());
        courses.add(new Course());
    }
}
////////////////////
@RequestMapping(value = "/listType")
    public ModelAndView listType(CourseList courseList){
        for(Course course:courseList.getCourses()){
            courseDAO.add(course);
        }
        ModelAndView modelAndView = new ModelAndView();
        modelAndView.setViewName("index");
        modelAndView.addObject("courses",courseDAO.getAll());
        return modelAndView;
    }
```

绑定json数据，需要在spring.xml中定义一个**消息转换器**

前台：

```html
$(function(){
        var course = {
            "id":"8",
            "name":"SSM框架整合",
            "price":"200"
        };
        $.ajax({
            url:"jsonType",
            data:JSON.stringify(course),
            type:"post",
            contentType:"application/json;charse=UTF-8", //传入的数据类型，编码防止中文乱码
            dataType:"json",  //返回的数据类型
            success:function(data){
                alert(data.name+"---"+data.price);
            }
        })
    })
```

#### web.xml

```xml
<url-pattern>/</url-pattern>  不会匹配*.jsp
<url-pattern>/*</url-pattern> 会匹配*.jsp
```

Servlet和Filter的url匹配顺序

① 完全匹配 /test/list.do
② 目录匹配 /test/*
③ 扩展名匹配 *.do servlet-mapping的重要规则：
   ☆ 容器会首先查找完全匹配，如果找不到，再查找目录匹配，如果也找不到，就查找扩展名匹配。
   ☆ 如果一个请求匹配多个“目录匹配”，容器会选择最长的匹配。

#### RESTful

representational state transfer: 资源(text,service等)，资源的表述(传送)，状态转移(资源修改)

特点：统一了客户端访问资源的接口；url更加简洁；有利于不同系统之间的资源共享

@RestController 使类的方法返回的数据直接返回给客户端，而不是一个页面名称；方法上不用再写@ResponseBody

#### 处理put、delete请求

一般来说，资源操有查询，新增，删除，更改四种类型，对应HTTP协议中四类请求：GET，POST，DELETE，PUT。 未声明情况下浏览器默认使用GET提交请求。需要注意的是，普通浏览器只支持GET，POST方式 ，其他请求方式如DELETE|PUT必须通过过滤器的支持才能实现。Spring自带了一个过滤器HiddenHttpMethodFilter，支持GET、POST、PUT、DELETE请求。首先创建一个filter，**然后在前端中写一个隐藏域**

```xml
    <form action="handler/testRest/1234" method="post">
        <input type="hidden"  name="_method" value="DELETE"/>
        <input type="submit" value="删">
    </form>
```

#### 拦截器

第一种方式是要定义的Interceptor类要实现了Spring的HandlerInterceptor 接口

  第二种方式是继承实现了HandlerInterceptor接口的类，比如Spring已经提供的实现了HandlerInterceptor接口的抽象类HandlerInterceptorAdapter

**preHandle()**： 这个方法在业务处理器处理请求之前被调用，SpringMVC 中的Interceptor 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个Interceptor 。每个Interceptor 的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor 中的preHandle 方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值Boolean 类型的，当它返回为false 时，表示请求结束，后续的Interceptor 和Controller 都不会再执行；当返回值为true 时就会继续调用下一个Interceptor 的preHandle 方法，如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller 方法。

   **postHandle()**：这个方法在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。postHandle 方法被调用的方向跟preHandle 是相反的，也就是说先声明的Interceptor 的postHandle 方法反而会后执行。

   **afterCompletion()**：该方法也是需要当前对应的Interceptor 的preHandle 方法的返回值为true 时才会执行。顾名思义，该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。

作用：权限控制，登录拦截，字符集，国际化

