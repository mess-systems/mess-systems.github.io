---
title: "在 JetBrains Air 里用 Codex，不必订阅 OpenAI：通过 agentgateway 接入自己的模型"
date: 2026-11-15T00:50:00+03:00
weight: 10
description: "在 JetBrains Air 里可以不订阅 OpenAI 就使用 Codex：客户端是开源的，模型走自己的 agentgateway 网关。我看这套链路，是为了以后可以由团队来管理。"
---

**导语：** 在 JetBrains Air 里可以不订阅 OpenAI 就使用 Codex。客户端本身是开源的，不为此收费，模型则是我自己的，通过 agentgateway 网关接入。我要搭的就是这个。其余的——提供商、限额、Figma、日志和费用——下面只作为背景：我看这套链路，是为了以后可以由团队来管理。

## 它是怎么搭起来的

Air in IDEs 在这里不是模型供应商。它是 IntelliJ IDEA 里的一个插件，是我启动智能体、查看它改了什么的地方。按 JetBrains 的文档，插件本身没有单独的许可证，通过 Agent Client Protocol（ACP）运行的本地智能体也不需要 JetBrains AI 订阅。

链路是这样的：Air in IDEs 通过 ACP 与智能体通信，一层包装只从环境变量里注入个人密钥，不把它写进文件，然后运行智能体，例如 Codex CLI 或 Qwen Code。我在 IntelliJ IDEA 里把 Qwen Code 当作智能体来跑，它直接访问 QwenCloud Token Plan，不经过网关：按条款，这个 endpoint 只能用在这类工具里。来自 Codex CLI 和其他智能体的请求则进入 agentgateway；网关后面的 Qwen 模型需要的是 QwenCloud Pay-as-you-go，不是 Token Plan。这套方案里没有 OpenAI 订阅。

![链路示意：开发者调用 Grok Bot，以及带有 Air in IDEs 插件的 IntelliJ IDEA；Grok Bot 与 IDE 内 MCP Server 的连接仍在核实；从 Air in IDEs 经 ACP 包装，Codex CLI、Qwen Code 等智能体再分开走：Qwen Code 直接进入 QwenCloud Token Plan；Codex CLI 和其他智能体进入 agentgateway 网关，提供商密钥存放在 OpenBao，开发者手上只有网关密钥；网关后面是 OpenCode Go、QwenCloud Pay-as-you-go、NVIDIA NIM、OpenRouter 以及其他提供商，其中包括 OrcaRouter、TokenRouter 和 DeepSeek API；Codex CLI 另外调用远程 Figma MCP server，目前还不在日志里；网关通过 OpenTelemetry 把日志写入 Langfuse](/blog/codex-in-air-scheme.svg)

下面不是生产环境的配置，只是示意。地址和密钥都换成了占位符：密钥放在环境变量里，网关的基础地址也不写进仓库。

```json
{
  "agents": [
    {
      "name": "codex",
      "command": ["codex-wrapper"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  ]
}
```

```toml
# 智能体配置，仅作示意
model = "gpt-6-luna"
base_url = "${GATEWAY_URL}"
```

换模型靠配置，工具还是同一个。旁边我同样跑着 Qwen Code，也是开源客户端。Air 里描述了几个智能体，我可以换智能体，而不换 IDE。

Air in IDEs 里的内置模型选择，以及是否需要为每种模型单独配一个 ACP 智能体，我还在弄清楚。

对于自己的模型，如果没有元数据目录，Codex CLI 会给出 Model metadata not found 警告，文档里就是这样写的。包装层该如何指定提供商，以及如何为 Air 单独准备一份 Codex 配置，我还在弄清楚。

## 另一个方向：外部智能体通过 MCP 调用 IDE

智能体到我的 IDE，有两个方向。

第一个我已经写过：在 IDE 内部，在 Air 里，智能体通过 ACP 工作。我在那里启动它，并查看它改了什么。

第二个方向是反过来的。从 2025.2 版起，IntelliJ IDEA 有内置的 MCP Server。MCP Server 插件随 IDE 一起提供，并且默认是开启的，但服务器本身需要在设置里打开，路径是 AI、MCP Server。打开时，IDE 会说明第三方应用将对已打开的项目获得哪些访问权限。从外部走进这套 IDE 的，是 Grok Bot：我通过 MCP 请它做项目、写测试，或做类似的事。其他外部智能体也可以这样进来。Grok Bot 现在具体是怎么接到这个服务器上的，等我核对后再写。

