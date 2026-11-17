# AI Closet & Personal Shopping Agent

An INFO 5940 project that helps users organize their wardrobe and decide what to wear, pack, or buy for everyday life, travel, and special occasions. The AI considers the user's existing clothes, personal style, weather, activities, and budget to create practical outfit recommendations.

Users can ask for a daily outfit, upload a travel itinerary, or share details about an upcoming event, such as a wedding, interview, or formal dinner. The assistant can suggest outfits for each activity, build a packing list using versatile pieces, and identify anything worth buying to complete the look. Recommendations start with what the user already owns and explain how each suggested addition meets their plans and fits their wardrobe.

## MVP

- Upload clothing photos and review AI-generated categories.
- Get outfits based on style, weather, and occasion.
- Upload an itinerary or describe an event to plan outfits and a packing list.
- Save outfits, swap pieces, and give feedback.
- Identify wardrobe gaps and recommend purchases for everyday needs, trips, or events.
- Build a monthly cart within a budget and approve simulated checkout.

The key idea is **outfit-first shopping**: explain how a purchase works with the user's closet, including when buying nothing is the best choice. The MVP uses a curated catalog; real payments and live retailer integrations are out of scope.

## Documentation

- [Vision](VISION.md) — purpose, scope, and success criteria
- [Technical Environment](TECHNICAL_ENVIRONMENT.md) — proposed stack and development practices
- [Inception](docs/inception/README.md) — requirements, stories, architecture, and delivery plan

Status: planning drafts prepared; team review pending. Implementation has not started.

Technical direction: Python backend, simple web UI, and a hosted AI API with vision and tool calling for weather/places. Claude Code is the chosen AI-DLC coding agent; setup is pending.

## Team

- May Wu — `zw757`
- Katherine Yang — `ky453`
- Emma Dai — `cd663`
