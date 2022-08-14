---
layout: post
title: 线上问题排查技巧
categories: Java
description: some word here
keywords: 问题排查, 内存溢出
---
### 诡异问题
- 线程执行一个任务迟迟没有返回，应用假死。
- 接口响应缓慢，甚至请求超时。
- CPU 高负载运行。


### 定位问题
####    1. 查看所有 java 进程
使用命令 `jps -v` 或 `ps aux|grep java` 找到应用的 pid。

#### 2. 输出 dump 文件
使用命令 `jstack 1523 > 1523.log` ,其中 `1523` 为进程号。如果应用程序简单，日志不大的话可直接查看，但是若应用程序复杂，导出来的日志较大的话，建议使用专门的分析工具。

**分析工具**：

- 在线工具

  > [heaphero](https://heaphero.io/index.jsp)

- IBM Memory Analyzer

- Eclipse Memory Analysis(MAT)

  > **下载地址**：http://www.eclipse.org/mat/downloads.php
  >
  > **使用技巧**：
  >
  > 1.双击报错解决办法：
  > 右键mat显示包内容，进入Contents->MacOS下面，会有一个MemoryAnalyzer的命令。
  >
  > 打开终端，进入此路径找到MemoryAnalyzer，运行
  >
  > ./MemoryAnalyzer -data dump文件所在文件夹路径
  >
  > 即可启动成功
  >
  > 2.
  > 问题：
  > 使用mac版的时候，默认配置的最大内存是1g，当hprof文件过大时，一打开就提示内存溢出：java.lang.OutOfMemoryError: Java heap space
  >
  > 解决办法： 
  > 找到MemoryAnalyzer的安装目录，如果不知道在哪，可以在打开着的MAT里，点击Help->About Eclipse Memory Analyzer->左下角Installation Details->Configuration选项卡，找到luncher，对应的值就是该程序的路径。进入该文件的父目录的父目录同一层有个Eclipse目录，进去有个配置文件MemoryAnalyzer.ini，修改里边的Xms为：-Xmx4g，即改为最大4g内存。重启MAT即可。

### 建议

- 尽量不要在线程中做大量耗时的网络操作，如查询数据库（可以的话在一开始就将数据从从 DB 中查出准备好）。

- 尽可能的减少多线程竞争锁。可以将数据分段，各个线程分别读取。

- 多利用 `CAS+自旋` 的方式更新数据，减少锁的使用。

- 应用中加上 `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp` 参数，在内存溢出时至少可以拿到内存日志。

- 线程池监控。如线程池大小、队列大小、最大线程数等数据，可提前做好预估。

- JVM 监控，可以看到堆内存的涨幅趋势，GC 曲线等数据，也可以提前做好准备。

>参考：
- [一次线上问题排查所引发的思考](https://www.jianshu.com/p/7c84f1179167)
