# Agent World — 具身化 LLM Agent 农场模拟

> "LLM reasons. Code decides what's allowed. Audit keeps it traceable."
> — Constraint Architecture, v1.5.0

一个**物理精确的农场世界**，由 DeepSeek LLM 作为农夫自主决策。4349 行物理引擎，6 层认知架构。

## 架构

```
物理世界 (agent_world_local.py, 4349 行)
  ├── 24h 连续时钟 · 天气 Markov 链 · GDD 积温
  ├── 土壤 NPK/pH/水分 · 14 种作物 · 孟德尔遗传
  ├── 5 种牲畜 · 5 级材料建造 · 工具升级链
  └── 经济系统 (累进税/贷款/合同/加工)
       │
       ▼ 感官编译层 (Phase E4)
  sensory_dictionary.json  →  SenseCompiler
  20 条确定性映射规则      →  自然语言观测
       │
       ▼ LLM 认知层
  agent-world-llm.py
  ├── MemGPT 风格三层记忆 (工作/短期/长期)
  ├── 感官感知 + 跨年度季节学习
  └── 自主决策: remember/recall/forget
       │
       ▼ 约束与审计 (Phase E5-E6)
  ├── 重要性评分器 (13 条规则, 日合成 + 季反思)
  ├── 紧急中断系统 (10 个触发器, P0/P1/P2 优先级)
  └── FarmEvent Schema v1.1 (57 种事件类型, JSONL 审计链)
```

## 快速启动

```bash
# 1. 仅需 requests
pip install requests

# 2. 设置 DeepSeek API key
export DEEPSEEK_KEY=$DEEPSEEK_KEY

# 3. 启动物理引擎
python agent_world_local.py

# 4. 启动 LLM Agent
python agent-world-llm.py
```

## 认知系统

| 层 | 文件 | 功能 |
|----|------|------|
| 感官编译 | `sensory_dictionary.json` + `sense_compiler.py` | 物理数据 → 确定性 NL |
| 事件 Schema | `vault_utils.py` | 57 种事件类型, 结构化 context/outcome |
| 重要性评分 | `importance_scorer.py` | 13 条规则, 日合成 + 季反思 |
| 中断检测 | `interrupt_system.py` | 10 个触发器, P0/P1/P2 优先级 |
| 记忆系统 | `agent-world-llm.py` | MemGPT 三层记忆 |

## 世界系统

- 24h 连续时钟 · 7 种天气 · 14 种作物 · 5 种牲畜
- 5 级材料建造 · 5 种工具 × 5 级升级链
- 孟德尔遗传 · 累进税 · 合同/贷款 · 食品加工
- 57 个可执行动作, 4349 行物理引擎

## 开发历程

4 个 Phase, 76+ 次提交, 从 if/else 到 6 层认知架构。
详见 [docs/agent-world-dev-journal.md](docs/agent-world-dev-journal.md).

## License

MIT © 2026 续仁舞 (Xu Renwu)
