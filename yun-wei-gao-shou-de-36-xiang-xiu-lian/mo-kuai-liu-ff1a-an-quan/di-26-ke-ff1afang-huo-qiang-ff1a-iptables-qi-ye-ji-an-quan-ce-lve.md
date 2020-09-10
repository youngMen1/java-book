# 第26课：防火墙：iptables 企业级安全策略

本课时我们来学习安全模块：iptables 企业级安全策略。

## 什么是 iptables

说到 iptables，如果你用过 Linux 会比较了解，它是 Linux 上的一套防火墙服务，调用的其实是 Netfilter 内核模块。Netfilter 是 Linux 操作系统核心层内部的一个数据包处理模块，它具有如下功能：
1.网络地址转换；
2.数据包内容修改；
3.数据包过滤防火墙功能。

在现实中，其实很多应用服务都调用了 Netfilter。这里我给你画了一张图：

