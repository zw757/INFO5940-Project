# Requirements: ClosetCast

**Stage**: INCEPTION - Requirements Analysis
**Depth**: Comprehensive
**Date**: 2026-11-19
**Inputs**: `docs/vision.md`, `docs/technical-environment.md` (updated 2026-11-19 for OpenAI), `requirement-verification-questions.md` (Q1–Q19, 3 extension opt-ins), `requirement-clarification-questions.md` (CQ1–CQ9)

---

## 1. Intent Analysis

| Item | Assessment |
| --- | --- |
| User request | "Using AI-DLC, let's start building ClosetCast. The vision is in docs/vision.md and the technical environment is in docs/technical-environment.md." |
| Request type | New Project (greenfield) |
| Request clarity | Clear; vision and technical environment are detailed, with 9 open questions now resolved |
| Scope | System-wide: a new multi-component application (web UI, JSON API, LLM agent, tools, persistence, deterministic optimizer) |
| Complexity | Complex: vision LLM, document parsing, multi-step tool calling, combinatorial packing optimizer, four external services |
| Depth chosen | Comprehensive, with traceability from each requirement to the source decision |

---

## 2. Decision Summary (Traceability)

Source key: **V** = `docs/vision.md`, **T** = `docs/technical-environment.md`, **Qn** = verification question n, **CQn** = clarification question n.

| # | Decision | Source |
| --- | --- | --- |
| D1 | Itinerary input accepts both pasted text and PDF upload from the start | Q1 = A |
| D2 | Batch photo upload: up to **20 photos per batch**, tagged in a background queue with limited concurrency and retry/backoff on rate limits | Q2 = X, CQ1 = C |
| D3 | Trips may span multiple cities; each day has its own city | Q3 = A |
| D4 | Daily outfit uses the home city from the profile, with an optional per-request city override | Q4 = A |
| D5 | Forecast within 16 days; beyond that, the average of the same calendar dates over the past 5 years, labeled "based on typical weather" | Q5 = A |
| D6 | Dress-code guidance: curated local guide first; the LLM fills gaps from its own knowledge, labeled "low confidence" and leaning more formal | Q6 = A |
| D7 | Curated guide covers about 15–20 common event and venue types | Q7 = A |
| D8 | Every outfit includes a top and bottom (or a dress) plus shoes; outerwear only when weather or dress code needs it; accessories optional | Q8 = A |
| D9 | Ratings affect later recommendations both as structured preferences used by code and as a summary given to the agent | Q9 = C |
| D10 | Variety: an "I wore this" button logs worn items; recently worn items are avoided | Q10 = B |
| D11 | Luggage capacity uses a bulk score per category and warmth, with a capacity budget per luggage size | Q11 = B |
| D12 | Re-wear defaults: bottoms and outerwear repeat freely; tops repeat after 2 days; editable per trip | Q12 = A |
| D13 | Missing items: the outfit uses the best owned item, and a "to buy" suggestion is shown next to it for affected days | Q13 = C |
| D14 | Packing list: owned items only, plus a separate "Buy before you go" section linking to the gap finder | CQ5 = A |
| D15 | Packing list covers clothing, shoes and accessories from the closet only (no toiletries or documents) | Q14 = A |
| D16 | Refinement revises the current saved plan; only the latest version is kept | Q15 = A |
| D17 | Trip and occasion plans are saved and listed. Daily outfits are saved by **target date**; the user can plan for today or tomorrow; the daily page shows the saved outfit for that date if one exists, with a "Regenerate" button | Q16 = X |
| D18 | When the LLM is unavailable, non-AI features keep working; AI features show a clear error with a stable error code | Q17 = A |
| D19 | **LLM provider changed from Google Gemini to OpenAI only.** Input documents updated accordingly | Q18 = X, CQ4 = A |
| D20 | Two configurable models: `OPENAI_MODEL_FAST` (tagging, parsing) and `OPENAI_MODEL_AGENT` (agent reasoning) | CQ2 = B |
| D21 | Cost control relies on the monthly spending limit in the OpenAI dashboard; the app logs approximate token usage per request | CQ3 = A |
| D22 | Accessibility target: WCAG 2.1 AA, checked manually with axe DevTools plus automated HTML checks in pytest | Q19 = B, CQ6 = C |
| D23 | Encryption at rest is provided by full-disk encryption (FileVault / BitLocker / LUKS), required in setup instructions | CQ7 = A |
| D24 | Logs: structured JSON to a rotating local file (`data/logs/`) and the console, with a request ID, kept for 90 days, never containing secrets or photo bytes | CQ8 = A |
| D25 | Extensions: Security Baseline **enabled**; Property-Based Testing **enabled (full)**; Resiliency Baseline **disabled**, with local-relevant resiliency concerns kept as ordinary NFRs (section 5.4) | Opt-ins, CQ9 = B |

