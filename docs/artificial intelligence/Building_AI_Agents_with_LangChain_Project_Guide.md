# Building AI Agents with LangChain — Project Guide

## 1. Why build an AI agent?

A useful AI agent is more than a chatbot that generates an answer.

A normal LLM interaction looks like:

```text
User
  ↓
LLM
  ↓
Answer
```

An agent can instead:

```text
User
  ↓
Agent
  ↓
Understand goal
  ↓
Decide what information/actions are needed
  ↓
Use tools
  ↓
Inspect results
  ↓
Decide what to do next
  ↓
Use more tools if necessary
  ↓
Produce result / take action
```

The key distinction is:

> **LLM = gives you an answer.**  
> **Agent = accomplishes a goal by taking actions.**

If ChatGPT can answer the question directly from the prompt and its existing knowledge, building an agent around it may add little value.

A stronger agent project involves a goal that requires interaction with external systems, multiple steps, state, decisions, retries, or actions.

---

# 2. What makes a good agent project?

A strong agent use case generally has several of these characteristics:

1. There is no single prompt that solves the whole problem.
2. The agent needs to interact with multiple systems or capabilities.
3. The result of one action determines the next action.
4. The agent needs to maintain state.
5. Things can fail and the agent needs to retry or recover.
6. The agent can take actions that produce a measurable outcome.
7. The agent may need human approval for important actions.
8. The agent's performance can be evaluated with objective metrics.

### Weak example

> "Analyze my company's architecture."

ChatGPT can often do this directly.

### Stronger example

> "Investigate why our production system is failing, inspect evidence across several systems, identify the likely root cause, create a remediation plan, implement the fix in a branch, run tests, and create a PR for human approval."

The second problem naturally requires agentic behaviour.

---

# 3. Candidate AI agent projects

Several project ideas were considered.

## 3.1 AI Cloud Architecture Agent

The agent takes a business requirement and designs an Azure architecture.

Example:

> "Build an architecture for a multi-tenant e-commerce platform processing 50M events/day with 99.99% availability."

Possible capabilities:

- Ask clarifying questions
- Search Azure documentation
- Select Azure services
- Compare alternatives
- Estimate costs
- Produce architecture diagrams
- Identify bottlenecks and failure scenarios
- Generate Terraform/Bicep
- Review its own architecture

Potential tools:

```text
Azure Documentation Search
Azure Pricing
Architecture Knowledge Base
Cost Calculator
Bicep Generator
Architecture Diagram Generator
```

This demonstrates:

- RAG
- Tool calling
- Planning
- Structured output
- Code generation
- Evaluation
- Architecture reasoning

### Assessment

Portfolio value: Very high  
Difficulty: High  
Fit for an experienced Azure developer: Excellent

---

## 3.2 AI Software Engineering Agent

Build an AI engineer that works on an actual Git repository.

Example:

> "Add support for OAuth2 authentication."

The agent could:

```text
Understand requirement
       ↓
Explore repository
       ↓
Identify relevant files
       ↓
Create implementation plan
       ↓
Modify code
       ↓
Run tests
       ↓
Analyze failures
       ↓
Fix implementation
       ↓
Generate PR
```

Potential tools:

```text
search_code()
read_file()
write_file()
run_tests()
run_build()
git_diff()
create_branch()
create_pull_request()
```

The important part is allowing the agent to iterate based on test results.

### Assessment

Portfolio value: Very high  
Difficulty: Very high  
Fit for an experienced software developer: Excellent

---

## 3.3 AI Business Intelligence Agent

The agent answers questions about company data.

Example:

> "Why did our revenue decline in Q2?"

Possible process:

```text
Understand question
       ↓
Query SQL
       ↓
Analyze revenue
       ↓
Segment by geography
       ↓
Segment by product
       ↓
Compare previous quarters
       ↓
Identify anomaly
       ↓
Generate explanation
       ↓
Create chart
```

Potential capabilities:

- SQL database
- Data dictionary
- RAG over business definitions
- Python data analysis
- Chart generation
- Report generation

This demonstrates:

- Agents
- SQL
- RAG
- Analytics
- Python
- Visualization
- Structured reasoning

---

## 3.4 AI Security Investigation Agent

Build an AI SOC analyst.

Inputs could include simulated:

- Login failures
- IP addresses
- Cloud logs
- Firewall events
- Application logs
- User activity

Example:

> "Investigate suspicious activity involving this user."

The agent correlates evidence across multiple sources and produces an incident report.

Potential flow:

```text
Authentication logs
        │
Azure logs
        │
Firewall
        │
Threat intelligence
        │
Application logs
        ↓
Correlation
        ↓
Investigation
        ↓
Incident report
```

---

## 3.5 AI Recruitment / Candidate Evaluation Agent

The agent evaluates a candidate against a job description using:

- Resume
- Interview transcript
- Coding assessment
- Job requirements

Possible output:

