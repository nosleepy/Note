---
title: 手写SpringIOC容器框架
date: 2020-05-06 15:37:49
tags:
- spring
categories:
- 框架
---

## 手写 SpringIOC XML版本

### 实现步骤

**1.解析xml文件，获取所有bean节点信息**

**2.使用方法参数beanId查找配置文件中bean节点的id信息是否一致，返回class地址**

**3.获取class信息地址，使用反射机制初始化**

### 项目结构

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/myspringioc01.png)

### 相应代码

User.java

```java
package com.zhx2020.springioc.entity;

public class User {
    private int id;
    private String name;
    private String pass;

    public User() {
    }

    public User(int id, String name, String pass) {
        this.id = id;
        this.name = name;
        this.pass = pass;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPass() {
        return pass;
    }

    public void setPass(String pass) {
        this.pass = pass;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pass='" + pass + '\'' +
                '}';
    }
}
```

ExtClassPathXmlApplicationContext.java

```java
package com.zhx2020.springioc.myapp;

import org.apache.commons.lang.StringUtils;
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.io.InputStream;
import java.util.List;

//自定义spring容器框架xml方式实现
public class ExtClassPathXmlApplicationContext {
    //xml读取路径地址
    private String xmlPath;

    public ExtClassPathXmlApplicationContext(String xmlPath) {
        this.xmlPath = xmlPath;
    }

    //重构+设计模式
    public Object getBean(String beanId) throws Exception {
        if (StringUtils.isEmpty(beanId)) {
            throw new Exception("beanId不能为空!");
        }
        //1.解析xml文件，获取所有bean节点信息
        List<Element> readerXML = readerXML();
        if (readerXML == null || readerXML.isEmpty()) {
            throw new Exception("配置文件中没有配置bean的信息!");
        }
        //2.使用方法参数beanId查找配置文件中bean节点的id信息是否一致，返回class地址
        String className = findByElementClass(readerXML, beanId);
        //3.获取class信息地址，使用反射机制初始化
        if (StringUtils.isEmpty(className)) {
            throw new Exception("该bean对象没有配置class地址");
        }
        return newInstance(className);
    }

    //使用方法参数beanId查找配置文件中bean节点的id信息是否一致，返回class地址
    public String findByElementClass(List<Element> readerXML, String beanId) {
        for (Element element : readerXML) {
            //获取属性信息
            String xmlBeanId = element.attributeValue("id");
            if (StringUtils.isEmpty(xmlBeanId)) {
                continue;
            }
            if (xmlBeanId.equals(beanId)) {
                String xmlClass = element.attributeValue("class");
                return xmlClass;
            }
        }
        return null;
    }

    //解析xml文件
    public List<Element> readerXML() throws DocumentException {
        //1解析xml文件信息
        SAXReader saxReader = new SAXReader();
        Document document = saxReader.read(getResourceAsStream(xmlPath));
        //2读取根节点
        Element rootElement = document.getRootElement();
        //3获取根节点下所有子节点
        List<Element> elements = rootElement.elements();
        return elements;
    }

    //初始化对象
    public Object newInstance(String className) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Class<?> classInfo = Class.forName(className);
        return classInfo.newInstance();
    }

    //获取当前上下文路径
    public InputStream getResourceAsStream(String xmlPath) {
        InputStream inputStream = this.getClass().getClassLoader().getResourceAsStream(xmlPath);
        return inputStream;
    }
}
```

UserService.java

```java
package com.zhx2020.springioc.service;

public interface UserService {

    void addUser();

}
```

UserServiceImpl.java

```java
package com.zhx2020.springioc.service.impl;

import com.zhx2020.springioc.service.UserService;

public class UserServiceImpl implements UserService {

    public void addUser() {
        System.out.println("add an user-xml!");
    }
}
```

Main.java

```java
package com.zhx2020.springioc;

import com.zhx2020.springioc.entity.User;
import com.zhx2020.springioc.myapp.ExtClassPathXmlApplicationContext;
import com.zhx2020.springioc.service.UserService;

//测试手写SpringIOC代码
public class Main {
    public static void main(String[] args) throws Exception {
        ExtClassPathXmlApplicationContext app = new ExtClassPathXmlApplicationContext("spring.xml");
        User user = (User) app.getBean("user");
        System.out.println(user);
        UserService userService = (UserService) app.getBean("userServiceImpl");
        userService.addUser();
    }
}
```

spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.0.xsd
       http://www.springframework.org/schema/aop
       http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

    <bean id="user" class="com.zhx2020.springioc.entity.User"></bean>

    <bean id="userServiceImpl" class="com.zhx2020.springioc.service.impl.UserServiceImpl"></bean>

</beans>
```

### 运行结果

```
User{id=0, name='null', pass='null'}
add an user-xml!
```

## 手写 SpringIOC 注解版本

### 实现步骤

**1.获取当前包下的所有类**

**2.判断类上是否有注入bean的注解**

**3.初始化类中有依赖注入注解的属性值**

**4.通过beanId获取到容器中的对象**

### 项目结构

![](https://cdn.jsdelivr.net/gh/zhx2020/picture/img/myspringioc02.png))

### 相应代码

UserDao.java

```java
package com.zhx2020.springioc.dao;

import com.zhx2020.springioc.extannotation.ExtService;

@ExtService
public class UserDao {

    public void addOne() {
        System.out.println("add an user!-annotation");
    }

}
```

User.java

```java
package com.zhx2020.springioc.entity;

import com.zhx2020.springioc.extannotation.ExtService;

@ExtService
public class User {
    private int id;
    private String name;
    private String pass;

    public User() {
    }

    public User(int id, String name, String pass) {
        this.id = id;
        this.name = name;
        this.pass = pass;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getPass() {
        return pass;
    }

    public void setPass(String pass) {
        this.pass = pass;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", pass='" + pass + '\'' +
                '}';
    }
}
```

ExtResource.java

```java
package com.zhx2020.springioc.extannotation;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

//自定义注解resource进行依赖注入
@Target({TYPE, FIELD, METHOD})
@Retention(RUNTIME)
public @interface ExtResource {
}
```

ExtService.java

```java
package com.zhx2020.springioc.extannotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

//自定义注解service注入bean容器
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ExtService {
}
```

ExtClassPathXmlApplicationContext.java

```java
package com.zhx2020.springioc.myapp;

import com.zhx2020.springioc.extannotation.ExtResource;
import com.zhx2020.springioc.extannotation.ExtService;
import com.zhx2020.springioc.utils.ClassUtil;
import org.apache.commons.lang.StringUtils;

import java.lang.reflect.Field;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

//手写SpringIOC注解版本
public class ExtClassPathXmlApplicationContext {
    //扫包的范围
    private String packageName;
    //spring bean容器
    private ConcurrentHashMap<String, Object> beans = null;

    public ExtClassPathXmlApplicationContext(String packageName) throws Exception {
        this.packageName = packageName;
        beans = new ConcurrentHashMap<String, Object>();
        initBeans();
        initEntryField();
    }

    //初始化属性
    public void initEntryField() throws Exception {
        //1遍历所有的bean容器对象
        for (Map.Entry<String, Object> entry : beans.entrySet()) {
            //2判断属性上面是否有加注解
            Object object = entry.getValue();
            attributeAssign(object);
        }
    }

    public Object getBean(String beanId) throws Exception {
        if (StringUtils.isEmpty(beanId)) {
            throw new Exception("beanId不能为空!");
        }
        //从spring容器获取bean
        Object object = beans.get(beanId);
        attributeAssign(object);
        return object;
    }

    //初始化对象
    public Object newInstance(Class<?> classInfo) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        return classInfo.newInstance();
    }

    //初始化对象
    public void initBeans() throws Exception {
        //1使用java的反射机制扫包，获取当前包下的所有的类
        List<Class<?>> classes = ClassUtil.getClasses(packageName);
        //2判断类上是否存在注入bean的注解
        ConcurrentHashMap<String, Object> classExistAnnotation = findClassExistAnnotation(classes);
        if (classExistAnnotation == null || classExistAnnotation.isEmpty()) {
            throw new Exception("该包下没有任何类加上注解!");
        }
    }

    //判断类上是否存在注入bean的注解
    public ConcurrentHashMap<String, Object> findClassExistAnnotation(List<Class<?>> classes) throws IllegalAccessException, InstantiationException, ClassNotFoundException {
        for (Class<?> classInfo : classes) {
            //判断类上是否有注解
            ExtService annotation = classInfo.getAnnotation(ExtService.class);
            if (annotation != null) {
                //获取当前类名
                String className = classInfo.getSimpleName();
                //将当前类名变为小写
                String beanId = toLowerCaseFirstOne(className);
                Object newInstance = newInstance(classInfo);
                beans.put(beanId, newInstance);
            }
        }
        return beans;
    }

