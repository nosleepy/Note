---
title: Linux常用命令
date: 2020-05-28 14:48:14
tags:
- linux
categories:
- Java面试
---

## 基础指令

**关机命令**

```
shutdown -h now // 正常关机
halt // 关闭内存
init 0 // 关机
poweroff // 关机
```
**ls**：列出文件和文件夹

```
ls -lha // -l 详细信息 -a 所有文件 -h 可读性更高的方式
ll // 作用和 ls -l 一样
```
**pwd**：打印当前工作目录

```
pwd
```

**cd**：切换目录

```
cd test 
```
**mkdir**：创建目录

```
mkdir test // 创建一个目录
mkdir -p a/b/c // -p 参数创建层级目录
mkdir aa bb cc // 创建多个目录
```

**touch**：创建文件

```
touch test.txt
```

**cp**：复制

```
cp 原路径 目标路径
cp test.txt .. // 复制test.txt文件到上级目录
cp -r test .. // -r 递归复制（复制目录）
```

**mv**：移动、剪切

```
mv 原路径 目标路径
mv test.txt .. // 移动test.txt文件到上级目录
mv test01.txt test02.txt // 重命名
```

**rm**：删除

```
rm -rf test // -r 递归删除 -f 强制删除
```

**vim**：文本编辑器

```
vim test.txt // 编辑text.txt文件
```

**>/>>**：输出重定向

```
ls -lah >/>> test.txt // > 覆盖输出 >> 追加输出
```

**cat**：查看文件内容

```
cat test.txt
```

## 进阶指令

**df**：查看磁盘空间

```
[root@localhost /]# df -h
文件系统                   容量  已用   可用  已用% 挂载点
/dev/mapper/centos-root   17G  2.5G   15G   15% /
devtmpfs                 898M     0  898M    0% /dev
tmpfs                    910M     0  910M    0% /dev/shm
tmpfs                    910M  9.5M  901M    2% /run
tmpfs                    910M     0  910M    0% /sys/fs/cgroup
/dev/sda1               1014M  146M  869M   15% /boot
tmpfs                    182M     0  182M    0% /run/user/0
```

**free**：查看内存使用情况

```
[root@localhost /]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1819         139        1496           9         183        1495
Swap:          2047           0        2047
```

**head**：查看一个文件的前n行，不指定n，默认查看前10行

```
[root@localhost test]# head -5 test.txt 
1 -------------------
2--------------------
3--------------------
4--------------------
5--------------------
```

**tail**：查看一个文件的末n行，不指定n，默认查看后10行

```
[root@localhost test]# tail -5 test.txt 
6--------------------
7--------------------
8--------------------
9--------------------
10-------------------
```

**less**：查看文件，以较少的内容进行输出

```
less test.txt
```

**wc**：统计文件内容信息

```
[root@localhost test]# wc -lwc test.txt // -l 行数 -w 单词数 -c 字节数
 12  27 310 test.txt
```

**date**：操作时间和日期

```
[root@localhost test]# date
2020年 05月 28日 星期四 16:23:39 CST
```

**cal**：操作日历

```
[root@localhost test]# cal
      五月 2020     
日 一 二 三 四 五 六
                1  2
 3  4  5  6  7  8  9
10 11 12 13 14 15 16
17 18 19 20 21 22 23
24 25 26 27 28 29 30
31
```

**clear / ctrl + l**：清除信息

```
clear / ctrl + l
```

**|**：管道

```
[root@localhost test]# ls -lah | grep hello // grep 过滤
-rw-r--r--.  1 root root   0 5月  28 16:26 aahelloaa.txt
-rw-r--r--.  1 root root   0 5月  28 16:26 aahello.txt
-rw-r--r--.  1 root root   0 5月  28 16:26 helloaa.txt
```

## 高级指令

**hostname**：操作服务器的主机名

```
[root@localhost ~]# hostname
localhost.localdomain
[root@localhost ~]# hostname localhost
[root@localhost ~]# hostname
localhost
```

**id**：查看一个用户的一些基本信息

```
[root@localhost home]# id root
uid=0(root) gid=0(root) 组=0(root) // 用户id 组id 附加组id
```

**whoami**：显示当前登录的用户名

```
[root@localhost home]# whoami
root
```

**ps**：查看服务器的进程信息

```
[root@localhost home]# ps -ef | grep firewalld // -e 所有列 -f 全字段
root       6272      1  0 17:31 ?        00:00:00 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
root       7653   7620  0 17:36 pts/0    00:00:00 grep --color=auto firewalld
```