```text
Candidate: X

Technical competency       8.5/10
System design              9.0/10
Cloud                      7.5/10
Communication              8.0/10

Strong evidence:
• Designed distributed event-driven systems

Concern:
• Limited production Kubernetes experience

Recommendation:
STRONG HIRE
```

A useful extension is requiring every score to be supported by evidence.

---

## 3.6 AI Contract / Policy Analysis Agent

The agent compares complex documents.

Example:

> "Compare these two vendor contracts and identify material differences."

It could:

```text
Parse documents
      ↓
Retrieve relevant clauses
      ↓
Compare clauses
      ↓
Identify contradictions
      ↓
Assess risks
      ↓
Generate summary
```

It can support:

- Contracts
- SLAs
- Security policies
- Procurement documents
- Compliance policies

The interesting aspect is cross-document reasoning rather than simple PDF summarization.

---

## 3.7 AI Travel Planning Agent

The agent can combine:

- Flights
- Hotels
- Weather
- Maps
- Attractions
- Currency
- Public transport
- Restaurants

Example:

> "Plan a 7-day trip to Japan for two adults with a ₹3 lakh budget."

The agent considers:

- Budget
- Travel time
- Opening hours
- Geographic proximity
- Weather
- Preferences

It can re-plan when circumstances change.

---

# 4. The strongest project direction

The discussion then moved toward an important observation:

> Most of the ideas above can be answered reasonably well by ChatGPT or Gemini.

That is correct.

Therefore, rather than building an "AI that knows things", build an **AI that does things**.

For an experienced software developer, the strongest candidate is:

# Autonomous Software Maintenance / SRE Agent

This is not merely a coding chatbot.

It is an AI engineer that can investigate and resolve software problems.

---

## Example

Give it a GitHub repository and an issue:

> "The `/api/parties` endpoint has become slow when filtering by classification."

The agent could have access to:

```text
GitHub
  ├── Source code
  ├── Issues
  ├── Pull requests
  └── Git history

Database
  └── Query execution

Application
  └── Logs / metrics

Development environment
  ├── Build
  ├── Unit tests
  └── Integration tests
```

Possible tools:

```text
search_code()
read_file()
search_git_history()
run_tests()
run_benchmark()
query_database()
inspect_logs()
edit_file()
create_branch()
commit_changes()
create_pull_request()
```

Possible execution:

```text
Issue
 ↓
Understand problem
 ↓
Search repository
 ↓
Find endpoint
 ↓
Trace execution path
 ↓
Find database query
 ↓
Inspect query/index
 ↓
Check git history
 ↓
Form hypothesis
 ↓
Run benchmark
 ↓
Confirm hypothesis
 ↓
Modify code
 ↓
Run tests
 ↓
Tests fail?
 ├── YES → investigate → modify → test
 └── NO
       ↓
Create commit
       ↓
Create Pull Request
       ↓
Human approval
```

This is much more genuinely agentic than:

> "Ask an LLM to explain this code."

---

# 5. A more ambitious version: production simulator

To make the project more demonstrable, create a small simulated production environment.

For example:

```text
Microservices
├── Order Service
├── Customer Service
├── Inventory Service
├── Notification Service
└── Payment Service

       ↓

Azure Service Bus simulator
       ↓
SQL / PostgreSQL
       ↓
Redis
```

Introduce deliberate faults:

- Slow SQL query
- Missing database index
- Message-processing failure
- Redis connection problem
- Incorrect configuration
- Memory leak
- API timeout
- Dead-lettered messages
- Race condition

Then give the agent issues such as:

> "Orders are taking 30 seconds to complete."

The agent has to investigate rather than simply produce a plausible answer.

---

# 6. Multi-agent extension

Once the single-agent version works, the project can evolve into multiple specialized agents:

```text
                    ┌──────────────────┐
                    │  Lead Engineer   │
                    │     Agent        │
                    └────────┬─────────┘
                             │
             ┌───────────────┼────────────────┐
             ↓               ↓                ↓
       Code Agent       Database Agent    Cloud Agent
             │               │                │
             └───────────────┼────────────────┘
                             ↓
                      Investigation
                             ↓
                         Fix Agent
                             ↓
                        Test Agent
                             ↓
                          PR Agent
```

This introduces multi-agent orchestration.

However, this should be an advanced stage, not the starting point.

---

# 7. The portfolio management project

A simpler and more personally relevant starting point is the financial portfolio management project.

The idea is to build a **Personal Finance / Portfolio Intelligence Agent** on top of a portfolio application.

The original application concept was a serverless personal financial portfolio/bookkeeping application that could consolidate things such as:

- Mutual funds
- Fixed deposits
- Bank balances
- Crypto
- PPF
- Other assets
- Credit-card dues
- Insurance payments
- Society maintenance
- Other recurring liabilities

A key architectural constraint was that sensitive financial data should not simply become a server-side repository. The application can be designed so that financial data remains client-controlled, with the AI receiving only the minimum information needed for a particular operation.

