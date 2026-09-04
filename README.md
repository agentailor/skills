# Agentailor Skills

Hand-crafted skills for building AI agents — each one distilled from a technique used and proven in real Agentailor projects, not auto-generated.

These are **skills for agent builders**: drop them into a skill-aware coding agent (Claude Code, or any tool that reads the [AgentSkills spec](https://agentskills.io/specification)) so it can help you design agents to a consistent, production-tested standard. They pair with the deep-dive write-ups on the [Agentailor blog](https://blog.agentailor.com).

---

## Skills

| Skill | What it does |
|-------|--------------|
| [`agent-prompt-engineering`](agent-prompt-engineering/) | Design system prompts for autonomous, tool-using agents. Covers the principles, heuristics, thinking guidance, and evaluation strategy that make agents reliable in a loop — with worked prompt examples and the anti-patterns to avoid. Also audits a prompt you already have: what to delete once the model outgrew needing it, what must never be deleted, and how to verify a deletion when there's no eval harness. Sources: [The Art of Agent Prompting](https://blog.agentailor.com/blog/the-art-of-agent-prompting), [Your System Prompt Has a Shelf Life](https://blog.agentailor.com/blog/system-prompt-shelf-life). |
| [`tool-design`](tool-design/) | Design tools an AI agent can actually use — framework- and language-agnostic (MCP, LangChain/LangGraph, function-calling; TypeScript, Python, …). Five production-tested principles, a validation checklist, a two-layer testing approach (deterministic tests — unit, integration, whatever the tool warrants — for the contract, evals for the behavior), and worked examples across surfaces and languages. Source: [Writing Effective Tools for AI Agents](https://blog.agentailor.com/blog/writing-tools-for-ai-agents). |
| [`agent-eval-cases`](agent-eval-cases/) | Decide which agent behaviors are worth an eval case, then write those cases — harness-, framework-, and language-agnostic (Evalite, pydantic-evals, Braintrust, LangSmith, Google ADK, or a home-made runner). Elicits observed failures instead of inventing them, pushes each down to the cheapest layer that can catch it, groups what survives by defect class, pairs every case that pushes a behavior with one that bounds it, and picks graders by whether the assertion target has one spelling or many. Also covers reading the first red run: why a case that passes before you fixed anything is a weak test, and when to loosen a grader rather than the agent. Sources: [How to Write Your First AI Agent Evals](https://blog.agentailor.com/blog/first-eval-cases), [Building an Agent Eval Harness](https://blog.agentailor.com/blog/agent-eval-harness), [How to Write AI Agent Evals That Prove You Wrong](https://blog.agentailor.com/blog/how-to-write-ai-agent-evals). |

More skills will be added here as they're proven useful in practice.

---

## Install

Install with the [`skills`](https://github.com/vercel-labs/skills) CLI. It asks which coding agent to install into — Claude Code, Cursor, Codex, Copilot, Gemini, Antigravity, etc:

```bash
npx skills add agentailor/skills --skill tool-design
npx skills add agentailor/skills --skill agent-prompt-engineering
npx skills add agentailor/skills --skill agent-eval-cases
```

Install everything:

```bash
npx skills add agentailor/skills --skill '*'
```

Useful flags:

| Flag | What it does |
|------|--------------|
| `-a, --agent <agents>` | Pick the target agent(s) up front instead of being prompted; `'*'` installs to all detected agents. |
| `-g, --global` | Install user-level (every project) instead of into the current project. |
| `-l, --list` | List the skills in this repo without installing anything. |
| `--all` | Shorthand for `--skill '*' --agent '*' -y` — everything, everywhere, no prompts. |

Update to the latest version later, and see what's installed:

```bash
npx skills update
npx skills list
```

---

## What is a Skill?

Each skill is a directory with a `SKILL.md` following the [AgentSkills spec](https://agentskills.io/specification):

```yaml
---
name: skill-name          # lowercase, hyphens only, max 64 chars; matches the directory name
description: What this skill does and when to use it.   # max 1024 chars — this is the trigger
---
```

The Markdown body holds the instructions. Skills follow the standard progressive-disclosure convention: keep `SKILL.md` lean and move detailed material into a `references/` directory (optionally `scripts/` and `assets/`), loaded only when needed.

```
skills/
└── <skill-name>/
    ├── SKILL.md          # required: frontmatter + instructions
    ├── references/       # optional: deep-dive docs loaded on demand
    ├── scripts/          # optional: executable helpers
    └── assets/           # optional: templates and resources
```

We deliberately keep skills:

- **Framework-independent where the idea allows** — a good tool-design principle holds whether you use MCP, LangChain, or raw function calling. (Some skills are legitimately framework-specific; that's fine when the pattern itself is.)
- **Language-neutral where the idea allows** — the same design thinking maps onto TypeScript, Python, and beyond.
- **Behavior-first** — they describe what an agent should do, not how one library does it.

---

## How to Use

**In a coding agent:** [install](#install) a skill into your agent's skills folder and let it load the skill when the task matches the `description` — you don't invoke it by hand.

**As a reference:** read the `SKILL.md` and its `references/` directly — they stand on their own as guides.

---

## Related

- [AgentSkills Specification](https://agentskills.io/specification)
- [Agentailor blog](https://blog.agentailor.com) — the articles behind these skills
- [Agentailor](https://agentailor.com) — the hub for developers building AI agents
