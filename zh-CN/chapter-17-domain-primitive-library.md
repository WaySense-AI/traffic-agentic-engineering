---
layout: default
title: "第17章 · 领域原语库——工程推理的构建模块"
nav_order: 67
---

# 第十七章：领域原语库——工程推理的构建模块

## 17.1 工程智能的原子单元

如果领域推理编译器（第十六章）是将责任转化为推理计划的引擎，那么**工程原语就是该引擎消耗的燃料**。原语是给定领域内工程能力的最小可复用单元——若进一步分解将失去其工程语义。

这一定义具有深远含义：

> **工程原语不是软件函数。它是一个工程知识单元，以机器可以执行但工程师能够理解、验证和质疑的形式编码。**

软件函数实现算法，而工程原语则**体现一种方法**——包含其理论基础、适用条件、输入要求、输出语义、已知局限性和验证准则。两者的区别在于代码与知识的区别。

本章全面阐述领域原语库：什么是原语、如何被规范、组织、组合以及作为活的工程资产进行维护。

---

## 17.2 定义工程原语

### 17.2.1 形式化定义

工程原语 `P` 定义为一个元组：

```
P = (name, category, inputs, outputs, assumptions, constraints,
     method, validation, provenance, strategy_applicability)
```

| 组成部分 | 描述 | 示例（饱和流率） |
|---|---|---|
| `name` | 规范标识符 | `CalculateSaturationFlowRate` |
| `category` | 原语类型（派生/估计/模拟/验证/决策） | `derivation` |
| `inputs` | 类型化输入规范 | `Observation(流量)`、`Reference(车道配置)` |
| `outputs` | 类型化输出规范 | `Derived(饱和流率)` |
| `assumptions` | 必须成立的前置条件 | "稳定交通流条件" |
| `constraints` | 输出的边界条件 | "1450 ≤ SFR ≤ 2500 veh/h/ln" |
| `method` | 所使用的算法或流程 | HCM 第六版方法 |
| `validation` | 应用的正确性检查 | 范围检查、合理性检查 |
| `provenance` | 来源权威 | HCM 第十九章 / 本地标定 |
| `strategy_applicability` | 哪些策略使用此原语 | 诊断、设计、评估 |

每个组成部分都是**显式的**。没有隐含内容，没有隐藏信息，不依赖约定或程序员直觉。这种显式性使原语具有可审计性、可争议性和可改进性。

### 17.2.2 原语 vs 软件函数

工程原语与软件函数的区别对于理解 TAE 至关重要：

| 维度 | 软件函数 | 工程原语 |
|---|---|---|
| **目的** | 执行计算 | 编码工程方法 |
| **接口** | 参数列表 | 类型化 I/O + 假设 + 约束 |
| **语义** | 由实现定义 | 由领域定义（HCM、MUTCD 等） |
| **正确性** | 通过单元测试 | 产生工程有效结果 |
| **错误处理** | 异常、错误码 | 需要工程判断 |
| **组合** | 函数调用 | 带类型检查的依赖图 |
| **文档** | 代码注释 | 完整规范文档 |
| **演进** | 重构 | 方法修订 / 新研究 |
| **问责** | 开发者 | 领域专家 + 来源权威 |

计算饱和流率的软件函数可能如下所示：

```python
def saturation_flow(volume, lanes):
    return volume / lanes * 1800  # ???
```

同一计算的工程原语包含确定此计算是否适用于当前工程任务、是否正确、是否充分所需的一切要素。

### 17.2.3 不可分原则

原语在其领域内是**不可分的**——并非因为算法上不能分解为更小的步骤，而是因为这样做会丢失工程意义。考虑：

```
CalculateDelay(HCMMethod, Volume, Capacity, ProgressionFactor)
```

这可以被分解为：
1. 计算 v/c 比
2. 查找均匀延误系数
3. 计算均匀延误分量
4. 查找增量延误系数
5. 计算增量延误分量
6. 求各分量之和

