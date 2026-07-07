# Treasury Risk Intelligence Platform

An IRRBB (Interest Rate Risk in the Banking Book) treasury platform combining
a quantitative risk engine, Monte Carlo simulation, an AI copilot (RAG over
Basel/policy docs), and a full-stack dashboard.

## Folder structure

```
treasury-dashboard/
├── backend/
│   ├── app/
│   │   ├── main.py                  # FastAPI entrypoint
│   │   ├── core/config.py           # settings (.env)
│   │   ├── quant/                   # Module 1 — Interest Rate Risk Engine
│   │   │   ├── duration.py          #   Macaulay/Modified Duration, DV01
│   │   │   ├── convexity.py         #   Convexity, 2nd-order price approx
│   │   │   ├── yield_curve.py       #   Curve model + shift utilities
│   │   │   ├── gap_analysis.py      #   Repricing gap by bucket
│   │   │   ├── nii.py               #   Net Interest Income sensitivity
│   │   │   ├── eve.py               #   Economic Value of Equity
│   │   │   └── var.py               #   Parametric / historical VaR, ES
│   │   ├── scenarios/                # Module 2 — Scenario Simulator
│   │   │   ├── definitions.py       #   Named scenario library (Basel + desk)
│   │   │   └── scenario_engine.py   #   Applies scenarios, recomputes metrics
│   │   ├── montecarlo/              # Module 3 — Monte Carlo Engine
│   │   │   ├── rate_paths.py        #   Vasicek / CIR short-rate simulation
│   │   │   ├── portfolio_sim.py     #   Portfolio value & NII distributions
│   │   │   └── liquidity_sim.py     #   Liquidity shortfall simulation
│   │   ├── copilot/                  # Module 4 — AI Treasury Copilot
│   │   │   ├── document_loader.py   #   Chunk Basel/policy docs
│   │   │   ├── vector_store.py      #   Embeddings + similarity search
│   │   │   ├── rag_pipeline.py      #   Retrieval + grounded LLM call
│   │   │   └── agent.py             #   Tool-calling orchestration, reports
│   │   ├── recommendations/          # Module 6 — AI Recommendations
│   │   │   └── risk_detector.py     #   Threshold-based risk flags
│   │   ├── api/                      # Module 5 — FastAPI routes
│   │   │   ├── routes_portfolio.py
│   │   │   ├── routes_risk.py
│   │   │   ├── routes_scenarios.py
│   │   │   ├── routes_montecarlo.py
│   │   │   └── routes_copilot.py
│   │   ├── models/schemas.py        # Pydantic request/response models
│   │   └── db/                       # SQLAlchemy models (Postgres)
│   ├── tests/                        # pytest unit tests
│   ├── data/sample_portfolio.csv
│   ├── requirements.txt
│   ├── Dockerfile
│   └── .env.example
├── frontend/                         # Module 5 — React dashboard
│   ├── src/
│   │   ├── components/               # YieldCurveChart, DurationPanel, VaRPanel,
│   │   │                             # NIISensitivity, StressTestResults,
│   │   │                             # RiskHeatmap, CopilotChat
│   │   ├── pages/Dashboard.jsx
│   │   ├── api/client.js
│   │   └── styles/theme.css          # design tokens ("Vault Ledger" theme)
│   ├── package.json
│   ├── vite.config.js
│   └── Dockerfile
├── docs/                             # RAG source corpus (Basel/IRRBB, policy)
│   ├── irrbb-outlier-test-summary.md
│   └── treasury-risk-policy.md
└── docker-compose.yml
```

## Running locally

**Backend**
```bash
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env   # add your ANTHROPIC_API_KEY
uvicorn app.main:app --reload
```
Note: run quant/scenario/montecarlo modules with `python -m app.quant.duration`
(package-relative imports), not `python app/quant/duration.py` directly.

**Frontend**
```bash
cd frontend
npm install
npm run dev
```

**Or full stack via Docker**
```bash
docker compose up --build
```

**Tests**
```bash
cd backend && pytest -q
```
All 10 unit tests (duration, scenarios, Monte Carlo) pass against this scaffold.

## What's fully wired vs. stubbed

- **Fully implemented & tested**: all of `quant/`, `scenarios/`, `montecarlo/`,
  and `recommendations/` — pure Python, no external dependencies, verified
  against sample bond/portfolio data.
- **Functional but needs your API key**: `copilot/rag_pipeline.py` calls the
  real Anthropic API (`claude-sonnet-4-6`); the retrieval layer uses a
  dependency-free hashing embedder by default — swap in a real embeddings
  model or Postgres+pgvector for production relevance (see comment block at
  the bottom of `vector_store.py`).
- **Scaffolded, needs a real DB**: `db/` is empty — wire SQLAlchemy models
  here and point `routes_portfolio.py` at Postgres instead of the in-memory
  sample data once you have real position data to load.

## Why this project

Most student finance projects are stock/crypto prediction or portfolio
optimization. This one is IRRBB-specific: duration/convexity/EVE/NII are the
exact metrics referenced in Basel's Standardised Outlier Test, the scenario
library mirrors the six prescribed regulatory shocks, and the copilot's RAG
grounding means it explains *why* a metric moved using the same numbers the
dashboard displays — not a hallucinated narrative. That combination (quant
engine + AI copilot + full-stack delivery) is what maps directly to a
treasury/risk-adjacent engineering role.
