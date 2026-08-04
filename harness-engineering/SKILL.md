---
name: harness-engineering
description: "Harness层编排工程 — Agent编排策略、工具组合模式、单写入者原则、只读强制、完成条件定义。基于stackwich Harness层定义。"
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [orchestration, agent, harness, tool-combination, guardrails, workflow]
    related_skills: [subagent-driven-development, writing-plans, planning, debugging]
---

# Harness 层编排工程

## 核心公理

> **单一写入者，只读强制在实践中执行，完成必须被观察。**

这三条公理构成 Harness 层的不变量。违反任何一条，系统就会出现幻觉状态、数据竞争或静默失败。

---

## 第一章：Agent 编排策略

### 1.1 编排模型

Harness 层将 Agent 任务分为三种角色：

| 角色 | 权限 | 典型工具集 | 职责 |
|------|------|-----------|------|
| **编排者（Orchestrator）** | 读写 | `todo`, `session_search`, `delegate_task` | 拆解任务、分配子任务、验证完成 |
| **执行者（Executor）** | 只读 + 受限写入 | `read_file`, `search_files`, `terminal`（只读命令） | 分析、推理、生成内容片段 |
| **写入者（Writer）** | 独占写入 | `write_file`, `patch`, `terminal`（写入命令） | 将结果写入持久存储 |

**关键规则：任何时刻，对同一目标文件/资源，只有一个写入者。**

### 1.2 编排流程

```
用户需求
  ↓
编排者：拆解任务 → 创建 todo 列表
  ↓
编排者：为每个子任务选择角色和工具集
  ↓
┌─────────────────────────────────────┐
│  子任务 N：                          │
│    1. 执行者读取上下文（只读）         │
│    2. 执行者生成方案（内存中）         │
│    3. 写入者提交变更（独占写入）       │
│    4. 验证者确认完成（只读检查）       │
└─────────────────────────────────────┘
  ↓
编排者：汇总结果，报告用户
```

### 1.3 任务拆解原则

**粒度标准：每个子任务 2-5 分钟可完成。**

拆解时问自己三个问题：
1. 这个任务是否只有一个写入目标？（单写入者）
2. 验证这个任务是否只需要读取操作？（只读验证）
3. 完成条件是否可以被工具自动检查？（可观察完成）

如果任一答案为「否」，继续拆解。

---

## 第二章：工具组合模式

### 2.1 模式一：读取-分析-报告（RAP）

最安全的模式，零副作用。

```
search_files → read_file → 分析 → 输出结论
```

**适用场景：** 代码审查、文档分析、架构理解

**示例：**
```python
# Step 1: 搜索相关文件
search_files("class.*Controller", target="content", file_glob="*.java")

# Step 2: 读取关键文件
read_file("src/main/java/com/example/UserController.java")

# Step 3: 分析并输出（纯内存操作，无写入）
```

**Harness 约束：** 此模式中禁止调用任何写入工具。如果分析结果需要持久化，进入模式二。

### 2.2 模式二：读取-生成-写入（RGW）

单写入者模式，变更可追溯。

```
read_file(s) → 生成内容 → write_file/patch（单一目标）
```

**适用场景：** 代码生成、文档编写、配置修改

**规则：**
- 写入前必须先读取目标文件（防止覆盖未知内容）
- 每次操作只写入一个文件
- 写入后立即验证（读取回来对比）

**示例：**
```python
# Step 1: 读取现有文件
read_file("config/settings.yaml")

# Step 2: 生成新内容（在内存中构建）

# Step 3: 写入（单一写入者）
write_file("config/settings.yaml", new_content)

# Step 4: 验证写入结果
read_file("config/settings.yaml")  # 确认内容正确
```

### 2.3 模式三：读取-执行-验证（REV）

需要执行命令时的安全模式。

```
read_file → terminal（只读命令） → 检查输出 → 决定下一步
```

**适用场景：** 测试运行、构建验证、状态检查

**规则：**
- `terminal` 命令默认只读（`cat`, `grep`, `pytest`, `npm test`）
- 写入类命令（`rm`, `mv`, `cp`, `echo >`）必须在明确的写入阶段执行
- 每个 `terminal` 调用后检查 `exit_code`

