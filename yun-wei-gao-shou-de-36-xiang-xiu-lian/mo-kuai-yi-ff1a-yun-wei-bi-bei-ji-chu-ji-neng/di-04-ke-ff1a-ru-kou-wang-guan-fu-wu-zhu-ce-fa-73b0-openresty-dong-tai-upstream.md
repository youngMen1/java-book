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