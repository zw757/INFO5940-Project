# Technical Environment: ClosetCast

> **Change log**
> - **2026-11-19 (AI-DLC Requirements Analysis):** LLM provider changed from Google Gemini to **OpenAI** by team decision (requirement-verification Q18, clarification Q2–Q4; see `aidlc-docs/audit.md`). All Gemini / `google-genai` references below were replaced with OpenAI / `openai` equivalents.
> - **2026-11-23 (after Inception):** Planned OpenAI models named by team decision: `gpt-6-luna` for `OPENAI_MODEL_FAST` and `gpt-6.2-sol` for `OPENAI_MODEL_AGENT` (see `aidlc-docs/audit.md`). No requirement, story, design, or unit changed.

## Project Technical Summary

| Item | Decision |
| --- | --- |
| Project type | Greenfield, single-user web application |
| Deployment | **Runs locally only** on each team member's laptop (macOS, Windows or Linux). No cloud hosting. |
| How it runs | A FastAPI server started with `uv run closetcast` serves both the web UI (server-rendered HTML) and a JSON API at `http://127.0.0.1:8000`. It calls the OpenAI API (hosted LLM) for vision tagging and agent reasoning, and free public APIs for weather and places. Data is stored in a local SQLite file. |
| LLM provider | **OpenAI API** (paid; each team member uses their own API key with a monthly budget set in the OpenAI dashboard). Two models, set by environment variables so they can change without code changes: `OPENAI_MODEL_FAST` (small, low-cost model for vision tagging and itinerary/event parsing) and `OPENAI_MODEL_AGENT` (stronger model for the agent's tool-calling and reasoning). Both must support image input, function calling and JSON-schema structured output. Planned models (chosen 2026-11-23 from OpenAI's models page): `OPENAI_MODEL_FAST=gpt-6-luna` ($0.10 / $0.50 per 1M input/output tokens) and `OPENAI_MODEL_AGENT=gpt-6.2-sol` ($2 / $10 per 1M input/output tokens). |
| Timeline | Student team project; Construction runs Nov 24 – Dec 16, 2026 |

## Programming Languages

### Required

| Language | Version | Use |
| --- | --- | --- |
| Python | 3.12 or newer | All backend code, agent, tools, tests |
| HTML + CSS | HTML5 | Jinja2 templates for the UI |

### Permitted

| Language | Constraint |
| --- | --- |
| JavaScript (vanilla) | Small inline scripts only (e.g., image preview before upload). No build step, no npm. |
| SQL | Only through SQLModel; no hand-written SQL strings except in migrations-free schema creation |

### Prohibited

| Language | Reason | Use instead |
| --- | --- | --- |
| TypeScript / Node.js backend | Splits the team across two stacks; adds a build pipeline | Python + FastAPI |
| Any compiled frontend framework requiring a bundler | No time to maintain a build toolchain | Jinja2 templates + HTMX |

## Package Management and Tooling

- **uv** for all dependency management (`uv add`, `uv sync`, `uv run`). `pyproject.toml` holds all project and tool configuration; `uv.lock` is committed.
- **ruff** for linting and formatting (line length 100; rule sets `E, F, I, B, UP, ASYNC`).
- **mypy** in strict mode for `src/`.
- Source layout: `src/closetcast/` (application package) and `tests/` at the repository root. Application code never goes inside `aidlc-docs/`.

## Frameworks and Libraries

### Required

| Library | Purpose |
| --- | --- |
| FastAPI | Web framework for the JSON API and HTML pages |
| Uvicorn | ASGI server for local runs |
| Pydantic v2 + pydantic-settings | Request/response validation, LLM structured output schemas, configuration from `.env` |
| openai (`from openai import AsyncOpenAI`) | Official OpenAI Python SDK: vision tagging, structured output, function calling |
| httpx (async) | All outbound HTTP calls (weather, geocoding, places) |
| SQLModel (on SQLAlchemy 2.x) + SQLite | Local persistence for closet items, style profile, ratings, trips, plans |
| Jinja2 | Server-rendered HTML templates |
| HTMX (loaded from a pinned CDN URL or vendored file) | Partial page updates without a JavaScript framework |
| python-multipart | Photo and PDF uploads |
| Pillow | Validate, orient and downscale photos (max 1024 px long side) before sending to the LLM |
| pypdf | Extract text from uploaded itinerary PDFs |

