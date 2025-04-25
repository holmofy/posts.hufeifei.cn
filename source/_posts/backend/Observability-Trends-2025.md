---
title: 2025 年可观测 10 大趋势预测
date: 2025-04-02
categories: 后端
mathjax: true
post_src: https://zhuanlan.zhihu.com/p/23702288056
tags:
- AIOps
- Observability
- OpenTelemetry
- eBPF
---

## AIOps

AIOps 本身就是可观测的一大重点方向，随着 LLM 的流行，AIOps 再次被吹到风口浪尖，几乎每篇预测 AIOps 都是一大主题。这里我们不纠结于专有名词，统一称为 AIOps 。涵盖的能力很广：

1. AIOps 平台：AIOps 能力快速演进，最终会形成平台化，平台将能管理整个 AIOps 的生命周期，包括集成复杂的异常发现、根因分析和自动化功能，实现统一的 AIOps 能力整合。
2. AI 驱动的预测：AI 的故障发现和事后分析将转向 AI 驱动的预测，以应对数据量和复杂性带来的挑战。AI 和机器学习算法将用于在问题影响业务运营之前进行预测，从而提高系统性能并增强干预能力。
3. AIOps 自动化：AIOps 将显著提升 ITOps 的自动化水平，自动检测和发现潜在问题，减少根因分析所需的手动工作量。
4. 自然语言交互：基于 LLM 的自然语言交互功能将使 IT 人员能够更方便地查询其可观测数据，例如 Chat2PromQL、Chat2SQL 等。
5. 云计算下 AIOps 必要性：随着企业继续向云迁移，以及容器化、各类云原生产品的应用，企业更需要 AIOps 能力来快速获得云环境的可观测能力，自动化地监控、分析和优化云资源的使用，从而确保系统的高效运行。
6. DevOps 和 AIOps 的融合：DevOps 和 AIOps之间的界限将开始模糊，甚至形成统一的运营团队。这些团队将整合 AI 专业知识与传统的软件开发和 IT 运营，管理软件生命周期和 AI 模型生命周期，并持续改进。

## OpenTelemetry

OpenTelemetry 是可观测领域与 AIOps 不分上下的火热话题，在 CNCF 以及各大云厂商、可观测独立厂商的推动下，OpenTelemetry 已经称为可观测领域的事实标准，除 Trace、Metric、Log 外，OpenTelemetry 在 2024 还推出了 Profiling 标准，意图标准化可观测领域的所有数据格式并形成统一关联。由于 OpenTelemetry 协议、OpenTelemetry Collector 的厂商无关性，2025 将逐步巩固在遥测数据采集中的基石地位。OpenTelemetry 由于只定义数据格式和提供采集能力，后端服务由厂商实现，在 2025 年将会有更多厂商开发的工具

## 统一观测平台

2025年可观测领域的一个重要趋势是向统一平台的转变。这些平台将 Log、Trace、Metrc 、Event、Profile 整合到一个集中的视图中，提供以下优势：

* 消除监控工具之间的数据孤岛，强化数据之间的关联。
* 在混合云和多云环境中实现无缝的可视化、问题排查能力。
* 通过从单一界面获取整体洞察结果，简化根因分析代价。
随着可观测性趋势的发展，Datadog、Splunk 和 New Relic 等供应商正引领着向更高集成度和效率的转变。

## 观测右移
用于边缘计算环境的消费和工业设备的数量预计将迅速增加，这些设备继续提供更强大的计算和连接能力。它们的增加也意味着观测和监控必须扩展到边缘设备。对于尚未提供此功能的观测公司来说，在2025年解决这一需求将至关重要，以满足那些正在将其技术栈扩展到边缘环境的客户。

此外公司将更关注涉及用户真实体验的前端监控，这些监控手段需要具备扩展到各类边缘、端设备的能力。观测目标从整体变为细节，企业将更关注对每个客户的监控，而不是整体的分位数情况。对于观测工具的核心能力要求：

* 轻量级的数据采集能力，具备部署到资源受限的 IoT 场景中，并具备一定的端处理能力。
* 高效、低延迟的全球化网络支持，具备网络加速能力。
* 支持大规模数据低成本存储和计算的数据平台。
* 全球化实时的数据汇聚能力，并支持在不移动数据的情况下完成统一视图。

## 观测左移

平台工程师、运维工程师、DevOps和所有利益相关者正在意识到，在开发周期中引入观测对开发者非常有用。对于像 Kubernetes 这样高度分布和互联的服务和应用程序而言，这一点尤为重要。除了测试，在非常详细的层次上观察堆栈及其在整个开发周期中与应用程序其他部分的交互是观测的另一个关键方面。预计这一方面将在 2025 年得到更广泛的部署。

随着近两年 Profiling 相关技术的成熟，开发者可以快速地在初期阶段引入 Profile、Trace 等技术观测软件的细节行为。这一增强大大改善了开发者的体验，因为分析提供了对代码影响的无与伦比的视图，从而促进更快、更具成本效益的优化。

Gartner 将这一左移趋势描述为观测驱动开发（ODD）工程实践的一部分，通过设计可观测的系统，提供对系统状态和行为的细粒度可见性和上下文，从而使得在开发周期的早期阶段以及生产环境中更容易检测、诊断和解决意外异常。

## 平台工程的下一个前沿：eBPF

平台团队正在经历显著的增长，Grafana 对于可观测的调查中有近 25% 的人在这个角色中工作。随着平台团队重要性的增加，他们的职责将扩展到包括新兴工具和技术——例如eBPF。eBPF起初是一种时髦的技术，如今将成为现代平台工程的支柱，从根本上重塑组织处理可观察性和安全性的方式。当前 eBPF正处于“重大转型的边缘”。