**top**：查看服务器的进程占的资源

```
top // 进入，动态显示
q // 退出
m // 按照内存进行排序
p // 按照cpu使用率排序
```

**du**：查看目录的真实大小

```
[root@localhost /]# du -sh test // -s 只显示汇总的大小 -f 以可读性更高的方式显示
4.0K	test
```

**find**：查找文件

```
find 路径范围 选项 选项的值

[root@localhost test]# find . -name test.txt
./test.txt
[root@localhost test]# find . -type f
./test.txt
[root@localhost test]# find . -type d
.
./test
```

**systemctl**：控制一些软件的服务（启动、停止、重启）

```
[root@localhost test]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 四 2020-05-28 17:31:17 CST; 19min ago
     Docs: man:firewalld(1)
 Main PID: 6272 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─6272 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

5月 28 17:31:16 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
5月 28 17:31:17 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
```

**kill**：杀死进程

```
kill pid
```

**ifconfig**：操作网卡相关

```
[root@localhost test]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.235.128  netmask 255.255.255.0  broadcast 192.168.235.255
        inet6 fe80::123c:fabf:d78d:fc58  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:37:73:94  txqueuelen 1000  (Ethernet)
        RX packets 1543  bytes 114922 (112.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 907  bytes 397657 (388.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 64  bytes 5568 (5.4 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 64  bytes 5568 (5.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

**reboot**：重新启动计算机

```
reboot // 重启
reboot -w //模拟重启
```

**shutdown**：关机

```
shutdown -h now // 立刻关机
shutdown -h 18:34 "关机提示" // 定时关机 
shutdown -c // 撤销关机
```

**uptime**：输出计算机的持续在线时间

```
[root@localhost test]# uptime
 17:56:01 up 24 min,  1 user,  load average: 0.00, 0.01, 0.05
```

**uname**：获取计算机操作系统相关信息

```
[root@localhost test]# uname
Linux
```

**netstat**：查看网络连接状态

```
[root@localhost test]# netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      7106/sshd           
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      7400/master         
tcp6       0      0 :::22                   :::*                    LISTEN      7106/sshd           
tcp6       0      0 ::1:25                  :::*                    LISTEN      7400/master 
```

**man**：手册

```
man vim // 查看 vim 命令的详细用法
```

## 用户管理

**三个文件**

```
/etc/passwd // 存储用户的信息（查看用户主组）
/etc/group // 存储用户组的信息（查看用户附加组）
/etc/shadow // 存储用户的密码信息
```

**useradd**：添加用户

```
useradd 选项 用户名

-g // 指定用户的用户主组，值可以是用户组id或者组名
-G // 指定用户的用户附加组，值可以是用户组id或者组名（多个附加组用,分开）
-u // uid，用户的标识符
-c // 添加注释
```

**usermod**：修改用户

```
usermod 选项 用户名
-g // 指定用户的用户主组，值可以是用户组id或者组名
-G // 指定用户的用户附加组，值可以是用户组id或者组名（多个附加组用,分开）
-u // uid，用户的标识符
-l // 修改用户名（usermod -l new_name old_name）
```

**passwd**：设置密码

```
passwd centos
```

**userdel**：删除用户

```
userdel 选项 用户名
-r // 删除用户的同时删除其家目录
```

## 用户组管理

**groupadd**：添加用户组

```
groupadd 选项 用户组名
-g // gid，用户组的标识符
```

**groupmod**：修改用户组

```
groupmod 选项 用户组名
-g // gid，用户的标识符
-n // 修改用户组名（groupmod -n new_name old_name）
```

**groupdel**

```
groupdel 用户组名
```

## 权限管理

**权限介绍**

```
read（读权限）
write（写权限）
execute（执行权限）
```

**身份介绍**

```
owner 文件所有者，默认为文档的创建者
group 与文件所有者同组的用户
others 其他人，相对于所有者
root 超级用户
```

**linux权限介绍**

```
[root@localhost test]# ll
总用量 0
drwxr-xr-x. 2 root root 6 5月  28 17:47 test
-rw-r--r--. 1 root root 0 5月  28 17:47 test.txt

d // 文件夹
- // 文件
l // 软链接
s // 套接字
```

**chmod**：权限设置

```
chmod 选项 权限模式 文件或目录路径

-R 递归设置权限（文件夹）
```

```
字母形式

