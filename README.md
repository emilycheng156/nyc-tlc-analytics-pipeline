# NYC Transportation Market Analysis Data Pipeline

This repository contains the backend data engineering and analytical preprocessing pipeline for a collaborative transportation market analysis project focused on NYC taxi and ride-hail activity.

The broader project explored transportation demand patterns, market share shifts, and geographic trip behavior using public NYC TLC trip records. My role focused on designing and building the data pipeline that transformed raw trip records into clean, reproducible datasets used by the visualization team.

---

## Project Context

This work was developed as part of a collaborative data visualization project examining transportation trends in New York City.

The broader team focused on questions such as:

- How has taxi vs ride-hail market share changed over time?
- Where are transportation demand hotspots across NYC?
- How do travel patterns vary by hour and day of week?
- How do demand patterns differ across transportation service types?

The frontend visualizations were built separately. This repository specifically highlights the backend data processing and analytical work that I owned.

---

## My Role

As the primary contributor responsible for data preparation, I was responsible for:

- sourcing and organizing public NYC TLC trip datasets
- reconciling schema differences across yellow taxi, green taxi, FHV, and HVFHV records
- designing SQL-based aggregation workflows using DuckDB
- cleaning and transforming large trip datasets
- generating derived datasets based on evolving requirements from visualization teammates
- validating outputs before frontend integration
- restructuring data products as analytical questions changed

One of the core challenges was translating high-level analytical requests into usable datasets. For example, generating transportation market share trends, zone-level hourly demand data, and standardized hour-by-weekday activity grids required balancing analytical correctness, reproducibility, and frontend usability.

---

## Data Scope

The project uses public NYC Taxi & Limousine Commission (TLC) trip record data spanning multiple transportation service types and multiple years of trip activity.

Datasets include:

- Yellow Taxi
- Green Taxi
- For-Hire Vehicles (FHV)
- High-Volume For-Hire Vehicles (HVFHV)

Because these datasets contain millions of trip records, the pipeline uses DuckDB for efficient SQL-based querying over parquet files instead of loading all raw data directly into memory.

Data source:

https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

---
# Raw Data

Raw NYC TLC parquet datasets are not included because of file size.

Download public NYC TLC trip records from:

https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

Expected folders:

- data/raw/yellow/
- data/raw/green/
- data/raw/fhv/
- data/raw/hvfhv/

---

## Analytical Outputs

This pipeline generates three datasets.

### 1. Zone-Level Hourly Transportation Activity (`zone_hourly.csv`)

Used for geographic analysis of transportation demand.

Includes:

- pickup zone
- borough
- pickup date
- pickup hour
- vehicle type
- trip counts
- average trip price (where available)

Key processing:

- aggregate hourly trip counts by pickup zone
- merge borough metadata
- generate complete zone/hour/vehicle combinations for consistency
- fill missing combinations with zero-trip observations

---

### 2. Yearly Market Share Analysis (`yearly_market_share.csv`)

Used to analyze how NYC transportation market composition changed over time, particularly the growth of ride-hail relative to traditional taxi services.

Includes:

- year
- transportation category
- trip counts
- market share percentages

Key analytical challenge:

NYC TLC data began reporting High-Volume For-Hire Vehicle (HVFHV) trips as a separate dataset in 2019. Before that, ride-hail trips from services such as Uber and Lyft were mixed into the broader FHV dataset, making direct year-over-year comparisons difficult.

To create a consistent historical market share analysis, this pipeline reconstructs pre-2019 ride-hail activity by identifying dispatching base numbers associated with high-volume operators (e.g., Uber, Lyft, Via, Juno) from the dedicated HVFHV dataset and using those identifiers to classify older FHV trip records as either:

- `hvfhv` (ride-hail / high-volume for-hire)
- `other_fhv` (traditional for-hire vehicle activity)

Key processing:

- aggregate trip counts by year and vehicle category
- reconstruct pre-2019 ride-hail classifications from mixed FHV records
- calculate annual market share proportions
- enforce consistent output structure across all years for downstream comparison

---

### 3. Hour-by-Weekday Demand Patterns (`heatmap_data.json`)

Used for temporal demand analysis.

Key processing:

- aggregate trip counts by hour and weekday
- standardize full hour/day grids
- combine multiple vehicle categories into unified output structures

---

## Technologies Used

- Python
- Pandas
- DuckDB
- SQL
- Parquet

---

## Repository Structure

```text
nyc-tlc-analytics-pipeline/
├── README.md
├── requirements.txt
├── .gitignore
├── src/
│   └── build_datasets.py
├── data/
│   ├── raw/
│   ├── processed/
│   └── zones/
└── docs/
```

---

## Setup

Install dependencies:

```bash
pip install -r requirements.txt
```

Place raw TLC parquet files in:

```text
data/raw/yellow/
data/raw/green/
data/raw/fhv/
data/raw/hvfhv/
```

Place taxi zone lookup file in:

```text
data/zones/taxi_zone_lookup.csv
```

Run:

```bash
python src/build_datasets.py
```

---

## Notes

This repository is intended as a code sample highlighting reproducible data engineering, analytical preprocessing, SQL-based aggregation, and dataset construction from large public datasets.