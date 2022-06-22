# Why the Pod?
 

**本章涵盖：**

* 什么是Pod ？
* 为什么我们需要Pod ？
* Kubernetes 如何构建Pod
* Kubernetes 控制面


在上一章中，我们提供了Kubernetes的高级概述，并介绍了其功能，核心组件和体系结构。我们还展示了一些业务用例，并概述了一些容器定义。Kubernetes Pod抽象以灵活的方式运行数千个容器一直是企业中过渡到容器的基本部分。在本章中，我们将介绍Pod以及如何建立Kubernetes作为基本应用程序构建块支持它

正如第1章中简要提到的那样，Pod是在Kubernetes API中定义的对象，Kubernetes中的大多数内容也是如此。Pod是可以部署到Kubernetes集群的最小原子单元，而Kubernetes围绕Pod定义构建。Pod允许我们定义一个可以包含多个容器的对象，该对象允许Kubernetes创建一个或多个在节点上托管的容器

<center><img src="./images/node.jpg"></center><br />

许多其他Kubernetes API对象要么直接使用Pod，要么是支持Pod的API对象。例如，Deployment使用Pod，以及StatefulSets和DaemonSets。几个不同的高级Kubernetes控制器创建和管理Pod生命周期。控制器是在控制平面上运行的基础组件。内置控制器的示例包括控制器管理器，云管理器和调度程序。但是首先，让我们通过布置Web应用程序，然后将其协调到Kubernetes，Pod和控制平面。

```
NOTE：
您可能会注意到，我们使用控制平面定义运行控制器，controller  manager和scheduler的节点组。它们也被称为Master，但是在本书中，我们将在谈论这些组件时使用control plane

```

## web application

让我们浏览一个示例Web应用程序，以了解为什么我们需要一个Pod以及如何构建Kubernetes来支持Pod和容器化的应用程序。为了更好地了解Pod是什么，我们将在本章的大部分时间内使用以下示例。


Zeus ZAP Energy Drink Company拥有一个在线网站，该网站允许消费者购买其不同的碳酸饮料。该网站由三个不同的层组成：用户界面（UI），中间层（各种微服务）和后端数据库。他们还具有消息传递和队列协议。像Zeus ZAP这样的公司通常具有各种Web前端，包括面向消费者和公司内部系统，构成中间层的不同微服务以及一个或多个后端数据库。这是Zeus Zap的Web应用程序的一个分层：

* Nginx提供的JavaScript前端
* 两个网络控制层是python微服务，托管了django
* 后端数据库CockroachDB提供端口6379

<center><img src="./images/web.jpg"></center><br />

现在，让我们想象他们在生产设置中以四个不同的容器运行这些应用程序。然后，他们可以使用这些Docker Run命令启动该应用：


```
$ docker run -t -i ui -p 80:80
$ docker run -t -i miroservice-a -p 8080:8080
$ docker run -t -i miroservice-b -b 8081:8081
$ docker run -t -i cockroach:cockroach -p 6379:6379

```

一旦这些服务启动并运行，该公司很快就会：

* 它们不能运行 UI 容器的多个副本，除非它们进行负载平衡在80端口前面，因为主机上只有一个 80 端口镜像正在运行
* 他们无法将 CockroachDB 容器迁移到不同的服务器，除非IP 地址被修改并注入到 Web 应用程序中（或者他们添加了DNS CockroachDB 容器移动时动态更新的服务器）
* 他们需要在单独的服务器上运行每个 CockroachDB 实例以产生高可用性
* 如果 CockroachDB 实例在一台服务器上死机，他们需要一种方法将其数据移动到新节点并回收未使用的存储空间

Zeus Zap还意识到存在容器编排平台的一些要求。这些包括：

* 数百个进程之间的共享网络都绑定到同一个端口
* 存储卷与二进制文件的迁移和解耦，同时避免弄脏本地磁盘
* 化可用 CPU 和内存资源的利用率以实现节约成本


```
NOTE：
在服务器上运行更多进程通常会导致noisy neighbor(互相干扰)现象：应用拥挤导致稀缺竞争过度
资源（CPU、内存）。 系统必须减轻noisy neighbor(互相干扰)现象

```

大规模（甚至小规模）运行的容器化应用程序需要在调度服务和管理负载均衡器方面具有更高的意识。 因此，还需要以下要求：

* Storage-aware scheduling 存储感知调度 - 调度进程以使其数据可用

* Service-aware network load balancing 服务感知网络负载平衡 — 将流量发送到不同的 IP地址容器从一台机器移动到另一台机器

刚刚在我们的应用程序中分享的启示与2000 年代的分布式调度和编排工具，包括 Mesos和Borg。 Borg 是 Google 内部的容器编排系统，Mesos 是一个开源应用，两者都提供集群管理和早于Kubernetes


### Web应用程序的基础架构

如果没有Kubernetes等容器编排软件，组织就需要其基础架构中的许多组件。为了运行应用程序，您需要在云或物理计算机上充当服务器的各种虚拟机（VM），如前所述，您需要稳定的标识符来定位服务。

服务器工作负载可能会有所不同。例如，您可能需要具有更多内存的服务器来运行数据库，或者您可能需要一个具有较低内存的系统，但用于微服务的CPU。另外，您可能需要像MySQL或Postgres这样的数据库的低延迟存储，但是备份和其他通常将数据加载到内存的应用程序较慢，然后再也不会触摸磁盘。此外，您的连续集成服务器（例如Jenkins或CircleCi）需要完全访问服务器，但是您的监视系统需要仅阅读您的某些应用程序。现在，还要添加人类授权和身份验证。总而言之，您将需要：