u // 文件所有者身份owner
g // 与文件所有者同组用户
o // 其它用户
a // 所有人

r // 读
w // 写
x // 执行
- // 没有

+ // 新增
- // 删除
= // 设置成

[root@localhost test]# ll
总用量 0
drwxr-xr-x. 2 root root 6 5月  28 17:47 test
-rw-r--r--. 1 root root 0 5月  28 17:47 test.txt
[root@localhost test]# chmod u+x,g+wx,o+wx test.txt
[root@localhost test]# ll
总用量 0
drwxr-xr-x. 2 root root 6 5月  28 17:47 test
-rwxrwxrwx. 1 root root 0 5月  28 17:47 test.txt
```

```
数字形式

r 读 4
w 写 2
x 执行 1
没有 0

[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root root 6 5月  28 17:47 test
-rwxrwx---. 1 root root 0 5月  28 17:47 test.txt
[root@localhost test]# chmod 505 test.txt 
[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root root 6 5月  28 17:47 test
-r-x---r-x. 1 root root 0 5月  28 17:47 test.txt
```

**属主和属组设置**

```
属主：所属的用户（文件所有者）
属组：所属的用户组（主组）
```

**chown**：更改文档的所属用户

```
-R 递归设置权限（文件夹）

[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root root 6 5月  28 17:47 test
-r-x---r-x. 1 root root 0 5月  28 17:47 test.txt
[root@localhost test]# chown centos test.txt
[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root   root 6 5月  28 17:47 test
-r-x---r-x. 1 centos root 0 5月  28 17:47 test.txt
```

**chgrp**：更改文档的所属组（主组）

```
-R 递归设置权限（文件夹）

[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root   root 6 5月  28 17:47 test
-r-x---r-x. 1 centos root 0 5月  28 17:47 test.txt
[root@localhost test]# chgrp centos test.txt
[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root   root   6 5月  28 17:47 test
-r-x---r-x. 1 centos centos 0 5月  28 17:47 test.txt
```

**更改所属用户和所属组**

```
chown -R username:groupname 文件或目录路径

[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 root   root   6 5月  28 17:47 test
-r-x---r-x. 1 centos centos 0 5月  28 17:47 test.txt
[root@localhost test]# chown -R centos:centos test
[root@localhost test]# ll
总用量 0
dr-xr-xr-x. 2 centos centos 6 5月  28 17:47 test
-r-x---r-x. 1 centos centos 0 5月  28 17:47 test.txt
```

## 其它命令

**ping**：检测当前主机和目标主机之间的连通性

```
ping 106.15.45.114
```

**su**：切换用户

```
su centos
```

**tar**：解压

```
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
```

**&**：用在一个命令的最后，可以把这个命令放到后台执行

```
java -jar blog.jar &
```

**ctrl + z**：可以将一个正在前台执行的命令放到后台，并且暂停

```
ctrl + z
```

**jobs**：查看当前有多少在后台运行的命令

```
[root@iZuf6a4hx8f9zxoz4hqzevZ www]# jobs
[1]+  Running    java -jar blog.jar &
```

**fg**：将后台中的命令调至前台继续运行

```
[root@iZuf6a4hx8f9zxoz4hqzevZ www]# fg %1
java -jar blog.jar
```

**bg**：将一个在后台暂停的命令，变成继续执行

```
[root@iZuf6a4hx8f9zxoz4hqzevZ www]# bg %1
[1]+ java -jar blog.jar &
```

**ln**：建立软链接

```
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/test$ ln -s helloworld.sh hello
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/test$ ls -lah
total 32K
drwxrwxr-x  7 wlzhou wlzhou 4.0K Mar  9 12:08 .
drwxr-xr-x 48 wlzhou wlzhou 4.0K Mar  9 10:57 ..
lrwxrwxrwx  1 wlzhou wlzhou   13 Mar  9 12:08 hello -> helloworld.sh
-rwxrw-r--  1 wlzhou wlzhou   32 Mar  9 12:06 helloworld.sh
wlzhou@wlzhou-Vostro-3888-China-HDD-Protection:~/test$ ./hello
hello world!!!
```

**mount**：磁盘挂载

```
mount /dev/vdb /data # 将/dev/vdb磁盘挂载到/data目录下
```

**fdisk -l**：查看磁盘信息

```
fdisk -l
Device     Boot Start      End  Sectors  Size Id Type
/dev/sdb1        2048 15378398 15376351  7.3G  c W95 FAT32 (LBA)
```
