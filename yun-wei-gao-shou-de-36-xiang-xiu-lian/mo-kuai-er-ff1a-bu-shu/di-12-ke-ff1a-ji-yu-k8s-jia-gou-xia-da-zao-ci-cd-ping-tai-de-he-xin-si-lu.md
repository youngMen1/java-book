# 第12课：基于 K8S 架构下打造 CI\CD 平台的核心思路

本课时我们讲解一套在 K8s 环境下打造 CI\CD 平台的核心思路。

## K8s 架构中的 CI\CD 平台

### 课程意义
在开始之前，我给你讲一下学习本课时的意义。在课时 9 中，我们介绍了基于 Jenkins 的持续集成架构及普通使用模式下需要注意的问题。在本课时，我们来讲解基于 K8s 和 Docker 容器化部署架构下需要实现 CI\CD 平台所需关注核心知识，如： CI/CD 平台实现后需要具备什么样的特性？架构设计上需要注意什么？希望在本课时能带你找到对应的答案。
### K8s 中的 CI\CD 平台应该实现的功能
首先，开门见山的来讲一下 K8s 和 Docker 部署架构下，所具备的特性：
1.支持容器部署，作到基于镜像方式发布、K8s 进行编排，所以在 CI\CD 平台里面，我们务必要遵循容器及 K8s 的发布部署方式。

2.在 K8s环境下，Devops 理念更加突出，需要作到通过 Jenkins 的 API 创建和管理 Job 。

3.Pipeline 灵活性管理 Job ，基于流水线的方式，能给我们带来更多的灵活性的 Job 管理和定义。

4.需要支持多环境、多版本的发布场景。我们在大的企业环境下，常遇到多套工程的部署环境，可能也会有多套代码分支，CI\CD 平台应该更好地兼容这样的环境进行设计。

5.我们需要注重 Jenkins，也就是 CI\CD 平台本身的高可用，如果是基于单点的方式部署 Jenkins，它本身就不是高可用的，我们需要考虑它本身的高可用性。

基于这样的理念和要求，我认为要做这样的一个平台，归纳起来需要包括以下几个核心的功能：

1.Jenkins 部署高可用。

2.通过 Pipeline 创建\管理 Job。

3.通过 API 接口创建\管理 Job。

以上是接下来我要介绍的在 K8s 架构下搭建 Jenkins 的 CI\CD 平台的三点必要能力，接下来我们详细的讲解一下。

### 学前提示
在学习本课时之前，你需要了解 Jenkins 基础搭建和基础配置，同时需要对 K8s 有一定了解，比如 pod、node、service 分别是什么概念？K8s 结构组成是什么样子的？另外，如果要把整个平台搭建起来，还需要具备 Docker 镜像打包和发布整个流程的技术操作能力。

### 核心思路

接下来，就基于刚刚讲到的三点核心内容，我们逐一来进行介绍。

### 思路一、Jenkins 服务高可用

首先第一点，就是 Jenkins 本身服务的部署高可用。

Cgq2xl58Pj2ARAUEAAECOYYrTH8534.png

回顾一下 Jenkins 本身的部署模式，通常有以下三种方式：

第一种方式，就是在操作系统上面直接部署 Jenkins，然后让 Java 进程直接运行，这是比较传统做法。



第二种方式，是将 Jenkins 封装成一个标准镜像，并且以容器化的方式运行，这个是在课时 9 里面推荐你部署的方式，这样会更容易安装、维护和管理。



第三种方式，是建立在第二种方式基础上，接下来需要给你介绍的。



第三种方式重点解决单例模式Jenkins 部署下，服务存在的高可用问题，如何做到一台 Jenkins 服务出现问题，有另外的服务能够自动把它拉起？另外，由于Job 的增加，常常导致 Jenkins 本身系统负载能力趋于饱和，从 Jenkins 本身的性能瓶颈上如何进行水平化的扩容？



接下来，我们把Jenkins 做成主从模式，做成主从架构可以让我们把需要消耗 Jenkins 性能的 Job 工作交给从节点（slave）去完成。



还需要解决 Jenkins 本身的高可用，我们可以把 Jenkins 部署在 K8s 的 Pod 中，让 K8s 负责对 Jenkins 服务进行监控和拉起，这样就保证了 Jenkins 服务本身的高可用，同时由于k8S弹性资源调度优势，还要借助 K8s 对 Jenkins 资源进行水平化扩展。



