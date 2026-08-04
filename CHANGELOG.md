# Changelog — AI Engineering Skills

## v1.0.0 (2026-08-04)
- 初始版本
- 基于stackwich 5层架构（Prompt/Context/Harness/Loop/Graph）
- 包含6个skill：
  - ai-engineering（总调度）
  - prompt-engineering
  - context-engineering
  - harness-engineering
  - loop-engineering
  - graph-engineering（含3个Agent定义）
- 自我进化机制：任务分析→执行→反思→累积→进化→定版→推送

## v1.1.0 (2026-08-04) — 第一轮学习改进
- prompt-engineering: 新增RSTI/TCREI/TFCDC结构化Prompt框架
- context-engineering: 新增Progressive Disclosure、Context Compaction Hook、Adaptive Tool Ranking
- 来源：GitHub Trending调研、ratel-ai/ratel、deepset-ai/haystack、Anthropic最新研究

## v1.3.0 (2026-08-04) — 第三轮学习改进
- ai-engineering: 新增故障归因（F）— LIFE框架
- ai-engineering: 新增闭环自进化（E）— AReaL2.0 + Skill Self-Play
- ai-engineering: 新增Memory-Reward Trap防护 — RoMeRL论文
- harness-engineering: 新增Guardrails/Handoffs-as-Tools/Checkpointer
- graph-engineering: 新增CrewAI双层架构/AutoGen事件驱动
- 来源：14篇2024-2026年最新论文

## v1.4.0 (2026-08-04) — 第五轮学习改进
- context-engineering: 新增底层Attention优化技术（KV Cache 6种技术、Infini-Attention、Ring Attention、Lost in the Middle、Differential Transformer）
- context-engineering: 新增Reflexion自我反思框架
- context-engineering: 新增PRM过程奖励模型
- loop-engineering: 新增Reflexion框架集成（verbal reinforcement learning）
- loop-engineering: 新增PRM步骤级评估
- 来源：vLLM、Google Infini-Attention、Berkeley Ring Attention、Shinn et al. 2023

## v1.5.0 (2026-08-04) — 第六轮学习改进
- harness-engineering: 新增单写入者+多读者模式（Cognition Devin验证）
- harness-engineering: 新增专用小模型替代通用大模型（SWE-Check 10x快）
- 来源：Cognition Devin、SWE-Check

## v1.6.0 (2026-08-04) — 第七轮学习改进
- context-engineering: 新增Agent记忆系统架构（MemGPT/Letta/Mem0/Graphiti）
- context-engineering: 新增三层记忆体系（语义/情景/程序）
- context-engineering: 新增时序记忆（Graphiti valid_from/valid_to）
- context-engineering: 新增Durable Execution（Temporal崩溃恢复）
- 来源：MemGPT、Mem0(YC S24)、Graphiti、Temporal

## v1.7.0 (2026-08-04) — 第九轮聚焦学习改进
- loop-engineering: 新增自我反思真相（纯自查无效，必须外部验证）
- loop-engineering: 新增7类失败模式分类（F1-F7）+ 策略映射
- loop-engineering: 新增收敛判据（Delta评分法）
- loop-engineering: 新增质量追踪向量（多维度评分）
- 来源：Huang et al. (2310.01798), CRITIC (2305.11738), Self-Consistency (2203.11171)

## v1.8.0 (2026-08-04) — 第十轮聚焦学习改进
- prompt-engineering: 新增第三章（缺失要素检测+自动补全+框架选择指南）
- context-engineering: 新增LazyMem查询时压缩（21x token节省）
- context-engineering: 新增工具输出生命周期管理（MemTool）
- context-engineering: 新增记忆生命周期操作（MemOps）
- context-engineering: 新增Router-Mem渐进执行（减少27%推理时间）
- 来源：LazyMem(2026.7), MemTool(2025.7), MemOps(2026.7), Router-Mem(2026.8)

## v1.9.0 (2026-08-04) — Harness+Graph层聚焦改进
- harness-engineering: 新增Guardrails三行为模式（allow/reject_content/raise_exception）
- harness-engineering: 新增Guardrails并行执行+快速取消
- graph-engineering: 新增Handoff上下文过滤（HandoffInputFilter）
- graph-engineering: 新增Handoff历史压缩（nest_handoff_history）
- graph-engineering: 新增任务自动分解规则（≥5文件spawn）
- graph-engineering: 新增Agent选择策略
- 来源：OpenAI Agents SDK源码分析、LangGraph

