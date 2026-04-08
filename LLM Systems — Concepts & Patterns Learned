# LLM Systems — Concepts & Patterns Learned

> **Context:** Discovered while planning a multi-agent LLM orchestration system  
> **Purpose:** Reference guide to revisit, explore and go deeper on each concept  
> **Format:** Concept → What it is → Real world example → How we used it

-----

## Software Architecture Patterns

-----

### Task Queue Pattern

**What it is:**  
A list of jobs waiting to be picked up and processed by workers. Workers pull tasks off the queue independently rather than being directly called.

**Real world example:**  
Every major tech company uses this. Celery (Python), Bull (Node.js), Sidekiq (Ruby) are popular implementations. When you upload a video to YouTube, a task queue handles the transcoding in the background.

**How we used it:**  
Our `tasks` table in Postgres acts as a task queue. Claude writes tasks to it, worker agents pick them up based on tags and categories.

**Go deeper:**

- Search: “task queue pattern distributed systems”
- Tool to explore: Celery, Bull, RabbitMQ

-----

### Separation of Concerns

**What it is:**  
Each component should have one job and one job only. Don’t mix responsibilities.

**Real world example:**  
Robert Martin’s SOLID principles — foundational software engineering. The S stands for Single Responsibility.

**How we used it:**  
We separated tasks (what to do) from outputs (what was done) into separate tables. Budget logic lives in its own table. Each layer has one job.

**Go deeper:**

- Search: “SOLID principles software engineering”
- Book: “Clean Code” by Robert Martin

-----

### CQRS — Command Query Responsibility Segregation

**What it is:**  
Separate the act of writing data (commands) from reading data (queries). They have different needs and shouldn’t be mixed.

**Real world example:**  
Used heavily in enterprise systems. Microsoft and Martin Fowler have written extensively about it.

**How we used it:**  
Tasks table = commands (write intent). Outputs table = queries (read results). Agents only pull what they need from outputs, not everything.

**Go deeper:**

- Search: “CQRS pattern Martin Fowler”
- Article: martinfowler.com/bliki/CQRS.html

-----

### Actor Model

**What it is:**  
Each agent is an independent actor that only communicates via messages. Actors never share memory directly — they pass messages to each other.

**Real world example:**  
Erlang was built on this model and powers WhatsApp (handling billions of messages). Akka (Java/Scala) is another famous implementation.

**How we used it:**  
Our agents never talk to each other directly. They communicate only via DB rows (messages). This keeps agents fully independent and replaceable.

**Go deeper:**

- Search: “Actor model distributed systems”
- Tool to explore: Akka, Erlang

-----

### Event-Driven Architecture

**What it is:**  
Components react to events (things that happened) rather than being directly called. An event is an immutable record of something that occurred.

**Real world example:**  
Kafka (used by LinkedIn, Uber, Airbnb) is the gold standard. Every action becomes an event that other services can subscribe to.

**How we used it:**  
Token consumption is recorded as an event. Budget is calculated from the sum of events, not stored directly. Agents react to task status changes.

**Go deeper:**

- Search: “event driven architecture kafka”
- Tool to explore: Apache Kafka, AWS EventBridge

-----

### MapReduce

**What it is:**  
Break a big problem into smaller pieces (map), process each piece independently, then combine the results (reduce).

**Real world example:**  
Google invented this to index the entire web. Hadoop is built on it.

**How we used it:**  
Claude breaks your request into subtasks (map). Worker agents process each subtask independently (process). Claude assembles the final answer (reduce).

**Go deeper:**

- Search: “MapReduce Google paper”
- Tool to explore: Apache Hadoop, Apache Spark

-----

### Chain of Responsibility

**What it is:**  
A request passes through a chain of handlers. Each handler either processes it or passes it to the next one.

**Real world example:**  
One of the original Gang of Four design patterns (1994 book “Design Patterns” — foundational software engineering). Used in middleware, HTTP request pipelines, customer support escalation.