### Preferred

| Library | Purpose |
| --- | --- |
| Pico CSS (classless) | Clean default styling with no custom CSS framework |
| tenacity | Retry with exponential backoff for LLM and weather calls (rate limits, transient errors) |
| structlog or standard `logging` | Structured logs; never log API keys or photo bytes |

### Prohibited

| Prohibited | Reason | Use instead |
| --- | --- | --- |
| LangChain, LlamaIndex, CrewAI, AutoGen | Heavy abstractions that hide the tool-calling loop, make debugging hard, and change APIs frequently | A small custom agent loop using OpenAI function calling (≤ 200 lines) |
| `google-genai`, `google-generativeai` (Gemini SDKs) | LLM provider changed to OpenAI (2026-11-19); a second provider would double prompt and schema testing | `openai` |
| Third-party OpenAI wrappers (e.g., LiteLLM) | Hide the request/response shapes the agent loop depends on | `openai` SDK directly |
| requests | Blocking; stalls the async event loop | httpx (async) |
| Flask, Django | Project uses FastAPI | FastAPI |
| React, Vue, Next.js, Svelte | Requires a JS build toolchain; not needed for the MVP UI | Jinja2 + HTMX |
| Streamlit, Gradio | Limits UI control and mixes UI with logic; hard to test | FastAPI + Jinja2 |
| BeautifulSoup / Selenium / Playwright for retailer scraping | Retailer scraping is out of scope (brittle, against terms of service) | Generic search links for "consider buying" suggestions |
| pandas, numpy | Not needed; data sizes are tiny | Standard library |
| Any vector database (Chroma, Pinecone, FAISS) | Closet has < 500 items; SQL filters are enough | SQLModel queries |
| pip, poetry, pipenv, conda | Project uses uv exclusively | uv |
| black, flake8, isort | Replaced by ruff | ruff |
| `eval()`, `exec()`, `pickle` on untrusted data | Arbitrary code execution risk | Pydantic parsing, JSON |

## Cloud Environment and External Services

**There is no cloud deployment.** The product runs locally (`uv run closetcast`) and binds to `127.0.0.1:8000` only. External services are called over HTTPS from the local server.

### Service Allow List

| Service | Purpose | Constraints |
| --- | --- | --- |
| OpenAI API (`api.openai.com`) | Vision tagging, itinerary parsing, dress-code reasoning, agent tool calling | Paid per request. Per-minute request and token limits depend on each account's usage tier: queue photo tagging with limited concurrency, retry with backoff on `429`, cache results. Model names from `OPENAI_MODEL_FAST` / `OPENAI_MODEL_AGENT`. |
| Open-Meteo Forecast API (`api.open-meteo.com/v1/forecast`) | Daily forecast up to 16 days ahead | No key. Free for non-commercial use. |
| Open-Meteo Historical Weather API (`archive-api.open-meteo.com/v1/archive`) | Typical weather for trip dates beyond the forecast horizon (average of the same dates over the past 5 years) | No key. Cache results in SQLite. |
| Open-Meteo Geocoding API (`geocoding-api.open-meteo.com/v1/search`) | City name → latitude/longitude | No key. |
| OpenStreetMap Nominatim (`nominatim.openstreetmap.org`) | Place name → place type (e.g., place_of_worship, restaurant, park) for dress-norm lookup | Max 1 request/second; must send an identifying `User-Agent`; cache all results. |

### Service Disallow List

