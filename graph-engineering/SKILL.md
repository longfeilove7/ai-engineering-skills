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

### 交接上下文过滤⭐ 第八轮学习新增

> 来源：OpenAI Agents SDK — HandoffInputData + HandoffInputFilter + nest_handoff_history

**交接时不是传递全部历史，而是过滤后传递。**

源码结构 (`handoffs/__init__.py:42-90`):
```python
@dataclass(frozen=True)
class HandoffInputData:
    input_history: str | tuple          # 原始输入
    pre_handoff_items: tuple[RunItem]   # 交接前 items
    new_items: tuple[RunItem]           # 当前 turn items
    input_items: tuple[RunItem] | None  # 过滤后的输入（替代 new_items）

HandoffInputFilter = Callable[[HandoffInputData], HandoffInputData]
```

**内置过滤器** (`extensions/handoff_filters.py:33-56`):
```python
def remove_all_tools(handoff_input_data):
    """过滤掉所有工具调用/输出，只保留用户消息和Agent回复"""
    # 移除：function_call, function_call_output, computer_call, web_search_call...
    # 保留：user messages, assistant messages
    return handoff_input_data.clone(
        input_history=filtered_history,
        pre_handoff_items=filtered_pre_handoff,
        new_items=filtered_new_items,
    )
```

**历史嵌套** (`handoffs/history.py:83-157`):
```python
def nest_handoff_history(handoff_input_data, *, history_mapper=None):
    """将之前的对话转录压缩为一个摘要消息"""
    # 输出格式：
    # "For context, here is the conversation so far between the user and the previous agent:
    #  <CONVERSATION HISTORY>
    #  1. user: 我需要修改配置文件
    #  2. assistant: 我来帮你修改...
    #  </CONVERSATION HISTORY>"
```

**Hermes 实践（delegate_task 改进）：**
```python
# 当前：传递全部上下文（token浪费）
delegate_task(goal="...", context=all_history)

# 改进：过滤后传递
def build_handoff_context(full_history, goal):
    return {
        "goal": goal,
        "relevant_files": extract_file_refs(full_history, goal),
        "completed_steps": extract_completed(full_history),
        "constraints": extract_constraints(full_history),
        # 不传递：其他任务历史、中间推理、工具调用细节
    }
delegate_task(goal="执行修改", context=build_handoff_context(history, goal))
```

### 任务自动分解⭐ 第八轮学习新增

**判断一个任务需要几个Agent：**

```python
def should_spawn_executor(task):
    if task.affected_files >= 5:          # 大批次
        return True
    if task.read_operations > 20:         # 读取密集型，会污染主上下文
        return True
    if task.is_reversible and task.affected_files < 3:  # 小且可逆
        return False
    if task.depends_on_other_tasks:       # 有依赖，保持串行
        return False
    return False  # 默认单独工作
```

| 维度 | 单Agent | 多Agent串行 | 多Agent并行 |
|------|---------|------------|------------|
| 文件数 | <3 | 3-10 | >10 且无交叉 |
| 复杂度 | 简单修改 | 需要验证 | 独立子任务 |
| 可逆性 | 可逆 | 不可逆 | 可逆 |
| 依赖关系 | 无 | 有顺序依赖 | 无依赖 |

### Agent选择策略⭐ 第八轮学习新增

```python
def select_agent(task, available_agents):
    if task.has_write_operations:
        if task.is_mechanical:
            return "executor"  # 机械执行
        else:
            return "advisor"   # 复杂写入先规划
    if task.is_verification:
        return "verifier"
    if task.is_planning or task.is_review:
        return "advisor"
    if task.is_small and task.is_reversible:
        return "executor"      # 小任务直接执行
    return "advisor"           # 默认先规划
```

## Stackwich Policy Block（精华版）

### CrewAI双层架构参考⭐ 第二轮学习新增

> 来源：CrewAI v1.15 — Flows + Crews双层架构

