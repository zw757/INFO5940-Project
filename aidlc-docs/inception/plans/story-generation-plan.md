# Story Generation Plan: ClosetCast

**Stage**: INCEPTION - User Stories, Part 1 (Planning)
**Inputs**: `aidlc-docs/inception/requirements/requirements.md` (approved), `docs/vision.md`, `aidlc-docs/inception/plans/user-stories-assessment.md`
**Outputs**: `aidlc-docs/inception/user-stories/personas.md`, `aidlc-docs/inception/user-stories/stories.md`

---

## Part A: Questions for the Team

**How to answer:** put a letter after each `[Answer]:` tag. If no option fits, choose the last option (Other) and describe your choice. A short reason after the letter is welcome. When you are done, tell Claude "done". These answers decide how the stories are written, so they are recorded in the audit log.

### Breakdown approaches (background for Question 1)

| Approach | How stories are grouped | Strengths | Weaknesses for ClosetCast |
| --- | --- | --- | --- |
| User journey-based | By end-to-end flows (onboard closet → get today's outfit; plan trip → pack → buy → refine) | Shows how features connect; good for demo scripts and acceptance testing | Shared steps (e.g., weather lookup) appear in several journeys |
| Feature-based | By the 8 MVP features (FR-1 to FR-8) | Maps one-to-one to requirements and likely units of work | Hides cross-feature flows, such as trip → packing → gap finder |
| Persona-based | By Everyday dresser, Traveler, Occasion attendee | Keeps each user's needs clear | Closet and profile stories serve all personas and get duplicated |
| Domain-based | By domain (wardrobe, weather and places, planning, packing) | Lines up with code modules | Reads like a technical design, not user value |
| Epic-based | Epics (one per feature or journey), each with small stories | Gives both a big picture and small, estimable stories | Needs a clear rule for what an epic is |

## Question 1
How should the stories be organized?

A) Epic-based hybrid: one epic per MVP feature (8 epics, matching FR-1 to FR-8), each with small stories; plus a short "Key journeys" section that links stories across epics (e.g., "plan a trip end to end")

B) User journey-based: stories grouped by end-to-end journeys only

C) Feature-based: a flat list of stories per feature, with no epics or journeys

D) Persona-based: stories grouped under each persona

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2
How small should a story be?

A) Small: each story is one user-visible capability that one person can build and test in about 1–2 days (expected total: roughly 30–45 stories)

B) Medium: each story is a larger slice, roughly one per feature area (expected total: roughly 15–20 stories)

C) Very small: one story per screen action or API endpoint (expected total: 60+ stories)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
Which personas should be written?

A) The three personas from the vision (Everyday dresser, Traveler, Occasion attendee), each as a named, realistic character (age, context, goals, frustrations, tech comfort, typical closet size)

B) The three vision personas plus a fourth: a **first-time user** whose closet is still empty (focuses on onboarding, upload and tagging)

C) The three vision personas, described briefly (needs and goals only, no character details)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
What format should acceptance criteria use?

A) Given / When / Then scenarios (easy to turn into pytest tests)

B) Checklist of bullet points ("The outfit includes shoes", "The weather source is labeled")

C) Given / When / Then for the main scenario and edge cases, plus a short checklist for simple rules

X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 5
How should failure and edge cases (LLM unavailable, empty closet, rejected upload, out-of-range forecast, luggage over capacity) be covered?

A) As acceptance criteria inside the related story (e.g., the daily outfit story includes "Given the LLM is unavailable...")

B) As their own separate stories (e.g., "As a user, when the AI is unavailable, I can still browse my closet")

C) Both: cross-cutting failures (LLM unavailable, graceful degradation) get their own stories; feature-specific edge cases go in the related story's criteria

X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 6
How should non-functional requirements (security, accessibility, privacy notice, performance) appear?

A) Only as references: each story lists the NFR IDs it must meet (e.g., "NFR-S.4, NFR-U.1"); NFRs stay in `requirements.md`

B) User-visible NFRs become stories (privacy notice, accessible keyboard navigation, response-time expectations); the rest stay as references

C) No NFRs in stories at all

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
Should stories carry extra fields to help with Units Generation and testing later?

A) Yes: story ID (e.g., `US-3.2`), persona(s), traceability to FR/NFR IDs, and a size estimate (S / M / L)

B) Yes, but only story ID, persona(s) and FR/NFR traceability (no size estimate)

C) No: only the story text and acceptance criteria

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
Where should the curated dress-code guide content (the ~15–20 event and venue types) be defined?

