# Super-Speeder Policy Dashboard

A dashboard that flags New York drivers who would trigger mandatory Intelligent Speed Assistance (ISA, "speed limiter") installation under NY bill [A.2299 / S.4045](https://www.nysenate.gov/legislation/bills/2025/S4045/amendment/A), and surfaces drivers who are close to the line. It was built for the **DSSG-NYC Transportation Safety Hackathon** (Families for Safe Streets), where it took **1st place** (December 2025).

> **Branches:** the dashboard code is on the default branch, `quackhacks`. A UI-only earlier version is on `demo_ui`, and `main` holds the original hackathon starter material (task brief, notebook, point-value seeds).

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
| Speed-related license points | 11 or more | trailing 18 months |
| Warning band | within 2 tickets or 2 points below either threshold | same windows |

A driver who meets either threshold is a super speeder. Warning-band drivers are reported with how many tickets or points remain until the threshold.

These match the bill as amended (S4045C): 11 or more license points within 18 months, or 16 or more speed-camera tickets within 12 months. Windows are computed as months x 30 days and are configurable via `POINTS_WINDOW_MONTHS` and `CAMERA_TICKET_WINDOW_MONTHS`.

## Stack

- **DuckDB** is the warehouse and the query engine for all detection logic.
- **Polars** is a declared dependency and was used in the starter notebook for data access; the upload/cleaning module (`backend/src/cleaning.py`) uses **pandas**.
- **FastAPI + Uvicorn + Jinja2** serve the web app; pytest tests are in `backend/tests/`.
- Python 3.10+, managed with `uv`.

## How to run

```bash
uv sync                      # or: pip install -e .
uv run python backend/app.py # serves http://localhost:8000
```

Then open <http://localhost:8000> and upload CSVs. The sample CSVs and DuckDB file described in `docs/DATA.md` are not committed to the branch (upstream removed seed data), so you must supply your own speed-camera and violation CSVs. Tests: `uv run pytest backend/tests`.

Further docs: `docs/BACKEND.md`, `docs/FRONTEND.md`, `docs/DATA.md`, `docs/NOTEBOOKS.md`.

## Contributions

Abhiram Kandadi built the detection logic and policy threshold implementation. Shrikar Swami built the backend and dashboard UI. Commits are under Shrikar's account because we paired over VS Code Remote on a single machine during the event.

1st place, DSSG-NYC Transportation Safety Hackathon, December 2025.

## Upstream attribution

This repository is a fork of [ShrikarSwami/stop-super-speeders-hackathon-Shrikar](https://github.com/ShrikarSwami/stop-super-speeders-hackathon-Shrikar). Full upstream history is preserved unchanged. Only this README differs from upstream. The original README (event brief, task list, schedule) is available in the upstream repo and in this fork's history.
