# Components: ClosetCast

**Design decisions applied** (from `application-design-plan.md`, all answered A):
- **AD-1:** The service gathers context first; the agent's 8 tool calls are reserved for decisions.
- **AD-2:** One shared agent engine, configured per use case.
- **AD-3:** The agent has no write tools; services validate and save (`save_plan` removed).
- **AD-4:** An in-process job runner backed by a SQLite `job` table.
- **AD-5:** HTML and API routes both call the same services.
- **AD-6:** Repositories per aggregate.
- **AD-7:** Gap detection in code; the LLM writes the buying details.
- **AD-8:** An application container built at startup, accessed through FastAPI `Depends`.

**Package layout** follows `docs/technical-environment.md`, with three additions marked **(new)**: `security/`, `jobs/` and `services/`. The technical environment placed "service classes per use case" inside `agent/`. They get their own package here because several services (closet, profile, packing) use no agent at all.

```text
src/closetcast/
  main.py              # app factory, container wiring, CLI entry point
  config.py            # Settings (pydantic-settings)
  logging_config.py    # structured JSON logging setup
  container.py         # AppContainer (AD-8)
  security/            # (new) middleware, upload validation
  api/routes/          # JSON API routers
  web/routes/          # HTML + HTMX routers; web/templates/, web/static/ (vendored HTMX, Pico)
  services/            # (new) use-case services (AD-5)
  agent/               # agent engine, use-case configs, prompts, tool registry, LLM client adapter
  tools/               # functions the agent or services call: weather, geocoding, places, dress codes, closet search, profile, packing, search links, parsing, tagging
  domain/              # Pydantic models, errors, pure rules (validation, composition, weather rules, optimizer, gaps, variety, units)
  db/                  # SQLModel tables, session factory, repositories (AD-6)
  jobs/                # (new) job runner and job handlers (AD-4)
  knowledge/           # dress_codes.json + loader
```

---

## Component Catalog

Security-critical components (NFR-S.8) are marked 🔒.

### C1 Web UI (`web/`)
- **Purpose**: Server-rendered pages and HTMX partials for every user flow.
- **Responsibilities**: Render pages (`/`, `/closet`, `/profile`, `/today`, `/occasions`, `/occasions/{id}`, `/trips`, `/trips/{id}`, `/gaps`); handle form posts; return partials for polling and inline edits; show stable error messages; render the privacy notice (US-9.1).
- **Interfaces**: HTTP (HTML). Calls services (C5) only. Templates use Jinja2 autoescaping; static assets (HTMX, Pico CSS, app JS) are vendored, with no inline scripts (NFR-S.3).

### C2 JSON API (`api/routes/`)
- **Purpose**: JSON endpoints under `/api/...` and `/health`, with an OpenAPI page in debug mode only.
- **Responsibilities**: Validate requests with Pydantic, call services, and map domain errors to `{"detail": {"code", "message"}}` (4xx user errors, 502 upstream failures).
- **Interfaces**: HTTP (JSON). Calls services (C5) only.

### C3 🔒 Security Middleware (`security/middleware.py`)
- **Purpose**: Cross-cutting HTTP protections.
- **Responsibilities**: Security headers including CSP and HSTS (NFR-S.3); Host-header allowlist (NFR-S.5); Origin check / CSRF token on state-changing requests (NFR-S.5); request body size limit (NFR-S.4); request ID assignment and logging context (NFR-S.2).
- **Interfaces**: ASGI middleware registered in `main.py`.

### C4 🔒 Upload Validation (`security/uploads.py`)
- **Purpose**: Turn untrusted uploads into safe stored files.
- **Responsibilities**: Verify image content with Pillow (JPEG/PNG/WebP), enforce size limits, strip EXIF, re-orient, downscale to ≤ 1024 px, re-encode, store under a UUID name; check PDF size and page count and extract text with pypdf.
- **Interfaces**: Called by ClosetService and TripService.

### C5 Services (`services/`)
- **Purpose**: One class per use case; the only place where orchestration happens (AD-5).
- **Responsibilities**: Gather context (AD-1), call the agent engine or pure domain functions, validate results (AD-3), save through repositories, and enqueue jobs. See `services.md`.
- **Members**: ClosetService, TaggingService, ProfileService, DailyOutfitService, OccasionService, TripService, PackingService, GapService, RefinementService, and the shared ContextBuilder.

