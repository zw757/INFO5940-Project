# Application Design Plan: ClosetCast

**Stage**: INCEPTION - Application Design
**Inputs**: `requirements.md`, `stories.md`, `personas.md`, `execution-plan.md`, `docs/technical-environment.md` (layered modular monolith, agent design rules)
**Outputs** (in `aidlc-docs/inception/application-design/`): `components.md`, `component-methods.md`, `services.md`, `component-dependency.md`, `application-design.md`

This stage decides the **components, their interfaces and how they talk to each other**. Detailed business rules (optimizer algorithm, weather thresholds, schema columns) come later in Functional Design.

---

## Part A: Design Questions

**How to answer:** put a letter after each `[Answer]:` tag; add a short reason if you like. If no option fits, choose the last option (Other) and describe it. Tell Claude "done" when finished. Where Claude has a recommendation, it is marked **(Recommended)** with the reason.

## Question 1: Context gathering vs. the 8-tool-call cap
The technical environment caps the agent at **8 tool calls per request** (FR-9.2). A 5-day, 2-city trip needs about 5 weather lookups, 2 geocodes and several place lookups **before** choosing any outfits, which already exceeds 8. How should trip and occasion planning gather that context?

A) **(Recommended)** Service code gathers the deterministic context first (geocode each city, weather per day, place types, dress-code guidance, filtered closet items), then calls the agent with that context already included; the agent's tool calls are reserved for decisions (e.g., extra closet searches, checking guidance for an unexpected activity). The 8-call cap stays meaningful. *Reason: fewer LLM round trips (faster, cheaper, NFR-P.1), deterministic and testable lookups, and the cap works for any trip length.*

B) The agent gathers everything through tool calls, and the cap is raised for trips (e.g., 8 + 4 per day)

C) Split the trip into one agent request per day, each with its own 8-call cap

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2: One agent or one per use case?

A) **(Recommended)** One shared agent loop (tool-calling engine), configured per use case: daily outfit, occasion, trip and refinement each get their own system prompt, allowed tool subset and output schema. *Reason: one loop to build and test (≤ 200 lines per the technical environment), with use-case differences kept in small configuration objects.*

B) A single general agent with all tools and one system prompt for every use case

C) Separate agent classes per use case, each with its own loop

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3: Who saves plans?
The technical environment says tools are read-only "except `save_plan`".

A) **(Recommended)** The agent returns its final answer as a structured output (Pydantic schema); the **service** validates it (`validate_outfit`, schema, composition rules) and then saves it. The agent has **no write tools at all**, and `save_plan` is removed from the tool list. *Reason: stronger guardrail (fail closed, NFR-S.11 and FR-9.4); a prompt-injected agent cannot write anything.*

B) Keep `save_plan` as an agent tool; it validates before saving

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4: Background work (photo tagging queue, trip planning)

A) **(Recommended)** A small in-process **job runner**: jobs are stored in a SQLite `job` table (type, status, progress, result or error code), and an asyncio worker started with the app processes them with a concurrency limit and retry/backoff. Status endpoints read the table. Pending jobs resume after restart. *Reason: meets the rate-limited tagging queue (FR-1.3), "tagging resumes when AI is available" (US-9.3), and status polling, without adding new libraries.*

B) FastAPI `BackgroundTasks` only (simplest; jobs are lost if the server restarts, and no concurrency limit is built in)

C) An external queue library (e.g., Celery or RQ with Redis), which would need new services and changes to the technical environment

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5: How does the HTML UI use the backend?

A) **(Recommended)** HTML routes and JSON API routes are both thin layers that call the **same service classes** directly. HTMX requests hit HTML routes that return partial templates. *Reason: no double HTTP hop; one place for logic; both layers stay thin, as the technical environment requires.*

B) HTML pages are static shells, and HTMX calls the JSON API and renders the JSON on the client side (needs more JavaScript)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6: Database access

A) **(Recommended)** **Repository classes** per aggregate (closet items, profile and ratings, plans, jobs, cache, change log) wrap SQLModel sessions; services depend on repositories, never on sessions directly. *Reason: services can be unit-tested with in-memory SQLite or fakes; closet change logging (FR-1.8) lives in one place.*

B) Services use SQLModel sessions directly (fewer classes, but harder to test and to keep logging consistent)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7: How the gap finder decides what is missing

A) **(Recommended)** **Hybrid:** deterministic code detects gaps (e.g., rain probability ≥ 60% and no rain-suitable outerwear owned; formality required by the guide is above the highest owned formality; a required category is missing); the LLM then writes the "consider buying" details (attributes that match the closet, the reason). Priority comes from code (number of days or events covered). *Reason: gap detection is testable and consistent with NFR-Q.2; the LLM does the language work.*

B) The LLM decides gaps as part of the plan output; code only checks the format

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8: Wiring dependencies (LLM client, HTTP client, repositories, settings)

A) **(Recommended)** One **application container** built at startup in `main.py` (settings, shared `httpx.AsyncClient`, OpenAI client adapter, repositories, services, job runner), stored on `app.state` and exposed to routes through small FastAPI `Depends` functions; tests override those functions with fakes. *Reason: matches the technical environment's example (`get_outfit_agent` wired at startup) and keeps one shared HTTP client with timeouts.*

B) Each module creates its own clients when needed (simpler at first, harder to fake in tests)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part B: Execution Checklist

Completed after the questions are answered and any ambiguities are resolved. Each item is marked `[x]` when done.

### Step 1: Analyze
- [x] 1.1 Map the 41 stories and FR-1 to FR-9 to functional areas
- [x] 1.2 Apply Part A answers to the layered structure from the technical environment

### Step 2: Components (`components.md`)
- [x] 2.1 Define each component: name, purpose, responsibilities, interfaces (API routes, web routes, services, agent engine, tools, domain, repositories, job runner, LLM client adapter, knowledge guide, security middleware, logging)
- [x] 2.2 Mark security-critical components (NFR-S.8)

### Step 3: Methods (`component-methods.md`)
- [x] 3.1 Method signatures (inputs and outputs as Pydantic or domain types) for every service, tool, repository and the agent engine
- [x] 3.2 Interface `Protocol`s for the LLM client and the agent (for fakes in tests, NFR-M.2)
- [x] 3.3 Note where detailed business rules are deferred to Functional Design

### Step 4: Services (`services.md`)
- [x] 4.1 Service definitions per use case (closet and tagging, profile and ratings, daily outfit, occasion, trip, packing, gap finder, refinement)
- [x] 4.2 Orchestration flow for each key journey (J1–J5)
- [x] 4.3 Error and degradation behavior per service (stable error codes, NFR-R.3)

### Step 5: Dependencies (`component-dependency.md`)
- [x] 5.1 Dependency matrix (which component may call which; no upward calls)
- [x] 5.2 Communication patterns (sync calls, background jobs, polling)
- [x] 5.3 Data flow diagrams (Mermaid, validated, with text alternatives) for daily outfit and trip planning

### Step 6: Consolidate and Validate
- [x] 6.1 Write `application-design.md` consolidating all four documents
- [x] 6.2 Check that every story maps to at least one component and service
- [x] 6.3 Security Baseline compliance check for this stage
- [x] 6.4 Validate Mermaid syntax and markdown per `common/content-validation.md`
- [x] 6.5 Update `aidlc-state.md`, log in `audit.md`, present completion message