eBPF 带来一个显著的变化将是把 Profiling 甚至是整体观测责任从应用程序团队转移到平台团队。尤其是 OpenTelemetry Profiling 协议的完善以及与 eBPF 的集成，能够以标准化平台的方式收集和处理这些可观测数据。

## 下一代可观测主力军：Log

随着企业数字化在 2024 年达到历史新高，我们预期开发、安全和运营团队需要更加紧密地合作，以帮助解决业务、技术和安全运营面临的最难问题。这一演变导致了 AI 驱动的观测平台的迅速崛起，并使人们更加广泛地理解日志作为系统记录的重要性。在 2025 年，组织的结构化和非结构化日志数据中蕴含的洞察力将通过传统 AI/ML 和生成式 AI 技术得以释放。这将提供无与伦比的上下文和洞察力，并最终兑现对应用程序和数字服务观测的承诺。

此外，Log 分析管理工具也将产生巨大的技术提升，包括大规模分析技术、低成本冷热存储分离、数据湖能力等。

## Cost-Effective Observability

随着系统复杂性的增加，观测成本也在逐渐上涨，关注成本逐渐成为可观测发展趋势中的一个关键方面。到 2025 年，企业将采用以下策略节省成本：

* 更智能的数据采样和保留策略以减少存储开销。
* 采用按使用量付费模式的 Serverless 观测工具。
* 选择在功能性和成本效益之间取得平衡的解决方案。

## 超越传统运维的观测性

2025年的观测性趋势将超越传统的基础设施、中间、应用程序的监控和观测，涵盖以下方面：

* 业务流程观测性：提供对客户产品使用流程和公司运营效率的洞察。
* DevSecOps观测性：确保高效、安全的部署。
* 可持续性观测性：通过遥测跟踪和优化碳中和足迹。

这些发展将重新定义观测性的潜力和实现范围。

## 从事后回溯到事前预防

随着客户对应用体验要求的逐渐提升，企业越来越希望观测系统能提前预测潜在的服务中断、容量问题和性能下降。这种主动的方法帮助企业在问题影响最终用户之前，减轻风险并有效管理资源，增强服务可靠性并减少计划外停机时间。

与早期因缺乏上下文理解而难以实现的传统 AIOps 方法不同，新一代 AI 驱动的观测性整合了跨系统的观测数据，能够快速识别根因、预测级联故障，使事前预防成为可能。

## 参考

* [Observability in 2025: OpenTelemetry and AI to Fill In Gaps](https://thenewstack.io/observability-in-2025-opentelemetry-and-ai-to-fill-in-gaps/)
* [The Future of Observability: Trends to Watch in 2025](https://www.skedler.com/blog/the-future-of-observability-trends-to-watch-in-2025/)
* [2025 observability predictions and trends from Grafana Labs](https://grafana.com/blog/2024/12/16/2025-observability-predictions-and-trends-from-grafana-labs/)
* [Five observability predictions for 2025](https://www.dynatrace.com/news/blog/observability-predictions-for-2025/)
* [Best Data Observability Tools in 2025: A Buyer’s Guide](https://www.intellectyx.com/best-data-observability-tools-2025-a-buyers-guide/)
* [2025 Observability Trends](https://www.constellationr.com/research/2025-observability-trends)
* [2025 Observability Predictions - Part 1](https://www.apmdigest.com/2025-observability-predictions-part-1)
* [Observability Driven Development (ODD)-Enhancing System Reliability](https://medium.com/@bijit211987/observability-driven-development-2bc2cdde8661)
* [Observability-Driven Development Explained: 8 Steps for ODD Success](https://www.splunk.com/en_us/blog/learn/odd-observability-driven-development.html)
* [Drive optimization and sustainability with continuous profiling](https://www.elastic.co/observability/universal-profiling)
* [Continuous Profiler](https://docs.datadoghq.com/profiler/)
* [OpenTelemetry announces support for profiling](https://opentelemetry.io/blog/2024/profiling/)
* [opentelemetry-ebpf-profiler](https://github.com/open-telemetry/opentelemetry-ebpf-profiler)
* [Grafana pyroscope](https://grafana.com/oss/pyroscope/)
* [Datadog acquires Quickwit](https://www.datadoghq.com/blog/datadog-acquires-quickwit/)
* [Aliyun 可观测链路 OpenTelemetry 版](https://help.aliyun.com/zh/opentelemetry/)
* [quickwit](https://quickwit.io/)
* [Elastic Contributes its Continuous Profiling Agent to OpenTelemetry](https://opentelemetry.io/blog/2024/elastic-contributes-continuous-profiling-agent/)
* [Chat2Query：创新的 AI 驱动的 SQL 生成器，可更快地获得见解](https://www.pingcap.com/chat2query-an-innovative-ai-powered-sql-generator-for-faster-insights/)
* [什么是阿里云应用监控 eBPF 版](https://help.aliyun.com/zh/arms/application-monitoring-ebpf/product-overview/what-is-alibaba-cloud-application-monitoring-ebpf-version)
* [Announcing Prometheus 3.0](https://prometheus.io/blog/2024/11/14/prometheus-3-0/)
* [Awesome LLM AIOps](https://github.com/Jun-jie-Huang/awesome-LLM-AIOps)
* [Navigating Observability: Logs, Metrics, and Traces Explained](https://openobserve.ai/articles/logs-metrics-traces-observability/)
* [Elastic Rerank](https://www.elastic.co/docs/explore-analyze/machine-learning/nlp/ml-nlp-rerank)
* [The future is now, introducing Dynamic Observability from AI innovations built on logs](https://www.sumologic.com/blog/dynamic-observability-ai-innovations-logs/)
