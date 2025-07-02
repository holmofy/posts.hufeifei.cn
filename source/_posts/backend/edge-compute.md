---
title: 边缘计算云原生开源方案选型比较
date: 2025-07-02
categories: 后端
mathjax: true
post_src: https://zhuanlan.zhihu.com/p/353429279
tags:
- KubeEdge
- OpenYurt
- SuperEdge
---

随着Kubernetes已经成为容器编排和调度的事实标准，各大公有云厂商都已经基于Kubernetes提供了完善的Kubernetes云上托管服务。

同时也看到越来越多的企业、行业开始在生产中使用Kubernetes, 拥抱云原生。在各行各业数字化转型和上云过程中，公有云厂商也在主动拥抱传统线下环境，在思考各种各样的解决方案使云上能力向边缘(或线下)延伸。

而Kubernetes由于屏蔽了底层架构的差异性，可以帮助应用平滑地运行在不同的基础设施上的特性，云上的Kubernetes服务也在考虑拓展其服务边界，云原生和边缘计算结合的想法自然就呼之欲出了。

目前国内各个公有云厂商也都开源了各自基于Kubernetes的边缘计算云原生项目。如华为云的KubeEdge，阿里云的OpenYurt，腾讯云的SuperEdge。

目前网上很少有从技术视角来介绍这几个项目优缺点的文章，本文试着从技术视角，从开源视角来分析这几个项目，希望可以给大家做项目选型时提供一些借鉴。

## 1. 比较思路

这几个项目都是云边一体，云边协同的架构，走的是Kubernetes和边缘计算结合的路数，因此决定从以下几点比较:

（1） 各个项目的开源状况：比如开源项目的背景、开源的时间、是否进入了CNCF等；

（2）Kubernetes架构:

* 先对比与Kubernetees的架构差异：主要关注是否修改Kubernetes，和；Kubernetes一键式转换等
* 根据架构差异对比和Kubernetes的能力增强点；主要关注边缘自治，边缘单元化，轻量化等能力
* 最后看一下架构差异可能带来的影响: 主要关注运维监控能力，云原生生态兼容性，系统稳定性等方面

（3）对边缘计算场景支持能力:

* 主要关注是否具备端设备的管理能力

接下来以项目的开源顺序，从上述几个方面来介绍各个项目。

## 2.边缘云原生开源项目对比

### 2.1 [KubeEdge](https://kubeedge.io/zh/)

#### （1）开源状况

KubeEdge是华为云于2018年11月份开源的，目前是CNCF孵化项目。其架构如下:

![KubeEdge](https://release-1-20.docs.kubeedge.io/assets/images/kubeedge_arch-a0fa6324bc543a933d766e45d5f00f77.png)

#### （2）与Kubernetes的架构差异

首先从架构图可以看到，云端(k8s master)增加了Cloud Hub组件和各类controller，而在边缘端(k8s worker)没有看到原生的kubelet和kube-proxy，而是一个对原生组件进行重写了EdgeCore组件。

从架构图看EdgeCore是基于kubelet重构的，为了保证轻量化，裁剪了原生kubelet的部分能力，同时也增加了很多适配边缘场景的能力。具体如下:

* Cloud Hub+EdgeHub模块: 抛弃了原生kubernetes 的组件间数据同步list/watch机制，改成基于websocket/quic协议从云端往边缘推送模式。
* 节点元数据缓存模块(MetaManager): 把节点维度的数据持久化在本机的SQLite数据库中，当云边网络不稳定时Edged模块将从本地数据库中获取数据用于业务的生命周期管控。
* DeviceController+设备管理模块(DeviceTwin): 把设备管理能力直接集成到EdgeCore中，为用户提供原生的设备管理能力。

上述的架构设计，**对比Kubernetes的能力增强点主要有**：

* 边缘自治：通过增加节点元数据缓存，可以规避云边断网状态下，边缘业务或者节点重启时，边缘组件可以利用本地缓存数据进行业务恢复，这就带来了边缘自治的好处。
* 轻量化: 削减了部分kubelet功能(如CSI，CNI等)，从而使边缘EdgeCore组件相比原生kubelet组件更加轻量。同时因为节点上增加了SQLite数据库，所以节点维度相比原生节点是否轻量待确认，欢迎熟悉的同学提供数据。

**架构差异可能带来的影响**：

* **云原生生态兼容性不足**：
 - 跟随社区同步演进挑战大: 由于对Kubernetes系统的侵入式修改，后续跟随Kubernetes社区的演进将会遇到很大挑战。
 - 边缘节点无法运行Operator：因为云边通信机制的修改，Cloud Hub只能往边缘推送有限的几种资源(如Pod，ConfigMap等)。而Operator既需要自定义CRD资源，又需要list/watch云端获取关联资源，因此社区的Operator无法运行的KubeEdge的边缘节点上。
 - 边缘节点不适合运行需要list/watch云端的应用: 因为云边通信机制的修改，导致原来需要使用list/watch机制访问kube-apiserver的应用，都无法通过hub tunnel 通道访问kube-apiserver，导致云原生的能力在边缘侧大打折扣。
 - 运维监控能力支持有限：因为目前云边通信链路是kube-apiserver --> controller --> Cloud Hub -->EdgeHub -->MetaManager等，而原生Kubernetes运维操作(如kubectl proxy/logs/exec/port-forward/attch等)是kube-apiserver直接请求kubelet。目前KubeEdge社区最新版本也仅支持kubectl logs/exec/metric，其他运维操作目前还不支持。
 - 系统稳定性提升待确定:
 - 基于增量数据的云边推送模式：可以解决边缘watch失败时的重新全量list从而引发的kube-apiserver 压力问题，相比原生Kubernetes架构可以提升系统稳定性。
 - Infra管控数据和业务管控数据耦合：Kubernetes集群的管控数据(如Pod，ConfigMap数据)和边缘业务数据(设备管控数据)使用同一条websocket链路，如果边缘管理大量设备或者设备更新频率过高，大量的业务数据将可能影响到集群的正常管控，从而可能降低系统的稳定性。

**边缘计算场景支持能力**

- 设备管理能力: 这个能力直接集成在edged中，给iot用户提供了一定的原生设备管理能力。

###  2.2 [OpenYurt](https://openyurt.io/zh/)

#### （1）开源状况

OpenYurt是阿里云于2020年5月份开源的，目前是CNCF沙箱项目。架构如下:

![OpenYurt](https://openyurt.io/zh/assets/images/arch-2c77ff23e9b7f4fe4956fe22700f5c0c.png)

#### （2）与Kubernetes的架构差异

OpenYurt的架构设计比较简洁，采用的是无侵入式对Kubernetes进行增强。在云端(K8s Master)上增加Yurt Controller Manager, Yurt App Manager以及Tunnel Server组件。而在边缘端(K8s Worker)上增加了YurtHub和Tunnel Agent组件。从架构上看主要增加了如下能力来适配边缘场景：

* YurtHub: 代理各个边缘组件到K8s Master的通信请求，同时把请求返回的元数据持久化在节点磁盘。当云边网络不稳定时，则利用本地磁盘数据来用于边缘业务的生命周期管控。同时云端的Yurt Controller Manager会管控边缘业务Pod的驱逐策略。
* Tunnel Server/Tunnel Agent: 每个边缘节点上的Tunnel Agent将主动与云端Tunnel Server建立双向认证的加密的gRPC连接，同时云端将通过此连接访问到边缘节点及其资源。
* Yurt App Manager：引入的两个CRD资源: NodePool 和 UnitedDeployment. 前者为位于同一区域的节点提供批量管理方法。 后者定义了一种新的边缘应用模型以节点池维度来管理工作负载。

上述的架构设计，对比Kubernetes的能力增强点主要有：

* 边缘单元化：通过Yurt App Manager组件，从单元化的视角，管理分散在不同地域的边缘资源，并对各地域单元内的业务提供独立的生命周期管理，升级，扩缩容，流量闭环等能力。且业务无需进行任何适配或改造。
* 边缘自治: 因为每个边缘节点增加了具备缓存能力的透明代理YurtHub，从而可以保障云边网络断开，如果节点或者业务重启时，可以利用本地缓存数据恢复业务。
* 云边协同(运维监控)： 通过Tunnel Server/Tunnel Agent的配合，为位于防火墙内部的边缘节点提供安全的云边双向认证的加密通道，即使边到云网络单向连通的边缘计算场景下，用户仍可运行原生kubernetes运维命令(如kubectl proxy/logs/exec/port-forward/attach等)。同时中心式的运维监控系统(如prometheus, metrics-server等)也可以通过云边通道获取到边缘的监控数据。
* 云原生生态兼容:
  - 所有功能均是通过Addon或者controller形式来增强Kubernetes，因此保证来对Kubernetes以及云原生社区生态的100%兼容。
  - 另外值得一提的是：OpenYurt项目还提供了一个YurtCtl工具，可以用于原生Kubernetes和OpenYurt集群的一键式转换，

**架构差异可能带来的影响**

* 原生Kubernetes带来的系统稳定性挑战：因为OpenYurt没有修改Kubernetes，所以这个问题也是原生Kubernetes在边缘场景下的问题。当云边长时间断网再次恢复时，边缘到云端会产生大量的全量List请求，从而对kube-apiserver造成比较大的压力。边缘节点过多时，将会给系统稳定性带来不小的挑战。
* 边缘无轻量化解决方案: 虽然OpenYurt没有修改Kubernets，但是在边缘节点上增加YurtHub和Tunnel Agent组件。目前在最小的1C1G的系统上运行成功，更小规格机器待验证。

### 边缘计算场景

* 无设备管理能力：OpenYurt目前没有提供设备管理的相关能力，需要用户以workload形式来运行自己的设备管理解决方案。虽然不算是架构设计的缺点，但是也算是一个边缘场景的不足点。

### 2.3.[SuperEdge](https://superedge.io/zh/)

#### （1）开源状况

SuperEdge是腾讯云于2020年12月底开源的，目前还是开源初期阶段。其架构如下：

![SuperEdge](https://picx.zhimg.com/v2-d399ee534081687024d24d5068df7883_1440w.jpg)

#### （2）与Kubernetes的架构差异

SuperEdge的架构设计比较简洁，也是采用的无侵入式对Kubernetes进行增强。在云端(K8s Master)上增加Application-Grid Controller, Edge-Health Admission以及Tunnel Cloud组件。而在边缘端(K8s Worker)上增加了Lite-Apiserver和Tunnel Edge，Application-Grid Wrapper组件。从架构上看主要增加了如下能力来适配边缘场景：

* Lite-Apiserver: 代理各个边缘组件到K8s Master的通信请求，同时把请求返回的元数据持久化在节点磁盘。当云边网络不稳定时，则利用本地磁盘数据来用于边缘业务的生命周期管控。同时基于边缘Edge-Health上报信息，云端的Edge-Health Admission会管控边缘业务Pod的驱逐策略。
* Tunnel Cloud/Tunnel Edge: 每个边缘节点上的Tunnel Edge将主动与云端Tunnel Cloud建立双向认证的加密的gRPC连接，同时云端将通过此连接访问到边缘节点及其资源。
* Application-Grid Controller：引入的两个CRD资源: ServiceGrids和 DeploymentGrids. 前者为位于同一区域的业务流量提供闭环管理。 后者定义了一种新的边缘应用模型以节点池为单位来管理工作负载。

#### 与OpenYurt对比

* 从SuperEdge的架构以及功能分析下来，发现SuperEdge从架构到功能和OpenYurt基本一致。这也从侧面印证，边缘计算云原生这个领域，各大厂商都在如火如荼的投入。
* SuperEdge与Kubernetes的对比分析可以参照OpenYurt的分析，这里我们从代码角度分析SuperEdge和OpenYurt的差异：
* YurtHub和Lite-Apiserver: YurtHub采取了完善的证书管理机制和本地数据缓存设计，而Lite-Apiserver使用的是节点kubelet证书和数据简单缓存。
* Tunnel组件：OpenYurt的Tunnel组件是基于Kubernetes社区的开源项目ANP(https://github.com/kubernetes-sigs/apiserver-network-proxy)，同时实现了完善的证书管理机制。而SuperEdge的Tunnel组件同样也是使用节点证书，同时请求转发是基于自行封装的gRPC连接。OpenYurt底层的ANP相比原生gRPC，会更好适配kube-apiserver的演进。
* 单元化管理组件: OpenYurt单元化管理支持Deployment和StatefulSet,而SuperEdge的单元化管理只支持Deployment。另外OpenYurt的NodePool和UnitedDeployment的API定义是标准云原生的设计思路，而SuperEdge的ServiceGrids和 DeploymentGrids的API定义显得随意一些。
* 边缘状态检测，这个能力OpenYurt未实现，SuperEdge的设计需要kubelet监听节点地址用于节点间互相访问，有一定安全风险。同时东西向流量也有不少消耗在健康检查上。期待这个部分后续的优化。

## 5. 对比结果一览

根据上述的对比分析，结果整理如下表所示：

|项目	|华为KubeEdge|	阿里OpenYurt|	腾讯SuperEdge|
|-----|------------|-------------|---------------|
|是否CNCF项目|	是(孵化项目)|	是(沙箱项目)|	否|
|开源时间|	2018.11|	2020.5|	2020.12|
|侵入式修改Kubernetes|	是|	否|	否|
|和Kubernetes无缝转换|	无|	有|	未知|
|边缘自治能力|	有(无边缘健康检测能力)|	有(无边缘健康检测能力)|	有(安全及流量消耗待优化)|
|边缘单元化|	不支持|	支持|	支持(只支持Deployment)|
|是否轻量化|	是(节点维度待确认)|	否|	否|
|原生运维监控能力|	部分支持|	全量支持|	全量支持(证书管理及连接管理待优化)|
|云原生生态兼容|	部分兼容|	完整兼容|	完整兼容|
|系统稳定性挑战|	大(接入设备数量过多)|	大(大规模节点并且云边长时间断网恢复)|	大(大规模节点并且云边长时间断网恢复)|
|设备管理能力|	有(有管控流量和业务流量耦合问题)|	无|	无|

## 3. 总结

各个开源项目，整个比较下来自己的感受是：KubeEdge和OpenYurt/SuperEdge的架构设计差异比较大，相比而言OpenYurt/SuperEdge的架构设计更优雅一些。而OpenYurt和SuperEdge架构设计相似，SuperEdge的开源时间晚于OpenYurt，项目成熟度稍差。

如果打算选择一个边缘计算云原生项目用于生产，我会从以下角度考虑：

* 如果需要内置设备管理能力，而对云原生生态兼容性不在意，建议选择KubeEdge
* 如果从云原生生态兼容和项目成熟度考虑，而不需要设备管理能力或者可以自建设备管理能力，建议选择OpenYurt
