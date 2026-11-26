# AI Agents and Tools — Understanding the Fundamentals

## What is a "tool"?

A **tool is simply a capability that you expose to an AI agent**.

The tool is usually an ordinary piece of software you wrote.

For example, your portfolio application might have something like:

```python
def get_assets():
    return [
        {"name": "HDFC FD", "value": 500000},
        {"name": "Nifty 50 MF", "value": 350000},
        {"name": "PPF", "value": 800000}
    ]
```

You can expose that function to the LLM as a **tool**.

Conceptually:

```text
Your application
      │
      │  exposes
      ▼
┌─────────────────┐
│ get_assets()    │  ← Tool
└─────────────────┘
```

The LLM doesn't magically know how `get_assets()` works. **You provide the implementation.**

---

## Who creates tools?

**You do.**

Or another developer/company can provide them.

There are essentially three sources.

### 1. You create your own tools

For a portfolio project:

```python
@tool
def get_assets():
    """Get all assets in the user's portfolio."""
    ...
```

You decide:

- What the tool does
- What inputs it accepts
- What it returns
- What permissions it has

### 2. Libraries/frameworks provide tools

LangChain itself provides integrations/tools for things like:

- Web search
- SQL databases
- Vector databases
- APIs
- File systems
- etc.

You configure and expose those to your agent.

### 3. External services expose capabilities

For example, you might have APIs:

```text
GET /portfolio/assets
GET /portfolio/transactions
GET /portfolio/returns
```

You can wrap those APIs as agent tools.

So a tool doesn't have to be a Python function. It can ultimately be **anything your agent can invoke through a defined interface**.

---

## What does the agent actually do with tools?

Suppose you ask:

> "How much do I have invested in fixed-income assets?"

The LLM receives a description of its available tools:

```text
Tool: get_assets

Description:
Returns the user's portfolio assets.

Parameters:
None
```

The LLM reasons:

> "I need the user's assets before I can answer."

It requests:

```text
get_assets()
```

Your application executes it.

It returns:

```json
[
  {"name": "HDFC FD", "type": "FD", "value": 500000},
  {"name": "Nifty 50", "type": "Mutual Fund", "value": 350000},
  {"name": "PPF", "type": "PPF", "value": 800000}
]
```

The LLM then processes that result and answers.

The flow is:

```text
USER
 │
 │ "How much is in fixed income?"
 ▼
LLM
 │
 │ decides it needs portfolio data
 ▼
TOOL
 │
 │ get_assets()
 ▼
YOUR APPLICATION
 │
 │ returns data
 ▼
LLM
 │
 │ analyzes data
 ▼
ANSWER
```

---

## Are agents entirely dependent on tools?

**No.**

An LLM can reason and answer questions **without tools**.

For example:

> "What is the difference between an FD and a bond?"

No tool is necessary.

The model already has the knowledge required.

But consider:

> "How much money do I currently have in FDs?"

The LLM **cannot know this** unless you provide the information somehow.

That's where tools come in.

### A useful mental model

**LLM = brain**

**Tools = senses + hands**

The brain can reason, but without access to the outside world it can't:

- Look up current information
- Read your database
- Execute code
- Query an API
- Modify a file
- Send an email
- Create a GitHub PR
- Check your portfolio

Tools give it those capabilities.

---

## How does an agent differ from a chatbot?

### Normal chatbot

```text
User
 ↓
LLM
 ↓
Answer
```

### Agent

```text
                 ┌─────────────┐
                 │     LLM     │
                 └──────┬──────┘
                        │
                "What should I do?"
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           Tool A    Tool B     Tool C
              │         │         │
              └─────────┼─────────┘
                        ▼
                    Tool result
                        │
                        ▼
                       LLM
                        │
                 "What next?"
                        │
                       ...
```

The agent can **choose which tools to use and in what sequence**.

That's the core idea.

---

## Your portfolio project as an example

Imagine we give the agent these tools:

```text
get_assets()
get_liabilities()
get_transactions()
get_upcoming_payments()
get_maturing_investments()
calculate_returns()
calculate_net_worth()
```

Now ask:

> "Will I have enough liquidity for my upcoming obligations?"

There isn't necessarily one tool called:

```text
answer_liquidity_question()
```

Instead, the agent might reason:

```text
I need:
    ↓
Upcoming obligations
    ↓
get_upcoming_payments()

Available liquid assets
    ↓
get_assets()

Liabilities
    ↓
get_liabilities()

Compare them
    ↓
Answer
```

**That's agentic behaviour.**

---

## Don't create a tool for every possible question

Bad design:

```text
answer_portfolio_question()
answer_fd_question()
answer_networth_question()
answer_liquidity_question()
answer_mutualfund_question()
```

You're essentially hardcoding the intelligence into tools.

Better:

```text
get_assets()
get_liabilities()
get_transactions()
get_payments()
```

Give the LLM **primitive capabilities** and let it combine them.

That's where the intelligence lives.

---

## Tools don't necessarily mean "functions"

A tool could be:

```text
Python function
       ↓
REST API
       ↓
SQL query
       ↓
Vector search
       ↓
Web search
       ↓
Another AI agent
       ↓
External SaaS API
       ↓
Operating-system command
```

For example:

```text
Agent
 │
 ├── get_portfolio()       → your code
 ├── search_documents()    → vector DB
 ├── get_stock_price()     → external API
 ├── calculate_return()    → Python
 └── get_market_news()     → web search
```

The agent doesn't really care **how** the capability is implemented.

It cares about the **tool's description and interface**.

---

## Where LangChain fits

When we build a project, we'll have something conceptually like:

```python
@tool
def get_assets() -> list:
    """Return the user's investment assets."""
    ...
```

LangChain takes that function and gives the LLM a structured description of it.

The LLM essentially sees:

```text
You have a tool called get_assets.

Purpose:
Return the user's investment assets.

Arguments:
None.
```

Then the model can decide:

> "I should call `get_assets`."

LangChain handles much of the machinery around that interaction.

---

## Think of LangChain as orchestration

Don't think of LangChain as "the AI."

Think of it as the **orchestration layer between the LLM and the capabilities you've given it**.

```text
                  ┌───────────────┐
                  │     LLM       │
                  │   Reasoning   │
                  └───────┬───────┘
                          │
                    LangChain
                  /      |       \
                 /       |        \
          Portfolio    RAG       APIs
            Tools       │          │
              │         │          │
              ▼         ▼          ▼
           Your DB    Documents   Services
```

### LangGraph

LangGraph can later provide a more explicit way to control:

- Agent state
- Loops
- Branching
- Retries
- Multi-step workflows
- Human approval

---

## The key takeaway

A useful mental model is:

> **LLM = reasoning engine**  
> **Tools = capabilities/actions**  
> **Agent = LLM that can decide when and how to use those capabilities**  
> **LangChain = orchestration framework connecting the reasoning engine to those capabilities**  
> **LangGraph = framework for building more controlled, stateful agent workflows**

For the portfolio project, a good learning path is:

```text
One LLM
   ↓
One tool
   ↓
Multiple tools
   ↓
Agent chooses tools
   ↓
Multi-step reasoning
   ↓
RAG
   ↓
State / memory
   ↓
LangGraph
   ↓
Human-in-the-loop
   ↓
Evaluation + observability
```

This lets you learn what is actually happening under the hood instead of hiding everything behind framework abstractions.
