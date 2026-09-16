# Data Sources and Modeling Assumptions

This document describes the data, parameters, and assumptions underlying the blood allocation model, and explains how each value was obtained or derived.

## Background

In July 2026, the American Red Cross declared its second national blood shortage emergency, following a decline in donations that left the national supply of type O blood at less than a one-day reserve. The Red Cross distributes roughly 40% of the nation's blood supply to more than 2,500 hospitals, and the shortage has already led some hospitals to postpone elective surgeries. This project uses that episode as a case study for a more general problem: how should a limited, perishable supply of blood be allocated across hospitals with different clinical roles when demand is uncertain and supply falls short?

The model and its parameters are meant to be illustrative of the underlying allocation problem rather than a literal reconstruction of Red Cross operations, which are not publicly disclosed at that level of detail. Where possible, parameters are grounded in published statistics; where they are not, the derivation is stated explicitly below.

## Hospital network

The model uses a six-hospital network intended to represent, at small scale, the composition of a real regional trauma system. The American College of Surgeons reports roughly 213 Level I, 313 Level II, and 470 Level III trauma centers nationally. Scaling that ratio down to six hospitals gives one Level I, two Level II, and three Level III centers, which is the network used here.

| Hospital | Trauma level | Criticality weight | Base daily demand (units) |
|---|---|---|---|
| H1 | I | 3 | 15 |
| H2 | II | 2 | 8 |
| H3 | II | 2 | 8 |
| H4 | III | 1 | 3 |
| H5 | III | 1 | 3 |
| H6 | III | 1 | 3 |

The criticality weights reflect the roles the American College of Surgeons assigns to each level: Level I centers carry system leadership and comprehensive trauma responsibility and have no equivalent facility to refer patients to; Level II centers provide definitive care across most injury types; Level III centers serve a more limited role and have formal protocols for transferring patients whose needs exceed their resources. Weighting Level I three times higher than Level III, with Level II in between, reflects this asymmetry in what a shortfall actually costs.

Base daily demand for Level I is calibrated from the ACS designation criterion of at least 1,200 trauma admissions per year (roughly 3-4 patients per day), combined with published transfusion rates in which about 25% of admitted trauma patients receive a red blood cell transfusion. Level II and Level III demand figures scale down proportionally with typical patient volume at those levels. These are engineering estimates, not published per-hospital figures — hospitals do not report daily blood consumption publicly — and they are the parameters most worth revisiting in a sensitivity analysis.

## Demand scenarios

Three scenarios represent the range of daily trauma volume a hospital might see, motivated by the fact that a small share of trauma patients account for a disproportionate share of blood use: roughly 3% of trauma patients require massive transfusion (10 or more units within 24 hours), and this group alone can consume 70% of a center's blood supply. A single severe motor vehicle collision patient can require up to 50 units.

| Scenario | Description | Demand multiplier | Probability |
|---|---|---|---|
| S1 | Normal day | 1.0x | 0.85 |
| S2 | Elevated activity (weekend/holiday) | 1.5x | 0.10 |
| S3 | Mass casualty event | 4.5x | 0.05 |

The probabilities are a judgment call rather than an empirical estimate — they were not derived from incident-frequency data — and reflect the intuition that mass-casualty-scale demand is rare but not negligible for a Level I center over a given day.

## Total available supply

Total daily supply available to the network is set at 28 units. This follows from summing normal-day demand across all six hospitals (40 units) and applying the reduction in Red Cross collections reported during the 2026 shortage, which fell 25-35% depending on the report cited. Using a 30% reduction: 40 x 0.70 ≈ 28.

## Model formulation

**Sets:** H = hospitals, S = demand scenarios.

**Parameters:** w_h (criticality weight), D_base_h (base daily demand), factor_s (scenario demand multiplier), p_s (scenario probability), I (total supply). Realized demand is D_{h,s} = D_base_h x factor_s.

**Decision variables:** x_h, the units allocated to hospital h before demand is realized (first-stage); u_{h,s}, the unmet demand at hospital h under scenario s (second-stage, recourse).

**Objective:** minimize the expected, criticality-weighted shortage:

min sum over h,s of p_s * w_h * u_{h,s}

**Constraints:**
- sum over h of x_h <= I (total allocation cannot exceed available supply)
- u_{h,s} >= D_{h,s} - x_h for all h, s (shortage is realized demand minus allocation, when positive)
- x_h, u_{h,s} >= 0

An equity-floor variant adds x_h >= min_fraction x D_base_h for all h, guaranteeing every hospital a minimum share of its baseline demand regardless of criticality weight.

## Sources

1. Nurse.org, "Red Cross Blood Crisis: What Nurses Know," July 2026. nurse.org/news/red-cross-blood-crisis-what-nurses-know
2. Axios, "US blood supply crisis prompts Red Cross donation push," July 28, 2026. axios.com/2026/07/28/us-blood-supply-health-red-cross-donate
3. American Red Cross, press release, "Red Cross Declares Emergency Blood Shortage," 2026. redcross.org/about-us/news-and-events/press-release/2026/red-cross-declares-emergency-blood-shortage.html
4. American Red Cross, press release, "Red Cross Declares Shortage After Blood Supply Falls 35%," 2026. redcross.org/about-us/news-and-events/press-release/2026/red-cross-declares-shortage-after-blood-supply-falls-35-.html
5. Deseret News, "American Red Cross declares blood shortage emergency, hospitals postpone surgeries," July 28, 2026.
6. American College of Surgeons, ACS Brief, "US Faces a Blood Supply Crisis That Could Impact Surgery," August 4, 2026.
7. WTTW Chicago, "Red Cross, Hospitals Confront Challenge of Nationwide Blood Shortage," August 3, 2026.
8. American College of Surgeons, "Trauma Systems, Part IV" (national trauma center counts by level). facs.org/quality-programs/trauma/systems/trauma-series/part-iv
9. OTA International, "Trauma Center Proliferation in the United States," 2025.
10. Nursa.com, "Trauma Center Levels Explained." nursa.com/blog/trauma-center-levels
11. OU Health, "The Vital Role of Blood Donations in Level I Trauma Care," June 2025.
12. National Library of Medicine (PMC), "Optimizing Blood Product Use in Trauma Care." pmc.ncbi.nlm.nih.gov/articles/PMC3226118
13. National Library of Medicine (PMC), "Optimal Use of Blood Products in Trauma Patients." ncbi.nlm.nih.gov/pmc/articles/PMC3126660

## Limitations

This is a small representative network, not a claim to model the US trauma system as a whole; its purpose is to demonstrate the allocation logic at a scale where the trade-offs remain interpretable. Several parameters — hospital-level demand, scenario probabilities, and criticality weights — are engineering assumptions grounded in the best available public statistics rather than internal Red Cross data, which is not published. The Pareto analysis and sensitivity checks in the accompanying notebook are intended to show how much the results depend on these choices.