A) Not in the stories: the stories only say the guide exists and how it is used; the list of types and their guidance is written in Functional Design

B) Stories include the list of types the guide must cover (as acceptance criteria), with the detailed guidance written in Functional Design

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Part A Decisions (answers recorded 2026-11-19)

| Question | Answer | Generation rule |
| --- | --- | --- |
| Q1 Organization | A | 8 epics (EPIC-1 to EPIC-8, matching FR-1 to FR-8) plus EPIC-9 for cross-cutting stories (guardrails, privacy notice, failures); a "Key journeys" section links stories across epics |
| Q2 Size | A | Each story is one user-visible capability, about 1–2 days for one person; about 30–45 stories in total |
| Q3 Personas | A | Three named, realistic characters: Everyday dresser, Traveler, Occasion attendee (age, context, goals, frustrations, tech comfort, closet size) |
| Q4 Acceptance criteria | C | Given/When/Then for the main scenario and each edge case; a short checklist for simple one-condition rules (e.g., "the weather source is labeled") |
| Q5 Failures | C | Cross-cutting failures (LLM unavailable/not configured, upstream weather or places failure, tool-call cap reached) get their own stories in EPIC-9; feature-specific edge cases (empty closet, rejected upload, over capacity, unparseable itinerary) are acceptance criteria in the related story |
| Q6 NFRs | A | NFRs appear as ID references on each story (e.g., "NFR-S.4, NFR-U.1"). Interpretation: where a cross-cutting failure story from Q5 also corresponds to an NFR (e.g., graceful degradation, NFR-R.3), the story exists because of Q5 and references the NFR |
| Q7 Fields | A | Each story has: ID (`US-<epic>.<n>`), persona(s), FR/NFR traceability, size (S / M / L) |
| Q8 Dress-code guide | A | Stories only state that the curated guide exists and how it is used; the list of types and their guidance is defined in Functional Design |

---

## Part B: Execution Checklist (Part 2, Generation)

This checklist is completed after the team answers Part A and approves this plan. Each item is marked `[x]` as it is done.

### Step 1: Prepare
- [x] 1.1 Re-read `requirements.md` (FR-1 to FR-9, NFRs, decisions D1–D25, assumptions A1–A4) and the answers in Part A
- [x] 1.2 Confirm the story organization, size, persona set, acceptance criteria format and extra fields chosen in Part A

### Step 2: Personas
- [x] 2.1 Write the personas chosen in Question 3 in `aidlc-docs/inception/user-stories/personas.md`
- [x] 2.2 For each persona: goals, frustrations, context of use, and which features matter most
- [x] 2.3 Add a persona-to-feature map (table)

### Step 3: Stories
- [x] 3.1 Create `aidlc-docs/inception/user-stories/stories.md` with a header explaining the structure, ID scheme and acceptance criteria format
- [x] 3.2 Write stories for FR-1 Digital closet and tagging
- [x] 3.3 Write stories for FR-2 Style profile and ratings
- [x] 3.4 Write stories for FR-3 Daily outfit (including today/tomorrow, saved by date, Regenerate, "I wore this")
- [x] 3.5 Write stories for FR-4 Occasion planner
- [x] 3.6 Write stories for FR-5 Trip planner (text and PDF, multi-city, parsed-day review)
- [x] 3.7 Write stories for FR-6 Packing optimizer (bulk capacity, re-wear rules, "Buy before you go")
- [x] 3.8 Write stories for FR-7 Gap finder
- [x] 3.9 Write stories for FR-8 Explain and refine
- [x] 3.10 Cover FR-9 guardrails (closet-ID-only outfits, tool-call cap, privacy notice) as stories or acceptance criteria, as decided in Questions 5–6
- [x] 3.11 Cover failure and edge cases as decided in Question 5
- [x] 3.12 Cover NFRs as decided in Question 6
- [x] 3.13 Add the "Key journeys" section if chosen in Question 1

### Step 4: Quality Checks
- [x] 4.1 Check every story against INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable) and split or merge where needed
- [x] 4.2 Check that every story has acceptance criteria in the chosen format
- [x] 4.3 Check that every persona is mapped to stories and every story to at least one persona
- [x] 4.4 Check that every FR (FR-1 to FR-9) is covered by at least one story (coverage table at the end of `stories.md`)
- [x] 4.5 Validate markdown and any diagrams per `common/content-validation.md`

### Step 5: Wrap-up
- [x] 5.1 Update `aidlc-docs/aidlc-state.md`
- [x] 5.2 Log completion and the approval prompt in `aidlc-docs/audit.md`
- [x] 5.3 Present the User Stories completion message for review
