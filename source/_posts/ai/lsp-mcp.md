---
title: 从LSP看MCP：协议标准化如何改变开发与AI生态
date: 2025-06-02
categories: AI
mathjax: true
post_src: https://juejin.cn/post/7488277639205404709
tags: 
- AI
- MCP
- LSP
---


在技术领域，标准化协议往往是推动行业变革的隐形力量。语言服务器协议（LSP，Language Server Protocol）和模型上下文协议（MCP，Model Context Protocol）就是两个典型的例子。LSP在过去十年中彻底改变了开发工具的生态，而MCP则有望在AI领域掀起类似的革命。本文将通过对比LSP和MCP，探讨协议标准化如何为开发者带来便利，并展望MCP的未来潜力。

## 语言服务器协议（LSP）：开发工具的革命

### LSP出现前的混乱

在2016年[LSP](https://github.com/Microsoft/language-server-protocol)发布之前，开发工具生态可以用“各自为政”来形容。当时，集成开发环境（IDE）或代码编辑器（如VSCode、Sublime Text、VIM）需要为每种编程语言单独实现特定的工具支持。这意味着：

* 语言开发者需要为不同的编辑器分别开发支持。例如，TypeScript团队可能需要实现“TypeScript-Sublime-Server”和“TypeScript-VSCode-Server”，这导致资源浪费和功能不一致。
* 编辑器开发者需要为每种语言单独开发支持。如果一个编辑器不支持某种语言，用户体验就会大打折扣。例如，VSCode可能对JavaScript支持很好，但在Sublime Text中表现不佳；而对于更小众的编辑器（如VIM），用户可能只能依赖简陋的开源插件，甚至完全没有工具支持。

这种碎片化的生态让开发者不得不在语言支持和编辑器功能之间做出妥协：要么选择支持自己语言的IDE，要么放弃一些高级功能（如语法高亮、自动完成、错误检查）。

### LSP的诞生与变革

2016年，微软推出了语言服务器协议（LSP），为开发工具生态带来了革命性的变化。LSP是一个基于JSON-RPC的开放协议，定义了编辑器/IDE（客户端）与语言工具（服务器）之间的通信标准。它支持的特性包括代码自动完成、语法高亮、错误检查、跳转定义等。

### LSP的核心创新在于解耦：

* 语言开发者（LSP服务器端）不再需要为每个编辑器单独实现支持。只需要开发一个通用的“LSP服务器”（如“TypeScript-LSP-Server”），就可以与所有支持LSP的编辑器协作。
* 编辑器开发者（LSP客户端端）无需为每种语言单独实现支持。只要编辑器实现了LSP客户端，它就能连接到任何LSP服务器，获得一致的语言支持。

这种解耦带来的好处是显而易见的：

* 语言生态：即使是小众语言也能通过实现一个LSP服务器，在主流编辑器中获得一流的开发体验。例如，Rust语言通过[rust-analyzer](https://github.com/rust-lang/rust-analyzer)（一个LSP服务器）在VSCode、VIM等编辑器中提供了强大的支持。
* 编辑器生态：用户可以根据编辑器的功能（如VSCode的扩展生态、Cursor的AI功能）选择工具，而无需担心语言支持问题。
* 开发者体验：开发者不再需要在语言支持和编辑器功能之间妥协，可以专注于编码本身。

### LSP的现状

![LSP的现状](https://code.visualstudio.com/assets/api/language-extensions/language-server-extension-guide/lsp-languages-editors.png)

截至2025年，LSP已经成为开发工具的默认标准。VSCode、VIM、Emacs等主流编辑器都支持LSP，许多编程语言（如TypeScript、Python、Rust）也提供了官方的[LSP服务器](https://microsoft.github.io/language-server-protocol/implementors/servers/)。然而，LSP的普及并非一帆风顺。一些公司（如JetBrains、Apple）更倾向于使用自己的专有协议，例如JetBrains的IDE和Apple的XCode至今仍未完全拥抱LSP。尽管如此，LSP的开放性和跨平台特性使其逐渐成为连接编辑器与语言工具的桥梁。

## 模型上下文协议（MCP）：AI领域的LSP？

### MCP的背景

2024年底，Anthropic推出了模型上下文协议（MCP），一个旨在解决AI模型与外部系统集成问题的开放协议。MCP的目标与LSP有异曲同工之妙：通过标准化协议，简化AI客户端（如Claude、Cursor、OpenAI）与外部服务（如数据库、第三方API）的交互。

在MCP出现之前，AI领域的集成生态与LSP出现前的开发工具生态非常相似：

* 服务提供者（如Supabase、Stripe）需要为不同的AI客户端分别实现集成。例如，Supabase可能需要分别支持Claude、Cursor、OpenAI等，这增加了开发成本。
* AI客户端需要为每种服务单独开发支持，或者依赖自己的“集成市场”（如OpenAI的GPT市场），这导致用户体验碎片化。
* 小型服务提供者由于资源有限，很难与主流AI平台集成，限制了他们的发展。

这种碎片化让开发者在AI应用开发中面临诸多挑战：要么花费大量时间处理集成问题，要么局限于少数支持的平台。

### MCP的架构与目标

MCP的架构与LSP类似，分为三部分：

* MCP服务器：由服务提供者实现，负责与外部系统（如数据库、API）交互。例如，Supabase通过Postgres MCP服务器提供只读数据库查询支持。
* MCP客户端：由AI工具实现，负责与MCP服务器通信。例如，Cursor可以通过MCP连接到Supabase，执行自然语言查询。
* MCP主机：协调客户端和服务器之间的通信。

### MCP的目标是通过标准化协议实现解耦：

* 服务提供者只需要实现一个MCP服务器，就能与所有支持MCP的AI客户端协作。例如，Supabase只需要一个“Supabase-MCP-Server”，无需为每个AI平台单独开发支持。
* AI客户端无需为每种服务单独实现支持。只要实现了MCP客户端，就能连接到任何MCP服务器，获取外部数据或功能。
* 最终用户（开发者）可以自由选择AI工具，而无需担心服务支持问题。例如，用户可以在Claude或Cursor中使用Supabase的数据库查询功能，无需额外配置。

### MCP与LSP的相似性

MCP的设计灵感很大程度上来源于LSP。David Soria Parra在X帖子中提到：“[LSP was a big inspiration](https://x.com/dsp_/status/1897821339332882617)”（LSP是很大的灵感来源），并感叹LSP的伟大之处被低估。这种相似性不仅体现在架构上，还体现在目标上：

* LSP让编程语言与编辑器解耦，MCP则让外部服务与AI客户端解耦。
* LSP通过标准化提升了小众语言的开发体验，MCP则有望让小型服务提供者更容易与AI平台集成。
* LSP减少了编辑器开发者的负担，MCP则可能终结AI客户端的“集成市场”（如OpenAI的GPT市场），让用户更方便地使用第三方工具。

一个简单的类比可以帮助理解两者的关系：如果将LSP中的“编程语言”替换为“第三方服务”（如Supabase），将“编辑器”替换为“AI客户端”（如Claude），你就得到了MCP。

## MCP的潜力与挑战

![MCP的潜力与挑战](https://i-blog.csdnimg.cn/direct/29100a3d093048a5aa8280c1b5ebf451.png)

### MCP的潜力

MCP有潜力在AI领域复制LSP的成功,MCP 带来了几个核心优势

作者：谦行的总结 阅读[原文链接](https://juejin.cn/post/7482236799268864040)

* **协议标准化驱动生态统一：** MCP通过统一的协议简化了AI与外部工具的连接，开发者无需为每个工具单独编写接口代码，实现一次开发多工具复用

  > 这里有数千开源的 MCP Server 实现可以使用、借鉴，Cursor、Windsurf 等 IDE 均已支持
  >
  > * [glama.ai/mcp/tools](https://glama.ai/mcp/tools)
  > * [smithery.ai/](https://smithery.ai/)
  > * [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)

* **开发效率**：MCP 官方提供了[开发工具包 & 调试工具](https://github.com/modelcontextprotocol)，相对于兼容各种 AI 模型的 Function Call，实现一个通用的 MCP Server 极其简单

```python
from mcp.server.fastmcp import FastMCP

# Create an MCP server
mcp = FastMCP("Demo")

# Add an addition tool
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers"""
    return a + b

# Add a dynamic greeting resource
@mcp.resource("greeting://{name}")
def get_greeting(name: str) -> str:
    """Get a personalized greeting"""
    return f"Hello, {name}!"
```

* **根治上下文爆炸：** MCP 采用模块化上下文管理，将外部数据源抽象为独立模块，模型仅在需要时激活指定模块，同时通过增量索引，MCP 仅同步变更数据，相比 Function Call 的全量注入模式，Token 消耗显著降低
* **动态发现与灵活性**：MCP 支持动态发现可用工具，AI 可自动识别并调用新接入的数据源或功能，无需提前配置

MCP 通过协议层革新重构了 AI 与外部系统的协作范式，其标准化、动态化、安全性的特征，正在解决 Function Call 面临的生态碎片化、上下文冗余、权限粗放等核心痛点。随着 Anthropic 携 Claude 之利 的生态推进，MCP 有望成为下一代智能系统的核心基础设施

## 优势总结

1. 标准化需求：AI领域的集成问题比开发工具领域更为迫切。随着AI应用的爆炸式增长，开发者需要一个统一的协议来简化与外部系统的交互。MCP正好满足了这一需求。
2. 实际应用：MCP已经在一些场景中落地。例如，Supabase通过MCP服务器支持与AI工具（如Cursor）的集成，用户可以使用自然语言命令执行数据库查询。
3. 社区支持：MCP是一个开源项目，由Anthropic推动，并吸引了大量社区参与。LSP的成功很大程度上得益于其开放性，MCP也有望通过社区力量加速普及。

### MCP的挑战

尽管前景光明，MCP也面临一些挑战：

1. 生态系统阻力：与LSP类似，MCP可能会遇到大公司的阻力。例如，OpenAI可能更希望维护自己的GPT市场，而不是采用MCP。类似地，JetBrains和Apple在开发工具领域也更倾向于使用专有协议。
2. 技术复杂性：AI领域的集成需求比开发工具更复杂。LSP主要处理语言特性（如语法高亮），而MCP需要处理更广泛的场景（如数据库查询、API调用）。目前，Supabase的MCP服务器仅支持只读查询，未来可能需要扩展到更复杂的交互。
3. 普及速度：LSP用了近10年时间（2016-2025）才成为开发工具的默认标准。MCP在2024年底推出，普及可能需要时间，尤其是在开发者认知度不足的情况下。


如果对MCP感兴趣，不妨关注它的实际应用案例（如Supabase的MCP服务器），或者参与社区讨论，分享您的经验和见解。协议标准化看似不起眼，但它往往是技术进步的基石——LSP已经证明了这一点，而MCP的征程才刚刚开始。

参考资料:

* [code.visualstudio.com/api/languag…](https://code.visualstudio.com/api/language-extensions/language-server-extension-guide#why-language-server)
* [reddit](https://www.reddit.com/r/mcp/comments/1jofsdz/hypeless_opinion_of_mcp/?%24deep_link=true&correlation_id=fd5ab237-a01e-4c33-a746-1644e49beaa5&post_fullname=t3_1jofsdz&post_index=0&ref=email_digest&ref_campaign=email_digest&ref_source=email&utm_content=post_body&%243p=e_as&_branch_match_id=1426411557553020972&utm_medium=Email%20Amazon%20SES&_branch_referrer=H4sIAAAAAAAAA22P3U7EIBCFn6Z71%2F0p7G402Rij8TXIFKYtCgwZaOp64bM7ddUrEyCH75wZhqnWXO53O0bnfN1Cztvg09tO5Yem0ypf0EDZiCT2o08QzMzhMq1VjXpsuhdZy7Jsf%2BotRQEsO9osp9wjplpEHl5pKO5D1HTNGLAUQ9knT8nQYG7xRkm%2FY6cdYjbrHI16rjxj050sMWOAuua9Ez64I%2FSdOrewP2CrrVItnPWpPZy0Rn3XI8BR6jKVaoY5hAQR13bK%2FE1yM31y%2BC7OXgDjIAoj%2BGCcH7HUGzQWYgY%2Fpv%2FdQjNb%2FPUEzjUaS6nK34V%2BP9OTu24%2BJY3MPo2mZ1oK8uVpYor4BXjnH%2FeIAQAA)
* [supabase.com/docs/guides…](https://supabase.com/docs/guides/getting-started/mcp)