但这些子步骤**不是独立的工程概念**——它们是 HCM 延误流程的实现细节。没有交通工程师会请求"计算增量延误系数"作为独立任务。工程工作的原子单元是**延误估算**，而非其内部算术运算。

不可分原则确保原语库在**正确的抽象层次**运作——即工程师实际思考、交流和做出决策的层次。

---

## 17.3 工程原语分类体系

### 17.3.1 类别划分

原语根据其在推理链中的作用分为六个基本类别：

#### 类别一：观测原语 (`obs:`)

从原始观测中提取结构化数据。

| 原语 | 输入 | 输出 | 说明 |
|---|---|---|---|
| `obs:ExtractVolumeCounts` | 检测器日志 | `Observation(流量矩阵)` | 高峰小时提取、缺失数据处理 |
| `obs:ExtractSpeedData` | 检测器/视频 | `Observation(速度分布)` | 点速度 vs 行程速度 |
| `obs:ExtractOccupancy` | 检测器日志 | `Observation(占有率)` | 时间序列占有率模式 |
| `obs:ExtractQueueLength` | 视频/检测器 | `Observation(排队估计)` | 最大/平均/持续排队长度 |
| `obs:ExtractEventData` | 事件日志 | `事件记录>` | 持续时间、类型、位置、影响 |

观测原语是**物理世界与推理系统之间的边界**。它们将原始传感器数据转换为类型化的工程观测值，在此过程中应用质量检查和不确定性量化。

#### 类别二：派生原语 (`drv:`)

应用确定性变换以产生派生量。

| 原语 | 输入 | 输出 | 方法 |
|---|---|---|---|
| `drv:CalculateSaturationFlow` | 流量、车道配置 | `Derived(饱和流率)` | HCM 第十九章 / 实地测量 |
| `drv:CalculateCapacity` | 饱和流率、相位 | `Derived(容量)` | 关键车道求和 / 共用车道 |
| `drv:CalculateDegreeOfSaturation` | 流量、容量 | `Derived(Xc)` | v/c 比 |
| `drv:CalculateDelay` | Xc、C、g/C、协调系数 | `Derived(延误)` | HCM 均匀 + 增量 |
| `drv:CalculateLOS` | 延误、控制类型 | `Derived(LOS)` | HCM LOS 阈值 |
| `drv:CalculateQueueStorage` | 几何条件、到达模式 | `Derived(存储比)` | 可用 vs 需求 |

派生原语是**交通工程计算的骨干**。它们实现来自 HCM 等来源的成熟方法，具有清晰的输入、确定性的输出和详尽的假设文档。

#### 类别三：估计原语 (`est:`)

推断无法直接测量的量。

| 原语 | 输入 | 输出 | 不确定性来源 |
|---|---|---|---|
| `est:EstimateODMatrix` | 计数、部分 OD | `Estimated(OD矩阵)` | 欠定系统 |
| `est:EstimateTurningProportions` | 流向计数 | `Estimated(转向比例)` | 抽样误差、时变 |
| `est:EstimateDemand` | 观测计数、修正 | `Estimated(需求)` | 高峰系数、增长 |
| `est:EstimateFutureVolume` | 历史、土地利用 | `Estimated(预测流量)` | 土地利用模型不确定性 |
| `est:EstimateSaturationFlowDefault` | 区域类型、设施类型 | `Estimated(SFR)` | 默认值、本地变异 |

估计原语显式地**量化不确定性**。它们的输出始终是 `Estimated(T)` 类型，而非 `Derived(T)` 类型——这一区别提醒所有人涉及了推断过程。

#### 类别四：模拟原语 (`sim:`)

执行交通模拟模型。

| 原语 | 输入 | 输出 | 模型 |
|---|---|---|---|
| `sim:RunSynchro` | 网络、配时、流量 | `Simulated(性能指标)` | Synchro Studio |
| `sim:RunVISSIM` | 网络、行为参数 | `Simulated(微观指标)` | PTV VISSIM |
| `sim:RunParamics` | 网络、需求矩阵 | `Simulated(网络指标)` | Paramics |
| `sim:RunMesoscopic` | 走廊、简化模型 | `Simulated(走廊级指标)` | 快速中观模型 |

