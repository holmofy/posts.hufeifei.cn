---
title: Google 后 Hadoop 时代的新 “三驾马车” -- Caffeine(搜索)、Pregel(图计算)、Dremel(查询)
date: 2021-11-22
categories: 后端
mathjax: true
post_src: https://www.cnblogs.com/chenmingjun/p/10758270.html
tags:
- Caffeine
- Hadoop
---

> **摘要**：Google 在 2003 年到 2004 年公布了关于 GFS、MapReduce 和 BigTable 三篇技术论文（旧三驾马车），这也成为后来`云计算发展的重要基石`，如今 Google 在后 Hadoop 时代的新“三驾马车” -- Caffeine、Pregel、Dremel 再一次影响着全球大数据技术的发展潮流。

Mike Olson(迈克尔·奥尔森) 是 Hadoop 运动背后的主要推动者，但这还远远不够，目前 Google 内部使用的大数据软件 Dremel 使大数据处理起来更加智能。

Mike Olson 目前任职于世界上最热的软件专业公司 -- Cloudera（硅谷的创业企业），并担任 Cloudera 的首席执行官。Cloudera 围绕开源软件平台 Hadoop 发展自身的业务，开源软件平台 Hadoop 已经使得 Google 变身网络上最主导的力量。

预计到 2016 年 Hadoop 将会推动软件市场，并创造 8.13 亿美元的价值。不过 Mike Olson 表示这已经是老新闻了。

Hadoop 的火爆要得益于 Google 在 2003 年底和 2004 年公布的两篇研究论文，其中一份描述了 GFS（Google File System），`GFS 是一个可扩展的大型数据密集型应用的分布式文件系统`，该文件系统可在廉价的硬件上运行，并具有可靠的容错能力，该文件系统可为用户提供极高的计算性能，而同时具备最小的硬件投资和运营成本。

另外一篇则描述了 MapReduce，`MapReduce 是一种处理大型及超大型数据集并生成相关执行的编程模型`。其主要思想是从函数式编程语言里借来的，同时也包含了从矢量编程语言里借来的特性。基于 MapReduce 编写的程序是在成千上万的普通 PC 机上被并行分布式自动执行的。8 年后，Hadoop 已经被广泛使用在网络上，并涉及数据分析和各类数学运算任务。但 Google 却提出更好的技术。

在 2009 年，网络巨头开始使用新的技术取代 GFS 和 MapReduce。Mike Olson 表示 “这些技术代表未来的趋势。如果你想知道大规模、高性能的数据处理基础设施的未来趋势如何，我建议你看看 Google 即将推出的研究论文”。

自Hadoop 兴起以来，Google 已经发布了三篇研究论文，主要阐述了`基础设施如何支持庞大网络操作`。其中一份详细描述了 Caffeine，`Caffeine 主要 为Google 网络搜索引擎提供支持`。

在 Google 采用 Caffeine 之前，Google 使用 MapReduce 和分布式文件系统（如 GFS）来构建搜索索引（从已知的 Web 页面索引中）。在 2010 年，Google 搜索引擎发生了重大变革。Google 将其搜索迁移到新的软件平台，他们称之为 “Caffeine”。Caffeine 是 Google 出自自身的设计，Caffeine 使 Google 能够更迅速的添加新的链接（包括新闻报道以及博客文章等）到自身大规模的网站索引系统中，相比于以往的系统，新系统可提供 “50% 新生” 的搜索结果。

在本质上 Caffeine 丢弃 MapReduce 转而将索引放置在由 Google 开发的分布式数据库 BigTable 上。作为 Google 继 GFS 和 MapReduce 两项创新后的又一项创新，其在设计用来针对海量数据处理情形下的管理结构型数据方面具有巨大的优势。这种海量数据可以定义为在云计算平台中数千台普通服务器上 PB 级的数据。（1PB = 1024T）

