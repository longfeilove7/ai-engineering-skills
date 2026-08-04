---
name: context-engineering
description: "上下文工程：管理AI对话中的上下文窗口、信息压缩、多轮对话维护、记忆管理和RAG检索策略。Use when managing long conversations, context overflow, memory persistence, or building RAG systems."
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [context, memory, rag, conversation, compression, retrieval]
    related_skills: [prompt-engineering, planning, llm-wiki]
---

# Context Engineering 上下文工程

> **核心思想**：上下文是AI系统最重要的"带宽"——如何高效利用有限的上下文窗口，决定了AI系统的能力上限。
>
> 参考来源：Stackwich Context层定义（分叉研究和读取密集型扫描，内存/计划/任务各司其职）、DAIR-AI Prompt Engineering Guide

---

## 一、上下文窗口管理

### 1.1 窗口容量认知

| 模型 | 上下文窗口 | 约等于 | 适用场景 |
|------|-----------|--------|---------|
| GPT-4o | 128K tokens | ~300页文档 | 长文档分析、代码库理解 |
| Claude 3.5 | 200K tokens | ~500页文档 | 大规模文档处理 |
| Gemini 1.5 | 1M+ tokens | ~2000页文档 | 超长文档、视频理解 |
| 本地模型（7B-70B） | 4K-32K tokens | 10-80页 | 简单对话、代码补全 |

**关键规则**：
- 永远不要等到上下文溢出才处理——提前规划
- 上下文越长，推理质量越低（"大海捞针"问题）
- 最重要的信息放在上下文的**开头和结尾**（首因效应+近因效应）

### 1.2 窗口分配策略

将上下文窗口想象为一个**预算表**，按比例分配：

```
┌─────────────────────────────────────────┐
│  系统指令 (System Prompt)     5-10%     │  ← 固定，不变
├─────────────────────────────────────────┤
│  核心上下文 (Key Context)     15-25%    │  ← 最高优先级
├─────────────────────────────────────────┤
│  工作记忆 (Working Memory)    30-40%    │  ← 当前任务相关信息
├─────────────────────────────────────────┤
│  对话历史 (History)           20-30%    │  ← 按相关性压缩
├─────────────────────────────────────────┤
│  预留空间 (Reserve)           10-15%    │  ← 模型输出 + 工具调用
└─────────────────────────────────────────┘
```

### 1.3 上下文优先级排序

当空间不足时，按以下优先级保留内容：

1. **P0 - 必须保留**：系统指令、当前任务目标、关键约束
2. **P1 - 强烈保留**：当前子任务上下文、关键决策记录、最近3轮对话
3. **P2 - 可压缩**：早期对话摘要、参考资料摘要、工具调用结果
4. **P3 - 可丢弃**：已完成的子任务详情、冗余的工具输出、过时的中间结果

### 1.4 实操：Hermes Agent中的窗口管理

```python
# 检查当前上下文使用情况
# 在Hermes对话中，通过以下方式管理：

# 1. 使用 session_search 回顾历史，而不是把所有内容都放在当前对话中
session_search(query="项目需求", limit=3)

# 2. 使用 memory_save 保存关键信息，从上下文中释放
memory_save(content="用户是东华软件售前工程师，负责智慧城市建设方案", type="fact")

# 3. 长文档处理：分段读取而不是一次性加载
read_file(path="large_doc.txt", offset=1, limit=100)  # 先读前100行
read_file(path="large_doc.txt", offset=101, limit=100)  # 再读下100行
```

---

## 二、信息压缩策略

### 2.1 压缩的必要性

当对话超过**20轮**或上下文使用超过**60%**时，必须主动压缩。

### 2.2 五种压缩技术

#### 技术1：摘要压缩（Summarization）

将长段对话压缩为结构化摘要：

```
原文（500 tokens）：
  用户讨论了智慧城市项目的3个子系统需求，包括交通管理、
  环境监测、公共安全。每个子系统都有详细的技术要求...
  （省略大量细节）

压缩后（80 tokens）：
  ## 智慧城市项目摘要
  - 3个子系统：交通管理、环境监测、公共安全
  - 关键决策：采用微服务架构，数据中台统一管理
  - 待定：预算审批（预计下周确认）
  - 下一步：编写技术方案初稿
```

#### 技术2：符号化压缩（Symbolic Compression）