    // 首字母转小写
    public static String toLowerCaseFirstOne(String s) {
        if (Character.isLowerCase(s.charAt(0)))
            return s;
        else
            return (new StringBuilder()).append(Character.toLowerCase(s.charAt(0))).append(s.substring(1)).toString();
    }

    //依赖注入注解原理
    public void attributeAssign(Object object) throws Exception {
        //1使用反射机制，获取当前类的所有属性
        Class<? extends Object> classInfo = object.getClass();
        Field[] declaredFields = classInfo.getDeclaredFields();
        //2判断当前类的属性是否存在注解
        for (Field field : declaredFields) {
            ExtResource extResource = field.getAnnotation(ExtResource.class);
            if (extResource != null) {
                //获取属性名称
                String beanId = field.getName();
                Object bean = getBean(beanId);
                if (bean != null) {
                    //3默认使用属性名称，查找bean容器对象，1参数：当前对象 2参数：给属性赋值的对象
                    field.setAccessible(true); //允许访问私有属性
                    field.set(object, bean);
                }
            }
        }
    }

}
```

UserService.java

```java
package com.zhx2020.springioc.service;

public interface UserService {

    void addUser();

}
```

UserServiceImpl.java

```java
package com.zhx2020.springioc.service.impl;

import com.zhx2020.springioc.dao.UserDao;
import com.zhx2020.springioc.extannotation.ExtResource;
import com.zhx2020.springioc.extannotation.ExtService;
import com.zhx2020.springioc.service.UserService;

@ExtService
public class UserServiceImpl implements UserService {

    @ExtResource
    private UserDao userDao;

    public void addUser() {
        userDao.addOne();
    }

}
```

ClassUtil.java

```java
package com.zhx2020.springioc.utils;

import java.io.File;
import java.io.FileFilter;
import java.io.IOException;
import java.net.JarURLConnection;
import java.net.URL;
import java.net.URLDecoder;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

public class ClassUtil {

    /**
     * 取得某个接口下所有实现这个接口的类
     */
    public static List<Class> getAllClassByInterface(Class c) {
        List<Class> returnClassList = null;

        if (c.isInterface()) {
            // 获取当前的包名
            String packageName = c.getPackage().getName();
            // 获取当前包下以及子包下所以的类
            List<Class<?>> allClass = getClasses(packageName);
            if (allClass != null) {
                returnClassList = new ArrayList<Class>();
                for (Class classes : allClass) {
                    // 判断是否是同一个接口
                    if (c.isAssignableFrom(classes)) {
                        // 本身不加入进去
                        if (!c.equals(classes)) {
                            returnClassList.add(classes);
                        }
                    }
                }
            }
        }

        return returnClassList;
    }

    /*
     * 取得某一类所在包的所有类名 不含迭代
     */
    public static String[] getPackageAllClassName(String classLocation, String packageName) {
        // 将packageName分解
        String[] packagePathSplit = packageName.split("[.]");
        String realClassLocation = classLocation;
        int packageLength = packagePathSplit.length;
        for (int i = 0; i < packageLength; i++) {
            realClassLocation = realClassLocation + File.separator + packagePathSplit[i];
        }
        File packeageDir = new File(realClassLocation);
        if (packeageDir.isDirectory()) {
            String[] allClassName = packeageDir.list();
            return allClassName;
        }
        return null;
    }

