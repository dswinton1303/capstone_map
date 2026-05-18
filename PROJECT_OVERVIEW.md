# EdTrust College Affordability Gap — Project Overview

An interactive visualization of the college **affordability gap**: the difference between each institution's *reported* cost of attendance and a *modeled* total living-cost benchmark built from external data sources (HUD rents, USDA food plans, BLS transportation, IPEDS tuition/COA).

The deliverable is a single-page web map (`index.html`) covering **2,796 U.S. institutions**, with sidebar filters, multiple color encodings, an institution search, and detailed per-school popups.

Open `index.html` (or visit the GitHub Pages site) to use it.

---

## Live App

| File | Purpose |
|---|---|
| `index.html` | Single-page map app — Leaflet + vanilla JS, no build step |
| `colleges.json` | Per-institution dataset consumed by `index.html` |
| `affordability_gap_clean.csv` | Underlying tabular dataset used to build `colleges.json` |
| `docs/presentations/` | Weekly status decks (see below) |

---

## Affordability Framework

```
Modeled Total Living Cost = HUD Rent + Food Assumption + Transportation Assumption
Modeled Total Affordability Gap = Modeled Total Living Cost − Reported Living Cost
```

* **Reported Living Cost**: IPEDS Cost of Attendance minus tuition and fees.
* **Sign convention**: positive gap = institution underestimates local living cost; negative gap = reported COA exceeds modeled local living cost.

### Cost Assumptions (baseline scenario)

| Component   | Source                                            | Adjusted by                  | 9-mo total |
|-------------|---------------------------------------------------|------------------------------|-----------:|
| Housing     | HUD Fair Market Rents — 1-bedroom, school CBSA    | School location              |  varies    |
| Food        | USDA Low-Cost Food Plan (Mar 2026)                | State price level + locale   |  $3,240    |
| Transport   | BLS Consumer Expenditure Survey                   | BEA region + locale          |  $1,800    |

Sensitivity scenarios (`docs/presentations/From_Housing_Gaps_to_Full_LivingCost_Modeling.pdf`):

| Scenario  | Food (mo) | Transport (mo) | 9-mo total | Use case                       |
|-----------|----------:|---------------:|-----------:|--------------------------------|
| Low       | $291      | $100           | $3,519     | Walkable / transit-heavy       |
| Baseline  | $360      | $200           | $5,040     | National student default       |
| High      | $449      | $350           | $7,191     | Car-dependent / high-mobility  |

### Key Data Decisions
* 9-month academic year (matches institutional reporting).
* USDA food: averaged male/female figures across student population.
* Missing rent for 106 schools imputed with median within (region × locale) — missingness was concentrated in small towns, not random.
* Special-focus schools (theological, medical, law) excluded from gap analysis because of missing tuition data.
* 474 schools with all tuition data missing are excluded from gap calculations but retained in display where coordinates allow.

---

## Modeling (Week 7)

| Model                          | Drop NAs | R² | Accuracy | Notes |
|--------------------------------|:--------:|:--:|:--------:|-------|
| Logistic regression            | yes      | 0.72 | 90.6%  | Predicts whether modeled gap > 0 |
| OLS regression                 | yes      | 0.79 | 91.4%  | Predicts modeled total gap ($) |
| Decision tree (regression)     | yes      | 0.88 | 85.5%  | Interpretable, splits on IPEDS COA + annual rent |
| Random forest (regression)     | yes      | 0.87 | 88.4%  | Highest accuracy & R² overall |
| Random forest (NA retained)    | no       | lower | same   | Same accuracy, lower R² |

**Predictors:** Pell-grant share, urbanicity (rural/suburb/town/city), Carnegie classification, BEA region. Annual rent dropped from independent variables after Week 7 to avoid circularity (rent is a component of the dependent variable). 466 rows with NAs dropped.

Earlier OLS specifications (Week 6):
* Baseline: R² 0.289, adj. R² 0.306, n=721
* Including rent: R² 0.61, adj. R² 0.60, n=739 (circularity issue later corrected)
* Updated CSV: R² 0.365, adj. R² 0.351, n=2263 — most representative

---

## Interactive Map Features

* **Map**: Leaflet w/ CARTO Light basemap; markers colored by selected encoding.
* **Color modes**: Affordability Gap $, Pell %, BEA Region.
* **Filters**: BEA region chips, locale (urban/suburb/town/rural), Pell quartile, Carnegie classification, gap-$ range slider, full-text institution search, reset-all.
* **Popups**: school name, city/state/ZIP, big-number gap, Pell %, Pell quartile, locale, BEA-region tag, Carnegie classification.
* **Counters**: live count of visible institutions; bucketed counts by color class.

### Per-institution fields (`colleges.json`)

`id`, `name`, `city`, `state`, `zip`, `cbsa`, `lat`, `lng`, `gap_pct`, `affordability_gap`, `pct_pell`, `pell_quartile`, `locale_type`, `bea_region`, `carnegie_class`.

### Coordinate provenance

`lat`/`lng` are re-geocoded from each institution's `zip` against an open-source ZIP-centroid dataset, with a `(city, state)` fallback when the ZIP centroid is unreliable (a small number of ZIPs in the source dataset share a default "state region" centroid; those are detected and rejected). When multiple institutions share an identical centroid (same ZIP), a deterministic radial jitter (~0.3 mi for shared-ZIP, ~0.8 mi for city-fallback) is applied so points don't visually stack on top of each other. The original `colleges.json` shipped with scrambled coordinates (Boston schools rendering in Florida, etc.) — this is now fixed.

---

## Presentations

Slide decks are mirrored in `docs/presentations/`:

| File | When | Topic |
|---|---|---|
| `docs/presentations/Week7_EdTrust_Model_Updates_and_Map.pptx` | Week 7 | Model comparison (logistic, OLS, decision tree, random forest); heat-map prototype |
| `docs/presentations/Week6_EdTrust_Affordability_Gap_Updates.pdf` | Week 6 | Data sourcing, key decisions, baseline OLS + logistic results |
| `docs/presentations/From_Housing_Gaps_to_Full_LivingCost_Modeling.pdf` | Methodology brief | Full framework: housing → food + transport, sensitivity scenarios, ranking & equity lens |

---

## Goal

Publish a **public interactive cost-of-attendance map** on the EdTrust website (akin to the MIT Living Wage Calculator) that lets stakeholders filter by region, urbanicity, and Pell share to identify alignment trends in higher-ed cost reporting.

### Core Principles
1. Maintain transparency about modeled assumptions.
2. Emphasize *alignment* with local benchmarks rather than institutional intent.
3. Ground metrics in student lived reality (HUD/USDA/BLS, not self-reported).

### Limitations
* Food and transportation are evidence-based *modeled* assumptions, not observed institution-specific data.
* COA data is missing for many institutions, shrinking the analytic sample.
* Results describe statistical alignment patterns, not causal claims about institutional intent.
* Sensitivity analysis is essential: assumptions materially affect gap sizes.
