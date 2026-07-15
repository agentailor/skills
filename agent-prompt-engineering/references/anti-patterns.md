# Agent Prompting Anti-Patterns

Common mistakes when prompting agents and how to fix them.

## Anti-Pattern 1: Over-Prescriptive Few-Shot Examples

### ❌ What NOT to do:

```
Here's how you should reason through financial questions:

Example 1:
User: "How much did I spend on groceries?"
Thinking: First, I need to identify this is a spending query. Then I need to determine the time period - since none was specified, I'll assume the current month. Next, I need to use the get_expenses_by_date_range tool with the category filter set to 'groceries'. After retrieving the data, I'll sum the amounts and present a clear answer.
Action: [calls get_expenses_by_date_range(start_date='2024-01-01', end_date='2024-01-31', category='groceries')]
Result: [data returned]
Response: "You spent $427.34 on groceries this month."

Always follow this exact reasoning pattern for all queries.
```

**Why this fails:**
- Limits the agent's reasoning capabilities
- Doesn't generalize to different scenarios
- Agent may try to match the example even when inappropriate
- Wastes context window tokens

### ✅ Better approach:

```
When analyzing spending queries:
1. Determine the time period (default to current month if unspecified)
2. Use get_expenses_by_date_range with appropriate parameters
3. Present clear summaries with relevant context

Adapt your approach based on query complexity and available data.
```

## Anti-Pattern 2: The Perfectionism Loop

### ❌ What NOT to do:

```
Always ensure all data is completely accurate and verified before responding.
Continue searching and cross-checking until you have perfect information.
Never settle for incomplete or uncertain data.
```

**Real-world consequence:**
Agent searches indefinitely until hitting context window limit, even when perfect data doesn't exist.

### ✅ Better approach:

```
Strive for accuracy while being pragmatic:
- For simple queries: 2-3 tool calls typically sufficient
- For complex analysis: Use up to 7-10 tool calls
- If you haven't found reliable information after 10 searches, acknowledge limitations
- Incomplete data is acceptable with appropriate disclaimers
```

## Anti-Pattern 3: Ambiguous Tool Selection

### ❌ What NOT to do:

```
Available tools:
- search_slack
- search_notion  
- search_github
- search_drive

Use the appropriate search tool based on the context.
```

**Why this fails:**
- Agent doesn't know which tool to use when
- May randomly pick or always default to one tool
- Wastes tool calls trying multiple options

### ✅ Better approach:

```
Available tools and when to use them:
- search_slack: For team communications, decisions, discussions
- search_notion: For documented processes, project specs, meeting notes
- search_github: For code examples, technical discussions, PR context
- search_drive: For formal documents, presentations, spreadsheets

Decision strategy:
- "How did we decide...": Start with search_slack
- "What's the process for...": Start with search_notion
- "How is X implemented...": Start with search_github
- "Where's the Q3 report...": Start with search_drive
```

## Anti-Pattern 4: Tool Name Collisions

### ❌ What NOT to do:

```
<!-- Multiple MCP servers all exposing "search" tool -->
Tools:
- search (from Slack server)
- search (from Notion server)
- search (from GitHub server)
```

**Why this fails:**
- Agent can't distinguish between tools
- May call wrong tool repeatedly
- Creates confusion in tool selection logic

### ✅ Better approach:

```
Tools with clear, unique names:
- slack_search
- notion_search  
- github_search

Always prefix tools with their domain/source when multiple servers provide similar functionality.
```

## Anti-Pattern 5: Missing Thinking Guidance

### ❌ What NOT to do:

```
You are a research assistant. Find information and provide answers.

[No guidance on planning, reflection, or decision-making]
```

**Why this fails:**
- Agent doesn't know when to stop searching
- May not verify information quality
- Misses opportunities to use reasoning capabilities effectively

### ✅ Better approach:

```
Before searching:
1. Plan: What type of question is this? What quality sources do I need?
2. Estimate: How many searches will this likely require?

After each search:
1. Reflect: Is this information relevant and high-quality?
2. Assess: Do I have enough to answer, or should I continue?
3. Verify: Should I cross-check this with another source?
```

## Anti-Pattern 6: Ignoring the Human-in-the-Loop

### ❌ What NOT to do:

```
When user requests budget changes, update the budget using update_budget() tool.
Process refunds when requested using process_refund() tool.
```

**Why this fails:**
- Takes irreversible actions without confirmation
- User loses control over important decisions
- Potential for costly mistakes

### ✅ Better approach:

```
For irreversible actions (budget changes, refunds, deletions):

1. Present a clear summary of the proposed action:
   - What will change
   - The impact/consequences
   - Any relevant context

2. Request explicit confirmation:
   "I can update your grocery budget from $500 to $600/month. This will affect 
    your monthly savings by $100. Should I proceed?"

3. Wait for clear "yes", "confirm", or "proceed" before executing

4. If response is unclear, ask clarifying questions
```

## Anti-Pattern 7: Vague Heuristics

### ❌ What NOT to do:

```
Be careful with sensitive information.
Don't make too many tool calls.
Try to provide good answers.
```