另一篇介绍了 Pregel，`Pregel 主要绘制大量网上信息之间关系的“图形数据库”`。而最吸引人的一篇论文要属被称之为 Dremel 的工具。

专注于大型数据中心规模软件平台的加利福尼亚伯克利分校计算机科学教授 Armando Fox 表示 “如果你事先告诉我 Dremel 可以做什么，那么我不会相信你可以把它开发出来”。

`Dremel 是一种分析信息的方式，Dremel 可跨越数千台服务器运行，允许“查询”大量的数据`，如 Web 文档集合或数字图书馆，甚至是数以百万计的垃圾信息的数据描述。这类似于使用结构化查询语言分析传统关系数据库，这种方式在过去几十年被广泛使用在世界各地。

Google 基础设施负责人 Urs Hölzle 表示 “使用 Dremel 就好比你拥有类似 SQL 的语言，并可以无需任何编程的情况下只需将请求输入命令行中就可以很容易的制定即时查询和重复查询”。

区别在于 Dremel 可以在极快的速度处理网络规模的海量数据。据 Google 提交的文件显示你可以在几秒的时间处理 PB 级的数据查询。

目前 Hadoop 已经提供了在庞大数据集上运行类似 SQL 的查询工具（如 Hadoop 生态圈中的项目 Pig 和 Hive）。但其会有一些延迟，例如当部署任务时，可能需要几分钟的时间或者几小时的时间来执行任务，虽然可以得到查询结果，但相比于 Pig 和 Hive，Dremel 几乎是瞬时的。

Holzle 表示 Dremel 可移执行多种查询，而同样的任务如果使用 MapReduce 来执行通差需要一个工作序列，但执行时间确是前者的一小部分。`Dremel 可在大约 3 秒钟时间里处理 1PB 的数据查询请求`。

Armando Fox 表示 Dremel 是史无前例的，Hadoop 作为大数据运动的核心一直致力构建分析海量数据工具的生态圈。但就目前的大数据工具往往存在一个缺陷，与传统的数据分析或商业智能工具相比，Hadoop 在数据分析的速度和精度上还无法相比。但目前 Dremel 做到了鱼和熊掌兼得。

Dremel 做到了 “不可能完成的任务”，Dremel 设法将海量的数据分析于对数据的深入挖掘进行有机的结合。Dremel 所处理的数据规模的速度实在令人印象深刻，你可以轻松的探索数据。在 Dremel 出现之前还没有类似的系统可以做的像 Dremel 这样出色。

据 Google 提交的文件来看，Google 从 2006 年就在内部使用这个平台，有“数千名”的 Google 员工使用 Dremel 来分析一切，从 Google 各种服务的软件崩溃报告到 Google 数据中心内的磁盘行为。这种工具有时会在数十台服务器上使用，有时则会在数以千计的服务器上使用。

Mike Olson 表示尽管 Hadoop 取得的成功不容置疑，但构建 Hadoop 生态圈的公司和企业显然慢了，而同样的情况也出现在 Dremel 上，Google 在 2010 年公布了 Dremel 的相关文档，但这个平台还没有被第三方企业充分利用起来，目前以色列的工程团队正在建设被称为 OpenDremel 的克隆平台。David Gruzman 表示 OpenDremel 目前仅仅还在开始阶段，还需要很长时间进行完善。

换句话说即使你不是 Google 的工程师你同样可以使用 Dremel。Google 现在提供的 BigQuery 的服务就是基于 Dremel。用户可通过在线 API 来使用这个平台。用户可以把数据上传到 Google，并在 Google 基础设施中运行用户的查询服务。而这只是 Google 越来越多云服务的一部分。

早期用户通过 Google App Engine 构建、运行、并将应用托管在 Google 基础设施平台之上。而现今 Google 提供了包括 BigQuery 和 Google Compute Engine 等服务和基础设施，这些服务和基础设施可使用户瞬时接入虚拟服务器。

全球很多技术都落后于 Google，而 Google 自身的技术也正在影响全球。