模拟原语封装外部模拟工具，无论使用何种具体模拟器都提供**统一接口**。它们还将模拟元数据（模型版本、种子值、运行时长）附加到所有输出上。

#### 类别五：验证原语 (`val:`)

验证其他原语输出的正确性。

| 原语 | 目标 | 检查类型 | 严重程度 |
|---|---|---|---|
| `val:CheckDataQuality` | 观测数据 | 完整性、合理性 | 错误/警告 |
| `val:CheckPhysicalBounds` | 派生物理量 | 物理可行性 | 错误 |
| `val:CheckConstraintSatisfaction` | 设计结果 | 政策/工程限制 | 错误/警告 |
| `val:CrossValidate` | 多源数据 | 方法间一致性 | 警告 |
| `val:CheckHistoricalConsistency` | 结果 vs 历史 | 合理变化幅度 | 警告 |
| `val:SanityCheckResult` | 最终交付物 | 工程师级合理性 | 信息/错误 |

验证原语在 TAE 中**绝非可选**——它们由 DRC 根据策略需求自动插入（第十六章）。

#### 类别六：决策原语 (`dec:`)

在算法无法决定处应用工程判断。

| 原语 | 上下文 | 输出 | 需要 |
|---|---|---|---|
| `dec:SelectAnalysisMethod` | 数据可用性、精度需求 | `Decision(方法选择)` | 权衡判断 |
| `dec:SelectDesignCriteria` | 机构政策、上下文 | `Decision(设计标准)` | 政策解读 |
| `dec:ResolveConflictingEvidence` | 多源数据不一致 | `Decision(证据加权)` | 专家判断 |
| `dec:DetermineScope` | 责任陈述 | `Decision(分析边界)` | 范围判断 |
| `dec:ClassifySituation` | 观测、上下文 | `Decision(态势类型)` | 模式识别 |

决策原语代表**人类专业能力的不可约核心**——工程判断尚不能（还）完全自动化的节点。TAE 使这些节点**显式化**，而非将它们隐藏在不透明的模型权重之中。

### 17.3.2 跨类别组合

真实工程任务组合来自多个类别的原语。以下是信号配时优化任务的典型组合模式：

```
[obs:] ExtractVolumeCounts ─────┐
[obs:] ExtractGeometry ─────────┤
[obs:] ExtractSignalTiming ────┤
                                  ├─→ [drv:] CalculateSaturationFlow
[ref:] HCM_SaturationDefaults ──┤      │
                                  │      ├─→ [drv:] CalculateCapacity
[drv:] CalculateLaneGroups ──────┘      │
                                         │
[ref:] Webster_Method ──────────────────┤      │
                                         ├─→ [drv:] AllocateGreenTimes
[constr:] MinGreen_Policy ───────────────┤      │
[constr:] PedClearance_Requirements ────┘      │
                                                │
[val:] ValidateTimingPlan ◄────────────────────┘
        │
        ├─→ [drv:] EvaluateLOS
        │         │
│         └─→ [del:] GenerateTimingReport
│
└─→ [val:] CrossValidateWithSimulation
```

这一组合涉及**全部六个类别**协同工作——这是非平凡工程任务的典型模式。

---

## 17.4 原语规范语言

### 17.4.1 完整规范模板

库中的每个原语都有完整的机器可读规范。以下是带有具体示例的完整模板：

