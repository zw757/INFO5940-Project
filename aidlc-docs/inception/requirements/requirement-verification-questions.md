# Requirements Verification Questions: ClosetCast

These questions resolve the open questions listed in `docs/vision.md` and other gaps found while reviewing `docs/vision.md` and `docs/technical-environment.md`. Decisions already made in those documents (stack, services, security baseline, testing tools) are not asked again.

**Feature numbers** refer to the "MVP Scope — Features IN" table in `docs/vision.md`: 1 Digital closet with AI tagging, 2 Style profile, 3 Daily outfit recommendation, 4 Special occasion planner, 5 Trip planner from an itinerary, 6 Packing optimizer, 7 Gap finder, 8 Explain and refine.

**How to answer:** put the letter of your choice after each `[Answer]:` tag. If no option fits, choose the last option (Other) and describe what you want. You may add a short reason after the letter (for example `[Answer]: B, because...`). Your reasons go into the requirements and the audit log. When everything is answered, tell Claude "done".

---

## Part 1: Inputs and Data

## Question 1
What itinerary input formats should the MVP trip planner (Feature 5: Trip planner from an itinerary) accept?

A) Both pasted text and PDF upload from the start

B) Pasted text first; PDF upload added only after the critical path (features 1, 3, 5, 6) works

C) Pasted text only; PDF moved to Phase 2

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
How many clothing photos may the user upload in a single batch during closet onboarding, given the Gemini free-tier limit of roughly 10 requests per minute?

A) Up to 10 photos per batch, tagged in a background queue with retry and backoff

B) Up to 25 photos per batch, tagged in a background queue that waits out rate limits (slower, but fewer upload rounds)

C) One photo at a time (simplest; no queue needed)

X) Other (please describe after [Answer]: tag below)

[Answer]: X We will change our technical environment to OPENAI and we do not have a free tier limit. Therefore, we will not impose an upload limit at this point. But if you think this would cause other issues, please let us know and we will reconsider. 

## Question 3
Should a trip itinerary support more than one city (for example, Barcelona days 1–3, then Madrid days 4–5)?

A) Yes: each day has its own city, and weather and places are looked up per day

B) No: one destination city per trip in the MVP

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
Feature 3 (Daily outfit recommendation) answers "What should I wear today?" by fetching today's weather for a location. Where should that location come from?

A) The home city from the style profile, with an optional city field to override it on each request

B) Only the home city from the style profile

C) The browser's location (with permission), falling back to the home city

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part 2: Weather and Dress Codes

## Question 5
When should trip weather switch from forecast to historical averages, and what average should be used?

A) Forecast for days within 16 days of today; beyond that, the average of the same calendar dates over the past 5 years, labeled "based on typical weather"

B) Same 16-day switch, but average the past 3 years (fewer API calls, less precise)

C) Same 16-day switch, but use a ±3-day window around each date over 5 years to smooth out unusual years

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6
Where does dress-code guidance come from (Feature 4: Special occasion planner, and per-place norms in Feature 5: Trip planner)?

A) Curated local guide (`knowledge/` JSON) first; when an event or venue type is not in the guide, the LLM may fill the gap from its own knowledge, labeled with a "low confidence" note and leaning toward the more formal option

B) Curated local guide only; unknown types get a generic "when unsure, dress one level more formal" message

C) LLM knowledge only, with no curated guide

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
Roughly how many event and venue types should the curated dress-code guide cover for the MVP?

A) About 15–20 common types (e.g., black tie, cocktail, business formal, smart casual, interview, wedding, funeral, place of worship, fine-dining restaurant, beach, hiking trail, nightclub, museum)

B) About 30–40 types, including regional and cultural norms

C) Under 10 core dress codes only

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part 3: Outfits and Recommendations

## Question 8
Which item types must every outfit include?

A) Always a top and bottom (or a dress) plus shoes; outerwear only when the weather or dress code calls for it; accessories optional

B) Always top/bottom (or dress), shoes and outerwear

C) Only top and bottom (or dress); shoes and outerwear are suggested separately

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
How should thumbs-up/down ratings (Feature 2: Style profile) influence later recommendations in the MVP?

A) The most recent ratings and comments are summarized and given to the agent as context through `get_style_profile()`

B) Ratings are stored as structured preferences (e.g., liked or disliked item pairs and items) that code uses to filter or rank closet search results

C) Both: structured filtering in code plus a summary passed to the agent

X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 10
Should daily recommendations actively add variety (avoid suggesting the same items again on consecutive days)?

A) Yes: keep a log of recommended or accepted outfits and tell the agent to avoid items used in the last few days

B) Yes, but only through a simple "I wore this" button the user presses; only those items count as recently worn

C) No: variety comes only from the LLM, with no history tracked in the MVP

X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Part 4: Packing Optimizer and Gap Finder

## Question 11
How should luggage capacity (personal item, carry-on, checked) be modeled in the packing optimizer?

A) Item-count limits per luggage size (e.g., personal item ≤ 8, carry-on ≤ 14, checked ≤ 30 clothing items, excluding underwear, socks and sleepwear)

B) An estimated "bulk" score per category and warmth level (e.g., a coat counts more than a t-shirt), with a capacity budget per luggage size

