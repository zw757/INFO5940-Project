# Requirements

## Functional Requirements

### Profile

- **FR-1.1:** Create and access a private profile.
- **FR-1.2:** Edit style, size, and monthly budget preferences.
- **FR-1.3:** Supply occasion, activities, dress code, and weather context for outfits.
- **FR-1.4:** Retrieve weather and place context through validated tools when needed; allow manual input if lookups fail and label unavailable forecasts.

### Closet

- **FR-2.1:** Upload supported clothing images.
- **FR-2.2:** Suggest category, color, style, material, and season.
- **FR-2.3:** Let users confirm or correct inferred attributes.
- **FR-2.4:** View, filter, edit, and delete items.
- **FR-2.5:** Restrict items and images to their owner.

### Outfits

- **FR-3.1:** Generate compatible combinations from owned clothes.
- **FR-3.2:** Account for occasion, weather, and preferences.
- **FR-3.3:** Swap individual pieces when alternatives exist.
- **FR-3.4:** Save, like, or dislike outfits.
- **FR-3.5:** Exclude items marked unavailable.

### Wardrobe Analysis

- **FR-4.1:** Identify missing pieces for daily outfits, trips, or events using the closet and confirmed plans.
- **FR-4.2:** Explain gaps using categories or outfit combinations.
- **FR-4.3:** Support reuse or “buy nothing” recommendations.

### Shopping

- **FR-5.1:** Evaluate products from a curated catalog.
- **FR-5.2:** Filter products by budget and confirmed size.
- **FR-5.3:** Explain each suggestion, show an outfit it complements, and identify the relevant activity or event when supplied.
- **FR-5.4:** Add, remove, or skip products.
- **FR-5.5:** Show cart total and remaining budget; block over-budget approval.
- **FR-5.6:** Require explicit confirmation for simulated checkout.
- **FR-5.7:** Label sample products and checkout as simulated.

### Travel and Event Planning

- **FR-6.1:** Upload an itinerary in a supported format or manually enter trip/event details.
- **FR-6.2:** Let users confirm or correct extracted dates, destinations, activities, and dress codes; flag missing details.
- **FR-6.3:** Generate outfits for confirmed activities using available owned clothes, noting any gaps.
- **FR-6.4:** Generate an editable packing list that consolidates repeated items and separates owned clothes from suggested purchases.
- **FR-6.5:** Save, edit, and delete private plans and associated uploaded files.

## Quality Requirements

- **NFR-1:** Support keyboard navigation, labels, visible focus, and readable contrast.
- **NFR-2:** Target usable main pages within three seconds on broadband, excluding AI processing.
- **NFR-3:** Show immediate progress and recoverable errors for AI requests.
- **NFR-4:** Enforce authentication, ownership, input validation, and server-only secrets.
- **NFR-5:** Delete associated files when clothing records or plans are deleted.
- **NFR-6:** Validate AI output against budget, size, ownership, and approval rules.
- **NFR-7:** Ground recommendation explanations in actual wardrobe data.
- **NFR-8:** Use typed domain modules and automated business-rule tests.
- **NFR-9:** Support phone, tablet, and desktop layouts.

Scope and exclusions are defined in the [vision](../../VISION.md).

