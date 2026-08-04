---
name: prompt-engineering
description: Prompt Engineering引导Skill — 每次对话时检查用户prompt质量，提醒按要素写提示词。自动加载，pinned。Use when user sends any message to guide them write better prompts with role, context, task, constraints, output format.
---

# Prompt Engineering 引导

每次对话时，检查用户的提示词是否包含以下要素，缺少时主动提醒补充。

## Prompt七要素（检查清单）

| 要素 | 说明 | 示例 |
|------|------|------|
| **🎯 目标(Task)** | 你要我做什么？ | "帮我写一封邮件"、"分析这个数据" |
| **👤 角色(Role)** | 我应该扮演什么角色？ | "你是一个资深售前工程师"、"你是Python专家" |
| **📋 背景(Context)** | 相关背景信息 | "这是一个军队采购项目，预算4574万" |
| **📏 约束(Constraints)** | 限制条件 | "不超过500字"、"用中文"、"不要编造" |
| **📐 格式(Format)** | 输出格式要求 | "用表格"、"分点列出"、"写成Word文档" |
| **💡 示例(Example)** | 给一个参考样本 | "类似这样：..." |
| **🎭 风格(Tone)** | 语气和风格 | "正式"、"口语化"、"专业技术文档" |

## 自动检查规则

当用户发送消息时，按以下规则检查：

### 1. 缺少🎯目标 → 提醒
- 如果消息只是模糊描述（如"看看这个"、"处理一下"），提醒用户明确目标

### 2. 缺少👤角色 → 默认
- 如果没指定角色，根据上下文自动选择合适角色
- 不需要每次都提醒

### 3. 缺少📋背景 → 提醒（重要任务时）
- 复杂任务（文档分析、方案编写）缺少背景时提醒
- 简单任务（查个东西、执行命令）不需要

### 4. 缺少📏约束 → 提醒（输出类任务）
- 写文档、生成内容时提醒
- 查信息、执行命令时不需要

### 5. 缺少📐格式 → 提醒（输出类任务）
- 需要特定格式时提醒
- 自由对话时不需要

### 6. 缺少💡示例 → 建议
- 复杂任务时建议提供示例
- 简单任务不需要

### 7. 缺少🎭风格 → 默认
- 根据用户profile自动选择（中文、专业技术风格）
- 不需要每次都提醒

## 提醒模板

当检测到缺少关键要素时，用以下格式提醒：

```
💡 提示：你的prompt可以更完善：

✅ 已包含：目标、背景
❌ 缺少：
- 📐 格式：你希望输出什么格式？（表格/文档/列表）
- 📏 约束：有什么限制条件？

补充后我能更准确地帮你。
```

## 优先级

- **必须有**：🎯目标
- **强烈建议**：📋背景、📏约束
- **建议**：📐格式、🎭风格
- **可选**：👤角色、💡示例

## 自动优化

当用户的prompt质量较高时（包含4个以上要素），直接执行，不做提醒。

当用户的prompt质量中等（包含2-3个要素），快速提醒后执行。

当用户的prompt质量较低（只有1个要素），提醒补充后再执行。

## 特殊场景

### 文件处理
用户发送文件时，自动识别文件类型，不需要额外提醒格式。

### 连续对话
多轮对话中，后续消息不需要重复提醒已提供的要素。

### 紧急/简单任务
一句话能完成的任务（如"几点了"、"star这个项目"），不提醒。

## 第二章：结构化Prompt框架⭐ 第一轮学习新增

> 来源：GitHub调研 — RSTI/TCREI/TFCDC框架

### 2.1 RSTI框架（Role-Situation-Task-Intent）

| 要素 | 说明 | 示例 |
|------|------|------|
| **R - Role** | 角色 | "你是一个资深售前工程师" |
| **S - Situation** | 情境 | "客户是东北大学IT负责人" |
| **T - Task** | 任务 | "准备拜访材料" |
| **I - Intent** | 意图 | "了解需求，寻找合作机会" |

### 2.2 TCREI框架（Task-Context-Role-Example-Instruction）

适用于复杂任务，比RSTI更详细。

### 2.3 TFCDC框架（Task-Format-Context-Details-Constraints）

适用于需要严格格式的输出任务。

### 2.4 与七要素的关系

| 七要素 | RSTI | TCREI | TFCDC |
|--------|------|-------|-------|
| 🎯目标 | T | T | T |
| 👤角色 | R | R | - |
| 📋背景 | S | C | C |
| 📏约束 | - | - | C |
| 📐格式 | - | - | F |
| 💡示例 | - | E | - |
| 🎭风格 | I | I | D |

