# ClosetCast

**An AI stylist that plans daily outfits, event outfits and trip packing lists from the clothes you already own.**

## Team

| Full name | NetID |
| --- | --- |
| May Wu | zw757 |
| Katherine Yang | ky453 |
| Emma Dai | cd663 |

## Tools

- **AI coding agent:** Claude Code
- **Methodology:** **AI-DLC v1.0.1** ([release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1)), installed at `CLAUDE.md` and `.aidlc-rule-details/`
- **LLM in the product:** OpenAI API with two models: **`gpt-6-luna`** (`OPENAI_MODEL_FAST`) for photo tagging and itinerary/event parsing, and **`gpt-6.2-sol`** (`OPENAI_MODEL_AGENT`) for the agent's tool calling and reasoning. Both support image input, function calling and structured output; model names are config settings, so they can change without code changes. Changed from Google Gemini on 2026-11-19 during AI-DLC Requirements Analysis.
- **Other services:** Open-Meteo (weather, geocoding), OpenStreetMap Nominatim (places)

## Problem and why an LLM fits

Getting dressed means weighing several things at once: the weather, the day's activities, the dress expectations of each place, personal style and, when traveling, luggage space. People handle this badly. Day to day, they rotate the same few outfits while most of their closet goes unworn. For trips and events, they overpack, forget the one item they needed, or misread dress codes like "cocktail attire." ClosetCast is for college students and young professionals who want quick, reliable answers to three questions: *What should I wear today? What should I wear to this event? What should I pack for this trip?* It answers using only clothes the user owns and says clearly what is missing.

An LLM fits because almost every input is unstructured and needs judgment. Clothing arrives as photos, which a vision model turns into structured attributes far faster than typing. Itineraries arrive as free text or PDFs ("Day 2: Sagrada Família, tapas, then a rooftop bar") that must be split into days, places and activities. Dress codes and cultural norms ("covered shoulders at churches") need interpretation, not lookup. Picking an outfit trades off weather, activity, formality and taste at once, and the user needs an explanation they can trust. Existing closet apps match outfits to temperature with fixed rules; they can't read an itinerary, interpret a dress code or explain a choice.

## Planned LLM integration

ClosetCast is an **LLM agent with tools**, built as a small custom function-calling loop on the **OpenAI API** (`gpt-6-luna` for tagging and parsing, `gpt-6.2-sol` for the agent) via the official `openai` Python SDK. LLM capabilities appear in four places: **vision tagging** turns each closet photo into an editable item record; **parsing** turns itineraries and event descriptions into structured days and occasions; the **agent loop** chooses which tools to call (`get_weather`, `geocode`, `lookup_place`, `get_dress_code_guidance`, `search_closet`, `get_style_profile`, `optimize_packing`, `build_search_link`) and combines the results into outfits with explanations; and **refinement** revises a plan from plain-language feedback ("no skirts this trip"). Features that depend on the LLM: closet tagging, daily outfits, the occasion planner, the trip planner, the gap finder and refinement.

We deliberately keep some work out of the LLM. The packing optimizer is deterministic Python, because LLMs are unreliable at counting. Every proposed outfit is checked in code against the closet database, so the app never recommends clothes the user doesn't own, and all LLM output is parsed into validated Pydantic schemas. We chose a hosted model over a local Ollama model because we need reliable image understanding and tool calling together, which small local models handle inconsistently on student laptops.

## Scope and feasibility

By December 16, ClosetCast will be a locally running web app (FastAPI with server-rendered pages) with eight MVP features: (1) a digital closet with AI photo tagging, (2) a style profile with outfit ratings, (3) daily outfits from live weather, (4) an occasion planner that interprets dress codes, (5) a trip planner that reads an itinerary and plans each day, (6) a packing optimizer that fits the plan into the chosen luggage, (7) a gap finder that explains what to buy with search links, and (8) plain-language refinement. This combines vision, document parsing, multi-step tool use and optimization, well beyond a single feature.

It stays achievable because it runs locally for one user and uses free data sources (only OpenAI needs a key; each team member uses their own key with a monthly budget). The riskiest ideas are out of scope: price comparison and scraping, checkout, virtual try-on, a mobile app, email or calendar import, and accounts. The critical path is features 1 → 3 → 5 → 6; features 7 and 8 can be simplified if time runs short. Details: [`docs/vision.md`](docs/vision.md) and [`docs/technical-environment.md`](docs/technical-environment.md).

AI-DLC Inception split the work into four units of work, built in order during Construction ([`unit-of-work.md`](aidlc-docs/inception/application-design/unit-of-work.md)):

| Dates | Work |
| --- | --- |
| Nov 17 – Nov 24 | AI-DLC Inception as a team (this milestone) |
| Nov 25 – Nov 30 | **U1 Foundation & Agent Core**: app skeleton, database, LLM client, agent loop, weather/geocoding tools |
| Dec 1 – Dec 5 | **U2 Closet, Profile & Gaps** and **U3 Daily, Occasion & Refine** |
| Dec 6 – Dec 11 | **U4 Trip, Packing & Trip Refinement** |
| Dec 12 – Dec 16 | Build and test, evaluation against our success metrics, demo video and presentation |

## Collaboration plan

- **Meetings:** twice a week, ~90 minutes, on Tuesdays and Thursdays during and after class, with Zoom as backup. The first meeting is dedicated to the problem, users and why an LLM fits.
- **AI-DLC sessions:** run together. One member drives Claude Code with screen shared; the team agrees on every answer and approval, which AI-DLC records in `aidlc-docs/audit.md`.
- **Between meetings:** an iMessage group chat for quick questions; GitHub Issues for tasks. Everyone reads newly generated AI-DLC documents before the next session.
- **Construction:** each member owns AI-DLC units of work: **U1** Foundation & Agent Core (whole team), **U2** Closet, Profile & Gaps (May Wu), **U3** Daily, Occasion & Refine (Katherine Yang), and **U4** Trip, Packing & Trip Refinement (Emma Dai). The unit owner drives that unit's AI-DLC session, works on a feature branch, and gets PRs reviewed by a teammate before merging to `main`. Everyone shares testing, the demo video and the presentation.

## Repository structure

```text
README.md                  # This file
CLAUDE.md                  # AI-DLC v1.0.1 core workflow (copy of core-workflow.md)
.aidlc-rule-details/       # AI-DLC v1.0.1 stage rules (unmodified release files)
.env.example               # Placeholder settings; real keys go in a gitignored .env
docs/
  vision.md                # Input to AI-DLC
  technical-environment.md # Input to AI-DLC
aidlc-docs/                # Generated by AI-DLC
  audit.md                 # Every question, answer and approval, with timestamps
  aidlc-state.md           # Stage progress (completed / skipped / in progress)
  inception/
    requirements/          # requirements.md and the answered question files
    user-stories/          # stories.md, personas.md
    plans/                 # execution-plan.md and per-stage plans
    application-design/    # components, services, dependencies, units of work
```

Application code (`src/`, `tests/`) is added at the repo root during Construction, never inside `aidlc-docs/`.
