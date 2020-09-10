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

我们看看具体的配置：



```
og_format  main  '$remote_addr - $remote_user [$time_local] "$request" '

                      '$status $body_bytes_sent "$http_referer" '

                      '"$http_user_agent" "$http_x_forwarded_for"';

```
示例中通过 log_format 定义了一个名称“ main”，在 access_log 的配置选项里面，如果引用mian，日志的内容就会按照 main 的定义方式来打印具体内容。

具体看下这里的配置，整体上最大的外层是通过一个单引号，我们会看到第一段单引号归纳的内容，一个是 remote_addr，它是 Nginx 的内置变量，记录的是直接请求客户端的ip地址。然后通过一个 “-” 符号，和 remote_user 也就是请求客户端的用户做了分隔，然后加入了一个括号，括号里面记录用户请求到 Nginx 上面的时间，request 就是用户请求的内容。

$status 主要记录的是 Nginx 返回给客户端的状态码，我们看到有 200、300、400、500 等相关的 HTTP 状态码。body_bytes_sent 是服务端向客户端发送的 body 的字节大小。$http_referer 主要记录客户端请求地址的上一跳的地址。

$http_user_agent 它记录用户请求时用的是哪个客户端。$http_x_forwarded_for 之前也讲过，记录的是用户的 IP 和 Proxy 的 IP 头信息。

所有的格式内容默认以空格来进行分隔，除非我们自定义加入一些分隔符，比如“ / ”和“ [] ”都会在打印的格式里面进行打印。最终打印的内容会是这样的：



```
47.105.45.235 - - [01/Apr/2020:10:40:03 +0800] "GET /jeson/ HTTP/1.1" 200 17460 "-" "Chrome/57" "-”

```
这里我列出了一个具体的内容，前面我们会看到 IP 地址（47.105.45.235），对应的是 $remote_addr，在后面有两个横杠，其中一个横杠"-"表示获取的内容为空（它是$remote_user）另外一个横杠就是我们刚刚加入的自定义的分隔符。后面就是具体的请求时间、方法、路径和协议，然后就是请求返回的状态发送的字节数。后面我们会同样看到 http_referer 内容为空，它说明用户是直接请求路径 /jeson，所以它没有携带 referer 信息。后面的 Chrome/57 说的是客户端的 Agent 版本是用的 Chrome 浏览器。

这个内容就是我们刚刚定义好的日志格式("main")所打印出来的一一对应的内容。

# 分析 Nginx 日志常见场景

Nginx access_log 内容，我们常常要去做对应的分析，主要的场景如下：。

## 访问量统计

首先，就是需要我们去了解 Nginx 服务的访问量，通常我们对于访问量统计有这么几大类：

### 1、总访问量频繁的 IP

第一大类就是对于最频繁访问的 IP，了解哪一个 IP 可能频繁访问并对服务端造成的请求压力比较大，所以我们就需要分析 Nginx 打印出来的 access 日志基于 IP 的统计排名。这时我们可以通过这样的一段命令来进行分析：



```
cat access.log|awk '{print $1}'|sort |uniq -c |sort -n -k 1 -r|more

```

通过 cat 读取整体的 access.log，然后通过 awk 进切割，只打印第一列的内容（就是 IP 的信息），后面加一个排序，这个排序的作用就是把所有的 IP 进行排序，u niq -c 就是整体地按照同样的 IP 来进行统计，并且分析出同一个 IP 请求的个数，最后按照个数进行由大到小的排序 sort -n -k 1 ，-k 1 表示第一列按照请求的个数来进行排序。就能够完整地分析出请求的 Nginx 服务端里面哪一些 IP 排在最高，从上往下的排序结果如下：



```
342 221.219.98.129
138 120.26.213.206
69 221.220.172.233
46 47.94.196.61

```

最上面的是请求最多的，第一列就是这个 IP 请求的个数。

### 2、查看某个 IP 访问量频繁 URL

第二个常常要做的就是 Nginx 访问量统计，既然知道哪个 IP 访问比较多，就想继续了解某一个 IP 的行为，比如查看某个 IP 访问最频繁的 url。



```
cat access.log|grep '221.219.98.129'|awk '{print $7}'|sort |uniq -c |sort -n -k 1 -r|more

    114 /videotech/getip/

    114 /videotech/getpages

    114 /jeson

```
这时我们就可以通过 grep 把 access.log 按照某一个 IP 做一个筛选，然后把第 7 列 $reques（也就是请求路径等情况）打印出来。然后再进行一个同样的排序，我们会看到这个 IP 主要请求的是哪些地址，和它们对应的请求次数。

### 3、更多场景
当然还有更多的场景：

**（1）查看爬虫、机器人访问**

比如我们想要去分析这个 IP 或者某一些请求是否可能有爬虫，或者有一些通过脚本类似机器人的行为，因为通常这种行为对于服务端可能会造成请求流量的影响，所以需要通过 access.log 来进行分析来排查这些行为。

通常第一类可以通过组合命令的方式来做：



```
cat access.log|grep -iv "MSIE|Firefox|Chrome|Opera|Safari|Gecko|Mozilla|wordpress"

```

cat 一个日志以后分析 grep -iv，i 表示查看的内容，里面是不区分大小写的；-v 是做一个反向的排查，也就是把正常浏览器通过 Agent (MSIE|Firefox|Chrome|Opera|Safari|Gecko|Mozilla|wordpress)端进行访问全部做一个反向的排查（看到通过非正常的 Agent 所转发的请求）。这样就可以判断有没有可能是机器人或爬虫的请求，以及大概的占比以及它们的行为。

**（2）过滤没有 Agent 的请求**

