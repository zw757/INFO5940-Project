# User Stories

All stories are in the MVP. Requirement IDs link to [requirements](requirements.md); build dependencies are in the [delivery plan](delivery-plan.md).

## US-1.1 — Set Preferences

As a user, I want to save my style, sizes, and budget so suggestions fit my needs.

Acceptance: preferences are editable and private; invalid or negative budgets are rejected.  
Requirements: FR-1.1–FR-1.2.

## US-2.1 — Add Clothing

As a user, I want photo-based categorization to reduce closet-entry effort.

Acceptance: upload shows progress; proposed attributes can be confirmed or corrected; unknown attributes are labeled; failed analysis offers retry or manual entry.  
Requirements: FR-2.1–FR-2.3.

## US-2.2 — Manage My Closet

As a user, I want to browse, filter, edit, and delete items so my closet stays accurate.

Acceptance: all attributes are editable; items can be marked unavailable; confirmed deletion removes the image; other users cannot access the item.  
Requirements: FR-2.4–FR-2.5, FR-3.5.

## US-3.1 — Get an Outfit

As a user, I want an outfit for my weather, activities, and dress code to make dressing for daily life or a special occasion easier.

Acceptance: the request captures and displays context; weather/place lookups have a manual fallback; unavailable forecasts are labeled; results use available owned items; insufficient clothing produces an explanation; outfits can be saved.  
Requirements: FR-1.3–FR-1.4, FR-3.1–FR-3.2, FR-3.4–FR-3.5.

## US-3.2 — Refine an Outfit

As a user, I want to swap pieces and rate outfits to improve suggestions.

Acceptance: swaps preserve category compatibility and exclude the current item; likes and dislikes are saved for later recommendations.  
Requirements: FR-3.3–FR-3.4.

## US-4.1 — Understand a Gap

As a user, I want evidence that something is missing before shopping.

Acceptance: gaps reference actual closet items and any confirmed travel/event needs; the user can dismiss a gap; reuse or buying nothing is a valid result.  
Requirements: FR-4.1–FR-4.3.

## US-4.2 — Evaluate a Product

As a user, I want price and outfit examples to judge a suggested purchase.

Acceptance: products come from the labeled sample catalog, meet size and budget constraints, and explain which outfit and relevant travel/event need they support.  
Requirements: FR-5.1–FR-5.3, FR-5.7.

## US-4.3 — Control My Cart

As a user, I want to edit and approve the cart before checkout.

Acceptance: adding or removing products updates total and remaining budget; suggestions can be skipped; over-budget approval is blocked; simulated checkout requires explicit confirmation.  
Requirements: FR-5.4–FR-5.7.

## US-5.1 — Share My Plans

As a user, I want to upload an itinerary or enter event details so recommendations match my plans.

Acceptance: extracted dates, destinations, activities, and dress codes are editable before confirmation; missing details are flagged; unreadable uploads offer manual entry; saved plans and files are private and deletable.  
Requirements: FR-6.1–FR-6.2, FR-6.5.

## US-5.2 — Plan What to Pack or Buy

As a user, I want outfits for each activity and a packing list so I know what to bring and what is missing.

Acceptance: each confirmed activity has an outfit or a stated gap; versatile pieces can be reused; the editable packing list consolidates repeated items and distinguishes owned clothes from suggested purchases; shopping suggestions follow existing budget and size rules.  
Requirements: FR-6.3–FR-6.4, FR-4.1–FR-4.3, FR-5.2–FR-5.3.

