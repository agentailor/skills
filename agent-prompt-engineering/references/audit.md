# Auditing an Existing Prompt

Most prompt engineering advice assumes a blank page. This file assumes the opposite: a prompt that already works, has been in production for months, and has grown a paragraph at a time — one incident, one edge case, one "just add a line telling it not to do that" at a time.

That prompt is not badly written. It is **over-fitted to a model that no longer exists.** Half of it was compensating for behavior the model you run today produces on its own, and every one of those lines is still costing you tokens, still constraining reasoning, and still capable of contradicting the lines added after it.

A maintenance pass is not a rewrite. It is **re-fitting instruction density to the model you actually run.**

The scale this can reach: Anthropic [removed over 80% of Claude Code's system prompt](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) for Claude Opus 5 and Claude Fable 5 — with, in their words, *no measurable loss on our coding evaluations*. Those four words are the entire warrant for the number. Note what the deletions were: not corrections. The instructions were load-bearing when written; the model outgrew needing them.

## Table of Contents

1. [Step 0: evals are a prerequisite](#step-0-evals-are-a-prerequisite)
2. [When to run a pass](#when-to-run-a-pass)
3. [Finding candidates](#finding-candidates)
4. [What not to delete](#what-not-to-delete)
5. [Running the pass](#running-the-pass)

---

## Step 0: evals are a prerequisite

**Do not start without a baseline.** This is not a preamble to skip; it is the step that makes every other step meaningful.

Deleting instructions from a prompt is invisible work. The prompt gets shorter, nothing obviously breaks, and the failure — if there is one — shows up weeks later on the specific input the deleted line existed to handle. Without a baseline you cannot tell "I removed 60% of the prompt and quality held" from "I removed 60% of the prompt and quality dropped on the 8% of traffic I don't test."

Strip *no measurable loss on our coding evaluations* from the 80% claim and what remains is "we deleted most of our prompt and it seemed fine" — which is worth nothing. The number is only meaningful because of the measurement attached to it.

Deleting on intuition is also how prompts get bloated in the first place. Something breaks, you add a line, nobody ever checks whether it helped, and eighteen months later nobody knows which ten of the hundred lines are doing the work.

The bar is low, and it is the same bar as the [3-Test Rule](../SKILL.md#the-3-test-rule) in the main skill:

1. Collect 5–15 realistic cases — happy paths, the edge cases you remember, and any input that caused a past incident. **The inputs that motivated the instructions you're about to delete belong in this set**, because they are exactly what a deletion would regress.
2. Run them against the current prompt and record the outputs. That's the baseline.
3. Make your deletions.
4. Re-run the same cases. Compare.

Keep the cases fixed across iterations. If quality is graded subjectively, grade the before and after in the same sitting — a week apart, your standards will have moved.

**A passing case is evidence only if you know why it passed.** Green is not the same as correct: a case can pass for a reason unrelated to the instruction you deleted, which makes it useless as protection for that deletion. Before you trust a green run, check that each case would actually have gone red had the deleted line mattered.

There's a second reason to write the cases first. They're also how you discover that the thing you were about to fix was never broken — a deletion candidate that turns out to be load-bearing, or an instruction added long ago for a bug that no longer exists (or never did).

If you cannot build even this, you can still audit — but be honest that you are trading measured confidence for an assumption, and delete conservatively (start with duplication, which is the safest category, and leave the judgment calls).

---

## When to run a pass

- **You changed the model.** Both directions — see [the model-tier rule](#the-model-tier-rule) below.
- **The prompt has grown past the point anyone reads it end to end.** If nobody on the team can say what's in it, nobody can tell you whether a new line contradicts an old one.
- **You're about to add another instruction.** The cheapest moment to remove three stale lines is while you're already in the file.
- **The agent is behaving oddly in a way that smells like over-constraint** — refusing reasonable requests, looping, over-qualifying, applying a rule in a context it was never meant for. Adding a line to correct that is the reflex; the fix is often deleting the line that caused it.

---

## Finding candidates

Six patterns to search for. Each one is a *candidate* for deletion, not a verdict — run every hit past [What not to delete](#what-not-to-delete) before cutting.

### 1. Instructions stated twice

The same rule in the system prompt *and* in a tool description. This happens naturally: someone adds a constraint to the prompt, someone else later writes it into the tool where it belongs, and neither deletes the other.

**Keep the tool-description copy.** It sits closest to the decision it governs, it's loaded exactly when the tool is a candidate, and it stays correct if the tool moves to another agent. The system-prompt copy is the redundant one.

The same logic applies within the prompt itself — a constraint stated in the role section and restated in a "critical reminders" block at the end. See [Anti-Pattern 12](anti-patterns.md#anti-pattern-12-excessive-repetition).

### 2. Rules that are now judgment calls

Hard caps and absolute prohibitions written to stop a behavior the current model may no longer exhibit. Anthropic's own before/after is the clearest illustration. The old instruction:

> In code: default to writing no comments. Never write multi-paragraph docstrings or multi-line comment blocks, one short line max.

The new one:

> Write code that reads like the surrounding code: match its comment density, naming, and idiom.

The first is a **rule** — it exists because a weaker model, left alone, wrote too many comments, and a hard cap was the fix that worked. The second is **judgment**: it assumes the model can read a file, infer the local convention, and match it. That assumption was false in 2024 and is true enough now to ship.

That's the shape of every item in this list. Not "we were wrong," but *the model got good enough that the workaround costs more than it returns.*

The same applies to enumerated decisions:

```
❌ If the user asks about billing, respond formally. If the user asks about a bug,
   respond casually. If the user seems frustrated, apologize first and skip the
   pleasantries. If the user is a returning customer, reference their history.
```

Four rules encoding one instruction: *match the register to the situation.* A capable model does this unprompted, and does it better — because it handles the fifth case, the one nobody wrote a rule for.

The tell: a hard cap aimed at a behavior you haven't actually seen recently, or a list of `if <situation>, do <obvious thing>` clauses where a person reading them would say "well, obviously." Remove it and see if the eval moves.

### 3. Examples a better schema would replace

Few-shot examples pinned to the prompt to teach an output format that a **structured output schema, an enum, or a typed tool parameter** now enforces directly.

If you're spending 40 lines showing the model what a valid category string looks like, and the tool could take `category: "dining" | "groceries" | "transport"`, the examples are doing a job the type system does better and for free. Move the constraint into the schema and delete the examples.

The question to ask is Anthropic's: *what parameters does the model have, and how can they be more expressive?* An enum of `pending | in_progress | completed` teaches the usage pattern without a single example — **the parameter is the instruction**, and unlike an example it cannot drift out of sync with the code.

This is the constructive half of [Anti-Pattern 1](anti-patterns.md#anti-pattern-1-over-prescriptive-few-shot-examples): don't just delete the prescriptive examples, ask what mechanism should have been carrying that constraint.

### 4. Situational detail that belongs elsewhere

Long passages covering a specific workflow, an occasional integration, or a rare task type — loaded into every single request, relevant to a small fraction of them.

This is what **skills and reference files exist for.** A prompt that inlines everything front-loads the whole surface area of the agent's job into every call. Move the passage into a skill or a reference document, leave a one-line pointer, and let it load when it's needed.

If your prompt has section headers, the sections nobody's request ever touches are the candidates.

### 5. Over-constrained heuristics

Bounds tightened past the point of usefulness, and unbounded instructions with no exit condition. The classic:

```
❌ Keep searching until you find the highest quality possible source
```

There is no state in which that instruction is satisfied, so the agent searches until it runs out of context. See [Prepare for Side Effects](../SKILL.md#6-prepare-for-side-effects) for the general shape and [Anti-Pattern 2](anti-patterns.md#anti-pattern-2-the-perfectionism-loop) for the fix.

In an audit, look for two specific things: **superlatives with no stopping rule** (`best`, `all possible`, `completely`, `every`), and **numeric bounds that were set for a weaker model** — a 3-tool-call cap added when the agent used to thrash is now just a ceiling on a model that would have planned its way to the answer in five.

### 6. Density mismatched to the model tier

The catch-all: instruction that exists to compensate for a capability the current model has. Formatting scaffolds, forced reasoning structures, explicit "think step by step" framing, restatements of default behavior. If you can't articulate what would go wrong without a line, that's the signal.

### The model-tier rule

Read the 80% result carefully and there's a scope condition inside it that mostly goes unstated: it was measured on **Claude Opus 5 and Claude Fable 5.** Haiku and Sonnet are not mentioned — **absent, not exempted.** This is frontier-model guidance, and on a smaller model much of it inverts.

That matters because plenty of production agents run smaller models *on purpose*: classification, routing, extraction, anything high-volume where latency and cost dominate and the task is narrow enough not to need a frontier model. On those, the instruction you're about to delete as an over-constraint may be the only thing keeping the task on rails. **Terseness that reads as trust on a frontier model reads as ambiguity to a model with less headroom to resolve it.**

So the pass is not "delete instructions." It is:

> **Re-fit instruction density to the model you are actually running.**

Which has the consequence people miss: a **downgrade triggers the audit exactly as much as an upgrade.** Route a step from Opus to Haiku to cut costs and you've changed the amount of judgment you can assume — the prompt that got leaner for the frontier model may need some of that scaffolding back.

The practical consequence: **treat prompt and model as a versioned pair.** A prompt is fitted to a model; changing either invalidates the fit. If you serve multiple tiers, expect their prompts to diverge rather than trying to maintain one that serves both.

> **On sourcing:** this tier argument is our reading, not Anthropic's published position. Their [prompting best practices](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices) note that newer models may need behaviors requested more explicitly — but that's a point about *generations*, not tiers. We've found no Anthropic guidance stating that Haiku needs denser instruction than Opus. The mechanism above is the argument; treat it as reasoning to test against your own evals rather than a vendor claim.

---

## What not to delete

**This half matters more than the deletions.** The failure mode of the entire exercise is not "I didn't trim enough." It is removing something load-bearing and finding out in production, on the one input that instruction was quietly handling.

The rule that resolves every case:

> **Delete what the model can infer. Keep what only you know.**

A model can infer conventions, formats, reasonable defaults, tone, and how to break down a task. It cannot infer facts about *your* product, *your* environment, or *your* domain that exist nowhere in its training data.

Three categories always stay:

**Product invariants.** Rules that are true because you decided they are true, not because they're generally sensible. "Never move money without explicit human approval." "Every capability goes through the same approval gate." "Never quote a price without checking the live rate." A model will not derive your product's non-negotiables from first principles, and the ones that read as *obviously good practice* are the most dangerous to cut — obviousness is exactly what makes them look redundant.

**Non-obvious facts about the harness or environment.** Which tool returns stale data. Which API is rate-limited and how the agent should back off. That one integration returns `200` with an error in the body. That the deploy tool is irreversible. This is information that exists only in your system, and no amount of model capability substitutes for it.

**Domain conventions.** How your industry, your team, or your customers use a term that means something else elsewhere. What your users mean by "closed," "settled," "active." Jargon and local convention are precisely what a general-purpose model will get plausibly and confidently wrong.

### The test to apply

For each deletion candidate, ask both:

1. **Could a competent new hire, given the tools and no other context, infer this?** If yes, the model probably can too. If no — if they'd have to be *told* — it stays.
2. **What specific input breaks if this line is gone?** If you can name one, add it to your eval set before deleting and let the run decide. If you genuinely can't name one, that's evidence for deletion, not proof — which is what step 0 is for.

When both tests are ambiguous, keep the line. An unnecessary instruction costs tokens; a missing invariant costs an incident.

---

## Running the pass

1. **Build the baseline** ([step 0](#step-0-evals-are-a-prerequisite)). Include cases covering the instructions you expect to delete.
2. **Read the prompt end to end,** marking candidates against the [six patterns](#finding-candidates). Don't edit yet — reading first is what surfaces the duplicates and the contradictions.
3. **Screen every candidate** through [what not to delete](#what-not-to-delete). Anything that survives both tests is a keeper; put it back.
4. **Delete in batches, re-running evals between them.** Batches, not all at once and not one line at a time: a single cut makes a regression impossible to attribute, and line-by-line is slow and hides interaction effects — the cases where two stale instructions were cancelling each other out.
5. **Keep whatever the evals say to keep** — whatever your instinct said. The instinct that added the line is the same instinct that will defend it.
6. **On a regression, restore narrowly.** Find the specific case that broke and restore the specific instruction that handled it — usually rewritten tighter, since the original was often a broad rule aimed at a narrow problem. Don't revert the whole pass.
7. **Record the model and date at the top of the prompt.** The next person to open it needs to know what it was fitted to. A prompt without that is a prompt nobody dares to trim.

On a model change, run step 1 *on the new model with the prompt unchanged* — that baseline is what the deletions are measured against, not the old model's numbers.

The habit worth building runs counter to instinct: **on a model upgrade, the first move is to look for what to remove.** Then treat it as recurring maintenance rather than a one-off cleanup — the prompt starts drifting out of fit again the moment either side of the pair changes.

### When the prompt isn't the problem

One thing to watch for while you do this. If some behavior refuses to respond to prompt changes at all — you delete the instruction, you restore it, you reword it, and the behavior doesn't move — you may be fighting the **model's priors** rather than its instructions. No amount of editing fixes that, and you can lose an afternoon rewording a prompt that was never the cause.

Running the same eval suite across two model versions makes the difference visible quickly. A single-model suite hides it.

## Further reading

- [The new rules of context engineering for Claude 5 generation models](https://claude.com/blog/the-new-rules-of-context-engineering-for-claude-5-generation-models) — Anthropic. The source for the 80% result and the before/after quoted above.
- [Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic. The broader frame: curating the whole context window, not just the prompt.
- [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents) — Anthropic. The tool-description half of the duplication problem, and the case for expressive parameters over examples.
- [Your System Prompt Has a Shelf Life](https://blog.agentailor.com/blog/system-prompt-shelf-life) — the article this checklist is drawn from, including a worked audit of a real 100-line production prompt.