后面介绍的组合命令就是分析没有带 Agent 的请求。有可能一些请求是没有把 Agent 带过来的，我们需要关注并分析。



```
cat access.log|awk '{if($11~"-"){print $0}}’

```
我们可以通过 awk 分析指定的列，然后看是否拿到了字符横杠，因为横杠“-”代表 access log 没有拿到对应内容，所以可以判断 Agent对应的一列进行判断，看它的内容是否为"-"，如果是就通过 print $0，把没有 Agent 的每一条请求打印出来。
## 错误性能统计
作为运维工程师，我们还要分析的第二大类型，就是 Nginx 的错误日志请求。
**（1）状态码响应统计**
比如状态码的错误，我们知道 200 是正常状态码，300 主要是重定向类型的状态码，400/500 说明对客户的请求可能存在问题，通常需要进行整体的分析。



```
cat access.log|awk '{print $9}'|sort|uniq -c|more

    922 200
    235 301
      1   404
     10  500

```

这样的组合命令通过 cat access.log，然后 awk 处理，并把 $9（第九列） 打印出来，$9 在 Nginx 配置的 log_format 中代表服务端所返回的状态码，并且基于状态码做了排序。

可以看到，访问网站的大部分状态码是 200，以及 301 有多少个，404 有多少个，500 有多少个，然后就了解整体的用户对网站的请求和中间的错误率大概是什么样子的。
**（2）请求延时分析**
除了对错误码的分析，我们也需要去关注性能上的问题，比如请求的延时分析。在 Nginx 的 access.log 里面主要有两个变量，一个是 request_time，还有一个是 upstream_response_time。


```
$request_time:

```
指的就是从接受用户请求的第一个字节到发送完响应数据的时间，即包括接收请求数据时间、程序响应时间、输出响应数据时间。


```
$upstream_response_time

```
是指 Nginx 发出以后，到接到后端 real server 后端的服务，再给到 Nginx 的时间，upstream_response_time 主要用于反向代理模式里面所需要记录的时间，记录的是反向代理发出请求到拿到后端(Real server)给到反向代理的数据的时间。



```
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '

                      '$status $body_bytes_sent "$http_referer" '

                      ‘“$http_user_agent” “$http_x_forwarded_for” $upstream_response_time';

```
所以在整个过程中，如果想要了解每一个请求的延迟情况，可以看到通过 log_forma t 来定义“main” 预定格式，这里可以在 main 的配置里最后加入一个 $upstream_response_time 变量，然后 reload(重启) Nginx，Nginx 的 access.log 里就可以把 upstream_response_time 时间记录下来了。


```
tail -f access.log|awk '{if($(NF)>6){print $0}}'

```

当用户请求 Nginx时，通过如上这个命令我来判断后端服务的响应延迟情况，通过这样的组合命令保持对文件的实时监听，了解每一行最新请求情况，并通过 awk 进行过滤，awk 语句中做一个条件判断，判断最后一行 upstream_response_time 数值的内容大于 6 秒时进行打印（表示打印后端响应整体大于 6 秒的请求），这个方式有助于运维和开发人员去分析每一个请求的具体请求延迟上的一些问题。这就是对于性能错误上的分析。

## 安全分析统计
还有就是对于安全的统计分析了，安全统计分析在 Nginx log 里面也是非常重要的，这里我列出了如下内容：

**（1）分析请求中存在的敏感 SQL 语句**
首先，就是对敏感词的 SQL 语句进行分析，我们知道 SQL 攻击是一种常见的攻击手段，关注这一类的攻击，我们在 Nginx access 请求日志里面需要去关心客户端的请求里面有没有携带敏感词，包括 MySQL 里常用的查询敏感的语句（具体可以关注 27 课时），如“selete”“and”“+and+”“ and ”这些关键词，我们需要对 access.log 进行分析，所以可以基于这些关键词来进行筛选，我们会看到这里有三条组合命令：



```
cat access.log|awk '/select/{print $1}'|sort -n|uniq -c|sort -nr

cat access.log|awk '/\/and\//||/\+and\+/||/%20and%20and/{print $1}'|sort -n|uniq -c|sort -nr

cat access.log|awk '/sleep/{print $1}'|sort -n|uniq -c|sort -nr

```

第一个是通过 awk 来做敏感词的筛选，这里只筛选 select 敏感词，把这个请求打印出来并且进行排序。
第二个关注的是“and”敏感词，然后把这些请求打印出来。
第三个是 sleep，sleep 是黑客经常用到的一个攻击手段。比如在注入成功以后，用户就会进行一个 sleep，使得后面的 MySQL 响应延迟变得更长。我们就可以通过 awk 命令在 access.log 里面直接进行关键词的筛选，并且把这个请求打印出来，看有没有这样的攻击行为。

**（2）分析请求中存在的敏感 Shell 命令**
第二类关注的是请求里面有没有携带敏感的 Shell 危险命令，比如这里我列出的 "cat /etc/passwd" 文件。



```
cat access.log|awk '/\/etc\/passwd/{print $1}'|sort -n|uniq -c|sort –nr

```

我们知道操作系统如果在请求里面有带“etc passwd”关键词，说明这个请求十分不正常。这类请求我们通常也是需要去进行关注和分析的。

这就是 Nginx 中我们通常需要分析的几大块内容，一个是性能，一个是统计访问情况，还有就是安全类别。

# 解析 MySQL 日志
接下来给你介绍的就是 MySQL 的日志。

## MySQL 日志类型
MySQL 日志相对来说更加简单，它只有几个类型，分别是 error_log、slow_log、binlog 和查询日志。

Ciqah16ipcmAR_TTAAKv0YH9EQw405.png