# AI Engineering Skills — 5层架构 + 自我进化

基于stackwich的5层架构理念，为Hermes Agent打造的AI Engineering Skill体系，支持自我进化。

## 架构

```
ai-engineering（总调度，Pin住）
    ↓ 自动分析任务复杂度
    ├─ Prompt层 → prompt-engineering
    ├─ Context层 → context-engineering
    ├─ Harness层 → harness-engineering
    ├─ Loop层 → loop-engineering
    └─ Graph层 → graph-engineering（含3个Agent）
```

## 自我进化机制

```
执行任务 → 自我反思 → 存储学习 → 累积改进 → patch skill → 版本+1 → 推送GitHub
```

## 六个Skill

| Skill | 职责 | 层 |
|-------|------|---|
| **ai-engineering** | 总调度+进化引擎 | 入口 |
| **prompt-engineering** | 七要素prompt检查 | Prompt |
| **context-engineering** | 上下文管理、压缩、RAG | Context |
| **harness-engineering** | 工具编排、单写入者、只读 | Harness |
| **loop-engineering** | 迭代优化、失败上限、质量门控 | Loop |
| **graph-engineering** | 多Agent协作（advisor/executor/verifier） | Graph |

## 安装

```bash
git clone https://github.com/longfeilove7/ai-engineering-skills.git ~/.hermes/skills/productivity/ai-engineering-skills
```

## 灵感来源

- [stackwich](https://github.com/ovhirup/stackwich) — 5层架构理念
- [dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide)
- [Meirtz/Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering)

## License

MIT