用符号和缩写替代冗长描述：

| 原始表达 | 压缩表达 |
|---------|---------|
| "该项目采用分布式微服务架构" | "架构=微服务/分布式" |
| "数据库使用MySQL 8.0作为主数据库" | "DB=MySQL8.0" |
| "前后端分离，前端Vue，后端Spring Boot" | "FE=Vue, BE=SpringBoot" |

#### 技术3：结构化提取（Structured Extraction）

从非结构化对话中提取结构化数据：

```markdown
## 项目上下文卡片
| 字段 | 值 |
|------|-----|
| 项目名称 | 智慧城市 |
| 客户 | XX市政府 |
| 预算 | 4574万 |
| 技术栈 | 微服务+数据中台 |
| 状态 | 方案阶段 |
| 截止日期 | 2026-09-01 |
```

#### 技术4：分层压缩（Hierarchical Compression）

不同层级保留不同粒度：

```
Level 0（详细）：最近2轮对话 — 完整保留
Level 1（要点）：最近3-10轮 — 保留关键决策和结论
Level 2（摘要）：10轮以前 — 仅保留一句话摘要
Level 3（索引）：20轮以前 — 仅保留主题标签
```

#### 技术5：工具输出压缩

工具调用的输出往往占据大量上下文，需要智能压缩：

```
# 原始工具输出（可能上千行）
terminal: find / -name "*.py" | head -100
# ... 100行文件列表 ...

# 压缩后
terminal结果摘要：找到Python文件97个，主要分布在src/(45), tests/(32),
utils/(20)。完整列表已保存至 /tmp/py_files.txt。
```

### 2.3 自动压缩触发条件

| 条件 | 动作 |
|------|------|
| 上下文使用 > 60% | 对历史对话执行摘要压缩 |
| 上下文使用 > 80% | 激进压缩，仅保留P0+P1内容 |
| 单个工具输出 > 2000 tokens | 自动截断 + 摘要 |
| 对话轮次 > 30 | 用memory_save持久化关键信息 |

---

## 三、多轮对话上下文维护

### 3.1 Stackwich Context层核心原则

> "分叉研究和读取密集型扫描，内存/计划/任务各司其职"

这意味着：
- **内存（Memory）**：长期知识，跨会话持久化 → `memory_save` / `memory_recall`
- **计划（Plan）**：当前任务的结构化计划 → `todo` / `.hermes/plans/`
- **任务（Task）**：当前正在执行的具体步骤 → 工作记忆（对话上下文中）

三者**不要混在一起**——计划放在todo里，记忆存在memory里，当前任务在对话中。

### 3.2 对话状态机

将多轮对话建模为状态机：

```
                    ┌──────────┐
          ┌────────│  IDLE    │────────┐
          │        └──────────┘        │
          ▼                            ▼
    ┌──────────┐                 ┌──────────┐
    │GATHERING │ ──信息充足──→  │ PLANNING │
    │  INFO    │                 │          │
    └──────────┘                 └────┬─────┘
          ▲                           │
          │                           ▼
     不足│                    ┌──────────┐
          │                    │EXECUTING│
          └───── 需补充 ──────│          │
                              └────┬─────┘
                                   │
                                   ▼
                              ┌──────────┐
                              │ REVIEWING│
                              └──────────┘
```

每个状态对应不同的上下文需求：

| 状态 | 需要的上下文 | 不需要的上下文 |
|------|------------|--------------|
| GATHERING | 历史需求、用户偏好、约束条件 | 工具输出、中间结果 |
| PLANNING | 完整需求、技术约束、资源限制 | 调研细节、原始数据 |
| EXECUTING | 当前任务步骤、相关代码、工具输出 | 前期调研、已完任务 |
| REVIEWING | 最终结果、验收标准、原始需求 | 中间过程、调试信息 |

### 3.3 对话轮次管理策略

```
轮次 1-5：   需求收集阶段
             → 完整保留所有对话
             → 关注用户意图、约束条件

轮次 6-15：  方案制定阶段
             → 压缩早期需求对话为摘要
             → 保留关键决策点
             → 使用todo记录计划

轮次 16-30： 执行阶段
             → 将方案保存为计划文件
             → 仅保留当前执行步骤上下文
             → 工具输出即时压缩

轮次 30+：   持续工作阶段
             → 执行memory_save保存关键结论
             → 每10轮做一次上下文清理
             → 考虑开新会话继续
```

