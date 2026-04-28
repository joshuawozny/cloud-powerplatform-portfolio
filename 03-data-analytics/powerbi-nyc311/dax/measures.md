# DAX Measures — NYC311 Data Platform

All measures are housed in the `_Measures` table. The leading underscore sorts the table to the top of the field list in Power BI Desktop and keeps all measures isolated from dimension and fact table columns.

The active relationship between `Fact_Service_Request` and `Dim_Date` is on `date_key_created`. Measures that require a different date context (closed date, due date) use `USERELATIONSHIP` to activate the appropriate inactive relationship explicitly.

Aggregation-aware measures marked with ⚡ are eligible to resolve against `gold_agg_date_borough` or `gold_agg_date_complaint` Import-mode tables rather than hitting `Fact_Service_Request` via DirectQuery, depending on the filter context of the visual.

---

## Volume & Status

---

### Total Complaints ⚡

**Purpose:** Base complaint count. The foundation measure referenced by all other volume and time-intelligence measures.

```dax
Total Complaints =
COUNTROWS(Fact_Service_Request)
```

**Notes:** Aggregation tables store `total_complaints` at pre-computed grain levels. When filtered by `date_key_created` and `location_key` or `complaint_key`, the engine resolves this measure from Import-mode agg tables rather than scanning the DirectQuery fact.

---

### Open Complaints

**Purpose:** Count of complaints currently in an open status.

```dax
Open Complaints =
CALCULATE(
    [Total Complaints],
    Dim_Status[status] = "Open"
)
```

**Notes:** Filters through the `Dim_Status` dimension. Because `status_key` is not included in the aggregation table grain, this measure always resolves via DirectQuery.

---

### Closed Complaints

**Purpose:** Count of complaints with a closed status.

```dax
Closed Complaints =
CALCULATE(
    [Total Complaints],
    Dim_Status[status] = "Closed"
)
```

---

### Total Closed

**Purpose:** Closed complaint count used as a standalone KPI card value, independent of cross-filter context from other visuals.

```dax
Total Closed =
CALCULATE(
    COUNTROWS(Fact_Service_Request),
    Dim_Status[status] = "Closed",
    ALL(Dim_Date),
    ALL(Dim_Location),
    ALL(Dim_Agency),
    ALL(Dim_Complaint)
)
```

**Notes:** The `ALL()` wrappers remove cross-filter context, making this suitable for a header KPI card that always shows the total closed count regardless of report-level slicer selections.

---

### Overdue Complaints ⚡

**Purpose:** Count of complaints where the SLA due date was breached.

```dax
Overdue Complaints =
CALCULATE(
    [Total Complaints],
    Fact_Service_Request[is_overdue] = TRUE()
)
```

**Notes:** `is_overdue` is a boolean flag set at the Gold layer based on `date_key_closed` vs `date_key_due`. The `gold_agg_date_borough` and `gold_agg_date_complaint` tables include `overdue_count`, enabling aggregation hits when filtered by date and location or complaint type.

---

### Overdue Rate

**Purpose:** Percentage of total complaints that breached SLA.

```dax
Overdue Rate =
DIVIDE(
    [Overdue Complaints],
    [Total Complaints],
    0
)
```

**Format string:** `0.0%`

**Notes:** `DIVIDE` with a zero alternate result prevents division-by-zero errors when no complaints exist in the current filter context.

---

### Complaints Last 30 Days

**Purpose:** Rolling 30-day complaint count anchored to the most recent date in the current filter context.

```dax
Complaints Last 30 Days =
CALCULATE(
    [Total Complaints],
    DATESINPERIOD(
        Dim_Date[date],
        LASTDATE(Dim_Date[date]),
        -30,
        DAY
    )
)
```

**Notes:** `LASTDATE` adapts to the current filter context — on a dashboard without a date slicer it uses the latest loaded date, while with a date slicer it uses the last date in the selected range.

---

## Time Intelligence

All time-intelligence measures operate against the `Dim_Date[date]` column via the active `date_key_created` relationship. NYC fiscal year runs July 1 through June 30; fiscal measures use `"6/30"` as the year-end date argument.

---

### YTD Complaints ⚡

**Purpose:** Year-to-date complaint count based on calendar year.

```dax
YTD Complaints =
TOTALYTD(
    [Total Complaints],
    Dim_Date[date]
)
```

---

### Fiscal YTD Complaints ⚡

**Purpose:** Year-to-date complaint count based on NYC fiscal year (July 1 – June 30).

```dax
Fiscal YTD Complaints =
TOTALYTD(
    [Total Complaints],
    Dim_Date[date],
    "6/30"
)
```

---

### Fiscal QTD Complaints ⚡

**Purpose:** Fiscal quarter-to-date complaint count using `fy_quarter` from `Dim_Date`.

```dax
Fiscal QTD Complaints =
CALCULATE(
    [Total Complaints],
    FILTER(
        ALL(Dim_Date),
        Dim_Date[fy_year] = MAX(Dim_Date[fy_year])
            && Dim_Date[fy_quarter] = MAX(Dim_Date[fy_quarter])
            && Dim_Date[date] <= MAX(Dim_Date[date])
    )
)
```

