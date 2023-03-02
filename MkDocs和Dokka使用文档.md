#### MkDocs使用

+ 安装mkdocs

升级 pyhton 和 pip3 版本，python 版本 3.8 以上

```shell
pip3 install mkdocs
```

+ 创建项目

```shell
mkdocs new testdocs
```

+ 文档预览

```shell
mkdocs serve
```

+ 项目目录结构

```
.
├── docs
│   └── index.md
└── mkdocs.yml
```

mkdocs.yml 配置文件

```yml
site_name: My Docs
```

index.md 文档主页

```md
# Welcome to MkDocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.
```

+ 下载主题

```shell
pip3 install mkdocs-material mkdocs-windmill
```

+ 启用主题

```shell
theme:
  name: material
```

#### Dokka使用

+ 配置dokka插件

```gradle
plugins {
    id 'org.jetbrains.dokka' version '1.7.20' apply false
}
```

+ 模块中使用插件

```gradle
plugins {
    id 'org.jetbrains.dokka'
}

dependencies {
    dokkaPlugin("org.jetbrains.dokka:kotlin-as-java-plugin:1.7.20")
}
```

```gradle
plugins {
    id 'org.jetbrains.dokka'
}

tasks.withType<DokkaTask>().configureEach {
    dokkaSourceSets {
        named("main") {
            // used as project name in the header
            moduleName.set("Dokka Gradle Example")

            // contains descriptions for the module and the packages
            includes.from("Module.md")

            // adds source links that lead to this repository, allowing readers
            // to easily find source code for inspected declarations
            sourceLink {
                localDirectory.set(file("src/main/kotlin"))
                remoteUrl.set(URL("https://github.com/Kotlin/dokka/tree/master/" +
                        "examples/gradle/dokka-gradle-example/src/main/kotlin"
                ))
                remoteLineSuffix.set("#L")
            }
        }
    }
}
```

+ 运行gradle插件

```
+Tasks
	+documention
		+dokkaHtml          #生成html文档
		+dokkaJavadoc   #生成java文档
		+dokkaJekyll        #生成md文档
```

#### 参考

+ [mkdocs官网](https://squidfunk.github.io/mkdocs-material/)
+ [dokka官网](https://github.com/Kotlin/dokka)