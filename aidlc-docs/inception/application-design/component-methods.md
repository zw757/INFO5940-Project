# Component Methods: ClosetCast

Signatures are Python-style and indicative. Types ending in `Request`/`Result`/`Plan` are Pydantic models in `domain/models.py` (fields are defined in Functional Design). All methods are `async` unless marked *(sync, pure)*. **Detailed business rules are deferred to Functional Design**; this document fixes names, inputs, outputs and errors so units can be built in parallel.

---

## Shared Interfaces (Protocols)

```python
class LLMClient(Protocol):
    async def structured(
        self, *, model_role: Literal["fast", "agent"], system: str,
        user_parts: list[TextPart | ImagePart], schema: type[T],
    ) -> T: ...
    async def tool_turn(
        self, *, system: str, messages: list[AgentMessage],
        tools: list[ToolDeclaration], output_schema: type[T],
    ) -> ToolCallRequest | FinalOutput[T]: ...
    # Raises LLMNotConfiguredError, LLMUnavailableError, LLMOutputInvalidError

class Clock(Protocol):
    def today(self) -> date: ...   # injectable for deterministic tests
```

## C6 Agent Engine

| Method | Input | Output | Notes |
| --- | --- | --- | --- |
| `AgentEngine.run(config, context, user_data)` | `UseCaseConfig`, pre-gathered `PlanningContext`, delimited user text | `AgentResult[T]` (`output: T`, `partial: bool`, `note: str \| None`, `tool_calls: int`) | Enforces the 8-call cap; user data never goes into the system prompt |
| `UseCaseConfig` (dataclass) | `name`, `system_prompt`, `allowed_tools`, `output_schema`, `model_role` | — | Configs: `DAILY`, `OCCASION`, `TRIP`, `REFINE`, `GAP_DETAILS` |

## C7 Tool Registry

| Method | Input | Output |
| --- | --- | --- |
| `register(name, args_model, fn, description)` *(sync)* | tool definition | — |
| `declarations(names)` *(sync)* | allowed tool names | `list[ToolDeclaration]` |
| `execute(name, raw_args)` | name + raw JSON from the LLM | `ToolResult` (ok data or structured error); rejects unknown tools and invalid arguments |

**Agent-visible tools (all read-only)**: `search_closet`, `get_style_profile`, `get_dress_code_guidance`, `get_weather`, `lookup_place`. `geocode`, `optimize_packing` and `build_search_link` are called by services, not exposed to the agent (AD-1, AD-7). Functional Design may adjust the agent-visible subset per use case.

## C9 Tools

| Function | Input | Output | Errors |
| --- | --- | --- | --- |
| `geocode(city)` | `str` (≤ 100) | `GeoLocation` (name, country, lat, lon) | `CITY_NOT_FOUND`, `GEOCODING_UNAVAILABLE` |
| `get_weather(location, day, today)` | `GeoLocation`, `date`, `date` | `DailyWeather` (`source`: forecast \| historical_average) | `WEATHER_UNAVAILABLE` |
| `get_forecast(...)` / `get_typical_weather(...)` | as above | `DailyWeather` | used by `get_weather` |
| `lookup_place(name, city)` | `str`, `str` | `PlaceInfo` (place_type, confidence) or `None` | never raises on not-found; `PLACES_UNAVAILABLE` on failure |
| `get_dress_code_guidance(event_or_venue_type)` | `str` | `DressCodeGuidance` (`source`: curated \| llm_low_confidence) | — |
| `search_closet(filters)` | `ClosetFilters` | `list[ClosetItemSummary]` (ranked by preferences and variety) | — |
| `get_style_profile()` | — | `StyleProfileView` (profile + ratings summary) | `PROFILE_MISSING` |
| `optimize_packing(plan, luggage, rewear_rules)` | `TripPlan`, `LuggageSize`, `RewearRules` | `PackingResult` | — |
| `build_search_link(item)` *(sync)* | `GapSuggestion` | `str` (URL) | — |
| `parse_itinerary(text, start_date)` | `str` (≤ 20,000), `date \| None` | `ParsedItinerary` | `ITINERARY_UNPARSEABLE` |
| `parse_event(text)` | `str` (≤ 2,000) | `ParsedEvent` (missing fields listed) | — |
| `tag_photo(image_bytes, mime)` | bytes, str | `ClothingTags` | `TAGGING_FAILED`, LLM errors |

## C10 Domain *(all sync, pure)*

| Function | Input | Output |
| --- | --- | --- |
| `validate_outfit(outfit, closet_ids)` | `Outfit`, `Iterable[int]` | `Outfit` or raises `UNKNOWN_CLOSET_ITEM` / `DUPLICATE_CLOSET_ITEM` |
| `check_composition(outfit, items, needs)` | `Outfit`, `list[ClosetItem]`, `OutfitNeeds` | `list[CompositionIssue]` (D8) |
| `check_weather_fit(outfit, items, weather)` | … | `list[WeatherViolation]` (NFR-Q.2) |
| `recently_worn(worn_log, day, window_days=3)` | … | `set[int]` (A3) |
| `rank_items(items, preferences, recently_worn)` | … | `list[ClosetItem]` (D9, D10) |
| `optimize_packing(days, items, budget, rewear_rules)` | `list[DayOutfit]`, `list[ClosetItem]`, `LuggageBudget`, `RewearRules` | `PackingResult` (checklist, matrix, overflow explanation); deterministic |
| `detect_gaps(needs_by_day, items)` | … | `list[Gap]` (type, required attributes, covered days, priority) |
| `c_to_f(c)` / `f_to_c(f)` | `float` | `float` |

## C11 Repositories

