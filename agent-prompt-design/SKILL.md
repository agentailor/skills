---
name: agent-prompt-design
description: Guide users through designing well-structured prompts for AI agents using proven architecture patterns and core principles. Helps users create clear instructions, explicit heuristics, and effective evaluation strategies for production-ready agents.
---

# Agent Prompt Design

### Intent
Enable an agent to help users design effective, production-ready prompts for AI agents by applying structured architecture patterns and core prompting principles.

### When to Apply
Activate this skill when users:
- Ask how to write prompts for AI agents
- Need help structuring agent instructions or system prompts
- Want to improve existing agent prompts that aren't working well
- Request guidance on agent architecture or design patterns
- Ask about best practices for agent prompting
- Need help debugging agent behavior or tool usage
- Want to understand how to make agents more reliable

### Inputs
Required information:
- The agent's intended purpose or role
- What tools or capabilities the agent will have access to
- What tasks the agent should perform

Helpful context:
- Current prompt (if improving an existing agent)
- Specific problems or failures encountered
- Domain-specific requirements or constraints
- Success criteria or evaluation needs

### Instructions

#### 1. Guide Prompt Architecture

Explain that well-structured agent prompts have five essential components:

1. **Role definition** - Who is the agent and what's its purpose
2. **Dynamic content retrieval** - How to access relevant context
3. **Detailed instructions** - Step-by-step behavioral guidance
4. **Optional examples** - When helpful for complex tasks
5. **Repeated critical instructions** - For long prompts, repeat key rules at the end

Help the user structure their prompt following this architecture.

#### 2. Apply the "Start Simple" Principle

- Guide users to begin with straightforward prompts rather than trying to perfect everything upfront
- Recommend drafting initial versions quickly, then iterating based on testing
- Suggest using AI tools to help draft the initial prompt structure
- Emphasize that refinement comes from real-world testing, not upfront speculation

#### 3. Apply Empathetic Design

Explain the core principle: "If a human cannot follow the instructions, neither can the agent."

Guide users to:
- Put themselves in the agent's position
- Simulate having only the described tools and context
- Check if they could actually follow the instructions as written
- Identify ambiguities by role-playing as the agent
- Clarify any steps that require assumptions or unclear decision-making

#### 4. Identify Explicit Heuristics Needed

Ask users to consider domain-specific decision-making rules that must be written explicitly:

- When something is irreversible vs. safe to try
- What "good enough" means in their context
- Resource limits and budgets (API calls, tokens, time)
- Priority ordering when goals conflict
- When to ask the user vs. make autonomous decisions

Help users articulate these heuristics clearly in the prompt.

#### 5. Address Strategic Tool Selection

When the agent has multiple similar tools:

- Suggest using clear, descriptive prefixes (e.g., `slack_search` vs `notion_search`)
- Guide users to provide explicit guidance on which tool to use when
- Help them explain trade-offs between tool options
- Clarify when tools overlap and how to choose between them

#### 6. Guide Reasoning Approach

Direct users to include instructions that:
- Ask the agent to plan before acting on complex tasks
- Encourage reflection after retrieving data
- Request explanations of reasoning for important decisions

**Important**: Warn against prescribing exact thought patterns. Explain that modern models benefit from interleaved thinking between tool calls, not rigid "think, then act" sequences.

#### 7. Manage Side Effects

Help users identify and address potential issues:

- **Unintended loops**: Guide users to specify when the agent should stop
- **Excessive tool usage**: Set clear budgets or limits
- **Irreversible actions**: Require confirmation or add safety checks
- **Runaway costs**: Define resource constraints
- **Perfectionism**: Allow agents to achieve "good enough" results

Ask users what actions their agent could take that can't be undone, and ensure the prompt addresses these.

#### 8. Design Evaluation Strategy

Guide users to create 3-5 realistic test queries before considering the prompt complete.

Help them identify success criteria:
- Does the agent use the right tools?
- Does it know when to stop?
- Are the outputs useful?
- Does it handle edge cases appropriately?

Recommend running these tests manually before building automation.

#### 9. Frame Iteration Process

Explain that effective agent prompting is about clear communication, not clever tricks.

Set expectations for iteration:
- Test with real scenarios
- Identify specific failures
- Refine based on observed behavior
- Don't try to anticipate every edge case upfront

### Failure Modes

**Overcomplicating initial versions**: Agents often start with overly complex prompts trying to handle every edge case. Remind users to start simple and iterate.

**Assuming implicit knowledge**: Users often forget to specify domain-specific heuristics they take for granted. Actively probe for decision-making rules that need explicit definition.

**Rigid reasoning structures**: Avoid suggesting strict "Chain of Thought" patterns that force thinking into a single step. Modern agents work better with flexible reasoning.

**Missing evaluation criteria**: Users may not know how to tell if their prompt works. Always help them define specific test queries and success criteria.

**Ignoring the empathy test**: If the user's instructions are unclear to you (the agent helping them), they'll be unclear to the target agent. Surface this immediately.

**Perfectionist expectations**: Users may set unrealistic standards. Guide them toward "good enough" goals with clear stopping conditions.

### Evaluation

The skill was applied successfully if:

1. The user has a complete prompt with all five architectural components
2. Domain-specific heuristics are explicitly stated, not assumed
3. The user can articulate 3-5 realistic test queries
4. Clear success criteria exist (right tools, knows when to stop, useful outputs)
5. The prompt passes the empathy test - a human could follow the instructions with the same tools and context
6. Side effects and irreversible actions are identified and addressed
7. The user understands the iteration process and isn't trying to perfect everything upfront

The agent should confirm these elements are in place before concluding the skill application.