---

## 3. Users and Context

| Persona | Need | Primary features |
| --- | --- | --- |
| Everyday dresser | A quick, weather-appropriate outfit from their own closet, with variety | FR-1, FR-2, FR-3 |
| Traveler (2–10 day trips, often carry-on only) | A day-by-day plan and a minimal packing list | FR-1, FR-5, FR-6, FR-7, FR-8 |
| Occasion attendee | Dress-code interpretation and an outfit, or what to buy | FR-1, FR-4, FR-7, FR-8 |

There is one local user per installation, with no accounts (V).

---

## 4. Functional Requirements

Each requirement has acceptance criteria (AC) that can be tested. "The agent" means the LLM tool-calling loop. "Validated" means checked by deterministic code, not the LLM.

### FR-1: Digital Closet with AI Tagging (Feature 1)

- **FR-1.1** The user can upload clothing photos (JPEG, PNG or WebP; ≤ 10 MB each; one item per photo), up to **20 per batch** (D2).
- **FR-1.2** Each photo is checked by Pillow (not by file extension), EXIF-stripped, re-oriented, downscaled to ≤ 1024 px on the long side, and stored as `data/images/<uuid>.jpg`.
- **FR-1.3** Tagging runs in a background queue with limited concurrency and retry with exponential backoff on `429` and temporary errors. The UI polls the status every 2 seconds and shows per-photo progress (pending / tagged / failed).
- **FR-1.4** The vision model (`OPENAI_MODEL_FAST`) returns structured tags: category, subcategory, primary and secondary color, pattern, formality (1–5), warmth (1–5), season(s) and rain suitability. These are validated against a Pydantic schema.
- **FR-1.5** If a photo is not clothing (`category = not_clothing`), the item is not saved and the user is told why.
- **FR-1.6** The user can review and edit every tag, and add items manually without a photo.
- **FR-1.7** The user can delete items. An item used in a saved plan is marked "removed from closet" in that plan, not silently dropped.
- **FR-1.8** Item create, update and delete events are logged with timestamp and before/after values (SECURITY-13 data integrity).
- **AC** Tagged items appear in the closet with editable tags. A failed tag shows a retry action. A non-image file is rejected with error code `INVALID_IMAGE`. The 21st photo in a batch is rejected with `BATCH_TOO_LARGE`.

### FR-2: Style Profile and Ratings (Feature 2)

- **FR-2.1** Onboarding form: preferred styles, colors to avoid, comfort rules (e.g., "no heels"), temperature sensitivity (runs cold / normal / runs warm), unit preference (°F/°C), and **home city** (D4).
- **FR-2.2** The user can rate any recommended outfit thumbs up or down, with an optional comment (≤ 2,000 characters).
- **FR-2.3** Ratings are stored as **structured preferences** (liked or disliked items and item pairs), which code uses to filter or rank closet search results. Code never excludes an item based on ratings alone if that would leave no valid outfit (D9).
- **FR-2.4** `get_style_profile()` also returns a **summary of recent ratings and comments** for the agent's context (D9).
- **AC** A disliked item pair is ranked below other options in `search_closet` results. The profile summary includes the latest rating comments.

