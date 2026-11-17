# Product Vision

## Purpose

Help people decide what to wear, pack, or buy using their existing wardrobe. The app combines a digital closet, outfit planning for daily life, travel, and special occasions, and a shopping assistant that explains what each purchase adds.

Target users include people seeking faster daily outfit decisions, travelers planning what to pack, and shoppers preparing for an event within a budget.

## MVP Scope

1. Capture style preferences, clothing sizes, and a monthly budget.
2. Upload, classify, review, edit, and delete clothing items.
3. Generate outfits for weather and occasion, using weather/place lookups when needed; support swaps, saves, and feedback.
4. Accept itinerary uploads or event details, let users confirm extracted plans, and generate activity-specific outfits and an editable packing list.
5. Identify missing pieces for everyday use, trips, or events.
6. Recommend products from a curated catalog, showing price and compatible outfits.
7. Let users edit a monthly cart and approve simulated checkout.

The agent can recommend buying nothing. Real payments, live retailer inventory, receipt imports, detailed wear tracking, and native mobile apps are outside the MVP.

## Main Journey

Set preferences → upload clothes → request a daily outfit or provide travel/event plans → review outfits and a packing list when needed → review missing pieces → edit and approve a simulated shopping cart.

## Success Criteria

- A user completes the closet, daily outfit, travel/event planning, and shopping journeys.
- Packing lists reuse versatile pieces, consolidate repeated items, and distinguish owned clothes from suggested purchases.
- Each product recommendation explains its value using specific closet items.
- Recommendations respect confirmed sizes and the cart stays within budget.
- Users can correct AI output, reject suggestions, and recover from failed requests.
- Usability testing checks whether users understand and find the suggestions useful.

## Constraints and Risks

The three-person team targets the November 24 milestone and December 16 final project. A curated catalog and simulated checkout keep the scope manageable.

Inaccurate clothing or itinerary analysis is handled through user review and manual entry. Server validation enforces constraints even when AI output is incorrect. Private storage protects photos and uploaded itineraries.

## Open Decisions

- Select the hosted vision/tool-calling model and weather/places API providers.
- Confirm frameworks and hosting for the chosen Python backend and simple web UI.
- Choose supported itinerary formats and upload limits.
- Select sample wardrobes, itineraries, events, and products for the demo.

