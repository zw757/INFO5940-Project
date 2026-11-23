# Units of Work: ClosetCast

**Source**: `aidlc-docs/inception/plans/unit-of-work-plan.md` (approved 2026-11-19), with one boundary adjustment made during validation (see "Boundary Adjustment" below).

ClosetCast is a **single deployable application** (modular monolith). Units of work are **logical modules** used for planning, ownership and the AI-DLC Construction loop. They are not separate services.

---

## Summary

| Unit | Name | Owner | Stories | Effort points (S=1, M=2, L=3) |
| --- | --- | --- | --- | --- |
| U1 | Foundation & Agent Core | Whole team (May Wu, Katherine Yang, Emma Dai) | 5 | 7 + platform work |
| U2 | Closet, Profile & Gaps | May Wu | 11 | 16 |
| U3 | Daily, Occasion & Refine | Katherine Yang | 12 | 18 |
| U4 | Trip, Packing & Trip Refinement | Emma Dai | 13 | 21 |
| | **Total** | | **41** | **62** |

## Boundary Adjustment (made during validation, step 4.1)

The approved grouping had a **cyclic dependency** between U3 and U4:
- U3 contained trip refinement (US-8.1, US-8.3), which needs U4's trip plans and packing optimizer.
- U4 contained place dress norms (US-5.5), which needs U3's dress-code guide and places lookup.

Because units are built in sequence (U1 → U2 → U3 → U4), U3 could not refine trips that don't exist yet. Smallest fix:
- **US-8.1** (refine a trip plan) and **US-8.3** (re-validate and re-pack) moved **U3 → U4**.
- **US-5.5** (dress norms for places) moved **U4 → U3**. U3 already owns the dress-code guide and `lookup_place`, and U4's trip planning calls U3's place-norms capability.
- Effort changed from 16 / 19 / 20 to **16 / 18 / 21**.
- **Scope-reduction note (Q7):** the first cut, "refinement supports trip plans only", now means dropping US-8.2 in U3 while keeping US-8.1 and US-8.3 in U4.

---

## Code Organization Strategy (greenfield; plan Q6 = A)

One layered package, as in the technical environment and application design. Units own **files across layers**, not folders.

```text
src/closetcast/
  main.py, config.py, logging_config.py, container.py          # U1
  security/middleware.py, security/uploads.py                  # U1 (uploads: image + PDF helpers)
  api/routes/   health.py (U1), closet.py, profile.py, gaps.py (U2),
                outfits.py, occasions.py (U3), trips.py (U4)
  web/routes/   pages.py base (U1), closet/profile/gaps (U2), today/occasions (U3), trips (U4)
  web/templates/ base.html, errors, privacy notice (U1); feature templates per unit as above
  web/static/   vendored htmx + pico + app.js (U1); feature JS files per unit
  services/     context_builder.py, health.py (U1); closet.py, tagging.py, profile.py, gaps.py (U2);
                daily.py, occasion.py, refinement.py (U3); trip.py, packing.py (U4)
  agent/        engine.py, usecases.py (U1, configs added by each unit), llm.py, tools_registry.py (U1);
                prompts/ daily, occasion, refine (U3); gap_details (U2); trip (U4)
  tools/        weather.py, geocoding.py (U1); tagging.py, closet_search.py, style_profile.py,
                search_links.py (U2); places.py, dress_codes.py (U3); parsing.py (U3 event, U4 itinerary), packing.py (U4)
  domain/       models.py, errors.py, outfits.py (validate_outfit), weather_rules.py, units.py (U1);
                preferences.py, gaps.py (U2); composition.py, variety.py (U3); packing.py (U4)
  db/           session.py, tables base, JobRepository, CacheRepository (U1); ClosetRepository,
                ProfileRepository (U2); PlanRepository daily + occasion parts (U3), trip + packing parts (U4)
  jobs/         runner.py (U1); handlers registered by U2 (tag_photo) and U4 (plan_trip)
  knowledge/    dress_codes.json + loader (U3)
tests/
  unit/ and integration/ mirror the layers; property-based tests in tests/unit/property/
  fakes/ FakeLLMClient, fixtures (U1)
```

Rule: a unit may **add** to a shared file it doesn't own (e.g., a new `UseCaseConfig` or model field) only through the shared-contract process below.

## Working Model (plan Q2, Q4, Q5 = A)

- **U1 first, together.** All three members build U1 in shared AI-DLC sessions. U2–U4 start only after U1 is merged to `main`.
- **Sequential AI-DLC sessions:** U1 → U2 → U3 → U4. The unit owner drives the session (Functional Design → NFR Requirements → NFR Design → Code Generation) and leads review. Others review PRs, test, or prepare data for their own unit.
- **Shared contracts** (`domain/models.py`, `domain/errors.py`, `agent/llm.py` Protocol, `agent/usecases.py`, repository interfaces, `container.py`, job payload schemas): **additive-only** changes, in a small separate PR reviewed by **all three** members.
- **Feature branches and PRs** reviewed by a teammate; the quality gate (`ruff`, `mypy --strict`, `pytest`) passes before merge.

## Scope-Reduction Order (plan Q7 = A, adjusted)
1. Drop US-8.2 (refine daily or occasion plans); keep trip refinement (US-8.1, US-8.3 in U4). Users use Regenerate or re-plan instead.
2. Gap finder (U2) uses code-detected gaps with **template text** instead of LLM-written details.
3. Drop PDF itinerary upload (US-5.2); text paste stays.

