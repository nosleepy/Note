---
title: Spring面试题
date: 2020-05-15 10:40:06
tags:
- spring
categories:
- Java面试
---

## IOC和DI的理解

**IOC（控制反转）是一种设计思想**

+ **控制**就是指对对象的创建、维护、销毁等生命周期的控制，这个过程一般是由我们的程序去主动控制的，如使用 `new` 关键字去创建一个对象（创建），在使用过程中保持引用（维护），在失去全部引用后由 `GC` 去回收对象（销毁）。

+ **反转**就是指对对象的创建、维护、销毁等生命周期的控制由程序控制改为由 `IOC` 容器控制，需要某个对象时就直接通过名字去 `IOC` 容器中获取。

**DI（依赖注入）是 IOC 的一种重要实现**

+ 一个对象的创建往往会涉及到其他对象的创建，比如一个对象 `A` 的成员变量持有着另一个对象 `B` 的引用，这就是**依赖**，`A` 依赖于 `B`。

+ `IOC` 机制既然负责了对象的创建，那么这个依赖关系也就必须由 `IOC` 容器负责起来。负责的方式就是 `DI`（依赖注入），通过将依赖关系写入配置文件，然后在创建有依赖关系的对象时，由 `IOC` 容器注入依赖的对象，如在创建 `A` 时，检查到有依赖关系，`IOC` 容器就把 `A` 依赖的对象 `B` 创建后注入到 `A` 中（组装，通过反射机制实现），然后把 `A` 返回给对象请求者，完成工作。

**IOC 的意义**

+ `IOC` 并没有实现更多的功能，但它的存在使我们不需要很多代码、不需要考虑对象间复杂的耦合关系就能从 `IOC` 容器中获取合适的对象，而且提供了对对象的可靠的管理，极大地降低了开发的复杂性。

## AOP相关

**描述下AOP，AOP是如何实现的**

+ `AOP`（面向切面编程），就是在不改变原程序的基础上为代码段增加新的功能，对代码段进行增强处理。

+ 通过代理设计模式实现（ `JDK` 代理和 `CGLIB` 代理）。

**AOP和OOP的区别是什么**

+ `OOP` 是面向对象编程，核心思想是将客观存在的不同事物抽象成相互独立的类，然后把与事物相关的属性和行为封装到类里，并通过继承和多态来定义类彼此间的关系，最后通过操作类的实例来完成实际业务逻辑的功能需求。

+ `AOP` 是面向切面编程，核心思想是将业务逻辑中与类不相关的通用功能切面式的提取分离出来，让多个类共享一个行为，一旦这个行为发生改变，不必修改类，而只需要修改这个行为即可。

**动态代理和静态代理的区别**

+ **静态代理**：由程序员创建代理类或特定工具自动生成源代码再对其编译。在程序运行前代理类的.class文件就已经存在了。

+ **动态代理**：在程序运行时运用反射机制动态创建而成。

## Bean的scope

singleton（单例模式）、prototype（原型模式）、request（HTTP请求）、session（会话）、globalsession（全局会话）。

## 其它

**spring注解有哪些**

@Service、@Controller、@Repository、@Autowired、@Resource、@RestController、@RequestMapping、@Component、@ResponseBody、@RequestBody、@Value。

**对Mybatis的理解**

首先Mybatis是一个对象关系映射框架，是为了解决面向对象与关系数据库存在的互不匹配的现象。也就是说Mybatis的关注点在于对象与数据库之间的映射，Mybatis会把从数据库中得到的松散数据进行封装，使开发者直接拿到一个对象。Mybatis其实是对jdbc的操作数据库的过程进行了封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。

**Spring介绍一下**

Spring是一种可以解决对象创建以及对象之间依赖关系的框架，核心是IOC和AOP。

