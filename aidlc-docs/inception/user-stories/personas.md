# Personas: ClosetCast

These three personas come from the target users in `docs/vision.md` (Everyday dresser, Traveler, Occasion attendee). They are fictional composites, used to write stories, choose demo scenarios and brief outside testers during testing week. Every persona is the single local user of their own ClosetCast installation.

---

## P1: Jordan Reyes — the Everyday Dresser

| Attribute | Detail |
| --- | --- |
| Age / role | 20, third-year undergraduate in Ithaca, NY |
| Context of use | Laptop on the desk, usually the night before or 10 minutes before leaving; checks the weather every morning |
| Closet | About 60 items: mostly casual tops, jeans, two jackets, a puffer coat, sneakers and boots, one "nice" outfit |
| Tech comfort | High; uses apps daily but won't tolerate slow or fiddly setup |
| Style | Minimal and streetwear; avoids bright orange; runs cold |

**Goals**
- Pick an outfit that fits the weather and the day (class, gym, dinner downtown) without thinking about it
- Wear more of the closet, not the same 3 outfits
- Decide the night before so mornings are quick

**Frustrations**
- Ithaca weather swings: leaves in a light jacket and freezes by evening
- Forgets what was worn yesterday and repeats outfits
- Typing a whole closet into an app sounds exhausting

**Features that matter most**: Closet tagging (FR-1), Style profile and ratings (FR-2), Daily outfit (FR-3), Refine (FR-8)

**Quote**: "Just tell me what to wear tomorrow. And don't make me type in 60 shirts."

---

## P2: Priya Nair — the Traveler

| Attribute | Detail |
| --- | --- |
| Age / role | 24, first-year master's student; travels for spring break, conferences and family visits |
| Context of use | Plans trips 1–6 weeks ahead on a laptop; itineraries arrive as pasted text from friends or PDF confirmations |
| Closet | About 90 items, including business-casual pieces for conferences |
| Tech comfort | Medium-high; comfortable with spreadsheets and PDFs |
| Style | Preppy and smart casual; comfort rule: "flat shoes on walking days" |

**Goals**
- A day-by-day outfit plan for a multi-city trip (e.g., Barcelona days 1–3, Madrid days 4–5) that fits each day's weather and places
- Carry-on only: the fewest items that make enough distinct outfits
- Know in advance if something is missing (rain jacket, covered-shoulders top for a church visit)

**Frustrations**
- Overpacks "just in case" and still forgets the one thing needed
- Doesn't know the dress norms of unfamiliar places (churches, rooftop bars)
- Forecasts aren't available for trips more than two weeks away

**Features that matter most**: Trip planner (FR-5), Packing optimizer (FR-6), Gap finder (FR-7), Refine (FR-8)

**Quote**: "I want to open my suitcase and know every piece has a job."

---

## P3: Marcus Okafor — the Occasion Attendee

| Attribute | Detail |
| --- | --- |
| Age / role | 23, first year in an entry-level job |
| Context of use | A few times a semester: a wedding, an interview, a gala, a funeral; plans a few days to a few weeks ahead |
| Closet | About 45 items; one suit, two dress shirts, dress shoes, mostly casual otherwise |
| Tech comfort | Medium; wants clear answers, not fashion jargon |
| Style | Classic and simple; unsure about formal dress codes |

**Goals**
- Understand what a dress code like "cocktail attire" or "garden-party semi-formal" actually means
- Find out whether the closet already has a suitable outfit
- If not, know exactly what to buy and why, before the event

**Frustrations**
- Ambiguous invitations; afraid of being underdressed
- Outdoor events where the weather changes what's appropriate (grass, heat, rain)
- Shopping without knowing what to look for

**Features that matter most**: Occasion planner (FR-4), Gap finder (FR-7), Refine (FR-8), Closet tagging (FR-1)

**Quote**: "Tell me what 'cocktail attire' means for me, with the clothes I actually have."

---

## Persona-to-Feature Map

| Feature | P1 Jordan (Everyday) | P2 Priya (Traveler) | P3 Marcus (Occasion) |
| --- | --- | --- | --- |
| FR-1 Digital closet and tagging | Primary | Primary | Primary |
| FR-2 Style profile and ratings | Primary | Secondary | Secondary |
| FR-3 Daily outfit | Primary | — | — |
| FR-4 Occasion planner | — | Secondary | Primary |
| FR-5 Trip planner | — | Primary | — |
| FR-6 Packing optimizer | — | Primary | — |
| FR-7 Gap finder | Secondary | Primary | Primary |
| FR-8 Explain and refine | Primary | Primary | Primary |
| FR-9 Guardrails and privacy (cross-cutting) | All | All | All |
