---
title: SpringBoot使用
date: 2020-12-08 10:59:08
tags:
categories:
- 框架
---

## SpringBoot环境搭建

**使用 IDEA 创建一个 maven 项目**

**pom.xml 文件配置**

```pom
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>springboot-demo</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.0.RELEASE</version>
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

**创建一个启动类**

```java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

**浏览器测试**

```java
// 访问 http://localhost:8080/

Whitelabel Error Page
This application has no explicit mapping for /error, so you are seeing this as a fallback.

Tue Dec 08 11:34:59 CST 2020
There was an unexpected error (type=Not Found, status=404).
No message available
```

**编写控制器**

```java
// 访问 http://localhost:8080/test/hello

@RestController
@RequestMapping("/test")
public class HelloController {

    @RequestMapping("/hello")
    public String sayHello() {
        return "hello spring boot";
    }
}
```

**数据库 account 表**

```mysql
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `money` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

## SpringBoot整合JPA

**添加 jpa 依赖**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

**编写实体类**

```java
@Entity
@Table(name = "account")
public class Account implements Serializable {

    @Id
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Column(name = "name")
    private String name;

    @Column(name = "money")
    private Integer money;
    
    // doGet()、doSet()、toString()
}
```

**编写业务层**

```java
@Service
public class AccountService {
    
    @Autowired
    private AccountDao accountDao;

    public List<Account> findAllAccount() {
        return accountDao.findAll();
    }
}
```

**编写持久层**

```java
@Repository
public interface AccountDao extends JpaRepository<Account, Integer> {
}
```

**编写控制器**

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAll")
    public List<Account> findAllAccount() {
        return accountService.findAllAccount();
    }
}
```

**核心配置文件**

```yml
server:
  port: 8080
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
  jpa:
    database: mysql
    show-sql: true
    hibernate:
      ddl-auto: update
  redis:
    host: 127.0.0.1
    port: 6379
    password: 123456
```

**浏览器测试**

```
// http://localhost:8080/account/findAll

[{"id":1,"name":"aaa","money":100},{"id":2,"name":"bbb","money":88}]
```

## SpringBoot整合Mybatis

**添加 mybatis 依赖**

```xml
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>
```

**编写持久层**

```java
@Mapper
public interface AccountMapper {

    @Select("select * from account where id = #{id}")
    List<Account> findAccountById(Integer id);
}
```

**业务层**

```java
@Service
public class AccountService {

    @Autowired
    private AccountMapper accountMapper;

    public Account findAccountById(Integer id) {
        return accountMapper.findAccountById(id);
    }
}
```

**控制器**

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/findAccountById")
    public Account findAccountById(Integer id) {
        return accountService.findAccountById(id);
    }
}
```

**浏览器测试**

```
// http://localhost:8080/account/findAccountById?id=1

{"id":1,"name":"aaa","money":100}
```

## SpringBoot整合Redis

**添加 redis 依赖**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**业务层**

```java
@Service
public class AccountService {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    public void setCache(String key, String value) {
        redisTemplate.opsForValue().set(key, value);
    }

    public String getCache(String key) {
        return redisTemplate.opsForValue().get(key);
    }
}
```

**控制器**

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @Autowired
    private AccountService accountService;

    @RequestMapping("/set")
    public void set(String key, String value) {
        accountService.setCache(key, value);
    }

    @RequestMapping("/get")
    public String get(String key) {
        return accountService.getCache(key);
    }
}
```

**浏览器测试**

```
// http://localhost:8080/account/set?key=id&value=1328

将key=id,value=1328存放到redis

// http://localhost:8080/account/get?key=id

取出redis中key=id的value
```

## SpringBoot整合Junit

**添加test依赖**

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

**测试类**

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = DemoApplication.class)
public class SpringBootJunitTest {

    @Autowired
    private AccountService accountService;

    @Test
    public void testFindAll() {
        List<Account> list = accountService.findAllAccount();
        System.out.println(list);
    }
}
```

**运行结果**

```
[Account{id=1, name='aaa', money=100}, Account{id=2, name='bbb', money=88}]
```