| Repository | Key methods |
| --- | --- |
| `ClosetRepository` | `add(item)`, `get(id)`, `list(filters)`, `update(id, changes)` (writes change log), `delete(id)` (soft-marks in plans), `ids()` |
| `ProfileRepository` | `get_profile()`, `save_profile(p)`, `add_rating(r)`, `ratings_summary(limit)`, `preferences()`, `log_worn(date, item_ids)`, `worn_log(since)`, `notice_acknowledged()`, `acknowledge_notice()` |
| `PlanRepository` | `get_daily(date)`, `save_daily(date, rec)`, `save_occasion(p)`, `get_occasion(id)`, `list_occasions()`, `save_trip(p)`, `get_trip(id)`, `list_trips()`, `save_packing(trip_id, r)`, `save_gaps(plan_ref, gaps)` |
| `JobRepository` | `create(type, payload)`, `claim_next()`, `update_progress(id, …)`, `complete(id, result)`, `fail(id, code)`, `pending()` |
| `CacheRepository` | `get(kind, key)`, `put(kind, key, value, ttl)` |

## C12 Job Runner

| Method | Input | Output |
| --- | --- | --- |
| `enqueue(job_type, payload)` | `"tag_photo" \| "plan_trip"`, Pydantic payload | `job_id` |
| `get(job_id)` | `int` | `JobStatus` (status, progress, result ref, error code) |
| `start()` / `stop()` | — | — (lifespan hooks; `start` resumes pending jobs) |

## C5 Services

| Service | Methods (input → output) |
| --- | --- |
| `ClosetService` | `upload_batch(files) → BatchUploadResult` (≤ 20, validates, stores, enqueues tagging) · `list_items(filters) → list[ClosetItem]` · `update_tags(id, TagUpdate) → ClosetItem` · `add_manual(ManualItem) → ClosetItem` · `delete_item(id) → None` · `batch_status(batch_id) → BatchStatus` · `retry_tagging(item_id) → JobStatus` |
| `TaggingService` (job handler) | `tag(item_id) → ClosetItem` (calls `tag_photo`; on `not_clothing`, removes the pending item and records the reason) |
| `ProfileService` | `get() → StyleProfile` · `save(ProfileInput) → StyleProfile` (geocodes home city) · `rate(RatingInput) → Rating` · `mark_worn(date, item_ids) → None` · `acknowledge_privacy_notice() → None` |
| `DailyOutfitService` | `get_saved(target: Literal["today","tomorrow"]) → DailyRecommendation \| None` · `recommend(DailyRequest) → DailyRecommendation` (city override, plans; saves by date) · `regenerate(target) → DailyRecommendation` |
| `OccasionService` | `parse(text) → ParsedEvent` · `plan(ConfirmedEvent) → OccasionPlan` · `get(id)` · `list()` |
| `TripService` | `parse(text \| pdf, start_date) → ParsedItinerary` · `start_planning(ConfirmedItinerary, LuggageSize, RewearRules) → job_id` · `plan(job payload) → TripPlan` (job handler) · `get(id)` · `list()` · `update_rewear_rules(id, RewearRules) → PackingResult` |
| `PackingService` | `pack(trip_plan, luggage, rewear_rules) → PackingResult` (wraps the domain optimizer; adds "Buy before you go" from gaps) |
| `GapService` | `find(needs, items, context) → list[GapSuggestion]` (domain detection + LLM details through `GAP_DETAILS` + search links) |
| `RefinementService` | `refine_trip(id, feedback) → TripPlan` · `refine_occasion(id, feedback) → OccasionPlan` · `refine_daily(date, feedback) → DailyRecommendation` (re-validate, re-pack, keep the previous plan on failure) |
| `ContextBuilder` | `for_daily(location, date, plans) → PlanningContext` · `for_occasion(event) → PlanningContext` · `for_trip(itinerary) → PlanningContext` (geocode, weather per day, places, guidance, filtered and ranked closet) |
| `HealthService` | `check() → HealthStatus` (process, DB, LLM key configured: yes or no) |

## C3 / C4 Security

| Function / class | Purpose |
| --- | --- |
| `SecurityHeadersMiddleware` | Adds the NFR-S.3 headers |
| `HostAllowlistMiddleware` | Rejects Hosts other than `127.0.0.1` / `localhost` (400) |
| `OriginCheckMiddleware` | Rejects state-changing requests with a foreign or missing `Origin` (403), or uses a CSRF token |
| `BodySizeLimitMiddleware` | Rejects bodies over the configured maximum (413) |
| `RequestIdMiddleware` | Assigns a request ID and binds it to the log context |
| `validate_and_store_image(upload) → StoredImage` | Pillow check, EXIF strip, downscale, UUID name |
| `extract_pdf_text(upload) → str` | Size and page limits, pypdf |

## Error Codes (stable, initial set)
`INVALID_IMAGE`, `IMAGE_TOO_LARGE`, `BATCH_TOO_LARGE`, `PDF_TOO_LARGE`, `PDF_NO_TEXT`, `CITY_NOT_FOUND`, `PROFILE_MISSING`, `CLOSET_EMPTY`, `ITINERARY_UNPARSEABLE`, `EVENT_INCOMPLETE`, `UNKNOWN_CLOSET_ITEM`, `DUPLICATE_CLOSET_ITEM`, `LLM_NOT_CONFIGURED`, `LLM_UNAVAILABLE`, `LLM_OUTPUT_INVALID`, `WEATHER_UNAVAILABLE`, `PLACES_UNAVAILABLE`, `GEOCODING_UNAVAILABLE`, `TAGGING_FAILED`, `JOB_NOT_FOUND`, `PLAN_NOT_FOUND`. Functional Design may add codes; existing codes never change meaning.
