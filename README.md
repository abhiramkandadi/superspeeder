# Super-Speeder Policy Dashboard

A dashboard that flags New York drivers who would trigger mandatory Intelligent Speed Assistance (ISA, "speed limiter") installation under NY bill [A.2299 / S.4045](https://www.nysenate.gov/legislation/bills/2025/S4045/amendment/A), and surfaces drivers who are close to the line. It was built for the **DSSG-NYC Transportation Safety Hackathon** (Families for Safe Streets), where it took **1st place** (December 2025).

> **Where the code is:** the dashboard lives on the [`quackhacks`](../../tree/quackhacks) branch (a UI-only earlier version is on `demo_ui`). `main` holds the original hackathon starter material (task brief, notebook, point-value seeds).

## What the dashboard does

1. **Upload** speed-camera and traffic-violation CSVs through a web page.
2. **Clean and load** them: columns are normalized, bad rows dropped, and the result is loaded into a DuckDB warehouse (`fct_violations`, `dim_driver`, and related tables; schema in `backend/sql/01_schema.sql`).
3. **Detect** super speeders and warning-band drivers with SQL over the warehouse.
4. **Display** results in a tabbed UI: a results table, an Analytics and Summary tab, per-driver detail pages (`/driver/{id}`), and a resources page for policy staff.

The intended audience is legislative and DMV policy staff, not data engineers.

## Thresholds implemented

From `backend/src/super_speeder_detector.py`:

| Rule | Threshold | Window |
|---|---|---|
| Speed-camera tickets | 16 or more | trailing 12 months |
| Speed-related license points | 11 or more | trailing 18 months (see note) |
| Warning band | within 2 tickets or 2 points below either threshold | same windows |

A driver who meets either threshold is a super speeder. Warning-band drivers are reported with how many tickets or points remain until the threshold.

**Note on the points window:** the bill text, as summarized in the hackathon brief, describes 11 points within 24 months. The code uses 18 months, so treat the points rule here as a configurable approximation (`POINTS_WINDOW_MONTHS`) rather than a verbatim implementation of the bill. Windows are computed as months x 30 days.

## Stack

- **DuckDB** is the warehouse and the query engine for all detection logic.
- **Polars** is a declared dependency and was used in the starter notebook for data access; the upload/cleaning module (`backend/src/cleaning.py`) uses **pandas**.
- **FastAPI + Uvicorn + Jinja2** serve the web app; pytest tests are in `backend/tests/`.
- Python 3.10+, managed with `uv`.

## How to run

```bash
git checkout quackhacks
uv sync                      # or: pip install -e .
uv run python backend/app.py # serves http://localhost:8000
```

Then open <http://localhost:8000> and upload CSVs. The sample CSVs and DuckDB file described in `docs/DATA.md` are not committed to the branch (upstream removed seed data), so you must supply your own speed-camera and violation CSVs. Tests: `uv run pytest backend/tests`.

Further docs on the `quackhacks` branch: `docs/BACKEND.md`, `docs/FRONTEND.md`, `docs/DATA.md`, `docs/NOTEBOOKS.md`.

## Contributions

Built jointly by Abhiram Kandadi and Shrikar Swami. Commits are under Shrikar's account because we paired over VS Code Remote on a single machine during the event.

The upstream commit history also credits other hackathon teammates as co-authors (GitHub handles: Akash Lal, Dman320, SrinidhiPalani, RaghavNanavati, Daksh Aggarwal), and DSSG-NYC organizers wrote the original brief and starter material.

## Upstream attribution

This repository is a fork of [ShrikarSwami/stop-super-speeders-hackathon-Shrikar](https://github.com/ShrikarSwami/stop-super-speeders-hackathon-Shrikar). Full upstream history is preserved unchanged. Only this README differs from upstream. The original README (event brief, task list, schedule) is available in the upstream repo and in this fork's history.
