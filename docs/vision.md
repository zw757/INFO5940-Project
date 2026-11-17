# Vision: ClosetCast — An AI Stylist That Plans Outfits From the Clothes You Already Own

## Executive Summary

ClosetCast is a locally run web app that acts as a personal stylist built on the user's real wardrobe. The user photographs their clothes once; an LLM with vision tags every item (category, color, formality, warmth). From then on, ClosetCast answers three questions: *What should I wear today?*, *What should I wear to this event?*, and *What should I pack for this trip?* An LLM agent answers each one by calling tools: a weather service, a place and dress-code lookup, the user's closet database, and a packing optimizer. It then explains its choices. When the closet can't cover a situation (no rain jacket for a rainy trip, nothing formal enough for a black-tie wedding), ClosetCast says what is missing, why it matters, and what to look for when buying it. The product is for students and young professionals who don't want to think about getting dressed every morning, who travel with limited luggage, or who have occasions with dress codes they don't fully understand.

## Problem Statement

Deciding what to wear means weighing several things at once: the weather (often at a destination the user has never been to), what the user is doing that day, the dress expectations of each place (a temple that requires covered shoulders, a restaurant with a jacket rule, an outdoor wedding on grass), personal style, and, for trips, how much fits in a suitcase. People handle this badly in two ways. Day to day, they waste time and fall back on the same few outfits while most of the closet goes unworn. For trips and events, they overpack "just in case," forget the one item they needed, or show up underdressed because a dress code like "cocktail attire" or "smart casual" was ambiguous.

Existing closet apps catalog clothes and match outfits to a temperature, but they treat each day separately and mostly ignore *where* the user is going. Packing-list apps don't know what the user owns. Neither can interpret an unstructured itinerary ("Day 2: Sagrada Família in the morning, tapas dinner, then a rooftop bar") or explain why an outfit fits. These tasks need language understanding (reading itineraries and dress codes), visual understanding (recognizing clothing from photos), and reasoning over several constraints at once. That combination is what an LLM agent with tools can do and a rules-based app cannot.

## Target Users

| User type | Description | Primary needs |
| --- | --- | --- |
| Everyday dresser | Student or young professional getting ready on a normal day | A quick, weather-appropriate outfit from their own closet; variety beyond the same 3 outfits |
| Traveler | Takes 2–10 day trips (spring break, conferences, visiting family, study abroad), often carry-on only | A day-by-day outfit plan and a minimal packing list that covers every day's weather and activities |
| Occasion attendee | Has a specific event: wedding, interview, formal, gala, funeral, first date | An explanation of what the dress code means and an outfit from their closet, or what to buy if they don't own one |

All users are a single local user per installation for the MVP (no accounts, no sharing).

## Success Metrics

| Metric | Target | How measured |
| --- | --- | --- |
| Clothing tagging accuracy | ≥ 85% of items get the correct category and primary color with no user edits | Team test set of 60 labeled clothing photos |
| Closet faithfulness | 100% of recommended outfits use only items in the user's closet (anything else is labeled as a "buy" suggestion) | Automated check in tests; verified by code, not the LLM |
| Weather appropriateness | 0 outfits that contradict the weather data (e.g., no coat when the high is above 27°C / 80°F, a rain layer when rain probability ≥ 60%) | Rule-based validator run on 20 test scenarios |
| Trip coverage | 100% of itinerary days get an outfit; packing list contains every item used in the plan | Automated check on 10 sample itineraries |
| Packing efficiency | For a 5-day carry-on trip, ≤ 12 clothing items (excluding underwear, socks and sleepwear) produce ≥ 5 distinct outfits | Packing optimizer output on sample itineraries |
| Response time | Daily outfit ≤ 15 s; 7-day trip plan ≤ 90 s on a laptop with a normal connection | Timed in demo runs |
| User satisfaction | Average ≥ 4/5 "would wear this" rating from team members and 5 outside testers | Ratings collected in the app during testing week |

## Full Scope Vision