### C6 Agent Engine (`agent/engine.py`, `agent/usecases.py`, `agent/prompts/`)
- **Purpose**: The single tool-calling loop (AD-2), ≤ 200 lines.
- **Responsibilities**: Send the system prompt, delimited user data (FR-9.4), pre-gathered context and allowed tool declarations to the LLM; execute requested tools through the registry; enforce the 8-call cap (FR-9.2); return a structured final output parsed into the use case's Pydantic schema; return a partial result with a note when the cap is hit (US-9.5).
- **Interfaces**: `AgentEngine.run(...)`; `UseCaseConfig` objects for `daily`, `occasion`, `trip`, `refine` and `gap_details`.

### C7 🔒 Tool Registry (`agent/tools_registry.py`)
- **Purpose**: Exposes **read-only** tools to the agent (AD-3).
- **Responsibilities**: Hold tool declarations (JSON schema from Pydantic argument models); validate LLM-chosen arguments before execution (FR-9.4); allow only the tools in the current use-case config; convert tool errors into structured tool results.
- **Interfaces**: `ToolRegistry.declarations(names)`, `ToolRegistry.execute(name, raw_args)`.

### C8 LLM Client Adapter (`agent/llm.py`)
- **Purpose**: Hide the OpenAI SDK behind a `Protocol` (NFR-M.2).
- **Responsibilities**: Structured-output calls (text and image); tool-calling turns; timeouts and retry/backoff on `429` and 5xx (NFR-R.1, NFR-R.2); token-usage logging (NFR-R.7); map SDK errors to `LLMUnavailableError` / `LLMNotConfiguredError` (NFR-R.3).
- **Interfaces**: `LLMClient` Protocol; `OpenAILLMClient` (real); `FakeLLMClient` (tests).

### C9 Tools (`tools/`)
- **Purpose**: Functions with one job each, called by services (context gathering) or by the agent (through the registry).
- **Members**: `weather` (forecast + historical average), `geocoding`, `places` (Nominatim, ≤ 1 request/second), `dress_codes` (curated guide lookup), `closet_search`, `style_profile`, `packing` (wraps the domain optimizer), `search_links`, `parsing` (itinerary and event text → structured, via LLM), `tagging` (photo → tags, via LLM).
- **Interfaces**: Async functions taking Pydantic arguments and returning Pydantic results; external calls through the shared `httpx.AsyncClient` with timeouts; cached through the CacheRepository.

### C10 Domain (`domain/`)
- **Purpose**: Pure business logic with no I/O; the main target of property-based testing.
- **Members**: `models` (shared Pydantic models), `errors` (`ClosetCastError` with stable codes), `outfits` (`validate_outfit`, composition rules D8), `weather_rules` (NFR-Q.2 checks), `variety` (recently worn, A3), `preferences` (rating-based ranking, D9), `packing` (deterministic optimizer, D11, D12), `gaps` (gap detection, AD-7), `units` (°C/°F).
- **Interfaces**: Pure functions and dataclasses or Pydantic models.

### C11 Persistence (`db/`)
- **Purpose**: SQLModel tables and repositories (AD-6).
- **Members**: ClosetRepository (with change log, FR-1.8), ProfileRepository (profile, ratings, worn log, notice acknowledgement), PlanRepository (daily outfits by date, occasion plans, trip plans and days, packing results, gaps), JobRepository, CacheRepository (weather, geocoding, places with time-to-live).
- **Interfaces**: Repository classes over a session factory; tables created at startup.

### C12 Job Runner (`jobs/`)
- **Purpose**: Background work with concurrency limits that survives restarts (AD-4).
- **Responsibilities**: Store jobs in the `job` table; one asyncio worker loop with a configurable concurrency limit; dispatch to handlers (`tag_photo`, `plan_trip`); update progress and status; retry with backoff; resume pending jobs at startup; when the LLM is unavailable, keep tagging jobs queued and retry them later (US-9.3).
- **Interfaces**: `JobRunner.enqueue(...)`, `JobRunner.get(...)`, `start()`, `stop()`.

### C13 Knowledge (`knowledge/`)
- **Purpose**: The curated dress-code guide (about 15–20 types; contents defined in Functional Design).
- **Interfaces**: `DressCodeGuide.lookup(type) -> DressCodeGuidance | None`, `known_types()`.

### C14 Platform (`config.py`, `logging_config.py`, `container.py`, `main.py`)
- **Purpose**: Settings, structured logging (NFR-S.2), the app container (AD-8), the global exception handler (NFR-S.11), startup and shutdown (create tables, start and stop the job runner, close the HTTP client), and the `closetcast` CLI (binds to `127.0.0.1`).
