# Agentailor Skills

Hand-crafted skills for building AI agents — each one distilled from a technique used and proven in real Agentailor projects, not auto-generated.

These are **skills for agent builders**: drop them into a skill-aware coding agent (Claude Code, or any tool that reads the [AgentSkills spec](https://agentskills.io/specification)) so it can help you design agents to a consistent, production-tested standard. They pair with the deep-dive write-ups on the [Agentailor blog](https://blog.agentailor.com).

---

## Skills

| Skill | What it does |
|-------|--------------|
| [`agent-prompt-engineering`](agent-prompt-engineering/) | Design system prompts for autonomous, tool-using agents. Covers the principles, heuristics, thinking guidance, and evaluation strategy that make agents reliable in a loop — with worked prompt examples and the anti-patterns to avoid. Source: [The Art of Agent Prompting](https://blog.agentailor.com/blog/the-art-of-agent-prompting). |
| [`tool-design`](tool-design/) | Design tools an AI agent can actually use — framework- and language-agnostic (MCP, LangChain/LangGraph, function-calling; TypeScript, Python, …). Five production-tested principles, a validation checklist, and worked examples across surfaces and languages. Source: [Writing Effective Tools for AI Agents](https://blog.agentailor.com/blog/writing-tools-for-ai-agents). |

More skills will be added here as they're proven useful in practice.

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

**In a coding agent:** point a skill-aware agent (e.g. Claude Code) at this repo, or copy a skill directory into your agent's skills folder, and let it load the skill when the task matches the `description`.

**As a reference:** read the `SKILL.md` and its `references/` directly — they stand on their own as guides.

---

## Related

- [AgentSkills Specification](https://agentskills.io/specification)
- [Agentailor blog](https://blog.agentailor.com) — the articles behind these skills
- [Agentailor](https://agentailor.com) — the hub for developers building AI agents