### 3.4 上下文切换技巧

当需要切换话题或子任务时：

```markdown
## 上下文切换协议

1. **保存当前状态**
   memory_save(content="正在处理XX任务，进度60%，关键发现：...", type="workflow")

2. **明确切换声明**
   "现在切换到YY任务，之前的XX任务进度已保存。"

3. **加载新上下文**
   memory_recall(query="YY任务相关背景")

4. **切换回来时恢复**
   memory_recall(query="XX任务进度")
```

---

## 四、记忆管理

### 4.1 记忆分类体系

| 记忆类型 | 说明 | 存储位置 | 生命周期 |
|---------|------|---------|---------|
| **工作记忆** | 当前对话中的信息 | 对话上下文 | 会话级 |
| **短期记忆** | 最近几次会话的关键信息 | session_search | 天级 |
| **长期记忆** | 用户偏好、项目信息、决策记录 | memory_save | 永久 |
| **外部记忆** | 文档、知识库、代码库 | 文件系统/RAG | 永久 |

### 4.2 记忆存储决策树

```
信息来了 → 这个信息重要吗？
              │
    ┌─────────┴─────────┐
    是                   否
    │                    └→ 仅保留在工作记忆中
    ▼
  需要跨会话保留吗？
    │
  ┌─┴─┐
  是   否
  │    └→ session_search 可以找到吗？
  ▼       │
memory_save ┌─┴─┐
(type=*)  是   否
           │    └→ 写入文件或memory_save
           ▼
        无需额外存储
```

### 4.3 记忆存储最佳实践

```python
# ✅ 好的记忆存储 — 具体、可检索、有上下文
memory_save(
    content="东华软件智慧城市项目：客户XX市政府，预算4574万，技术栈微服务+数据中台，2026年9月交付",
    type="fact",
    concepts="智慧城市,东华软件,售前",
    files="proposal.docx"
)

# ❌ 差的记忆存储 — 模糊、无法检索
memory_save(
    content="用户说了一个项目",
    type="fact"
)
```

### 4.4 记忆检索策略

```python
# 场景1：回忆用户偏好
memory_recall(query="用户写作风格偏好")

# 场景2：回忆项目上下文
memory_recall(query="智慧城市项目 需求 决策")

# 场景3：回忆技术方案
memory_recall(query="微服务架构 数据中台 技术选型")

# 场景4：智能搜索（混合语义+关键词）
memory_smart_search(query="上次讨论的预算问题")
```

### 4.5 记忆维护

定期清理和更新记忆：

```python
# 查看所有记忆
memory_export()

# 删除过时记忆
memory_governance_delete(
    memoryIds="mem_abc123,mem_def456",
    reason="项目已完成，不再需要中间状态信息"
)

# 更新已有记忆（用新内容覆盖旧的）
memory_save(
    content="智慧城市项目状态：已从方案阶段进入投标阶段，预算调整为4200万",
    type="fact",
    concepts="智慧城市,项目状态"
)
```

---

## 五、RAG检索策略

### 5.1 RAG架构概述

```
用户查询
    │
    ▼
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  查询理解    │ ──→ │  检索策略    │ ──→ │  上下文组装  │
│ Query        │     │ Retrieval    │     │ Context      │
│ Understanding│     │ Strategy     │     │ Assembly     │
└──────────────┘     └──────────────┘     └──────────────┘
                           │                     │
                           ▼                     ▼
                     ┌──────────────┐     ┌──────────────┐
                     │  知识库      │     │  LLM生成     │
                     │ Knowledge    │     │ Generation   │
                     │ Base         │     │              │
                     └──────────────┘     └──────────────┘
```

### 5.2 查询改写（Query Rewriting）

用户查询往往不适合直接检索，需要改写：

| 原始查询 | 改写后查询 | 改写策略 |
|---------|-----------|---------|
| "这个项目怎么搞" | "智慧城市项目技术方案 架构设计" | 扩展上下文+具体化 |
| "之前说的那个方案" | "智慧城市建设方案 技术架构 预算" | 恢复隐含引用 |
| "帮我写个文档" | "售前技术方案文档 智慧城市 微服务" | 补充约束条件 |

### 5.3 检索策略选择

