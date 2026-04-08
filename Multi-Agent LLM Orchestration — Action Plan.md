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

## Owner Context

- **Hardware:** Desktop with Nvidia 3090 (24GB VRAM) for local LLM inference
- **Planning style:** Mobile-first — most planning happens via phone in limited time windows
- **Experience level:** Novice to intermediate — learning concepts as we go
- **Development time:** Limited — planning is ongoing, implementation happens in focused sessions
- **Goal:** Generic enough system to handle coding, research, writing, and analysis tasks

-----

## Key Decisions & Why (Architecture Decision Records)

### Claude handles Tier 1 (user interaction) always

Claude is the most capable model for planning, reasoning, and natural language understanding. Since it’s already on a fixed Pro plan ($20/mo), using it for the strategic layer costs nothing extra. Worker agents handle the cheaper repetitive tasks.

### Local DB as shared backbone (not direct agent-to-agent communication)

Agents communicating directly creates tight coupling — if one agent changes, others break. A shared DB means agents are fully independent and replaceable. This is the Actor Model pattern — agents only communicate via messages (DB rows).

### LiteLLM as the model proxy

Writing code that talks directly to Claude, Gemini, and Copilot APIs separately means maintaining three different integrations. LiteLLM provides one unified API — swap models by changing one parameter. Also provides cost tracking across all providers out of the box.

### CrewAI for orchestration

CrewAI already implements task queues, agent roles, and shared memory — approximately 80% of what we need. Building from scratch would take significantly longer. The remaining 20% (budget awareness, semantic caching, custom routing rules) gets built on top.

### Postgres + pgvector (not a separate vector DB)

Keeping everything in one database reduces operational complexity. pgvector adds vector search to Postgres without needing a separate service like Chroma or Qdrant. Simpler to manage, backup, and reason about.

### GitHub as single source of truth for planning docs

Planning happens across multiple sessions with no persistent memory between them. GitHub gives permanent, versioned storage that can be fetched at the start of any new session via raw URLs. Mobile-friendly via GitHub app for committing updates.

### Instruction field kept to 1-2 sentences max

Worker agents should never receive the full planning conversation as context — that would waste tokens. Claude distils the entire conversation into a succinct instruction. This is the Prompt Compression pattern — preserve meaning, minimise tokens.

### DB owns budget figures, never the LLM

LLMs hallucinate numerical data. Token counts come directly from API response headers — a guaranteed source of truth. Budget figures are calculated from summing immutable audit events (Event Sourcing pattern). The LLM only receives a pre-calculated status label (healthy/warning/critical) and acts on it.

-----

## Session Starter (for new sessions)

Paste these two URLs at the start of any new conversation:

```
https://raw.githubusercontent.com/Curiouskoalasnakebear/llm-orchestration-plan/refs/heads/main/Multi-Agent%20LLM%20Orchestration%20—%20Action%20Plan.md

https://raw.githubusercontent.com/Curiouskoalasnakebear/llm-orchestration-plan/refs/heads/main/LLM%20Systems%20—%20Concepts%20%26%20Patterns%20Learned.md
```

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