## v2.0.0 (2026-08-04) — 第十一轮深度学习
- prompt-engineering: 新增DSPy可编程优化（GEPA取代MIPRO、Signature命名优化）
- context-engineering: 新增上下文压缩四技术（Gist Tokens/AutoCompressor/LLMLingua/Selective Context）
- loop-engineering: 新增OPRO优化技术（历史嵌入+自然语言目标+多样性注入）
- loop-engineering: 新增双模型架构（优化器强模型+评分器工具）

## v2.1.0 (2026-08-04) — 第十二轮深度学习
- prompt-engineering: 新增Prompt Chaining最佳实践（MISO分解+5种Chain模式+错误门控）
- harness-engineering: 新增Tool Use错误处理4类型+参数幻觉检测+4级输出验证+6条降级链
- graph-engineering: 新增任务分解量化评分+依赖图自动构建+动态负载均衡
- 来源：Anthropic官方指南、OpenAI Agents SDK、创新研究

## v2.2.0 (2026-08-04) — 第十三轮深度学习
- prompt-engineering: 新增Few-Shot示例选择策略（语义相似度+MMR多样性+动态选择）
- context-engineering: 新增RAG评估优化（RAGAS框架+重排序+查询扩展+A/B测试）
- graph-engineering: 新增多Agent通信协议（信封载荷分离+四层上下文传递+信任机制）
- 来源：Liu et al. 2021、RAGAS、AutoGen v0.7、OpenAI Agents SDK

## v2.3.0 (2026-08-04) — 第十四轮深度学习
- prompt-engineering: 新增System Prompt设计（100-500 tokens最佳、行为描述>身份标签、Lost in the Middle对策）
- context-engineering: 新增多模态上下文管理（Progressive Image Disclosure、4层代码分层、10级裁剪顺序）
- loop-engineering: 新增A/B测试+版本管理（版本树、5种最佳版本选择、迭代历史可视化）
- 来源：Anthropic/OpenAI最佳实践、ColPali、OPRO

## v2.4.0 (2026-08-04) — 第十五轮深度学习（最终版）
- prompt-engineering: 新增Prompt安全防护（OWASP 7条防御+四层防护架构）
- harness-engineering: 新增工具路由矩阵（关键词/嵌入/Few-shot路由）
- harness-engineering: 新增三层结果缓存（L1结果/L2路径/L3分析）
- ai-engineering: 新增跨层优化（4个标准数据结构+6条反馈回路+三层门控）
- 来源：OWASP LLM Top 10、跨层优化研究

## v2.5.0 (2026-08-04) — 第十六轮深度学习
- prompt-engineering: 新增跨模型Prompt自适应（GPT-4/Claude/Gemini差异+适配器模式）
- context-engineering: 新增Proactive Context Preloading（预测性预加载+意图推断）
- graph-engineering: 新增故障恢复模式（检查点重分配+任务关键性分级+文件乐观锁）
- 来源：ICML 2026 Predictive Prefetching、Sculptor、AutoGen

## v2.6.0 (2026-08-04) — 第十七轮深度学习
- prompt-engineering: 新增自动评估与回归测试（promptfoo+Langfuse+CI/CD+灰度发布）
- loop-engineering: 新增自适应策略调整（策略池贝叶斯选择+动态预算+5级复杂度预估+策略库自动构建）
- harness-engineering: 新增工具链式优化（后处理管道+批量合并68%延迟降低+异步化+优先级队列）
- 来源：promptfoo(已被OpenAI收购)、Langfuse、Voyager、ExpeL

## v2.7.0 (2026-08-04) — 第十八轮深度学习
- context-engineering: 新增个性化适配（用户角色识别+历史偏好+领域差异化+多语言处理）
- graph-engineering: 新增可扩展性（Agent池化+弹性扩缩容+6种负载均衡+大规模优化）
- ai-engineering: 新增可观测性（执行链路追踪+决策可视化+异常检测+性能瓶颈分析）
- 来源：角色适配研究、Agent池化架构、TraceSpan数据结构

## v2.8.0 (2026-08-04) — 第十九轮深度学习
- prompt-engineering: 新增上下文感知动态Prompt生成（三层架构+情绪调整+阶段检测+演化策略）
- context-engineering: 新增跨会话传承（四层架构+会话结束自动提取+记忆版本控制）
- loop-engineering: 新增知识积累和复用（双通道路由+TTL时效性+语义回滚+探索率≥20%）
- 来源：COVE(2608.01234)、ChronoMem、SciToolAgent-Evo