| 策略 | 适用场景 | 实现方式 |
|------|---------|---------|
| **精确检索** | 找特定文件、特定信息 | 关键词匹配、文件名搜索 |
| **语义检索** | 找相关概念、类似问题 | embedding相似度 |
| **混合检索** | 通用场景 | BM25 + embedding加权 |
| **层级检索** | 大规模知识库 | 先粗筛再精排 |
| **时间衰减检索** | 信息有时效性 | 近期文档权重更高 |

### 5.4 Hermes Agent中的RAG实践

```python
# === 本地文件检索 ===

# 1. 精确搜索文件内容
search_files(pattern="智慧城市.*预算", target="content", file_glob="*.md")

# 2. 查找相关文件
search_files(pattern="*方案*", target="files")

# 3. 读取并处理大文档（分块）
content = read_file(path="招标文件.docx", offset=1, limit=200)
# ... 处理 ...
content = read_file(path="招标文件.docx", offset=201, limit=200)

# === 历史对话检索 ===

# 4. 搜索历史会话中的相关信息
session_search(query="智慧城市技术方案", limit=5)

# 5. 检索长期记忆
memory_smart_search(query="售前方案写作规范")

# === Web检索 ===

# 6. 搜索外部知识
# web_search(query="智慧城市 最新技术趋势 2026")
```

### 5.5 上下文组装策略

检索到的内容需要智能组装进上下文：

```
┌─────────────────────────────────────────┐
│  查询：写一份智慧城市技术方案            │
├─────────────────────────────────────────┤
│  检索结果组装顺序：                      │
│                                         │
│  1. 用户最近的指令和偏好 (500 tokens)    │  ← 最高优先级
│  2. 项目基础信息 (300 tokens)            │
│  3. 招标文件要求摘要 (800 tokens)        │
│  4. 参考方案要点 (600 tokens)            │
│  5. 技术栈和约束 (400 tokens)            │
│                                         │
│  总计：~2600 tokens（上下文预算的15-20%）│
└─────────────────────────────────────────┘
```

### 5.6 RAG质量优化

| 问题 | 原因 | 解决方案 |
|------|------|---------|
| 检索结果不相关 | 查询太模糊 | 查询改写+扩展 |
| 遗漏关键信息 | 分块不当 | 重叠分块+多粒度检索 |
| 上下文超限 | 检索太多 | 设置token上限+重排序 |
| 回答不准确 | 检索噪声大 | 提高相似度阈值+过滤 |
| 时效性差 | 文档过时 | 时间戳过滤+定期更新 |

---

## 六、综合实践：售前工程师场景

### 6.1 典型工作流

```
1. 新项目启动
   → memory_recall(客户公司信息)
   → session_search(类似项目历史)
   → todo(创建项目计划)

2. 需求分析阶段
   → read_file(招标文件, 分段读取)
   → memory_save(关键需求点)
   → 压缩原始文档为结构化摘要

3. 方案编写阶段
   → todo(跟踪写作进度)
   → memory_recall(技术方案模板)
   → search_files(历史方案参考)
   → 分章节编写，每章完成后压缩上下文

4. 投标阶段
   → memory_save(最终方案要点)
   → 压缩所有中间过程
   → 仅保留最终交付物上下文
```

### 6.2 上下文检查清单

在每次重要对话前，检查：

- [ ] 是否有明确的任务目标？
- [ ] 是否检索了相关的历史信息？
- [ ] 是否保存了上一个任务的关键结论？
- [ ] 当前上下文使用是否在合理范围（<60%）？
- [ ] 是否为输出预留了足够的空间？

---

## 八、Progressive Disclosure（渐进式披露）⭐ 第一轮学习新增

> 来源：ratel-ai/ratel (406⭐) — 2026年最新Context Engineering实践

**核心理念：按需加载信息，而不是一次性加载所有上下文。减少80% token使用。**

### 8.1 三层披露策略

| 层 | 内容 | 何时加载 |
|---|------|---------|
| **L1 摘要** | 一句话概述 | 每次都加载 |
| **L2 结构** | 目录/索引 | 需要定位时 |
| **L3 详情** | 完整内容 | 确认需要时 |

### 8.2 实现方式

