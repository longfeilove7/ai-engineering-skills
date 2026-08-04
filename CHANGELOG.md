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