| 层 | 说明 |
|---|------|
| **Flows** | 事件驱动工作流，控制整体流程 |
| **Crews** | 角色扮演Agent团队，执行具体任务 |

角色系统：role/goal/backstory三要素定义Agent人格。

### AutoGen事件驱动参考⭐ 第二轮学习新增

> 来源：AutoGen v0.4 — 3层架构

| 层 | 说明 |
|---|------|
| **Core** | 基础消息传递和事件系统 |
| **AgentChat** | 高级Agent对话API |
| **Extensions** | 第三方集成 |

核心：事件驱动、异步优先、GraphFlow工作流。

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

### 任务依赖图自动构建⭐ 第九轮深化

> 完整报告：`references/task-decomposition-strategies.md`

**三阶段构建：LLM分解 → 规则推断依赖 → 拓扑排序提取并行层**

```python
# 核心：基于文件读写关系自动推断依赖（无需LLM）
def build_dependency_graph(subtasks):
    for t1, t2 in combinations(subtasks, 2):
        if t1.writes & t2.reads:  # T2读了T1写的 → 依赖
            dag[t2.id].deps.add(t1.id)
        if t1.writes & t2.writes:  # 写冲突 → 强制串行
            dag[t2.id].deps.add(t1.id)
    return topological_layers(dag)  # → 同层可并行
```

### 动态负载均衡⭐ 第九轮深化

```python
# 最小负载 + 文件亲和性 + 能力匹配
class LoadBalancer:
    def assign(self, task):
        candidates = [a for a in self.agents if a.role == task.role and a.is_available]
        candidates.sort(key=lambda a: (a.load, a.avg_task_time))
        # 文件亲和性加分：之前处理过相同文件的Agent优先
        return candidates[0]
```

### 并行编排模式⭐ 第九轮深化

| 拓扑 | 实现 | 写约束 |
|------|------|--------|
| 串行 | advisor→executor→verifier→review | 单executor |
| 并行 | asyncio.gather + 扇入verifier | 文件集不重叠 |
| DAG | 拓扑排序 → 按层执行 → 层间传递结果 | 同层不重叠，跨层可重叠 |

## 故障恢复⭐ 第十轮新增

> 完整报告：`references/fault-recovery-patterns.md`

### Agent崩溃时的任务重分配

| 情况 | 策略 |
|------|------|
| executor超时/崩溃 | 从检查点恢复，分配给其他executor |
| verifier崩溃 | 跳过验证，直接进review（标记WARNING） |
| advisor崩溃 | 用最近的计划缓存恢复，或降级为executor直执行 |

**检查点机制**：每个子步骤完成后保存checkpoint（已完成步骤 + 剩余步骤 + 已修改文件）。恢复时从断点继续，不从头开始。

**重分配决策树**：
```
崩溃 → 有检查点？→ 是 → 从检查点恢复，换Agent
                 → 否 → 从头开始，换Agent
       已失败Agent数 < 2？→ 是 → 重分配
                          → 否 → 升级到human
```

### 部分失败时的降级策略

**任务关键性分级**：
- CRITICAL：有下游依赖的任务（失败则重试换Agent）
- IMPORTANT：验证/测试任务（失败可降级跳过）
- OPTIONAL：文档/格式/注释（失败直接忽略）

**降级层次**：L1重试 → L2跳过非关键 → L3简化方案 → L4回退 → L5人工接管

### 分布式Agent状态同步

**推荐方案：DAG状态机 + 文件锁**

| 机制 | 用途 |
|------|------|
| 文件级乐观锁 | 防止并行executor写同一文件 |
| DAG显式传递 | 完成节点显式输出给下游，无隐式共享 |
| 事件日志 | 记录所有修改，verifier可审计 |

```python
# 执行前检查写冲突
conflicts = check_file_locks(agent_id, task.writes)
if conflicts:
    return {"status": "blocked", "conflicts": conflicts}
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