```
用户问："帮我看看这个项目的架构"
    ↓
L1: "这是一个5层架构的AI Engineering Skill体系"
    ↓ 用户说"详细说说"
L2: "包含Prompt/Context/Harness/Loop/Graph五层"
    ↓ 用户说"Context层具体是什么"
L3: 加载context-engineering的完整SKILL.md
```

### 8.3 在Hermes中的实践

```python
# L1: 用session_search获取摘要
session_search(query="项目架构", limit=3)

# L2: 用search_files获取结构
search_files(pattern="*.md", target="files")

# L3: 用read_file获取详情
read_file(path="具体文件", offset=1, limit=100)
```

## 九、Context Compaction Hook（上下文压缩钩子）⭐ 第一轮学习新增

> 来源：deepset-ai/haystack (26.1K⭐) — 最新Context Compaction Hook

**核心理念：在迭代中自动压缩上下文，防止窗口溢出。**

### 9.1 触发条件

| 条件 | 动作 |
|------|------|
| context使用量 > 70% | 轻度压缩（摘要旧对话） |
| context使用量 > 85% | 中度压缩（删除工具输出） |
| context使用量 > 95% | 重度压缩（只保留关键信息） |

### 9.2 压缩策略

1. **对话历史** — 保留最近5轮，旧的摘要化
2. **工具输出** — 只保留结论，删除原始输出
3. **文件内容** — 只保留相关段落
4. **记忆** — 保留，不压缩

## 十、Adaptive Tool Ranking（自适应工具排名）⭐ 第一轮学习新增

> 来源：ratel-ai/ratel — 进程内BM25检索 + 自适应工具排名

**核心理念：根据任务类型自动排序工具优先级。**

| 任务类型 | 优先工具 |
|---------|---------|
| 文件操作 | read_file > write_file > terminal |
| 代码分析 | search_files > read_file > terminal |
| 网络研究 | web_search > web_extract > browser |
| 文档处理 | docx > pdf > ocr |

## 十一、底层Attention优化技术 ⭐ 第五轮深度研究

> 来源：2024-2025年前沿研究综述，详见 `~/context-engineering-deep-research.md`

### 11.1 KV Cache 优化技术栈

| 技术 | 原理 | 效果 | 代表工作 |
|------|------|------|---------|
| GQA/MQA | 多Query头共享KV头 | 4-8x KV减少 | LLaMA 3, Mistral |
| PagedAttention | 虚拟内存分页管理KV | 消除碎片，3-4x吞吐 | vLLM |
| KV量化 | FP16→INT8/INT4 | 2-4x内存节省 | KIVI, KVQuant |
| Token驱逐 | 淘汰低重要性token的KV | 动态内存 | H2O, StreamingLLM |
| Prefix Caching | 共享前缀复用KV | 多请求共享 | SGLang, vLLM |
| MLA | 低秩latent压缩KV | ~1/10 KV大小 | DeepSeek-V2/V3 |

### 11.2 Attention Sinks + StreamingLLM

**发现**：LLM初始token（前4个）的attention权重异常高，充当"注意力垃圾桶"。

**StreamingLLM方案**：`[Sink Tokens (前4)] + [Sliding Window (最近N个)]`
- 固定内存，支持无限流式推理
- 局限：无法回忆窗口外内容

### 11.3 Infini-Attention（Google 2024）

**核心**：标准attention + 压缩记忆矩阵，支持无限上下文。
- 分段处理，历史段压缩为固定大小记忆矩阵M
- 记忆更新：`M_s = M_{s-1} + σ(K_s)^T · V_s`
- 门控融合：`output = gate · A_mem + (1-gate) · A_local`
- 1B模型在1M序列上验证有效

### 11.4 Ring Attention（Berkeley 2024）

**核心**：多GPU环形通信分担长序列attention计算。
- 序列分块→各设备并行计算→环形传递KV块→在线softmax归全
- 上下文长度与GPU数量线性扩展
- Striped Attention改进：条纹分配消除负载不均，快1.5x

### 11.5 Differential Transformer（Microsoft 2024）

```
DiffAttn = softmax(Q₁K₁ᵀ)·V - λ·softmax(Q₂K₂ᵀ)·V
```
- 两个attention map做差消除噪声
- 天然减少attention sinks，更好利用上下文中间部分

### 11.6 位置编码外推演进

