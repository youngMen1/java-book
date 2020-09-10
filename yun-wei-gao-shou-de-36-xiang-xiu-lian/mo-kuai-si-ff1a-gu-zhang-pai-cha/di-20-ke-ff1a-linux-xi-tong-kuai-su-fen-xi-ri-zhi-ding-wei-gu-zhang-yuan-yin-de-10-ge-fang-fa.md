# 第20课：Linux 系统快速分析日志定位故障原因的 10 个方法

本课时我们主要是学习如何通过 Linux 组合命令来做日志的快速定位和分析，它的使用场景主要是：

1.企业在没有可用的日志检索分析系统（如：EFK、ELK 等），需要通过 Shell 或 Linux 组合命令去进行日志分析；
2.企业可能已有日志分析和检索系统，但还是避免不了临时性日志分析问题，如处理紧急的故障场景，就需要用到 Linux 组合命令来进行细致的排查。

## 网站架构图示

可以说， Linux 系统上存储的日志类型非常多：

Ciqah16ipEeAGWevAAbzAGpV_r4743.png

从入口层、逻辑层、数据层到基础建设和公共组建层，基本上所有的日志都可以存储在 Linux 操作系统上。

即使是基建部分，通过收集抓取系统硬件底层的控制器信息或者网络上的设备信息，并存储在操作系统上的。

日志的分析类型、种类非常多，本课时我提取出两个应用来进行详细介绍，一个是入口层（常用的代理服务 Nginx），第二个是数据层（如常用 MySQL 数据库） 。

## 学习基础

学习本课时，你只需要了解 Nginx 操作基础，并且了解 MySQL 的使用基础。

# 解析 Nginx 日志

在本课时的内容里，首先给你介绍的就是 Nginx 的日志。

## Nginx 的日志类型

说到 Nginx 日志，我们必须要了解 Nginx 的日志类型，主要分为两个类型：

**第一个是处理请求的日志，记录了用户相关的访问信息。它的配置模块叫作 access_log，配置格式如下：**



```
Syntax:
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log off;

```

access_log 后面加 path （日志路径）和相关的选项。如果我们想关闭这个请求日志，配置为：access_log off，通过这个配置把请求日志关闭。


默认配置格式如下：

```
Default: access_log logs/access.log combined;

```


access_log 后面加了默认日志路径（logs/access.log），然后加上日志类型的名称（combined）。什么是日志类型的名称呢？接下来我会给你做一个详细的介绍。

然后是，下面的 Context 段落信息，它代表 access_log 这个配置模块可以在 Nginx。conf 配置的哪一层级进行配置。


```
Context: http, server, location, if in location, limit_except

```

我们会看到 access_log 配置允许配置的层级，在 Nginx conf 里配置到 http、server、location 等层级中。

如果需要自定义配置access_log， 需要具体配置相关的选项，如下：

* path：指定路径，即日志的存放位置。
* format：定义日志的格式。默认使用预定义的 combined（combined 就是一个默认的预定义格式的名称），如果想要修改 Nginx 的 access_log 记录的内容及格式，就需要通过 log_format （另外一个配置项来进行格式的预定义，并通过 access_log这个选项引用 log_format 的预定义格式的名称）。
* buffer：用来指定日志写入时的缓存区大小。默认是 64k。
* gzip：设置日志写入前先进行压缩，压缩比越高，消耗也越大；压缩比越小，占用的空间就会越多。
* flush：设置日志缓存的有效时间。
* if：条件判断。也就是需要满足什么条件才记录访问日志，比如我们可以只把请求等于 200 的条件的日志来做记录。


这就是 access_log 在 Nginx 服务里面的定义规则， access_log 是 Nginx 里面一个非常重要的日志，它记录了用户请求的相关内容。

**第二个重要的日志类型就是 Nginx 错误日志。**它主要用于分析 Nginx 的配置、Nginx 本身服务的问题，所以它记录的是 Nginx 进程启动、停止、重启及处理请求过程中发生的所有错误信息。它的配置方式是这样的：
Syntax: error_log file [level];
在 Nginx conf 里面，我们只需要将 error_log 加具体的文件路径，再加上日志级别，就可以进行错误日志的配置并打印输出。

它默认的配置样例是这样的：



```
Default: error_log logs/error.log error;

```
表示错误日志统一记录到 logs/error.log，并且日志级别是 error 级别，所有满足这个错误日志级别的都会输出到 error.log 里。

## Nginx 的日志内容

接下来我重点围绕 access_log 给你做介绍和分析。

我们刚讲到了，access_log 需要定义好日志内容和日志内容的格式，是通过 log_format 选项来提前定义的。log_format 也有对应的配置选项，比如：

* name：格式名称。格式名称是在 access 刚刚讲到的配置语法里面所需要引用的。
* escape：主要是用来定义字符的编码方式，可以是 JSON 格式，它默认使用的是 default 编码方式。
* string：十分重要，主要定义日志格式和对应内容，它是通过 Nginx 变量来做具体定义。
