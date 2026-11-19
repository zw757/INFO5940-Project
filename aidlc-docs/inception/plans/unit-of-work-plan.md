# Unit of Work Plan: ClosetCast

**Stage**: INCEPTION - Units Generation, Part 1 (Planning)
**Inputs**: `requirements.md`, `stories.md` (41 stories), `application-design/` (components C1–C14, services, shared contracts), `execution-plan.md`, README collaboration plan (3 members; each owns units; feature branches; PRs reviewed by a teammate)
**Outputs** (in `aidlc-docs/inception/application-design/`): `unit-of-work.md`, `unit-of-work-dependency.md`, `unit-of-work-story-map.md`

A **unit of work** here is a logical module of the single ClosetCast app (a monolith). Units are not deployed separately. Each unit goes through the Construction loop (Functional Design → NFR Requirements → NFR Design → Code Generation) before the next unit starts its loop.

---

## Part A: Questions

**How to answer:** put a letter after each `[Answer]:` tag; add a short reason if you like. If no option fits, choose the last option (Other) and describe it. Tell Claude "done" when finished.

### Category check
| Category | Covered by |
| --- | --- |
| Story grouping and business domain | Q1 |
| Dependencies and integration | Q2, Q5 |
| Team alignment | Q3, Q4 |
| Code organization | Q6 |
| Technical considerations (scaling, deployment) | Not asked: every unit runs in the same local process with the same deployment, so there are no differences between units to decide |
| Scope risk | Q7 |

## Question 1: How should the stories be grouped into units?

A) **(Recommended)** **4 units**: a shared foundation plus one feature unit per team member. *Reason: matches 3 members; each feature unit is a coherent business area; story counts are roughly balanced.*

| Unit | Contents | Stories |
| --- | --- | --- |
| U1 Foundation & Agent Core | Platform (config, logging, container, main), security middleware and upload validation, domain models and errors, DB base and repositories, LLM adapter, agent engine and tool registry, job runner, ContextBuilder with weather and geocoding tools, base templates, `/health`, privacy notice, cross-cutting failures | US-9.1–9.5 (5) |
| U2 Closet & Profile | Closet upload, tagging jobs, tag editing, manual items, delete; profile, ratings, preferences ranking, worn log | US-1.1–1.6, US-2.1–2.3 (9) |
| U3 Daily, Occasion & Refine | Daily outfit (today/tomorrow, variety), occasion planner, dress-code guide content, places lookup, refinement for all plan types | US-3.1–3.6, US-4.1–4.4, US-8.1–8.3 (13) |
| U4 Trip, Packing & Gaps | Itinerary parsing (text/PDF), trip planning job, packing optimizer, gap finder | US-5.1–5.7, US-6.1–6.5, US-7.1–7.2 (14) |

B) **7 smaller units**: Foundation, Closet & Profile, Daily, Occasion, Trip, Packing, Gaps & Refine (more design documents and more review cycles; finer ownership)

C) **1 unit** covering the whole app (fewest documents, but no natural way to split work between members)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 2: How should the Foundation unit (U1) be built?
Everything else depends on U1's shared contracts (domain models, error codes, LLM Protocol, repositories, container).

A) **(Recommended)** The whole team builds U1 together in the first sessions (one driver, screen shared, as the README describes for AI-DLC sessions); feature units start only after U1 is merged. *Reason: everyone understands the shared contracts; no one builds on guesses.*

B) One member builds U1 while the other two prepare work that doesn't depend on U1 code (e.g., the dress-code guide content, the 60-photo tagging test set, sample itineraries)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3: Who owns which unit?
Team: May Wu, Katherine Yang, Emma Dai.

A) Assign now: write the owner of U2, U3 and U4 after the letter (e.g., `A, U2 = …, U3 = …, U4 = …`); U1 is shared

B) Decide at the first Construction meeting (Nov 24); the unit documents show "Owner: TBD"

X) Other (please describe after [Answer]: tag below)

[Answer]: X claude assigns evenly

## Question 4: How should the AI-DLC Construction loop run with three people?
AI-DLC completes each unit's design and code before moving to the next unit, and logs everything in one `audit.md` and one `aidlc-state.md`.

A) **(Recommended)** **Sequential AI-DLC sessions, parallel human work:** run the Construction loop for one unit at a time in dependency order (U1 → U2 → U3 → U4), in team sessions; the unit owner drives that unit's session and leads its review, while the others test, review PRs, or prepare data for their own unit. *Reason: one clean audit trail and state file, no merge conflicts in `aidlc-docs/`, and it follows the AI-DLC rule of finishing a unit before starting the next.*

B) **Parallel AI-DLC sessions on feature branches:** after U1, each owner runs their unit's loop on their own branch at the same time; `audit.md` and `aidlc-state.md` changes are merged by hand (faster wall-clock time, but expect merge conflicts in the audit log and state file)

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5: How are changes to shared contracts handled after U1?
For example, U4 needs a new field in `domain/models.py` or a new error code.

A) **(Recommended)** Shared-contract changes are **additive only** (new fields with defaults, new codes, never renamed or removed) and go in a small separate PR that all three members review. *Reason: no unit breaks another unit's code.*

B) Any change is allowed in the owner's feature PR, reviewed by one teammate

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 6: Code organization
The AI-DLC default for a multi-unit monolith is one folder per unit (`src/{unit-name}/`). The approved technical environment and application design use one package with layers (`src/closetcast/{api,web,services,agent,tools,domain,db,...}`).

