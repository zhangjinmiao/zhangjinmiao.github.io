---
layout: post
title: Java 开发环境搭建
categories: Java
description: some word here
keywords: Java, 软件安装, 环境搭建
---

## Mac 环境搭建

参考：https://mp.weixin.qq.com/s/JcIVBo6LJw_zyZolZ1MYGQ

## JDK 安装

参考：

> 1. [https://www.linuxidc.com/Linux/2016-09/134941.htm](https://www.linuxidc.com/Linux/2016-09/134941.htm)
> 2. [https://zhuanlan.zhihu.com/p/28852767](https://zhuanlan.zhihu.com/p/28852767)

**步骤：**

1. 在 usr 下新建文件夹 jdk

   ````shell
   [root@VM_0_16_centos usr]# mkdir jdk
   ````

2. 下载 [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) 并解压

   ````shell
   [root@VM_0_16_centos jdk]# tar -zxvf jdk-8u161-linux-x64.tar.gz 
   ````

3. 设置环境变量

   ````shell
   # set java environment
   JAVA_HOME=/usr/jdk/jdk1.8.0_161
   JRE_HOME=/usr/jdk/jdk1.8.0_161/jre
   CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
   PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
   export JAVA_HOME JRE_HOME CLASS_PATH PATH
   ````

   让其生效：

   ````shell
   [root@VM_0_16_centos jdk]# source /etc/profile
   ````

   

4. 验证 JDK 有效性

   ````shell
   [root@VM_0_16_centos jdk]# java -version
   java version "1.8.0_161"
   Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
   Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
   [root@VM_0_16_centos jdk]# 
   ````



**使用 root 安装，其他用户使用不了，需授权**

**chmod 755** 设置用户的权限为：

1.文件所有者可读可写可执行

2.与文件所有者同属一个用户组的其他用户可读可执行

3.其它用户组可读可执行

````shell
[root@VM_0_16_centos jdk]# sudo chmod -R 755 /usr/jdk/jdk1.8.0_161/
[root@VM_0_16_centos jdk]# sudo chown -R zhangjm /usr/jdk/jdk1.8.0_161/
````



## Redis 安装

**参考：**

单机安装：

> 1. [http://www.redis.net.cn/tutorial/3503.html](http://www.redis.net.cn/tutorial/3503.html)
> 2. http://www.redis.cn/download.html
> 3. [https://segmentfault.com/a/1190000010709337#articleHeader0](https://segmentfault.com/a/1190000010709337#articleHeader0)

集群安装：

> 1. [http://bbs.redis.cn/forum.php?mod=viewthread&tid=483](http://bbs.redis.cn/forum.php?mod=viewthread&tid=483)
> 2. [https://segmentfault.com/a/1190000010682551](https://segmentfault.com/a/1190000010682551)



### 单机安装

**步骤：**

1. 在 usr 下新建文件夹 redis

   ````shell
   [root@VM_0_16_centos usr]# mkdir redis
   ````

   

2. 下载 [redis](http://download.redis.io/releases/redis-4.0.8.tar.gz) 解压、编译

   ````shell
   wget http://download.redis.io/releases/redis-4.0.8.tar.gz
   $ tar xzf redis-4.0.8.tar.gz
   $ cd redis-4.0.8
   $ make
   ````

   

3. 查看版本

   ````shell
   zhangjm@VM_0_16_centos redis-4.0.8]$ ./src/redis-server -v
   Redis server v=4.0.8 sha=00000000:0 malloc=jemalloc-4.0.3 bits=64 build=50ba0c8cdd44abe3
   [zhangjm@VM_0_16_centos redis-4.0.8]$ 
   ````

   

**连接 linux 下的 redis 服务器（如果无法连接一般是防火墙或保护模式的问题，按以下步骤操作可解决**

1. 修改 redis.conf 配置文件

   在 127.0.0.1 前面加上注释（redis4.0 以下版本默认是注释掉的）

   将受保护模式修改为 no（redis4.0 以下的版本没有这个模式配置，不用修改）

   

2. 在 linux 下的防火墙中开放 6379 端口（与 centos7 以下版本开放端口的方式有区别）

   ````shell
   [root@localhost bin]# firewall-cmd --zone=public --add-port=6379/tcp --permanent  
   success  
   ````

   若报 FirewallD is not running 

   参考：[https://jingyan.baidu.com/article/5552ef47f509bd518ffbc933.html](https://jingyan.baidu.com/article/5552ef47f509bd518ffbc933.html)

   

3. 重启防火墙

   ````shell
   [root@localhost bin]# systemctl restart firewalld  
   ````

   

4. 启动 redis

   ````shell
   [root@localhost bin]# ./redis-server redis.conf  
   ````

   

5. 连接测试



**脚本使用**

脚本文件位于 /usr/redis/redis-4.0.8/utils 下的 redis_init_script 。

复制到 /etc/init.d/  redisd 下并修改为如下：

````shell
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.
#端口号，这是默认的，如果你安装的时候不是默认端口号，则需要修改
REDISPORT=6379 
#redis-server启动脚本的位置，你如果忘了可以用find或whereis找到 
EXEC=/usr/redis/redis-4.0.8/src/redis-server
#redis-cli客户端启动脚本的位置，你如果忘了可以用find或whereis找到  
CLIEXEC=/usr/redis/redis-4.0.8/src/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
#redis.conf配置文件的位置，需在 etc 下新建文件夹 redis, 并将 redis.conf 重命名为 6379.conf 并复制到 /etc/redis 中
CONF="/etc/redis/${REDISPORT}.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac

````

设置可执行权限：

设置可执行权限：

````shell
chmod 755  /etc/init.d/redis 
````



启动测试：

````shell
$ /etc/init.d/redis start
````

可设置开机自启动:

````shell
chkconfig redis on
````



停止：

````shell
$ /etc/init.d/redis stop
````





### 集群安装





## Nginx 安装



## Mongodb 安装



## Zookeeper 安装



## MySQL 安装

