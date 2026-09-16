# Equity–Efficiency Trade-offs in Blood Supply Allocation

A stochastic optimization framework for allocating a perishable, scarce resource (blood units) across a hospital network under demand uncertainty, motivated by the American Red Cross's July 2026 national blood shortage declaration.

## Problem

When blood supply falls short of demand, someone has to decide which hospitals receive priority. This project formulates that decision as a two-stage stochastic program: allocation decisions are made before demand is known, and unmet demand (shortage) is penalized according to each hospital's clinical criticality.

## Key Findings

- **VSS (Value of Stochastic Solution) = 10.8** — allocating based on average expected demand instead of explicitly modeling demand scenarios results in 24% worse outcomes.
- **EVPI (Expected Value of Perfect Information) = 1.4** — even perfect knowledge of which scenario will occur would only improve outcomes by about 4%, suggesting that better decision-modeling matters far more than better forecasting.
- **Feasibility threshold at 70% equity** — guaranteeing every hospital at least 70% of its baseline demand is the mathematical limit given current supply; beyond that, no allocation policy can satisfy the constraint.

## Repository Structure

- `notebook/` — Pyomo/Gurobi model (Jupyter notebook + data files)
- `dashboard/` — Interactive HTML dashboard (open dashboard/index.html in any browser)
- `figures/` — Static output charts (Pareto frontier)
- `docs/` — Data sources and modeling assumptions

## Methodology

Two-stage stochastic programming, solved with Pyomo and Gurobi. Six hospitals (1 Level I, 2 Level II, 3 Level III, scaled from national ACS trauma center ratios) under three demand scenarios (normal day, elevated activity, mass casualty event). Full parameter derivations and sources are documented in docs/data_sources_and_assumptions.md.

## Running the Model

1. Open notebook/model.ipynb in Google Colab or Jupyter.
2. Install dependencies: pip install -r requirements.txt
3. Run all cells in order.

## Interactive Dashboard

Open dashboard/index.html in any browser. Drag the equity slider to see how the allocation and expected shortage change in real time.

## Tools

Python, Pyomo, Gurobi, pandas, NumPy, Matplotlib, Chart.js

## Data Sources

See docs/data_sources_and_assumptions.md for the full list of sources (American Red Cross, American College of Surgeons, peer-reviewed literature on trauma transfusion rates).