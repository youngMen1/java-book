# 第26课：防火墙：iptables 企业级安全策略

本课时我们来学习安全模块：iptables 企业级安全策略。

## 什么是 iptables

说到 iptables，如果你用过 Linux 会比较了解，它是 Linux 上的一套防火墙服务，调用的其实是 Netfilter 内核模块。Netfilter 是 Linux 操作系统核心层内部的一个数据包处理模块，它具有如下功能：
1.网络地址转换；
2.数据包内容修改；
3.数据包过滤防火墙功能。

在现实中，其实很多应用服务都调用了 Netfilter。这里我给你画了一张图：

CgqCHl7GXLmAQNIJAAAx_JTgoQQ756.png

可以看到，在最中央就是 Netfilter 的 Linux 内核模块了，我们看到最下层 iptables 会调用 Netfilter 模块来作为安全防火墙。在 CentOS7 以后，出来一款新的防火墙服务叫作 firewalld，其实也是调用了 Netfilter 模块。总体来说，无论是 iptables，还是 firewalld，它们都是调用了 Linux 的 Netfilter 内核模块。

我们再来看图片左边，这里有一个 ipvs 应用工具，它也调用了 Netfilter，但是它的功能和安全防护的防火墙功能有一些区别，它主要用来实现地址转换这样的一个核心功能，所以 ipvs 会为很多负载均衡提供服务，也就是为 LVS 提供服务，如果你了解过 LVS 就会清楚，其实 LVS 就是用到了 ipvs 的命令模块来对数据包规则进行设置。所以 ipvs 的上一层会供 LVS 调用，它在运维管理中可以作为防火墙服务来使用。所以我们会看到 ipvs 是作为负载均衡的服务来使用的，可以总结为负载均衡的服务其实也是调用了 Netfilter 内核模块。

除此之外，我们在这张图的左下角还会看到一个 Kube-proxy 命令模块，它也调用了 ipvs，同时也调用 iptables。其实 Kube-proxy 是 K8s 里的一个组件，负责管理 K8s 里的 service 模块，也就是后端服务，管理后端服务需要对数据包进行转发，并实现负载均衡。我们会看到在 K8s 整套模块的组件里，Kube-proxy 其实也调用了 ipvs，同时基于 ipvs 去调用 Linux 内核的 Netfilter，那么为什么 Kube-proxy 还要去调用 iptables？这是因为早期的 Kube-proxy 版本是基于 iptables 实现的，但是这里有一个性能问题，因为 iptables 毕竟不是专门用作数据包转发的，所以在 service 高于 1000 个的情况下，性能表现上 ipvs 优于 iptables。所以就逐渐使用了 ipvs 服务来代替 iptables 工具。

在这张图里，我们会看到当前整个 Netfilter 在应用平台服务里面，起到的至关性作用。而 iptables 发布的其实是两款专业的防火墙工具，那么对于 iptables 和 firewalld，这两个防火墙服务之间有两个比较大的差异，这里给你来介绍一下。