### FR-3: Daily Outfit Recommendation (Feature 3)

- **FR-3.1** The user chooses a **target date: today or tomorrow** (D17) and can optionally describe their plans (≤ 1,000 characters).
- **FR-3.2** The location defaults to the profile home city and can be overridden per request (D4).
- **FR-3.3** The agent fetches weather for the target date and location, then returns a **primary outfit and one alternative**, each with an explanation covering weather, activities and style.
- **FR-3.4** Outfit composition follows D8: a top and bottom (or a dress) plus shoes; outerwear only when the weather or dress code requires it; accessories optional.
- **FR-3.5** Recently worn items are avoided where possible (D10).
- **FR-3.6** The recommendation is **saved by target date**. Opening the daily page for a date that already has a saved outfit shows it, with a **"Regenerate"** button that replaces it (D17).
- **FR-3.7** An **"I wore this"** button logs the outfit's items as worn on that date (D10).
- **FR-3.8** If the closet cannot produce a valid outfit, the response explains what is missing (links to FR-7).
- **AC** Asking for tomorrow's outfit the night before, then opening the daily page the next morning, shows the same outfit. The outfit satisfies the weather rules in NFR-Q.2.

### FR-4: Special Occasion Planner (Feature 4)

- **FR-4.1** The user describes an event in free text (≤ 2,000 characters): type, date, location, stated dress code, indoor/outdoor.
- **FR-4.2** The LLM parses the description into a structured occasion (Pydantic-validated).
- **FR-4.3** Dress-code guidance comes from the **curated guide** (`knowledge/` JSON, about 15–20 types; D7). For types not in the guide, the LLM gives guidance labeled **"low confidence"** that leans toward the more formal option (D6).
- **FR-4.4** The agent checks the weather at that place and date (forecast or typical; D5), then proposes outfits from the closet or reports what is missing (FR-7, D13).
- **FR-4.5** Occasion plans are saved and listed (D17).
- **AC** "Cocktail attire, garden party, June" returns the curated cocktail guidance, an outfit or gap list, and the weather source label.

### FR-5: Trip Planner from an Itinerary (Feature 5)

- **FR-5.1** The user pastes itinerary text (≤ 20,000 characters) **or** uploads a PDF (≤ 5 MB, ≤ 20 pages, text extracted with pypdf) (D1), and selects a luggage size (personal item, carry-on, checked).
- **FR-5.2** The LLM extracts a structured plan: days, with **a city per day** (D3), places and activities. The user can review and correct the parsed days before planning.
- **FR-5.3** Weather is fetched per day and city: forecast if the date is within 16 days, otherwise the 5-year average of the same calendar dates, labeled "based on typical weather" (D5).
- **FR-5.4** Each place is resolved with `lookup_place` (Nominatim, ≤ 1 request/second, cached) to a place type, then mapped to dress norms (D6).
- **FR-5.5** The agent produces a day-by-day outfit plan with an explanation per day. Trip planning runs as a background task with a status endpoint.
- **FR-5.6** Trip plans are saved and listed (D17).
- **AC** Every itinerary day gets an outfit (100% trip coverage, V). A two-city itinerary uses each city's weather on its days.

### FR-6: Packing Optimizer (Feature 6)

- **FR-6.1** A **deterministic** Python function (no LLM) selects the smallest set of owned items that covers every day's outfit needs.
- **FR-6.2** Capacity uses a **bulk score per category and warmth** with a budget per luggage size (D11). The default scores and budgets are defined in Functional Design.
- **FR-6.3** Default re-wear rules: bottoms and outerwear may repeat freely; tops may repeat after 2 days. The user can **edit them per trip** (D12).
- **FR-6.4** Output: a packing checklist (owned items only) and an outfit matrix (item × day) (D14).
- **FR-6.5** A separate **"Buy before you go"** section lists gap items, linking to the gap finder (D14).
- **FR-6.6** If the plan cannot fit the luggage budget, the optimizer returns the best partial result and a clear explanation of what doesn't fit. It never silently drops days.
- **FR-6.7** Scope is clothing, shoes and accessories from the closet only (D15).
- **AC** For a 5-day carry-on trip, ≤ 12 clothing items produce ≥ 5 distinct outfits (V). Every item used in the plan appears in the packing list (V). The output is identical for identical input.