* VM或物理服务器作为部署平台
* 负载均衡
* 服务发现
* 存储
* 安全系统

为了维持该系统，您的DevOps员工还需要维护以下（除了更多子系统之外）：

* 集中日志
* 监控、高级和指标
* CI/CD 系统
* 备份
* 密钥管理

与大多数内部应用交付平台相比，Kubernetes内置日志轮换、检查和开箱即用的管理工具。 随之带来了业务挑战：操作要求。


### 操作要求

Zeus Zap 能量饮料公司没有典型的季节性增长期像大多数在线零售商一样，但他们确实赞助了各种电子竞技赛事很多流量。 这是因为营销部门和各种网络游戏彩带运行在这些活动期间推广的比赛。这些在线用户流量模式为 DevOps 团队提供了管理突发流量的最具挑战性的模式之一。扩展在线应用程序是一个难以维护和解决的问题，现在团队必须安排爆发模式！ 此外，由于网络社交围绕电子竞技赛事创建的媒体活动，企业关注关于停电。 停机时间的成本是丑陋和巨大的。

根据 Gartner 2018 年的一项研究平均IT 停机成本为每分钟 5,600 美元，考虑到两者之间的差异企业。 应用程序有两个小时的停机时间并不少见，导致平均成本为 672,000 美元。 钱是一回事，但钱呢？人力成本？ DevOps 工程师面临中断； 它是生活的一部分，但它穿在身上员工，以及，并可能导致倦怠。 美国员工倦怠成本行业每年大约 125 到 1900 亿美元。

多公司在他们的产品中需要某种程度的高可用性和回滚系统。 这些要求与对冗余的需求齐头并进
应用程序和硬件。 然而，为了节省成本，这些公司可能希望在要求不高的时间段内向上和向下扩展应用程序可用性。因此，成本管理通常与更广泛的业务需求相矛盾正常运行时间左右。 回顾一下，一个简单的 Web 应用程序需要

* 扩容
* 高可用
* 版本控制和应用可回滚
* 成本管理


## Pod 是什么 ？

粗略地，Pod是一个或多个OCI镜像，它们在Kubernetes cluster节点上以容器的形式运行。Kubernetes节点是一个运行kubelet的单个计算能力（服务器）。像Kubernetes中的所有其他内容一样，节点也是API对象。部署Pod与发行以下内容一样简单

```
cat << EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
spec:
    container:
    - name: busybox
      image: mycontainerregistry.io/foo
EOF

```

```
$ kubectl create -f pod.yaml
```

先前的语法使用Linux Bash Shell和Kubectl命令运行。kubectl命令是提供命令行界面的二进制文件

在大多数情况下，Pod不会直接部署。取而代之的是，它们是由我们定义的其他API对象自动为我们创建的，例如Deployments，Jobs，StatefulSets和DaemonSets：

* Deployments — Kubernetes群集中最常用的API对象。它们是部署微服务的典型API对象
* Jobs - Pod 运行任务
* StatefulSets - 需要特定需求并且通常是数据库等状态应用程序的主机应用程序。
* DaemonSets - 当我们想在群集的每个节点上运行一个Pod作为“代理”时使用（通常用于涉及网络，存储或日志的系统服务）。

以下是StatefulSet特性：

* 顺序Pod名获取网络唯一标识
* 始终挂载到同一个 Pod 的持久存储
* 有序启动、扩展和更新

Docker 镜像名称支持使用名为 latest 的标签。 不要使用镜像名称 mycontainerregistry.io/foo 在生产中，因为这会拉来仓库的最新标记，这是镜像的最新版本。 总是使用版本化标签名称，而不是最新的，甚至更好的 SHA 来安装图片。 镜像标签名称不是不可变的，但图像 SHA 是不可变的。 许多系统失败是因为容器的更新版本不经意间安装。 朋友不要让镜像版本使用最新latest 的标签！


Pod 启动后，可以查看运行在默认 Namespace 中的 Pod一个简单的 kubectl get po 命令。 现在我们已经创建了一个正在运行的容器，它是在 Zeus Zap Web 应用程序中部署组件很简单。 只需使用 Docker 或 CRI-O 之类的常用镜像工具来捆绑各种二进制文件及其依赖项转换为不同的镜像，这些镜像只是包含一些文件定义。 在下一章中，我们将介绍如何制作自己的镜像和Pod


而不是让系统调度各种 docker run 命令时服务器启动后，我们定义了四个更高级别的 API 对象，它们创建 Pod 并调用Kubernetes API 服务器。 正如我们所提到的，Pod 很少独立用于安装应用程序在 Kubernetes 上。 用户通常使用更高级别的抽象，例如DeploymentsStatefulSets。 但是我们仍然回到循环控制创建Pod，因为 Deployments 和 StatefulSets创建副本对象，然后创建 Pod。


### Linux namespaces

Kubernetes 命名空间（您使用 kubectl create ns 创建的命名空间）不是与 Linux 命名空间相同。 Linux 命名空间是一个 Linux 内核特性，它允许用于内核内部的进程分离。 Pod，在基础级别，是一堆名字特定配置中的空间。 Pod 具有以下 Linux 命名空间