C) Item counts, with shoes and heavy outerwear counted as two items each

X) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 12
What re-wear rules should the packing optimizer use, and can the user change them?

A) Defaults from the vision (bottoms and outerwear may repeat freely; tops may repeat after 2 days), editable per trip

B) The same defaults, fixed for the MVP (not editable)

C) Different defaults (describe after [Answer]:), editable per trip

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 13
When the trip needs an item the user doesn't own (e.g., a rain jacket), how should the plan and packing list handle it?

A) Plan around the gap with the best owned item, flag the affected days, and list the missing item in the gap finder; the packing list contains only owned items

B) Insert a clearly labeled "to buy" placeholder into the affected outfits and packing list

C) Show both: the outfit uses the best owned option, and a "to buy" suggestion is shown next to it for the affected days

X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 14
Should the packing list include non-clothing essentials (toiletries, chargers, documents)?

A) No: clothing, shoes and accessories from the closet only

B) Yes: add a short fixed checklist of essentials (not optimized, not from the closet)

C) Yes: the LLM adds trip-specific essentials (e.g., an umbrella, a plug adapter) as a separate unoptimized list

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part 5: Refinement and Persistence

## Question 15
How should plain-language refinement (Feature 8: Explain and refine) work?

A) Each feedback message revises the current saved plan; only the latest plan version is kept

B) Each feedback message creates a new plan version; the user can view and go back to earlier versions

C) Refinement works as a chat thread attached to the plan, with history kept

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 16
Which results should be saved for viewing later?

A) Trip plans and occasion plans are saved and listed; daily outfits are not saved (except for ratings)

B) Everything is saved: daily outfits, occasion plans and trip plans

C) Only trip plans are saved

X) Other (please describe after [Answer]: tag below)

[Answer]: A, plus daily outfits are saved by the date they're for. The user can plan for today or tomorrow, and opening the daily page shows the saved outfit for that date if one exists, with a 'regenerate' button.

---

## Part 6: Non-Functional and Project Setup

## Question 17
What should the app do when the Gemini API is unavailable (no key, quota exhausted, outage)?

A) Closet browsing, manual item entry, tag editing, profile and viewing saved plans still work; AI features show a clear error with a stable error code

B) The app refuses to start without a valid `GEMINI_API_KEY`

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 18
The committed `.env.example` currently lists `OPENAI_API_KEY` and `OPENAI_MODEL`, but `docs/technical-environment.md` requires Gemini (`GEMINI_API_KEY`, `GEMINI_MODEL`) and prohibits the OpenAI SDK. How should this be resolved?

A) Replace with `GEMINI_API_KEY=` and `GEMINI_MODEL=gemini-2.5-flash` during Construction (Gemini stays the only provider)

B) Keep both: Gemini is the default, but OpenAI can be configured as an alternative provider (this changes the technical environment)

X) Other (please describe after [Answer]: tag below)

[Answer]: X OpenAI only (e.g. a GPT model with vision); update the technical environment

## Question 19
What level of accessibility should the web UI meet?

A) Basic: semantic HTML, alt text on item photos, labeled form fields, keyboard-usable controls (mostly provided by Pico CSS)

B) Target WCAG 2.1 AA, checked with an automated tool such as axe

C) No specific accessibility requirements for the MVP

X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Part 7: AI-DLC Extensions

## Question: Security Extensions
Should security extension rules be enforced for this project?

*Context: `docs/technical-environment.md` already defines security requirements (localhost binding, secrets in `.env`, input validation, EXIF stripping, OWASP Top 10 and OWASP LLM Top 10). The extension adds blocking checks at every stage. Some of those checks are aimed at cloud and production systems and would be marked N/A for a local app.*

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)

B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question: Resiliency Extensions
Should the resiliency baseline be applied to this project?

**What this extension is.** Enabling it applies a set of **directional, design-time best practices** for building resilient systems, derived from the **AWS Well-Architected Framework (Reliability Pillar)** and resilience-review guidance. It steers requirements, design, and code toward fault tolerance, high availability, observability, and recoverability — covering 15 practice areas across business goals, change management, observability, high availability, disaster recovery, and continuous improvement.

**What this extension is NOT.** Enabling it does **not** make your workload production-ready, nor does it certify or guarantee any availability, RTO, or RPO target. It is a **starting point** that scaffolds good resiliency decisions early — it is not a substitute for a formal **AWS Well-Architected Review** of the built system.

Treat the output as a well-grounded **first draft of your resiliency posture** to build on and validate — not a finished, production-certified result.

A) Yes — apply the resiliency baseline as directional best practices and design-time guidance (recommended for business-critical workloads, as an informed starting point that you can validate and harden before go-live)

B) No — skip the resiliency baseline (suitable for PoCs, prototypes, and experimental projects where rapid iteration matters more than reliability)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

*Context: ClosetCast has pure logic that suits property-based tests: the packing optimizer, outfit validation and unit conversion. Enabling this would add the `hypothesis` library, which is not yet in the technical environment.*

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)

B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)

C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)

X) Other (please describe after [Answer]: tag below)

[Answer]: A
