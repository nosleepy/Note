---
title: ubuntu搭建文件服务器
date: 2023-06-15 00:00:00
tags:
categories:
- apache2
---

apache2 服务器安装

```sh
apt install apache2
```

浏览器输入本机ip,访问80端口

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/apache2_ubuntu.png)

基础命令

```sh
/etc/init.d/apache2 start # 启动
/etc/init.d/apache2 stop # 停止
/etc/init.d/apache2 restart # 重启
```

在 /var/www/html 目录下创建软链接

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/var/www/html$ ln -s ~/test files
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/var/www/html$ ls -lah
total 28K
lrwxrwxrwx 1 wlzhou wlzhou   17 Jun 15 10:53 files -> /home/wlzhou/test
-rwxrwxrwx 1 root   root    11K Jun 15 10:20 index.html
```

通过本机 ip + files 可以访问到 /home/wlzhou/test 目录

![](https://cdn.jsdelivr.net/gh/nosleepy/picture/img/apache2_files.png)

也可以直接将 /home/wlzhou/test 目录拷贝到 /var/www/html 目录

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/var/www/html$ cp -r  ~/test files
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:/var/www/html$ ls -lah
total 32K
drwxrwxr-x 3 wlzhou wlzhou 4.0K Jun 15 10:57 files
-rwxrwxrwx 1 root   root    11K Jun 15 10:20 index.html
```

修改默认端口为8080

```sh
vim /etc/apache2/ports.conf # Listen 8080
vim /etc/apache2/sites-enabled/000-default.conf # <VirtualHost *:8080>
/etc/init.d/apache2 restart # 重启
```

wget 下载文件

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ wget http://192.168.124.128/files/main.sh
--2023-06-15 11:06:59--  http://192.168.124.128/files/main.sh
Connecting to 192.168.124.128:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 47 [text/x-sh]
Saving to: 'main.sh'

main.sh                                            100%[================================================================================================================>]      47  --.-KB/s    in 0s      

2023-06-15 11:06:59 (19.0 MB/s) - 'main.sh' saved [47/47]

wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ ls -lah
total 12K
-rw-rw-r--  1 wlzhou wlzhou   47 Jun 15 10:57 main.sh
```

curl 下载文件

```sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ curl -O http://192.168.124.128/files/main.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    47  100    47    0     0  47000      0 --:--:-- --:--:-- --:--:-- 47000
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ ls -lah
total 12K
-rw-rw-r--  1 wlzhou wlzhou   47 Jun 15 11:08 main.sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ curl -o test.sh http://192.168.124.128/files/main.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    47  100    47    0     0  47000      0 --:--:-- --:--:-- --:--:-- 47000
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/mytest$ ls -lah
total 16K
-rw-rw-r--  1 wlzhou wlzhou   47 Jun 15 11:08 main.sh
-rw-rw-r--  1 wlzhou wlzhou   47 Jun 15 11:08 test.sh
```
