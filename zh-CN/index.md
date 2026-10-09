---
layout: default
title: "中文版"
nav_order: 50
---

# 交通智能体工程（TAE）

### 一份关于交通智能体应如何思考、推理并承担责任的开放规范

**v0.1 — 草案。** 卷一与卷二完整——第一至二十章。

这不是一本书，是一份规范。它所定义的概念具有足够的精确性：两个互不相关的团队读完之后，应当能够构建出可互操作的交通智能体——或者至少，能够就分歧之处进行有成效的争论。

[English →](../docs/index.md) · [术语表 →](../glossary/index.md) · [原语规范 →](../primitives/index.md)

---

## 缺口在哪

智能交通系统的二十年，让交通变得**可观测**、**可仿真**，却从未让交通工程变得**可执行**。

我们的模型能感知拥堵、预测需求、高保真地仿真整条走廊。但工程流程本身没有变：工程师仍然在收集数据、诊断问题、计算配时方案、撰写报告。计算机变聪明了，工程没有。

大语言模型本身填不上这个缺口。它能谈论排队长度，却无法为其负责。它能生成**读起来像**工程判断的文字，却产不出让工程判断站得住脚的证据链——而在交通工程里，**可辩护性就是产品本身**。一份无法追溯到方法、标准和实测数据的配时方案，是不能签字的。

## 核心论点

交通工程不是一个靠更好的模型就能解决的知识问题。它是一个**编译问题**。

交通工程的基本单元不是问题，也不是任务，而是**工程责任**——一项具备明确范围、约束条件、所需证据、完成标准与责任主体的交付承诺。

接受这一点之后，整个技术栈自然展开：

```
责任  →  中间表示  →  推理规划器  →  原语
      →  依赖图  →  编译器  →  验证  →  交付
```

## 卷一 · 章节

### 第一部分 · 基础

| | 章节 |
|---|------|
| 1 | [智能交通的终结？](./chapter-01-the-end-of-intelligent-transportation.md) |
| 2 | [从交通工程到交通智能体工程](./chapter-02-from-traffic-engineering-to-tae.md) |

### 第二部分 · 架构

| | 章节 |
|---|------|
| 3 | [工程责任——交通工程的原子计算单元](./chapter-03-engineering-responsibility.md) |
| 4 | [工程中间表示——连接工程责任与可执行推理的桥梁](./chapter-04-engineering-intermediate-representation.md) |
| 5 | [推理规划器——编译工程思维](./chapter-05-reasoning-planner.md) |
| 6 | [工程原语——可执行推理的最小单元](./chapter-06-engineering-primitive.md) |
| 7 | [原语依赖：工程推理的语法](./chapter-07-primitive-dependencies.md) |
| 8 | [工程交付：推理创造价值之处](./chapter-08-engineering-delivery.md) |
| 9 | [工程验证：从生成结果到可信决策](./chapter-09-engineering-validation.md) |

## 卷二 · 章节

| | 章节 |
|---|------|
| 10 | [工程分析策略](./chapter-10-engineering-analysis-strategy.md) |
| 11 | [领域推理编译器](./chapter-11-domain-reasoning-compiler.md) |
| 12 | [工程交付引擎](./chapter-12-engineering-delivery-engine.md) |
| 13 | [工程验证](./chapter-13-engineering-validation.md) |
| 14 | [交通态势评估](./chapter-14-traffic-situation-assessment.md) |
| 15 | [交通工程策略](./chapter-15-traffic-engineering-strategies.md) |
| 16 | [领域推理编译器——高级机制](./chapter-16-drc-advanced-mechanisms.md) |
| 17 | [领域原语库——工程推理的构建模块](./chapter-17-domain-primitive-library.md) |
| 18 | [交通智能体运行时——执行工程推理](./chapter-18-traffic-agent-runtime.md) |
| 19 | [工程交付系统——从推理到可行动成果](./chapter-19-engineering-delivery-system.md) |
| 20 | [WayMind——交通智能体工程的参考实现](./chapter-20-waymind-reference-implementation.md) |

## 按角色选读

| 你的身份 | 建议读 |
|---------|--------|
| **判断 TAE 是否值得关注** | [术语表](../glossary/index.md)，然后第 1–2 章 |
| **正在实现交通智能体** | [原语规范 / TPS](../primitives/index.md)，然后第 3–4、6–7 章 |
| **关心可信与可审计** | 第 8–9 章，以及[开放议题](../primitives/index.md#open-issues) |
| **持怀疑态度** | 第 2.7 节（TAE 的定义），以及[术语表中刻意排除的词](../glossary/index.md#terms-deliberately-excluded) |

## 状态与坦诚

这是草案，包含**已知且公开列出的开放议题**——见[开放议题](../primitives/index.md#open-issues)。其中最关键的一条：TPS schema 中 `category` 字段的定义值集与实际使用的值集不一致。该问题尚未解决，我们宁愿把它作为公开问题发布，也不愿私下选一个、然后假装它本来就是这么设计的。

卷二把论证推进到编译器、运行时，以及一个可运行的参考实现。如果说前面的章节说明一个组件**是什么**，卷二说明它**必须做到什么**——以契约与保证的形式写出来，精确到可以让独立的实现被逐条对照检查。

## 许可

| 内容 | 协议 |
|------|------|
| 正文——章节、论述 | [CC BY-NC-SA 4.0](./LICENSE.md) |
| 术语表、TPS、原语标识符 | [CC BY 4.0](../LICENSE-SPEC.md) |

这种不对称是刻意设计的。我们没打算靠一份规范来收许可费。我们要做的是让这套词汇变得无法绕开。

---

*交通工程不是知识问题，而是编译问题。*
