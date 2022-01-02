---
title: 谈阿里核心业务监控平台 SunFire 的技术架构
date: 2021-11-22
categories: 后端
mathjax: true
post_src: https://www.infoq.cn/article/alibaba-core-business-monitoring-platform-sunfire
tags:
- Alibaba
- monitor
- Sunfire
---

在 2016 年双 11 全球购物狂欢节中，天猫全天交易额 1207 亿元，前 30 分钟每秒交易峰值 17.5 万笔，每秒支付峰值 12 万笔。承载这些秒级数据背后的监控产品是如何实现的呢？接下来本文将从阿里监控体系、监控产品、监控技术架构及实现分别进行详细讲述。

阿里有众多监控产品，且各产品分工明确，百花齐放。整个阿里监控体系如下图：

![img](https://static001.infoq.cn/resource/image/bb/2b/bb8fd397d31beaa3d0bc0d16f519902b.jpg)

集团层面的监控，以平台为主，全部为阿里自主研发（除引入了第三方基调、博睿等外部检测系统，用于各地 CDN 用户体验监控），这些监控平台覆盖了阿里集团 80% 的监控需求。

此外，每个事业群均根据自身特性自主研发了多套监控系统，以满足自身特定业务场景的监控需求，如广告的 GoldenEye、菜鸟的棱镜、阿里云的天基、蚂蚁的金融云（基于 XFlush）、中间件的 EagleEye 等，这些监控系统均有各自的使用场景。

阿里的监控规模早已达到了千万量级的监控项，PB 级的监控数据，亿级的报警通知，基于数据挖掘、机器学习等技术的智能化监控将会越来越重要。

阿里全球运行指挥中心（GOC）基于历史监控数据，通过异常检测、异常标注、挖掘训练、机器学习、故障模拟等方式，进行业务故障的自动化定位，并赋能监控中心 7*24 小时专业监控值班人员，使阿里集团具备第一时间发现业务指标异常，并快速进行应急响应、故障恢复的能力，将故障对线上业务的影响降到最低。

接下来将详细讲述本文的主角，承载阿里核心业务监控的 SunFire 监控平台。

## SunFire 简介

SunFire 是一整套海量日志实时分析解决方案，以日志、REST 接口、Shell 脚本等作为数据采集来源，提供设备、应用、业务等各种视角的监控能力，从而帮您快速发现问题、定位问题、分析问题、解决问题，为线上系统可用率提供有效保障。

SunFire 利用文件传输、流式计算、分布式文件存储、数据可视化、数据建模等技术，提供实时、智能、可定制、多视角、全方位的监控体系。其主要优势有：

- 全方位实时监控：提供设备、应用、业务等各种视角的监控能力，关键指标秒级、普通指标分钟级，高可靠、高时效、低延迟。
- 灵活的报警规则：可根据业务特征、时间段、重要程度等维度设置报警规则，实现不误报、不漏报。
- 管理简单：分钟级万台设备的监控部署能力，故障自动恢复，集群可伸缩
- 自定义便捷配置：丰富的自定义产品配置功能，便捷、高效的完成产品配置、报警配置。
- 可视化：丰富的可视化 Dashboard，帮助您定制个性化的监控大盘。
- 低资源占用：在完成大量监控数据可靠传输的同时，保证对宿主机的 CPU、内存等资源极低占用率。

Sunfire 技术架构如下：

![Sunfire](https://static001.infoq.cn/resource/image/fe/c3/fe8537a7abd2fc1bd30381ab1b30c4c3.jpg)

### 主要模块实现及功能

针对架构图中的各个组件，其中最关键的为采集（Agent）、计算（Map、Reduce）组件，接下来将对这两个组件进行详细介绍。

### 1. 采集

Agent 负责所有监控数据的原始采集，它以 Agent 形式部署在应用系统上，负责原始日志的采集、系统命令的执行。日志原始数据的采集，按周期查询日志的服务，且日志查询要低耗、智能。Agent 上不执行计算逻辑。主要具有以下特点：

**低耗**

采集日志，不可避免要考虑日志压缩的问题，通常做日志压缩则意味着它必须做两件事情：一是磁盘日志文件的内容要读到应用程序态；二是要执行压缩算法。

这两个过程就是 CPU 消耗的来源。但是它必须做压缩，因为日志需要从多个机房传输到集中的机房。跨机房传输占用的带宽不容小觑，必须压缩才能维持运转。所以低耗的第一个要素，就是避免跨机房传输。SunFire 达到此目标的方式是运行时组件自包含在机房内部，需要全量数据时才从各机房查询合并。

网上搜索 zero-copy，会知道文件传输其实是可以不经过用户态的，可以在 Linux 的核心态用类似 DMA 的思想，实现极低 CPU 占用的文件传输。SunFire 的 Agent 当然不能放过这个利好，对它的充分利用是 Agent 低耗的根本原因。以前这部分传输代码是用 c 语言编写的 sendfile 逻辑，集成到 Java 工程里，后来被直接改造为了 Java 实现。

最后，在下方的计算平台中会提到，要求 Agent 的日志查询服务具备“按周期查询日志”的能力。这是目前 Agent 工程里最大的难题，我们都用过 RAF（RandomAccessFile），你给它一个游标，指定 offset，再给它一个长度，指定读取的文件 size，它可以很低耗的扒出文件里的这部分内容。然而问题在于：周期≠offset。从周期转换为 offset 是一个痛苦的过程。

在流式计算里一般不会遇到这个问题，因为在流式架构里，Agent 是水龙头，主动权掌握在 Agent 手里，它可以从 0 开始 push 文件内容，push 到哪里就做一个标记，下次从标记继续往后 push，不断重复。这个标记就是 offset，所以流式不会有任何问题。

而计算平台周期任务驱动架构里，pull 的方式就无法提供 offset，只能提供 Term（周期，比如 2015-11-11 00:00 分）。Agent 解决此问题的方式算是简单粗暴，那就是二分查找法。而且日志还有一个天然优势，它是连续性的。所以按照对二分查找法稍加优化，就能达到“越猜越准”的效果（因为区间在缩小，区间越小，它里面的日志分布就越平均）。

于是，Agent 代码里的 LogFinder 组件撑起了这个职责，利用上述两个利好，实现了一个把 CPU 控制在 5% 以下的算法，目前能够维持运转。其中 CPU 的消耗不用多说，肯定是来自于猜的过程，因为每一次猜测，都意味着要从日志的某个 offset 拉出一小段内容来核实，会导致文件内容进入用户态并解析。这部分算法依然有很大的提升空间。

**日志滚动**

做过 Agent 的同学肯定都被日志滚动困扰过，各种各样的滚动姿势都需要支持。SunFire 的 pull 方式当然也会遇到这个问题，于是我们简单粗暴的穷举出了某次 pull 可能会遇到的所有场景，比如：

- 正常猜到了 offset
- 整个日志都猜不到 offset
- 上次猜到了 offset，但是下次再来的时候发现不对劲（比如滚动了）

这段逻辑代码穷举的分支之多，在一开始谁都没有想到。不过仔细分析了很多次，发现每个分支都必不可少。

**查询接口**

Agent 提供的查询服务分为 first query 和 ordinary query 两种。做这个区分的原因是：一个周期的查询请求只有第一次需要猜 offset，之后只需要顺序下移即可。而且计算组件里有大量的和 Agent 查询接口互相配合的逻辑，比如一个周期拉到什么位置上算是确定结束？一次 ordinary query 得到的日志里如果末尾是截断的（只有一半）该如何处理…… 这些逻辑虽然缜密，但十分繁琐，甚至让人望而却步。但现状如此，这些实现逻辑保障了 SunFire 的高一致性，不用担心数据不全、报警不准，随便怎么重启计算组件，随便怎么重启 Agent。但这些优势的背后，是值得深思的代码复杂度。

**路径扫描**

为了让用户配置简单便捷，SunFire 提供给用户选择日志的方式不是手写，而是像 windows 的文件夹一样可以浏览线上系统的日志目录和文件，让他双击一个心仪的文件来完成配置。但这种便捷带来的问题就是路径里若有变量就会出问题。所以 Agent 做了一个简单的 dir 扫描功能。Agent 能从应用目录往下扫描，找到同深度文件夹下“合适”的目标日志。

### 2. 计算

由 Map、Reduce 组成计算平台，负责所有采集内容的加工计算，具备故障自动恢复能力及弹性伸缩能力。计算平台一直以来都是发展最快、改造最多的领域，因为它是很多需求的直接生产者，也是性能压力的直接承担者。因此，在经过多年的反思后，最终走向了一条插件化、周期驱动、自协调、异步化的道路。主要具有以下几个特点：

**纯异步**

原来的 SunFire 计算系统里，线程池繁复，从一个线程池处理完还会丢到下一个线程池里；为了避免并发 bug，加锁也很多。这其中最大的问题有两个：CPU 密集型的逻辑和 I/O 密集型混合。

对于第一点，只要发生混合，无论你怎么调整线程池参数，都会导致各式各样的问题。线程调的多，会导致某些时刻多线程抢占 CPU，load 飙高；线程调的少，会导致某些时刻所有线程都进入阻塞等待，堆积如山的活儿没人干。

对于第二点，最典型的例子就是日志包合并。比如一台 Map 上的一个日志计算任务，它要收集 10 个 Agent 的日志，那肯定是并发去收集的，10 个日志包陆续（同时）到达，到达之后各自解析，解析完了 data 要进行 merge。这个 merge 过程如果涉及到互斥区（比如嵌套 Map 的填充），就必须加锁，否则 bug 满天飞。

但其实我们重新编排一下任务就能杜绝所有的锁。比如上面的例子，我们能否让这个日志计算任务的 10 个 Agent 的子任务，全部在同一个线程里做？这当然是可行的，只要回答两个问题就行：

- 如果串行，那 10 个 I/O 动作（拉日志包）怎么办？串行不就浪费 cpu 浪费时间吗？
- 把它们都放到一个线程里，那我怎么发挥多核机器的性能？

第一个问题，答案就是异步 I/O。只要肯花时间，所有的 I/O 都可以用 NIO 来实现，无锁，事件监听，不会涉及阻塞等待。即使串行也不会浪费 CPU。第二个问题，就是一个大局观问题了。现实中我们面临的场景往往是用户配置了 100 个产品，每个产品都会拆解为子任务发送到每台 Map，而一台 Map 只有 4 个核。所以，你让一个核负责 25 个任务已经足够榨干机器性能了，没必要追求更细粒度子任务并发。因此，计算平台付出了很大的精力，做了协程框架。

我们用 Akka 作为协程框架，有了协程框架后再也不用关注线程池调度等问题了，于是我们可以轻松的设计协程角色，实现 CPU 密集型和 I/O 密集型的分离、或者为了无锁而做任务编排。接下来，尽量用 NIO 覆盖所有的 I/O 场景，杜绝 I/O 密集型逻辑，让所有的协程都是纯跑 CPU。按照这种方式，计算平台已经基本能够榨干机器的性能。

## 周期驱动

所谓周期驱动型任务调度，说白了就是 Map/Reduce。Brain 被选举出来之后，定时捞出用户的配置，转换为计算作业模型，生成一个周期（比如某分钟的）的任务, 我们称之为拓扑 (Topology), 拓扑也很形象的表现出 Map/Reduce 多层计算结构的特征。所有任务所需的信息，都保存在 topology 对象中，包括计算插件、输入输出插件逻辑、Map 有几个、每个 Map 负责哪些输入等等。这些信息虽然很多，但其实来源可以简单理解为两个：一是用户的配置；二是运维元数据。拓扑被安装到一台 Reduce 机器（A）上。

A 上的 Reduce 任务判断当前集群里有多少台 Map 机器，就产生多少个任务（每个任务被平均分配一批 Agent），这些任务被安装到每台机器上 Map。被安装的 Map 任务其实就是一个协程，它负责了一批 Agent，于是它就将每个 Agent 的拉取任务再安装为一个协程。至此，安装过程结束。Agent 拉取任务协程（也称之为 input 输入协程，因为它是数据输入源）在周期到点后，开始执行，拉取日志，解析日志，将结果交予 Map 协程；Map 协程在得到了所有 Agent 的输入结果并 merge 完成后，将 merge 结果回报到 Reduce 协程（这是一个远程协程消息，跨机器）；Reduce 协程得到了所有 Map 协程的汇报结果后，数据到齐，写入到 Hbase 存储，结束。

上述过程非常简单，不高不大也不上，但经过多年大促的考验，其实非常的务实。能解决问题的架构，就是好的架构，能简单，为何要把事情做得复杂呢？

这种架构里，有一个非常重要的特性：任务是按周期隔离的。也就是说，同一个配置，它的 2015-11-11 00:00 分的任务和 2015-11-11 00:01 分的任务，是两个任务，没有任何关系，各自驱动，各自执行。理想情况下，我们可以做出结论：一旦某个周期的任务结束了，它得到的数据必然是准确的（只要每个 Agent 都正常响应了）。所以采用了这种架构之后，SunFire 很少再遇到数据不准的问题，当出现业务故障的时候我们都可以断定监控数据是准确的，甚至秒级都可以断定是准确的，因为秒级也就是 5 秒为周期的任务，和分钟级没有本质区别，只是周期范围不同而已。能获得这个能力当然也要归功于 Agent 的“按周期查询日志”的能力。

## 任务重试

在上节描述的 Brain->Reduce->Map 的任务安装流程里，我们对每一个上游赋予一个职责：监督下游。当机器发生重启或宕机，会丢失一大批协程角色。每一种角色的丢失，都需要重试恢复。监督主要通过监听 Terminated 事件实现，Terminated 事件会在下游挂掉 (不论是该协程挂掉还是所在的机器挂掉或是断网等) 的时候发送给上游。由于拓扑是提前生成好且具备完备的描述信息，因此每个角色都可以根据拓扑的信息来重新生成下游任务完成重试。

- 若 Brain 丢失，则 Brain 会再次选主, Brain 读取最后生成到的任务周期, 再继续生成任务。
- 若 Reduce 丢失，每个任务在 Brain 上都有一个 TopologySupervisor 角色, 来监听 Reduce 协程的 Terminated 事件来执行重试动作。
- 若 Map 丢失，Reduce 本身也监听了所有 Map 的 Terminated 事件来执行重试动作。
- 为确保万无一失，若 Reduce 没有在规定时间内返回完成事件给 Brain，Brain 同样会根据一定规则重试这个任务。

过程依然非常简单，而且从理论上是可证的，无论怎么重启宕机，都可以确保数据不丢，只不过可能会稍有延迟（因为部分任务要重新做）。

输入共享：在用户实际使用 SunFire 的过程中，常常会有这样的情况：用户配了几个不同的配置，其计算逻辑可能是不同的，比如有的是单纯计算行数，有的计算平均值，有的需要正则匹配出日志中的内容，但这几个配置可能都用的同一份日志，那么一定希望几个配置共享同一份拉取的日志内容。否则重复拉取日志会造成极大的资源消耗。那么我们就必须实现输入共享，输入共享的实现比较复杂，主要依赖两点：

- 其一是依赖安装流，因为拓扑是提前安装的，因此在安装到真正开始拉取日志的这段时间内，我们希望能够通过拓扑信息判断出需要共享的输入源，构建出输入源和对应 Map 的映射关系。
- 其二是依赖 Map 节点和 Agent 之间的一致性哈希，保证 Brain 在生成任务时，同一个机器上的日志，永远是分配相同的一个 Map 节点去拉取的（除非它对应的 Map 挂了）。

站在 Map 节点的视角来看：在各个任务的 Reduce 拿着拓扑来注册的时候，我拿出输入源（对日志监控来说通常可以用一个 IP 地址和一个日志路径来描述）和 Map 之间的关系，暂存下来，每当一个新的 Reduce 来注册 Map，我都判断这个 Map 所需的输入源是否存在了，如果有，那就给这个输入源增加一个上游，等到这个输入源的周期到了，那就触发拉取，不再共享了。

## 其他组件

**存储**：负责所有计算结果的持久化存储，可以无限伸缩，且查询历史数据保持和查询实时数据相同的低延迟。Sunfire 原始数据存储使用的是阿里集团的 Hbase 产品（HBase ：Hadoop Database，是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统），用户配置存储使用的是 MongoDB。

**展示**：负责提供用户交互，让用户通过简洁的建模过程来打造个性化的监控产品。基于插件化、组件化的构建方式，用户可以快速增加新类型的监控产品。

**自我管控**：即 OPS-Agent、Ops-web 组件，负责海量 Agent 的自动化安装监测，并且承担了整个监控平台各个角色的状态检测、一键安装、故障恢复、容量监测等职责。