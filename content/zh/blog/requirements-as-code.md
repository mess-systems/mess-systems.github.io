---
title: "需求即代码"
date: 2026-10-22
weight: 2
description: "为什么要把需求以 Markdown 形式放在代码旁边、在 CI 中检查并追溯到测试，以及 mess-systems/requirements 模板是怎么做的。"
---

需求通常与代码分开存放：在 wiki 里、表格里、任务系统里。团队小、大家都记得的时候，这样行得通。之后就会出现难以回答的问题：哪个需求已实现，具体在哪里，有没有对应的测试，谁在什么时候改了措辞。

当 AI 智能体加入工作，问题会更多。智能体写代码很快，但它需要知道自己在完成哪个需求，而且不能由它自己判定需求已经完成。

我们发布了 [mess-systems/requirements](https://github.com/mess-systems/requirements) 模板，在其中像管理代码一样管理需求。下面介绍它的结构以及为什么这样设计。模板中的示例针对 mess.systems 公司的虚构产品 `x`。

## 一个需求，一个文件

每个需求是一个带 YAML 头的 Markdown 文件。文件名与 ID 一致，例如 `X-OPS-001.md`，目录与范围一致：`requirements/x-platform/`。头部包含类型、状态、优先级、负责人、来源、与 ADR 的关联和版本。负责人是角色，例如 `role-platform-engineers`，而不是人名。

正文有三个必填部分：需求本身、理由和验收标准。没有验收标准的需求无法通过检查。这样，"系统必须可靠"这样的表述就进不了登记表：必须把它改写成可以验证的内容。

为什么用文件而不用表格。git 中的文件有历史、通过 PR 评审，每次修改都有作者。需求可以在讨论代码的地方讨论，并在同一个 PR 中与代码一起修改。

## 规则写在配置里，而不是脚本里

ID 前缀、需求类型、范围、状态和优先级都在一个文件 `reqs.config.yaml` 中设定。脚本事先不知道任何一个 ID。要把模板用于自己的项目，改的是配置，而不是代码。

检查由 `validate.py` 完成：头部 schema（`schema/requirement.schema.json`）、文件名与 ID 是否一致、前缀、类型、目录、重复项、是否有验收标准和追溯标记、需求引用的 ADR 是否存在。登记表由 `generate.py` 生成：从文件得到 `REGISTER.md` 和 `register.json`。登记表从不手工编辑，CI 会检查它是否为最新。

## 严格程度逐步开启

新的检查规则不能一上来就以错误模式启用：它会因旧文件而让主分支变红。因此每条规则有各自的模式。规则先以 `warn` 模式运行：CI 打印违规项并附上报告，但不失败。然后由单独的 PR 按报告修复文件。报告清空后，再用一个小 PR 把规则切换为 `error`。

CI 同时提供 GitHub Actions 和 Forgejo Actions 版本。简单情况下一切归结为一条命令：`make check`。

## 状态由人来设定

需求的生命周期是 `draft`、`proposed`、`approved`、`implemented`、`verified`，任何状态都可以转为 `deprecated`。文件不会被删除。

最重要的规则：`approved` 和 `verified` 状态由人通过已批准的 PR 设定。智能体可以创建草稿、提出措辞、把需求与 ADR 关联、给代码打标签。但它不能判定需求已批准或已完成。这一点写进了流程，也写进了给智能体的说明。

## 追溯到代码和测试

追溯使用 [OpenFastTrace](https://github.com/mess-systems/tech-radar/blob/main/technologies/openfasttrace.md)。需求中有标记 `req~X-OPS-001~1` 和一行 `Needs: impl, itest`，表示它需要实现和集成测试。实现代码中加标签 `[impl->req~X-OPS-001~1]`，测试中加 `[itest->req~X-OPS-001~1]`。

`make trace` 生成报告：哪些需求已被代码和测试覆盖，哪些没有。标记末尾的数字是版本。如果已批准需求的措辞有实质性修改，版本号增加，追溯会显示代码中的旧标签需要复查。默认情况下报告不会让构建失败，因为示例只覆盖了部分需求。设置 `TRACE_STRICT=1` 后，任何追溯缺陷都会成为错误。

模板中有一个带这类标签的实现仓库示例，可以看到实际效果。

## 决策放在需求旁边

需求回答"需要什么"，ADR 回答"为什么这样决定"。模板中有 ADR 模板和两个示例：关于密钥和关于备份。关联是双向的：需求中有 `links.adr` 字段，ADR 中有相关需求列表。校验器会检查需求引用的 ADR 是否存在。

## 给不读 git 的人的网站

`make site` 用 [MkDocs](https://github.com/mess-systems/tech-radar/blob/main/technologies/mkdocs.md) 从需求、登记表、ADR 和流程文档构建网站。构建以严格模式运行，失效链接会导致构建失败。网站面向阅读需求但不使用 git 的人。

## 给智能体的规则

智能体和人遵守同样的规则。模板中有面向任何智能体的 `AGENTS.md`、`.cursor/rules/` 中的 Cursor 规则，以及八个现成的提示词：创建需求、关联 ADR、给代码打标签、评审变更、检查覆盖率、在新项目中初始化模板、按报告修复违规、编写验收标准。

对智能体的限制很简短：不提升状态，不绕过失败的 CI，提交 PR 前确认 ID 编号空闲，不删除需求而是转为 `deprecated`，不写入密钥、个人数据和内部地址。

## 与其他部分的关系

需求中的产品、域、环境和负责人名称来自[命名标准](https://github.com/mess-systems/naming-standard)。模板使用的工具（OpenFastTrace、MkDocs、GitHub Actions、Forgejo Actions）在[技术雷达](https://github.com/mess-systems/tech-radar)中有说明。OpenFastTrace 和 MkDocs 在雷达中处于 trial 环：我们在试用，但还不把它们当作默认选择。[blueprints](https://github.com/mess-systems/blueprints) 中参考架构的需求也可以用同一个模板管理。

## 从哪里开始

该仓库在 GitHub 上标记为模板：可以用它创建自己的仓库，并按 README 中简短的快速入门操作。在配置中修改前缀和范围，运行 `make check`，用自己的需求替换示例，保留需要的 CI，并开启分支保护。

我还不知道的是：这种方法在数百个需求、多个各有节奏的团队中表现如何。模板的文本采用 CC BY 4.0，代码采用 Apache-2.0。如果你试用了，欢迎[告诉我们](/zh/about/)哪里碍事。
