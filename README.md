# Skills

[![skills.sh](https://skills.sh/b/Sov3rain/skills)](https://skills.sh/Sov3rain/skills)

A collection of reusable skills for coding agents (Antigravity, Claude Code, Codex, etc.).

## Available Skills

| Skill | Invocation | Description |
| --- | --- | --- |
| [`improve-logging`](./improve-logging/SKILL.md) | `/improve-logging` (manual) | Audits a codebase via a sub-agent to surface missing or poor logging, proposes a refactoring plan toward **wide events** (high cardinality, business context, tail sampling), and implements and verifies the strategy after explicit approval. Inspired by [loggingsucks.com](https://loggingsucks.com/). |

## Installation

Install all skills or a specific skill using [`skills`](https://skills.sh):

```bash
# Install all skills from this repository
npx skills@latest add Sov3rain/skills

# Install a specific skill
npx skills@latest add Sov3rain/skills --skill improve-logging

# Install globally
npx skills@latest add Sov3rain/skills -g
```

Or install from a local clone:

```bash
npx skills@latest add ~/Workspaces/skills
```

## Usage

### `/improve-logging`

1. Run `/improve-logging` in your agent (optionally specifying a scope, e.g. `/improve-logging focus on the checkout module`).
2. A sub-agent explores the codebase and inventories:
   - uninstrumented entry points and critical business flows,
   - scattered ("log chaos") or unstructured logs,
   - missing correlation (high cardinality) and business context (high dimensionality),
   - swallowed or vague errors,
   - sampling gaps.
3. The agent presents a prioritized **Markdown report** and proposed strategy, then waits for your **explicit approval**.
4. Once approved, the agent implements wide events idiomatically for your stack and verifies the changes with tests.
