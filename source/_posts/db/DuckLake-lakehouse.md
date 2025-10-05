---
title: DuckLake 承袭 DuckDB 简洁理念，重构 Lakehouse 本质
date: 2025-10-02
categories: 数据库
mathjax: true
post_src: https://zhuanlan.zhihu.com/p/1911482904741127529
tags: 
- DuckDB
- Lakehouse
---

在大数据领域，“[湖仓一体](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=%E6%B9%96%E4%BB%93%E4%B8%80%E4%BD%93&zhida_source=entity)”这个概念早已不再陌生。它试图将[数据湖](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E6%B9%96&zhida_source=entity)（Data Lake）的开放性与[数据仓库](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=%E6%95%B0%E6%8D%AE%E4%BB%93%E5%BA%93&zhida_source=entity)（Data Warehouse）的事务一致性结合起来。但在现实中，很多 Lakehouse 系统为了兼顾开放性与一致性，不得不引入复杂的文件元数据系统、目录服务，甚至绕了一个大圈子又回到“数据库”。

![](https://pic4.zhimg.com/v2-39118c621f66495996d6564084f43de5_1440w.jpg)

[DuckLake](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=DuckLake&zhida_source=entity) 提出了一种看似简单却革命性的想法：**既然最终都要引入数据库来管理元数据，那不如从一开始就用数据库来管理所有元数据！**

> DuckLake 通过将所有元数据存储在标准 SQL 数据库中，而不是复杂的基于文件的系统，从而简化了数据湖仓的实现。同时，它仍然使用如 [Parquet](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=Parquet&zhida_source=entity) 这样的开放格式来存储数据。这种方式使系统更加可靠、更快速，也更易于管理。

## 背景介绍

像 [BigQuery](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=BigQuery&zhida_source=entity) 和 [Snowflake](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=Snowflake&zhida_source=entity) 这样的创新型数据系统表明，在存储已成为一种虚拟化商品的时代，将存储与计算解耦是一个非常棒的想法。这样，存储和计算可以独立扩展，我们也不需要为了存储那些永远不会被读取的表而购买昂贵的数据库设备。

与此同时，市场力量推动人们要求数据系统采用诸如 Parquet 这样的开放格式，以避免数据被某个厂商“绑架”的常见问题。在这个新世界中，许多数据系统快乐地围绕着建立在 Parquet 和 S3 之上的“数据湖”运作，一切似乎都很美好。谁还需要那些老派的数据库呢？

但很快人们就发现——令人震惊的是——用户其实还想**修改**他们的数据集。简单的追加操作还算顺利，只需将新文件丢进文件夹即可，但除此之外的操作就需要依赖复杂且容易出错的自定义脚本，既没有正确性保障，更别提**事务保障**了。

## Iceberg and Delta

为了解决“在数据湖中修改数据”这一基本需求，各种新的开放标准应运而生，其中最突出的是 **Apache Iceberg** 和 **Linux Foundation [Delta Lake](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=Delta+Lake&zhida_source=entity)**。这两种格式的设计初衷，是在不放弃“在对象存储中使用开放格式”这一核心理念的前提下，引入对数据表进行修改的合理方式。例如，Iceberg 通过一系列复杂的 JSON 和 Avro 文件来定义 schema、快照以及某一时刻哪些 Parquet 文件属于该表。这种体系被称为 **“Lakehouse（湖仓一体）”** —— 实质上是在数据湖的基础上叠加了数据库特性，从而支持了许多令人兴奋的新型数据管理用例，比如跨引擎数据共享。

![](https://pic3.zhimg.com/v2-0df96da59a030cb86eb3cec1d6878b3c_1440w.jpg)

但这两种格式都遇到了同一个难题：**在一致性不稳定的对象存储中定位表的最新版本很困难**。原子性更新（ACID 中的 “A”）变得棘手——很难确保一个“指针”的切换能被所有用户即时看到表的最新状态。此外，Iceberg 和 Delta Lake 都只关注于**单个表的管理**，而现实是用户还希望能同时管理**多个表**。

## 目录（Catalogs）

为了解决上述问题，人们又引入了一层技术方案：**在这些文件之上增加一个目录服务（catalog service）**。这个目录服务会连接一个数据库，用来管理所有表对应的文件夹名称。

![](https://pic3.zhimg.com/v2-f32159c479b29edddff8c0ae9fcd95b0_1440w.jpg)

同时，它还维护着一张“史上最悲伤的表”：这张表每个表只占一行，记录着该表的**当前版本号**。借助数据库的事务保障（transactional guarantees），我们就可以安全地更新这个版本号。这样一来，所有人都能一致地看到最新版本，大家也就都开心了。

## 绕了一圈，还是得用数据库？

但问题在于：**Iceberg 和 Delta Lake 的初衷就是不依赖数据库**。它们的设计者煞费苦心地将读取和更新表所需的全部信息编码进了对象存储（blob store）上的文件中，力求摆脱数据库的束缚。

为此，它们做出了很多妥协。例如，Iceberg 中的每个根文件（root file）都包含了所有现存的快照（snapshots），包括完整的 schema 信息等等。每次数据变更，都会生成一个新文件，记录完整的历史。为了避免频繁读写过多小文件（在对象存储中效率很低），其他元数据还必须批量组织，比如采用双层的 manifest 文件结构。

至于对数据进行**小规模修改**，这仍然是一个悬而未解的难题，通常需要复杂的清理过程（cleanup procedures），而这些过程目前在开源实现中既不成熟也缺乏支持。

然而，正如上文所提到的：**Iceberg 和 Delta Lake 最终还是不得不妥协，引入数据库作为 catalog 的一部分以确保一致性**。但它们并没有因此重新审视自己原有的设计限制与技术架构来适应这一根本性的改变。

## DuckLake 是什么？

在 [DuckDB](https://zhida.zhihu.com/search?content_id=258373320&content_type=Article&match_order=1&q=DuckDB&zhida_source=entity)，我们其实是喜欢数据库的。它们是用来安全、高效地管理相当大规模数据集的绝佳工具。既然数据库最终已经被引入了 Lakehouse 体系，那么用它来管理其余的表元数据也就变得非常合理！我们仍然可以利用对象存储的“无限”容量和“无限”扩展性，将实际的表数据以 Parquet 等开放格式存储在其中；但用于支持数据变更所需的元数据，放在数据库中管理会更加高效、可靠！

巧的是，Google BigQuery（用的是 Spanner）和 Snowflake（用的是 FoundationDB）也是这么做的——只是它们在底层并没有采用开放格式罢了。

![](https://pic1.zhimg.com/v2-24c719052568992a3b3a04a3f3f721fe_1440w.jpg)

为了解决当前 Lakehouse 架构中存在的根本性问题，我们创建了一种新的开放表格式，名为 **DuckLake**。DuckLake 重新构想了 “Lakehouse” 格式应有的样子，基于两个简单的事实：

*   **将数据文件以开放格式存储在对象存储中**，在可扩展性方面非常优秀，同时也能避免厂商锁定。  
*   **元数据管理是一项复杂且高度关联的数据管理任务，最适合由数据库管理系统来完成**。  

DuckLake 的基本设计理念是：**将所有元数据结构（包括目录信息和表级数据）都移入一个 SQL 数据库中进行管理**。该格式被定义为一组关系型表，以及在这些表上的纯 SQL 事务，用来描述各种数据操作，比如：创建和修改 schema、增删改数据等。

DuckLake 格式能够管理任意数量的表，并支持跨表事务。它还支持一些“高级”数据库概念，比如视图、嵌套类型、事务性 schema 更改等。这一设计的一个重要优势是，可以借助关系型数据库提供的**参照完整性**（即 ACID 中的 “C”），例如确保不会出现重复的快照 ID。

> DuckLake 的架构非常直观：**就是一些 Parquet 文件 + 一个 SQL 数据库**。

![](https://pic3.zhimg.com/v2-370f703e3fe078bceacd142a87e7d0b4_1440w.jpg)

具体使用哪种 SQL 数据库由用户自行决定，唯一的要求是系统必须支持 ACID 操作和主键，并具备标准的 SQL 支持。DuckLake 内部使用的表结构故意设计得非常简洁，以最大化对不同 SQL 数据库的兼容性。以下通过一个示例展示核心表结构。

让我们来看当在一个全新空表上执行以下查询时，DuckLake 中依次发生的查询操作：

![](https://pic4.zhimg.com/v2-3dd97d0d009d8223e636e0db34c0b4b7_1440w.jpg)

我们看到一个完整的、连贯的 SQL 事务，它：

*   插入新的 Parquet 文件路径  
*   更新全局表统计信息（现在行数增加了）  
*   更新全局列统计信息（最小值和最大值发生了变化）  
*   更新文件列统计信息（同时记录最小值/最大值等内容）  
*   创建一个新的模式快照（编号为 #2）  
*   记录在该快照中发生的变更  

注意，实际写入 Parquet 文件并不在这段操作序列中，它是在此之前完成的。但无论添加多少数据，该操作序列的成本始终保持相同（且较低）。

## 为什么我们需要 DuckLake？

1\. 数据湖的困境：虽然 Parquet + S3 的组合非常受欢迎，但一旦你需要做更复杂的操作，比如：

*   修改表结构   
*   更新或删除数据  
*   跨表事务  

你就会发现，仅靠文件操作根本无法保证一致性和事务性。这时候就需要像 Iceberg 或 Delta Lake 这样的 Lakehouse 格式。

2\. Iceberg/Delta 的折中与复杂性：为了支持上述需求，Iceberg 和 Delta 引入了大量 JSON/Avro 文件用于维护快照、schema、manifest 等。最终它们还不得不引入“Catalog”服务，而这个服务背后又是一个数据库。也就是说：

> 为了避免用数据库，最终还是用了数据库，只不过是个更复杂的版本。

## DuckLake 的核心设计

DuckLake 承认两件简单的事情：

*   **用 Parquet 存储数据很棒，可以扩展且避免供应商锁定**  
*   **但管理元数据这事，还是数据库更擅长**  
    

因此 DuckLake 的设计是：

*   **所有元数据用 SQL 表结构来表达**（例如：快照、文件统计、列统计、表结构等）  
*   **所有数据操作都是纯 SQL 事务**，如插入、更新、结构变更、快照提交等  
*   **支持跨表事务、视图、嵌套类型、schema 演进等高级能力**  
    

这一切都通过一组干净、规范的 SQL 表来完成。没有 JSON、没有额外 API，只有 SQL。

## DuckLake 的三大原则

## 1\. 简单（Simplicity）

DuckLake 延续了 DuckDB 的设计理念：**简单、渐进**。

*   **轻量易用**：只需在笔记本电脑上安装 DuckDB 及其 DuckLake 扩展即可运行，非常适合测试、开发和原型验证。在这种情况下，Catalog 存储就是一个本地的 DuckDB 文件。  
    
*   **灵活的存储支持**：DuckLake 的数据文件是不可变的，无需就地修改或重用文件名，因此可以兼容各种存储系统，包括本地磁盘、NAS、S3、Azure Blob Store、GCS 等。只需在创建元数据表时指定数据文件的存储前缀（如 `s3://mybucket/mylake/`）即可。  
    
*   **元数据托管数据库可自由选择**：只要是支持 ACID 和主键约束的 SQL 数据库（如 PostgreSQL 或 DuckDB 本身）都可以作为元数据存储。大多数组织已具备这类数据库的运维经验，因此无需额外的软件部署，降低了使用门槛。同时，SQL 数据库已被高度商品化，有大量托管服务可选，迁移也非常方便，不需要移动任何数据文件，因其模式简单且标准化。  
    
*   **完全基于 SQL**：DuckLake 不需要 Avro 或 JSON 文件，也不需要额外的 Catalog 服务或自定义 API。所有操作只需 SQL，即可完成。我们都懂 SQL，这让一切变得更简单。  
    

## 2\. 可扩展（Scalability）

DuckLake 实际上将数据架构中的职责进行了**更清晰的三分**：**存储、计算、元数据管理**，实现了更强的模块化和可扩展性。

*   **存储层:**数据仍然保存在专用的文件存储系统中（如 S3 等对象存储）。DuckLake 依赖开放格式（如 Parquet），可在存储层实现无限扩展。  
*   **计算层:**任意数量的计算节点可以并发查询和更新元数据，同时独立地从存储层读取或写入数据。这意味着 DuckLake 在计算层也具备横向无限扩展能力。  
*   **元数据管理层（Catalog 数据库）:**Catalog 数据库仅处理由计算节点发起的元数据事务，其负载相比真实数据操作小几个数量级。因此，DuckLake 对 Catalog 的性能要求较低，而且支持替换，比如从 PostgreSQL 迁移到其他数据库。这种灵活性来源于 DuckLake 所用的数据结构仅为普通的 SQL 表，语义简单、易移植。  
    

> 实际上，这正是 BigQuery 和 Snowflake 管理超大规模数据集的核心设计思想。

## 3\. 高性能（Speed）

DuckLake 延续 DuckDB 的理念，专注**高性能和低复杂度**，相较于 Iceberg 和 Delta Lake 有明显优势：

*   元数据集中管理，查询更快：所有元数据存在 SQL 数据库中，仅需**一条查询**即可完成分区裁剪、统计过滤，获取需要读的文件列表。无需多次 HTTP 请求，避免 S3 限速、失败和一致性问题。  
*   小变更更高效：避免为小改动生成快照/manifest 文件；支持**将小变更内联写入元数据库表**，实现亚毫秒写入；大幅减少小文件，简化清理与压缩。  
*   高并发写入，无压力：每次表变更仅需执行**一条 SQL 事务**；减少冲突窗口，SQL 数据库擅长处理并发事务；即使使用 PostgreSQL，也能支持每秒上千次提交，支持**上千节点并发写入**。  
*   快照轻量，支持百万级别：每个快照只是几行元数据；可共享数据文件的一部分；无需主动清理，也不会引发文件膨胀。  
    

DuckLake 把元数据放入 SQL 数据库中，简化架构、加速读写、支持高并发，是一个**更快、更轻、更强**的 Lakehouse 新方案。

## DuckLake 特征

DuckLake 拥有你喜爱的所有 Lakehouse 特性：

*   **任意 SQL 查询**：支持与 DuckDB 相同的丰富 SQL 功能。  
*   **数据更改支持**：高效支持追加、更新和删除操作。  
*   **多 Schema、多表管理**：可在同一元数据结构中管理任意数量的 schema 和其中的多张表。  
*   **跨表事务**：支持完整的 ACID 跨表事务，涵盖所有 schema、表及其内容。  
*   **复杂类型支持**：支持如列表、嵌套等各种复杂数据类型。  
*   **完整模式演进**：表结构可任意变更，包括新增/删除列、修改列类型等。  
*   **模式级时间旅行与回滚**：支持快照隔离和时间旅行，可查询任意时间点的表状态。  
*   **增量扫描**：支持查询两个快照之间发生的数据变更。  
*   **SQL 视图支持**：可定义惰性求值的 SQL 视图。  
*   **隐藏分区与扫描剪枝**：自动感知分区和表/文件级统计信息，提前剪枝以提升效率。  
*   **事务性 DDL**：创建、修改、删除 schema、表、视图等操作均支持事务处理。  
*   **避免频繁压缩**：相比其他格式，DuckLake 需要的压缩操作更少，且支持高效快照压缩。  
*   **数据内联**：小变更可直接写入 catalog 数据库，无需频繁写小文件。  
*   **数据加密**：可选择加密所有数据文件，实现零信任数据托管，密钥由 catalog 数据库管理。  
*   **兼容性好**：DuckLake 写入的存储文件（包括删除文件）**完全兼容 Apache Iceberg**，可实现**仅元数据的迁移**。  

## DuckLake DuckDB 扩展：让 Lakehouse 直接运行起来

定义一个 Lakehouse 格式很容易，真正让它跑起来却不简单。因此，我们同步发布了 DuckLake 的计算节点实现 —— **ducklake DuckDB 扩展**。

这个扩展完全实现了上文描述的 DuckLake 格式，具备全部特性。它是 MIT 许可下的免费开源软件，知识产权归属于非营利的 DuckDB 基金会。

ducklake 扩展将 DuckDB 从原本的单机分析工具，拓展为支持**中心化数据仓库**场景的 Lakehouse 引擎。企业只需部署一个中心 catalog 数据库和文件存储（如 RDS + S3 或本地部署），然后在任意设备上运行带有 ducklake 扩展的 DuckDB —— 包括员工电脑、手机、应用服务器，甚至无服务器代码。

扩展支持使用本地 DuckDB 文件作为元数据存储，也支持连接任意 DuckDB 支持的数据库，包括 PostgreSQL、SQLite、MySQL 和 MotherDuck 等。同时可使用本地磁盘、S3、Azure Blob、GCS 等存储系统。

DuckLake 扩展**不会取代** DuckDB 对 Iceberg 和 Delta 的原生支持，反而可以作为它们的本地缓存或加速层。

从 DuckDB v1.3.0（代号 “Ossivalis”）开始，DuckLake 扩展正式可用。

安装扩展：

![](https://pic4.zhimg.com/v2-f173a48ebb23a80ad82bb719c8fd47f3_1440w.jpg)

可以通过`ATTACH`DuckDB 中的命令来初始化一个 DuckLake。例如：

![](https://pica.zhimg.com/v2-1ea85f7d98f92885a379912b1ff8fb12_1440w.jpg)


接下来我们创建一个表并插入一些数据：

![](https://pic1.zhimg.com/v2-2d9dc90c8151ee1d0f3198239f646d46_1440w.jpg)


查询数据：

![](https://pic1.zhimg.com/v2-275a13e6b2f550739ccf4fdf6346bfdc_1440w.jpg)  

一个包含两行数据的 Parquet 文件已经创建完毕

![](https://pic1.zhimg.com/v2-f24146d8c14501cb6923c7eaac976bba_1440w.jpg)


删除数据：

![](https://pic1.zhimg.com/v2-3b79a3852c95f0c5b3d912c9f528d5d0_1440w.jpg)


再次检查该文件夹，我们会看到一个新文件出现，`-delete`出现了名称中带有的第二个文件，这也是一个包含已删除行的标识符的 Parquet 文件。


![](https://pic1.zhimg.com/v2-d10f98eff685311f5ffa6b79e65181f2_1440w.jpg)

当然，DuckLake支持_时间旅行，_我们可以使用该`ducklake_snapshots()`功能查询可用的快照。

![](https://pic4.zhimg.com/v2-3850acb9413f666f1d395299bab02d09_1440w.jpg)

假设我们想要读取第 43 行被删除之前的表，我们可以使用DuckDB 中的新`AT`语法：

![](https://pic3.zhimg.com/v2-d51d6ebd70c190e9cbaf8867998e4092_1440w.jpg)

版本 2 仍然有此行，所以就是这样。这也适用于快照时间戳而不是版本号：只需使用`TIMESTAMP`而不是`VERSION`。

我们还可以使用函数查看版本之间发生了什么变化`ducklake_table_changes()`，例如:
  

![](https://pic4.zhimg.com/v2-6a24e780b91308ac76a52d265c6746f7_1440w.jpg)

DuckLake 中的更改当然是事务性的，之前我们运行在“自动提交”模式下，每个命令都是一个独立的事务。但我们可以使用`BEGIN TRANSACTION`and`COMMIT`或 来更改这一点`ROLLBACK`。

![](https://pic1.zhimg.com/v2-6e82988011b9d76fbe9c99ecbec89c46_1440w.jpg)

![](https://pic1.zhimg.com/v2-77a68465d61b1f6406cad3bddb7b17aa_1440w.jpg)

## 总结

DuckLake 并不是另一个“炫酷但难用”的新格式，而是一次回归本质的尝试：**我们为什么不直接用 SQL 数据库管理元数据？**

它保留了数据湖的开放性，又拥抱了数据库的强大管理能力，将复杂性降到最低，同时提升了可靠性和可扩展性。

正如 DuckDB 所追求的“轻盈、简洁、高效”，DuckLake 将继续沿着这条道路，为我们带来下一代的 Lakehouse 架构。
