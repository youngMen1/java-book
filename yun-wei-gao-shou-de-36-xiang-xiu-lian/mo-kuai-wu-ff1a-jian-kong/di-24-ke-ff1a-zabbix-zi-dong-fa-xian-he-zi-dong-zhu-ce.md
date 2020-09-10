# 第24课：Zabbix 自动发现和自动注册

这个课时我们来学习 Zabbix 自动发现和自动注册。

## Zabbix 及其优势介绍

你应该对 Zabbix 这个主流开源企业级监控系统并不陌生，它的官方网站是 [https://www.zabbix.com/。Zabbix](https://www.zabbix.com/。Zabbix) 具备如下几点特性：

1.你应该对 Zabbix 这个主流开源企业级监控系统并不陌生，它的官方网站是 [https://www.zabbix.com/。Zabbix](https://www.zabbix.com/。Zabbix) 具备如下几点特性：

2.功能丰富，具有图形化的界面展示，支持网络拓扑图示，同时也支持自定义面板。它还具备一套 CMDB功能（资产管理系统，可以对所有主机及硬件资产信息进行管理），支持数据聚合的分析方式。  
服务高可用性，Zabbix 支持 proxy 代理模式，所有的 客户端 节点汇报到 proxy 的代理中间节点，再统一上报到 server 节点，这样减少 zabbix\_server 节点的压力。另外 zabbix\_server 支持 ha 切换来作保证服务的高可用。

3.丰富的数据采集模式，Zabbix 有自己的 agent 用于数据采集，也支持一些公共的协议方式，如：SNMP、JMX 等。采集模式支持主动和被动模式（主动模式是指服务端能够主动向客户端抓取数据，被动模式是指客户端能向服务端上报数据）。它能够支持用户自定义方式编写插件（监控脚本、主机监控模版、主机组模版等）。Zabbix 提供一套完整的 API，可以方便地集成到企业内部 DevOps 或者是运维管理系统中。

除此之外，对于规模庞大的主机监控信息管理十分自动化，Zabbix 支持自动发现和自动注册功能，本课时我们会来具体讲解：

自动发现是指在服务端能够自动发现客户端的主机信息。这种机制我们在前面的课时 10 里有详细讲解过（通过 Django Python 开发一套自动发现资产信息的 CMDB 系统）。

Zabbix 的自动发现其实就是自动发现并自动添加监控客户端的主机及相关信息。它通过服务端来发起扫描，扫描局域内部的网络设备、服务器、主机等设备，收集这些设备的信息，添加设备固定的模板中，从发现-&gt;添加-&gt;配置-&gt;监控全面自动化实现，这就是自动发现的第 1 种模式。

第 2 个模式，Zabbix\_agent 主动注册，当 Zabbix\_agent 启动 agent 以后，会主动上报自己的主机信息给服务端，这样就可以实现主动上报并在服务端注册相关信息。

## Zabbix 服务端自动发现

首先我们来讲解 Zabbix 服务端自动发现，它的原理是这样的，首先扫描指定的 IP 地址段，匹配符合条件的主机，比如主机的类型，是否有 agent 等，如果满足条件就把设备信息录入。

接下来我们重点介绍 Zabbix 自动发现主机的设置步骤。主要有 2 个步骤：

* 首先设置好自动发现的规则，定义扫描的网段 IP 信息，然后定义自动扫描的时间间隔等。

* 当满足规则和条件以后，就需要配置好执行动作，配置执行动作可以添加主机，或者添加主机到某一个主机组或关联主机的模板等相关动作，都是在执行动作这里进行配置的。

接下来做一个演示，我这里拿两台主机，一台是服务端主机，作为 Zabbix\_server 来部署的，另外一台是客户端的主机，安装了 Zabbix\_agent，安装 Zabbix 时你可按照官方文档要求部署，并按其要求启用相应的服务。接下来就可以在控制台进行 Zabbix 具体配置了。

我们登录到 Zabbix 控制台，首先配置自动发现规则，我们按顺序点击 Configuration、Discovery 和 create Discovery rule，这就是配置的自动发现规则。

CgqCHl68_HmAbetTAAClb6f9nPQ911.jpg

我们来看一下，这里有几个框，一个是配置规则的名称（Name），由于我们没有用到代理模式，所以 Discovery by proxy 是 no proxy，下面定义的 IP range 就是规定的扫描主机段范围，如果 Zabbix_server 在多个主机段，我只想扫描一个子网段或者是某一个主机段中的主机，就需要定义好它针对的主机段信息，可以按照这样的格式去填入。

下面的 Update interval 就是扫描的周期，也就是上次扫描完成以后间隔多久再次扫描。下面的 Checks，zabbix agent "system.uname"表示检查主机的原数据，可以调用 Zabbix_agent 的 system.uname 函数来获取客户端主机的元数据。再往下的 Device uniqueness 就是定义设备的唯一性要求，这个单选框选到了 IP address 这里，代表以 IP 作为唯一的标识来做一台主机。如果我们选择 Zabbix_agent 的 system.uname，就是以主机的元数据来标识这是一台唯一性的主机。

我们在刚刚的选项框里面能看到需要填入的具体信息选项的说明，你可以具体来看一下：

Ciqc1F68_IWACLmRAAA5Dtpkssg217.jpg

在配置好相应的录入规则以后，接下来我们需要做的就是配置它的执行动作，我们同样在 Zabbix 控制台，点击 Configuration 按钮，然后点击 Actions，在下面我们来新建一个 Discovery 类型的 action。

Ciqc1F68_IuAMgiIAAB-YdH43CY334.jpg

接下来就会进入这个动作的具体配置界面。

Action 标签栏内最上面是 action 的名称（Name），我们可以自定义来书写。中间（Type of calculation）是配置它的条件等等,好了我们先看到这里，如果选择右边的标签“ Operations”，就是来定义具体执行动作。

CgqCHl68_JOAIENYAADdSo5DXj4465.jpg

回到 Actions 这一栏，继续详细讲解，这里添加的动作需要设置条件（供包含 3 大块），我们先来看下面的 New condition（就是添加新的条件），我们可以下拉对应的选项框，选择好对应的条件，比如这里规定了主机 IP 的范围。如果列好了这个条件以后，点击 add 按钮就会增加到 Conditions 这个选项框里面去，也就是说，新加的这些条件都会放入到这个选项框里面。

接下来我们可以具体看一下这个事例新加的这些条件在选项框中有哪些，service type equals Zabbix agentt 表示扫描的主机中需要安装了 Zabbix_agent（Zabbix 的 agent 程序），所以添加这个动作的条件是扫描到的这一台主机必须装有 Zabbix_agent。Host IP equals 172.21.64.1-254 表示主机 IP 等于这个范围段，也就是说它必须要是某一个范围段的主机 IP。

Type of calculation中的设置表示 Conditions 这个选项框里给它添加了两个条件（分别对应是条件 A 和条件 B）。它们必须满足什么关系（如：and 或者是 or），我这里是设置的是 A and B，也就是说添加的条件中需要同时成立，才能够满足整体的条件。

整体的条件满足以后，我们就会继续配置 actions 里面的另外一大块 Operations。我们点击 Operations 会看到，它负责具体设置执行动作，我们可以在这里定义它的主题，还有它的 message（信息），这些都是用于我们发送这些文字提醒。


CgqCHl68_JyABJdqAAC4yAEr8K4251.jpg

如果不发送文字提醒，只是为了添加主机到 Zabbix 列表里，监控主机的信息列表的话，我们就可以添加如下的几个对应的 Operations，首先在 Operations 这一栏里来添加 3 个操作：

1.一个是添加主机（add host），也就是发现主机满足条件以后，首先把主机添加到主机列表里面。
2.添加主机到某一个组里面去（Add to host groups），在 Zabbix 里面有个默认的主机组，就是 Linux servers，只要满足条件的都可以添加到 Linux servers 组中。
3.关联到主机需要监控的模板（Link to templates），这里关联的是 Linux 主机模版Template，Template OS Linux by zabbix agent 就是 Zabbix 里一个针对 Linux servers 的默认模板，关联好这个模板以后，就可以利用默认的模板对客户端主机进行数据的采集，然后监控并且画图。

整个添加完成以后，我们点击 update，就完成了 Zabbix 服务端自动发现规则的整体设置，完成服务端在控制台这一端的规则设置以后，接下来我们在 agent 这一端确认 zabbix_agentd.conf 中的几个配置：

```
Server=172.21.64.12
ServerActive=172.21.64.12  //主动上报zabbix服务端的Ip地址
HostMetadataItem=system.uname  //通过system.uname函数动态获取主机元数据。
```
* Server=172.21.64.12 配置，也就是 Zabbix_server（zabbix服务端） IP 地址。
* HostMetadataItem 主要是来配置主机的元数据，这里可以通过 system.uname 来动态获取元数据信息。

完成 Zabbix_agent 配置以后重启，这样就可以开始等待一段时间来判断服务端是否有发现这台主机，是不是能够自动发现新的主机。

我们在介绍完第 2 种模式，也就是 Zabbix_agent 主动上报模式配置以后，接下来教你如何在 Zabbix 控制台判断是否找到新的主机.

## Zabbix agent 主动上报

Zabbix_agent 主动上报的一种配置这里需要配置 2 大块：

* 配置 agent 自动注册的规则和动作；
* 客户端配置地调整。