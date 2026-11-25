# Volunteer Management & Performance Analysis Dashboard

**An interactive Power BI dashboard that turns raw volunteer records into decision-ready insights about applications, participation trends, engagement levels and operational performance for a nonprofit organization.**

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black) ![DAX](https://img.shields.io/badge/Language-DAX-7B7FDC) ![Status](https://img.shields.io/badge/Status-Complete-3FB58F) ![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Project Overview

This project analyzes volunteer data for a nonprofit organization through a single-page Power BI report. It covers volunteer applications, geography, education, car ownership and logged volunteering hours, and exposes them through KPI cards, trend charts, rankings and slicers.

The source `volunteers.csv` is cleaned in **Power Query** (header promotion, type casting, replacement of inconsistent boolean text, removal of unnecessary columns and date normalization), then loaded into a compact model with a DAX-generated `dcalendar` table and a `Key_Measures` table holding the report's measures. The complete 13-table backend schema is documented in [`docs/Database_Tables_Reference.md`](docs/Database_Tables_Reference.md).

---

## Project Structure

```
volunteer-management-performance-dashboard/
│
├── PowerBI/
│   └── Edman.pbix                         # Power BI report
│
├── data/
│   ├── volunteers.csv                     # Main volunteer records used by the report
│   ├── users.csv                          # User accounts and roles
│   ├── admins.csv                         # Admin assignments
│   ├── sub_admins.csv                     # Sub-admin assignments
│   ├── centers.csv                        # Volunteer centers
│   ├── places.csv                         # Field locations with GPS thresholds
│   ├── rules.csv                          # Versioned volunteer rules
│   ├── tasks.csv                          # Scheduled volunteer tasks
│   ├── task_enrollments.csv               # Volunteer-to-task enrollments
│   ├── sessions.csv                       # Check-in / check-out sessions
│   ├── session_photos.csv                 # Photos attached to sessions
│   └── gps_audit_logs.csv                 # GPS validation audit trail
│
├── docs/
│   └── Database_Tables_Reference.md       # Full schema, enums and relationships
│
├── screenshots/
│   ├── Screenshot 2026-04-02 224813.png   # Volunteer Overview dashboard
│   └── Screenshot (209).png               # Power Query cleaning steps
│
├── .gitignore
├── LICENSE
└── README.md
```

---

## Dashboard Preview

### Volunteer Overview

![Volunteer Overview Dashboard](screenshots/Screenshot%202026-04-02%20224813.png)

### Power Query — Volunteers Table

![Power Query Cleaning](screenshots/Screenshot%20%28209%29.png)

---

## Data Architecture

```
volunteers.csv
      │
      ▼
Power Query (promote headers, set types, replace True/False text,
             remove unused columns, cast appliedAt to date)
      │
      ▼
volunteers table  ──  dcalendar (CALENDARAUTO)  ──  Key_Measures (DAX)
      │
      ▼
1-page Power BI report (18 visuals)
```

- **`volunteers`** is the fact/grain table: one row per volunteer record (140 rows).
- **`dcalendar`** is a DAX calculated table built with `CALENDARAUTO()` and helper columns for Year, Quarter, Month and Month Number. In the delivered report it is used as a disconnected date table for date labels and slicers — there is no active relationship in the PBIX.
- **`Key_Measures`** is a calculated table that hosts the four DAX measures.
- Power BI also generates hidden local date tables for auto date/time hierarchies.

---

## Data Catalog

### `volunteers` (model table)

| Column | Type | Purpose |
| --- | --- | --- |
| `fullName` | Text | Volunteer name (drives Top 10 ranking) |
| `governorate` | Text | Governorate slicer and grouping |
| `area` | Text | Area slicer and "Volunteers by Area" chart |
| `educationalLevel` | Text | Education pie chart |
| `hasCar` | Text | Car ownership pie chart |
| `applicationStatus` | Text | Approved / Pending / Rejected / Banned KPIs |
| `appliedAt` | Date | Application date for monthly trend |
| `totalVolunteeringHours` | Decimal | Hours logged, used in rankings and totals |

### `dcalendar` (calculated table)

```dax
dcalendar =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month", FORMAT([Date], "mmm"),
    "Month Number", MONTH([Date])
)
```

### Source database (13 CSVs)

The full entity-relationship reference — enums, primary/foreign keys, constraints and a relationship diagram covering users, volunteers, centers, places, rules, tasks, enrollments, sessions, photos and GPS logs — lives in [`docs/Database_Tables_Reference.md`](docs/Database_Tables_Reference.md).

---

## DAX Measures

| Measure | Expression |
| --- | --- |
| `Total Volunteers` | `COUNTROWS(volunteers)` |
| `Accepted` | `CALCULATE(COUNTROWS(volunteers), volunteers[applicationStatus] = "approved")` |
| `Rejected` | `CALCULATE(COUNTROWS(volunteers), volunteers[applicationStatus] = "rejected")` |
| `Pending` | `CALCULATE(COUNTROWS(volunteers), volunteers[applicationStatus] = "pending")` |

Implicit measures such as `Sum(totalVolunteeringHours)` power the hours card and the monthly combo chart.

---

## Dashboard Pages & Visuals

The report has one page, **"Vounteer Overview"** (original spelling preserved from the PBIX), with 18 visual containers:

| Area | Visuals |
| --- | --- |
| KPI cards | Accepted, Rejected, Pending, Total Volunteers, Hours Logged |
| Trends | Monthly Volunteers vs. Volunteering Hours (line + stacked column combo) |
| Ranking | Top 10 Volunteers By Hours (bar chart with TopN filter) |
| Geography | Volunteers by Area (clustered column) |
| Profile | Educational Level and Has a Car pie charts, with Arabic data labels |
| Slicers | Month/Year dropdown, Area dropdown, Governorate dropdown |
| Branding | Title textbox and supporting images |

---

## Key Findings

- **112 approved**, **19 pending**, **6 rejected** and **3 banned** out of 140 volunteer records — an **81.7% acceptance rate** across the 137 non-banned applications.
- **8,202.73 volunteering hours** logged in total; the top contributor, **Israa Wael Al‑Shenawi**, logged **149.61 hours**.
- **February 2024** was the peak intake month with **19 new applications**.
- **Al‑Dokki** was the most active area with **36 volunteers**, ahead of Faisal and Mohandeseen.
- **50%** of volunteers hold a bachelor's degree and **77.14%** do not own a car — useful inputs for field-operation logistics.

---

## Tools & Technologies

| Category | Tool | Purpose |
| --- | --- | --- |
| Data preparation | Power Query (M) | Cleaning, type casting, column removal, text replacement |
| Data modeling | Power BI Desktop | Table/model construction and report layout |
| Calculation | DAX | Four explicit measures plus implicit aggregations |
| Documentation | Markdown | Schema reference and README |
| Version control | Git & GitHub | Project hosting |

---

## Getting Started

1. Clone this repository.
2. Open [`PowerBI/Edman.pbix`](PowerBI/Edman.pbix) in **Power BI Desktop**.
3. If prompted to refresh, point the `volunteers` Power Query source to the local copy of `data/volunteers.csv` — the PBIX stores the original authoring path.
4. Explore the single report page and use the Month/Year, Area and Governorate slicers.

---

## License

MIT License — see [`LICENSE`](LICENSE).

## Author

**Mohamed Basem**