```yaml
# ============================================================
# 原语规范：CalculateSaturationFlowRate
# ============================================================

meta:
  id: drv:CalculateSaturationFlowRate
  version: "2.1.0"
  last_updated: "2025-03-15"
  author: "TAE 核心团队"
  reviewer: "Dr. [交通工程专家]"
  status: active  # active | deprecated | experimental

classification:
  category: derivation
  subcategory: capacity_analysis
  hcm_reference: "HCM 第6版，第十九章，图表19-14"
  alternative_methods:
    - id: field_measurement
      reference: "公路容量手册，附录"
    - id: regression_model
      reference: "本地机构标定研究（2023）"

inputs:
  - name: approach_volumes
    display_name: "进口道高峰小时流量"
    type: "Observation(Volume) | Estimated(Volume)"
    required: true
    description: "按流向划分的高峰小时进口道流量（veh/h）"
    units: "veh/h"
    constraints:
      min: 0
      max: 5000
      sanity: "应与几何容量一致"

  - name: lane_configuration
    display_name: "车道配置"
    type: "Reference(LaneConfiguration)"
    required: true
    description: "车道数量、车道功能分配、共享流向"
    units: N/A
    source: "实地观测或竣工图纸"

  - name: heavy_vehicle_percentage
    display_name: "大型车比例"
    type: "Observation(HV%) | Estimated(HV%)"
    required: false
    default: 0.02
    description: "交通流中的大型车百分比"
    units: "%"
    range: [0, 100]

outputs:
  - name: saturation_flows
    display_name: "饱和流率"
    type: "Derived(SaturationFlow)"
    description: "各车道组饱和流率（veh/h/ln）"
    units: "veh/h/ln"
    shape: "array[lane_groups]"

assumptions:
  - id: stable_flow
    text: "交通流处于稳定（非崩溃）状态"
    violation_handling: "标记警告；结果可能高估真实 SFR"
  - id: base_conditions
    text: "基础饱和流率假定为 1900 pc/h/ln（小客车）"
    violation_handling: "若条件不同则应用修正系数"

constraints:
  - name: minimum_sfr
    expression: "output >= 1450"
    source: "HCM 信号交叉口下限"
    severity: error
    message: "饱和流率低于物理合理最小值"

  - name: maximum_sfr
    expression: "output <= 2500"
    source: "正常条件下经验上限"
    severity: warning
    message: "异常高的饱和流率；请核实数据"

method:
  primary: hcm_default
  algorithm: |
    对每个车道组 LG：
      1. 确定基础饱和流率：SFR_base = 1900 pc/h/ln
      2. 应用大型车修正：f_HV = 1/(1 + P_HV*(E_HV-1))
      3. 应用坡度修正（如适用）：f_G = 1 - 0.005*grade%
      4. 应用其他修正（停车、公交、区域、车道宽度）
      5. SFR_LG = SFR_base × f_HV × f_G × f_P × f_B × f_A × f_W
    返回各车道组的 SFR 数组

  alternatives:
    - name: field_measurement
      trigger: "检测器数据充足"
      method: "基于车头时距的饱和绿灯时间测量"
    - name: local_calibration
      trigger: "本地机构已标定模型"
      method: "使用机构特定的回归方程"

validation:
  pre_checks:
    - name: data_completeness
      condition: "approach_volumes not null AND lane_configuration not null"
      severity: error
    - name: volume_sanity
      condition: "sum(approach_volumes) < 10000"
      severity: warning

  post_checks:
    - name: range_check
      condition: "ALL(1450 <= sfr <= 2500)"
      severity: error
    - name: consistency_check
      condition: "max(sfr)/min(sfr) < 3.0"
      severity: warning
    - name: comparison_to_defaults
      condition: "abs(sfr - hcm_default) < 30%"
      severity: info

evidence_requirements:
  - type: input_data
    retention: full_traceability
  - type: intermediate_values
    retention: key_adjustment_factors
  - type: assumptions_applied
    retention: list_of_assumptions_with_status

strategy_applicability:
  diagnosis: true       # 容量分析所需
  design: true          # 配时计算的核心输入
  evaluation: true      # LOS 确定所需
  prediction: conditional # 仅在预测容量时需要
  control: false        # 不用于实时控制
  planning: true        # 用于长期规划研究

dependencies:
  - target: approach_volumes
    type: data
  - target: lane_configuration
    type: reference
  - target: heavy_vehicle_percentage
    type: data_optional
  - target: hcm_adjustment_factors
    type: reference

known_limitations:
  - "假定分析期内到达模式均匀"
  - "对变配时的感应信号精度较低"
  - "大型车小客车当量可能存在地方差异"
  - "未考虑下游回流效应"

version_history:
  - version: "2.1.0"
    date: "2025-03-15"
    changes: "新增 local_calibration 替代方案；更新 HV PCE 值"
  - version: "2.0.0"
    date: "2024-06-01"
    changes: "重构为新规范格式；增加证据追踪"
  - version: "1.0.0"
    date: "2023-01-15"
    changes: "初始规范"
```