| Service | Reason | Use instead |
| --- | --- | --- |
| Any cloud hosting (AWS, GCP, Azure, Vercel, Heroku) | Not required; product runs locally | Local Uvicorn server |
| Google Places API, Google Maps Platform | Requires billing account | Nominatim |
| Paid weather APIs (OpenWeatherMap paid tiers, etc.) | Cost; Open-Meteo covers needs | Open-Meteo |
| Retailer or shopping APIs, price scrapers | Out of MVP scope | Generic search links |
| Image generation APIs | Virtual try-on is out of scope | Show original item photos |

## Architecture and Patterns

### Architecture

A layered modular monolith in one Python package:

```text
src/closetcast/
  main.py            # FastAPI app factory, startup wiring, `closetcast` CLI entry point
  config.py          # pydantic-settings Settings (reads .env)
  api/routes/        # FastAPI routers: closet, profile, outfits, occasions, trips (thin: validate, call service, map errors)
  web/templates/     # Jinja2 templates; web/static/ for CSS and HTMX
  agent/             # Agent loop, system prompts, tool registry, service classes per use case
  tools/             # Functions the agent can call: weather, geocoding, places, dress codes, closet search, packing
  domain/            # Pydantic models, errors, pure business logic (outfit validation, packing optimizer)
  db/                # SQLModel tables, session handling, repositories
  knowledge/         # Curated dress-code guide (JSON) for event and venue types
tests/
  unit/  integration/
data/                # Local SQLite DB and uploaded photos (gitignored)
```

### Agent design rules

- **Custom tool-calling loop** using OpenAI function calling: the model receives tool declarations, chooses tools, the app executes them and returns results, repeating until the model produces a final answer. Hard cap of **8 tool calls per request**; on hitting the cap the agent returns its best partial answer with a note.
- **Expected tools** (final set decided in Application Design): `get_weather(city, date)` (forecast, or historical average when beyond 16 days), `geocode(city)`, `lookup_place(name, city)`, `get_dress_code_guidance(event_or_venue_type)`, `search_closet(filters)`, `get_style_profile()`, `optimize_packing(trip_plan, luggage_size, rewear_rules)`, `build_search_link(item_description)`.
- **The LLM references clothes only by closet item ID.** Every outfit the LLM proposes passes through `validate_outfit()` (deterministic code) before it reaches the user; unknown IDs are rejected and the agent is asked to retry once.
- **Counting and optimization are done in code, not by the LLM.** The packing optimizer is a deterministic Python function (greedy selection with constraint checks) with unit tests.
- **Structured output everywhere:** every LLM response the app consumes is parsed into a Pydantic model via OpenAI's JSON-schema structured output. Free-text responses are only used for explanations shown to the user.
- **The LLM client sits behind an interface** (`Protocol`) so all tests run with a fake client and no API key.

### API design

- JSON endpoints under `/api/...`; HTML pages at `/`, `/closet`, `/today`, `/occasion`, `/trips/{id}`.
- Request and response bodies are Pydantic models; FastAPI generates OpenAPI docs at `/docs`.
- Errors return `{"detail": {"code": "<STABLE_CODE>", "message": "<human readable>"}}` with 4xx for user errors and 502 when an upstream service (LLM, weather) fails.
- Long-running operations (trip planning, batch photo tagging) run as FastAPI background tasks; the UI polls a status endpoint via HTMX every 2 seconds.

### Data

- SQLite file at `data/closetcast.db`; photos stored at `data/images/<uuid>.jpg` (downscaled copy) and referenced by path.
- Tables created with `SQLModel.metadata.create_all` at startup (no migration tool for the MVP).
- Weather, geocoding and place lookups cached in SQLite with a timestamp (forecast cache: 3 hours; historical, geocoding and places: 30 days).
- Temperatures stored in Celsius; converted for display based on the user's unit preference.

### Frontend

- Server-rendered Jinja2 pages, HTMX for partial updates (upload progress, tag editing, plan refresh), Pico CSS for styling.
- Outfit cards show the items' photos side by side with the explanation text.
- Must work in current Chrome, Firefox and Safari on a laptop screen; mobile layout is not required.