按 [JetBrains 文档](https://www.jetbrains.com/help/idea/mcp-server.html)，外部客户端会获得 IDE 自身的工具：在项目范围内处理文件和补丁、搜索、代码检查、运行配置、内置终端，以及作为重构的重命名。[调试器](https://www.jetbrains.com/help/idea/agentic-debugging.html)从 2026.1.3 版起加入这个集合。可以在设置的 Exposed Tools 里逐个查看和开启工具；完整列表也在 [JetBrains 帮助文章](https://support.jetbrains.com/hc/en-us/articles/40158173063186-MCP-Available-Tools)里。终端命令和运行配置默认会请求确认，除非打开无需确认的模式，也就是 brave mode。对一部分客户端，比如 Codex、Claude Code、VS Code 和 Air，IDE 能自己写好连接；其余的要手工接入，复制 SSE、stdio 或 HTTP Stream 配置。我开了 brave mode，所以命令和配置运行时不再确认。我留下了哪些工具，等我核对后再写。

对团队来说，我看重的是这件事本身。一个 IDE 成为工作真正发生的地方：智能体可以住在里面，像在 Air 里那样，也可以从外面进来，像 Grok Bot 那样，但代码、测试和运行都经过同一个地方，带着 IDE 的索引和检查，而不是绕开它。这边的调用以后会如何进入共用日志，等我核对后再写。

我还不确定要不要通过 agentgateway 把 IntelliJ IDEA 作为 MCP Server 发布出去：网关可以做 [MCP 联邦](https://agentgateway.dev/docs/standalone/main/integrations/mcp/servers/virtual/)。这件事我大概还会再想想，至少会先拿来做个实验。

## 这对团队意味着什么

我看这套方案，不是把它当成「绕过订阅」的技巧，而是当成一条在有几个人的时候也能管起来的链路。

一个网关就是一个入口。每个人有自己的密钥，而不是设置文件里的一把共用密钥。模型靠配置来选：如果一扇门没有响应，或碰到了限额，可以换另一扇，而不换工具。请求和工具调用通过 OpenTelemetry 进入我自己部署的 Langfuse：能看到谁调用了什么、用了多少 token 和多少次调用，以及花了多少钱。OpenCode Go 的花钱情况，我还在订阅的美元窗口里看。日志里的密钥会被掩码。提供商密钥放在 OpenBao 里，我手上只有网关密钥。除了网关之外，Codex CLI 自己会不会写 OpenTelemetry 追踪，我还在弄清楚。

但个人订阅不会自己变成可规模化的方案。OpenCode Go 面向只有一名成员的工作空间；几个人一起用，需要 Go Teams，每人每月 $20，至少两个人。QwenCloud Token Plan Individual 也只供个人使用：密钥可以装在自己的几台设备上，但不能共享，团队需要 Team Edition。这把密钥我不会放到网关上。一个人用自己的 OpenCode Go 或 QwenCloud Pay-as-you-go 密钥，再配自己的网关，还说得通。一把密钥经共用网关分给几个人，我不会这样写，自己也不会这样做。

日志里写完整提示词，我只在自己的实验室里这样做。在团队里，面对别人的数据，需要单独的实例，以及关于写什么、谁可以看的自己的规则。

## 提供商和限额

网关上我接了几扇门。OpenCode Go 每月 $10，限额不是按请求次数算的，而是为每个模型单独设一个美元窗口：五小时 20%，一周 50%，一个月 100%。经网关接入的还有 QwenCloud Pay-as-you-go、NVIDIA NIM 试用、OpenRouter、OrcaRouter、TokenRouter、DeepSeek API 和其他提供商。NVIDIA NIM 对大多数模型每分钟最多 40 次请求，仅用于原型；OpenRouter 的免费模型是每分钟 20 次、每天 50 次，购买从 $10 起的额度后，每天限额是 1,000 次。QwenCloud Token Plan Individual，Standard 档：2026 年 11 月 15 日我预付了一年，$195.00 不含税（含税 $214.99）。套餐每月 80,000 积分，大约可同时跑 3–4 个智能体。这个计划我从 Qwen Code 直接使用，按 [Token Plan 条款](https://docs.qwencloud.com/token-plan/personal/token-plan-personal-overview)，不经过网关。

我用自己的八个任务做过一次窄的试跑，但这不是基准测试。其中 gpt-6-luna 用大约 17 秒完成了 8 个里的 8 个，claude-haiku-5-5 大约 21 秒，kimi-k3、glm-5.3 和 minimax-m3 也是 8 个里的 8 个；确切时间等我测完再写。以前的 deepseek-v4-flash 在同样的任务上大约花了 188 秒；完整结果等我按日志核对后再写。

## 目前还缺什么

我还没有让 MCP 经过网关。因此 Figma 的调用进不了共用日志，可观测性的画面还不完整。

## 我日常待的地方

我每天仍然在 Cursor 里工作，旁边是 Grok Bot。这是我的主要工具。Cursor Pro+ 订阅按 2026 年 5 月 24 日的账单是一年 $576.00，账户里显示为每月 $48，下次续费是 2027 年 5 月 24 日。账户里写着 Grok Bot 包含在 Pro+ 中，并有每周限额。Air、Codex 和网关是我在这个位置旁边搭起来的，不是用来取代它。

## 背景：Figma

Figma 对我的日常工作很重要，订阅是付费的：Professional，一个 Full 席位，按 2026 年 6 月 21 日的账单是一年 $192.00，每月 3,000 个 AI 积分。它通过官方的远程 Figma MCP server 接到智能体上，在 Codex CLI 里是 `codex mcp add` 和 `codex mcp login`。大多数 MCP 工具不消耗积分；会消耗的是 `generate_image`、`generate_vector_illustration` 和 Make 里的 `send_prompt`。我这个席位上的读取限制是每天 200 次、每分钟 10 次，写入不计入这个限额。从 2026 年 11 月 19 日起，Slides 和 FigJam 里的内置 Figma 智能体开始扣除积分。如果通过 Codex 和普通的 MCP 写入工具来做演示文稿，按文档是不扣积分的。

## 这要花多少钱

数字来自我的账单，以及 2026 年 11 月 14–15 日对官方页面的核对，不含税，也不含硬件。

| 组件 | 方案 | 价格 | 来源 |
|---|---|---|---|
| Codex CLI | 开源客户端，Apache 2.0 | 程序本身 $0 | [github.com/openai/codex](https://github.com/openai/codex) |
| Air in IDEs | IDE 插件，本地 ACP 智能体不必订阅 JetBrains AI | 插件 $0；IDE 本身的许可证，以及我为此付了多少，等我核对后再写 | [Air in IDEs](https://www.jetbrains.com/help/air/air-in-ides-overview.html) |
| agentgateway | 自行部署，Apache 2.0 | 程序本身 $0 | [FAQ](https://agentgateway.dev/docs/standalone/main/documentation/faqs/) |
| Langfuse v4 | 自行部署；核心为 MIT，EE 目录另有许可证 | 开源核心 $0 | [Self-Hosted Pricing](https://langfuse.com/pricing-self-host) |
| OpenCode Go | 个人版，一名成员 | $10/月，按月付则一年 $120 | [Go 文档](https://opencode.ai/docs/console/go/) |
| QwenCloud Token Plan | Individual Standard，按我的账单年付 | 一年 $195.00 不含税（含税 $214.99），2026 年 11 月 15 日已付 | 我的账单；[Token Plan 页面](https://www.qwencloud.com/pricing/token-plan) |
| Cursor 和 Grok Bot | Pro+，按我的账单年付 | 一年 $576.00，账单日期 2026 年 5 月 24 日。这个方案已经不再向新买家出售；相近的 Super 是一年 $1000 | 我的账单；[Cursor 价格](https://cursor.com/help/account-and-billing/pricing)，[Grok Bot](https://cursor.com/help/grok-bot/plans) |
| Figma | Professional，1 个 Full，年付 | $192.00，账单日期 2026 年 6 月 21 日 | 我的账单；[figma.com/pricing](https://www.figma.com/pricing/) |
| NVIDIA NIM、OpenRouter `:free` | 试用和免费模型 | 在已公布的限额内 $0 | [NIM](https://build.nvidia.com/llms.txt)，[OpenRouter 限额](https://www.openrouter.ai/docs/api_reference/limits) |
| 网关和 Langfuse 所用的硬件 | 自己的 | 金额等我算完再写 | 没有公开价格 |

按我实际支付、不含税来算：$576 + $192 + $120 + $195 = 一年 $1083，大约每月 $90，不含硬件。

如果今天从零搭一套相近的环境，我的 Pro+ 多半得换成一年 $1000 的 Super。那样的话，Super、Figma Full、OpenCode Go，再加上我为 QwenCloud Token Plan 实际支付的一年 $195，大约是一年 $1507。这里 Token Plan 的数字是我的支付金额，不是货架价格。这是按货架估价，再加上我为 QwenCloud Token Plan 支付的金额，并不是一套全新组合的完整账单。对团队来说，个人版的 OpenCode Go 和 QwenCloud Token Plan 不能共用：那边已经有团队套餐。

## 先这样

我知道，可以在 Air 里装上 Codex，不必订阅 OpenAI，并用网关给它接入自己的模型。眼下我觉得，正是这样的方案带来了掌控：密钥、模型选择、日志和费用在一个地方，而不是分散在五个订阅里。

硬件成本等我算完再写。等 MCP 也经过网关之后，完整日志会是什么样子，等我核对后再写。

我很想知道别人是怎么组织团队使用 AI 工具的：可观测性、财务、使用是否方便，以及其他方方面面。如果愿意，欢迎讲讲你们那边是怎么安排的。
