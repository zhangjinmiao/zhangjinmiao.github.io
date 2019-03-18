## 网络操作
查看网络通不通
ping www.baidu.com
>ubutu@10-10-80-30:~$ ping www.baidu.com
PING www.a.shifen.com (180.97.33.108) 56(84) bytes of data.
64 bytes from 180.97.33.108: icmp_seq=1 ttl=48 time=27.6 ms
64 bytes from 180.97.33.108: icmp_seq=2 ttl=48 time=27.6 ms
64 bytes from 180.97.33.108: icmp_seq=3 ttl=48 time=27.5 ms
64 bytes from 180.97.33.108: icmp_seq=4 ttl=48 time=27.6 ms

telnet 域名/IP 端口号
>test_user@10-10-80-30:~$ ttelnet www.baidu.com 80
Trying 61.135.169.125...
Connected to www.a.shifen.com.
Escape character is '^]'.

hosts 加入域名

>1.使用root
>sudo vi /etc/hosts
>按 “Inter”输入，
>按 “Esc”,":wq" 保存退出

##打包
tar格式（该格式仅仅打包，不压缩）

>打包：tar -cvf [目标文件名].tar [原文件名/目录名]
解包：tar -xvf [原文件名].tar
注：c参数代表create（创建），x参数代表extract（解包），v参数代表verbose（详细信息），f参数代表filename（文件名），所以f后必须接文件名。


##压缩

tar.gz格式

方式一：利用前面已经打包好的tar文件，直接用压缩命令。

>压缩：gzip [原文件名].tar
解压：gunzip [原文件名].tar.gz

方式二：一次性打包并压缩、解压并解包

>打包并压缩： tar -zcvf [目标文件名].tar.gz [原文件名/目录名]
解压并解包： tar -zxvf [原文件名].tar.gz
注：z代表用gzip算法来压缩/解压。

打包并压缩当前目录下所有文件为 test.tgz   
>tar -zcvf test.tgz *

>单个文件压缩打包 tar czvf my.tar file1
多个文件压缩打包 tar czvf my.tar file1 file2,...
单个目录压缩打包 tar czvf my.tar dir1
多个目录压缩打包 tar czvf my.tar dir1 dir2
解包至当前目录：tar xzvf my.tar

##查看压缩文件中的文件
>tar -ztvf test.tgz

##解压缩
>tar -zxvf test.tgz 

##传包

>scp Alipay.jar 用户名@127.0.0.1:/usr/work/alipay

##拷贝
从其他目录拷贝文件到当前目录：
>先进入当前目录，然后：cp 源文件目录  .         
   (  .代表当前目录)

## 查看java进程
>ps -aux | grep java
或
ps -ef | grep java
(grep ——global search regular expression(RE) and print out the line 全文正则匹配并打印)
```txt
ps：将某个进程显示出来

-A 　显示所有程序。 

-e 　此参数的效果和指定"A"参数相同。

-f 　显示UID,PPIP,C与STIME栏位。 

grep命令是查找

中间的|是管道命令 是指ps命令与grep同时执行

UID PID PPID C STIME TTY TIME CMD

各相关信息的意义：

UID 程序被该 UID 所拥有

PID 就是这个程序的 ID 

PPID 则是其上级父程序的ID

C CPU 使用的资源百分比

STIME 系统启动时间

TTY 登入者的终端机位置

TIME 使用掉的 CPU 时间。

CMD 所下达的指令为何
```

## 后台方式启动

>nohup java -jar xxx.jar &

## [linux 打开文件数 too many open files 解决方法](http://blog.csdn.net/fdipzone/article/details/34588803)