## Security Requirements

### Authentication and authorization

- No user accounts in the MVP (single local user). The server **binds to 127.0.0.1 only** so it is not reachable from other machines on the network.

### Secrets management

- The OpenAI API key is read from `OPENAI_API_KEY` in a local `.env` file loaded by pydantic-settings. `.env` is in `.gitignore`; a committed `.env.example` lists variable names with empty values.
- **No API keys or secrets are ever committed** to the repository, written to logs, or included in error messages.

### Input validation

- All request bodies validated by Pydantic with explicit length limits (free-text fields ≤ 2,000 characters; itinerary text ≤ 20,000 characters).
- Uploads: photos must be JPEG, PNG or WebP verified by Pillow (not just file extension), ≤ 10 MB, re-encoded and downscaled before storage; PDFs ≤ 5 MB and ≤ 20 pages. Stored files get server-generated UUID names, never user-supplied names.
- **Prompt injection:** itinerary and event text from the user is passed to the LLM as clearly delimited data, never concatenated into system instructions. Tool arguments chosen by the LLM are validated by Pydantic before execution; tools are read-only except `save_plan`.

### Data protection and privacy

- All personal data (closet photos, profile, trips) stays on the local machine except the content sent to the OpenAI API for processing. The UI shows a one-time notice explaining that clothing photos and itinerary text are sent to OpenAI's API.
- Photos are stripped of EXIF metadata (including GPS location) before storage or upload.

### Dependency security

- Dependencies pinned in `uv.lock`; run `uv run pip-audit` before each milestone submission.

### Security compliance framework

- Follow the **OWASP Top 10 (2021)** categories that apply to a local web app: A03 Injection (Pydantic validation, no `eval`, ORM-only SQL), A05 Security Misconfiguration (localhost binding, debug off by default), A06 Vulnerable Components (pip-audit), A08 Software and Data Integrity (lockfile), and the **OWASP Top 10 for LLM Applications**: LLM01 Prompt Injection (data/instruction separation, validated tool args) and LLM05 Improper Output Handling (structured output + validation; LLM text is HTML-escaped by Jinja2 autoescaping).

## Testing Requirements

| Test type | Tooling | Scope and rules |
| --- | --- | --- |
| Unit | pytest, pytest-asyncio (`asyncio_mode = "auto"`) | Domain logic (outfit validation, packing optimizer, unit conversion), tools with mocked HTTP, prompt builders. **No real network or LLM calls.** |
| HTTP mocking | respx | Mock Open-Meteo and Nominatim responses |
| LLM mocking | Fake client implementing the LLM `Protocol` / `unittest.mock.AsyncMock` | Agent loop tested with scripted tool-call sequences |
| Integration | FastAPI `TestClient` with dependency overrides | Each endpoint: happy path, validation error, upstream failure |
| Live smoke tests | pytest marker `@pytest.mark.live`, skipped unless `RUN_LIVE_TESTS=1` | A few end-to-end calls to the real OpenAI and Open-Meteo APIs, run manually before the demo |
| Coverage | pytest-cov | ≥ 80% line coverage for `domain/` and `tools/`; ≥ 60% overall |
| Static checks | ruff, mypy --strict | Must pass before merging to `main` |

Quality gate before every merge to `main`: `uv run ruff check . && uv run ruff format --check . && uv run mypy src && uv run pytest`.

## Example Code Patterns

These examples are the canonical patterns for code generation. (They are kept inside this document, not as `.py` files in the repository, so the workspace stays empty until Construction.)

### Example: a typical endpoint

Thin route: validate input with Pydantic, call a service through an injected dependency, map domain errors to HTTP errors.

