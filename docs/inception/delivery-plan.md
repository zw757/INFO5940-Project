# Delivery Plan

## Work Units

| Unit | Deliverable | Stories | Requirements | Depends on |
| --- | --- | --- | --- | --- |
| U1 Foundation | Python app, simple web UI, auth, database, CI | US-1.1 foundation | FR-1.1 | — |
| U2 Profile | Style, size, budget, context forms | US-1.1, US-3.1 context | FR-1.2–FR-1.3 | U1 |
| U3 Closet | Upload, classify, confirm, browse, edit, delete | US-2.1–US-2.2 | FR-2.1–FR-2.5 | U1 |
| U4 Outfits | Hosted AI, weather/place tools, generate, swap, save, rate | US-3.1–US-3.2 | FR-1.4, FR-3.1–FR-3.5 | U2, U3 |
| U8 Plans | Itinerary upload, event details, review, activity outfits, packing lists | US-5.1–US-5.2 | FR-6.1–FR-6.5 | U4 |
| U5 Analysis | Daily/trip/event gaps and reuse suggestions | US-4.1, US-5.2 gaps | FR-4.1–FR-4.3 | U4, U8 |
| U6 Shopping | Catalog, recommendations, cart, approval | US-4.2–US-4.3 | FR-5.1–FR-5.7 | U2, U5 |
| U7 Release | Integration checks, deployment, demo | All | NFR-1–NFR-9 | U6 |

U2 and U3 can run in parallel. Test each unit as it is built; assign team owners before construction.

## Delivery Sequence

1. **Working closet:** foundation, profile, and manual clothing entry work in a preview deployment.
2. **AI styling:** photo analysis is editable; users generate, swap, save, and rate outfits.
3. **Travel and events:** users confirm itinerary details, review activity outfits, and edit packing lists.
4. **Shopping review:** daily/trip/event gaps produce explained recommendations and an editable cart with simulated approval.
5. **Final demo:** all journeys pass tests and a team rehearsal.

Milestone documents are due November 24; the final project is due December 16.

## Completion Checks

- Closet, daily outfit, travel/event planning, and shopping journeys work with seeded data.
- Tests cover budget, size, ownership, invalid AI output, weather/place tool failures, and approval.
- Empty closets, unreadable itineraries, missing event details, and no product matches have useful responses.
- Packing lists consolidate repeated items and separate owned clothes from suggested purchases.
- Keyboard navigation and phone layouts work.
- Sample products and simulated checkout are clearly labeled.

Team review of scope, implementation details, open providers, and unit ownership remains pending. The Python/simple web UI direction is selected.