### FR-7: Gap Finder (Feature 7)

- **FR-7.1** When the closet cannot satisfy a requirement (weather, dress code, activity), the outfit uses the **best owned item**, and a **"to buy" suggestion** is shown next to it for the affected days or events (D13).
- **FR-7.2** A prioritized "consider buying" list gives the item type, required attributes (e.g., "waterproof, packable, neutral color to match your navy and grey items"), the reason, and the days or events it covers.
- **FR-7.3** Each suggestion includes a generic web search link built by `build_search_link`. No prices and no scraping.
- **AC** A rainy trip with no rain-suitable outerwear yields a flagged outfit, a rain jacket in the gap list, and that item in "Buy before you go".

### FR-8: Explain and Refine (Feature 8)

- **FR-8.1** Every recommendation includes a short explanation.
- **FR-8.2** The user can give plain-text feedback (≤ 2,000 characters), e.g., "no skirts this trip". The agent revises the **current saved plan** and keeps unaffected parts unchanged. Only the latest version is kept (D16).
- **FR-8.3** After refinement, revised outfits are validated again (FR-9), and trip plans re-run the packing optimizer.
- **AC** "Swap the blazer on day 2 for something warmer" changes only day 2 (and the packing list if needed).

### FR-9: Cross-Cutting Agent Behavior

- **FR-9.1** The LLM references clothes **only by closet item ID**. Every outfit passes `validate_outfit()` before reaching the user. Unknown or duplicate IDs are rejected, and the agent is asked to retry once (T).
- **FR-9.2** There is a hard cap of **8 tool calls per request**. When the cap is reached, the agent returns its best partial answer with a note (T).
- **FR-9.3** Every LLM response that the program uses is parsed into a Pydantic model through OpenAI JSON-schema structured output (T, D19).
- **FR-9.4** User-supplied text (itineraries, events, feedback) is passed to the LLM as clearly delimited data, never placed inside system instructions. Tool arguments are validated by Pydantic before execution. Tools are read-only except `save_plan` (T).
- **FR-9.5** Agent tools (final set decided in Application Design): `get_weather`, `geocode`, `lookup_place`, `get_dress_code_guidance`, `search_closet`, `get_style_profile`, `optimize_packing`, `build_search_link`, `save_plan`.
- **FR-9.6** A one-time notice explains that clothing photos and itinerary text are sent to OpenAI's API (T).

---

## 5. Non-Functional Requirements

### 5.1 Performance

| ID | Requirement | Source |
| --- | --- | --- |
| NFR-P.1 | Daily outfit ≤ 15 s; 7-day trip plan ≤ 90 s on a laptop with a normal connection | V |
| NFR-P.2 | Weather, geocoding and place lookups cached in SQLite (forecast: 3 h; historical, geocoding, places: 30 days) | T |
| NFR-P.3 | Photo tagging queue throughput is limited by the user's OpenAI tier. Concurrency is configurable, with a conservative default | D2 |

### 5.2 Quality of Results (Success Metrics)