```python
# src/closetcast/api/routes/outfits.py
from typing import Annotated

from fastapi import APIRouter, Depends, HTTPException, status
from pydantic import BaseModel, Field

from closetcast.agent.service import OutfitAgent, get_outfit_agent
from closetcast.domain.errors import ClosetCastError
from closetcast.domain.models import OutfitRecommendation

router = APIRouter(prefix="/api/outfits", tags=["outfits"])


class DailyOutfitRequest(BaseModel):
    city: str = Field(min_length=1, max_length=100)
    plans: str | None = Field(default=None, max_length=1000)


@router.post("/daily", response_model=OutfitRecommendation)
async def recommend_daily_outfit(
    request: DailyOutfitRequest,
    agent: Annotated[OutfitAgent, Depends(get_outfit_agent)],
) -> OutfitRecommendation:
    try:
        return await agent.recommend_daily(city=request.city, plans=request.plans)
    except ClosetCastError as exc:
        raise HTTPException(
            status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
            detail={"code": exc.code, "message": exc.message},
        ) from exc
```

Supporting models, errors and the agent interface used above:

```python
# src/closetcast/domain/models.py
from datetime import date

from pydantic import BaseModel, Field


class DailyWeather(BaseModel):
    day: date
    temp_max_c: float
    temp_min_c: float
    precipitation_probability: int | None = Field(default=None, ge=0, le=100)
    source: str = Field(pattern="^(forecast|historical_average)$")


class Outfit(BaseModel):
    item_ids: list[int] = Field(min_length=1)
    explanation: str


class OutfitRecommendation(BaseModel):
    primary: Outfit
    alternative: Outfit | None = None
    weather: DailyWeather


# src/closetcast/domain/errors.py
class ClosetCastError(Exception):
    """Base error. Every error carries a stable machine-readable code."""

    def __init__(self, code: str, message: str) -> None:
        super().__init__(message)
        self.code = code
        self.message = message


class WeatherUnavailableError(ClosetCastError):
    pass


class UnknownClosetItemError(ClosetCastError):
    pass


# src/closetcast/agent/service.py
from typing import Protocol

from closetcast.domain.models import OutfitRecommendation


class OutfitAgent(Protocol):
    """Interface the API layer depends on. The real implementation runs the
    LLM tool-calling loop; tests substitute a fake that needs no API key."""

    async def recommend_daily(self, city: str, plans: str | None) -> OutfitRecommendation: ...


def get_outfit_agent() -> OutfitAgent:
    raise NotImplementedError("Wired up in closetcast.main at startup")
```

### Example: a typical function (deterministic guardrail)

Pure business logic with a stable error code; no I/O.

```python
# src/closetcast/domain/outfits.py
from collections.abc import Iterable

from closetcast.domain.errors import UnknownClosetItemError
from closetcast.domain.models import Outfit


def validate_outfit(outfit: Outfit, closet_item_ids: Iterable[int]) -> Outfit:
    """Guardrail run on every LLM-proposed outfit before it reaches the user.

    The LLM may only reference items by closet ID. Any ID not in the user's
    closet is rejected, so the product never recommends clothes the user doesn't own.
    """
    owned = set(closet_item_ids)
    unknown = [item_id for item_id in outfit.item_ids if item_id not in owned]
    if unknown:
        raise UnknownClosetItemError(
            code="UNKNOWN_CLOSET_ITEM",
            message=f"Outfit references items not in the closet: {unknown}",
        )
    if len(set(outfit.item_ids)) != len(outfit.item_ids):
        raise UnknownClosetItemError(
            code="DUPLICATE_CLOSET_ITEM", message="Outfit lists the same item twice"
        )
    return outfit
```

### Example: a typical agent tool (external API call)

Async httpx call with a timeout, upstream failures converted to a domain error, result returned as a Pydantic model. `today` is injectable so tests are deterministic.