### 2.4 模式四：并行读取-串行写入（PR-SW）

多源读取、单一写入的高级模式。

```
┌─ read_file(A) ─┐
├─ read_file(B) ─┤→ 合并分析 → write_file(C)
└─ read_file(C) ─┘
```

**适用场景：** 多文件对比分析、合并报告、跨模块重构

**规则：**
- 读取可以并行（批量调用）
- 写入必须串行（一次一个）
- 合并逻辑在内存中完成，不产生中间文件

### 2.5 模式五：委托-审查-提交（DRC）

子 Agent 编排模式。

```
编排者 → delegate_task(执行者) → 审查结果 → write_file
```

**适用场景：** 复杂代码生成、多步骤任务、需要专业判断的工作

**规则：**
- 子 Agent 只返回文本结果，不直接写入文件
- 编排者审查后统一写入
- 每个子 Agent 有明确的工具集限制

---

## 第三章：单写入者原则

### 3.1 定义

> 对任何共享资源（文件、数据库、API），同一时刻只允许一个 Agent 或任务执行写入操作。

### 3.2 为什么需要单写入者

| 问题 | 表现 | 后果 |
|------|------|------|
| 写入冲突 | 两个任务同时修改同一文件 | 一方覆盖另一方的变更 |
| 状态不一致 | 读取了过时的内容再写入 | 数据丢失或逻辑错误 |
| 幻觉状态 | Agent 认为写入成功但实际被覆盖 | 静默失败，难以排查 |

### 3.3 实践规则

**规则 1：写入目标锁定**

在创建 todo 时，每个任务必须声明其写入目标：

```python
todo([
    {
        "id": "task-1",
        "content": "修改 src/config.yaml — 添加数据库配置",
        "status": "pending",
        "write_targets": ["src/config.yaml"]  # 声明写入目标
    },
    {
        "id": "task-2",
        "content": "更新 README.md — 添加安装说明",
        "status": "pending",
        "write_targets": ["README.md"]
    }
])
```

**规则 2：冲突检测**

如果两个任务的 `write_targets` 有交集，必须串行执行，不能并行。

```python
# ❌ 错误：两个任务都写 src/config.yaml，不能并行
# ✅ 正确：先完成 task-1，再执行 task-2
```

**规则 3：子 Agent 不直接写入**

子 Agent 通过 `delegate_task` 执行时，返回结果文本，由编排者统一写入：

```python
# 子 Agent 只返回修改建议
result = delegate_task(
    goal="分析 src/app.py 并提出优化建议",
    toolsets=['file']  # 只给文件读取权限
)

# 编排者审查后写入
if review_passes(result):
    patch("src/app.py", old_code, new_code)
```

### 3.4 违反单写入者的诊断

出现以下症状时，检查是否违反了单写入者原则：

- 文件内容部分正确、部分是旧版本
- `git diff` 显示意外的回退
- Agent 报告「已修改」但文件内容未变化
- 多个任务完成后，只剩最后一个任务的变更

---

## 第四章：只读强制

### 4.1 定义

> 在 Harness 层，所有操作默认为只读。写入操作必须显式声明、显式执行、显式验证。

### 4.2 只读工具清单

以下工具天然只读，可以自由使用：

| 工具 | 用途 | 风险等级 |
|------|------|---------|
| `read_file` | 读取文件内容 | 零 |
| `search_files` | 搜索文件内容或文件名 | 零 |
| `session_search` | 搜索会话历史 | 零 |
| `skills_list` | 列出可用技能 | 零 |
| `skill_view` | 查看技能详情 | 零 |
| `browser_snapshot` | 获取页面快照 | 零 |
| `browser_vision` | 页面截图分析 | 零 |
| `vision_analyze` | 图片分析 | 零 |
| `process(action='poll')` | 检查进程状态 | 零 |

### 4.3 受控写入工具清单

以下工具有副作用，必须在明确的写入阶段使用：

| 工具 | 用途 | 前置条件 |
|------|------|---------|
| `write_file` | 创建/覆盖文件 | 必须先 `read_file` 或确认文件不存在 |
| `patch` | 精确替换文件内容 | 必须先 `read_file` 确认当前内容 |
| `terminal`（写入命令） | 执行 shell 命令 | 命令意图必须明确，非只读命令需标注 |
| `todo` | 更新任务列表 | 仅编排者使用 |
| `skill_manage` | 创建/修改技能 | 必须先 `skill_view` 确认当前状态 |