**How we used it:**  
Our fallback chain. If Local LLM can’t handle a task, it passes to Gemini, then Copilot, then Claude. Each handler either accepts or escalates.

**Go deeper:**

- Search: “Chain of Responsibility design pattern”
- Book: “Design Patterns” by Gang of Four (Gamma, Helm, Johnson, Vlissides)

-----

### Strategy Pattern

**What it is:**  
Select an algorithm (strategy) at runtime based on context, without changing the code that uses it.

**Real world example:**  
Payment processing — same checkout flow, different payment strategy (card, PayPal, crypto) selected at runtime.

**How we used it:**  
Selecting which LLM model to use based on task capability requirements and budget status. The orchestrator swaps strategies without changing the task logic.

**Go deeper:**

- Search: “Strategy pattern software design”
- Book: “Design Patterns” by Gang of Four

-----

## Reliability Patterns

-----

### Circuit Breaker Pattern

**What it is:**  
Automatically stop sending requests to a failing or overloaded service. Has three states: Closed (normal), Open (blocked), Half-open (testing recovery).

**Real world example:**  
Netflix built Hystrix specifically for this. Every major cloud architecture uses it to prevent cascade failures.

**How we used it:**  
Budget status drives our circuit breaker. When Claude’s budget hits critical, the circuit opens and requests route to cheaper models automatically.

**Go deeper:**

- Search: “Circuit breaker pattern Martin Fowler”
- Tool to explore: Netflix Hystrix, Resilience4j

-----

### Retry with Exponential Backoff

**What it is:**  
When a request fails, wait before retrying. Double the wait time each retry (1s, 2s, 4s, 8s…). Prevents hammering a struggling service.

**Real world example:**  
AWS, Google Cloud, and every major API recommends this. It’s in the official docs of virtually every cloud service.

**How we used it:**  
When a model fails (rate limit, timeout), we wait exponentially before retrying or escalating to the next model in the chain.

**Go deeper:**

- Search: “exponential backoff retry pattern”
- AWS docs: “Error retries and exponential backoff”

-----

### Admission Control

**What it is:**  
Before accepting a new request, check if you have the resources to handle it. Reject or queue if not.

**Real world example:**  
Netflix uses this to protect services during traffic spikes. Hospital triage is a real world analogy — assess before admitting.

**How we used it:**  
Before dispatching a task to a cloud LLM, check the budget table first. If budget is exhausted, the task is queued rather than sent.

**Go deeper:**

- Search: “admission control distributed systems”

-----

## Data Patterns

-----

### Event Sourcing

**What it is:**  
Instead of storing current state, store every event that led to that state. Current state is always derived by replaying events.

**Real world example:**  
Banks don’t store your balance — they store every transaction. Your balance is the sum of all transactions. Git is another example — it stores every commit, not just the current state.

**How we used it:**  
Token consumption is stored as immutable audit events. Budget figures are calculated from summing events, never stored directly. This makes the budget tamper-proof and fully auditable.

**Go deeper:**

- Search: “Event sourcing Martin Fowler”
- Article: martinfowler.com/eaaDev/EventSourcing.html

-----

### RAG — Retrieval Augmented Generation

**What it is:**  
Before asking an LLM a question, first search your own knowledge base for relevant information. Give that context to the LLM along with the question. Reduces hallucination and token costs.

**Real world example:**  
Industry standard for enterprise LLM deployments. Used by virtually every production AI system that needs to answer questions about specific data.

**How we used it:**  
Before creating a new task, check if a semantically similar task has already been completed. Retrieve and reuse that output instead of calling the LLM again. This is our semantic cache.

**Go deeper:**

- Search: “RAG retrieval augmented generation”
- Tools to explore: LlamaIndex, LangChain, pgvector

-----

### Progressive Summarisation

**What it is:**  
Distil information into increasingly concise layers. Raw notes → summary → key insight. Each layer is smaller and more useful than the last.