The critical path is never cut: closet (U2) → daily outfit (U3) → trip (U4) → packing (U4).

---

## U1: Foundation & Agent Core
- **Owner**: whole team · **Stories**: US-9.1, US-9.2, US-9.3, US-9.4, US-9.5
- **Purpose**: Everything other units build on: platform, security, shared contracts, the agent engine, background jobs, weather and geocoding, and the cross-cutting failure behavior.
- **Components**: C1 (base layout, error pages, privacy notice), C2 (`/health`, error mapping), C3, C4, C6, C7, C8, C9 (weather, geocoding), C10 (models, errors, `validate_outfit`, weather rules, units), C11 (session, base tables, JobRepository, CacheRepository), C12, C14, ContextBuilder (weather and geocoding parts), HealthService.
- **Provides**: all shared contracts; `AppContainer`; `FakeLLMClient`; test fixtures; the confirmed OpenAI call shapes (live smoke test).
- **Construction stages**: Functional Design ✔ (shared models, error codes, weather rules, job lifecycle) · NFR Requirements ✔ (security controls, Hypothesis PBT-09, timeouts and retries, OpenAI models) · NFR Design ✔ (middleware order, logging, retry and backoff, job runner) · Code Generation ✔
- **PBT targets**: `validate_outfit` (invariants), weather rule checks, °C/°F round-trip, Pydantic model serialization round-trips, cache time-to-live logic, job state transitions (stateful PBT, PBT-06).

## U2: Closet, Profile & Gaps
- **Owner**: May Wu · **Stories**: US-1.1–1.6, US-2.1–2.3, US-7.1, US-7.2
- **Purpose**: Know what the user owns and likes, and what is missing.
- **Components**: ClosetService, TaggingService (`tag_photo` job handler), ProfileService, GapService; tools `tagging`, `closet_search`, `style_profile`, `search_links`; domain `preferences`, `gaps`; ClosetRepository (with change log), ProfileRepository; `GAP_DETAILS` use case; closet, profile and gap pages and API.
- **Extra deliverable**: the 60-photo labeled tagging test set (NFR-Q.1).
- **Integration note**: U2 builds GapService and the "to buy" display component against sample inputs; U3 and U4 call it from their planning flows.
- **Construction stages**: Functional Design ✔ (tag schema, filters, preference ranking, gap detection rules, change log) · NFR Requirements ✔ (light: upload limits, tagging concurrency) · NFR Design ✔ (upload pipeline, tagging queue) · Code Generation ✔
- **PBT targets**: `rank_items` (invariants: same items, disliked pairs never ranked above equivalent alternatives), `detect_gaps` (every unmet need yields a gap; no gap when the need is met), tag serialization round-trip, image downscale invariants (long side ≤ 1024).

## U3: Daily, Occasion & Refine
- **Owner**: Katherine Yang · **Stories**: US-3.1–3.6, US-4.1–4.4, US-5.5, US-8.2
- **Purpose**: Recommend outfits for a day or an event, interpret dress codes and place norms, and refine daily and occasion plans.
- **Components**: DailyOutfitService, OccasionService, RefinementService (daily + occasion; trip refinement is added by U4); tools `places`, `dress_codes`, event parsing; domain `composition`, `variety`; knowledge guide content (about 15–20 types); `DAILY`, `OCCASION`, `REFINE` use cases; PlanRepository (daily and occasion); ContextBuilder extended with places and guidance; today and occasion pages and API.
- **Construction stages**: Functional Design ✔ (composition rules D8, variety A3, guide contents, place-type to norm mapping, daily save-by-date rules) · NFR Requirements ✔ (≤ 15 s daily) · NFR Design ✔ (light: Nominatim rate limit and caching) · Code Generation ✔
- **PBT targets**: `check_composition` (complete outfits always pass; outfits missing a required category always fail), `recently_worn` (window invariants), daily save-by-date idempotence (saving twice for the same date keeps one record), guide lookup (every known type resolves to `curated`).

## U4: Trip, Packing & Trip Refinement
- **Owner**: Emma Dai · **Stories**: US-5.1–5.4, US-5.6, US-5.7, US-6.1–6.5, US-8.1, US-8.3
- **Purpose**: Turn an itinerary into a day-by-day plan and a packing list that fits the luggage, and let the user refine it.
- **Components**: TripService (`plan_trip` job handler), PackingService, RefinementService trip methods; tools `parsing` (itinerary), `packing`; domain `packing` (optimizer); PDF extraction; `TRIP` use case and trip refinement config; PlanRepository (trip, days, packing); ContextBuilder `for_trip`; trip pages and API.
- **Construction stages**: Functional Design ✔ (optimizer algorithm, bulk scores, luggage budgets, re-wear rules, over-capacity handling, parsed-day review rules) · NFR Requirements ✔ (≤ 90 s for 7-day plans) · NFR Design ✔ (job progress reporting, context-gathering concurrency within rate limits) · Code Generation ✔
- **PBT targets**: `optimize_packing`: every day covered, capacity budget respected or overflow reported, re-wear rules respected, every checklist item used at least once, deterministic output (same input gives the same result), and an oracle test against brute force on small closets (PBT-05). Itinerary model round-trips.
