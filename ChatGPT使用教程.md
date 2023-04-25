---
title: ChatGPT使用教程
date: 2023-04-25 19:06:13
tags:
- chatgpt
categories:
- 其他
---

** chatgpt 官网访问需配置国外网络代理**

#### 云服务器配置

![云服务器](https://raw.githubusercontent.com/nosleepy/picture/master/img/cloud_server.png)

1. 选购两台云服务器,配置选择 2 核 2 G
2. 香港云服务器用于 chatgpt 网页版搭建, 域名无需备案
3. 美国云服务器安装 tinyproxy 软件,搭建代理服务器

#### chatgpt 账号注册

1. 账号注册参考 [ChatGPT账号注册手把手教程（最新版）](https://juejin.cn/post/7214512376669175845#heading-4)
2. 国外手机号验证 [境外手机号注册](https://sms-activate.org/)

#### 网页版搭建

1. [chatgpt-mirror](https://github.com/yuezk/chatgpt-mirror)
2. [ChatGPT-Web搭建秘籍：了解最新AI技术的前沿应用！](https://zhuanlan.zhihu.com/p/616708963)

#### 常用网站

+ OpenAI登录：https://chat.openai.com/auth/login
+ ChatGPT对话：https://chat.openai.com/chat
+ 查看API剩余额度：https://platform.openai.com/account/usage
+ 新建API KEY：https://platform.openai.com/account/api-keys

#### nginx 配置

```
server {
    listen    80;
    server_name  aaa.com;
    location / {
        proxy_pass http://localhost:4000;
    }
}
server {
    listen    80;
    server_name  bbb.com;
    location / {
        proxy_pass http://localhost:3000;
   }
}
```

#### 其他链接

+ [在ubantu18.04中安装最新的nodejs——nvm安装方式](https://blog.csdn.net/qq_43206482/article/details/123509340)
+ [ubuntu 安装 nginx](https://juejin.cn/post/7111157784707612708)
+ [Linux中crontab命令定时任务基本语法](https://blog.csdn.net/bolinmengling/article/details/123994386)
