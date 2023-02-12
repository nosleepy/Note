---
title: Dubbo入门实例
date: 2020-05-24 10:03:52
tags:
categories:
- 框架
---

## 创建Maven工程

使用 IDEA 新建一个名为 dubbodemo 的 maven 工程

建立三个子模块：
+ dubbodemo-interface // 公共接口
+ dubbodemo-service // 服务提供者
+ dubbodemo-web // 服务消费者

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbodemo_项目目录结构.png)

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zhx2020.dubbo</groupId>
    <artifactId>dubbodemo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <modules>
        <module>dubbodemo-interface</module>
        <module>dubbodemo-service</module>
        <module>dubbodemo-web</module>
    </modules>

</project>
```

## dubbodemo-interface

**项目目录结构**

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbodemo_interface_项目目录结构.png)

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbodemo</artifactId>
        <groupId>com.zhx2020.dubbo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbodemo-interface</artifactId>
    <!-- 打包方式为jar包 -->
    <packaging>jar</packaging>

</project>
```

**UserService接口**

```java
public interface UserService {
    String getName();
}
```

## dubbodemo-service

**项目目录结构**

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbodemo_service_项目目录结构.png)

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbodemo</artifactId>
        <groupId>com.zhx2020.dubbo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbodemo-service</artifactId>
    <packaging>war</packaging>

    <properties>
        <spring.version>4.2.4.RELEASE</spring.version>
    </properties>

    <dependencies>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- dubbo相关 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <dependency>
            <groupId>javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.11.0.GA</version>
        </dependency>
        <!-- dubbodemo-interface -->
        <dependency>
            <groupId>com.zhx2020.dubbo</groupId>
            <artifactId>dubbodemo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <!-- 指定端口 -->
                    <port>8081</port>
                    <!-- 请求路径 -->
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

**UserServiceImpl实现类**

```java
@Service // 使用dubbo框架提供的@Service注解（com.alibaba.dubbo.config.annotation.Service）
public class UserServiceImpl implements UserService {
    @Override
    public String getName() {
        return "dubbodemo";
    }
}
```

**applicationContext-service.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <dubbo:application name="dubbodemo-service"/>
    <dubbo:registry address="zookeeper://192.168.235.128:2181"/>
    <dubbo:annotation package="com.zhx2020.dubbo.service.impl" />

</beans>
```

**web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">	
	
	<!-- 加载spring容器 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext*.xml</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
</web-app>
```

## dubbodemo-web

**项目目录结构**

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbodemo_web_项目目录结构.png)

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>dubbodemo</artifactId>
        <groupId>com.zhx2020.dubbo</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>dubbodemo-web</artifactId>
    <packaging>war</packaging>

    <properties>
        <spring.version>4.2.4.RELEASE</spring.version>
    </properties>

    <dependencies>
        <!-- Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jms</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <!-- dubbo相关 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.3</version>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.6</version>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <dependency>
            <groupId>javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>3.11.0.GA</version>
        </dependency>
        <!-- dubbodemo-interface -->
        <dependency>
            <groupId>com.zhx2020.dubbo</groupId>
            <artifactId>dubbodemo-interface</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.3.2</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <!-- 指定端口 -->
                    <port>8082</port>
                    <!-- 请求路径 -->
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

**UserController控制器**

```java
@Controller
@RequestMapping("/user")
public class UserController {
    @Reference // 使用dubbo框架提供的@Reference注解（com.alibaba.dubbo.config.annotation.Reference;）
    private UserService userService;

    @RequestMapping("/showName")
    @ResponseBody
    public String showName() {
        return userService.getName();
    }
}
```

**springmvc.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<mvc:annotation-driven >
		<mvc:message-converters register-defaults="false">
			<bean class="org.springframework.http.converter.StringHttpMessageConverter">  
				<constructor-arg value="UTF-8" />
			</bean>  
		</mvc:message-converters>	
	</mvc:annotation-driven>

	<!-- 引用dubbo 服务 -->
	<dubbo:application name="dubbodemo-web" />
	<dubbo:registry address="zookeeper://192.168.235.128:2181"/>
	<dubbo:annotation package="com.zhx2020.dubbo.controller"/>

</beans>
```

**web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">

   <!-- 解决post乱码 -->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
		<init-param>  
            <param-name>forceEncoding</param-name>  
            <param-value>true</param-value>  
        </init-param>  
	</filter>
	<filter-mapping>
	    <filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>	
	
    <servlet>
  	    <servlet-name>springmvc</servlet-name>
  	    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  	    <!-- 指定加载的配置文件 ，通过参数contextConfigLocation加载-->
  	    <init-param>
  		    <param-name>contextConfigLocation</param-name>
  		    <param-value>classpath:springmvc.xml</param-value>
  	    </init-param>
    </servlet>

    <servlet-mapping>
  	    <servlet-name>springmvc</servlet-name>
  	    <url-pattern>*.do</url-pattern>
    </servlet-mapping>
	
</web-app>
```

## 启动Zookeeper 

**zookeeper在linux系统的安装**

1、linux 系统上安装好 jdk
2、把 zookeeper 的压缩包上传到 linux 系统
3、解压缩压缩包

```
tar -zxvf zookeeper-3.4.6.tar.gz
```
4、进入 zookeeper-3.4.6 目录，创建 data 文件夹

```
mkdir data
```
5、进入 conf 目录，把 zoo_sample.cfg 改名为 zoo.cfg

```
cd conf
mv zoo_sample.cfg zoo.cfg
```
6、打开 zoo.cfg，修改 data 属性：`dataDir=/www/zookeeper-3.4.6/data`

**zookeeper服务启动**

进入 bin 目录，启动服务输入命令

```
./zkServer.sh start
```
输出以下内容表示启动成功

```
JMX enabled by default
Using config: /www/zookeeper-3.4.6/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

## 测试dubbodemo

父工程 dubbodemo 通过 maven 进行 package 和 install

运行 dubbodemo-service 中的 tomcat7 插件

```
[INFO] --- tomcat7-maven-plugin:2.2:run (default-cli) @ dubbodemo-service ---
[INFO] Running war on http://localhost:8081/
[INFO] Using existing Tomcat server configuration at C:\Users\wenlo\Desktop\Java框架\dubbodemo\dubbodemo-service\target\tomcat
[INFO] create webapp with contextPath: 
```

运行 dubbodemo-web 中的 tomcat7 插件

```
[INFO] --- tomcat7-maven-plugin:2.2:run (default-cli) @ dubbodemo-web ---
[INFO] Running war on http://localhost:8082/
[INFO] Creating Tomcat server configuration at C:\Users\wenlo\Desktop\Java框架\dubbodemo\dubbodemo-web\target\tomcat
[INFO] create webapp with contextPath: 
```

浏览器输入：localhost:8082/user/showName.do

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbodemo_项目运行结果.png)

出现 dubbodemo 则证明服务消费方成功调用了服务提供方的服务！

## dubbo-admin

我们在开发时，需要知道注册中心都注册了哪些服务，以便我们开发和测试。我们可以通过部署一个管理中心来实现。其实管理中心就是一个 web 应用，部署到 tomcat 即可。

在 linux 服务器上安装 tomcat，将 dubbo-admin.war 包上传到 linux 服务器的 tomcat 的 webapps 下。

启动 tomcat：`./startup.sh`

浏览器输入：192.168.235.128:8080/dubbo-admin/，登录用户名和密码都为 root

服务提供者

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbo_admin_providers.png)

服务消费者

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/dubbo_admin_consumers.png)




