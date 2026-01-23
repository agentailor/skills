# Agentailor Skills

A curated collection of skills for building production-ready AI agents.

These skills enable AI agents to **assist humans in building better agents** — covering prompt design, tool creation, evaluation, and more. They follow the [AgentSkills specification](https://agentskills.io/specification) and are compatible with registries like [skills.sh](https://skills.sh/) and [smithery.ai](https://smithery.ai/).

---

## What is a Skill?

Each skill is defined by a `SKILL.md` file following the [AgentSkills spec](https://agentskills.io/specification):

### Required (per spec)

```yaml
---
name: skill-name          # lowercase, hyphens only, max 64 chars
description: What this skill does and when to use it.  # max 1024 chars
---
```

The markdown body contains the skill instructions.

### Agentailor Structure (our convention)

Skills in this collection use a structured format:

| Section | Purpose |
|---------|---------|
| **Intent** | What behavior this skill enables |
| **When to Apply** | Trigger conditions for activation |
| **Inputs** | Information the agent needs to gather |
| **Instructions** | Step-by-step guidance for the agent |
| **Failure Modes** | Common mistakes to avoid |
| **Evaluation** | How to verify correct behavior |

---

## Repository Structure

```
skills/
└── <skill-name>/
    ├── SKILL.md           # Required: skill definition
    ├── scripts/           # Optional: executable code
    ├── references/        # Optional: additional documentation
    └── assets/            # Optional: templates, resources
```

### Naming Constraints

- Lowercase letters, numbers, and hyphens only
- Must not start or end with a hyphen
- No consecutive hyphens (`--`)
- Directory name must match the `name` field in SKILL.md

---

## Skills in This Collection

These are **meta-skills** — they enable AI agents to help users build production-ready agents:

| Skill | Description |
|-------|-------------|
| `agent-prompt-design` | Design effective prompts for AI agents |
| `tool-design` | Create well-structured tools for AI agents |
| *(more coming)* | |

---

## How to Use

**For agent developers:** Reference these skills in your agent's system prompt or use a skill-aware framework.

**For AI agents:** When activated, follow the skill instructions to assist users with agent development tasks.

**For registries:** Skills are spec-compliant and can be indexed by [skills.sh](https://skills.sh/) or similar services.

---

## Philosophy

Skills are:
- **Model-agnostic** — work with any LLM
- **Framework-independent** — not tied to specific libraries
- **Implementation-neutral** — describe behavior, not code

The goal is to describe **what an agent should do**, not how a specific framework should do it.

---

## Contributing

Contributions welcome. To propose a new skill:

1. Create `<skill-name>/SKILL.md` following the spec
2. Include the Agentailor structure sections
3. Open a PR with a clear description

---

## Related

- [AgentSkills Specification](https://agentskills.io/specification)
- [Agentailor Blog](https://blog.agentailor.com)
- [skills.sh Registry](https://skills.sh/)