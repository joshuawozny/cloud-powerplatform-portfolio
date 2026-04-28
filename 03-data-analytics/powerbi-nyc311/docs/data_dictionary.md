# Data Dictionary — NYC311 Data Platform

This document covers the full column inventory across two layers: the source dataset as received from the NYC Open Data API, and the Gold layer tables consumed by the Power BI semantic model. Transformation notes explain how source columns map to Gold layer outputs and where columns are derived, renamed, or excluded.

---

## Source Dataset — NYC Open Data 311 (`erm2-nwe9`)

**API Endpoint:** `https://data.cityofnewyork.us/resource/erm2-nwe9.json`
**Grain:** One row per 311 service request complaint
**Key:** `unique_key`

### Temporal Columns

| Column | Type | Description | Used In Gold |
|--------|------|-------------|--------------|
| `created_date` | datetime | Date and time the complaint was submitted | ✅ `Fact_Service_Request.created_date`, `date_key_created` |
| `closed_date` | datetime | Date and time the complaint was closed | ✅ `date_key_closed` |
| `resolution_action_updated_date` | datetime | Date and time of the most recent resolution action update | ✅ `date_key_updated` |

**Note:** A due date (`date_key_due`) for SLA tracking is derived at the Gold layer based on complaint type SLA rules rather than sourced directly from the API.

### Service & Agency Columns

| Column | Type | Description | Used In Gold |
|--------|------|-------------|--------------|
| `unique_key` | int64 | Unique complaint record identifier | ✅ `Fact_Service_Request.unique_key` |
| `agency` | text | Agency code (e.g., `HPD`, `DOT`, `NYPD`) | ✅ `Dim_Agency.agency_code` |
| `agency_name` | text | Full agency name | ✅ `Dim_Agency.agency_name` |
| `complaint_type` | text | Primary complaint category | ✅ `Dim_Complaint.complaint_type` |
| `descriptor` | text | Complaint sub-type detail | ✅ `Dim_Complaint.descriptor` |
| `descriptor_2` | text | Secondary descriptor (sparse) | ❌ Excluded — low population, overlaps with `descriptor` |
| `status` | text | Complaint status (`Open`, `Closed`, `Pending`, etc.) | ✅ `Dim_Status.status` |
| `resolution_description` | text | Text description of the resolution action taken | ✅ `Fact_Service_Request.resolution_description` |
| `open_data_channel_type` | text | Submission channel (`Phone`, `Online`, `Mobile`, `Unknown`) | ✅ `Fact_Service_Request.source` |

### Location Columns

| Column | Type | Description | Used In Gold |
|--------|------|-------------|--------------|
| `borough` | text | NYC borough (`MANHATTAN`, `BROOKLYN`, `QUEENS`, `BRONX`, `STATEN ISLAND`) | ✅ `Dim_Location.borough` |
| `city` | text | City name (generally matches borough for NYC complaints) | ✅ `Dim_Location.city` |
| `community_board` | text | Community board district identifier | ❌ Excluded — granularity below current model scope |
| `council_district` | int64 | City council district number | ❌ Excluded — granularity below current model scope |
| `police_precinct` | text | Responsible NYPD precinct | ❌ Excluded — granularity below current model scope |
| `latitude` | double | Geographic latitude of incident | ✅ `Dim_Location.latitude` |
| `longitude` | double | Geographic longitude of incident | ✅ `Dim_Location.longitude` |
| `incident_address` | text | Street address of the complaint | ❌ Excluded — high cardinality, not used in model |
| `street_name` | text | Street name | ❌ Excluded |
| `cross_street_1` | text | First cross street | ❌ Excluded |
| `cross_street_2` | text | Second cross street | ❌ Excluded |
| `intersection_street_1` | text | Intersection street 1 | ❌ Excluded |
| `intersection_street_2` | text | Intersection street 2 | ❌ Excluded |
| `incident_zip` | int64 | ZIP code of incident | ❌ Excluded — not used in current model scope |
| `address_type` | text | Address type classification (`ADDRESS`, `INTERSECTION`, etc.) | ❌ Excluded |
| `location_type` | text | Type of location (`Street`, `Park`, `Building`, etc.) | ❌ Excluded |
| `bbl` | int64 | Borough-Block-Lot number for property identification | ❌ Excluded |
| `x_coordinate_state_plane` | int64 | X coordinate in NY State Plane system | ❌ Excluded — latitude/longitude used instead |
| `y_coordinate_state_plane` | int64 | Y coordinate in NY State Plane system | ❌ Excluded |
| `location.type` | text | GeoJSON geometry type | ❌ Excluded |
| `location.coordinates.0` | text | GeoJSON longitude coordinate | ❌ Excluded — raw coordinate string, superseded by `longitude` |
| `location.coordinates.1` | text | GeoJSON latitude coordinate | ❌ Excluded — raw coordinate string, superseded by `latitude` |

### Facility & Special-Use Columns