A) **(Recommended)** Keep the **layered single package** from the technical environment; each unit owns specific files across the layers (listed in `unit-of-work.md`), and tests mirror the layers (`tests/unit/...`, `tests/integration/...`). *Reason: consistent with the approved technical environment and application design; avoids circular imports between unit folders.*

B) One subpackage per unit (`src/closetcast/closet/`, `src/closetcast/trips/`, ...), each with its own routes, services and repositories; shared code in `src/closetcast/core/`

X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7: If time runs short, what gets simplified first?
The vision says features 7 (gap finder) and 8 (refine) can be simplified.

A) **(Recommended)** In this order: (1) refinement supports **trip plans only** (daily: use Regenerate; occasion: re-plan); (2) gap finder uses code-detected gaps with **template text** instead of LLM-written details; (3) PDF itinerary upload is dropped (text paste stays). The critical path (closet → daily → trip → packing) is never cut.

B) Same as A, but keep refinement for all plan types and simplify the gap finder first

X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Follow-up Questions (from the answer to Question 3)

You asked Claude to assign the units **evenly**. Two things need confirming first.

**Ambiguity 1: The Q1 units are not even in effort.** Weighting story sizes as S = 1, M = 2, L = 3:

| Unit (Q1 = A) | Stories | Effort points |
| --- | --- | --- |
| U2 Closet & Profile | 9 | 12 |
| U3 Daily, Occasion & Refine | 13 | 19 |
| U4 Trip, Packing & Gaps | 14 | 24 |

Moving the **gap finder** (US-7.1, US-7.2; 4 points) from U4 to U2 gives **16 / 19 / 20**, the most even split possible without breaking a feature apart. The gap finder works in the same area as the closet: it compares what is needed against what the user owns.

### Follow-up Question 1
How should "evenly" be applied to the units?

A) Rebalance: move the gap finder to U2, giving **U2 Closet, Profile & Gaps (16)**, **U3 Daily, Occasion & Refine (19)** and **U4 Trip & Packing (20)**. Also, the U2 owner prepares the 60-photo tagging test set (NFR-Q.1), which offsets U2's slightly smaller size

B) Keep the Q1 groupings as they are (12 / 19 / 24); "evenly" only means one unit per person

X) Other (please describe after [Answer]: tag below)

[Answer]: A *(answered in chat: "1: A")*

**Ambiguity 2: Claude has no information about each member's strengths or interests**, so any name assignment is arbitrary. Proposed assignment (alphabetical by first name, in unit order U2 → U4):

| Unit | Proposed owner |
| --- | --- |
| U2 | Emma Dai |
| U3 | Katherine Yang |
| U4 | May Wu |

### Follow-up Question 2
Is this owner assignment OK?

A) Yes, use the proposed assignment

B) Swap owners (write the new assignment after the letter, e.g., `B, U2 = …, U3 = …, U4 = …`)

X) Other (please describe after [Answer]: tag below)

[Answer]: B, U2 = May Wu, U3 = Katherine Yang, U4 = Emma Dai *(answered in chat: "2: swap emma and may, others are fine")*

---

## Final Decisions

| Decision | Result |
| --- | --- |
| Units | U1 Foundation & Agent Core (shared); U2 Closet, Profile & Gaps (16 pts); U3 Daily, Occasion & Refine (19 pts); U4 Trip & Packing (20 pts) |
| Owners | U1 = whole team; U2 = May Wu (also prepares the 60-photo tagging test set); U3 = Katherine Yang; U4 = Emma Dai |
| Foundation | Built together first; feature units start after U1 is merged (Q2 = A) |
| AI-DLC sessions | Sequential per unit in order U1 → U2 → U3 → U4; owner drives (Q4 = A) |
| Shared contracts | Additive-only changes in a separate PR reviewed by all (Q5 = A) |
| Code organization | Layered single package `src/closetcast/` (Q6 = A) |
| Scope reduction order | Refine trips only → template gap text → drop PDF upload (Q7 = A) |

---

## Part B: Execution Checklist (Part 2, Generation)

Completed after Part A is answered and the plan is approved. Each item is marked `[x]` when done.

### Step 1: Unit Definitions (`unit-of-work.md`)
- [x] 1.1 Define each unit: purpose, scope, owner, components and files owned, shared contracts provided or consumed
- [x] 1.2 Document the code organization strategy (greenfield), per Question 6
- [x] 1.3 Document the Construction sequence and working model, per Questions 2, 4 and 5
- [x] 1.4 Document the scope-reduction order, per Question 7
- [x] 1.5 Per unit, note which Construction stages apply (Functional Design, NFR Requirements, NFR Design, Code Generation) and the PBT targets

### Step 2: Dependencies (`unit-of-work-dependency.md`)
- [x] 2.1 Dependency matrix between units
- [x] 2.2 Shared contracts and which unit provides each
- [x] 2.3 Build order and integration checkpoints (Mermaid with text alternative)

### Step 3: Story Map (`unit-of-work-story-map.md`)
- [x] 3.1 Map all 41 stories to exactly one owning unit
- [x] 3.2 Note stories that touch more than one unit and how they're split
- [x] 3.3 Coverage check: every story assigned; key journeys J1–J5 traced across units

### Step 4: Validate and Wrap Up
- [x] 4.1 Validate unit boundaries (no cyclic unit dependencies; every component C1–C14 owned by a unit)
- [x] 4.2 Security Baseline compliance check for this stage
- [x] 4.3 Validate markdown and Mermaid per `common/content-validation.md`
- [x] 4.4 Update `aidlc-state.md` (including the per-unit Construction list), log in `audit.md`, present the completion message
