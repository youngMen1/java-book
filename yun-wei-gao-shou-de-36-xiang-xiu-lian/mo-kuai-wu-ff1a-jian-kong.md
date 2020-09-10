# 第25课：容器监控实践：Prometheus、Grafana 方案介绍

本课时我们来学习：Prometheus，并结合组件 Grafana，整体介绍一套监控方案。

## Prometheus 介绍

首先来介绍 Prometheus，可能你对它的了解相比 Zabbix 会更陌生一些，Prometheus 是一套由 GO语言开发的开源监控系统。它是继 Kubernetes 之后的第二个 CNCF 托管项目，近些年被广泛使用在基于 K8s 或 Swarm 这种容器编排的整体服务平台中作为监控系统使用。它的官方网站是 https://prometheus.io/（如果想具体了解的话，你可以登录官方网站，去详细看它的帮助使用手册）。

Prometheus 是由一套组件所组成，接下来我们具体来介绍一下它的核心组件：

* 第 1 个组件是 Prometheus server（主要组件），它是一个核心组件，用于拉取监控数据并存储到时序数据库，相当于 Prometheus 整个监控系统里面的心脏。
* 第 2 个就是客户端 SDK ，用于植入监控的应用程序中，完成数据的采集。
* 第 3 个组件就是 Push Gateway，它是一个支持客户端使用主动推送监控数据的中间网关。刚讲到 Prometheus server 默认以拉取监控数据的方式来抓取客户端数据。当我们的客户端需要通过 push 主动推送时就需要 Push Gateway 组件来作中间转化。
* 第 4 个组件是 Exporter，它是 Prometheus 的一类数据采集组件的总称，负责从目标节点处搜集数据，并将其转化为 Prometheus 支持的抓取的格式，这提供了一个统一形式的接口供服务端抓取。
* 第 5 个组件是 Alertmanager，警告管理器，在监控系统中负责报警模块，所有 Prometheus 发出的报警都是通过它进行处理和触发告警消息的。