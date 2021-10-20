---
title: Kubernetes解决了哪些问题
date: 2021-10-04
categories: 后端
mathjax: true
post_src: https://www.zhihu.com/question/329365548/answer/1545488275
tags: 
- Kubernetes
- Docker
---

随着容器的火爆，利用容器架构来搭建业务系统的人越来越多。可是，大家在实操中发现，像 Docker之类的容器引擎，折腾少量容器还行。但如今的云原生应用、机器学习任务或者大数据分析业务，动辄就要使用成百上千的容器。要管理这么多容器，Docker们就力不从心了。

![img](https://pica.zhimg.com/80/v2-6009ceb058505c1fe3a3f1a94ef131ed_1440w.jpg?source=1940ef5c)

江山代有才人出，各领风骚三五年，有需求就有改变，于是乎，市场上就出现了一批容器编排工具，典型的是 Swarm、Mesos 和 K8S。

**经过几年大浪淘沙，K8S “击败” Swarm 和 Mesos，几乎成了当前容器编排的事实标准。**

![img](https://pic2.zhimg.com/80/v2-36e7d61356ff58a78cf57d4ee7c88cc3_1440w.jpg?source=1940ef5c)

K8S 最初是由 Google 开发的，后来捐赠给了 CNCF（云原生计算基金会，隶属 Linux 基金会）。

![img](https://pic2.zhimg.com/80/v2-0c2cdb3632ae1d8f21c481a9cb3b729d_1440w.jpg?source=1940ef5c)

K8S 的全名是 kubernetes，读作“库伯耐踢死”，很多国人既拼不对也写不对，而 K 和 S 之间有 8 个字母，索性就简单一点，叫“开八司”了。

K8S 是个杂技高手，最擅长的就是“搬箱子”，盘各种容器玩。

![img](https://pic3.zhimg.com/80/v2-35b39d0fd312c19b519f313cfa60d50f_1440w.jpg?source=1940ef5c)

K8S 的大致架构，就像上面。Master 节点，用来放“脑子”，“腿脚”搭在工作节点上“搬砖”，工作节点就是实际业务容器的存放地。

单个容器或多个关系密切的容器，被编成一组，称为 pod。K8S 就是以 pod 为单位进行编排操作。

同时，K8S 还要和其它相关软件配合，来完成联网、存储、安全等功能。

![img](https://pic3.zhimg.com/80/v2-ddff0e1a01733f832d3ff67ad94b29b3_1440w.jpg?source=1940ef5c)

诞生六年来，K8S 一路高歌，成为容器编排和调度领域的 No.1。但需要注意的是，K8S 和 Docker 们不是替代关系，而是配合关系。

K8S 仍然会使用 Docker 之类的容器引擎（Docker、Containerd、RKT、CRI-O 等），来对容器进行生命周期管理。

**K8S 既然那么猛，直接拿来用不香吗？**

这样做，看起来没毛病，K8S 是开源软件，社区版 K8S 也很完美。

你可以在网上找到各种安装指导文档，然后从 github 轻松找到最新的版本，然后一步一步搭建集群。

只是安装过程漫长而痛苦，毕竟搭建集群不是我们的目的，我们的目的是利用集群来干活。

![img](https://pic1.zhimg.com/80/v2-5db516d3a2a228a1c081361daa1677b0_1440w.jpg?source=1940ef5c)

搭一个 K8S 学习环境倒也罢了，权当练手涨经验。可当我们要搭建生产环境的时候，事情就变得不一样了。

这时候，为了保证集群的可靠性，我们可能要跨多个可用区来部署 K8S 集群。对于大多数人来说，这个工作不太好玩。

![img](https://pic3.zhimg.com/80/v2-e6f3e4ce7db599dd09d8f22f8b782597_1440w.jpg?source=1940ef5c)

不止搭建集群过程很复杂，后期还要面对更繁琐的 K8S 控制平面维护工作：版本升级、安全管控、数据备份等等。

所以，面对生产级别的业务，大家往往喜欢选择 Turnkey （一站式）的商用方案，而不是自己慢慢鼓捣，老牛拉破车。

目前，各大云服务商几乎都推出了 Turnkey 方案，帮助用户快速搭建 K8S 集群。

到底哪家强呢？王婆卖瓜，自卖自夸，似乎没有定论。

但是有个数据很有参考意义，根据咨询机构「Nucleus Research」的数据，**所有云中 K8S 的工作负载，竟然有 82%都是运行在亚马逊云科技 (Amazon Web Services)  上的**。

![img](https://pica.zhimg.com/80/v2-783ce8e1cad908c7d3978cde57fdabe1_1440w.jpg?source=1940ef5c)

So，我们差不多可以这样说，**云上 K8S，还是亚马逊云科技最强！**

亚马逊云科技提供了一个神器，叫做 Amazon Elastic Kubernetes Service (Amazon EKS)，可以快速帮我们搭建高可用的云上托管 K8S 服务。

![img](https://pic3.zhimg.com/80/v2-604c9eb28ba06c56bb281167b46bfd33_1440w.jpg?source=1940ef5c)

**Amazon EKS 到底牛在哪儿？**

作为一个从来没摸过 K8S 的生手，我用了不到 10 分钟，就创建了一个横跨 3 个可用区的生产级集群，实在太魔幻了。

**整个过程，只需要区区两步**

![img](https://pica.zhimg.com/80/v2-bd443913bcebd8dc86c6076465897b0a_1440w.jpg?source=1940ef5c)

在添加工作节点的时候，可以选择各种 Amazon EC2 实例，亚马逊云科技准备了丰富的实例类型，满足不同的容器用途。

当然，还可以选择新酷的 Amazon Fargate 工作节点，这是一种 Serverless 的方式，说白了，你不需要去考虑什么实例呀、服务器呀，直接按需使用容器即可，要多少有多少，计费精确到容器，而非主机。

集群创建完成后，我们就可采用自己习惯的工具，比如 `kubectl`，像使用标准 K8S 集群一样，进行各种业务部署的操作了。

![img](https://pica.zhimg.com/50/v2-6338c62c0ba0a270c7970abfb6def3ba_720w.jpg?source=1940ef5c)

除了**简单、易用、生产级高可用**以外，Amazon EKS 与社区版的 K8S 是保持同步的，原生体验完全一致，可以使用社区所有插件和工具…

So，不需要额外的学习成本，也不用担心锁定，轻松迁移。

![img](https://pic1.zhimg.com/80/v2-603aaaf6a61702992ead3fe17c04a97d_1440w.jpg?source=1940ef5c)

作为云上 K8S 大户，亚马逊云科技也充分发扬开源精神，源于社区、反哺社区，不断为 K8S 项目做贡献，推动 K8S 的改进。

![img](https://pic2.zhimg.com/80/v2-c853a3606ac2f918a1d1d7abcfaa0c01_1440w.jpg?source=1940ef5c)

亚马逊云科技为 Amazon EKS 提供了多达 270 种节点，可以满足所有工作负载和业务需求，并提供为 Amazon EKS 定制优化的操作系统镜像，高效、安全、开源。

同时，Amazon EKS 还与亚马逊云科技其他服务无缝集成，诸如负载均衡、弹性伸缩、身份认证、存储、安全、监控、日志，用户不需要苦逼滴自己造轮子，站在亚马逊云科技肩膀上就行。

更令人心动的是，不止于 Amazon EKS，围绕容器、K8S、微服务这些云原生的关键技术，亚马逊云科技提供了一揽子解决方案。

随着云计算进入深水区，云原生的理念越来越深入人心，利用亚马逊云科技的「容器全家桶」，用户可以**轻松搭建各种高可用「云原生」服务**，**把上云的价值最大化**。