    /**
     * 从包package中获取所有的Class
     *
     * @param packageName
     * @return
     */
    public static List<Class<?>> getClasses(String packageName) {

        // 第一个class类的集合
        List<Class<?>> classes = new ArrayList<Class<?>>();
        // 是否循环迭代
        boolean recursive = true;
        // 获取包的名字 并进行替换
        String packageDirName = packageName.replace('.', '/');
        // 定义一个枚举的集合 并进行循环来处理这个目录下的things
        Enumeration<URL> dirs;
        try {
            dirs = Thread.currentThread().getContextClassLoader().getResources(packageDirName);
            // 循环迭代下去
            while (dirs.hasMoreElements()) {
                // 获取下一个元素
                URL url = dirs.nextElement();
                // 得到协议的名称
                String protocol = url.getProtocol();
                // 如果是以文件的形式保存在服务器上
                if ("file".equals(protocol)) {
                    // 获取包的物理路径
                    String filePath = URLDecoder.decode(url.getFile(), "UTF-8");
                    // 以文件的方式扫描整个包下的文件 并添加到集合中
                    findAndAddClassesInPackageByFile(packageName, filePath, recursive, classes);
                } else if ("jar".equals(protocol)) {
                    // 如果是jar包文件
                    // 定义一个JarFile
                    JarFile jar;
                    try {
                        // 获取jar
                        jar = ((JarURLConnection) url.openConnection()).getJarFile();
                        // 从此jar包 得到一个枚举类
                        Enumeration<JarEntry> entries = jar.entries();
                        // 同样的进行循环迭代
                        while (entries.hasMoreElements()) {
                            // 获取jar里的一个实体 可以是目录 和一些jar包里的其他文件 如META-INF等文件
                            JarEntry entry = entries.nextElement();
                            String name = entry.getName();
                            // 如果是以/开头的
                            if (name.charAt(0) == '/') {
                                // 获取后面的字符串
                                name = name.substring(1);
                            }
                            // 如果前半部分和定义的包名相同
                            if (name.startsWith(packageDirName)) {
                                int idx = name.lastIndexOf('/');
                                // 如果以"/"结尾 是一个包
                                if (idx != -1) {
                                    // 获取包名 把"/"替换成"."
                                    packageName = name.substring(0, idx).replace('/', '.');
                                }
                                // 如果可以迭代下去 并且是一个包
                                if ((idx != -1) || recursive) {
                                    // 如果是一个.class文件 而且不是目录
                                    if (name.endsWith(".class") && !entry.isDirectory()) {
                                        // 去掉后面的".class" 获取真正的类名
                                        String className = name.substring(packageName.length() + 1, name.length() - 6);
                                        try {
                                            // 添加到classes
                                            classes.add(Class.forName(packageName + '.' + className));
                                        } catch (ClassNotFoundException e) {
                                            e.printStackTrace();
                                        }
                                    }
                                }
                            }
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return classes;
    }

    /**
     * 以文件的形式来获取包下的所有Class
     *
     * @param packageName
     * @param packagePath
     * @param recursive
     * @param classes
     */
    public static void findAndAddClassesInPackageByFile(String packageName, String packagePath, final boolean recursive,
                                                        List<Class<?>> classes) {
        // 获取此包的目录 建立一个File
        File dir = new File(packagePath);
        // 如果不存在或者 也不是目录就直接返回
        if (!dir.exists() || !dir.isDirectory()) {
            return;
        }
        // 如果存在 就获取包下的所有文件 包括目录
        File[] dirfiles = dir.listFiles(new FileFilter() {
            // 自定义过滤规则 如果可以循环(包含子目录) 或则是以.class结尾的文件(编译好的java类文件)
            public boolean accept(File file) {
                return (recursive && file.isDirectory()) || (file.getName().endsWith(".class"));
            }
        });
        // 循环所有文件
        for (File file : dirfiles) {
            // 如果是目录 则继续扫描
            if (file.isDirectory()) {
                findAndAddClassesInPackageByFile(packageName + "." + file.getName(), file.getAbsolutePath(), recursive,
                        classes);
            } else {
                // 如果是java类文件 去掉后面的.class 只留下类名
                String className = file.getName().substring(0, file.getName().length() - 6);
                try {
                    // 添加到集合中去
                    classes.add(Class.forName(packageName + '.' + className));
                } catch (ClassNotFoundException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

Main.java

```java
package com.zhx2020.springioc;

import com.zhx2020.springioc.entity.User;
import com.zhx2020.springioc.myapp.ExtClassPathXmlApplicationContext;
import com.zhx2020.springioc.service.UserService;

//测试手写SpringIOC代码
public class Main {
    public static void main(String[] args) throws Exception {
        ExtClassPathXmlApplicationContext app = new ExtClassPathXmlApplicationContext("com.zhx2020.springioc");
        User user = (User) app.getBean("user");
        System.out.println(user);
        UserService userService = (UserService) app.getBean("userServiceImpl");
        userService.addUser();
    }
}
```

### 运行结果

```
User{id=0, name='null', pass='null'}
add an user!-annotation
```