Linear Scaling → NTK-aware → **YaRN**（分频段混合插值） → **ABF**（调整base频率）
- LLaMA 3.1用YaRN从8K→128K
- Gemini用ABF支持10M上下文

### 11.7 Context Distillation 技术

| 技术 | 原理 | 代表 |
|------|------|------|
| CoT蒸馏 | 强模型思维链→弱模型训练 | Orca, Phi |
| 上下文压缩蒸馏 | 长文本→固定summary tokens | AutoCompressor |
| Gisting | 长指令→~10个gist tokens | Gist Tokens |
| Prompt蒸馏 | few-shot知识→零样本能力 | Meta ICD |

### 11.8 Lost in the Middle 缓解

**问题**：模型对上下文中间位置信息利用率低。
**解法**：重要信息放首尾、分段摘要、显式标记、训练时随机放置。

## 十一、底层Attention优化技术⭐ 第五轮学习新增

> 来源：vLLM/Gemma/DeepSeek/Google/Berkeley最新研究

### 11.1 KV Cache优化栈（6种可叠加技术）

| 技术 | 原理 | 效果 |
|------|------|------|
| GQA/MQA | 多查询注意力，减少KV头数 | 内存减半 |
| PagedAttention | 按页管理KV内存（vLLM核心） | 消除碎片 |
| KV量化(KIVI) | 2bit量化KV Cache | 内存减75% |
| Token驱逐(H2O) | 动态驱逐不重要token | 固定内存 |
| Prefix Caching | 共享前缀复用KV | 命中率提升 |
| MLA(DeepSeek) | 多头潜在注意力 | 压缩KV维度 |

### 11.2 Infini-Attention（Google 2024）

线性attention记忆矩阵压缩历史segment，门控融合局部+全局。1B模型验证1M序列。

### 11.3 Ring Attention（Berkeley 2024）

多GPU环形传递KV块，在线softmax归全，上下文长度线性扩展。

### 11.4 Lost in the Middle

**问题：** 中间信息利用率低，首尾信息被更好利用。

**解法：**
- 重要信息放首尾
- 分段摘要
- 训练增强

### 11.5 Differential Transformer（Microsoft 2024）

两组attention map做差消除噪声，天然减少attention sinks。

## 十二、Reflexion自我反思框架⭐ 第五轮学习新增

> 来源：Shinn et al. 2023 — verbal reinforcement learning

**核心：失败后用自然语言反思存入长期记忆，下次自动规避。**

| 步骤 | 动作 |
|------|------|
| 1. 执行 | Agent执行任务 |
| 2. 评估 | 环境给出反馈 |
| 3. 反思 | 用自然语言总结失败原因 |
| 4. 存储 | 存入长期记忆 |
| 5. 重试 | 下次自动参考反思 |

效果：HumanEval 80→91%

在Hermes中的实践：用memory_save存储反思，用memory_recall检索。

## 十三、PRM过程奖励模型⭐ 第五轮学习新增

> 来源：Lightman et al. 2023 + Math-Shepherd 2024

**步骤级评估优于结果级评估(ORM)。**

| 评估方式 | 粒度 | 效果 |
|---------|------|------|
| ORM | 只看最终结果 | 基准 |
| PRM | 每步评估 | Best-of-N MATH 78.2% |

在Loop层中的应用：每轮迭代不仅检查最终结果，也检查中间步骤。

## 十四、Agent记忆系统架构⭐ 第七轮学习新增

> 来源：MemGPT/Letta、Mem0、Graphiti、Cognee

### 14.1 四种跨会话持久化模式

| 模式 | 代表 | 特点 |
|------|------|------|
| Agent-Native | Letta | OS式虚拟内存管理 |
| Memory-as-a-Service | Mem0 | ADD-only算法，92.5 LoCoMo |
| Graph-Backed | Graphiti | 时序Context Graph |
| Store-Backed | LangMem | PostgreSQL + 自动提取 |

### 14.2 三层记忆体系（Mem0）

| 层 | 类型 | 说明 |
|---|------|------|
| User | 语义记忆 | 用户偏好、事实 |
| Session | 情景记忆 | 单次会话上下文 |
| Agent | 程序记忆 | 技能、工具使用经验 |

### 14.3 时序记忆（Graphiti）

每个fact带`valid_from/valid_to`时间窗口，支持历史推理。

### 14.4 在Hermes中的实践