###查看每个用户最大允许打开文件数量
>ulimit -a
```shell
[zhxdidi@web-mod md-consume]$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 172032
max locked memory       (kbytes, -l) 32
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 172032
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

其中 open files (-n) 1024   表示每个用户最大允许打开的文件数量是1024

###查看当前系统打开的文件数量
(list open files)是一个列出当前系统打开文件的工具

>lsof | wc -l  &nbsp;&nbsp;&nbsp; &nbsp; 或  
watch "lsof | wc -l" &nbsp;&nbsp;&nbsp;&nbsp; 每2秒监控

###查看某一进程的打开文件数量
>lsof -p pid | wc -l  &nbsp;&nbsp;&nbsp;&nbsp; pid 为进程号

###设置open files数值方法
方式一:临时
>ulimit -n 2048

方式二：永久
>vim /etc/security/limits.conf  
>在最后加入  
>\* soft nofile 4096  
>\* hard nofile 4096  
最前的 * 表示所有用户，可根据需要设置某一用户
>重启系统，通过shutdown -r now  命令重启即可。

##开端口
使用root
>1.查看状态
iptables -L -n
2.6016为端口号
/sbin/iptables -I INPUT -p tcp --dport 6016 -j ACCEPT
3.保存命令
/etc/rc.d/init.d/iptables save
4.查看状态
/etc/init.d/iptables status

###查看某一端口的占用情况
>lsof -i:端口号

###查看所有端口情况
>netstat -antup

###显示tcp，udp的端口和进程等相关情况
>netstat -tunlp

###查看指定端口号的进程情况
>netstat -tunlp|grep 端口号

###通过进程号查看端口
>netstat -antup|grep PID号
##查看文件大小
ls
>ls  -lht     
h:易于阅读的形式显示
t:时间倒序显示

##使用curl获取HTTP头信息
>curl -I http://www.example.com

##新增用户
1.新增用户 username
>sudo useradd username

2.给用户username设置密码：
>sudo passwd username
>
>useradd命令添加用户，则默认用户名和组都是相同的名称；
/etc/passwd文件纪录的系统所有用户列表；
/etc/group文件纪录系统的组信息。

3.将用户添加到wheel组，否则无法使用sudo 命令
>sudo usermod -G wheel username

4.查看用户所属组
>groups

###禁止root远程登录
1.修改ssh配置文件：sudo vi /etc/sshd_config，将PermitRootLogin 设置为 no

2.重启ssh服务：sudo systemctl restart sshd

##Linux下rz/sz安装
>yum install lrzsz
或者
>cd tmp
 wget http://www.ohse.de/uwe/releases/lrzsz-0.12.20.tar.gz
tar zxvf lrzsz-0.12.20.tar.gz && cd lrzsz-0.12.20
 ./configure && make && make install

上面安装过程默认把lsz和lrz安装到了/usr/local/bin/目录下, 下面创建软链接, 并命名为rz/sz:

>cd /usr/bin
> ln -s /usr/local/bin/lrz rz
> ln -s /usr/local/bin/lsz sz

##命令行sftp
>格式：sftp <host>
通过sftp连接<host>，端口为默认的22，用户为Linux当前登录用户。
 >
>格式：sftp -oPort=<port> <host>
通过sftp连接<host>，指定端口<port>，用户为Linux当前登录用户。
 >
>格式：sftp <user>@<host>
通过sftp连接<host>，端口为默认的22，指定用户<user>。
 >
>格式：sftp -oPort=<port> <user>@<host>
通过sftp连接<host>，端口为<port>，用户为<user>。

pwd   查看远程服务器当前目录；
lpwd  查看本地系统的当前目录。
cd <dir>   将远程服务器的当前目录更改为<dir>；
lcd <dir>  将本地系统的当前目录更改为<dir>。
ls 显示远程服务器上当前目录的文件名；
ls -l  显示远程服务器上当前目录的文件详细列表；
ls <pattern> 显示远程服务器上符合指定模式<pattern>的文件名；
ls -l <pattern>  显示远程服务器上符合指定模式<pattern>的文件详细列表。
lls 显示本地系统上当前目录的文件名；
lls的其他参数与ls命令的类似。
get <file> 下载指定文件<file>；
get <pattern> 下载符合指定模式<pattern>的文件。
put <file> 上传指定文件<file>；
get <pattern> 上传符合指定模式<pattern>的文件。
progress 切换是否显示文件传输进度。
mkdir <dir> 在远程服务器上创建目录；
lmkdir <dir> 在本地系统上创建目录。
exit/quit/bye 退出sftp。
! 启动一个本地shell。
! <commandline> 执行本地命令行。

##命令Crontab
linux自带的定时任务

### 查看是否安装了crontab

>rpm -qa | grep crontab

![image.png](http://upload-images.jianshu.io/upload_images/626005-f5231df750394b70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 服务启动、关闭
>启动：/etc/init.d/crond start
关闭：/etc/init.d/crond stop
重启：/etc/init.d/crond restart
重新载入配置：/etc/init.d/crond reload

### 配置文件
在etc/目录下存在cron.hourly,cron.daily,cron.weekly,cron.monthly,cron.d五个目录和crontab,cron.deny二个文件

### 查看cron
>Crontab -l

### 编辑
>Crontab  -e

##查看连接状态
###各种连接：
>netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

```
解释:

返回结果示例： 
LAST_ACK 5  (正在等待处理的请求数) 
SYN_RECV 30 
ESTABLISHED 1597 (正常数据传输状态) 
FIN_WAIT1 51 
FIN_WAIT2 504 
TIME_WAIT 1057 (处理完毕，等待超时结束的请求数) 
 
状态：描述 
CLOSED：无连接是活动的或正在进行 
LISTEN：服务器在等待进入呼叫 
SYN_RECV：一个连接请求已经到达，等待确认 
SYN_SENT：应用已经开始，打开一个连接 
ESTABLISHED：正常数据传输状态 
FIN_WAIT1：应用说它已经完成 
FIN_WAIT2：另一边已同意释放 
ITMED_WAIT：等待所有分组死掉 
CLOSING：两边同时尝试关闭 
TIME_WAIT：另一边已初始化一个释放 
LAST_ACK：等待所有分组死掉
```

###并发连接数：
>netstat -nat|grep ESTABLISHED|wc -l

###查看某个端口的连接数
>netstat -nat | grep -iw "9090" | wc -l

###查看连接状况
>netstat -nat | grep -iw "9090"

###查看操作系统最大连接数：
>ulimit -a

##系统部分
###查看Linux系统位数
方法一：
>file /sbin/init 或者 file /bin/ls

![file /sbin/initpng](http://upload-images.jianshu.io/upload_images/626005-f243ffc5eb274ea1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![file /bin/ls.png](http://upload-images.jianshu.io/upload_images/626005-04a4c5a67036be5b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


方法二：
>uname -a

![uname -a.png](http://upload-images.jianshu.io/upload_images/626005-3bfc0f3d391c0e9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

x86_64表示64位，
i386表示32位

方法三：
>getconf LONG_BIT

![getconf LONG_BIT.png](http://upload-images.jianshu.io/upload_images/626005-dce74a9569802777.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###查看磁盘大小
>df -h

![df -h.png](http://upload-images.jianshu.io/upload_images/626005-70030b3c24d52304.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


df -hl 查看磁盘剩余空间

df -h 查看每个根路径的分区大小

du -sh [目录名] 返回该目录的大小

du -sm [文件夹] 返回该文件夹总M数

更多功能可以输入一下命令查看：

df --help

du --help

###查看内存大小
>free -m  按M计算

![free -m .png](http://upload-images.jianshu.io/upload_images/626005-b4e31034846318bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###查看CPU使用率
实时CPU使用率：
>top

![top.png](http://upload-images.jianshu.io/upload_images/626005-b7be603a49e5b8f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


CPU处理器使用率:
>cat  /proc/stat cpu

![cat  /proc/stat cpu.png](http://upload-images.jianshu.io/upload_images/626005-e6e85d369d3e3162.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


平均CPU使用率:
>cat  /proc/loadavg

![cat  /proc/loadavg.png](http://upload-images.jianshu.io/upload_images/626005-8e3b466d0151f7b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###查看CPU基本信息
>cat /proc/cpuinfo

![cat /proc/cpuinfo.png](http://upload-images.jianshu.io/upload_images/626005-5a3cc5a76870f6f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###查看内核数
方法一：
>cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c

![11.png](http://upload-images.jianshu.io/upload_images/626005-20d5474ee6110f4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


方法二：
>cat /proc/cpuinfo |grep "processor"|wc -l

![12.png](http://upload-images.jianshu.io/upload_images/626005-f92cb77094e72318.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

