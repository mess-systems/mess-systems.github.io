---
title: "Codex in JetBrains Air without an OpenAI subscription: my own models through agentgateway"
date: 2026-11-15T00:50:00+03:00
weight: 10
description: "In JetBrains Air you can use Codex without an OpenAI subscription: the client is open, and the models go through my own agentgateway. I look at this setup so a team can actually run it."
---

**Lead:** In JetBrains Air you can use Codex without an OpenAI subscription. The client itself is open and charges nothing for the program, and I connect my own models, through agentgateway. That is what I was putting together. The rest — providers, limits, Figma, the journal and the cost — is only context below: I look at this setup so a team can actually manage it.

## How it is put together

Air in IDEs is not a model provider here. It is a plugin in IntelliJ IDEA, the place where I start an agent and see what it changed. According to JetBrains documentation, the plugin itself has no separate license, and local agents through Agent Client Protocol (ACP) do not need a JetBrains AI subscription.

The chain looks like this: Air in IDEs talks to the agent over ACP, a wrapper injects a personal key only from an environment variable and never writes it to a file, then the agent runs, for example Codex CLI or Qwen Code. I run Qwen Code as an agent in IntelliJ IDEA, and it talks to QwenCloud Token Plan directly, without the gateway: under the terms, that endpoint may be used only in tools like this. Requests from Codex CLI and other agents go to agentgateway; for Qwen models behind the gateway you need QwenCloud Pay-as-you-go, not Token Plan. There is no OpenAI subscription in this setup.

![Diagram of the setup: the developer calls Grok Bot and IntelliJ IDEA with the Air in IDEs plugin; the link from Grok Bot to the MCP Server inside the IDE is still being checked; from Air in IDEs, through an ACP wrapper, the agents Codex CLI, Qwen Code and others fan out: Qwen Code goes directly to QwenCloud Token Plan; Codex CLI and the others go to the agentgateway gateway, provider keys are stored in OpenBao, and the developer holds only the gateway key; behind the gateway are OpenCode Go, QwenCloud Pay-as-you-go, NVIDIA NIM, OpenRouter and other providers, including OrcaRouter, TokenRouter and DeepSeek API; Codex CLI separately calls the remote Figma MCP server, still outside the journal; the gateway writes a Langfuse journal over OpenTelemetry](/blog/codex-in-air-scheme.svg)