**使用建议：** 简单任务用七要素，复杂任务用TCREI，格式严格用TFCDC。

## 第四章：DSPy 可编程Prompt优化⭐ 第三轮研究新增

> 来源：DSPy深度研究 — Signature/BootstrapFewShot/MIPRO/集成方案

### 4.1 核心理念：从"写prompt"到"编译prompt"

传统方式：人写prompt → 试错 → 手工调优
DSPy方式：声明Signature → 定义metric → 编译器自动优化prompt

**DSPy三件套**：
| 组件 | 作用 | 对应我们的skill |
|------|------|-----------------|
| Signature | 声明式任务定义（输入→输出） | 七要素的结构化版本 |
| Module | 执行策略（Predict/ChainOfThought/ReAct） | 框架选择（RSTI/TCREI） |
| Optimizer | 自动优化prompt | 自动补全建议的进阶版 |

### 4.2 Signature = 结化Prompt定义

```python
# 最简：内联签名
qa = dspy.Predict("question -> answer")

# 完整：类签名（推荐复杂任务）
class Summarize(dspy.Signature):
    """Summarize text into key points."""  # docstring = 🎯目标
    text = dspy.InputField(desc="full article")  # 📋背景
    summary = dspy.OutputField(desc="3-5 bullets")  # 📐格式
```

**与七要素的映射**：
| 七要素 | Signature对应 | 说明 |
|--------|-------------|------|
| 🎯目标 | docstring | Signature的三引号字符串 |
| 👤角色 | （由Module隐含） | ChainOfThought自带推理角色 |
| 📋背景 | InputField | 输入字段 + desc |
| 📏约束 | OutputField(desc=) | 在desc里写约束 |
| 📐格式 | OutputField(desc=) | "bullet points, 3-5 items" |
| 💡示例 | BootstrapFewShot自动注入 | 不用手写！ |
| 🎭风格 | instruction prefix | MIPRO自动优化 |

### 4.3 自动优化实践指南

**BootstrapFewShot** — 最常用，10-50条数据即可：
```python
from dspy.teleprompt import BootstrapFewShot
optimizer = BootstrapFewShot(metric=my_metric, max_bootstrapped_demos=3)
optimized = optimizer.compile(module, trainset=trainset)
```
原理：从训练数据中筛选高质量预测，自动作为few-shot示例。

**MIPROv2 / GEPA** — SOTA，50-200条数据，10-30分钟：
```python
# 推荐：GEPA（最新，用反思改进instruction）
from dspy.teleprompt import GEPA
optimizer = dspy.GEPA(metric=my_metric, reflection_lm=strong_lm, auto="light")
optimized = optimizer.compile(module, trainset=trainset, valset=valset)
# Metric返回 dspy.Prediction(score=X, feedback="原因") → 反馈驱动instruction改进

# 备选：MIPROv2（贝叶斯优化）
from dspy.teleprompt import MIPROv2
optimizer = MIPROv2(metric=my_metric, num_candidates=10)
optimized = optimizer.compile(module, trainset=trainset, valset=valset, num_trials=100)
```
原理：GEPA用反思+文本反馈自动改写instruction；MIPROv2用贝叶斯搜索。

**Metric定义** — 优化的核心：
```python
def my_metric(example, pred, trace=None):
    # 精确匹配
    return example.answer.lower() == pred.answer.lower()
    # 或F1 / 包含匹配 / 语义相似度
```

### 4.4 Skill增强建议

对于我们的prompt-engineering skill，DSPy启发的改进方向：

1. **Signature化**：把七要素框架变成可声明的结构，而不是自然语言提醒
2. **Metric化**：定义prompt质量评分函数，自动量化而不是主观判断
3. **Bootstrap化**：积累好的prompt-response对，自动作为few-shot示例
4. **MIPRO化**：对关键skill的instruction做A/B测试，数据驱动选最优

**当前可落地的一步**：把七要素检查从"提醒补充"升级为"结构化评分"：
```python
def prompt_quality_score(prompt: str) -> float:
    """七要素自动评分，0-10分"""
    score = 0
    score += 2 if has_clear_task(prompt) else 0      # 🎯目标
    score += 2 if has_context(prompt) else 0          # 📋背景
    score += 2 if has_constraints(prompt) else 0      # 📏约束
    score += 2 if has_format(prompt) else 0           # 📐格式
    score += 1 if has_example(prompt) else 0          # 💡示例
    score += 1 if has_role(prompt) else 0             # 👤角色
    return score / 10.0
```

### 4.5 推荐优化流程

```
Baseline（手工prompt）
  ↓ 定义Signature + metric
BootstrapFewShot（自动few-shot，10+数据）
  ↓ 数据量够时升级
MIPRO（自动instruction，50+数据）
  ↓ 持久化
Save/Load（.json序列化，跨环境复用）
```

