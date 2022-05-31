# Why Kubernetes exists


**本章涵盖：**

* Kubernetes 存在的原因
* Kubernetes 常用术语
* Kubernetes 特定用例
* Kubernetes 高级功能
* Kubernetes 不适用的场景

---
Kubernetes是一个开源平台，用于托管容器和定义以应用程序为中心的API，用于管理围绕这些容器如何通过存储，网络，安全性和其他资源来管理这些容器的基础设施。

为什么要在您的环境中实施kubernetes，而不是使用与DevOps相关的基础架构工具手动提供这些资源？

答案在于我们将DevOps定义为越来越多地集成到整个应用程序生命周期中的方式，DevOps越来越多地进化为包括流程，工程师和工具，这些工具和工具支持数据中心中更自动化的应用程序管理。成功执行此操作的关键之一是基础架构的可重复性。


## 常用术语解释

* CNI and CSI — 允许在Kubernetes中运行的Pods的容器网络和存储接口扩展自定义的网络和存储
* Container - 通常运行应用程序的Docker或OCI(Open Container Initiative) 镜像
* Control plane — Kubernetes集群的大脑，在该集群中进行了安排和管理所有Kubernetes对象（有时称为Masters）
* DaemonSet — 就像Deployment一样，但它在集群的每个节点上运行
* Deployment — 用来管理Pods 实现滚动更新

* kubectl — 与Kubernetes控制平面交互的命令行工具
* kubelet — 在集群节点上运行的Kubernetes agent 。管理容器声明周期。
* Node — 运行kubelet和容器的机器 通常所说的宿主机
* OCI — 容器定义的标准接口和规范
* Pod — Kubernetes 最小的调度单元，封装运行多个容器

## Kubernetes和不可变基础设施

管理基础设施是一种可再现的方法，可以将基础设施配置的“漂移”作为硬件，合规性和其他数据中心要求随时间变化而变化。这既适用于应用程序的定义以及这些应用程序运行的主机的管理。IT工程师都非常熟悉工作，例如

* 在服务器上更新Java版本
* 确保某些应用程序不会在特定的地方运行
* 应用程序从损坏的硬件或者旧的机器中进行迁移
* 手动管理负载平衡路由
* 一些语言运行时环境的强依赖忘记更新

当我们在数据中心或云中管理和更新服务器时，其原始定义从预期的IT体系结构中“消失”的几率增加了，应用程序可能在错误的位置，错误的资源分配或使用错误的存储模块运行。

Kubernetes为我们提供了一种使用一个方便的工具来管理所有应用程序：[kubectl](https://kubernetes.io/docs/tasks/tools/)，
命令行客户端，实现REST API调用到Kubernetes API服务器。我们还可以使用Kubernetes API客户端来编程执行这些任务。安装kubectl并在一个[kind](https://github.com/kubernetes-sigs/kind)管理的集群上进行测试非常容易，我们将在本书中进行。


以前管理复杂应用程序的方法包括技术Puppet, Chef, Mesos, Ansible,SaltStack。从Mesos等软件提供的一些应用程序和调度原始措施中借用概念

Ansible，saltstack和Terraform通常在基础设施配置中起着重要作用（例如防火墙或二进制安装）。Kubernetes也可以实现此功能，但是它在Linux环境上使用了特权容器（这些被称为Windows V1.22上的HostProcess Pods）。比如，Linux系统中的特权容器可以管理将iPtables规则路由到应用程序的规则，实际上，这正是Kubernetes Service Proxy（称为Kube-Proxy）所做的事情。

Google，Microsoft，Amazon，VMware和许多公司都采用了以容器作为核心，并为客户提供了在不同的云和裸露金属环境上运行数百或数千个应用程序的机器。因此，构造者是运行应用程序和管理应用基础设施（例如为容器提供为运行IP地址）的基本属性，该应用程序依赖于这些应用程序（例如 防火墙设置以及自定义存储，），最重要的是，本身运行应用程序。

Kubernetes 本质上已经成为了现代容器编排的标准，可以实现在任何云以及任何数据中心当中来管理任何服务。


## 容器和镜像

应用在主机中运行时必须要有它所需要的依赖， 在容器时代之前开发需要临时的完成这项配置。

从本质上讲，可以将Docker视为运行容器的一种方式，其中容器是运行的[OCI](https://github.com/opencontainers/image-spec)。OCI规范是定义可以由Docker等程序执行的镜像的标准方法，最终是具有各种层的Tarball。图像中的每个TARBALL都包含Linux二进制文件和应用程序文件等内容。因此，当您运行容器时，容器运行时（例如Docker，Container或Cri-O）下载镜像，解压，并在主机系统上启动一个运行镜像的过程

<center><img src="app_running_in_containers.jpg"></center><br />