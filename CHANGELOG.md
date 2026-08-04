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