**Notes:** Uses `fy_quarter` from `Dim_Date` rather than `DATESQTD`, which follows calendar quarter boundaries. This ensures the measure respects the July–June fiscal year structure.

---

### MTD Complaints ⚡

**Purpose:** Month-to-date complaint count.

```dax
MTD Complaints =
TOTALMTD(
    [Total Complaints],
    Dim_Date[date]
)
```

---

### Complaints vs Prior Year ⚡

**Purpose:** Total complaints for the same period in the prior calendar year. Used as the denominator in YoY calculations.

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

```dax
YoY Change =
[Total Complaints] - [Complaints vs Prior Year]
```

**Format string:** `+#,##0;-#,##0;0` (displays sign for positive values)

---

### YoY Change %

**Purpose:** Percentage year-over-year variance.

```dax
YoY Change % =
DIVIDE(
    [YoY Change],
    [Complaints vs Prior Year],
    0
)
```

**Format string:** `+0.0%;-0.0%;0%`

**Notes:** Returns blank rather than 0 when prior year data does not exist, preventing misleading 100% change values for new date ranges with no prior year comparison.

---

## Resolution Performance

Resolution measures filter to closed complaints with a positive `resolution_hours` value to exclude open complaints and records where resolution duration could not be calculated.

---

### Avg Resolution Hours

**Purpose:** Average resolution duration in hours across closed complaints.

```dax
Avg Resolution Hours =
AVERAGEX(
    FILTER(
        Fact_Service_Request,
        Fact_Service_Request[resolution_hours] > 0
            && Fact_Service_Request[is_overdue] <> BLANK()
    ),
    Fact_Service_Request[resolution_hours]
)
```

**Notes:** `sum_resolution_hours` and `count_resolution_hours` in the aggregation tables enable this measure to resolve from Import-mode agg tables when the filter context matches the agg grain. `AVERAGEX` can be computed as `sum_resolution_hours / count_resolution_hours` from the pre-aggregated values.

---

### Avg Resolution Days

**Purpose:** Average resolution duration converted from hours to days.

```dax
Avg Resolution Days =
DIVIDE(
    [Avg Resolution Hours],
    24,
    0
)
```

**Format string:** `0.0 "days"`

---

### Avg Resolution Days by Close Date

**Purpose:** Average resolution performance measured against the closed date rather than the created date. Used to evaluate agency performance in a given reporting period based on when work was completed.

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

**Notes:** Activates the inactive `date_key_closed` → `Dim_Date[date_key]` relationship. This changes the date filter context so that `Dim_Date` slicers filter by close date rather than created date for this measure only.

---

### Median Resolution Hours

**Purpose:** Median resolution duration in hours. More robust than average for skewed distributions where a small number of extremely long-running complaints inflate the mean.

```dax
Median Resolution Hours =
MEDIANX(
    FILTER(
        Fact_Service_Request,
        Fact_Service_Request[resolution_hours] > 0
    ),
    Fact_Service_Request[resolution_hours]
)
```

**Notes:** `MEDIANX` always resolves via DirectQuery — there is no aggregation table equivalent for median. Expect higher query latency for this measure on large filtered datasets.

---

### Avg Monthly Complaints

**Purpose:** Average monthly complaint volume across all months in the current filter context. Used as a benchmark line on trend visuals.

```dax
Avg Monthly Complaints =
AVERAGEX(
    VALUES(Dim_Date[cal_year_month_key]),
    [Total Complaints]
)
```

**Notes:** `VALUES(Dim_Date[cal_year_month_key])` iterates over each distinct month in the current filter context, computing `Total Complaints` for each and averaging the results. This produces a stable monthly average regardless of how many months are selected.

---

## Aggregation Coverage Summary

| Measure | Agg Table | Grain |
|---|---|---|
| Total Complaints | `gold_agg_date_borough`, `gold_agg_date_complaint` | date × location or complaint |
| Overdue Complaints | `gold_agg_date_borough`, `gold_agg_date_complaint` | date × location or complaint |
| YTD / MTD / QTD variants | `gold_agg_date_borough`, `gold_agg_date_complaint` | date × location or complaint |
| Complaints vs Prior Year | `gold_agg_date_borough`, `gold_agg_date_complaint` | date × location or complaint |
| Avg Resolution Hours / Days | `gold_agg_date_borough`, `gold_agg_date_complaint` | via sum + count columns |
| Open / Closed Complaints | DirectQuery only | status not in agg grain |
| Median Resolution Hours | DirectQuery only | no agg equivalent |
| Complaints Last 30 Days | DirectQuery only | rolling window not pre-aggregated |

---

## Validation Notes

All measures were validated using DAX Studio with **Query Plan** and **Server Timings** traces enabled. Aggregation hits are confirmed by the presence of `VertiPaq SE` cache reads in the query plan rather than `DirectQuery` engine calls. Measures listed as DirectQuery-only above were verified to fall back correctly and perform within acceptable latency thresholds for their intended visual usage.
