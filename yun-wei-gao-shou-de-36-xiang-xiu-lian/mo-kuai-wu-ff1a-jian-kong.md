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

了解了几个核心组件后，我们来看一下 Prometheus 这些组件所构成的 Promethues 整体监控架构。

CgqCHl7Dn2SAX4_sAAEeHG2vN7E056.png

在这张图里我们可以看到，中间的位置是 Prometheus server 核心组件，这个核心组件串联 Prometheus 整体监控流程。在 Prometheus server 这个大框里面有几部分：一个是收集模块（Retreval），另外一部分就是 TSDB，这是一个时序的数据库，HTTP server 是一个提供给外部访问的服务接口。

我们先来看收集模块，这个模块上面的“ discover targets”-代表去发现监控节点目标， Prometheus 一个非常重要的特性就是完美支持 K8s 对于 Docker 容器编排的服务架构，它能够动态地发现 K8s 集群里面目标节点pod ，发现并提取所有节点的监控信息，通过 pull metrics 针对性地提取具体每一个节点 pod 上面的监控数据。

接下来我们看到图中的左侧，它展示了发现目标节点信息以后，具体提取这些节点上面的监控数据的方式。我们看到第一种方式就是通过 exporters 组件去进行节点数据采集，并标准化成一个通用的 metric 接口，给到 Prometheus server 的服务端来进行抓取。

第二种方式是：Prometheus server 抓取一个 Push gateway（推送网关），客户端推送信息传递到推送网关，通过推送网关来进行转换，然后服务端再去拉取网关的数据，这样相当于Push gateway 做了一层转换。

收取监控数据以后，Prometheus server 会存到自己的时序数据库里。存取到数据库以后，通过 Prometheus server 端的运算、转换，逻辑处理，最后提供给对外部HTTP 接口进行调用访问。

如果触发了一些报警的规则，则需要把这些消息的报警规则推送给 Alertmanager 进行处理，可以对外发送邮件，提醒信息等行为，这些行为都是由 Alertmanager 组件负责实现的。

另外就是可视化展示了，可视化的展示需要集成一些可视化插件。刚刚讲到的 Grafana 就属于可视化的插件，可以从 Prometheus server 里进行数据提取和展示。除了 Grafana 以外，还有一个是 Prometheus web UI，这是 Prometheus 默认自带的一个可视化管理后台。

以上就是基于 K8s 服务架构下，Prometheus 组件的整体架构。

了解了这些组件贯穿的架构图示之后，我们接下来会重点讲解 Prometheus server、exporters、Grafana 这三个组件，为你演示从数据的采集到 Prometheus 服务端抓取 Grafana，也就是数据的监控信息展示报表，这个流程架构是如何实现的。

**Prometheus 和 Zabbix 功能、使用差异**

在介绍整体架构之前，想必你对我们整个模块里讲到的两个监控系统：Zabbix 和 Prometheus ，那它们有什么区别？在这里我列了一张表格，分别从开发语言成熟度、性能、社区活跃度以及容器 K8s 微服务的支持，还有部署复杂度以及监控配置的复杂度等维度来为你做了一个对比。

CgqCHl7Dn5yAKlxWAACmu5VzQHw993.png

它们之间的不同主要体现在以下几个方面：

首先 Zabbix 是由 PHP 开发的，而 Prometheus 是 GO 语言开发的。其次，从开发时间来看，Prometheus 比 Zabbix 晚出来很多年，没有太多复杂多余的逻辑，再次存储数据库的方式也不同（后面具体介绍），这些因素使得 Prometheus 在性能上会比 Zabbix 更优。

第二，正是由于 Zabbix 问世的时间更早，所以它的代码成熟度会略高，而 Prometheus 由于出来的更晚，所以它可能会有代码需要加固。

第三，Prometheus 相对 Zabbix 而言，可能存在的另外一个劣势就是配置的复杂度更高，我们在上节课里面有讲到 Zabbix 的自动发现和自动上报，Zabbix的监控配置在控制台相对完善，而 Prometheus 则需要去进行很多配置项的和数据规则这样的运算配置，所以它的复杂度相对 Zabbix 而言会更高一些。

于其他维度分析，我们看到 Prometheus 均是优于 Zabbix，尤其是对于 K8s+容器的支持，而 Zabbix 到了很晚才去兼容支持 K8s 这种动态发现、编排容器的架构模式，所以当前很多 K8s 部署场景里的监控系统都会基于 Prometheus 来实现。

此外，再总结一些更多 Prometheus的几个特性：

1.Prometheus 采用了一种 Pull（拉）且 HTTP 的方式获取数据降低客户端的复杂度，服务端可以更加方便地水平扩展。
2.Prometheus 存储使用时序数据库 Zabbix 采用关系数据库保存，这极大限制了 Zabbix 采集的性能， Prometheus 自研一套高性能的时序数据库， 在 V3 版本可以达到每秒千万级别的数据存储。