### 4.4 只读阶段与写入阶段的分离

每个任务必须明确划分阶段：

```
【只读阶段】← 可以自由探索、分析、推理
  read_file("目标文件")
  search_files("相关模式")
  read_file("依赖文件")
  → 形成修改方案

【写入阶段】← 按方案精确执行，一次一个写入
  write_file("目标文件", 新内容)
  → 立即验证

【验证阶段】← 回到只读，确认结果
  read_file("目标文件")
  → 对比预期
```

### 4.5 强制机制

**清单检查：** 在执行任何写入操作前，自问：

- [ ] 我是否已经读取了目标文件的当前内容？
- [ ] 我是否有明确的修改方案？
- [ ] 这个写入是否是当前任务的唯一写入？
- [ ] 写入后我能否立即验证？

全部「是」→ 执行写入。任一「否」→ 回到只读阶段。

---

## 第五章：完成条件定义

### 5.1 定义

> 完成必须被观察。一个任务的完成不是 Agent 的主观判断，而是工具验证的客观事实。

### 5.2 完成条件的三要素

每个任务的完成条件必须包含：

1. **可执行的验证命令** — 不是「检查一下」，而是具体的工具调用
2. **明确的预期结果** — 不是「应该没问题」，而是具体的输出模式
3. **失败时的行为** — 不是「如果失败就修复」，而是具体的重试/回退策略

### 5.3 完成条件模板

```markdown
### 任务 N: [描述]

**完成条件：**
- 验证命令: `pytest tests/test_xxx.py -v`
- 预期结果: 全部测试通过，输出 `X passed`
- 失败行为: 读取失败日志，修复代码，重新运行（最多 3 次）

**次级验证：**
- 文件存在: `search_files("expected_file.py", target="files")`
- 内容正确: `read_file("expected_file.py")` 包含关键函数签名
```

### 5.4 各类任务的完成条件

**代码修改类：**
```python
# 完成条件
terminal("pytest tests/ -v")  # 全部通过
terminal("python -m py_compile src/modified.py")  # 语法正确
read_file("src/modified.py")  # 包含预期变更
```

**文档生成类：**
```python
# 完成条件
search_files("generated_doc.md", target="files")  # 文件存在
read_file("generated_doc.md")  # 包含预期章节
terminal("wc -l generated_doc.md")  # 行数合理（> 50）
```

**配置修改类：**
```python
# 完成条件
read_file("config.yaml")  # 包含新配置项
terminal("python -c \"import yaml; yaml.safe_load(open('config.yaml'))\"")  # YAML 合法
```

**分析报告类：**
```python
# 完成条件
# 分析报告直接输出给用户，无需文件验证
# 但必须满足：
# - 覆盖了所有要求的分析维度
# - 结论有数据/证据支撑
# - 输出格式符合要求
```

### 5.5 完成观察的层级

```
Level 1: 存在性检查
  → 文件是否存在？命令是否执行成功？

Level 2: 内容检查
  → 文件内容是否包含预期变更？输出是否匹配模式？

Level 3: 集成检查
  → 变更后系统是否正常工作？测试是否全部通过？

Level 4: 业务检查
  → 变更是否满足用户需求？是否有遗漏？
```

**最低标准：** 每个任务必须达到 Level 2。涉及代码的任务必须达到 Level 3。面向用户的交付物必须达到 Level 4。

---

## 第六章：实战检查清单

### 6.1 任务开始前

- [ ] 任务是否已拆解为 2-5 分钟的粒度？
- [ ] 是否明确了写入目标？（单写入者）
- [ ] 是否声明了完成条件？（可观察完成）
- [ ] 是否区分了只读阶段和写入阶段？（只读强制）

### 6.2 任务执行中

- [ ] 只读阶段是否只使用了只读工具？
- [ ] 写入前是否先读取了目标文件？
- [ ] 写入操作是否一次只写一个文件？
- [ ] 写入后是否立即验证？

### 6.3 任务完成后

