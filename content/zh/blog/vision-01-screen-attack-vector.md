---
title: "屏幕作为攻击向量"
date: 2026-10-24
weight: 4
description: "预测：如今看着屏幕的是能理解所见内容并据此行动的机器，屏幕正在成为攻击向量。"
---

# 屏幕作为攻击向量

一切始于给自己写个程序的简单想法。我想截取屏幕的一部分，比如智能体正在工作的窗口或者正在运行的终端，把这部分的视频发给另一个智能体。一个专门的智能体，它什么都不做，只盯着那里看，一旦出了问题就提醒。说到底，这就是我平时手动做的事情的自动版：让一个智能体去检查另一个智能体的输出。

然后冒出一个问题：如果拿到我屏幕同样画面的不是我的智能体，而是攻击者呢？

我觉得，屏幕正在成为一个非常现实的攻击向量。不是因为出现了新的漏洞，而是因为如今看着屏幕的是机器，它们能理解所见的内容，并据此行动。

## 屏幕画面流将无处不在

过去，屏幕上的画面主要是给屏幕前的人看的。现在很多人都会想要它，而且每个人都有充分的理由。

防御者会在工作的智能体旁边放一个观察者智能体，用来发现危险命令和数据泄露。雇主现在就可以截取员工桌面的截图，很快就会由模型来看，而且直接给出结论。国家早就能接入通信：在俄罗斯，运营商必须安装 SORM 设备，其他国家也有各自的监听法律。如果每台电脑上都运行着一个能看到屏幕的智能体，这个来源恐怕不会被放过。

每一路这样的画面流，都是你所见内容的又一份副本，又多了一个可以被偷走的地方。

## 攻击者能做什么

**偷走画面流。** 如果有人拿到了你屏幕的视频，他可能根本不需要接触文件本身。聊天记录、代码、文档、不小心露出来的密码和短信验证码，在日常工作中都会从镜头前经过。以前这样的录像还得有人去看。现在由模型来看，又快又不知疲倦，只挑出有用的部分。

## 老软件被重新审视

即使软件本身没有变化，风险也可能增加。一个服务多年来运行良好，代码没变，但用来剖析它的工具越来越好。昨天还太贵、不值得研究的东西，明天可能就划算了。Google 有个例子：他们的智能体 Big Sleep [发现](https://projectzero.google/2024/10/from-naptime-to-big-sleep.html)了 SQLite 中一个此前未知的内存安全错误。这不意味着 AI 能攻破一切。但在早已熟悉的软件里，仍然有不熟悉的问题。

这时屏幕又出现了。截图上能看到一个人在用什么应用、遇到了什么错误、软件包的版本、刚刚运行的命令。这些本身都不是漏洞，但可以从这里开始。

我觉得，关键不在于有一个神奇的模型，看一眼桌面就能把一切攻破。关键在于深入研究正变得越来越便宜。以前需要一个细心的人来做的工作，可能会变成机器一直在做的事。

## 这些关于入侵的想法从何而来

现在围绕 AI 有很多恐慌，在我看来，OpenAI 和 Anthropic 这样的最大玩家明显在煽风点火。它们需要巨额资金建数据中心，而它们帮着起草的严格规则，正好能把小一些的竞争者挡在门外。这是我的看法，但有些从数字上也能看出来：据 [POGO](https://www.pogo.org/investigates/boom-in-major-ai-company-spending-on-lobbyists) 的数据，Anthropic 在 2024 年花在美国联邦游说上的钱是 104 万美元，2026 年截至第三季度末是 811 万美元；OpenAI 同期分别是 245 万和 477 万美元。这个想法，好像是我从 Steve Eisman 那里偷来或听来的。他在[“Why Dario Amodei and Sam Altman Are Faking the AI Doomsday Crisis”](https://www.youtube.com/watch?v=Q1jSJG1iBL8)这期节目里说，Anthropic 和 OpenAI 在制造恐慌、邀请政府监管这个行业，因为他们预见到价格战，希望通过监管保住自己的定价权，形成“AI 双寡头”。

同时，漏洞以前就有，以前也有人在找，而且一直都很重要。围绕 AI 的喧嚣并不会让风险变成虚构的。它只是让人更难分清哪些是声音大，哪些是真实的。而被智能体盯着的屏幕，恰恰是真实的。

这让我想起含铅汽油的故事。1921 年，通用汽车的一个团队发现了四乙基铅：它确实能防止发动机爆震，还带来了巨额利润。工厂工人在最初几年就开始死亡，但这个行业用自己的研究、对科学家施压和诉讼，几十年来一直在为这个产品辩护（[EPA](https://www.epa.gov/archive/epa/aboutepa/lead-poisoning-historical-perspective.html)）。Clair Patterson 在测定地球年龄时发现铅无处不在，并证明所谓的“正常本底”本身就是污染（[Caltech](https://magazine.caltech.edu/post/clair-patterson-lead)）。他 1963 年在 Nature 上发表文章后，有人威胁要断掉他的经费（[UChicago Magazine](https://mag.uchicago.edu/science-medicine/immeasurable)）。美国对道路车辆的禁令于 1996 年生效，在全球范围内，阿尔及利亚于 2021 年最后一个放弃（[UNEP](https://www.unep.org/ru/novosti-i-istorii/istoriya/po-sledam-20-letney-kampanii-izbavleniya-mira-ot-etilirovannogo-topliva)）。

AI 当然不是铅，但引入它同样要谨慎。关注那些富有的公司在做什么非常重要，围绕这些问题团结起来也同样重要。

## 屏幕已经在被拍下

这不是幻想。智能电视使用 ACR，即自动内容识别：它们提取画面和声音的指纹，与数据库比对，以弄清家里在看什么。得克萨斯州总检察长在[对 LG 的诉讼](https://www.texasattorneygeneral.gov/sites/default/files/images/press/LG%20TV%20Petition%20Filed.pdf)中写道，ACR 每 10 毫秒保存一次“哈希”，并且对包括 HDMI 在内的所有输入都起作用，即使电视被用作电脑显示器也是如此。类似的[诉讼也针对 Samsung 提起](https://www.texasattorneygeneral.gov/sites/default/files/images/press/Samsung%20TV%20Petition%20Filed.pdf)。[Gamers Nexus](https://www.youtube.com/watch?v=6IFVTcM28KA) 对 LG 电视有详细的分析。LG 不同意这些结论，并[表示](https://www.techradar.com/televisions/lg-tvs-collect-far-more-data-on-you-than-youd-expect-says-new-report-including-logging-microphone-audio-while-on-standby) ACR 只有在用户同意后才会开启。