这种详细程度看似过度——但正是这种细节使 DRC 能够编译正确的推理链、TAR 能够可靠地执行它们、工程师能够验证每一步。

### 17.4.2 规范质量标准

并非所有规范都是等同的。库强制执行质量门槛：

| 标准 | 是否必需 | 检查方式 |
|---|---|---|
| 完整 I/O 类型化 | 所有输入/输出均有类型 | 类型检查器 |
| 显式假设 | 至少列出一个假设 | Linter |
| 约束定义 | 至少有输出范围约束 | 验证器 |
| 验证规则 | 同时有前检和后检 | 测试框架 |
| 来源归属 | 主要方法有权威引用 | 审核者 |
| 策略映射 | 覆盖全部 6 种策略 | 注册表验证器 |
| 已知局限性 | 至少记录一个局限 | 审核者 |
| 版本历史 | 初始版本 + 当前版本 | Linter |

未通过任何**必需**标准的原语无法注册到活跃库中。这种严谨性正是将 TAE 原语库与随意的工具函数集合区分开来的关键。

---

## 17.5 原语注册表架构

### 17.5.1 注册表结构

领域原语库实现为**结构化注册表**——一个具有索引、搜索和依赖查询功能的原语规范数据库：

```
领域原语注册表
├── 核心原语（稳定、经过充分验证）
│   ├── 观测（25 个原语）
│   ├── 派生（48 个原语）
│   ├── 估计（22 个原语）
│   ├── 模拟（12 个原语）
│   ├── 验证（18 个原语）
│   └── 决策（15 个原语）
├── 扩展原语（领域特定）
│   ├── 信号配时（35 个原语）
│   ├── 快速路运营（28 个原语）
│   ├── 安全分析（19 个原语）
│   ├── 公交运营（14 个原语）
│   ├── 活跃交通（11 个原语）
│   └── 应急管理（8 个原语）
├── 实验原语（验证中）
│   └──（因版本而异）
└── 废弃原语（已归档，不可执行）
    └──（仅历史记录）
```

总计：当前注册表约有 **255 个原语**，覆盖交通工程实践的所有主要领域。

### 17.5.2 索引与发现

注册表支持多种访问模式：

| 访问模式 | 机制 | 用例 |
|---|---|---|
| **按名称** | `id` 上的哈希索引 | 已知原语时的直接查找 |
| **按类别** | `category` 上的 B 树索引 | 浏览所有派生原语 |
| **按策略** | `strategy_applicability` 上的倒排索引 | 查找设计策略所需的原语 |
| **按输入类型** | 输入类型上的倒排索引 | 查找消费 `Observation(流量)` 的原语 |
| **按输出类型** | 输出类型上的倒排索引 | 查找产生 `Derived(延误)` 的原语 |
| **按关键词** | 名称/描述上的全文检索 | 自然语言发现 |
| **按依赖** | 图遍历 | 查找某原语的所有祖先/后代 |
| **按来源** | `method.primary` 上的索引 | 查找所有基于 HCM 的原语 |

这些索引使 DRC 的依赖解析（第十六章）能够高效运行——即使有数百个原语，解析也在毫秒内完成。

### 17.5.3 版本控制与生命周期

原语遵循**语义版本号**方案，具有严格的生命周期阶段：

