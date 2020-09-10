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