**Pitfalls**：
- 数据量不足时不要用MIPRO，BootstrapFewShot就够
- metric必须匹配任务，太严格会导致优化失败
- max_bootstrapped_demos不要超过5，会过拟合
- 保存优化结果，避免重复计算

## Present Results to User

提醒时保持简洁，不要长篇大论。用表格或列表，一目了然。

## 第三章：增强检测与自动优化⭐ 第二轮研究新增

> 来源：GitHub调研 — DSPy/PEST/promptfoo/Outlines/Guidance

### 3.1 增强缺失要素检测规则

在原有七要素检查基础上，增加**负面模式匹配**和**任务类型判断**：

#### 负面模式检测（模糊词）
| 要素 | 负面模式（触发提醒） |
|------|---------------------|
| 🎯目标 | "看看这个"、"处理一下"、"帮我"、"做一下"、"弄一下" |
| 📏约束 | "不限"、"随便"、"自由发挥"、"你觉得呢" |
| 📐格式 | "随便什么格式"、"都行" |
| 📋背景 | prompt < 20字 且 无文件附件 |

#### 任务类型判断
```
简单任务（不提醒）: "几点了"、"star这个项目"、"删掉文件"
复杂任务（提醒背景）: 文档分析、方案编写、数据分析
输出类任务（提醒约束+格式）: 写文档、生成内容、写代码
```

### 3.2 自动补全建议模板

检测到缺失要素时，生成**具体可点击的补全建议**：

```
💡 Prompt质量: 6/10

✅ 已包含: 目标(写方案)、背景(军队采购)
❌ 缺少:
- 📏 约束: 建议添加 → "不超过50页"、"用专业技术语言"、"包含预算明细"
- 📐 格式: 建议指定 → "Word文档"、"带目录和页码"、"分章节"

💡 快速补全: 在你的prompt末尾加上：
  "格式要求：Word文档，带目录，不超过50页。风格：正式技术文档。"
```

### 3.3 框架选择指南

| 场景 | 推荐框架 | 说明 |
|------|----------|------|
| 简单任务 | 七要素 | 轻量够用，1分钟搞定 |
| 复杂任务 | TCREI / DSPy Signature | 要素齐全，可自动优化 |
| 格式严格 | TFCDC / Outlines约束 | 保证输出格式100%合法 |
| 程序化场景 | DSPy Signature | 声明式定义，自动优化prompt |
| 数据驱动优化 | DSPy + BootstrapFewShot | 有10+好例→自动few-shot |
| 指令自动优化 | DSPy + GEPA | 有50+数据→反思驱动instruction改进 |
| 中文场景 | RSTI + 七要素 | 直觉友好，角色-情境-任务-意图 |

### 3.4 相关工具（供高级用户参考）

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| **DSPy** (22k★) | 自动prompt优化、BootstrapFewShot/MIPRO | 需要数据驱动优化prompt |
| **promptfoo** (20k★) | Prompt测试/评估/比较/回归测试 | 建立prompt测试体系 |
| **Outlines** (8k★) | FSM约束生成，保证输出格式合法 | 需要严格结构化输出 |
| **Guidance** (18k★) | 正则/语法约束，token healing | 复杂约束生成场景 |
| **PEST** (新) | 配置驱动的prompt评估评分工具包 | 自动化prompt质量监控 |

## Pitfalls

1. **不要过度提醒** — 简单任务不提醒，避免用户反感
2. **不要阻塞执行** — 提醒后仍然执行，不要等用户补充所有要素
3. **根据上下文推断** — 很多要素可以从对话历史中推断，不需要每次都问
4. **用户说"直接做"时停止提醒** — 尊重用户意愿
5. **负面模式不要误判** — "帮我看看这个文件"是明确目标，不是模糊请求
6. **任务类型判断要结合上下文** — 同一句话在不同上下文中可能是简单或复杂任务
7. **自动补全建议要具体** — 不要说"建议添加约束"，要说"建议添加：不超过50页、用中文"

## 第五章：Prompt链式调用（Prompt Chaining）⭐ 第四轮研究新增

> 来源：Anthropic官方文档、LangChain社区、OpenAI Cookbook、实战经验
> 完整研究报告：references/prompt-chaining-research.md

### 5.1 核心概念

**Prompt Chaining** = 将复杂任务分解为多个顺序执行的子任务，每步由独立prompt处理，前一步输出作为后一步输入。

```
复杂任务 → [Step1: 分析] → [Step2: 规划] → [Step3: 执行] → [Step4: 验证] → 最终输出
```