- [ ] 验证命令是否已执行？
- [ ] 预期结果是否匹配？
- [ ] 是否有回归？（运行相关测试）
- [ ] 完成状态是否已更新到 todo？

---

## 第七章：常见反模式

### 反模式 1：先写后读

```
❌ write_file("a.py", content) → read_file("a.py")  # 应该先读
✅ read_file("a.py") → 分析 → write_file("a.py", content) → read_file("a.py") 验证
```

### 反模式 2：并行写入

```
❌ 同时 delegate_task 写入同一个文件
✅ 串行执行，一个任务完成后再执行下一个
```

### 反模式 3：主观完成

```
❌ "我觉得已经完成了"
✅ "pytest 全部通过，read_file 确认内容正确，wc -l 确认行数合理"
```

### 反模式 4：混合阶段

```
❌ 读一行、写一行、读一行、写一行（读写交织）
✅ 全部读取 → 形成方案 → 全部写入 → 全部验证
```

### 反模式 5：隐式写入

```
❌ terminal("echo 'new line' >> file.txt")  # 写入隐藏在 terminal 中
✅ write_file("file.txt", existing_content + "\nnew line")  # 写入意图明确
```

---

## 第八章：与现有技能的集成

| 技能 | 集成点 |
|------|--------|
| `writing-plans` | 计划中的每个任务必须声明 write_targets 和完成条件 |
| `subagent-driven-development` | 子 Agent 默认只读，写入由编排者统一执行 |
| `planning` | Spike 模式中，验证阶段严格只读 |
| `debugging` | 调试阶段全部只读，修复阶段单写入者 |
| `test-driven-development` | 红灯阶段只读（确认失败），绿灯阶段单写入（实现代码） |

---

## 第八章：Guardrails机制⭐ 第二轮学习新增

> 来源：OpenAI Agents SDK — 3级安全护栏

**核心理念：在工具调用的输入/输出/执行三个阶段设置安全检查。**

| 级别 | 检查点 | 检查内容 |
|------|--------|---------|
| **Input** | 工具调用前 | 参数合法性、权限检查、注入检测 |
| **Output** | 工具返回后 | 结果大小、敏感信息、错误码 |
| **Tool** | 工具选择时 | 工具是否在允许列表、是否需要确认 |

在Hermes中的实践：terminal命令检查dangerous关键词，文件输出检查大小。

## 第九章：Handoffs-as-Tools⭐ 第二轮学习新增

> 来源：OpenAI Agents SDK — Handoffs建模为工具

**核心理念：Agent间任务转移不是特殊指令，而是一个标准工具调用（transfer_to_xxx）。**

Input Filters在Handoff时过滤输入减少token。在Hermes中，delegate_task的context参数就是Input Filter。

## 第十章：Checkpointer检查点⭐ 第二轮学习新增

> 来源：LangGraph — Checkpointers故障恢复

**核心理念：长任务定期保存状态，失败时从检查点恢复。** 在Hermes中用todo跟踪进度，每个completed项就是一个检查点。

## 第十一章：单写入者+多读者模式⭐ 第六轮学习新增

> 来源：Cognition Devin — $260亿估值AI软件工程师

**Devin核心架构：单写入者+多读者的多Agent模式。** 与我们的Harness层"单一写入者"公理完全一致。

## 第十二章：专用小模型替代通用大模型⭐ 第六轮学习新增

> 来源：SWE-Check — 专用RL小模型做代码审查，快10x

**对Harness层启示：** 工具选择时，优先专用工具而非通用大模型。

## 记忆锚点

```
三条公理：
  单一写入者 — 同一目标同一时刻只有一个写入者
  只读强制   — 默认只读，写入必须显式声明和验证
  完成观察   — 完成是工具验证的事实，不是 Agent 的判断

五个工具组合模式：
  RAP — 读取-分析-报告（零副作用）
  RGW — 读取-生成-写入（单文件变更）
  REV — 读取-执行-验证（命令执行）
  PR-SW — 并行读取-串行写入（多源合并）
  DRC — 委托-审查-提交（子 Agent 编排）

四条反模式禁令：
  不先写后读
  不并行写入
  不主观完成
  不读写交织
```

**Harness 层的价值不在于限制 Agent 的能力，而在于让 Agent 的行为可预测、可验证、可信赖。**
