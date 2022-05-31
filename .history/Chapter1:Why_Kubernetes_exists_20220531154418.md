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