# Model Tier Advisor（模型档位顾问）

**一份 87 token 的规则，让你的 Agent 提醒你：当前这个模型，是不是选错了。**

[English](./README.md) · [方法论](./METHODOLOGY.md) · [效果示例](./examples/showcase.md)

---

## 解决什么问题

每天都在发生两种浪费：

- **大材小用** —— 格式化文件、改变量名、查 API 用法，却跑在旗舰模型上。为用不上的能力付钱。
- **小马拉车** —— 架构设计、跨服务排查交给经济模型。这个更隐蔽也更贵：**能力不足的模型不会报错，它会自信地给你一个看似合理但错的方案**，然后你花几倍时间才发现它错在哪。

## 这是什么

一个规则文件，丢进项目里。Agent 接到任务先判难度，发现当前模型明显错配时用一行提醒你，然后照常干活。

```
轻=格式/单文件/查资料/翻译 · 中=多文件/开发/重构/已知bug · 重=架构/跨系统/审计/模糊
模型明显错配时一行提示「换X档+理由」，然后照常干活。不追问、不中断、每会话一次。
```

**不自动切换、不追问、不打断。**

## 实测开销

一个宣称省 token 的工具，必须先把自己的开销交代清楚。用 cl100k 分词器实测：

| 文件 | token |
| --- | --- |
| `AGENTS.md`（英文版） | **87** |
| `AGENTS.zh-CN.md`（中文版） | **118** |

这就是全部的常驻开销。完整账本见 [METHODOLOGY.md](./METHODOLOGY.md) —— 即使按最悲观的 5% 任务错配率算，收益仍是成本的约 8 倍。

## 安装

复制一个文件到项目根目录：

| 你的工具 | 怎么做 |
| --- | --- |
| Claude Code | 复制 `AGENTS.md` → `CLAUDE.md` |
| Cursor | 内容粘进 `.cursorrules` |
| Codex | 直接复制 `AGENTS.md`（原生支持） |
| WorkBuddy | 复制 `AGENTS.md` 到工作区根目录 |
| 其他 | 粘进 system prompt |

可选：一并复制 `rules/`（打分表 + 模型映射）。按需加载，不占常驻上下文。

## 三档速览

| 档位 | 典型任务 |
| --- | --- |
| **轻** | 格式化、单文件小改、查资料、翻译、一次性脚本 |
| **中** | 多文件改动、功能开发、模块重构、修已知 bug |
| **重** | 架构设计、跨系统排查、性能/安全审计、需求模糊 |

打分表 → [`rules/tier-table.md`](./rules/tier-table.md)
模型映射 → [`rules/models.md`](./rules/models.md)

## 为什么不做自动切换

自动路由的方案已经很多 —— [RouteLLM](https://arxiv.org/abs/2406.18665)、[FrugalGPT](https://arxiv.org/abs/2305.05176)，以及 `claude-code-token-save` 这类工具专属项目。它们的前提都是**系统能自己换模型**。

本项目的场景是**人在 GUI 或 IDE 里手动选模型**，此时唯一能做的就是**提醒人**。

代价是强制力弱；收益是零依赖、零改造、任何工具都能用。

## 跨厂商：价差有多大

- 最便宜的轻档：DeepSeek V4 Flash，$0.06/M
- 最贵的重档：Claude Fable 5，$14.44/M

**相差约 240 倍。**

而单一厂商**内部**的档位价差通常只有 5–20 倍。这就是为什么必须做跨厂商，而不是只做某一家。

⚠️ 但要特别注意：**买了某家的 Coding Plan，你就只能在它内部选模型了。**
套餐选择会直接决定你还能不能享受跨厂商价差，详见 [`guides/subscription-choice.md`](./guides/subscription-choice.md)。

## 可选模块

| 模块 | 开销 | 增加什么 |
| --- | --- | --- |
| `modules/skill-expert.md` | 1,062 t | 该不该找现成 Skill？要不要加角色 / 专家团？**默认关闭** |
| `guides/subscription-choice.md` | 0 | Coding Plan vs Token Plan 怎么选。只给人读，永不进上下文 |

## 文件结构

```
AGENTS.md                      L0 核心     87 t  ← 唯一进上下文的文件
AGENTS.zh-CN.md                中文版     118 t
rules/tier-table.md            打分表     869 t  按需加载
rules/models.md                模型映射 1,075 t  按需加载
modules/skill-expert.md        可选模块 1,062 t  默认关闭
guides/subscription-choice.md  只给人读 —— 永不进上下文
METHODOLOGY.md                 只给人读 —— 永不进上下文
examples/showcase.md           只给人读
```

用 cl100k 分词器实测。除 L0 之外，不主动加载就不花一分钱。

## 常见问题

**会不会很烦？**
每会话最多一次，且只在明显错配时才说。说一句「别提了」就本会话永久闭嘴。

**判错了怎么办？**
什么都不会发生。它只是建议，不是闸门。Agent 不会停下来，也不会回头自我纠正、反复改口。

**我换不了模型（公司统一配置 / 买了固定套餐）**
规则会退而求其次，给同模型内的替代方案：降低推理强度、把任务拆小。

**模型名会不会过时？**
一定会。所以档位用**能力门槛**定义（永不过期），模型名单独放在标了 `valid_until` 的文件里。过期后只报档位，不报模型名。

## 许可

MIT · by [libra-eyes](https://github.com/libra-sys)