```
实验阶段 (0.x.x)
    │  ▼
提议阶段（待审核）
    │  ▼
活跃阶段 (1.x.x, 2.x.x, ...)
    │  ▼
废弃阶段（仍可用，发出警告）
    │  ▼
退役阶段（已归档，不再可执行）
```

阶段转换需要**正式审核**：

| 转换 | 触发条件 | 审批者 | 所需文档 |
|---|---|---|---|
| 实验 → 提议 | 内部测试完成 | 技术负责人 | 测试结果、性能数据 |
| 提议 → 活跃 | 外部审核通过 | 领域专家委员会 | 同行评审签署 |
| 活跃 → 废弃 | 被更优方法替代 | 注册表维护者 | 迁移指南、替代 ID |
| 废弃 → 退役 | 无依赖项残留 | 系统管理员 | 归档记录、历史注记 |

这种生命周期管理确保注册表保持**最新、正确、可信**——对安全关键型工程应用而言至关重要的属性。

---

## 17.6 原语组合模式

### 17.6.1 线性链

最简单的组合模式是线性链，其中每个原语的输出馈送给下一个：

```
RawDetectorData → obs:ExtractVolumes → drv:CalculateSaturationFlow → drv:CalculateCapacity → drv:CalculateXc → drv:CalculateDelay → drv:EvaluateLOS
```

线性链常见于**直截了当的分析任务**，其中一个明确定义的计算序列产生期望的结果。大多数 HCM 运营分析遵循此模式。

### 17.6.2 扇出（一对多）

单个原语的输出可能馈送给多个下游原语：

```
ApproachVolumes ─┬→ drv:CalculateSaturationFlow → drv:CalculateCapacity
                 ├→ est:EstimateTurningProportions → drv:CalculateLaneGroupVolumes
                 └→ val:CheckDataQuality → val:CheckTemporalConsistency
```

扇出发生在相同输入数据服务于多个分析目的时。DRC 通过一次计算共享输入并分发来优化扇出。

### 17.6.3 扇入（多对一）

多个原语的输出汇聚到单个下游原语：

```
ObservedDelays ──┐
SimulatedDelays ─┼→ val:CrossValidate → dec:ResolveDiscrepancy
HistoricalDelays ─┘
```

扇入代表**多源证据融合**——验证和决策的关键模式。

### 17.6.4 条件分支

基于运行时条件的不同执行路径：

```
IF data_quality == 'high':
    drv:CalculateSaturationFlow(observed_data)
ELSE IF data_quality == 'medium':
    est:EstimateSaturationFlow(observed_data + defaults)
ELSE:
    ref:GetDefaultSaturationFlow(area_type)
```

条件分支处理表征现实世界工程的**不完美信息**。DRC 尽可能在编译时解析分支条件；剩余的运行时分支由 TAR 处理。

### 17.6.5 迭代循环

某些计算需要迭代直至收敛：

```
LOOP:
    drv:AllocateGreenTimes(current_timing)
    val:CheckCycleTimeConstraints(result)
    IF constraint_violated:
        adjust_parameters()
        CONTINUE
    ELSE:
        BREAK
```

迭代循环被封装为**复合原语**——循环结构是内部的；外部呈现与其他任何原语相同的接口。这保持了 PDG 的无环特性，同时支持迭代算法。

---

## 17.7 动态原语图

### 17.7.1 静态注册表，动态图

一个关键的架构原则：**原语注册表是静态的，但原语图是动态的**。注册表包含所有可用原语及其规范。图则是针对每个特定工程责任按需构建的，仅选择和连接相关的原语。

考虑两个听起来相似但激活截然不同的图的请求：

| 请求 | 激活的原语 | 图规模 |
|---|---|---|
| "为什么交叉口 A 拥堵？" | 诊断导向：~18 个原语 | 中等 |
| "优化交叉口 A 的配时" | 设计导向：~24 个原语 | 大型 |
| "明年交叉口 A 还能正常运转吗？" | 预测导向：~15 个原语 | 中等 |
| "新的配时方案有效吗？" | 评估导向：~20 个原语 | 大型 |

