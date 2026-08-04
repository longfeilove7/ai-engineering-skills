---
name: graph-engineering
description: Graph Engineering — 多Agent协作流程编排。定义advisor/executor/verifier角色，实现计划→执行→验证→审查的完整闭环。Use when需要多个Agent协作完成复杂任务、代码审查、自动测试、质量门控。
---

# Graph Engineering — 多Agent协作流程

基于stackwich的Graph层，实现多Agent分工协作的完整闭环。

## 核心理念

**单一职责，明确交接。** 每个Agent只做一件事，做完交给下一个。

## 四角色模型

| 角色 | 职责 | 模型策略 | 权限 |
|------|------|---------|------|
| 🍞 **advisor** | 规划+审查 | 贵的（能力强） | 只读，不写文件 |
| 🥩 **executor** | 机械执行 | 便宜的（速度快） | 读写，唯一写入者 |
| 🧪 **verifier** | 验证门控 | 中等 | 只读，运行测试 |
| 👤 **human** | 最终决策 | — | 全部权限 |

## 标准流程

```
advisor规划 → executor执行 → verifier验证 → advisor审查
     ↑                                          ↓
     └──────── 失败时最多循环2次 ←──────────────┘
```

### 阶段1：Plan（advisor）

```
输入：任务描述 + 上下文
输出：执行计划（具体步骤、文件路径、验证命令）

规则：
- 只读，不修改任何文件
- 每个步骤必须包含：具体文件、具体改动、验证命令
- 标记不可逆操作
- 如果信息不足，停止并要求补充
```

### 阶段2：Execute（executor）

```
输入：advisor的执行计划
输出：完成的改动

规则：
- 严格按计划执行，不创新
- 不做计划外的优化
- 不创建备份文件（suffix拷贝）
- 完成后输出明确的完成标记
```

### 阶段3：Verify（verifier）

```
输入：executor的改动 + 验证命令
输出：GATE: PASS 或 GATE: FAIL + 原因

规则：
- 运行项目的真实测试/lint/构建
- 只报告结果，不修复问题
- PASS或FAIL，不模棱两可
- 失败时列出具体失败项
```

### 阶段4：Review（advisor）

```
输入：executor的改动 + verifier的结果
输出：APPROVE 或 REVISE: <修改列表>

规则：
- 对比计划vs实际改动
- 检查代码规范、影响范围
- APPROVE = 完成
- REVISE = 列出需要修改的具体项
```

## 升级机制

| 情况 | 处理 |
|------|------|
| verifier FAIL 1次 | 回到executor修复 |
| verifier FAIL 2次 | 停止，升级到human |
| advisor REVISE 1次 | 回到executor修改 |
| advisor REVISE 2次 | 停止，升级到human |
| 信息不足 | 立即升级到human |

**核心原则：从不静默失败。** 每次失败都必须报告原因。

## 快速模式（小任务）

小任务跳过完整流程：

```
小任务 → executor直接执行 → verifier验证 → 完成
```

判断标准：
- 改动少于3个文件
- 不涉及不可逆操作
- 有现成的验证命令

## 交接契约

每个阶段的输出必须包含：

```yaml
status: completed | failed | needs_input
output: 具体输出内容
next: 下一个阶段和需要的信息
blockers: 阻塞问题（如有）
```

## Stackwich Policy Block（精华版）

以下内容来自stackwich的CLAUDE.md policy block，是经过实战验证的行为准则：

### Prompt层
- 任何委托给子Agent的工作，都必须写一个上下文丰富的prompt——为什么做、已经尝试过什么、排除了什么、精确的文件/行号/标识符——而不是简短的命令。永远不要把理解委托给子Agent。
- 每次委托都要声明完成条件：验证命令和预期输出。没有完成条件的工作交回来是不可验证的。

### Context层
- 研究、探索、读取密集型扫描用fork，让原始工具输出离开主对话。当上下文可复用时优先fork而不是新建Agent。
- 记忆用于跨会话持久事实，计划用于实施前对齐，任务用于对话内进度——各自有各自用途，不要混用。
- 不要重新读取你刚写入/编辑的文件来"确认"——写入已经成功或已经报错了。

### Harness层
- 单一写入者：只有executor修改文件。advisor和verifier永远不写，即使修复看起来显而易见——审查者的修复是一个无人知晓的未审查变更。
- 只读意味着实践中的只读：给只读Agent的Bash权限仅用于检查和验证（测试、lint、构建、git diff/status）。
- 没有什么能凭"意图"就标为完成。报告观察到的结果——运行了什么命令、实际输出是什么。

### Loop层
- 循环工作默认用/loop，不是手动重跑。
- 失败路由，硬性停止：
  - `GATE: FAIL` → 把失败输出和原始计划交回executor
  - `REVISE: ...` → 把编号列表和原始计划交回executor
  - 两条路径最多循环2次。第三次，停止并把计划、尝试过的内容、未解决的问题交给用户。
- 两次打击规则：同一验证失败两次，停止盲目迭代。

### Graph层
- 默认：小/可逆改动单独工作。只在>=5文件的机械批次或会污染主上下文的读取密集型扫描时才spawn executor。
- 如果批次大且项目独立（无共享文件、无顺序依赖），并行shard到多个executor运行，扇入到单个verifier。如果项目涉及同一文件或必须按顺序，保持串行。
- 在任何难以逆转的变更之前咨询advisor：schema迁移、生产/发布配置、API/CLI接口、认证权限、force-push、依赖版本升级、CI/CD编辑、数据删除。
- 完整三明治（advisor计划→executor执行→verifier验证→advisor审查）是强制性的，不是可选的。

## 与Hermes的集成

### 使用delegate_task实现

```python
# advisor规划
plan = delegate_task(goal="规划...", role="orchestrator")

# executor执行
result = delegate_task(goal="按计划执行...", context=plan)

# verifier验证
verification = delegate_task(goal="验证...", context=result)

# advisor审查
review = delegate_task(goal="审查...", context=verification)
```

### 使用todo跟踪

```python
todo(todos=[
    {"id": "1", "content": "advisor规划", "status": "in_progress"},
    {"id": "2", "content": "executor执行", "status": "pending"},
    {"id": "3", "content": "verifier验证", "status": "pending"},
    {"id": "4", "content": "advisor审查", "status": "pending"}
])
```

## 反模式（避免）

1. **executor创新** — executor不应该做计划外的改动
2. **verifier修复** — verifier只报告，不修复
3. **静默失败** — 每次失败都必须报告
4. **跳过验证** — 即使是小改动也要验证
5. **无限循环** — 最多2次，必须升级

## Present Results to User

```markdown
## 执行结果

| 阶段 | 状态 | 说明 |
|------|------|------|
| 🍞 Plan | ✅ | 3个步骤 |
| 🥩 Execute | ✅ | 修改2个文件 |
| 🧪 Verify | ✅ | 所有测试通过 |
| 🍞 Review | ✅ | APPROVE |

**结果：** 任务完成。
```

## Pitfalls

1. **不要让executor做规划** — executor是机械执行者
2. **不要让verifier做修复** — verifier是门控，不是修复者
3. **不要跳过review** — 即使验证通过也要审查
4. **不要超过2次循环** — 否则升级到human
5. **不要静默** — 每个阶段都要输出明确结果
