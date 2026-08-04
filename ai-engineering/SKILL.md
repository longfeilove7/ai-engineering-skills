---
name: ai-engineering
description: AI Engineering总调度 — 自动分析任务复杂度，按需加载prompt/context/harness/loop/graph五层skill，执行后自我反思并进化，累积改进后自动定版推送GitHub。Use when任何任务开始时自动触发，是5层架构的入口和进化引擎。
version: 1.0.0
---

# AI Engineering 总调度

自动分析任务→按需加载skill→执行→自我反思→进化→定版→推送GitHub。

## 第一步：任务分析（每次对话开始时）

收到用户消息后，立即分析任务复杂度，决定激活哪些层：

```yaml
分析维度:
  - 文件数量: 1个/2-4个/5+个
  - 是否需要迭代: 一次完成/需要反复优化
  - 是否需要多Agent: 单人可完成/需要分工协作
  - 输出复杂度: 简单回复/结构化文档/多文件产出
  - 用户明确要求: 指定了哪个skill
```

### 层级激活规则

| 任务类型 | 激活的层 | 示例 |
|---------|---------|------|
| **简单** | Prompt | "几点了"、"star这个项目" |
| **中等** | Prompt + Context + Harness | "帮我查一下这个文件"、"分析这个文档" |
| **复杂** | Prompt + Context + Harness + Loop | "写一份技术方案"、"对比多个文档" |
| **超复杂** | 全部5层 | "帮我做投标文件"、"代码审查+重构" |
| **用户指定** | 按用户要求 | "用graph-engineering流程做" |

### 分析模板

```yaml
task_analysis:
  message: "用户的消息"
  complexity: simple | medium | complex | ultra
  activate:
    - prompt-engineering     # 总是激活
    - context-engineering    # 中等及以上
    - harness-engineering    # 中等及以上
    - loop-engineering       # 复杂及以上
    - graph-engineering      # 超复杂
  reason: "判断依据"
```

## 第二步：按skill流程执行

根据激活的层，按顺序执行：

### 2.1 Prompt层（总是执行）

检查用户prompt是否包含必要要素：
- 🎯目标、👤角色、📋背景、📏约束、📐格式、💡示例、🎭风格
- 缺少关键要素时提醒补充
- 简单任务不提醒

### 2.2 Context层（中等及以上）

管理上下文：
- 评估当前context使用量
- 决定是否需要压缩
- 管理多轮对话状态
- 决定哪些信息需要持久化到memory

### 2.3 Harness层（中等及以上）

编排工具：
- 选择合适的工具组合
- 定义完成条件
- 遵循单写入者原则
- 只读强制（只读工具不写入）

### 2.4 Loop层（复杂及以上）

迭代优化：
- 执行→自我反思→质量门控
- 失败最多2次重试
- 从不静默失败
- 记录每轮迭代结果

### 2.5 Graph层（超复杂）

多Agent协作：
- advisor规划 → executor执行 → verifier验证 → advisor审查
- 升级机制：2次失败后升级到human
- 交接契约：每个阶段明确输入输出

## 第三步：自我反思（每次任务完成后）

任务完成后，进行自我反思：

```yaml
reflection:
  task: "任务描述"
  layers_used: [prompt, context, harness]
  
  what_worked:
    - "哪些skill建议帮到了你"
    - "哪个工具组合效果好"
    
  what_failed:
    - "哪些建议没用或误导"
    - "哪个环节卡住了"
    
  user_feedback: "用户的反应（满意/不满/纠正）"
  
  improvement_ideas:
    - "如果重来，会怎么做不同"
    - "哪个skill需要调整"
```

### 反思触发条件

| 条件 | 是否反思 |
|------|---------|
| 简单任务（一句话完成） | ❌ 不反思 |
| 中等任务（使用2-3个工具） | ⚠️ 轻量反思 |
| 复杂任务（多步骤、多工具） | ✅ 完整反思 |
| 用户明确纠正了你 | ✅ 必须反思 |
| 任务失败或部分失败 | ✅ 必须反思 |

## 第四步：进化（累积改进）

### 4.1 学习成果存储

每次反思后，将学习成果存入memory：

```yaml
# 使用memory工具存储
content: "在XX类型任务中，YY策略效果好/不好，因为ZZ"
type: pattern | workflow
concepts: "skill名, 场景, 策略"
```

### 4.2 进化阈值

当累积的学习成果满足以下条件时，触发进化：

| 条件 | 阈值 |
|------|------|
| 同一skill的改进建议 | ≥3条 |
| 用户纠正同一问题 | ≥2次 |
| 某策略反复有效/无效 | ≥3次 |

### 4.3 进化执行

触发进化时：

1. **分析学习成果** — 综合所有相关memory
2. **提出改进方案** — 具体修改哪个skill的哪部分
3. **执行patch** — 使用skill_manage修改skill
4. **记录变更** — 写入changelog
5. **版本号+1** — minor版本（1.0→1.1）

```yaml
evolution:
  skill: "要修改的skill名"
  changes:
    - section: "修改的章节"
      old: "旧内容摘要"
      new: "新内容摘要"
      reason: "基于哪些学习成果"
  version_old: "1.0.0"
  version_new: "1.1.0"
```

## 第五步：定版与推送

### 5.1 定版条件

满足以下条件时，标记为稳定版：

| 条件 | 说明 |
|------|------|
| 迭代次数 ≥5 | 经过足够多的实战验证 |
| 用户纠正次数 ≤1 | 近期很少被纠正 |
| 成功率 ≥90% | 大部分任务成功完成 |
| 无重大改进建议 | 没有迫切需要修改的 |

