# Model Tier Advisor

**An 87-token rule that tells your coding agent when it's on the wrong model.**

[中文版](./README.zh-CN.md) · [Methodology](./METHODOLOGY.md) · [Examples](./examples/showcase.md)

---

## The problem

Two kinds of waste happen every day:

- **Overkill** — formatting files, renaming variables, looking up an API on a flagship model. You pay for capacity you never use.
- **Underkill** — architecture decisions and cross-service debugging on a budget model. This one is worse: **an underpowered model doesn't fail loudly, it fails confidently.** You get a plausible wrong answer, then spend hours discovering it was plausible.

## What this is

One rule file you drop into your project. Your agent judges task difficulty, and when the current model clearly mismatches, says so in **one line** — then keeps working.

```
Tier check: light=formatting/single-file/lookup · mid=multi-file/feature/refactor · heavy=architecture/cross-system/audit

If model clearly mismatches, one line: "suggest tier X — reason", then proceed.
```

**No auto-switching. No questions. No interruption. Once per session.**

## Measured cost

A token-saving tool has to account for its own cost. Measured with `cl100k_base`:

| File | Tokens |
| --- | --- |
| `AGENTS.md` (English) | **87** |
| `AGENTS.zh-CN.md` (Chinese) | **118** |

That is the entire per-session overhead. Full ledger in [METHODOLOGY.md](./METHODOLOGY.md) — even at a pessimistic 5% task-mismatch rate, savings exceed cost by roughly 8×.

## Install

Copy one file into your project root:

| Tool | What to do |
| --- | --- |
| Claude Code | copy `AGENTS.md` → `CLAUDE.md` |
| Cursor | copy contents into `.cursorrules` |
| Codex | copy `AGENTS.md` (native filename) |
| WorkBuddy | copy `AGENTS.md` to workspace root |
| Anything else | paste it into the system prompt |

Optional: also copy `rules/` for the scoring rubric and model mapping. These load on demand and never sit in context.

## Three tiers

| Tier | Typical tasks |
| --- | --- |
| **light** | formatting, single-file edits, lookup, translation, throwaway scripts |
| **mid** | multi-file changes, feature work, refactor, known-bug fixes |
| **heavy** | architecture, cross-system debugging, perf/security audit, vague requirements |

Scoring rubric → [`rules/tier-table.md`](./rules/tier-table.md)
Model mapping → [`rules/models.md`](./rules/models.md)

## Why not auto-switch?

Auto-routing already exists — [RouteLLM](https://arxiv.org/abs/2406.18665), [FrugalGPT](https://arxiv.org/abs/2305.05176), and tool-specific projects like `claude-code-token-save`. They all assume **the system can change the model**.

This project assumes **a human picks the model in a GUI or IDE**, and does the only thing possible in that setting: *tell the human*.

Tradeoff: weaker enforcement, but zero dependencies and works in every tool.

## Optional modules

| Module | Cost | Adds |
| --- | --- | --- |
| `modules/skill-expert.md` | 1,062 t | Should you grab an existing skill? Do you need multiple agent roles? Off by default. |
| `guides/subscription-choice.md` | 0 | Coding Plan vs Token Plan. Humans only, never enters context. |

## Files

```
AGENTS.md                      L0 core    87 t   ← the only file in context
AGENTS.zh-CN.md                Chinese   118 t
rules/tier-table.md            rubric    869 t   on demand
rules/models.md                mapping 1,075 t   on demand
modules/skill-expert.md        optional 1,062 t  off by default
guides/subscription-choice.md  humans only — never in context
METHODOLOGY.md                 humans only — never in context
examples/showcase.md           humans only
```

Measured with `cl100k_base`. Everything except L0 costs nothing until you ask for it.

## FAQ

**Won't this get annoying?**
At most once per session, only on clear mismatch. Say "just do it" and it stays silent for the rest of the session.

**What if it judges wrong?**
Nothing happens. It's a suggestion, not a gate. The agent never stops, never self-corrects, never re-litigates.

**I can't switch models (corporate config, locked subscription).**
The rule falls back to same-model alternatives: lower reasoning effort, or split the task into smaller pieces.

**Don't model names go stale?**
Constantly. That's why tiers are defined by *capability thresholds* (stable) while model names live in a separate file stamped `valid_until`. After expiry, report the tier only.

**Why is the English version cheaper than the Chinese one?**
Chinese tokenizes at roughly one token per character, so it costs ~36% more for identical content. See [METHODOLOGY.md](./METHODOLOGY.md) — in money the difference is cents, but it matters if you're budgeting context.

## License

MIT · by [libra-eyes](https://github.com/libra-sys)