**Why this fails:**
- No actionable guidance
- Agent doesn't know what "careful" or "too many" means
- Vague instructions provide no decision-making framework

### ✅ Better approach:

```
Heuristics with specific thresholds:

Sensitive information:
- Never include passwords, API keys, or tokens in responses
- For PII requests, verify user identity first
- Redact credit card numbers except last 4 digits

Tool call budgets:
- Simple queries: 2-3 tool calls maximum
- Medium complexity: 5-7 tool calls
- Complex research: 10-15 tool calls, then assess if more needed

Quality standards:
- Prefer official documentation over blog posts
- Cross-reference controversial claims with 2-3 sources
- Acknowledge when information is uncertain or outdated
```

## Anti-Pattern 8: No Edge Case Handling

### ❌ What NOT to do:

```
Answer user questions about their expenses using the get_expenses tool.
Provide financial advice based on their data.
```

**Why this fails:**
- Doesn't address missing data scenarios
- No guidance for ambiguous requests
- Agent doesn't know how to handle failures

### ✅ Better approach:

```
When providing financial advice:

Normal flow:
1. Retrieve expense data
2. Analyze patterns
3. Provide recommendations

Edge cases:
- If expense data is missing: Acknowledge gap, provide advice with disclaimers
- If date range is ambiguous: Default to current month, mention the assumption
- If tool returns error: Try alternative approach or explain limitation
- If user asks about investments: Acknowledge this is outside scope, recommend professional advice
- If insufficient data for reliable advice: Clearly state limitations, offer to help gather more information
```

## Anti-Pattern 9: Assuming Context That Doesn't Exist

### ❌ What NOT to do:

```
Review the code for bugs and security issues.
Analyze the financial data and provide insights.
```

**Why this fails:**
- Agent doesn't know WHERE the code or data is
- No instructions on HOW to access it
- Missing tool usage guidance

### ✅ Better approach:

```
Review code for bugs and security issues:

1. Access the code using analyze_file(file_path) for each changed file
2. Run static analysis using run_linter(file_path) 
3. Check for common security patterns using security_scan(file_path)
4. Search for similar issues in codebase using search_codebase(pattern)

Financial data access:
1. User expense data: Use get_expenses_by_date_range(start, end, category)
2. Budget information: Use get_budget_status(category)
3. Historical trends: Use get_spending_trends(months_back)
```

## Anti-Pattern 10: Rigid Workflows for Variable Tasks

### ❌ What NOT to do:

```
For EVERY user question, follow these exact steps:
1. Search the knowledge base
2. Verify with web search
3. Cross-reference with documentation
4. Generate summary
5. Request user confirmation
6. Provide final answer

Never skip any step.
```

**Why this fails:**
- Wastes time on simple questions ("What's my account balance?")
- Creates frustrating user experience
- Inflexible workflow doesn't adapt to context

### ✅ Better approach:

```
Adapt workflow to question complexity:

Simple factual queries (account balance, order status):
- Single tool call, direct answer
- Skip verification steps

Medium complexity (troubleshooting, recommendations):
- 2-4 tool calls as needed
- Basic verification if information conflicts
- Provide answer with brief reasoning

Complex analysis (deep research, multi-step problems):
- Plan multi-step approach
- Use 5-10 tool calls strategically
- Verify critical information
- Provide comprehensive analysis
```

## Anti-Pattern 11: Ignoring Model Capabilities

### ❌ What NOT to do:

```
You must always respond in this format:
<thinking>
[Your detailed reasoning here]
</thinking>
<answer>
[Your response here]
</answer>

Follow this format exactly for all responses.
```

**Why this fails:**
- Modern models have native thinking capabilities
- Forced format may conflict with model's natural reasoning
- Wastes tokens on unnecessary structure

### ✅ Better approach:

```
Use your thinking process to plan before responding.
After retrieving information, reflect on completeness and quality.

[Let the model use its native reasoning capabilities naturally]
```

## Anti-Pattern 12: Excessive Repetition

### ❌ What NOT to do:

```
<!-- Repeated throughout prompt -->
Always verify information before responding.
Remember to always verify information before responding.
When answering questions, always verify information.
Never forget to verify information.
Verification of information is critical.
```

**Why this fails:**
- Wastes context window
- Doesn't improve reliability beyond first mention
- May confuse rather than reinforce

### ✅ Better approach:

```
<!-- State once clearly -->
Before responding:
1. Verify information quality
2. Cross-reference if claims seem uncertain
3. Acknowledge when information cannot be verified

<!-- Optional: Repeat ONLY most critical constraint at end -->
Remember: Verify information quality before responding.
```

## Summary: Key Principles to Remember

**DO:**
- Provide clear heuristics with specific thresholds
- Guide HOW to think, not WHAT to think
- Give explicit tool selection criteria
- Define clear boundaries and edge cases
- Use adaptive complexity based on task
- Request confirmation for irreversible actions

**DON'T:**
- Show prescriptive few-shot examples
- Create perfectionism loops
- Use ambiguous guidance
- Ignore edge cases
- Force rigid workflows
- Assume context that doesn't exist
- Repeat instructions excessively
