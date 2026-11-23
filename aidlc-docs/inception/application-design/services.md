# Services: ClosetCast

Services (`services/`) are the only orchestration layer. Routes (web and API) are thin and call services. Services call the agent engine, tools, domain functions, repositories and the job runner. The agent never writes (AD-3).

## Standard Planning Pattern (AD-1, AD-3)

Every AI recommendation (daily, occasion, trip, refinement) follows the same five steps:

1. **Gather context in code**: ContextBuilder geocodes, fetches weather, looks up places and dress-code guidance, and loads the profile and a ranked closet subset.
2. **Decide with the agent**: `AgentEngine.run(config, context, user_data)`. The agent may call read-only tools (at most 8 calls) and returns a structured output.
3. **Validate in code**: `validate_outfit` (owned IDs only), `check_composition` (D8), `check_weather_fit` (NFR-Q.2). If validation fails, the agent is called once more with the reasons; a second failure raises `LLM_OUTPUT_INVALID` and **nothing is saved**.
4. **Gaps in code**: `detect_gaps`. If there are gaps, `GapService` asks the LLM for the buying details.
5. **Save and return**: a repository saves the result; trips also run `PackingService`.

### Text alternative for the pattern
Context (code) → Decision (agent, read-only tools) → Validation (code; one retry; else fail closed) → Gaps (code + LLM text) → Save (repository).

---

## Service Definitions

| Service | Responsibility | Uses | Stories |
| --- | --- | --- | --- |
| ClosetService | Upload batches, item CRUD, tag edits, batch status, retry tagging | upload validation, ClosetRepository, JobRunner | US-1.1–1.6 |
| TaggingService | Job handler: tag one photo | `tag_photo`, ClosetRepository | US-1.2, US-1.3, US-9.3 |
| ProfileService | Profile, home city geocoding, ratings, worn log, privacy notice | `geocode`, ProfileRepository | US-2.1, US-2.2, US-3.6, US-9.1 |
| DailyOutfitService | Today or tomorrow, saved by date, regenerate, city override | ContextBuilder, AgentEngine(DAILY), domain validation, GapService, PlanRepository | US-3.1–3.5, US-2.3 |
| OccasionService | Parse event, plan outfit, list and get | `parse_event`, ContextBuilder, AgentEngine(OCCASION), GapService, PlanRepository | US-4.1–4.4 |
| TripService | Parse text or PDF, confirm days, plan as a background job, list and get | `parse_itinerary`, PDF extraction, ContextBuilder, AgentEngine(TRIP), PackingService, GapService, JobRunner, PlanRepository | US-5.1–5.7 |
| PackingService | Deterministic packing; "Buy before you go"; re-run on rule changes | domain `optimize_packing`, PlanRepository | US-6.1–6.5, US-8.3 |
| GapService | Gap detection plus LLM details plus search links | domain `detect_gaps`, AgentEngine(GAP_DETAILS), `build_search_link` | US-7.1, US-7.2 |
| RefinementService | Revise trip, occasion or daily plans from feedback | AgentEngine(REFINE), validation, PackingService, PlanRepository | US-8.1–8.3 |
| ContextBuilder | Deterministic context gathering | `geocode`, `get_weather`, `lookup_place`, `get_dress_code_guidance`, `search_closet`, `get_style_profile` | supports US-3, 4, 5 |
| HealthService | `/health` | DB session, settings | NFR-R.5 |

## Orchestration by Journey

### J1 First-time setup → first outfit
1. `ProfileService.save` geocodes the home city (`CITY_NOT_FOUND` asks the user to correct it).
2. `ProfileService.acknowledge_privacy_notice`.
3. `ClosetService.upload_batch`: up to 20 files → `validate_and_store_image` each → pending items → `JobRunner.enqueue("tag_photo")` per item → a batch ID is returned. The UI polls `batch_status` every 2 seconds.
4. The JobRunner worker runs `TaggingService.tag` with the concurrency limit and backoff.
5. The user edits tags (`update_tags`, change log written).
6. `DailyOutfitService.recommend(today)` follows the standard pattern and saves by date.

### J2 Night-before planning
`recommend(target="tomorrow")` saves under tomorrow's date. Next morning, `get_saved("today")` returns the saved outfit without an LLM call. `regenerate` replaces it. `ProfileService.mark_worn` feeds variety.

### J3 Multi-city trip
1. `TripService.parse(text or PDF)` → `ParsedItinerary`, shown for review.
2. The user confirms or edits the days → `start_planning` → a job ID; the UI polls.
3. The job handler runs `ContextBuilder.for_trip` (geocode each city; weather per day, choosing forecast or typical; places per day) → `AgentEngine(TRIP)` → validation per day → `GapService.find` → `PackingService.pack` → `PlanRepository.save_trip`.
4. Refinement: `RefinementService.refine_trip` → validate → re-pack → save (replaces the previous plan).

### J4 Event
`OccasionService.parse` → if fields are missing, the user is asked (`EVENT_INCOMPLETE`) → `plan` follows the standard pattern → `refine_occasion` if needed.

### J5 AI outage
The LLM adapter raises `LLMUnavailableError` / `LLMNotConfiguredError` → services map it to 502 with a stable code. Non-AI services keep working. Tagging jobs stay `pending` with a retry delay. Saved plans load from repositories only.

## Error and Degradation Behavior

| Failure | Behavior | Code |
| --- | --- | --- |
| No API key | App starts; AI services fail fast | `LLM_NOT_CONFIGURED` (502) |
| OpenAI 429/5xx after retries | Request fails; nothing saved; tagging job re-queued | `LLM_UNAVAILABLE` (502) |
| Invalid LLM output twice | Fail closed; the previous plan is kept | `LLM_OUTPUT_INVALID` (502) |
| Forecast fails | Fall back to typical weather, labeled | — |
| Forecast and historical both fail | Daily: error; trip: that day marked "weather unavailable" | `WEATHER_UNAVAILABLE` |
| Nominatim fails or not found | Generic activity guidance, labeled | — |
| Tool-call cap reached | Partial output with a note | — (`partial = true`) |
| Unexpected exception | Global handler: log with request ID, generic 500 | `INTERNAL_ERROR` |
