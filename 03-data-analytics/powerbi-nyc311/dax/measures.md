# DAX Measures — NYC311 Data Platform

All measures are housed in the `_Measures` table. The leading underscore sorts the table to the top of the field list in Power BI Desktop and keeps all measures isolated from dimension and fact table columns.

Measures are organized into four display folders visible in the field list:

| Display Folder | Measures |
|---|---|
| **Core** | Total Complaints, Open Complaints, Closed Complaints, Total Closed, Overdue Complaints, Overdue Rate, Avg Monthly Complaints |
| **Resolution** | Avg Resolution Hours, Avg Resolution Days, Avg Resolution Days by Close Date, Median Resolution Hours |
| **Time Intelligence - CY** | YTD Complaints, MTD Complaints, Complaints Last 30 Days, Complaints vs Prior Year, YoY Change, YoY Change % |
| **Time Intelligence - FY** | Fiscal YTD Complaints, Fiscal QTD Complaints |

The active relationship between `Fact_Service_Request` and `Dim_Date` is on `date_key_created`. Measures requiring a different date context activate the appropriate inactive relationship explicitly via `USERELATIONSHIP`.

Aggregation-aware measures marked with ⚡ are eligible to resolve against `gold_agg_date_borough` or `gold_agg_date_complaint` Import-mode tables rather than scanning `Fact_Service_Request` via DirectQuery, depending on the filter context of the visual.

---

## Core

---

### Total Complaints ⚡

**Purpose:** Base complaint count. The foundation measure referenced by all other volume and time-intelligence measures.

**Format:** `0`

```dax
Total Complaints =
COUNTROWS(Fact_Service_Request)
```

**Notes:** Aggregation tables store `total_complaints` at pre-computed grain levels. When filtered by `date_key_created` and `location_key` or `complaint_key`, the engine resolves this measure from Import-mode agg tables rather than scanning the DirectQuery fact.

---

### Open Complaints

**Purpose:** Count of complaints currently in an open status.

**Format:** `0`

```dax
Open Complaints =
CALCULATE(
    [Total Complaints],
    Dim_Status[status] = "Open"
)
```

**Notes:** Filters through `Dim_Status`. Because status is not included in the aggregation table grain, this measure always resolves via DirectQuery.

---

### Closed Complaints

**Purpose:** Count of complaints with a closed status.

**Format:** `0`

```dax
Closed Complaints =
CALCULATE(
    [Total Complaints],
    Dim_Status[status] = "Closed"
)
```

---

### Total Closed

**Purpose:** Count of complaints that have a closed date recorded — evaluated in the context of the closed date relationship rather than the created date. Used as a KPI card showing work completed within a selected time period.

**Format:** `0`

```dax
Total Closed =
CALCULATE(
    [Total Complaints],
    USERELATIONSHIP(
        Fact_Service_Request[date_key_closed],
        Dim_Date[date_key]
    ),
    NOT ISBLANK(Fact_Service_Request[date_key_closed])
)
```

**Notes:** Activates the inactive `date_key_closed` → `Dim_Date[date_key]` relationship, so date slicers filter by close date rather than created date. The `NOT ISBLANK` filter excludes open complaints that have no closed date. Distinct from `Closed Complaints`, which filters by status value and uses the active (created date) relationship.

---

### Overdue Complaints ⚡

**Purpose:** Count of complaints where the SLA due date was breached.

**Format:** `0`

```dax
Overdue Complaints =
CALCULATE(
    [Total Complaints],
    Fact_Service_Request[is_overdue] = TRUE()
)
```

**Notes:** `is_overdue` is a boolean flag set at the Gold layer. The `gold_agg_date_borough` and `gold_agg_date_complaint` tables include `overdue_count`, enabling aggregation hits when filtered by date and location or complaint type.

---

### Overdue Rate

**Purpose:** Percentage of total complaints that breached SLA.

**Format:** `0.0%;-0.0%;0.0%`

```dax
Overdue Rate =
DIVIDE(
    [Overdue Complaints],
    [Total Complaints],
    0
)
```

**Notes:** `DIVIDE` with a zero alternate result prevents division-by-zero errors when no complaints exist in the current filter context.

---

### Avg Monthly Complaints

**Purpose:** Average monthly complaint volume across all months in the current filter context. Used as a benchmark reference line on trend visuals.

```dax
Avg Monthly Complaints =
AVERAGEX(
    VALUES(Dim_Date[cal_year_month_key]),
    CALCULATE([Total Complaints])
)
```

**Notes:** `VALUES(Dim_Date[cal_year_month_key])` iterates over each distinct month in the current filter context, computing `Total Complaints` for each and averaging the results. Produces a stable monthly average regardless of how many months are selected.

---

## Resolution

---

### Avg Resolution Hours

**Purpose:** Average resolution duration in hours across complaints with a recorded resolution time.

```dax
Avg Resolution Hours =
CALCULATE(
    AVERAGE(Fact_Service_Request[resolution_hours]),
    Fact_Service_Request[resolution_hours] > 0
)
```

**Notes:** The `resolution_hours > 0` filter excludes open complaints and any records where duration could not be calculated. `sum_resolution_hours` and `count_resolution_hours` in the aggregation tables enable this measure to resolve from Import-mode agg tables when the filter context matches the agg grain — the engine computes the average as `sum / count` from pre-aggregated values.

---

### Avg Resolution Days

**Purpose:** Average resolution duration converted from hours to days.

**Format:** `0.0`

```dax
Avg Resolution Days =
DIVIDE([Avg Resolution Hours], 24, 0)
```

---

### Avg Resolution Days by Close Date

**Purpose:** Average resolution performance measured against the closed date. Used to evaluate how long complaints resolved within a selected period actually took — regardless of when they were created.

