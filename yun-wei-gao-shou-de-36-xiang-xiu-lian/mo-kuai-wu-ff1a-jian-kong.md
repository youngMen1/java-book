# 第25课：容器监控实践：Prometheus、Grafana 方案介绍

本课时我们来学习：Prometheus，并结合组件 Grafana，整体介绍一套监控方案。

## Prometheus 介绍

首先来介绍 Prometheus，可能你对它的了解相比 Zabbix 会更陌生一些，Prometheus 是一套由 GO语言开发的开源监控系统。它是继 Kubernetes 之后的第二个 CNCF 托管项目，近些年被广泛使用在基于 K8s 或 Swarm 这种容器编排的整体服务平台中作为监控系统使用。它的官方网站是 https://prometheus.io/（如果想具体了解的话，你可以登录官方网站，去详细看它的帮助使用手册）。

Prometheus 是由一套组件所组成，接下来我们具体来介绍一下它的核心组件：