### 5.2 版本号规则

```
MAJOR.MINOR.PATCH

MAJOR — 架构级变更（如新增一层）
MINOR — 功能改进（基于进化）
PATCH — 错误修复（bug fix）
```

### 5.3 推送GitHub

定版后自动推送：

```bash
cd ~/.hermes/skills/productivity/ai-engineering-skills
# 同步所有skill文件
# git add -A
# git commit -m "v1.1.0: 基于XX次实战改进YY"
# git push origin main
```

## 第六步：进化记录

每次进化都记录在changelog中：

```markdown
# Changelog

## v1.1.0 (2026-08-04)
- prompt-engineering: 优化了提醒时机，简单任务不再提醒
- context-engineering: 新增RAG检索策略
- 基于3次实战学习成果

## v1.0.0 (2026-08-04)
- 初始版本
- 基于stackwich架构
```

## 工作流总结

```
用户消息 → 任务分析 → 加载skill → 执行 → 反思 → 存储学习
                                                      ↓
                                              累积够阈值？
                                                  ↓ Yes
                                            提出改进 → patch → 版本+1
                                                  ↓
                                              满足定版条件？
                                                  ↓ Yes
                                            推送GitHub
```

## 第七章：故障归因（F）⭐ 第三轮学习新增

> 来源：LIFE框架（arXiv:2605.14892）— 五层架构中缺失的关键层

**我们的5层架构对齐LIFE的L-I阶段，但缺少F（Fault Attribution）和E（Evolution）。**

### 7.1 故障归因流程

```
任务失败
    ↓
归因分析：是哪一层的问题？
    ├─ Prompt层问题 → prompt写得不清楚
    ├─ Context层问题 → 上下文不足或过载
    ├─ Harness层问题 → 工具选择错误或编排不当
    ├─ Loop层问题 → 迭代策略失效
    └─ Graph层问题 → Agent分工或交接出错
    ↓
记录归因到memory
    ↓
下次同类任务自动规避
```

### 7.2 故障分类

| 故障类型 | 归因层 | 示例 |
|---------|--------|------|
| 理解错误 | Prompt | 用户意图被误解 |
| 信息不足 | Context | 缺少关键上下文 |
| 工具失败 | Harness | 选错工具或参数错误 |
| 质量不达标 | Loop | 迭代次数用完仍未达标 |
| 协作失败 | Graph | Agent间交接信息丢失 |

## 第八章：闭环自进化（E）⭐ 第三轮学习新增

> 来源：AReaL2.0（arXiv:2607.01120）+ Skill Self-Play（arXiv:2607.22529）

### 8.1 进化闭环

```
执行任务 → 故障归因 → 存储经验 → 累积阈值 → 改进skill → 版本+1 → 推送GitHub
    ↑                                                              ↓
    └──────────── 下次任务自动应用改进 ←──────────────────────────┘
```

### 8.2 Memory-Reward Trap防护

> 来源：RoMeRL（arXiv:2608.02508，2026.8最新）

**问题：** Agent记忆过度积累高奖励路径导致"回音壁"，创新能力下降。

**解法：**
- 记忆健康检查：定期审视memory是否有过度集中
- 多样性激励：主动尝试新路径，不总是走记忆中的"安全路径"
- 遗忘机制：过时的记忆定期清理

### 8.3 Skill Self-Play三体进化

| 角色 | 职责 |
|------|------|
| **Proposer** | 提出新的skill改进建议 |
| **Solver** | 用改进后的skill执行任务 |
| **Validator** | 验证改进是否真正有效 |

三者协同进化，解决"任务多样性vs验证可靠性"两难。

## 第九章：跨层优化⭐ 第十五轮学习新增

**当前问题：5层各自独立运行，层间信息流断裂。**

### 9.1 四个标准数据结构

| 结构 | 流向 | 内容 |
|------|------|------|
| TaskCard | Prompt→所有层 | 任务描述+约束+完成条件 |
| ContextBudget | Context→Harness | token预算+优先级 |
| ExecResult | Harness→Loop | 执行结果+错误信息 |
| QualityVec | Loop→所有层 | 多维度质量评分+故障归因 |

### 9.2 六条反馈回路

| 回路 | 触发 | 动作 |
|------|------|------|
| Loop→Prompt | 质量分<7 | 改进prompt策略 |
| Loop→Context | 信息不足 | 补充上下文 |
| Loop→Harness | 工具失败 | 换工具或修正参数 |
| Context→Harness | token接近上限 | 启用压缩 |
| Graph→全局 | 故障归因 | 更新skill pitfalls |
| Harness→Graph | Guardrail触发 | 调整任务分配 |

### 9.3 三层门控体系

```
入口门控（Prompt质量）→ 执行门控（Guardrails）→ 交付门控（质量+收敛）
```

### 9.4 预期收益

| 指标 | 预期改善 |
|------|---------|
| 任务成功率 | +15% |
| 迭代次数 | -28% |
| Token浪费 | -50% |
| 故障定位时间 | -80% |

## Pitfalls

1. **不要过度反思** — 简单任务不需要反思
2. **不要频繁进化** — 至少累积3条同类建议才改
3. **不要破坏性修改** — 进化是增量改进，不是重写
4. **不要跳过验证** — patch后要验证skill还能正常加载
5. **不要静默进化** — 每次进化都要记录changelog