Hermes已有memory_save/memory_recall，对应Mem0的User层：
- type=fact → 语义记忆
- type=pattern → 程序记忆
- type=workflow → 情景记忆

## 十五、Durable Execution⭐ 第七轮学习新增

> 来源：Temporal — 崩溃后精确恢复

**核心理念：工作流执行状态持久化，崩溃后从断点恢复。** 在Hermes中用todo跟踪进度，每个completed项就是持久化检查点。

## 二十、上下文压缩四技术⭐ 第十一轮学习新增

> 来源：Gist Tokens + AutoCompressor + LLMLingua + Selective Context

**核心结论：不是所有token都平等，区分"必须保留"和"可以丢弃"。**

| 技术 | 原理 | 压缩率 | 适用场景 |
|------|------|--------|---------|
| **Gist Tokens** | 修改attention mask压缩指令到1个token | 26x | 指令压缩 |
| **AutoCompressor** | 递归分段压缩，每段生成summary vector | 30K+ | 长文档 |
| **LLMLingua** | 三层粗到细，差异化分配保留率 | 20x | prompt压缩 |
| **Selective Context** | 小模型计算自信息，删除低信息量词汇 | 可变 | 记忆筛选 |

### LLMLingua分层Budget Control

| 内容类型 | 保留率 | 原因 |
|---------|--------|------|
| 指令 | 高(80%) | 不能丢 |
| 问题 | 中(60%) | 核心意图 |
| 示例 | 低(30%) | 可压缩 |

### 在Hermes中的实践

```
长对话 → 信息密度评估 → 保留高密度信息 → 压缩低密度信息
```

1. **信息囤积症** — 不舍得丢弃上下文中的信息，导致窗口溢出、推理质量下降。**解法**：信任memory_save，信息存了就能找回来。

2. **记忆碎片化** — 每条信息都存，但没有结构，检索时找不到。**解法**：用concepts标签分类存储，content要包含关键词。

3. **上下文漂移** — 长对话中逐渐偏离原始目标。**解法**：用todo跟踪目标，定期回顾原始需求。

4. **过度压缩** — 压缩得太狠，丢失了关键细节。**解法**：P0/P1信息永远不压缩，只压缩P2/P3。

5. **忽视工具输出** — 工具调用的大量输出占据上下文但没被处理。**解法**：工具输出立即处理，只保留结论。

6. **死板的RAG** — 总是用同样的检索策略。**解法**：根据任务类型选择策略——精确查找用关键词，概念理解用语义检索。

7. **会话过长** — 一个会话做了太多不相关的事。**解法**：每完成一个独立任务就保存记忆，必要时开新会话。

## 十六、LazyMem查询时压缩⭐ 第十轮学习新增

> 来源：LazyMem (2026.7) — 21x token节省

**核心改变：不要过早压缩对话历史，保留原始数据，按需检索+选择性压缩。**

| 旧方式 | 新方式（LazyMem） |
|--------|-----------------|
| 每轮结束就压缩 | 查询时才压缩 |
| 丢失细节 | 保留原始数据 |
| 定期摘要 | 按需检索+选择性压缩 |

## 十七、工具输出生命周期管理⭐ 第十轮学习新增

> 来源：MemTool (2025.7) — 90-94%移除效率

| 工具类型 | 摘要策略 |
|---------|---------|
| terminal | 只保留exit_code + 关键输出行 |
| 文件读取 | 只保留相关段落 |
| 搜索结果 | 只保留匹配的文件名和行号 |

**自动过期：3轮未引用→自动移除。**

## 十八、记忆生命周期操作⭐ 第十轮学习新增

> 来源：MemOps (2026.7) — 记忆是操作序列，不是静态事实

| 操作 | 何时使用 |
|------|---------|
| **Remember** | 新发现的重要信息 |
| **Forget** | 过时或错误的信息 |
| **Update** | 信息发生变化 |
| **Reflect** | 任务完成后总结经验 |
| **Compose** | 组合多条记忆形成新知识 |

## 十九、Router-Mem渐进执行⭐ 第十轮学习新增

> 来源：Router-Mem (2026.8) — 减少27%推理时间

**改进Progressive Disclosure：加入自动充足性判断。**

L1摘要 → 充足？→ 直接回答 / 不充足 → L2结构 → 充足？→ 回答 / 不充足 → L3详情