```python
# src/closetcast/tools/weather.py
from datetime import UTC, date, datetime, timedelta

import httpx

from closetcast.domain.errors import WeatherUnavailableError
from closetcast.domain.models import DailyWeather

FORECAST_URL = "https://api.open-meteo.com/v1/forecast"
FORECAST_HORIZON_DAYS = 16


async def get_daily_forecast(
    client: httpx.AsyncClient,
    latitude: float,
    longitude: float,
    day: date,
    today: date | None = None,
) -> DailyWeather:
    """Agent tool: return the forecast for one day at one location.

    Raises WeatherUnavailableError if the day is outside the forecast horizon
    or the weather service fails. The agent then falls back to historical averages.
    """
    today = today or datetime.now(tz=UTC).date()
    if not (today <= day <= today + timedelta(days=FORECAST_HORIZON_DAYS - 1)):
        raise WeatherUnavailableError(
            code="OUTSIDE_FORECAST_HORIZON",
            message=f"{day.isoformat()} is outside the {FORECAST_HORIZON_DAYS}-day forecast window",
        )
    params: dict[str, str | float] = {
        "latitude": latitude,
        "longitude": longitude,
        "daily": "temperature_2m_max,temperature_2m_min,precipitation_probability_max",
        "timezone": "auto",
        "start_date": day.isoformat(),
        "end_date": day.isoformat(),
    }
    try:
        response = await client.get(FORECAST_URL, params=params, timeout=10.0)
        response.raise_for_status()
    except httpx.HTTPError as exc:
        raise WeatherUnavailableError(
            code="WEATHER_SERVICE_ERROR", message="Weather service request failed"
        ) from exc

    daily = response.json()["daily"]
    return DailyWeather(
        day=day,
        temp_max_c=daily["temperature_2m_max"][0],
        temp_min_c=daily["temperature_2m_min"][0],
        precipitation_probability=daily["precipitation_probability_max"][0],
        source="forecast",
    )
```

### Example: a typical LLM call (vision + structured output)

Every LLM call that feeds program logic uses a Pydantic schema as the response schema and validates the result.

```python
# src/closetcast/tools/tagging.py
import os
from typing import Literal

import base64

from openai import AsyncOpenAI
from pydantic import BaseModel, Field

from closetcast.domain.errors import ClosetCastError

TAGGING_PROMPT = (
    "You are tagging one clothing item for a wardrobe app. Describe only the garment "
    "in the photo. If the photo does not show a clothing item, set category to 'not_clothing'."
)


class ClothingTags(BaseModel):
    category: Literal["top", "bottom", "dress", "outerwear", "shoes", "accessory", "not_clothing"]
    subcategory: str = Field(description="e.g. t-shirt, jeans, blazer, sneakers")
    primary_color: str
    pattern: str = Field(description="solid, striped, plaid, floral, graphic, other")
    formality: int = Field(ge=1, le=5, description="1 = loungewear, 5 = black tie")
    warmth: int = Field(ge=1, le=5, description="1 = very light, 5 = heavy winter")
    rain_suitable: bool


async def tag_clothing_photo(
    client: AsyncOpenAI, image_bytes: bytes, mime_type: str
) -> ClothingTags:
    """Vision LLM call with structured output: the model must return JSON matching
    ClothingTags, which the SDK parses and Pydantic validates before anything is saved."""
    data_url = f"data:{mime_type};base64,{base64.b64encode(image_bytes).decode()}"
    response = await client.responses.parse(
        model=os.environ["OPENAI_MODEL_FAST"],
        input=[
            {
                "role": "user",
                "content": [
                    {"type": "input_image", "image_url": data_url},
                    {"type": "input_text", "text": TAGGING_PROMPT},
                ],
            }
        ],
        text_format=ClothingTags,
        temperature=0.2,
    )
    if response.output_parsed is None:
        raise ClosetCastError(code="TAGGING_FAILED", message="Model returned no tags")
    return response.output_parsed
```

*Note: confirm this call shape (Responses API `parse` with `text_format`) against the current `openai` Python SDK docs at the start of Construction, including which JSON-schema keywords (e.g., `minimum`/`maximum` from `ge`/`le`) strict structured output accepts.*

### Example: typical tests

Unit test for pure logic, a tool test with mocked HTTP (respx), an LLM test with a fake client, and an endpoint test with an injected fake agent. None of them use the network or an API key.