| ID | Requirement | Measured by |
| --- | --- | --- |
| NFR-Q.1 | ≥ 85% of items tagged with the correct category and primary color without edits | 60-photo labeled test set (V) |
| NFR-Q.2 | 0 weather-contradicting outfits (no coat when the high is > 27 °C / 80 °F; a rain layer when rain probability ≥ 60%) | Rule-based validator, 20 scenarios (V) |
| NFR-Q.3 | 100% closet faithfulness (only owned items; everything else labeled "to buy") | Automated tests (V) |
| NFR-Q.4 | 100% trip-day coverage; the packing list contains every item used | 10 sample itineraries (V) |
| NFR-Q.5 | 5-day carry-on: ≤ 12 items → ≥ 5 distinct outfits | Optimizer tests (V) |
| NFR-Q.6 | Average ≥ 4/5 "would wear this" rating | Team + 5 outside testers (V) |

### 5.3 Security (Security Baseline enabled)

| ID | Requirement | Rule |
| --- | --- | --- |
| NFR-S.1 | Encryption at rest is provided by OS full-disk encryption. The setup instructions require it and explain how to verify it on macOS, Windows and Linux. All outbound calls use HTTPS | SECURITY-01, D23 |
| NFR-S.2 | Structured JSON logging (timestamp, request ID, level, message) to a rotating file in `data/logs/` and the console, kept for 90 days. No API keys, photo bytes or full itinerary text in logs | SECURITY-03, -14, D24 |
| NFR-S.3 | HTML responses set `Content-Security-Policy` (at least `default-src 'self'`, with no `unsafe-inline`/`unsafe-eval`), `Strict-Transport-Security: max-age=31536000; includeSubDomains`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` and `Referrer-Policy: strict-origin-when-cross-origin`. To meet the CSP, **HTMX and Pico CSS are vendored** into `web/static/` (not loaded from a CDN), and **all JavaScript lives in static files** (no inline scripts; this tightens T, which allowed small inline scripts) | SECURITY-04, -13 |
| NFR-S.4 | All inputs are validated by Pydantic with explicit length limits (free text ≤ 2,000; plans ≤ 1,000; itinerary ≤ 20,000). There is a request body size limit (sized for 20 × 10 MB uploads). Uploads are checked by content. SQL only through SQLModel. Jinja2 autoescaping stays on | SECURITY-05 |
| NFR-S.5 | The server binds to `127.0.0.1` only. **Host-header allowlist** (`127.0.0.1`, `localhost`) to block DNS-rebinding attacks. **No CORS** (no cross-origin origins allowed). State-changing requests check the `Origin` header (or use a CSRF token) so other websites open in the browser cannot trigger actions | SECURITY-08, -11 |
| NFR-S.6 | Debug mode is off by default. The OpenAPI docs (`/docs`) are enabled only when a `DEBUG`/dev setting is true. User-facing errors are generic, with stable codes (`{"detail": {"code", "message"}}`), and never include stack traces, paths or keys | SECURITY-09 |
| NFR-S.7 | Dependencies are pinned in a committed `uv.lock`. `pip-audit` runs before each milestone. A CycloneDX SBOM is generated at each milestone. No unused dependencies | SECURITY-10 |
| NFR-S.8 | Security-critical logic is kept in dedicated modules: upload validation, LLM output validation (`validate_outfit`, schema parsing), tool-argument validation and HTTP security middleware. Misuse cases documented in design: prompt injection via itinerary text, malicious uploads, cross-site requests to localhost, runaway agent loops (tool-call cap) | SECURITY-11 |
| NFR-S.9 | No secrets in source. `OPENAI_API_KEY` is read only from `.env` via pydantic-settings. `.env` is gitignored, and `.env.example` has empty values | SECURITY-12 (credentials part) |
| NFR-S.10 | No `eval`, `exec` or `pickle` on untrusted data. All LLM output is parsed through Pydantic | SECURITY-13 |
| NFR-S.11 | A global exception handler logs the error and returns a safe 500 response. Every external call (OpenAI, Open-Meteo, Nominatim, SQLite, file I/O) has explicit error handling. The system fails closed: an invalid LLM output is rejected, never shown | SECURITY-15 |

### 5.4 Reliability (local scope; replaces the Resiliency Baseline per D25)

| ID | Requirement |
| --- | --- |
| NFR-R.1 | Every external HTTP call has an explicit timeout (weather and places: 10 s; LLM: a configurable timeout suited to the model) |
| NFR-R.2 | Retry with exponential backoff (tenacity) on `429` and temporary 5xx/network errors from OpenAI and Open-Meteo, with a bounded number of attempts |
| NFR-R.3 | Graceful degradation (D18): with no API key, quota exhausted or an outage, closet browsing, manual entry, tag editing, the profile and saved plans still work. AI features return 502 with a stable code (e.g., `LLM_UNAVAILABLE`, `LLM_NOT_CONFIGURED`). The app still starts without a key |
| NFR-R.4 | When the forecast is unavailable, fall back to historical averages with the "typical weather" label. When Nominatim is unavailable, use generic guidance labeled as such |
| NFR-R.5 | A `GET /health` endpoint reports the process status and the database connection, and whether an LLM key is configured (never the key itself) |
| NFR-R.6 | Backup: the setup docs explain how to back up and restore the `data/` folder (copy while the server is stopped) |
| NFR-R.7 | The approximate token usage of each LLM request is logged, so it can be compared against the OpenAI dashboard budget (D21) |

### 5.5 Usability and Accessibility

| ID | Requirement |
| --- | --- |
| NFR-U.1 | Target **WCAG 2.1 AA** (D22): semantic HTML, alt text on item photos (generated from tags), labeled form fields, keyboard-usable controls, visible focus, sufficient color contrast |
| NFR-U.2 | Verification (D22): automated pytest HTML checks (every `<img>` has `alt`, every input has a label, the page has a `lang` attribute and one `<h1>`), plus a manual axe DevTools run on every page before each milestone, with results recorded in the build-and-test docs |
| NFR-U.3 | Works in current Chrome, Firefox and Safari on a laptop screen. A mobile layout is not required (T) |
| NFR-U.4 | Temperatures are stored in Celsius and shown in the user's preferred unit (T) |

### 5.6 Maintainability and Testability

| ID | Requirement |
| --- | --- |
| NFR-M.1 | Layered modular monolith in `src/closetcast/` as defined in T. Tests live in `tests/unit` and `tests/integration` |
| NFR-M.2 | The LLM client sits behind a `Protocol`, so all tests run with a fake client and no API key or network (T) |
| NFR-M.3 | Quality gate before every merge: `ruff check`, `ruff format --check`, `mypy --strict src`, `pytest` (T) |
| NFR-M.4 | Coverage ≥ 80% for `domain/` and `tools/`; ≥ 60% overall (T) |
| NFR-M.5 | **Property-based testing (enabled, full enforcement):** Hypothesis is added as a dev dependency. PBT covers at least the packing optimizer (invariants: every day covered, capacity respected, re-wear rules respected, deterministic output), outfit validation, unit conversion round-trips and Pydantic serialization round-trips. Example-based tests are kept alongside (PBT-10). Properties are identified in Functional Design (PBT-01) |
| NFR-M.6 | Live smoke tests (`@pytest.mark.live`, `RUN_LIVE_TESTS=1`) call the real OpenAI and Open-Meteo APIs and are run manually before the demo |

---

## 6. Technical Constraints (summary of T, as updated)

- Python 3.12+, uv, FastAPI, Uvicorn, Pydantic v2, pydantic-settings, SQLModel/SQLite, Jinja2, HTMX (vendored), Pico CSS (vendored), httpx (async), Pillow, pypdf, tenacity, **openai SDK**; dev: pytest, pytest-asyncio, respx, pytest-cov, **hypothesis**, ruff, mypy, pip-audit.
- LLM: **OpenAI only** (D19). The `google-genai` SDK, LangChain-style frameworks, `requests`, JS build tools, vector databases and scraping libraries are prohibited.
- External services: OpenAI API, Open-Meteo (forecast, archive, geocoding), Nominatim (≤ 1 request/second, identifying User-Agent, cached).
- Runs locally only at `127.0.0.1:8000`, started with `uv run closetcast`.

---

## 7. Out of Scope (MVP)

As listed in V: price comparison and scraping, checkout and affiliate links, virtual try-on and image generation, background removal and collages, mobile app, accounts and cloud hosting, email/calendar/.ics import, laundry and cost-per-wear tracking, in-trip weather alerts, group trips, learned style models. Also out of scope per decisions: non-clothing packing essentials (D15), plan version history (D16), automatic LLM provider fallback (D19), and an app-level request cap (D21).

---

## 8. Assumptions (please confirm or correct during review)

| ID | Assumption | Why |
| --- | --- | --- |
| A1 | Each team member uses **their own OpenAI API key** in their own `.env`, with their own dashboard budget (no key sharing or rotation in code) | Discussed during clarification; consistent with CQ3 = A. Already reflected in the updated README and `.env.example` |
| A2 | Daily outfit target dates are limited to **today and tomorrow** (not further ahead) | Wording of the Q16 answer; occasions and trips cover later dates |
| A3 | "Recently worn" for variety means the last **3 days** of "I wore this" entries | Not specified; easy to change in Functional Design |
| A4 | Batch upload limit of 20 applies per batch; the user may upload several batches in a row | CQ1 = C wording |

---

## 9. Extension Compliance Summary (Requirements Analysis stage)

### Security Baseline (enabled)

| Rule | Status | Notes |
| --- | --- | --- |
| SECURITY-01 Encryption at rest/in transit | Compliant | Full-disk encryption as the control (NFR-S.1); HTTPS for all outbound calls; SQLite is a local file with no network connection |
| SECURITY-02 Access logging on intermediaries | N/A | No load balancer, API gateway or CDN; local server only |
| SECURITY-03 Application logging | Compliant | NFR-S.2 |
| SECURITY-04 HTTP security headers | Compliant | NFR-S.3 (HSTS header is set for compliance; browsers ignore it over plain HTTP on localhost) |
| SECURITY-05 Input validation | Compliant | NFR-S.4 |
| SECURITY-06 Least-privilege IAM | N/A | No cloud IAM; the only credential is a personal OpenAI key |
| SECURITY-07 Restrictive network config | N/A (equivalent control) | No cloud networking; the server binds to 127.0.0.1 only (NFR-S.5) |
| SECURITY-08 Application access control | Compliant / partly N/A | Single local user with no authentication (documented exception: localhost only). CORS disabled, Host allowlist, Origin/CSRF check (NFR-S.5). Object-level authorization N/A with a single user |
| SECURITY-09 Hardening | Compliant | NFR-S.6 |
| SECURITY-10 Supply chain | Compliant | NFR-S.7 |
| SECURITY-11 Secure design | Compliant | NFR-S.8; rate limiting N/A (not public-facing), with the agent tool-call cap and tagging-queue limits as abuse controls |
| SECURITY-12 Authentication | N/A (except credentials) | No user authentication; the no-hardcoded-credentials part is compliant (NFR-S.9) |
| SECURITY-13 Integrity | Compliant | NFR-S.10, vendored assets instead of CDN (NFR-S.3), closet change log (FR-1.8) |
| SECURITY-14 Alerting and monitoring | Partly N/A | 90-day log retention is compliant (NFR-S.2). Alerting and dashboards are N/A: no authentication events and no operations team for a local single-user app |
| SECURITY-15 Exception handling | Compliant | NFR-S.11 |

### Property-Based Testing (enabled, full)

| Rule | Status | Notes |
| --- | --- | --- |
| PBT-01 to PBT-10 | N/A at this stage | PBT rules apply from Functional Design onward. NFR-M.5 records the requirement and the Hypothesis framework (to be confirmed in NFR Requirements per PBT-09) |

### Resiliency Baseline

Disabled by team decision (CQ9 = B). The local-relevant concerns are covered by NFR-R.1 to NFR-R.7.