**核心思想**：与其让一个prompt做所有事，不如让每个prompt做好一件事。

### 5.2 分解原则（MISO原则）

| 原则 | 说明 |
|------|------|
| **M - Measurable** | 每步输出有明确判断标准（JSON/列表） |
| **I - Independent** | 每步可独立验证，不依赖隐含假设 |
| **S - Small** | 每步只做一件事，prompt < 500 tokens |
| **O - Observable** | 中间结果可被人类检查和调试 |

**经验法则**：每增加一步，端到端成功率下降5-10%。5步链理论成功率 ≈ 77%。建议链长3-5步，超过8步用并行子链。

### 5.3 输出格式设计

**黄金法则**：链中每步输出必须是机器可解析的，不能是自由文本。

| 格式 | 适用场景 | 优势 |
|------|---------|------|
| JSON | 纯自动化流水线 | 最易程序化解析 |
| XML标签 | 含大量自然语言 | 结构化+可读性（Anthropic推荐） |
| Markdown | 人机协作调试 | 人类友好 |

**JSON输出模板**（每步标准字段）：

```json
{
  "step": "step_name",           // 步骤标识
  "status": "success|partial|error",  // 执行状态
  "data": { ... },               // 业务数据
  "confidence": 0.92,            // 置信度 0-1
  "metadata": { "tokens_used": N, "timestamp": "..." }
}
```

**强制格式的方法**：Instructor + Pydantic（OpenAI/Claude）、Tool Use schema、Outlines FSM约束。

### 5.4 错误传播与恢复

#### 错误类型

| 类型 | 频率 | 对策 |
|------|------|------|
| 格式错误 | 高 | 用Instructor/Tool Use强制 + 即时重试 |
| 内容错误 | 中 | 置信度阈值 + 冗余验证 |
| 遗漏错误 | 中 | 检查清单验证覆盖度 |
| 超限错误 | 中 | 控制输出长度 + 分段处理 |

#### 恢复策略优先级

1. **即时重试**（格式错误）：将错误信息反馈给LLM，让它修正。第1次重试修复率60-70%
2. **回退安全输出**（内容错误）：使用模板化/保守的默认输出
3. **旁路跳过**（非关键步骤）：用默认值替代，标记警告
4. **人工介入**（自动恢复失败）：请求人类检查和决策

#### 质量门控（每步后执行）

```
Step输出 → 格式合法？→ 必填字段完整？→ 置信度>0.7？→ 逻辑一致？→ 传给下一步
             ↓ 不通过      ↓ 不通过        ↓ 不通过      ↓ 不通过
           重试格式      重试补全       标记人工审核    重试修正
```

### 5.5 Anthropic官方Chain模式

#### 5种链式模式

| 模式 | 结构 | 适用场景 |
|------|------|---------|
| **Sequential** | A→B→C→D | 步骤有严格依赖 |
| **Parallel** | A→[B1,B2,B3]→C | 多个独立子任务 |
| **Conditional** | A→if X then B else C | 需要动态路由 |
| **Iterative** | A→B→check→B'→check→done | 需要自我改进 |
| **Hierarchical** | Master→[Sub1,Sub2]→Merge | 超复杂分治任务 |

#### Anthropic推荐的实现方式

1. **预填充引导**：Assistant消息预填`{`强制JSON输出
2. **XML标签上下文**：用`<step_history>` `<current_step>`组织链上下文
3. **Tool Use链式调用**：定义工具集，Claude自动编排调用顺序（最灵活）
4. **分段处理链**：分段→摘要→合并，突破上下文窗口

#### Tool Use链式调用示例

```python
# 定义工具 = 链中每步能力
tools = [
    {"name": "analyze_requirements", ...},
    {"name": "match_products", ...},
    {"name": "draft_solution", ...}
]
# Claude自动决定调用顺序，实现链式编排
response = client.messages.create(model="claude-sonnet-4-20250514", tools=tools, messages=[...])
```

### 5.6 关键Pitfalls

1. **链太长（>8步）**：成功率指数下降 → 拆分为并行子链
2. **输出格式不固定**：下一步解析失败 → 用Instructor/Tool Use强制
3. **上下文无限膨胀**：token超限 → 每步只传必要信息，压缩历史
4. **没有验证层**：错误静默传播 → 每步后立即验证
5. **重试没有变化**：同样错误重复 → 重试时注入错误信息
6. **忽略置信度**：低质量输出混入 → 设阈值触发人工审核
7. **单点故障**：一个步骤失败全链终止 → 设计fallback和bypass
8. **调试困难**：不知道哪步出错 → 保存每步完整输入输出到日志