所以 Jenkins 服务要做到真正的高可用推荐遵循 Jenkins 主从架构，并且把 Jenkins 部署到 K8s 的架构体系中。



为了让你能够更加形象地来理解，这里画了一张图：

Cgq2xl58Pj2ASTTbAAPOlOYZ6DM633.png

我们看到这里有 K8s 整个集群，包含有三个 Node 节点，每个 Node 节点上有一组 Pod。Pod 会分为两大类型，一个是 Master 类型的 Pod，一个是 Slave 类型的 Pod。在 K8s 里面最小的管理单元是 Pod ，我们姑且认为它是一个独立的服务个体。



这个 Pod 上面运行的是 Jenkins 的 Master 服务，在另外两个 Node 上面的是 Jenkins 的 Slave 的 Pod 服务，所以我们会看到在 K8s 这套集群体系下，Jenkins 是以主从的方式运行在 K8s 的架构之上， Jenkins 的 Job 任务交给 Slave 节点去完成，而 Master 只负责进行 Job 的管控或中心平台的作用。

## 优势

1、Job 资源隔离

有一个优势是 Job 间资源隔离，我会看到 Jenkins 将 Job 分给了不同的 Pod 处理，Pod 本身就是以容器化（cgroup）进行资源隔离，所以 Job 间不会造成进程的争抢。



2、动态伸缩

同时，K8s 的支持动态伸缩特性，当 Pod 资源不足，需要扩容新的 Pod 的时候，可以借用 K8s 的动态伸缩模式来进行 Pod 资源的扩容。



3、高可用

同时也保证了服务的高可用，因为我们知道 K8s 对服务的拉起可以做到自动的故障诊断及故障拉起。



这个就是 Jenkins 本身融合到 K8s 架构里面的高可用架构的图示。在最外面挂了一组存储卷，我们可以把 Jenkins 需要持久化的相关内容，以 PVC 方式挂载到分布式的存储之中。这样的话就保证了数据的高可靠。

## 如何实现？

接下来我讲解一下，如果你要来部署这样的一套架构的话，它的核心思路是什么样的？整个部署完 K8s 的集群以后，我们首先需要部署的就是 Jenkins 的 Master 节点。



Master 节点相关的配置是通过 K8s 来创建几个重要的对象：一个是 namespace，这是 K8s 给 Jenkins 的一个独立的命名空间。第二个就是 PVC 对象，我们会在图中看到一组外挂的存储节点，创建 PVC 对象。接下来就是要创建 deployment 对象，如果你了解 K8s 一定会清楚，deployment 是一个非常重要的 K8s control 的对象，它直接控制着 Pod 资源的镜像、Pod 的资源使用，还有它的服务探针等相关内容，这个都是在 deployment 里面进行创建的。最后是创建 service 对象，service 对象创建以后，Pod 就可以正式的对集群内部提供服务。



以上，就是在 Master Jenkins 节点在 K8s 里面创建的过程。接下来，我们就可以在 Master 启用以后，登录Jenkins的管理控制台来配置 Slave 结点。



在控制台里面，我们需要重点的两步来配置 Slave 节点，就是：

1.我们需要安装 K8s 的插件(kubernetes plugin)。

2.需要配置 Slave 的模板信息，也就是 Slave deployment。

这都是在 Jenkins 的界面控制台来进行配置的。接下来，我们来具体看一下，在创建 Jenkins 的 Master deployment 这个具体的创建方式。



我们知道 K8s 是通过 yaml 的语法格式来创建、管理对象，如下是在 deployment 对象配置的几个核心部分：



第一部分，创建 Pod 里面的容器的镜像，以及对外服务的端口。



```
apiVersion: extensions/v1beta1

kind: Deployment 

spec:

    containers: 

        - name: jenkins 

           image: jeson_jenkins:1.1  //镜像

           - containerPort: 8080   //服务端口

              name: web

              protocol: TCP

          - containerPort: 50000

             name: agent

             protocol: TCP
```
第二部分，这里定义两个探针，用于 K8s 对服务进行健康检测。


```
livenessProbe: //定义存活探针

    httpGet: 

        path: /login 

        port: 8080

        …

livenessProbe:  //定义就绪探针

     httpGet: 

     path: /login

     port: 8080

     ….
```


