---
title: gradle自定义插件
date: 2023-05-16 00:00:00
tags:
categories:
- 安卓
---

#### 自定义插件步骤

androidstudio 新建一个 module 名叫 myplugin

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/create_plugin_module.png)

配置 myplugin/build.gradle

```groovy
plugins {
    id 'groovy'
    id 'maven-publish'
}

dependencies {
    implementation gradleApi()
    implementation localGroovy()
}

publishing {
    publications {
        maven(MavenPublication) {
            groupId = 'com.grandstream.myplugin'
            artifactId = 'test'
            version = '1.0.0'
            from components.java
        }
    }

    repositories {
        maven {
            url = '../repo'
        }
    }
}
```

由于使用 groovy 语言开发,新建 groovy 目录和 resources 目录

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/gradle_plugin_test.png)

TestPlugin.groovy

```groovy
package com.grandstream.myplugin

import org.gradle.api.Plugin
import org.gradle.api.Project

class TestPlugin implements Plugin<Project> {
    @Override
    void apply(Project project) {
        project.getExtensions().create("user", UserExtension.class)
        project.afterEvaluate {
            UserExtension userExtension = project.getExtensions().getByType(UserExtension.class)
            println "username = " + userExtension.username + ",password = " + userExtension.password
        }
        project.getTasks().create("testTask") {
			println '--------------------testTask--------------------------'
        }
    }
}
```

UserExtension.groovy

```groovy
package com.grandstream.myplugin

class UserExtension {
    String username;
    String password;
}
```

com.grandstream.myplugin.properties

```properties
implementation-class=com.grandstream.myplugin.TestPlugin
```

运行 maven-publish 插件的 publish 任务将插件发布到本地 repo 仓库

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/run_publish.png)

GradlePluginTest 工程下会生成 repo 文件夹

app 模块使用 myplugin 插件

/GradlePluginTest/settings.gradle

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
        maven {
            url uri('repo')
        }
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "GradePluginTest"
include ':app'
include ':myplugin'
```

app/build.gradle

```groovy
plugins {
    id 'com.android.application'
    id 'com.grandstream.myplugin'
}

user {
    username "admin"
    password "123456"
}
```

同步后 app 模块的 gradle 任务中会多一个 testTask 任务

![](https://cdn.jsdelivr.net/gh/nosleepy/picture@master/img/generate_test_task.png)

双击运行，查看日志信息

```
下午5:48:23: Executing 'testTask'...


> Configure project :app
--------------------testTask--------------------------
username = admin,password = 123456

> Task :app:testTask UP-TO-DATE

BUILD SUCCESSFUL in 177ms
下午5:48:23: Execution finished 'testTask'.
```

可以看到，打印出了 user 扩展信息和 testTask 任务的输出信息

#### 实现图片压缩插件

tinypng 官网注册账号,得到调用图片压缩的 apikey

源码地址：https://github.com/nosleepy/GradlePluginTest

#### 参考

+ [tinypng](https://tinify.cn/)
+ [从0开始写一个gradle插件](https://www.bilibili.com/video/BV1Xf4y157RT/?vd_source=d45c9618408f29c5df779b74044b7f18)