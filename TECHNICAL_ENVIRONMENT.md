# Technical Environment

Team choices: Claude Code for AI-DLC, a hosted AI API for the product, and a Python backend with a simple web UI. Specific frameworks and services below are proposed; pin dependency versions when implementation starts.

## Coding Agent

Use Claude Code to run AI-DLC. Its repository configuration belongs under `.claude/` when setup is performed. AI-DLC is not installed or configured yet. The product's hosted model is a separate choice from the coding agent.

## Stack

| Area | Proposed technology |
| --- | --- |
| Language | Python with type hints; SQL for database changes |
| Backend | FastAPI |
| Simple web UI | Jinja2 templates, HTML/CSS, and minimal JavaScript |
| Database and files | SQLite and private file storage for the local prototype; hosted storage to be selected |
| Authentication | Session-based sign-in with server-side ownership checks; library to be selected |
| Product AI | Hosted API supporting vision, structured output, and tool calling; provider/model to be selected |
| External tools | Weather and places APIs called through Python functions; providers to be selected |
| Validation | Pydantic for user input, tool arguments, and model output |
| Tests | pytest and Playwright |
| Hosting and CI | Hosting to be selected; GitHub Actions |

Use one application with separate modules for profiles, closet items, travel/event plans, outfits, analysis, and shopping. See [architecture](docs/inception/architecture.md).

## Data and AI

Store profiles, clothing items, outfits, feedback, travel/event plans, activities, packing lists, catalog products, recommendations, and cart items. User-owned records carry an owner ID. Activities link to outfits; packing entries distinguish owned items from suggested purchases.

AI classifies photos and suggests outfits or products. Server code checks ownership, size, budget, and approval state before accepting results. Recommendations reference real closet and catalog IDs. Failed analysis offers retry or manual entry.

The model can request weather or place lookups through defined tools. Python validates arguments, calls the APIs, and returns results to the model. Failed lookups allow manual context; unavailable forecasts are labeled rather than invented.

An itinerary parser extracts dates, destinations, activities, and dress codes for user confirmation. Confirmed plans guide outfit generation and packing lists that consolidate repeated items. Unsupported or unreadable files fall back to manual plan entry; supported formats and limits remain to be chosen.

## Security

- Keep API keys in server environment variables and out of Git.
- Use authenticated access and server-side ownership checks for every record and file.
- Validate uploaded image and itinerary types and sizes; serve private files through authenticated routes.
- Delete associated files when users delete clothing items or plans.
- Exclude secrets, images, and itinerary contents from logs. Treat uploaded text as data, not instructions.
- Use sample catalog data and simulated checkout; collect no payment details.

## Development and Testing

Use feature branches and teammate pull-request review. Before merging, run Python lint, type checks, tests, and an application startup check.

Test budget and size rules, data ownership, malformed AI output, weather/place tool calls, itinerary extraction, packing-list consolidation, and approval behavior. Use fixtures for models and external APIs in routine tests. Verify closet, daily outfit, travel/event planning, and shopping journeys, including keyboard navigation and phone layouts.

Local setup commands will be added with the Python application scaffold. Frameworks, hosting, authentication library, and AI/weather/places providers remain open choices.