| Column | Type | Description | Used In Gold |
|--------|------|-------------|--------------|
| `facility_type` | text | Type of facility associated with complaint | ❌ Excluded — sparse, complaint-type specific |
| `landmark` | text | Landmark name if applicable | ❌ Excluded — sparse |
| `park_facility_name` | text | Name of park facility if complaint is park-related | ❌ Excluded |
| `park_borough` | text | Borough of park facility | ❌ Excluded — covered by `borough` |
| `vehicle_type` | text | Vehicle type for vehicle-related complaints | ❌ Excluded — sparse, complaint-type specific |
| `bridge_highway_name` | text | Bridge or highway name | ❌ Excluded |
| `bridge_highway_direction` | text | Travel direction on bridge/highway | ❌ Excluded |
| `road_ramp` | text | Road ramp indicator | ❌ Excluded |
| `bridge_highway_segment` | text | Segment identifier for bridge/highway complaints | ❌ Excluded |
| `taxi_pick_up_location` | text | Pick-up location for taxi-related complaints | ❌ Excluded |

### Computed Region Columns

| Column | Type | Description | Used In Gold |
|--------|------|-------------|--------------|
| `:@computed_region_f5dn_yrer` | int64 | Socrata computed region — school districts | ❌ Excluded |
| `:@computed_region_yeji_bk3q` | int64 | Socrata computed region — police precincts | ❌ Excluded |
| `:@computed_region_sbqj_enih` | int64 | Socrata computed region — city council districts | ❌ Excluded |
| `:@computed_region_92fq_4b7q` | int64 | Socrata computed region — community districts | ❌ Excluded |

---

## Gold Layer — Fact Table

### `Fact_Service_Request`

**Storage mode:** DirectQuery
**Grain:** One row per 311 service request complaint
**Key:** `unique_key`

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| `unique_key` | int64 | `unique_key` | Source system complaint identifier |
| `agency_key` | int64 | Derived | Surrogate key → `Dim_Agency` |
| `complaint_key` | int64 | Derived | Surrogate key → `Dim_Complaint` |
| `location_key` | int64 | Derived | Surrogate key → `Dim_Location` |
| `status_key` | int64 | Derived | Surrogate key → `Dim_Status` |
| `date_key_created` | int64 | `created_date` | FK → `Dim_Date` (active relationship) |
| `date_key_closed` | int64 | `closed_date` | FK → `Dim_Date` (inactive — use `USERELATIONSHIP`) |
| `date_key_due` | int64 | Derived | FK → `Dim_Date` (inactive — SLA due date, derived from complaint type SLA rules) |
| `date_key_updated` | int64 | `resolution_action_updated_date` | FK → `Dim_Date` (inactive) |
| `created_date` | datetime | `created_date` | Raw created datetime retained for time-based calculations |
| `resolution_hours` | decimal | Derived | Hours between `created_date` and `closed_date`. Null for open complaints |
| `is_overdue` | boolean | Derived | TRUE if `closed_date` > `date_key_due` or complaint is open past due date |
| `resolution_description` | text | `resolution_description` | Closing resolution text |
| `source` | text | `open_data_channel_type` | Submission channel renamed for clarity |
| `load_date` | datetime | Pipeline | Timestamp of the pipeline run that loaded the record — used for data lineage |

---

## Gold Layer — Dimension Tables

### `Dim_Date`

**Storage mode:** Dual
**Grain:** One row per calendar date
**Key:** `date_key`

| Column | Type | Description |
|--------|------|-------------|
| `date_key` | int64 | Surrogate key in `YYYYMMDD` format |
| `date` | date | Calendar date value |
| `cal_year` | int64 | Calendar year (e.g., 2024) |
| `cal_quarter` | int64 | Calendar quarter number (1–4) |
| `cal_quarter_label` | text | Calendar quarter label (e.g., `Q1 2024`) |
| `cal_month` | int64 | Calendar month number (1–12) |
| `cal_month_name` | text | Full month name (e.g., `January`) |
| `cal_month_short` | text | Abbreviated month name (e.g., `Jan`) |
| `cal_month_year_label` | text | Month-year label (e.g., `Jan 2024`) |
| `cal_year_month_key` | int64 | Month surrogate key in `YYYYMM` format |
| `cal_year_month_label` | text | Year-month display label |
| `cal_year_quarter_key` | int64 | Quarter surrogate key in `YYYYQ` format |
| `cal_year_quarter_label` | text | Year-quarter display label |
| `cal_week` | int64 | ISO week number (1–53) |
| `cal_day_of_week` | int64 | Day of week number (1 = Sunday) |
| `cal_day_name` | text | Full day name (e.g., `Monday`) |
| `cal_day_short` | text | Abbreviated day name (e.g., `Mon`) |
| `day_of_month` | int64 | Day number within the month (1–31) |
| `day_of_year` | int64 | Day number within the year (1–366) |
| `fy_year` | int64 | NYC fiscal year (July 1 – June 30). FY2024 = July 2023 – June 2024 |
| `fy_year_key` | int64 | Fiscal year surrogate key |
| `fy_label` | text | Fiscal year display label (e.g., `FY2024`) |
| `fy_quarter` | int64 | Fiscal quarter number (1–4, starting July) |
| `fy_quarter_label` | text | Fiscal quarter display label |
| `fy_month` | int64 | Fiscal month number (1 = July) |
| `fy_week` | int64 | Fiscal week number |

