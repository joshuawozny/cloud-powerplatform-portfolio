# 🗽 NYC Open Data Power BI Dashboard (PL‑300)

## 📌 Overview
This project analyzes New York City 311 Service Requests using Power BI.  
It demonstrates advanced data modeling, DAX development, and visualization skills aligned with the PL‑300 certification.

**Dataset:** NYC 311 Service Requests  
**Source:** https://data.cityofnewyork.us  
**API Endpoint Example:**  
`https://data.cityofnewyork.us/resource/erm2-nwe9.json?$limit=50000`

The dashboard provides insights into:
- Request volume trends  
- Agency performance  
- Complaint type distribution  
- Geographic patterns  
- Resolution times  

---

## 🧩 Architecture
NYC Open Data API (Socrata) ↓ Power BI Power Query (Transformations) ↓ Star Schema Data Model ↓ DAX Measures (KPIs, Time Intelligence) ↓ Interactive Dashboard (Maps, Drill‑through, Tooltips)

---

## 📊 Data Model

### **Fact Table**
**FactRequests**
- Created Date  
- Closed Date  
- Resolution Hours  
- Complaint Type  
- Agency  
- Borough  
- Latitude / Longitude  

### **Dimension Tables**
- **DimDate** — calendar table  
- **DimAgency** — agency lookup  
- **DimComplaintType** — complaint categories  
- **DimLocation** — borough, coordinates  

---

## 🧠 Key DAX Measures
```DAX
Total Requests = COUNTROWS(FactRequests)

Requests YTD = TOTALYTD([Total Requests], 'DimDate'[Date])

Requests YoY % Change =
VAR CurrentYear = [Total Requests]
VAR PriorYear =
    CALCULATE([Total Requests], SAMEPERIODLASTYEAR('DimDate'[Date]))
RETURN
DIVIDE(CurrentYear - PriorYear, PriorYear)

Avg Resolution Time =
AVERAGEX(FactRequests, FactRequests[ResolutionHours])
```
Full list of measures:
➡️ ./dax/dax-measures.md (create this file when ready)

## 📈 Visuals Included
- KPI cards (Total Requests, Avg Resolution Time, YoY Change)
- Trend line (Requests over time)
- Category breakdown (Complaint Type, Agency)
- Map visual (Borough, coordinates)
- Drill‑through pages (Agency, Complaint Type)
- Tooltips with mini‑cards

## 📂 Folder Structure
```
/pbip
/dax
/screenshots
/docs
```
🔗 Links
- PBIP Files: ./pbip
- DAX Measures: ./dax
- Screenshots: ./screenshots
- NYC Open Data API: https://data.cityofnewyork.us

## 📝 Lessons Learned
(Add your reflections here — data quality issues, modeling decisions, DAX challenges, visualization insights.)

## 🎓 Related Certifications
- PL‑300 — Power BI Data Analyst
- PL‑900 — Power Platform Fundamentals