```python
# tests/unit/test_examples.py
from datetime import date, timedelta
from unittest.mock import AsyncMock, MagicMock

import httpx
import pytest
import respx
from fastapi import FastAPI
from fastapi.testclient import TestClient

from closetcast.agent.service import get_outfit_agent
from closetcast.api.routes.outfits import router
from closetcast.domain.errors import UnknownClosetItemError, WeatherUnavailableError
from closetcast.domain.models import DailyWeather, Outfit, OutfitRecommendation
from closetcast.domain.outfits import validate_outfit
from closetcast.tools.tagging import ClothingTags, tag_clothing_photo
from closetcast.tools.weather import FORECAST_URL, get_daily_forecast

TODAY = date(2026, 12, 1)  # fixed date keeps tests deterministic


def test_validate_outfit_rejects_items_not_in_closet() -> None:
    outfit = Outfit(item_ids=[1, 99], explanation="test")
    with pytest.raises(UnknownClosetItemError) as exc_info:
        validate_outfit(outfit, closet_item_ids=[1, 2, 3])
    assert exc_info.value.code == "UNKNOWN_CLOSET_ITEM"


@respx.mock
async def test_get_daily_forecast_parses_open_meteo_response() -> None:
    tomorrow = TODAY + timedelta(days=1)
    respx.get(FORECAST_URL).mock(
        return_value=httpx.Response(
            200,
            json={
                "daily": {
                    "temperature_2m_max": [18.5],
                    "temperature_2m_min": [9.0],
                    "precipitation_probability_max": [70],
                }
            },
        )
    )
    async with httpx.AsyncClient() as client:
        weather = await get_daily_forecast(client, 42.44, -76.50, tomorrow, today=TODAY)
    assert weather.temp_max_c == 18.5
    assert weather.precipitation_probability == 70
    assert weather.source == "forecast"


async def test_get_daily_forecast_rejects_dates_beyond_horizon() -> None:
    far_future = TODAY + timedelta(days=40)
    async with httpx.AsyncClient() as client:
        with pytest.raises(WeatherUnavailableError):
            await get_daily_forecast(client, 42.44, -76.50, far_future, today=TODAY)


async def test_tag_clothing_photo_parses_structured_output() -> None:
    parsed = ClothingTags.model_validate_json(
        '{"category": "outerwear", "subcategory": "rain jacket", "primary_color": "navy",'
        ' "pattern": "solid", "formality": 2, "warmth": 2, "rain_suitable": true}'
    )
    fake_client = MagicMock()
    fake_client.responses.parse = AsyncMock(return_value=MagicMock(output_parsed=parsed))

    tags = await tag_clothing_photo(fake_client, b"fake-jpeg-bytes", "image/jpeg")

    assert isinstance(tags, ClothingTags)
    assert tags.category == "outerwear"
    assert tags.rain_suitable is True


class FakeOutfitAgent:
    async def recommend_daily(self, city: str, plans: str | None) -> OutfitRecommendation:
        return OutfitRecommendation(
            primary=Outfit(item_ids=[3, 7, 12], explanation=f"Layers for a cool day in {city}"),
            weather=DailyWeather(day=TODAY, temp_max_c=12, temp_min_c=4, source="forecast"),
        )


def test_daily_outfit_endpoint_returns_recommendation() -> None:
    app = FastAPI()
    app.include_router(router)
    app.dependency_overrides[get_outfit_agent] = FakeOutfitAgent
    client = TestClient(app)

    response = client.post("/api/outfits/daily", json={"city": "Ithaca", "plans": "class"})

    assert response.status_code == 200
    assert response.json()["primary"]["item_ids"] == [3, 7, 12]


def test_daily_outfit_endpoint_validates_input() -> None:
    app = FastAPI()
    app.include_router(router)
    app.dependency_overrides[get_outfit_agent] = FakeOutfitAgent
    response = TestClient(app).post("/api/outfits/daily", json={"city": ""})
    assert response.status_code == 422
```