---

# 8. The Portfolio Query Agent

Instead of immediately building an "AI financial advisor", start with a constrained **Portfolio Query Agent**.

Give the agent a small set of capabilities:

```text
get_assets()
get_liabilities()
get_transactions()
get_upcoming_payments()
get_maturing_investments()
calculate_returns()
calculate_net_worth()
```

Then let the user ask natural-language questions:

> "How much money do I have in fixed-income investments?"

> "What investments mature this year?"

> "What are my biggest upcoming payments?"

> "What percentage of my portfolio is in equity?"

> "Which assets have performed the best?"

> "What is my net worth?"

The agent determines which capabilities it needs to answer the question.

---

# 9. Example of agentic portfolio reasoning

Suppose the user asks:

> "Will I have enough liquidity for my upcoming obligations?"

The agent may reason:

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

There may not be a single function called:

```text
answer_liquidity_question()
```

The agent combines several primitive capabilities.

That is the important part.

---

# 10. A useful architecture for the portfolio agent

A first version could look like:

```text
                    React
                      │
                      ▼
              Portfolio Web App
                      │
                      ▼
                Python API
                      │
                      ▼
              LangChain Agent
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
          Tools      RAG     Memory
             │        │        │
             ▼        ▼        ▼
       Portfolio   Documents  State
          Data
```

The implementation can start with a small mock portfolio dataset.

Once the agent works, the mock tools can be replaced with the actual application's data layer.

---

# 11. A useful second phase: financial documents

Once the basic agent works, introduce document processing.

Users could upload:

- Mutual fund statements
- FD certificates
- Insurance policies
- PPF statements

The agent could extract information and respond:

> "I found a ₹5L FD maturing on 14 November 2026. Would you like me to add it to your portfolio?"

This introduces:

```text
Document
   ↓
Extraction
   ↓
Validation
   ↓
Human approval
   ↓
Structured portfolio data
```

This is much more interesting than simply asking an LLM to summarize a PDF.

---

# 12. The recommended technology stack

A good learning stack for the project is:

- **Python**
- **LangChain**
- **LangGraph**
- **OpenAI model**
- **Chroma or FAISS** initially for vector retrieval
- **React** for the UI
- **Python API service**
- **Firebase / client-controlled storage** for the portfolio application
- Azure services can be introduced later where useful

The important thing is not to start with every component.

Start small.

---

# 13. Recommended learning progression

A practical progression is:

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

This prevents the framework from hiding what is actually happening.

---

# 14. Why the portfolio agent is a good first project

It is:

- Small enough to build
- Personally relevant
- Easy to demonstrate
- Easy to test with deterministic data
- Expandable into RAG
- Expandable into document extraction
- Expandable into stateful workflows
- Suitable for human approval
- Suitable for evaluation
- Suitable for a production-style architecture

Most importantly, it lets you learn the agent stack without immediately taking on the complexity of an autonomous software engineer.

---

# 15. What should and shouldn't be delegated to the LLM?

A useful principle is:

> **Give the LLM decision-making responsibilities, but keep deterministic operations deterministic.**

For example:

### Good use of LLM

```text
"Which information do I need to answer this question?"
```

### Good use of normal code

```text
net_worth = assets - liabilities
```

Don't ask an LLM to perform financial arithmetic that ordinary application code can perform reliably.

Instead:

```text
LLM
 ↓
Decides it needs net worth
 ↓
calculate_net_worth()
 ↓
Deterministic calculation
 ↓
LLM explains result
```

This architecture is both safer and easier to evaluate.

---

# 16. The deeper lesson about agents

A useful mental model is:

> **Don't build an AI that knows things. Build an AI that can accomplish things.**

An agent becomes valuable when it can:

```text
Reason
  +
Choose actions
  +
Interact with systems
  +
Inspect results
  +
Adapt
  +
Maintain state
  +
Recover from failure
  +
Request human approval when appropriate
```

That's where frameworks such as LangChain and LangGraph become useful.

---

# 17. Final project direction

For a career-oriented learning project, a strong progression would be:

### Phase 1 — Portfolio Query Agent

```text
LLM
 +
5–7 portfolio tools
```

Learn:

- Tool calling
- Agent loops
- Structured inputs/outputs
- Prompting
- Basic LangChain

### Phase 2 — Portfolio Intelligence

Add:

- Calculations
- RAG
- Financial documents
- More sophisticated queries

### Phase 3 — Stateful agent

Add:

- Conversation state
- LangGraph
- Multi-step workflows
- Validation
- Human approval

### Phase 4 — Production engineering

Add:

- Evaluation
- Observability
- Cost tracking
- Latency tracking
- Guardrails
- Error handling
- Security
- Privacy controls

This ultimately gives you a project that demonstrates much more than:

> "I know how to call an LLM API."

It demonstrates:

> **"I can design and build reliable AI systems around LLMs."**
