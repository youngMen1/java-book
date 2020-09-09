# 第04课：入口网关服务注册发现-Openresty 动态 upstream
本课时，我将带你一起了解入口网关服务的注册发现，并使用 OpenResty 实现一套动态 Upstream。

## 课前学习提示

基于本课时我们将要学习的内容，我建议你课前先了解一下 Nginx 的基础，同时熟悉基础的 Lua 语言语法，另外再回顾一下 HTTP 的请求过程，对于 Nginx  的负载均衡基本原理也要有基础的了解，掌握这些对我们学习此课时能起到一定的帮助。



关于本课时的内容，我用思维导图来先给你做一个整体的介绍：



首先，我会讲解动态 Upstream 的实现意义和场景，以及企业常见的基于开源实现方式。同时还会摘选其中一种实现方式，也就是基于 OpenResty 实现动态 Upstream 的案例进行演示。

## 动态 Upstream 场景

首先，我们来讲解第一部分内容，也就是动态 Upstream 的场景。

CgpOIF5XlqGAJ1iwAAE2ca-U8Go037.png

我们先来看一下这样一个典型的场景，整体了解动态 Upstream 的作用。



从图中流量的整体入口开始看，流量从左上方按照箭头方向访问站点，最先交给第一级服务 Nginx，Nginx 承担入口网关的作用，通常情况下，通过 Nginx Upstream来作负载均衡，从而实现将入口的流量均匀地分配给后台的 real server处理。



从图中我们可以看到，右上角的方框中有两台 Real server ，分别是 App server 1 和 App server 2。



那我们需要的动态 Upstream 是一个什么场景呢？当往这组集群中再添加一台新的后台服务 real server，也就是 App server 3 ，并且实现将入口的流量动态的(非手动方式)分发给新 App server 3，这个时候我们就需要用到动态配置upstream了。

## 动态 Upstream 实现意义

那么实现动态 Upstream 有什么好处呢？



再结合在上面的场景，要实现动态upsteam，首先后台服务节点App server 1 和 App server 2 ，它们将自己的信息(如：负载情况、服务状态情况、连接信息、服务名称等)实时的上报给管理中心，使得管理中心会收集 App server 1 和 App server 2 的节点状态，同时管理中心也会根据自身的策略动态地调整 Nginx 的 Upstream 配置。



假如当某一个Real server的负载过高，此时便可以通过管理中心去动态地调整 Nginx 的 Upstream 策略，入口网关动态地添加新的节点从而实时性的实现Real server资源水平的扩容，反之实现动态减少后台节点也是同样原理。这样的场景中作到及时调整upstream策略，从而使得入口网关动态分流，实现及时资源缩容和故障感知。



这个场景中，如果upstream配置不是通过动态的方式去实现调整，就需要通过手动的方式（先修改配置，再手工重启服务等），手动的方式没法作到自动化及实时触发调整。


Cgq2xl5XlrOAW-n0AAFbOCQssio629.png


我们再看一个常见的 K8s 的入口网关架构，这里就使用了动态 Upstream。我们知道 K8s + Docker 的服务部署模式下，ETCD 作为服务注册发现中的 KV存储数据库，存取着后台节点服务的注册信息，包含服务名称、连接方式等。所以，在图中会把所有后台的 服务节点(App1-App4) 的信息注册到 ETCD 的数据库中存储。然后 入口网关(如：Nginx) 通过查看etcd中数据并根据变化情况动态地调整 Upstream 配置，这个就是在K8S中动态入口网关理论原理。



图中上面的箭头表示一个服务的注册和动态的 Upstream 更新配置文件。下面的箭头表示动态的实现负载均衡策略流量转发，这个就是在 K8s 和 Docker 的部署场景中入口网关的实现原理。

## 动态 Upsteam 实现方式

那么，动态 Upstream 的实现方式都有哪些呢？下面我给你具体讲解一下。



我们知道Nginx 本身诞生的比 K8s 和 Docker 更早，Nginx 默认的配置模块是有来实现动态Upstream 是有局限的，这也是上一讲中我所认为Nginx存在缺陷，所以我们想要实现，通常有这么几种方式：

CgpOIF5XlsKAFA4dAACi4m6_YDE387.png

第一种方式是通过 Nginx+Lua，使用 Lua 语言开发接口，做到动态调用，也就是基于 Openresty 动态实现 Upstream，因为我们知道 Openresty 服务本身就是基于 Nginx 和 Lua 的一个结合体。



第二种方式是通过已经开源并成熟的组件，如 confd 工具或 国内某大公司开源的 nginx-upsync-module 的模块来实现动态 Upstream。



第三种方式是通过一个独立的入口网关把 Nginx 完全替换掉，比如我们现在了解的 Kong 网关还有 Traefik，都是在 K8s 中支持云原生的非常好的网关服务。



那我总结的实现方式有以上三种，接下来本课时我们来演示 Openresty 动态 Upstream 方案，使用 Lua 语言开发一套动态的接口，我们可以动态地调用接口实现一个动态的分流。



这个方案中实现了3个接口，让 Openresty 支持添加、删除、监控 Upstream 后台节点的功能（如图接口1-接口3）。



我们看下如图描述的内容：
Cgq2xl5Xls6AJ57nAAEo03hFxpQ626.png


通过开放接口，管理中心或者配置中心用可以调用这三个接口动态修改入口网关的负载均衡配置。我们接下来重点介绍Openresty Lua 语言如何开发实现这三个接口。



在正式演示之前再带你熟悉一下 Openresty。Openresty 是一个基于 Nginx 与 Lua 开发的高性能web平台，内部集成了大量的 Lua 库和第三方模块，如果使用Nginx则需要通过 Nginx 编译 Lua 模块非常繁琐，所以需要支持LUA直接使用 Openresty 就可以了。



大家用浏览器打开 Openresty 的官方网站（https://openresty.org/cn/），你可以看到 Openresty 的官方文档，同时因为本课时会作 Openresty 的演示，如果你希望模拟安装，可以按照官网的方式来进行安装。



同样，接下来演示案例也用到了Nginx模拟后台服务，Nginx 的安装建议你参考官方的安装方式，这里可以通过 官方提供yum 源的方式来安装 Nginx。

## 演示

接下来演示这个动态 Upstream， 分为三大部分进行讲解。

* 第一部分是安装流程；

* 第二部分是配置方式；

* 第三部分是测试;