同一交叉口，不同问题，不同图。这种动态性使得单一原语库能够在不产生组合爆炸的情况下服务于全谱系的工程任务。

### 17.7.2 图构建过程

图的构建过程（由 DRC 执行，见第十六章）分五步。每一步消费上一步的产出，并让图比它接手时更确定一些。

1. **判定策略。** 责任、态势与上下文共同确定正在被问的是哪一类工程问题。策略必须先定，因为它决定什么才算一个完整的回答。

2. **识别种子原语。** 责任陈述与注册表做匹配，得到直接回答该问题的原语——例如对一次设计请求，就是 `AllocateGreenTimes` 与 `EvaluateLOS`。

3. **展开依赖。** 从每个种子出发，顺着声明式需求向外找到能满足它们的原语，再继续向外，直到每一项需求要么由已提供的数据满足、要么被报告为缺失。正是这一步，把一份简短的回答清单展开成一条完整的推理链。

4. **插入验证。** 对策略要求必须验证的节点，在其下游挂上验证原语，使验证成为图的一部分，而不是某个人必须记得去做的步骤。

5. **优化。** 移除没有下游消费者的原语；把被多个消费者共用的计算合并成一次；识别出彼此独立的子图，使其可以并发执行。

产出的图是**最小的**（没有多余原语）、**完备的**（依赖全部满足）、**有效的**（通过类型与约束检查）、**可执行的**（TAR 可以直接运行）。


### 17.7.3 图的性质

每个生成的图都满足以下性质：

| 性质 | 定义 | 强制机制 |
|---|---|---|
| **无环性** | 无循环依赖（受控迭代除外） | 构建期间环检测 |
| **完备性** | 包含所有传递依赖 | BFS 扩展直至前沿为空 |
| **类型安全** | 所有连接满足类型兼容性 | 每条边插入时的类型检查 |
| **约束满足** | 所有声明的约束均可检验 | 构建期间的约束传播 |
| **策略对齐** | 每个原语适用于所选策略 | 每次包含时的策略过滤 |
| **可追溯性** | 每个节点都有来源元数据 | 自动附加元数据 |

这些性质是**不变量**——它们对 DRC 产生的每个图都成立，无论具体责任或领域如何。

---

## 17.8 原语与可解释性

### 17.8.1 固有可解释性

基于原语的架构最重要的属性之一是**固有可解释性**。因为推理链中的每一步对应一个命名的、文档化的、具有领域意义的原语，整个推理过程可以用工程术语来解释：

**不透明 AI 的解释：**
> "模型基于训练数据中的学习模式预测周期时长为 95 秒。"

**基于 TAE 原语的解释：**
> "推荐的 95 秒周期时长源于：
> 1. **饱和流率计算**（HCM 第十九章方法）：关键车道组 1850 veh/h/ln
> 2. **关键车道求和**：总关键流量 1,420 veh/h
> 3. **Webster 最优公式**：Co = (1.5L + 5) / (1 - Y) = 92.3s
> 4. **政策约束**：最小周期时长 = 90s（本地机构标准）
> 5. **取整**：最接近的 5 秒增量 = 95s
>
> 关键假设：大型车系数 = 2.0（HCM 默认值）；实际 HV% 未测量。"

差异显而易见。一种解释基于权威建立信任；另一种则邀请**基于理解的验证**。

### 17.8.2 解释生成

TAR 从执行的原语图中自动生成解释：

| 解释层级 | 内容 | 受众 |
|---|---|---|
| **摘要** | 高层结论 + 关键驱动因素 | 决策者 |
| **技术** | 逐步计算追踪 | 工程师 |
| **详细** | 包含所有中间值的完整 PDG | 审计员/审核者 |
| **调试** | 带时间戳和错误的执行日志 | 开发者 |

每个层级都可从同一底层执行机械导出——无需单独的解释模型。原语图**就是**解释。