```dax
Avg Resolution Days by Close Date =
CALCULATE(
    [Avg Resolution Days],
    USERELATIONSHIP(
        Fact_Service_Request[date_key_closed],
        Dim_Date[date_key]
    )
)
```

**Notes:** Activates the inactive `date_key_closed` → `Dim_Date[date_key]` relationship. Date slicers filter by close date rather than created date for this measure only, allowing period-specific resolution performance reporting.

---

### Median Resolution Hours

**Purpose:** Median resolution duration in hours. More robust than the average for skewed distributions where a small number of very long-running complaints inflate the mean.

```dax
Median Resolution Hours =
CALCULATE(
    MEDIAN(Fact_Service_Request[resolution_hours]),
    Fact_Service_Request[resolution_hours] > 0
)
```

**Notes:** `MEDIAN` always resolves via DirectQuery — there is no aggregation table equivalent. Expect higher query latency for this measure on large filtered datasets compared to the average resolution measures.

---

## Time Intelligence — Calendar Year

All CY time-intelligence measures operate against `Dim_Date[date]` via the active `date_key_created` relationship.

---

### YTD Complaints ⚡

**Purpose:** Year-to-date complaint count based on calendar year.

**Format:** `0`

```dax
YTD Complaints =
CALCULATE(
    [Total Complaints],
    DATESYTD(Dim_Date[date])
)
```

---

### MTD Complaints ⚡

**Purpose:** Month-to-date complaint count.

**Format:** `0`

```dax
MTD Complaints =
CALCULATE(
    [Total Complaints],
    DATESMTD(Dim_Date[date])
)
```

---

### Complaints Last 30 Days

**Purpose:** Rolling 30-day complaint count anchored to the latest date in the current filter context.

**Format:** `0`

```dax
Complaints Last 30 Days =
CALCULATE(
    [Total Complaints],
    DATESINPERIOD(
        Dim_Date[date],
        MAX(Dim_Date[date]),
        -30,
        DAY
    )
)
```

**Notes:** `MAX(Dim_Date[date])` adapts to the current filter context — without a date slicer it uses the latest loaded date; with a date slicer it uses the last date in the selected range.

---

### Complaints vs Prior Year ⚡

**Purpose:** Total complaints for the same period in the prior calendar year. Used as the denominator in YoY calculations.

**Format:** `0`

```dax
Complaints vs Prior Year =
CALCULATE(
    [Total Complaints],
    SAMEPERIODLASTYEAR(Dim_Date[date])
)
```

---

### YoY Change

**Purpose:** Absolute year-over-year variance in complaint volume.

**Format:** `0`

```dax
YoY Change =
[Total Complaints] - [Complaints vs Prior Year]
```

---

### YoY Change %

**Purpose:** Percentage year-over-year variance.

**Format:** `0.0%;-0.0%;0.0%`

```dax
YoY Change % =
DIVIDE(
    [YoY Change],
    [Complaints vs Prior Year],
    0
)
```

---

## Time Intelligence — Fiscal Year

NYC fiscal year runs July 1 through June 30. `Fiscal YTD Complaints` passes `"6/30"` as the year-end date argument to `DATESYTD`. `Fiscal QTD Complaints` uses `DATESQTD` against the calendar quarter boundary — for a true fiscal quarter boundary, use the `fy_quarter` column from `Dim_Date` if finer fiscal alignment is required.

---

### Fiscal YTD Complaints ⚡

**Purpose:** Year-to-date complaint count based on NYC fiscal year (July 1 – June 30).

**Format:** `0`

```dax
Fiscal YTD Complaints =
CALCULATE(
    [Total Complaints],
    DATESYTD(Dim_Date[date], "6/30")
)
```

---

### Fiscal QTD Complaints ⚡

**Purpose:** Quarter-to-date complaint count.

**Format:** `0`

```dax
Fiscal QTD Complaints =
CALCULATE(
    [Total Complaints],
    DATESQTD(Dim_Date[date])
)
```

---

## Aggregation Coverage Summary

| Measure | Agg Table | Notes |
|---|---|---|
| Total Complaints | `gold_agg_date_borough`, `gold_agg_date_complaint` | Resolves from `total_complaints` column |
| Overdue Complaints | `gold_agg_date_borough`, `gold_agg_date_complaint` | Resolves from `overdue_count` column |
| YTD / MTD variants | `gold_agg_date_borough`, `gold_agg_date_complaint` | Date filter applied over agg grain |
| Complaints vs Prior Year | `gold_agg_date_borough`, `gold_agg_date_complaint` | Prior year date shift applied over agg grain |
| Avg Resolution Hours / Days | `gold_agg_date_borough`, `gold_agg_date_complaint` | Computed as `sum_resolution_hours / count_resolution_hours` |
| Open / Closed Complaints | DirectQuery only | Status not in agg grain |
| Total Closed | DirectQuery only | Uses inactive date relationship |
| Median Resolution Hours | DirectQuery only | No agg equivalent for MEDIAN |
| Complaints Last 30 Days | DirectQuery only | Rolling window not pre-aggregated |
| Avg Monthly Complaints | DirectQuery only | Month-level iteration not in agg grain |

---

## Validation

All measures were validated using DAX Studio with **Query Plan** and **Server Timings** traces enabled against the workspace semantic model via the Analysis Services endpoint. Saved DAX Studio queries are stored in `reports/NYC311-Report.SemanticModel/DAXQueries/` and cover aggregation hit validation and fallback diagnostics. Aggregation hits are confirmed by `VertiPaq SE` cache reads in the query plan; DirectQuery-only measures were verified to fall back correctly and perform within acceptable latency thresholds for their intended visual usage.
