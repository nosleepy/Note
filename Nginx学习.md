---
title: Nginx学习
date: 2020-09-16 10:20:23
tags:
categories:
---

## Nginx介绍

**Nginx介绍**

Nginx是由俄罗斯人研发的，应对Rambler的网站，并且2004年发布的第一个版本。

**Nginx的特点**

1. 稳定性极强。
2. 提供了非常丰富的配置实例。
3. 占用内存小，并发能力强。

## Nginx的安装

[nginx安装及其配置详细教程](https://www.cnblogs.com/lywJ/p/10710361.html)

**Nginx的配置文件**

```conf
# nginx.conf

worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## Nginx实现反向代理

**正向代理**

1. 正向代理服务器是由客户端设立的。
2. 客户端了解代理服务器和目标服务器都是谁。
3. 帮助我们实现突破访问权限，提高访问的速度，对目标服务器隐藏客户端的ip地址。

**反向代理**

1. 反向代理服务器是配置在服务端的。
2. 客户端是不知道访问的到底是哪一台服务器。
3. 达到负载均衡，并且可以隐藏服务器真正的ip地址。

**基于Nginx实现反向代理**

```conf
# 基于反向代理访问到Tomcat服务器
server {
    listen       80;
    server_name  localhost;

    location / {
    	proxy_pass http://192.168.235.128:8080/;
    }
}
```

**关于Nginx的location路径映射**

```conf
location /xxx {
	# 匹配所有以/xxx开头的路径
}
```

## Nginx负载均衡

**Nginx为我们默认提供了三种负载均衡的策略**

1. 轮询：将客户端发起的请求，平均的分配给每一台服务器。
2. 权重：会将客户端的请求，根据服务器的权重不同，分配不同的数量。
3. ip_hash：基于发起请求的客户端的ip地址不同，它始终会将请求发送到指定的服务器上。

**轮询**

```conf
upstream my-server {
	server 192.168.235.128:8081;
	server 192.168.235.128:8082;
}

server {
    listen       80;
    server_name  localhost;

    location / {
    	proxy_pass http://my-server/;   
    }
}
```

**权重**

```conf
upstream my-server {
	server 192.168.235.128:8081 weight=10;
	server 192.168.235.128:8082 weight=2;
}

server {
    listen       80;
    server_name  localhost;

    location / {
    	proxy_pass http://my-server/;   
    }
}
```

**ip_hash**

```conf
upstream my-server {
	ip_hash;
	server 192.168.235.128:8081;
	server 192.168.235.128:8082;
}

server {
    listen       80;
    server_name  localhost;

    location / {
    	proxy_pass http://my-server/;   
    }
}
```

## Nginx动静分离

Nginx的并发能力公式：worker_processes * worker_connections / 4 | 2 = Nginx最终的并发能力。

动态资源需要 / 4，静态资源需要 / 2。

Nginx通过动静分离，来提升Nginx的并发能力，更快的给用户响应。

**动态资源代理**

```conf
# 配置如下
location / {
	proxy_pass http://192.168.235.128:8080/;
}
```

**静态资源代理**

```conf
# 配置如下

location / {
	root   html;
	index  index.html index.htm;
}

location /data {
	alias   data;
	index  index.html;
}

location /img {
	alias   image;
	autoindex on; # 代表展示静态资源下的全部内容，以列表的形式展开
}
```

## Nginx集群



## 参考

+ [2020最新 Nginx教程全面讲解（Nginx快速上手）]https://www.bilibili.com/video/BV1W54y1z7GM)