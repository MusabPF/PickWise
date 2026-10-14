# PickWise
A universal comparison platform that helps users make smarter decisions by comparing products and listings across multiple categories, including mobiles, laptops and cars.

Compare mobiles, laptops, and cars based on **your** priorities, not just raw specs.

Pick 2–5 options, set what matters to you (price, battery, performance, fuel economy...), and get a personalized ranking with a plain-English explanation of why one option fits you best.

> No database. No accounts. Data is fetched from external sources at request time and discarded afterwards.

## Features

- Search products from external data providers
- Compare 2–5 options side by side
- Priority sliders per attribute (0–10)
- Weighted scoring with per-attribute breakdown
- Template-based explanations ("Phone A wins on battery, your top priority")
- Manual entry for specs/prices a provider doesn't return
- Graceful handling of missing data and API failures

## How it works

```
React (Vite + TS + Tailwind)
        |
        v
FastAPI backend  -->  External data providers
 - fetch + normalize
 - in-memory cache (TTL)
 - scoring engine
```

1. User searches a category. Backend queries the provider and maps results to a common `Product` format.
2. User selects 2–5 products and sets priorities.
3. Frontend sends the full products + weights to `POST /api/compare`.
4. Backend normalizes each attribute across the selected set (min-max, flipped for "lower is better"), applies weights, and returns ranking + explanation.

The backend is stateless. Nothing is stored between requests.

### Scoring notes

- Scores are **relative to the selected set**. A 100 means "best of these," not "objectively great."
- Missing values are **not treated as zero**. That attribute is skipped for that product, weights are renormalized, and the UI shows "data unavailable."

## Categories

| Category | Data source | Notes |
|----------|-------------|-------|
| Mobiles  | MobileAPI.dev | Specs only; price entered manually |
| Laptops  | TBD (verify provider coverage) | Manual entry fallback |
| Cars     | TBD (external car-specs API) | No local prices from most APIs; manual entry fallback |

Adding a category = adding one config file in `backend/app/categories/`. No scoring changes needed.

## Tech stack

- **Frontend:** React, TypeScript, Vite, Tailwind CSS
- **Backend:** Python, FastAPI, Pydantic, httpx
- **Testing:** pytest
- **Deploy:** Vercel (frontend), Render (backend)

## Project structure

```
pickwise/
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── models.py
│   │   ├── categories/     # attribute configs per category
│   │   ├── providers/      # external API clients
│   │   ├── services/       # cache, normalize, scoring, explain
│   │   └── routers/        # search, compare, categories
│   ├── tests/
│   ├── requirements.txt
│   └── .env.example
└── frontend/
    ├── src/
    │   ├── api/
    │   ├── components/
    │   ├── hooks/
    │   ├── lib/
    │   └── pages/
    ├── package.json
    └── .env.example
```

## Getting started

### Prerequisites

- Python 3.11+
- Node.js 18+
- A MobileAPI.dev API key

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env             # add your API key
uvicorn app.main:app --reload
```

API runs at `http://localhost:8000`. Docs at `/docs`.

### Frontend

```bash
cd frontend
npm install
cp .env.example .env             # set VITE_API_URL=http://localhost:8000
npm run dev
```

App runs at `http://localhost:5173`.

### Tests

```bash
cd backend
pytest
```

## Environment variables

**Backend (`backend/.env`)**

| Variable | Description |
|----------|-------------|
| `MOBILEAPI_KEY` | MobileAPI.dev API key (never exposed to the frontend) |
| `ALLOWED_ORIGINS` | Comma-separated CORS origins |
| `CACHE_TTL_SECONDS` | In-memory cache lifetime |

**Frontend (`frontend/.env`)**

| Variable | Description |
|----------|-------------|
| `VITE_API_URL` | Backend base URL |

## API

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Health check |
| GET | `/api/categories` | Categories and their attributes (drives the sliders) |
| GET | `/api/{category}/search?q=` | Search products |
| POST | `/api/compare` | Score and rank selected products |

## Roadmap

- [x] Project plan
- [ ] Backend skeleton, models, category configs
- [ ] Mobile provider + search
- [ ] Scoring engine + tests
- [ ] Priority sliders, results page, explanations
- [ ] Laptops and cars
- [ ] Error states, manual entry
- [ ] Deploy
- [ ] User testing

**Later (optional):** LLM-assisted spec extraction from pasted descriptions, flats category, shareable comparison links (stateless, encoded in the URL).

## Limitations

- Provider data may be incomplete or outdated. Always verify on the original listing.
- Specs from international APIs don't include local (e.g. Pakistani) retail prices, so users enter prices themselves.
- Free-tier hosting may cause a slow first request after inactivity.

## License

MIT
