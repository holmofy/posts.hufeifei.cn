---
title: 一文图解Kubernetes的持久化存储解决方案
date: 2021-11-19
categories: 后端
mathjax: true
post_src: https://zhuanlan.zhihu.com/p/365780238
tags:
- Kubernetes
- Docker
- PersistentVolume
---

## 概述

Kubernetes（下称k8s）作为目前行业内使用最广泛的容器编排工具，已经深入到各个技术领域，正在彻底改变应用程序的开发和部署方式；但从另一个方面讲，k8s的架构是不断变化的。容器的创建和销毁，从本质上讲，它们的生命周期是短暂的。因而，K8s的发展历程势必无法绕开持久化的问题，本文就将从这一点出发，为大家讲解k8s在持久化存储方面所提供的解决方案，帮助大家更好的理解k8s的整体技术生态。

本文的章节内容分配如下：

- 概述
- K8s有哪些持久化方案
- Docker存储
- K8s原生存储
- 开源存储项目Ceph&Rook
- 总结

------

## K8s有哪些持久化方案

- **外部存储方案：**

先抛一张[CNCF（云原生计算基金会）](https://landscape.cncf.io/)公布的云原生存储解决方案一览图，这里只截取了存储的部分。

![img](https://pic1.zhimg.com/80/v2-cf616af7b8b84824c5381376c82eb50c_720w.jpg)

图中列举的存储方案，目前都可以集成到Kubernetes平台。

- **Docker存储卷**

  当使用Docker作为K8s的容器方案时，Docker自身所支持的存储卷也就成为了可选方案之一。Docker存储卷是容器服务在单节点的存储组织形式，作为解决数据存储、容器运行时的技术方案；

- **K8s存储卷**

  K8s自己的持久化存储方案更关注于应用和集群层面，主要用于容器集群的存储编排，从应用使用存储的角度提供存储服务。另一方面，K8s的持久化存储方案能够完全兼容自身的对象，如Pod对象等，即插即用，无需二次开发。

下面，我们就对这几种存储方案一一进行解释。

## Docker存储

**容器的读写层**

为了提高节点存储的使用效率，容器不光在不同运行的容器之间共享镜像资源，而且还实现了在不同镜像之间共享数据。共享镜像数据的实现原理：镜像是分层组合而成的，即一个完整的镜像会包含多个数据层，每层数据相互叠加、覆盖组成了最终的完整镜像。

![img](https://pic4.zhimg.com/80/v2-d5b7bd1951c46c530e0bd9a402b90047_720w.jpg)

为了实现多个容器间共享镜像数据，容器镜像每一层都是只读的。而容器使用镜像时，在多个镜像分层的最上面还添加了一个读写层。每一个容器在运行时，都会基于当前镜像在其最上层挂载一个读写层，用户针对容器的所有操作都在读写层中完成。一旦容器销毁，这个读写层也随之销毁。

**容器的数据卷**

容器中的应用读写数据都是发生在容器的读写层，镜像层+读写层映射为容器内部文件系统、负责容器内部存储的底层架构。当我们需要容器内部应用和外部存储进行交互时，还需要一个外置存储，容器数据卷即提供了这样的功能。

另一方面，容器本身的存储数据都是临时存储，在容器销毁的时候数据会一起删除。而通过数据卷将外部存储挂载到容器文件系统，应用可以引用外部数据，也可以将自己产出的数据持久化到数据卷中，因此容器数据卷是容器实现数据持久化的主要实现方式。
$$
容器存储组成：只读层（容器镜像） + 读写层 + 外置存储（数据卷）
$$
容器数据卷从作用范围可以分为：单机数据卷 和 集群数据卷。其中单机数据卷即为容器服务在一个节点上的数据卷挂载能力，docker volume 是单机数据卷的代表实现；

Docker Volume 是一个可供多个容器使用的目录，它绕过 UFS，包含以下特性：

> 数据卷可以在容器之间共享和重用；
> 相比通过存储驱动实现的可写层，数据卷读写是直接对外置存储进行读写，效率更高；
> 对数据卷的更新是对外置存储读写，不会影响镜像和容器读写层；
> 数据卷可以一直存在，直到没有容器使用。

### 1）Docker 数据卷类型

Bind：将主机目录/文件直接挂载到容器内部。

> 需要使用主机的上的绝对路径，且可以自动创建主机目录；
> 容器可以修改挂载目录下的任何文件，是应用更具有便捷性，但也带来了安全隐患。

Volume：使用第三方数据卷的时候使用这种方式。

> Volume命令行指令：docker volume (create/rm)；
> 是Docker提供的功能，所以在非 docker 环境下无法使用；
> 分为命名数据卷和匿名数据卷，其实现是一致的，区别是匿名数据卷的名字为随机码；
> 支持数据卷驱动扩展，实现更多外部存储类型的接入。

Tmpfs：非持久化的卷类型，存储在内存中。

> 数据易丢失。

### 2）数据卷使用语法

- **Bind挂载语法**

-v: src:dst:opts 只支持单机版。

> Src：表示卷映射源，主机目录或文件，需要是绝对地址；
> Dst：容器内目标挂载地址；
> Opts：可选，挂载属性：ro, consistent, delegated, cached, z, Z；
> Consistent, delegated, cached：为mac系统配置共享传播属性；
> Z、z：配置主机目录的selinux label。

示例：

```bash
$ docker run -d --name devtest -v /home:/data:ro,rslave nginx
$ docker run -d --name devtest --mount type=bind,source=/home,target=/data,readonly,bind-propagation=rslave nginx
$ docker run -d --name devtest -v /home:/data:z nginx
```

- **Volume 挂载方式语法**

-v: src:dst:opts 只支持单机版。

> Src：表示卷映射源，数据卷名、空；
> Dst：容器内目标目录；
> Opts：可选，挂载属性：ro（只读）。

示例：

```bash
$ docker run -d --name devtest -v myvol:/app:ro nginx
$ docker run -d --name devtest --mount source=myvol2,target=/app,readonly nginx
```

### 3）Docker数据卷插件

Docker 数据卷实现了将容器外部存储挂载到容器文件系统的方式。为了扩展容器对外部存储类型的需求，docker 提出了通过存储插件的方式挂载不同类型的存储服务。扩展插件统称为 Volume Driver，可以为每种存储类型开发一种存储插件。

可以查看链接：https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins

其特性简单来说可以总结为2点：

> 单个节点上可以部署多个存储插件；
> 一个存储插件负责一种存储类型的挂载服务。

![img](https://pic4.zhimg.com/80/v2-e9541900fd6cf76b79e9a5a55a618963_720w.jpg)

关于Volume plugin的代码实现，可以参考这篇小文章：[Docker 17.03-CE create plugin源码解析](https://jimmysong.io/blog/docker-create-plugin/)

Docker Plugin 是以Web Service的服务运行在每一台Docker Host上的，通过HTTP协议传输RPC风格的JSON数据完成通信。Plugin的启动和停止，并不归Docker管理，Docker Daemon依靠在缺省路径下查找Unix Socket文件，自动发现可用的插件。

当客户端与Daemon交互，使用插件创建数据卷时，Daemon会在后端找到插件对应的 socket 文件，建立连接并发起相应的API请求，最终结合Daemon自身的处理完成客户端的请求。

Docker Daemon 与 Volume driver 通信方式有：

- Sock文件：linux 下放在/run/docker/plugins 目录下
- Spec文件：/etc/docker/plugins/convoy.spec 定义
- Json文件：/usr/lib/docker/plugins/infinit.json 定义

实现接口：

> Create, Remove, Mount, Path, Umount, Get, List, Capabilities;

使用示例：

```bash
$ docker volume create --driver nas -o diskid="" -o host="10.46.225.247" -o path=”/nas1" -o mode="" --name nas1
```

Docker Volume Driver 适用在单机容器环境或者 swarm 平台进行数据卷管理，随着 K8s 的流行其使用场景已经越来越少，这里不做赘述。

## K8s原生存储

如果说Docker注重的是单节点的存储能力，那K8s 数据卷关注的则是集群级别的数据卷编排能力。

### 卷-Volume

Kubernetes 提供是卷存储类型，从存在的生命周期可分为临时和持久卷。 从卷的类型分，又可以分为本地存储、网络存储、Secret/ConfigMap、CSI/Flexvolume、PVC；有兴趣的小伙伴可以参考一下官方文档：https://kubernetes.io/zh/docs/concepts/storage/volumes/

这里就以一幅图来展示各个存储的存在形式。

![img](https://pic2.zhimg.com/80/v2-cfd94cca4b32bcf42cdd253a5812aff5_720w.jpg)

如上图所示：

- 最上层的pod和PVC由用户管理，pod创建volume卷，并指定存储方式。
- 中间部分由集群管理者创建StorageClass对象，StorageClass只需确定PV属性（存储类型，大小等）及创建PV所需要用的的存储插件；K8s会自动根据用户提交的PVC来找到对应的StorageClass，之后根据其定义的存储插件，创建出PV。
- 最下层指代各个实际的存储资源。

### PV和PVC

这里单独来聊聊PV和PVC，也是实际应用场景中最常用的一组概念，其中：

PV 是 PersistentVolume 的缩写，译为持久化存储卷；PV 在 K8s 中代表一个具体存储类型的卷，其对象中定义了具体存储类型和卷参数。即目标存储服务所有相关的信息都保存在 PV 中，K8s 引用 PV 中的存储信息执行挂载操作。

PVC的存在，是从应用角度对存储卷进行二次抽象；由于 PV 描述的是对具体存储类型，需要定义详细的存储信息，而应用层用户在消费存储服务的时候往往不希望对底层细节知道的太多，让应用编排层面来定义具体的存储服务不够友好。这时对存储服务再次进行抽象，只把用户关系的参数提炼出来，用 PVC 来抽象更底层的 PV。所以 PVC、PV 关注的对象不一样，PVC 关注用户对存储需求，给用户提供统一的存储定义方式；而 PV 关注的是存储细节，可以定义具体存储类型、存储挂载使用的详细参数等。其具体的对应关系如下图所示：

![img](https://pic1.zhimg.com/80/v2-bd7774b3daa4dabeca12532ab77f4850_720w.jpg)

### 使用方法

PVC 只有绑定了 PV 之后才能被 Pod 使用，而 PVC 绑定 PV 的过程即是消费 PV 的过程，这个过程是有一定规则的，下面规则都满足的 PV 才能被 PVC 绑定：

- VolumeMode：被消费 PV 的 VolumeMode 需要和 PVC 一致；
- AccessMode：被消费 PV 的 AccessMode 需要和 PVC 一致；
- StorageClassName：如果 PVC 定义了此参数，PV 必须有相关的参数定义才能进行绑定；
- LabelSelector：通过 label 匹配的方式从 PV 列表中选择合适的 PV 绑定；
- storage：被消费 PV 的 capacity 必须大于或者等于 PVC 的存储容量需求才能被绑定。

PVC模板：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: disk-1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: test-disk
  volumeMode: Filesystem
```

PV模板：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    failure-domain.beta.kubernetes.io/region: cn-zn
    failure-domain.beta.kubernetes.io/zone: cn-zn
  name: d-wz9g2j5qbo37r2lamkg4
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 30Gi
  flexVolume:
    driver: alicloud/disk
    fsType: ext4
    options:
      VolumeId: d-wz9g2j5qbo37r2lamkg4
  persistentVolumeReclaimPolicy: Delete
  storageClassName: test-disk
  volumeMode: Filesystem
```

## 开源存储项目Ceph&Rook

围绕云原生技术的工具和项目正在大量涌现。作为生产中最突出的问题之一，有相当一部分开源项目致力于解决“在云原生架构上处理存储”这个问题。

目前最受欢迎的存储项目是**Ceph**和**Rook**。

Ceph是一个动态管理的、水平可伸缩的分布式存储集群。Ceph提供了对存储资源的逻辑抽象。它被设计成不存在单点故障、可自我管理和基于软件的。Ceph同时为相同的存储集群提供块、对象或文件系统接口。它能够提供非常稳定的块存储系统，并且K8S对Ceph放出了完整的生态，几乎可以说是全面兼容。

Ceph的架构非常复杂，有许多底层技术，如RADOS、librados、RADOSGW、RDB，它的CRUSH 算法和监视器、OSD和MDS等组件。这里不深入解读其架构，关键在于，Ceph是一个分布式存储集群，它可提供更高的可伸缩性，在不牺牲性能的情况下消除了单点故障，并提供了对对象、块和文件的访问的统一存储。

![img](https://pic3.zhimg.com/80/v2-0d1ff3204a1eba68463e9fc56b2b7e0e_720w.jpg)



对于Rook，我们可以从以下几点来了解这个有趣的项目。它旨在聚合Kubernetes和Ceph的工具——将计算和存储放在一个集群中。

- Rook 是一个开源的cloud-native storage编排, 提供平台和框架；为各种存储解决方案提供平台、框架和支持，以便与云原生环境本地集成。
- Rook 将存储软件转变为自我管理、自我扩展和自我修复的存储服务，它通过自动化部署、引导、配置、置备、扩展、升级、迁移、灾难恢复、监控和资源管理来实现此目的。
- Rook 使用底层云本机容器管理、调度和编排平台提供的工具来实现它自身的功能。
- Rook 目前支持Ceph、NFS、Minio Object Store和CockroachDB。
- Rook使用Kubernetes原语使Ceph存储系统能够在Kubernetes上运行。

所以在ROOK的帮助之下我们甚至可以做到一键编排部署Ceph，同时部署结束之后的运维工作ROOK也会介入自动进行实现对存储拓展，即便是集群出现了问题ROOK也能在一定程度上保证存储的高可用性，绝大多数情况之下甚至不需要Ceph的运维知识都可以正常使用。

### 安装方法

1. **获取rook仓库：**https://github.com/rook/rook.git
2. **获取部署yaml文件**，在rook仓库之中的[cluster/examples/kubernetes/ceph/common.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/common.yaml) 文件。

```bash
#运行common.yaml文件 
kubectl create -f common.yaml
```

**3. 安装operator，编排文件为[/cluster/examples/kubernetes/ceph/operator.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/operator.yaml)**

```bash
#运行operator.yaml文件
kubectl create -f operator.yaml
```

**4. 安装完成之后需要等待所有的操作器正常运行之后才能继续还是ceph分部署集群的安装：**

```bash
#获取命名空间下运行的pod，等待所以的pod都是running状态之后继续下一步
kubectl -n rook-ceph get pod
```

**5. 创建Ceph集群，编排文件为[/cluster/examples/kubernetes/ceph/cluster.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/cluster.yaml)**

这里也需要进行一定的基础配置与修改才能继续，cluster.yaml文件内容如下：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephCluster
metadata:
  name: rook-ceph
  namespace: rook-ceph
spec:
  cephVersion:
    image: ceph/ceph:v14.2.6
    allowUnsupported: false
  dataDirHostPath: /var/lib/rook
  skipUpgradeChecks: false
  continueUpgradeAfterChecksEvenIfNotHealthy: false
  mon:
    #这里是最重要的，mon是存储集群的监控器，我们K8S里面有多少主机这里的就必须使用多少个mon
    count: 3
    allowMultiplePerNode: false
  dashboard:
   #这里是是否启用监控面板，基本上都会使用 
    enabled: true
    #监控面板是否使用SSL，如果是使用8443端口，不是则使用7000端口，由于这是运维人员使用建议不启用
    ssl: true
  monitoring:
    enabled: false
    rulesNamespace: rook-ceph
  network:
    hostNetwork: false
  rbdMirroring:
    workers: 0
  crashCollector:
    disable: false
  annotations:
  resources:
  removeOSDsIfOutAndSafeToRemove: false
  storage:
    useAllNodes: true
    useAllDevices: true
    config:
  disruptionManagement:
    managePodBudgets: false
    osdMaintenanceTimeout: 30
    manageMachineDisruptionBudgets: false
    machineDisruptionBudgetNamespace: openshift-machine-api
```



```bash
#运行cluster.yaml文件
kubectl create -f cluster.yaml
```

**6. 创建ceph控制面板**

如果上面部署时，启用了SSL则需要使用[/cluster/examples/kubernetes/ceph/dashboard-external-https.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/dashboard-external-https.yaml)，否则使用同目录下的[dashboard-external-http.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/dashboard-external-http.yaml)文件：

```bash
#dashboard没有启用SSL
kubectl create -f dashboard-external-http.yaml
#dashboard启用SSL
kubectl create -f dashboard-external-https.yaml
```

**7. 创建Ceph工具**

运维人员可以直接通过对这个容器的shell进行Ceph集群的控制（后面有实例），编排文件是[toolbox.yaml](https://github.com/rook/rook/blob/master/cluster/examples/kubernetes/ceph/toolbox.yaml)

```bash
#安装Ceph工具
kubectl create -f toolbox.yaml
```

**8，创建存储系统与存储类**

集群搭建完毕之后便是存储的创建，目前Ceph支持块存储、文件系统存储、对象存储三种方案，K8S官方对接的存储方案是块存储，他也是比较稳定的方案，但是块存储目前不支持多主机读写；文件系统存储是支持多主机存储的性能也不错；对象存储系统IO性能太差不考虑，所以可以根据要求自行决定。

存储系统创建完成之后对这个系统添加一个存储类之后整个集群才能通过K8S的存储类直接使用Ceph存储。

- 块存储系统+块存储类yaml文件：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephBlockPool
metadata:
  name: replicapool
  namespace: rook-ceph
spec:
  failureDomain: host
  replicated:
    size: 3 #这里的数字分部署数量，一样有几台主机便写入对应的值
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
    clusterID: rook-ceph
    pool: replicapool
    imageFormat: "2"
    imageFeatures: layering
    csi.storage.k8s.io/provisioner-secret-name: rook-csi-rbd-provisioner
    csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
    csi.storage.k8s.io/node-stage-secret-name: rook-csi-rbd-node
    csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
    csi.storage.k8s.io/fstype: xfs
reclaimPolicy: Delete
```

- 文件系统存储yaml文件：

```yaml
apiVersion: ceph.rook.io/v1
kind: CephFilesystem
metadata:
  name: myfs
  namespace: rook-ceph
spec:
  metadataPool:
    replicated:
      size: 3 #这里的数字分部署数量，一样有几台主机便写入对应的值
  dataPools:
    - replicated: 
        size: 3 #这里的数字分部署数量，一样有几台主机便写入对应的值
  preservePoolsOnDelete: true
  metadataServer:
    activeCount: 1
    activeStandby: true
```

- 文件系统存储类yaml文件：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: csi-cephfs
provisioner: rook-ceph.cephfs.csi.ceph.com
parameters:
  clusterID: rook-ceph
  fsName: myfs
  pool: myfs-data0
  csi.storage.k8s.io/provisioner-secret-name: rook-csi-cephfs-provisioner
  csi.storage.k8s.io/provisioner-secret-namespace: rook-ceph
  csi.storage.k8s.io/node-stage-secret-name: rook-csi-cephfs-node
  csi.storage.k8s.io/node-stage-secret-namespace: rook-ceph
reclaimPolicy: Delete
```

## 总结

本文通过介绍并图解K8s中各个存储方案实现方式以及可选择的开源项目，为读者呈现更全面的K8s存储方案选择。在我们实际的使用场景中，亦需要根据特定的需求来制定符合项目要求的存储方案，从而达到最好的实现效果。也希望有更多的朋友能够加入到kubernetes的队伍中来，让kubernetes真正深入到众多的用户和企业中去。

