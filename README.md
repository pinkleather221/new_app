# Retail & Travel Analytics

An exploratory data analysis (EDA) workspace covering **two business domains**:

1. **Pharmacy retail sales** — a Kenyan pharmacy chain selling in KES via M-Pesa, card, cash and insurance.
2. **Travel package bookings** — multi-year (2023–2025) booking data for holiday packages priced in USD.

The goal is to take messy, real-world exports and turn them into clean, analysis-ready datasets, then surface the business signals hiding inside them (revenue trends, discount behaviour, channel performance, data-quality traps, and more).

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Datasets](#datasets)
- [What the Analysis Covers](#what-the-analysis-covers)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Data Quality Notes](#data-quality-notes)
- [Roadmap](#roadmap)
- [Author](#author)

---

## Overview

This repository is a notebook-driven analytics lab. Each notebook is a self-contained investigation of one dataset, following a consistent workflow:

1. **Load & profile** — shape, dtypes, nulls, duplicates, summary statistics.
2. **Clean** — normalise categorical text, parse dates, fix phone numbers, handle missing values and outliers.
3. **Engineer** — derive new analytical features (net value, discount bands, booking size, etc.).
4. **Aggregate & explore** — group by year, product, package, city and channel to find trends.

Reusable analysis logic lives in an installable Python package under `src/`, so the notebooks focus on exploration while common code can graduate into the package over time.

---

## Project Structure

```
retail-travel-analytics/
├── data/
│   ├── raw/                      # Original, untouched source data
│   │   ├── Bookings_2023.csv
│   │   ├── Bookings_2024.csv
│   │   └── Bookings_2025.csv
│   └── processed/                # Cleaned / transformed outputs (git-ignored)
├── notebooks/
│   ├── bookings.ipynb            # Travel bookings EDA
│   └── pharmacy_sales.ipynb      # Pharmacy sales cleaning & EDA
├── outputs/                      # Charts, exports and reports
├── src/
│   └── retail_travel_analytics/  # Installable analysis package
│       └── __init__.py
├── pyproject.toml                # Project metadata & dependencies (uv)
├── uv.lock                       # Locked dependency versions
├── .python-version               # Pinned Python version (3.14)
└── README.md
```

---

## Datasets

### 1. Travel Bookings (`data/raw/Bookings_YYYY.csv`)

Booking-level records for holiday/travel packages, split one file per booking year (2023, 2024, 2025).

| Column | Description |
| --- | --- |
| `BookingID` | Unique booking identifier (e.g. `BK-2024-00001`) |
| `BookingDate` | Date the booking was made |
| `TravelStartDate` | Date travel begins |
| `CustomerID` | Customer identifier |
| `PackageID` | Travel package identifier |
| `AgentID` | Selling agent (may be missing) |
| `Adults` | Number of adult travellers |
| `Children` | Number of child travellers |
| `UnitPriceUSD` | Unit price in USD |
| `DiscountPct` | Discount applied (may be missing) |
| `Status` | Booking status (e.g. `Completed`, `Cancelled`) |
| `PaymentMethod` | Card, PayPal, Bank Transfer, etc. |

**Engineered features:** `Travellers`, `GrossValue`, `DiscountAmount`, `NetBookingValue`, `BookingSize` (Small/Large) and `DiscountBand` (No/Low/High discount).

### 2. Pharmacy Sales (`pharmacy_sales_raw.csv`)

~6,150 order lines (26 columns) from a Kenyan pharmacy retail operation. Key fields include:

- **Order:** `order_id`, `order_date`, `delivery_date`
- **Customer:** `customer_id`, `customer_name`, `customer_phone`, `customer_email`, `customer_gender`, `customer_age`, `city`
- **Product:** `product_id`, `product_name`, `category`, `requires_prescription`, `expiry_date`, `unit_price_kes`, `quantity`
- **Commercial:** `branch`, `channel`, `discount_pct`, `promo_code`, `total_amount_kes`, `delivery_fee_kes`, `payment_method`
- **Feedback:** `customer_rating`, `customer_review`

> **Note:** The raw pharmacy CSV is read from a local path in the notebook and is not tracked in the repo. Place `pharmacy_sales_raw.csv` in `data/raw/` and update the path in the notebook before running it.

---

## What the Analysis Covers

**`notebooks/bookings.ipynb`**
- De-duplicates bookings by `BookingID` and standardises status text.
- Parses booking/travel dates and extracts booking year and month.
- Derives net booking value, discount amounts and discount bands.
- NumPy practice: array indexing, boolean masks, aggregations.
- Yearly rollup of bookings, net value and travellers, including year-over-year growth.
- Package-level performance ranking by total bookings, travellers and net revenue.

**`notebooks/pharmacy_sales.ipynb`**
- Full profiling: shape, dtypes, nulls, duplicates and summary statistics.
- Normalises dirty categoricals — `city` (Nairobi/NRB/NAIROBI), `category`, `payment_method` (M-Pesa/mpesa/MPESA), `promo_code`, and `requires_prescription` (`Y/Yes/1/TRUE` → boolean).
- Outlier inspection with box plots and sensible business rules.
- Cleans product names, customer names, phone numbers and review text.
- Handles duplicates and casts dates to proper datetime types.

---

## Tech Stack

| Tool | Purpose |
| --- | --- |
| [Python 3.14](https://www.python.org/) | Language runtime |
| [uv](https://docs.astral.sh/uv/) | Dependency management & packaging |
| [pandas](https://pandas.pydata.org/) ≥ 3.0 | Data manipulation |
| [NumPy](https://numpy.org/) ≥ 2.5 | Numerical operations |
| [matplotlib](https://matplotlib.org/) ≥ 3.11 | Plotting |
| [seaborn](https://seaborn.pydata.org/) ≥ 0.13 | Statistical visualisation |
| [PyArrow](https://arrow.apache.org/docs/python/) ≥ 25.0 | Columnar/string backend |

---

## Getting Started

### Prerequisites

- Python **3.14** (see `.python-version`)
- [`uv`](https://docs.astral.sh/uv/getting-started/installation/) installed

### Install

```bash
# Clone the repository
git clone https://github.com/<your-username>/retail-travel-analytics.git
cd retail-travel-analytics

# Create the environment and install dependencies
uv sync
```

### Confirm the package installs

```bash
uv run retail-travel-analytics
# -> Hello from retail-travel-analytics!
```

---

## Usage

Launch JupyterLab and open a notebook, or run one directly:

```bash
uv run jupyter lab
```

```bash
uv run jupyter nbconvert --to notebook --execute notebooks/bookings.ipynb
```

To import the package in your own scripts:

```python
from retail_travel_analytics import main

main()
```

---

## Data Quality Notes

The datasets are deliberately messy and reflect realistic issues:

- **Duplicates** — repeated `BookingID`s and fully duplicated order rows.
- **Inconsistent casing/spacing** — `Nairobi`, `NAIROBI`, `nairobi ` for the same city.
- **Free-text booleans** — `requires_prescription` arrives as `Y/Yes/1/TRUE/FALSE/0/N`.
- **Unnormalised promo codes** — `MYDEAL5`, `mydeal 5`, ` MYDEAL-10 `, etc.
- **Impossible values** — ages of `999`, negative quantities, discounts over 100%.
- **Mixed phone formats** — `+254…`, `254…`, `07…`, and space/dash-separated.
- **Missing data** — `AgentID`, `DiscountPct`, delivery dates and ratings.

Cleaning these issues is part of the analysis and is handled inside the notebooks.

---

## Roadmap

- [ ] Extract reusable cleaning/feature functions from the notebooks into `src/retail_travel_analytics/`.
- [ ] Persist cleaned datasets to `data/processed/` as Parquet.
- [ ] Move `pharmacy_sales_raw.csv` into `data/raw/` and make the path configurable.
- [ ] Save figures to `outputs/` instead of only rendering them inline.
- [ ] Add unit tests for the cleaning and feature-engineering logic.
- [ ] Add a lightweight reporting notebook or dashboard summarising key KPIs.

---

## Author

**pinkleather221** — [nellymogere2002@gmail.com](mailto:nellymogere2002@gmail.com)