**SpringMVC工作流程（[博客园](https://www.cnblogs.com/luckyjcx/p/12332127.html)）**

1、用户通过客户端向服务器发送请求，请求会被Spring MVC的前端控制器DispatcherServlet所拦截。　
2、DispatcherServlet拦截请求后，会调用HandlerMapping处理器映射器。
3、处理器映射器根据请求的URL找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4、DispatcherServlet会通过返回信息选择合适的HandlerAdapter(处理器适配器)。
5、HandlerAdapter会调用并执行Handler(处理器)，这里的处理器指的是程序中编写的Controller类，也被称之为后端控制器。
6、Controlle执行完成后，会返回一个ModelAndView对象，该对象中会包含视图名或包含模型和视图名。
7、HandlerAdapter将ModelAndView对象返回给DispatcherServlet。
8、DispatcherServlet会根据ModelAndView对象选择一个合适的ViewReslover(视图解析器)。
9、ViewReslover解析后，会向DispatcherServlet中返回具体的View(视图)。
10、DispatcherServlet对View进行渲染(即将模型数据填充至视图中)。
11、视图渲染结果会返回给客户端浏览器显示（最后返回给用户）。

**Spring里面的设计模式（[知乎](https://zhuanlan.zhihu.com/p/66790602)）**

+ 工厂模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
+ 代理模式 : Spring AOP 功能的实现。
+ 单例模式 : Spring 中的 Bean 默认都是单例的。
+ 模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
+ 装饰者模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
+ 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
+ 适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。

**spring DI的几种注入方式（[知乎](https://zhuanlan.zhihu.com/p/26939845)）**

set注入、构造函数注入、p命名空间注入。

**AOP的应用场景**

性能统计／计数、事务处理、缓存处理、协议转换、日志打印。

**spring和springboot的区别？**

## SpringMVC

**SpringMVC常用注解都有哪些**

+ @RequestMapping：用于请求url映射。
+ @RequestBody：接收http请求的json数据，将json数据转换为java对象。
+ @ResponseBody：将Controller方法返回对象转化为json响应给客户。

**如何开启注解处理器和适配器**

在项目中一般会在 springmvc.xml 中通过开启 <mvc:annotation-driven>来实现注解处理器和适配器的开启。

## Spring

**Spring通知有哪些类型？**

+ before：前置通知，在一个方法执行前被调用。
+ after: 后置通知，在方法执行之后调用的通知，无论方法执行是否成功。
+ after-returning: 返回通知，仅当方法成功完成后执行的通知。
+ after-throwing: 异常通知，在方法抛出异常退出时执行的通知。
+ around: 环绕通知，在方法执行之前和之后调用的通知。

**JavaBean三种配置方式**

1. 基于XML的配置方式。
2. 基于注解的配置方式。
3. 基于Java类的配置方式。

## SpringBoot

**SpringBoot自动配置原理（[博客园](https://www.cnblogs.com/wenbochang/p/9851314.html)）**

+ SpringBoot主程序类上有@SpringBootApplication注解，运行这个启动类，SpringBoot就启动成功了，并且自动加载了好多Java配置类。
+ @SpringBootApplication注解是个组合注解，包含3个注解，@SpringBootConfiguration注解：标注在某个类上，表示这是一个SpringBoot的配置类。@EnableAutoConfiguration注解：开启自动配置功能。@ComponentScan：进行包扫描。
+ @EnableAutoConfiguration注解包含2个重要的注解，@AutoConfigurationPackage注解：自动配置包，内部使用@Import({Registrar.class})注解，通过new PackageImport(metadata).getPackageName()方法返回了当前主程序类的同级以及子级的包组件。@Import(AutoConfigurationImportSelector.class)注解: 导入自动配置的组件，内部通过加载META-INF/spring.factories外部文件，这个文件包含很多Java配置类。

**SpringBoot启动类**

启动Spring Boot项目，是基于Main方法来运行的。启动类在启动时会做注解扫描，扫描位置为同包或者子包下的注解，所以启动类应的位置应放于包的根目录下。

**SpringBoot启动器**

Spring Boot将所有的功能场景都抽取出来，做成一个个的starter(启动器)，只需要在项目里面引入这些starter，相关场景的的所有依赖都会导入进来，要用什么功能就导入什么场景，在jar包的管理上非常方便，最终实现一站式开发。

**启动类与启动器的区别**

启动类表示项目的启动入口，启动器表示jar的坐标。

**SpringBoot是什么**

Spring Boot是Spring框架的扩展，它消除了设置Spring应用程序所需的XML配置，开发更快、更高效。

**SpringBoot优点（[博客园](https://www.cnblogs.com/echola/p/10996214.html)）**

+ 创建独立的Spring应用程序。
+ 嵌入的Tomcat，Jetty或者Undertow，无须部署WAR文件。
+ 允许通过maven来根据需要获取starter。
+ 尽可能自动配置Spring。
+ 提供生产就绪型功能，如指标、健康检查和外部配置。
+ 绝对没有代码生成，对XML没有要求配置。

## 注解说明

**@Configuration**

标记在类上，将这个类作为配置类。

**@Bean**

标记在方法上，将方法的返回值作为一个bean注入IOC容器。

```java
@Configuration
public class AppConfig {

    @Bean
    public MyBean myBean() {
        // instantiate, configure and return bean ...
    }
}
```

## SpringCloud

**SpringCloud的结构**

1. 请求统一通过API网关(Zuul)来访问内部服务。
2. 网关接收到请求后，从注册中心(Eureka)获取可用服务。
3. 由(Ribbon)进行负载均衡后，分发到后端具体实例。
4. 微服务之间通过(Feign)客户端进行通信，处理业务。
5. (Hystrix)断路器负责处理服务超时熔断。
6. (Turbine)监控服务间的调用和熔断相关指标。

## Redis

## MyBatis

**mybatis中#{}和${}的区别**

+ Mybatis本身是基于JDBC封装的。
+ #{para}是预编译处理（PreparedStatement）范畴的。
+ ${para}是字符串替换。
+ Mybatis在处理#时，会调用PreparedStatement的set系列方法来赋值；处理$时，就是把${para}替换成变量的值。使用#{para}可以有效的防止SQL注入，提高系统安全性。

## 参考

+ [框架面试题:谈谈我对Spring IOC与DI的理解-阿善9](https://www.cnblogs.com/shan1393/p/8997429.html)
+ [静态代理和动态代理的区别](https://www.cnblogs.com/Eddyer/p/10984270.html)