**Real world example:**  
Popularised by Tiago Forte (Building a Second Brain). Journalists use this — full interview → article → headline.

**How we used it:**  
Our outputs table stores both `raw_output` and `summary`. When another agent needs this output as context, it gets the summary not the full raw output — saving tokens.

**Go deeper:**

- Search: “Progressive summarisation Tiago Forte”
- Book: “Building a Second Brain” by Tiago Forte

-----

## LLM-Specific Concepts

-----

### Grounding

**What it is:**  
Anchoring an LLM’s responses to verified external data sources rather than letting it generate facts from memory. Prevents hallucination on factual data.

**Real world example:**  
Google’s Gemini is “grounded” with Google Search. Microsoft’s Copilot is grounded with Bing. Enterprise systems ground LLMs with their own databases.

**How we used it:**  
The LLM never sees or calculates budget numbers. The system queries the DB directly and passes pre-calculated status to the LLM. The LLM only interprets, never generates numerical facts.

**Go deeper:**

- Search: “LLM grounding hallucination prevention”

-----

### Prompt Compression / Context Distillation

**What it is:**  
Reducing the amount of text sent to an LLM while preserving the essential meaning. Active research area in LLM efficiency.

**Real world example:**  
Microsoft’s LLMLingua, Meta’s research on prompt compression. Every token costs money so compressing context is economically significant.

**How we used it:**  
Claude distils an entire planning conversation into a 1-2 sentence `instruction` field in the task table. Worker agents only receive that succinct instruction, not the full conversation.

**Go deeper:**

- Search: “LLM prompt compression context distillation”
- Tool to explore: LLMLingua (Microsoft)

-----

### Semantic Caching

**What it is:**  
Instead of caching exact matches, cache by meaning. If someone asks “what’s the capital of France?” and another asks “which city is France’s capital?” — return the same cached answer.

**Real world example:**  
GPTCache is an open source tool built specifically for this. Redis is exploring semantic caching features.

**How we used it:**  
Before calling any LLM, embed the task instruction as a vector and search the outputs table for semantically similar completed tasks. If found, reuse the output without an LLM call.

**Go deeper:**

- Search: “semantic caching LLM vector similarity”
- Tool to explore: GPTCache, pgvector

-----

### Output Validation / Guardrails

**What it is:**  
Automatically checking LLM outputs against defined criteria before accepting them. Reject and retry if output doesn’t meet the bar.

**Real world example:**  
Guardrails AI and LangChain’s output parsers are popular tools. Used in production systems to ensure structured, safe, accurate outputs.

**How we used it:**  
Each task has a `success_criteria` field. After an agent completes a task, output is validated against criteria. If it fails, the task retries or escalates.

**Go deeper:**

- Search: “LLM output validation guardrails”
- Tools to explore: Guardrails AI, LangChain output parsers

-----

## Tools Referenced

|Tool           |What it does                           |Why relevant                 |
|---------------|---------------------------------------|-----------------------------|
|LiteLLM        |Unified API proxy for all LLM providers|Model swapping, cost tracking|
|CrewAI         |Multi-agent orchestration framework    |Agent roles, task queues     |
|Ollama         |Run local LLMs on your hardware        |Free inference on 3090       |
|pgvector       |Vector search extension for Postgres   |Semantic caching, RAG        |
|Chroma / Qdrant|Standalone vector databases            |Alternative to pgvector      |
|LangChain      |LLM application framework              |Output parsers, RAG chains   |
|LlamaIndex     |Data framework for LLM apps            |RAG, document indexing       |
|Guardrails AI  |LLM output validation                  |Quality gates                |
|GPTCache       |Semantic caching for LLMs              |Token cost reduction         |
|LLMLingua      |Prompt compression                     |Context cost reduction       |

-----

> **Note:** Every concept here was discovered organically while solving a real problem.  
> That’s the best way to make these ideas stick.  
> Revisit any section and search the suggested terms to go deeper.