The files below are not a production config, they are a sketch. Instead of addresses and secrets there are only placeholders: the key lives in the environment, and the gateway base URL is not baked into the repository either.

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
# agent profile, a sketch
model = "gpt-6-luna"
base_url = "${GATEWAY_URL}"
```

The model changes with the profile, the tool stays the same. Next to it I also run Qwen Code, another open client. Several agents are described in Air, and I can switch the agent without changing the IDE.

I am still figuring out the built-in model picker in Air in IDEs, and whether I need a separate ACP agent for each model.

For your own models, Codex CLI shows a Model metadata not found warning if there is no metadata catalog, that is how the documentation puts it. I am still figuring out how the wrapper should set the provider and a separate Codex config for Air.

## Another direction: an external agent calls the IDE over MCP

Agents reach my IDE in two directions.

The first I have already described: inside the IDE, in Air, the agent works over ACP. I start it there and see what it changed.

The second direction is the reverse. Starting with version 2025.2, IntelliJ IDEA has a built-in MCP Server. The MCP Server plugin ships with the IDE and is enabled by default, but the server itself has to be turned on in the settings, under AI, MCP Server. When you turn it on, the IDE shows what access third-party apps will get to the open projects. From the outside, Grok Bot talks to this IDE: I ask it over MCP to create a project, write tests, or something in the same vein. Other external agents can come in the same way. How Grok Bot is wired to this server in my setup right now, I'll publish once I've checked.

According to [JetBrains documentation](https://www.jetbrains.com/help/idea/mcp-server.html), an external client gets the IDE's own tools: work with files and patches within the project, search, inspections, run configurations, the built-in terminal, and rename as a refactoring. The [debugger](https://www.jetbrains.com/help/idea/agentic-debugging.html) joins that set starting with version 2026.1.3. You can review the tool list and turn them on one by one in the settings, under Exposed Tools; the full list is also in a [JetBrains help article](https://support.jetbrains.com/hc/en-us/articles/40158173063186-MCP-Available-Tools). Terminal commands and run configurations ask for confirmation by default, until you turn on the no-confirmation mode, brave mode. For some clients, such as Codex, Claude Code, VS Code and Air, the IDE can write the connection itself; the rest you connect by hand, by copying an SSE, stdio or HTTP Stream configuration. I have brave mode on, so commands and configurations run without confirmation. Which tools I left enabled, I'll publish once I've checked.

For a team, what matters to me here is the idea itself. One IDE becomes the place where the work actually happens: the agent can live inside it, as in Air, or come in from the outside, as Grok Bot does, but the code, the tests and the runs go through one place, with the IDE's indexing and checks, not around them. How calls from this side will land in the shared journal, I'll publish once I've checked.

I am not sure yet whether I should publish IntelliJ IDEA as an MCP Server through agentgateway: the gateway can do [MCP federation](https://agentgateway.dev/docs/standalone/main/integrations/mcp/servers/virtual/). I will probably think about this some more and at least add it as an experiment.

## Why this matters for a team

I look at this setup not as a trick to "dodge a subscription", but as a circuit a team can actually run, if there are several people.

One gateway gives one entry point. Each person has their own key, not a shared secret in a settings file. The model is chosen by profile: if one door does not answer or has hit a limit, you can switch to another without changing the tool. Requests and tool calls go over OpenTelemetry into Langfuse, which I run myself: you can see who called what, how many tokens and calls there were, and what it cost in money. I still also watch OpenCode Go spend in the subscription's dollar window. Secrets in the journal are masked. Provider keys live in OpenBao; I only hold the gateway key. Whether Codex CLI itself writes OpenTelemetry traces, on top of the gateway, I am still figuring out.

But individual subscriptions do not scale on their own. OpenCode Go is meant for a workspace with one member; for several people you need Go Teams, $20 per person per month, two people minimum. QwenCloud Token Plan Individual is also for personal use only: you can put the key on several of your own devices, you cannot share it, and a team needs Team Edition. I do not put this key on the gateway. A personal gateway for one person with their own OpenCode Go or QwenCloud Pay-as-you-go key still fits this picture. I would not describe one key handed out through a shared gateway to several people, and I would not do it myself.

I write the full text of prompts in the journal only for myself, in the lab. In a team, with other people's data, you need separate instances and your own rules for what to write and who can look.

## Providers and limits

I have several doors wired into the gateway. OpenCode Go costs $10 a month, and the limits are not a request count, but a separate dollar window for each model: 20% per five hours, 50% per week, 100% per month. Also connected through the gateway are QwenCloud Pay-as-you-go, NVIDIA NIM trial access, OpenRouter, OrcaRouter, TokenRouter, DeepSeek API and other providers. NVIDIA NIM allows up to 40 requests per minute for most models, for prototyping only; for OpenRouter's free models: 20 requests per minute and 50 per day, and after you buy credits from $10 the daily limit is 1,000. QwenCloud Token Plan Individual, Standard tier: on November 15, 2026 I paid a year in advance, $195.00 before tax ($214.99 with tax). The plan includes 80,000 credits a month, a rough guide of 3–4 agents at once. I use this plan from Qwen Code directly, under the [Token Plan terms](https://docs.qwencloud.com/token-plan/personal/token-plan-personal-overview), not through the gateway.

I did a narrow run on eight of my own tasks, but this is not a benchmark. On it gpt-6-luna closed 8 out of 8 in about 17 seconds, claude-haiku-5-5 in about 21 seconds, and kimi-k3, glm-5.3 and minimax-m3 also 8 out of 8; I'll publish the exact times once I've measured them. The earlier deepseek-v4-flash took about 188 seconds on the same tasks; I'll publish its full result once I've checked the log.

## What is still missing

I have not put MCP through the gateway yet. Because of that, Figma calls do not land in the shared journal, and the observability picture is still incomplete.

## My day-to-day place

Every day I still work in Cursor, with Grok Bot next to it. That is my main tool. The Cursor Pro+ subscription, by the invoice of May 24, 2026, is $576.00 a year; in the account that shows as $48 a month, and the next renewal is May 24, 2027. Grok Bot is listed in the account as included in Pro+, with a weekly limit. I put Air, Codex and the gateway together next to this place, not instead of it.

## Context: Figma

Figma matters in my daily work, and the subscription is paid: Professional, one Full seat, $192.00 a year by the invoice of June 21, 2026, 3,000 AI credits a month. It connects to the agent through the official remote Figma MCP server; in Codex CLI that is `codex mcp add` and `codex mcp login`. Most MCP tools do not spend credits; `generate_image`, `generate_vector_illustration` and `send_prompt` in Make do. Reads on my seat are limited to 200 calls a day and 10 a minute; writes are not part of this limit. Starting November 19, 2026, the built-in Figma agent in Slides and FigJam begins to spend credits. If you build a presentation through Codex and the ordinary MCP write tools, the documentation says credits are not charged for that.

## What it costs

The figures come from my invoices and a check of the official pages on November 14–15, 2026, before tax and without hardware.

| Component | Plan | Price | Source |
|---|---|---|---|
| Codex CLI | open client, Apache 2.0 | $0 for the program | [github.com/openai/codex](https://github.com/openai/codex) |
| Air in IDEs | IDE plugin, no required JetBrains AI subscription for local ACP agents | $0 for the plugin; the IDE license itself, and what I pay for it, I'll publish once I've checked | [Air in IDEs](https://www.jetbrains.com/help/air/air-in-ides-overview.html) |
| agentgateway | self-hosted, Apache 2.0 | $0 for the program | [FAQ](https://agentgateway.dev/docs/standalone/main/documentation/faqs/) |
| Langfuse v4 | self-hosted; core is MIT, EE catalogs under a separate license | $0 for the open core | [Self-Hosted Pricing](https://langfuse.com/pricing-self-host) |
| OpenCode Go | individual, one member | $10/mo, $120 a year if paid monthly | [Go documentation](https://opencode.ai/docs/console/go/) |
| QwenCloud Token Plan | Individual Standard, yearly payment from my invoice | $195.00 a year before tax ($214.99 with tax), paid November 15, 2026 | my invoice; [Token Plan page](https://www.qwencloud.com/pricing/token-plan) |
| Cursor and Grok Bot | Pro+, yearly payment from my invoice | $576.00 a year, invoice of May 24, 2026. This plan is no longer sold to new buyers; the comparable Super plan is $1000 a year | my invoice; [Cursor pricing](https://cursor.com/help/account-and-billing/pricing), [Grok Bot](https://cursor.com/help/grok-bot/plans) |
| Figma | Professional, 1 Full, year | $192.00, invoice of June 21, 2026 | my invoice; [figma.com/pricing](https://www.figma.com/pricing/) |
| NVIDIA NIM, OpenRouter `:free` | trial access and free models | $0 within the published limits | [NIM](https://build.nvidia.com/llms.txt), [OpenRouter limits](https://www.openrouter.ai/docs/api_reference/limits) |
| Hardware for the gateway and Langfuse | my own | I'll publish the amount once I've added it up | no public price |

In actual payments of mine, before tax: $576 + $192 + $120 + $195 = $1083 a year, about $90 a month, without hardware.

If I were putting together a similar place from scratch today, instead of my Pro+ I would most likely have to take Super at $1000 a year. Then Super, Figma Full, OpenCode Go and QwenCloud Token Plan at my yearly payment of $195 would come to about $1507 a year. The Token Plan figure here is my payment, not the price on the storefront. This is a storefront estimate plus my payment for QwenCloud Token Plan, not my full invoice for a new set. For a team, individual OpenCode Go and QwenCloud Token Plan cannot be shared: those already have team plans.

## For now

I know that Codex can be set up in Air without an OpenAI subscription and fed my own models through a gateway. Right now it seems to me that this is the setup that gives you control: keys, model choice, the journal and the cost in one place, not in five separate subscriptions.

I'll publish the hardware cost once I've added it up. What the full journal will look like once MCP also goes through the gateway, I'll publish once I've checked.

I am very interested in how others organize team work with AI tools: observability, money, convenience and everything else. If you want, tell me how it is set up for you.