---

## 17.9 维护与演进原语库

### 17.9.1 治理模型

原语库是一项**共享工程资产**，需要治理：

| 角色 | 职责 | 权限 |
|---|---|---|
| **领域专家** | 定义新原语、审核拟议变更 | 提议、批准内容 |
| **注册表维护者** | 管理版本、强制质量门槛 | 合并、拒绝提交 |
| **系统管理员** | 部署更新、管理基础设施 | 运维控制 |
| **最终用户（工程师）** | 报告问题、建议改进 | 反馈渠道 |
| **机构标准制定部门** | 设定政策约束、批准方法 | 监管权威 |

没有任何角色拥有不受制约的权限。变更需要相关利益相关者的共识。

### 17.9.2 变更管理流程

所有修改遵循结构化工作流：

```
提案 → 技术审查 → 领域审查 → 测试 → 预发布 → 部署 → 监控
   │           │              │          │           │            │
   ▼           ▼              ▼          ▼           ▼            ▼
 问题/功能    代码/规范       专家签署     自动化     金丝雀发布    全量上线     性能
   描述       质量门槛       正确性       测试套件                 到生产环境    指标
```

回滚始终可行——每次部署都有标签，可在几分钟内回退到前一版本。

### 17.9.3 社区贡献

虽然核心库由 TAE 团队维护，但**扩展原语**可由用户社区贡献：

1. 按规范模板开发原语
2. 连同测试用例提交至扩展注册表
3. 自动化质量检查验证格式合规性
4. 同行评审评估工程正确性
5. 审批后，原语进入实验阶段
6. 成功现场验证后，晋升至活跃阶段

这种社区模式使库能够**扩展至超出任何单一团队能够维护的规模**，同时通过严格的评审流程确保质量。

---

## 17.10 总结：原语作为工程 AI 的基础

领域原语库是所有其他 TAE 组件构建的基础。没有定义良好的原语，DRC 无物可编译；没有注册表，TAR 无物可执行；没有图结构，就没有可解释性。

本章的关键原则：

1. **原语是工程知识，而非代码。** 每个原语编码一个具有完整语义规范的方法。
2. **六个类别**（观测、派生、估计、模拟、验证、决策）覆盖所有工程活动。
3. **完整规范**包括 I/O 类型、假设、约束、方法、验证和策略适用性。
4. **注册表**组织约 255 个原语，具有丰富的索引用于高效查找和组合。
5. **五种组合模式**（线性、扇出、扇入、条件、迭代）表达所有推理结构。
6. **动态图构建**按任务选择相关原语——静态注册表，动态使用。
7. **固有可解释性**——原语图就是解释，无需单独模型。
8. **治理与演进**确保库保持最新、正确、社区增强。

原语库将交通工程从通过师徒传承的手艺转化为**系统的、可计算的、持续改进的工程知识体系**。这是工程 AI 必须建立的基础。

---

## 本章表格索引

| # | 表格名称 | 所在节 |
|---|---|---|
| 1 | 原语元组定义 | 17.2.1 |
| 2 | 原语 vs 软件函数 | 17.2.2 |
| 3 | 观测原语目录 | 17.3.1 |
| 4 | 派生原语目录 | 17.3.1 |
| 5 | 估计原语目录 | 17.3.1 |
| 6 | 模拟原语目录 | 17.3.1 |
| 7 | 验证原语目录 | 17.3.1 |
| 8 | 决策原语目录 | 17.3.1 |
| 9 | 规范质量标准 | 17.4.2 |
| 10 | 注册表结构概要 | 17.5.1 |
| 11 | 注册表访问模式 | 17.5.2 |
| 12 | 原语生命周期阶段 | 17.5.3 |
| 13 | 动态图示例 | 17.7.1 |
| 14 | 图的不变量属性 | 17.7.3 |
| 15 | 解释层级 | 17.8.2 |
| 16 | 治理角色 | 17.9.1 |

---
