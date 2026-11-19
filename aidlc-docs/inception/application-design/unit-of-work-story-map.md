# Unit of Work Story Map: ClosetCast

Each of the 41 stories is assigned to exactly **one owning unit**. The owning unit delivers the story's acceptance criteria; other units may supply parts of it, as noted in the last column.

| Story | Title | Size | Owning unit | Contributions from other units |
| --- | --- | --- | --- | --- |
| US-1.1 | Upload a batch of clothing photos | M | U2 | U1 upload validation, job runner |
| US-1.2 | See tagging progress and retry failures | M | U2 | U1 job runner, polling partial |
| US-1.3 | Review and edit AI tags | S | U2 | — |
| US-1.4 | Add an item manually | S | U2 | — |
| US-1.5 | Browse and filter my closet | S | U2 | — |
| US-1.6 | Delete an item | S | U2 | U3/U4 plan pages display "removed from closet" |
| US-2.1 | Set up and edit my style profile | S | U2 | U1 geocoding, units |
| US-2.2 | Rate an outfit | S | U2 | U3/U4 place the rating control on their outfit cards |
| US-2.3 | Recommendations respect my ratings | M | U2 | U3 verifies it in daily recommendations |
| US-3.1 | Get today's outfit | M | U3 | U1 weather, validation; U2 closet search, gaps |
| US-3.2 | Tailor the outfit to my plans | S | U3 | — |
| US-3.3 | Plan tomorrow's outfit the night before | M | U3 | — |
| US-3.4 | Regenerate a saved daily outfit | S | U3 | — |
| US-3.5 | Get an outfit for a different city | S | U3 | U1 geocoding |
| US-3.6 | Log what I wore and get variety | M | U3 | U2 worn log storage (ProfileRepository) |
| US-4.1 | Understand an event's dress code | M | U3 | — |
| US-4.2 | Get an outfit for the event | M | U3 | U1 weather (forecast or typical); U2 gaps |
| US-4.3 | Get guidance for an unusual event type | S | U3 | — |
| US-4.4 | Revisit my saved occasion plans | S | U3 | — |
| US-5.1 | Paste an itinerary and see the parsed days | M | U4 | — |
| US-5.2 | Upload an itinerary PDF | S | U4 | U1 PDF extraction helper |
| US-5.3 | Review and correct parsed days | M | U4 | — |
| US-5.4 | Weather for every day and city | M | U4 | U1 weather and geocoding, cache |
| US-5.5 | Dress norms for the places I visit | M | **U3** (moved) | U4 calls it from trip planning |
| US-5.6 | Get my day-by-day trip plan | L | U4 | U1 job runner, agent engine; U3 composition |
| US-5.7 | Revisit my saved trips | S | U4 | — |
| US-6.1 | Get a packing checklist that fits my luggage | L | U4 | — |
| US-6.2 | See which items I wear on which day | S | U4 | — |
| US-6.3 | Change re-wear rules for a trip | S | U4 | — |
| US-6.4 | Understand when clothes don't fit the luggage | S | U4 | — |
| US-6.5 | See what to buy before I go | S | U4 | U2 GapService |
| US-7.1 | See a "to buy" suggestion next to an affected outfit | M | U2 | U3/U4 show it on their outfit cards |
| US-7.2 | Get a prioritized "consider buying" list | M | U2 | — |
| US-8.1 | Refine a trip plan with plain-language feedback | M | **U4** (moved) | U3 RefinementService structure |
| US-8.2 | Refine an occasion plan or daily outfit | S | U3 | — |
| US-8.3 | Revised plans stay valid and packing stays in sync | S | **U4** (moved) | U1 validation |
| US-9.1 | Know what is sent to OpenAI | S | U1 | U2/U4 show the notice before the first upload |
| US-9.2 | Only clothes I own are recommended | M | U1 | Each planning unit calls validation |
| US-9.3 | Keep using ClosetCast when the AI is unavailable | M | U1 | Each unit verifies its own degraded behavior |
| US-9.4 | Handle weather or place lookup failures | S | U1 | U3 places fallback |
| US-9.5 | Get a partial answer when the AI hits its step limit | S | U1 | — |

## Totals by Unit

| Unit | Stories | S | M | L | Points |
| --- | --- | --- | --- | --- | --- |
| U1 | 5 (US-9.1–9.5) | 3 | 2 | 0 | 7 |
| U2 | 11 (US-1.1–1.6, 2.1–2.3, 7.1–7.2) | 6 | 5 | 0 | 16 |
| U3 | 12 (US-3.1–3.6, 4.1–4.4, 5.5, 8.2) | 6 | 6 | 0 | 18 |
| U4 | 13 (US-5.1–5.4, 5.6, 5.7, 6.1–6.5, 8.1, 8.3) | 7 | 4 | 2 | 21 |
| **Total** | **41** | **22** | **17** | **2** | **62** |

All 41 stories are assigned, each to exactly one unit.

## Key Journeys Across Units

| Journey | Units involved (in order) | Testable after |
| --- | --- | --- |
| J1 First-time setup → first outfit | U1 (notice) → U2 (profile, upload, tagging, edit) → U3 (daily) → U2 (rating) | Checkpoint 3 |
| J2 Plan tomorrow the night before | U3 (daily, variety) + U2 (worn log) | Checkpoint 3 |
| J3 Multi-city trip end to end | U4 (parse, review, plan) + U3 (place norms) + U4 (packing) + U2 (gaps) + U4 (refine, re-pack) | Checkpoint 4 |
| J4 Event with a dress code | U3 (parse, guide, plan) + U2 (gaps) + U3 (refine, saved plans) | Checkpoint 3 |
| J5 AI outage | U1 (degradation) verified in U2, U3 and U4 | Checkpoint 4 |