What ClosetCast could become past this course:

- **Wardrobe intelligence:** cost-per-wear tracking, "you haven't worn this in 6 months" nudges, donate/resell suggestions, laundry-aware planning.
- **Shopping:** price comparison across retailers, secondhand-first suggestions, links to buy, alerts when a wishlist item goes on sale.
- **Trips:** import itineraries from email, calendar or booking confirmations; multi-traveler trips (coordinating a group's outfits for a wedding party); weather-change alerts during the trip with re-planning.
- **Visual experience:** virtual try-on, auto-generated outfit collages with background removal, mobile app with camera capture.
- **Social:** share an outfit plan with a friend for feedback; borrow items from friends' closets.
- **Learning:** a style model that learns from what the user actually wears, not just ratings.

## MVP Scope — Features IN

| # | Feature | Description | Rationale |
| --- | --- | --- | --- |
| 1 | Digital closet with AI tagging | Upload photos of clothing (one item per photo, JPEG/PNG, up to 10 MB). A vision LLM returns structured tags: category, subcategory, primary/secondary color, pattern, formality (1–5), warmth (1–5), season, and rain-suitability. The user can review and edit any tag. Items can also be added manually without a photo. | Everything else depends on knowing what the user owns. Vision tagging removes the main barrier: typing in a whole closet by hand. |
| 2 | Style profile | A short onboarding form: preferred styles (e.g., minimal, streetwear, preppy), colors to avoid, comfort priorities (e.g., "no heels," "always flat shoes for walking days"), temperature sensitivity (runs cold / normal / runs warm), and preferred units (°F/°C). Users can rate any recommended outfit thumbs up or down with an optional comment. | Recommendations must reflect the person's taste, not generic fashion rules. Ratings give the agent feedback to use in later recommendations. |
| 3 | Daily outfit recommendation | User asks for today's outfit, optionally describing plans ("class, then gym, then dinner downtown"). The agent fetches today's weather for the user's home city, picks a primary outfit plus one alternative from the closet, and explains why (weather, activities, style). | Covers the most frequent use case and is the simplest end-to-end path through the agent. |
| 4 | Special occasion planner | User describes an event in free text (type, date, location, stated dress code, indoor/outdoor). The agent interprets the dress code using a curated dress-code guide, checks the weather at that place and date, and either proposes outfits from the closet or reports what is missing. | Dress codes are ambiguous and stressful to interpret. Turning "garden-party cocktail attire in June" into concrete guidance is a language task an LLM handles well. |
| 5 | Trip planner from an itinerary | User pastes an itinerary as text or uploads it as a PDF, then sets luggage size (personal item, carry-on, checked). The agent extracts each day's city, places and activities into a structured plan, fetches weather for each day (a forecast if the day is within 16 days; otherwise historical averages for those dates), looks up dress norms for each place, and produces a day-by-day outfit plan. | This is the most distinctive feature: it reasons across the whole trip instead of planning each day separately. |
| 6 | Packing optimizer | From the trip plan, a deterministic optimizer (plain Python, not the LLM) picks the smallest set of items that covers every day's outfit needs within the luggage limit and the user's re-wear rules (default: bottoms and outerwear may repeat; tops may repeat after 2 days). Outputs a packing checklist and an outfit matrix (item × day). | LLMs are unreliable at combinatorial counting. Delegating this to code gives provably consistent results and is a clear example of an agent using a tool. |
| 7 | Gap finder (what to buy) | When the closet can't satisfy a requirement (weather, dress code, activity), the agent produces a prioritized "consider buying" list: item type, required attributes (e.g., "waterproof, packable, neutral color to match your navy and grey items"), why it is needed, and which days or events it covers. Each suggestion includes a generic web search link; no prices are scraped. | Delivers the "what should I buy" value from the original idea without fragile retailer scraping. |
| 8 | Explain and refine | Every recommendation includes a short explanation. The user can give feedback in plain text ("no skirts this trip," "swap the blazer for something warmer," "I'll be walking 20,000 steps on day 3") and the agent revises the plan while keeping the rest of it unchanged. | Lets users trust and correct the agent, and turns a one-shot generator into a real assistant. |

## MVP Scope — Features OUT

| Feature | Reason for exclusion | Target phase |
| --- | --- | --- |
| Price comparison across retailers / scraping store websites | No free universal shopping API; scraping is brittle, often against retailers' terms of service, and would consume most of Construction | Phase 2 (via an official shopping API if one becomes available) |
| Buying or checkout inside the app, affiliate links | Not needed to validate the core value; adds legal and payment complexity | Phase 3 |
| Virtual try-on / AI image generation of outfits | Expensive, slow, and unreliable on free tiers; not core to the decision | Phase 3 |
| Automatic background removal and outfit collage images | Nice visual polish but not required for the demo; outfit cards show the original item photos side by side | Phase 2 |
| Mobile app | The course requires only a locally running product; a responsive web page is enough | Phase 3 |
| User accounts, login, multi-user, cloud hosting | Product runs locally for a single user; the course does not require hosting | Phase 2 |
| Importing itineraries from email, Google Calendar or booking sites | OAuth integrations add significant work; pasted text and PDF cover the core use case | Phase 2 |
| .ics calendar file import | Lower priority than text/PDF; may be added if time allows | Phase 2 |
| Laundry tracking and cost-per-wear analytics | Requires daily logging habits; not needed for the three core questions | Phase 2 |
| Weather alerts and re-planning during a trip (notifications) | Requires background jobs and notifications | Phase 2 |
| Group or shared trips | Multi-user coordination is out of scope for a single-user local app | Phase 3 |
| Learning a style model from wear history | Ratings and the style profile are enough for the MVP | Phase 3 |

## Risks and Open Questions

### Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Vision model mis-tags items (e.g., navy vs. black, blazer vs. shirt jacket) | Bad outfit suggestions | User review/edit step after tagging; tags stored as structured fields the user can correct |
| LLM recommends items the user doesn't own | Breaks user trust | Agent may only select items by closet ID; code validates every outfit against the database and rejects unknown IDs |
| Hosted LLM free-tier rate limits during bulk photo tagging | Slow onboarding, failed requests | Tag photos in a queue with retry and backoff; cache results; let the user tag in batches |
| Weather forecast unavailable beyond 16 days | Trip plans for distant dates lack forecasts | Fall back to historical averages for the same dates in recent years, and label the plan as "based on typical weather" |
| Dress-code guidance for a specific venue is wrong or unknown | User underdressed | Use a curated dress-code guide for common venue and event types; when uncertain, the agent says so and suggests the safer (more formal) option |
| Clothing photos sent to a hosted API raise privacy concerns | User discomfort | Clear notice before upload; send only clothing photos, never personal data; API key stored locally |
| Scope is too large for the Construction window (Nov 24 – Dec 16) | Unfinished features | Features 1, 3, 5, 6 are the critical path; 7 and 8 can be simplified if needed |

### Open Questions

- Should itinerary input in the MVP accept both pasted text and PDF upload, or only pasted text first?
- What exact threshold switches from forecast to historical averages (the technical environment assumes 16 days, the forecast limit), and is a 5-year average of the same dates a good enough estimate?
- Should the dress-code knowledge be a curated local guide only, or may the LLM fill gaps from its own knowledge when a venue type isn't in the guide (with a confidence label)?
- What are the right default re-wear rules for the packing optimizer, and should users be able to change them per trip?
- How should the optimizer handle items that are needed but not owned: plan around the gap, or assume the user will buy the suggested item?
- Should an outfit always include shoes and outerwear, or only when the weather or dress code calls for them?
- How should thumbs-up/down ratings influence future recommendations in the MVP: as notes the agent reads in its prompt, or as stored preferences?
- Should the daily outfit use the user's home city only, or detect the current location?
- How many clothing photos should onboarding support at once without hitting API rate limits (e.g., batches of 10)?