---

### `Dim_Agency`

**Storage mode:** Dual
**Grain:** One row per agency
**Key:** `agency_key`

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| `agency_key` | int64 | Derived | Surrogate key |
| `agency_code` | text | `agency` | Short agency code (e.g., `HPD`, `DOT`) |
| `agency_name` | text | `agency_name` | Full agency name (e.g., `Department of Transportation`) |

---

### `Dim_Location`

**Storage mode:** Dual
**Grain:** One row per unique borough/city/coordinate combination
**Key:** `location_key`

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| `location_key` | int64 | Derived | Surrogate key |
| `borough` | text | `borough` | NYC borough name, standardized to title case |
| `city` | text | `city` | City name |
| `latitude` | double | `latitude` | Geographic latitude |
| `longitude` | double | `longitude` | Geographic longitude |

**Hierarchy:** Borough Hierarchy — `borough` → `city` → `location_key`

---

### `Dim_Complaint`

**Storage mode:** Dual
**Grain:** One row per complaint type / descriptor combination
**Key:** `complaint_key`

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| `complaint_key` | int64 | Derived | Surrogate key |
| `complaint_type` | text | `complaint_type` | Primary complaint category |
| `descriptor` | text | `descriptor` | Complaint sub-type detail |

**Hierarchy:** Complaint Type Hierarchy — `complaint_type` → `descriptor`

---

### `Dim_Status`

**Storage mode:** Dual
**Grain:** One row per status value
**Key:** `status_key`

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| `status_key` | int64 | Derived | Surrogate key |
| `status` | text | `status` | Complaint status (`Open`, `Closed`, `Pending`) |

---

## Gold Layer — Aggregation Tables

### `gold_agg_date_borough`

**Storage mode:** Import
**Grain:** One row per `date_key_created` × `location_key` combination
**Supports:** Borough and date-level report visuals — resolves `Total Complaints`, `Overdue Complaints`, and resolution average measures from Import cache rather than DirectQuery

| Column | Type | Description |
|--------|------|-------------|
| `date_key_created` | int64 | FK → `Dim_Date[date_key]` |
| `location_key` | int64 | FK → `Dim_Location[location_key]` |
| `total_complaints` | int64 | Pre-aggregated complaint count |
| `open_count` | int64 | Count of open status complaints |
| `overdue_count` | int64 | Count of complaints where `is_overdue = TRUE` |
| `sum_resolution_hours` | decimal | Sum of resolution hours (used with `count_resolution_hours` to compute average) |
| `count_resolution_hours` | int64 | Count of records with a valid `resolution_hours` value |

---

### `gold_agg_date_complaint`

**Storage mode:** Import
**Grain:** One row per `date_key_created` × `complaint_key` combination
**Supports:** Complaint type and date-level report visuals

| Column | Type | Description |
|--------|------|-------------|
| `date_key_created` | int64 | FK → `Dim_Date[date_key]` |
| `complaint_key` | int64 | FK → `Dim_Complaint[complaint_key]` |
| `total_complaints` | int64 | Pre-aggregated complaint count |
| `overdue_count` | int64 | Count of complaints where `is_overdue = TRUE` |
| `sum_resolution_hours` | decimal | Sum of resolution hours |
| `count_resolution_hours` | int64 | Count of records with a valid `resolution_hours` value |

---

## Transformation Summary

| Source Column | Gold Destination | Transformation |
|---|---|---|
| `unique_key` | `Fact_Service_Request.unique_key` | Direct carry-through |
| `created_date` | `date_key_created`, `created_date` | Integer key generated; raw datetime retained |
| `closed_date` | `date_key_closed` | Integer key generated |
| `resolution_action_updated_date` | `date_key_updated` | Integer key generated |
| `agency` / `agency_name` | `Dim_Agency` | Deduped; surrogate key assigned |
| `complaint_type` / `descriptor` | `Dim_Complaint` | Deduped; surrogate key assigned; hierarchy built |
| `borough` / `city` / `latitude` / `longitude` | `Dim_Location` | Deduped; surrogate key assigned; hierarchy built |
| `status` | `Dim_Status` | Deduped; surrogate key assigned |
| `open_data_channel_type` | `Fact_Service_Request.source` | Renamed for clarity |
| `closed_date` - `created_date` | `Fact_Service_Request.resolution_hours` | Calculated in hours at Gold layer; null for open complaints |
| SLA rule logic | `Fact_Service_Request.is_overdue` | Boolean derived from complaint type SLA thresholds vs. `closed_date` or current date |
| Pipeline metadata | `Fact_Service_Request.load_date` | Injected by notebook at write time |
| Multiple address fields | Excluded | High cardinality, not used in current model scope |
| Computed region columns | Excluded | Socrata-internal geographic metadata |
