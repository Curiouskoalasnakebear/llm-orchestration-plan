# Multi-Agent LLM Orchestration — Action Plan

> **Status:** Planning Phase  
> **Current Step:** 1 of 6  
> **Last Updated:** Session 1

-----

## Project Goal

Build a local multi-agent LLM orchestration system that:

- Maximises use of free/cheap LLM tiers (Claude Pro, Gemini CLI, Copilot, Local LLM)
- Uses a shared local database as the backbone for all agent communication
- Avoids unnecessary API costs by reusing outputs and routing intelligently
- Allows easy swapping between models via a unified proxy layer

-----

## Agreed Stack

|Component      |Tool                       |Purpose                                           |
|---------------|---------------------------|--------------------------------------------------|
|Strategic agent|Claude Code (Pro $20/mo)   |Talks to user, plans, creates tasks               |
|Model proxy    |LiteLLM                    |Unified API across all providers                  |
|Local LLM      |Ollama on 3090 (24GB VRAM) |Free worker agent (Qwen 2.5 Coder 32B recommended)|
|Cloud LLMs     |Gemini CLI, Copilot, Claude|Escalation targets                                |
|Orchestration  |CrewAI                     |Multi-agent task management                       |
|Database       |Postgres + pgvector        |Shared state, task queue, outputs, vectors        |

-----

## System Architecture

```
You
 ↓
Claude (Tier 1 — Strategic Layer)
 ↓ produces structured tasks
DB Task Queue
 ↓
Orchestrator Agent (Tier 2)
 ↓ routes based on capability + budget
LiteLLM Proxy
 ↓ dispatches to best available model
Local LLM / Gemini / Copilot / Claude
 ↓ writes output back
DB Outputs Table
 ↓
Claude presents final result to you
```

-----

## Step 1 — Model Routing Rules ✅ AGREED

### Two Tier Design

- **Tier 1 (Claude):** Always handles user interaction, planning, task decomposition
- **Tier 2 (Orchestrator):** Receives structured tasks, routes to best model

### Routing Priority (cheapest first)

```
Local LLM (free)
  ↓ if insufficient capability
Gemini CLI (free tier — 1000 req/day)
  ↓ if daily limit hit
Copilot ($10/mo)
  ↓ if complex reasoning needed
Claude Pro ($20/mo)
  ↓ if budget critical
Queue and wait
```

### Task Queue Table Schema

```sql
tasks
├── id
├── created_at
├── status          -- pending/in_progress/done/failed
├── priority        -- 1-5
├── category        -- research/coding/writing/analysis
├── tags            -- ["python", "database", "refactor"]
├── instruction     -- succinct 1-2 sentence max (token saver)
├── success_criteria
├── dependencies    -- task ids that must complete first
├── assigned_model
└── output_ref      -- pointer to outputs table
```

### Model Registry Table Schema

```sql
model_registry
├── id
├── provider
├── model_name
├── capability_score    -- 1-10
├── cost_tier           -- free/low/medium/high
├── strengths           -- ["coding", "reasoning", "summarising"]
├── weaknesses          -- ["maths", "long_context"]
├── daily_limit
├── status              -- available/degraded/exhausted
└── fallback_to         -- next model id in chain
```

-----

## Step 2 — Budget Awareness ✅ AGREED

### Golden Rule

> The DB owns the budget. The LLM never calculates or reports budget figures.

### Budget Table Schema

```sql
budgets
├── id
├── provider
├── period          -- daily/monthly
├── allocated       -- estimated plan limit
├── consumed        -- tracked in real time
├── reset_at
├── status          -- healthy/warning/critical/exhausted
└── fallback_model
```

### Budget Flow

```
API call completes
  ↓
Extract token count from response headers (not from LLM)
  ↓
Write to audit table (immutable event log)
  ↓
Recalculate budget from sum of events
  ↓
Update status (healthy/warning/critical/exhausted)
  ↓
Routing layer reads status before next task dispatch
```

### Audit Table Schema

```sql
attempts
├── task_id
├── model_tried
├── attempt_number
├── failed_reason       -- rate_limit/poor_quality/timeout
├── retry_after         -- exponential backoff
└── escalated_to
```

-----

## Step 3 — Fallback Chain ✅ AGREED

Fallback is triggered by:

1. Budget status (Circuit Breaker)
1. Capability mismatch (Strategy Pattern)
1. Model unavailability (Chain of Responsibility)
1. Poor quality output (Output Validation)

```
Output received
  ↓
Meets success criteria?
  ↓ yes → accept and store
  ↓ no  → retry or escalate up chain
```

-----

## Steps Still To Plan

- [ ] **Step 4** — Output table schema & reuse logic
- [ ] **Step 5** — Shared memory schema (cross-agent context)
- [ ] **Step 6** — Observability & dashboard

-----

## Implementation Order (after planning is complete)

1. Get Ollama + model running locally
1. Set up LiteLLM proxying all providers
1. Set up Postgres + pgvector
1. Layer CrewAI on top
1. Define routing rules in config
1. Build budget awareness layer
1. Wire fallback chain
1. Observability last

-----

> **Note:** This document will be updated at the end of each planning session.  
> Implementation begins only when all 6 planning steps are agreed.
