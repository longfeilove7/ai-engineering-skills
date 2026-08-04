# AI Engineering Skills — 5层架构

基于stackwich的5层架构理念，为Hermes Agent打造的AI Engineering Skill体系。

## 五层架构

| 层 | Skill | 职责 |
|---|-------|------|
| **Prompt** | prompt-engineering | 检查prompt七要素质量，提醒按要素写提示词 |
| **Context** | context-engineering | 上下文窗口管理、信息压缩、多轮对话维护、RAG策略 |
| **Harness** | harness-engineering | Agent编排策略、工具组合模式、单写入者原则、只读强制 |
| **Loop** | loop-engineering | 迭代优化、自我反思、失败循环上限（最多2次）、质量门控 |
| **Graph** | graph-engineering | 多Agent协作：advisor→executor→verifier完整闭环 |

## 学习路径

```
Prompt（写好提示词）
  → Context（管理上下文）
    → Harness（编排工具）
      → Loop（迭代优化）
        → Graph（多Agent协作）
```

## 安装

```bash
# 克隆到Hermes skills目录
git clone https://github.com/longfeilove7/ai-engineering-skills.git ~/.hermes/skills/productivity/ai-engineering-skills

# 或者单独安装某个skill
cp -r prompt-engineering ~/.hermes/skills/productivity/
```

## 灵感来源

- [stackwich](https://github.com/ovhirup/stackwich) — 5层架构理念
- [dair-ai/Prompt-Engineering-Guide](https://github.com/dair-ai/Prompt-Engineering-Guide) — Prompt Engineering指南
- [Meirtz/Awesome-Context-Engineering](https://github.com/Meirtz/Awesome-Context-Engineering) — Context Engineering综述

## License

MIT
