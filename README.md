# 🏥 Patient Diagnostic Dashboard
### Databuzz Ltd — Power BI Contest | March 2026

---

## 📌 Overview

A 3-page interactive Power BI dashboard built on a synthetic healthcare dataset of **10,000 patient visits** across **3,678 unique patients** (Jan 2024 – Feb 2026). The dashboard integrates demographics, diagnostic indicators, chronic condition markers, blood test results, medication usage, and visit behaviour to support faster and clearer clinical decision-making.

**Contest Theme:** Patient Diagnostic Dashboard  
**Submitted by:** Gyanankur Baruah  
**Submission Date:** March 2026  

---

## 📂 Repository Structure

```
├── Databuzz Gyanankur march 2026.pbix   ← Power BI report file
├── Databuzz March 2026.pdf              ← PDF export of all pages
├── README.md                            ← This file
├── data/
│   └── Patient Diagnostic Dashboard.xlsx
├── images/
│   ├── Screenshot 2026-03-23 022214.png ← Page 1: Executive Overview
│   ├── Screenshot 2026-03-23 022351.png ← Page 2: Chronic Monitor
│   └── Screenshot 2026-03-23 022454.png ← Page 3: Patient Detail
├── page background/
│   └── Screenshot 2026-03-23 021351.png
├── theme/
│   └── 21march.json                     ← Custom violet Power BI theme
├── AI_Prompt.docx.pdf
├── Dashboard Requirements.docx.pdf
└── JudgmentCriteria.docx.pdf
```

---

## 📊 Dashboard Pages

### Page 1 — Executive Overview

![Executive Overview](images/Screenshot%202026-03-23%20022214.png)

High-level summary of all patient activity and key clinical indicators.

**KPIs displayed:**
| Metric | Value |
|---|---|
| Total Patients | 3,678 |
| High Risk % | 58.6% |
| Admission Rate | 65.8% |
| Avg Diabetes (2hr Post-Meal) | 162 mg/dL |
| Total Visits | 10,000 |

**Visuals included:**
- Donut chart — Risk Level distribution (High / Medium / Low)
- Donut chart — Gender split (Male / Female)
- Horizontal bar — Total Visits by Fever Type
- Line chart — Visit trend by Year and Month (Jan 2024 – Feb 2026)
- Bar chart — Patient distribution by Weight
- Dark / Light mode toggle (bookmarks)
- Slicers: Gender, Risk Level, Age, Fever Type, Location, Visit Date range

---

### Page 2 — Chronic Condition Monitor & Risk Analysis

![Chronic Monitor](images/Screenshot%202026-03-23%20022351.png)

Deep-dive into chronic conditions, risk patterns, and admission behaviour.

**KPIs displayed:**
| Metric | Value |
|---|---|
| Diabetic % | 52% |
| Hypertension % | 77% |
| Admission Rate | 65.8% |
| Avg Diabetes | 162 mg/dL |
| Avg Temperature | 39°C |

**Visuals included:**
- Clustered bar — Total Visits by Fever Type
- Horizontal bar — Admission Rate by Fever Type (Typhoid 68% → COVID 64%)
- Donut — Diabetic Count by Diabetes Category (Diabetic / Pre-Diabetic / Normal)
- Scatter — Correlation: Patient Age vs Post-Prandial Glucose Levels, coloured by Risk Level
- Table — Visit Frequency (Single Visit: 826 → Frequent 5+: 432)
- Footer card — Avg gap: ~177 days | Max visits per patient: 11
- Dark / Light mode toggle
- Slicers: Gender, Risk Level, Age, Fever Type, Location, Visit Date range

---

### Page 3 — Patient Detail (Drill-Through View)

![Patient Detail](images/Screenshot%202026-03-23%20022454.png)

Individual patient-level deep-dive. Activated via drill-through from Page 2 or via Patient_ID slicer.

**Sections:**
- **Patient Profile card** — Patient ID, Gender, Age, Location, Total Visits, Admission History
- **Latest Visit Vitals** — BP Diastolic, BP Systolic, Temperature, Diabetes mg/dL, Visit Date
- **Diagnostic Trend Direction** — BP Systolic (Stable), Diabetes (Improving), Risk Level (High unchanged), Temperature (Worsening)
- **Temperature trend chart** — Avg Temperature by Year, Month, and Risk Level
- **Latest Blood Test Results** — Hemoglobin, WBC, RBC, Platelets, ESR, CRP displayed as KPI cards
- **Prescribed Medications table** — Full medication list filtered by Fever Type (Medication, Dose, Duration, Frequency)
- **Symptom Indicator visual** — Body Pain and Cold cross-analysis
- **Smart Narrative** — Auto-generated patient summary text
- Slicers: Patient_ID, Gender, Age, Fever Type, Location, Visit Date range

---

## 🧱 Data Model

**Schema type:** Star Schema

| Table | Type | Join Key |
|---|---|---|
| `Patient_Symptoms` | Fact | Visit_ID, Patient_ID |
| `Blood_Test_Results` | Dimension | Visit_ID → Fact |
| `Fever_Type_Profile` | Dimension | Fever_Type → Fact |
| `Medication` | Dimension | Fever_Type → Fact |
| `DateTable` | Date dimension | Date → Visit_Date |

**Power Query transformations applied:**
- `BP S/D` column split into `BP_Systolic` and `BP_Diastolic` (integer)
- Medication `Feavertype` mapped to short keys matching Fact table (`Viral Fever` → `Viral`, etc.)
- Calculated columns added: `BP_Category`, `Diabetes_Category`, `Age_Group`, `BMI`, `Visit_Frequency_Label`, `Days_Since_Prev_Visit`
- DateTable created via DAX `CALENDAR()` and marked as Date Table

---

## 📐 DAX Measures — Key Logic

**Risk KPIs**
```dax
High_Risk_Pct = DIVIDE(
    CALCULATE(COUNTROWS(Patient_Symptoms), Patient_Symptoms[Risk_Level] = "High"),
    COUNTROWS(Patient_Symptoms), 0)
```

**BP Category (Calculated Column)**
```dax
BP_Category =
IF(Patient_Symptoms[BP_Systolic] <= 120, "Normal",
IF(Patient_Symptoms[BP_Systolic] <= 130, "Elevated",
"High Stage 1"))
```

**Diabetes Category (Calculated Column)**
```dax
Diabetes_Category =
IF(Patient_Symptoms[Diabetes_mgdL] <= 140, "Normal",
IF(Patient_Symptoms[Diabetes_mgdL] <= 180, "Pre-Diabetic", "Diabetic"))
```

**Visit Gap (Calculated Column)**
```dax
Days_Since_Prev_Visit =
VAR current_patient = Patient_Symptoms[Patient_ID]
VAR current_date = Patient_Symptoms[Visit_Date]
VAR prev_date = CALCULATE(MAX(Patient_Symptoms[Visit_Date]),
    FILTER(Patient_Symptoms,
        Patient_Symptoms[Patient_ID] = current_patient &&
        Patient_Symptoms[Visit_Date] < current_date))
RETURN IF(ISBLANK(prev_date), BLANK(), DATEDIFF(prev_date, current_date, DAY))
```

**Diagnostic Trend**
```dax
Diabetes_Trend_Label =
VAR latest = [Latest_Diabetes]
VAR prior = [Prior_Diabetes]
RETURN
    IF(ISBLANK(prior), "First Visit",
    IF(latest - prior > 5, "▲ Worsening",
    IF(prior - latest > 5, "▼ Improving", "→ Stable")))
```

---

## 🎨 Design System

| Element | Detail |
|---|---|
| Theme | Custom violet — imported via `theme/21march.json` |
| Primary colour | `#7C5CBF` |
| Accent | `#A084DC` |
| High risk | `#C0392B` |
| Medium risk | `#E67E22` |
| Low risk | `#27AE60` |
| Background (light) | `#F5F0FF` |
| Background (dark) | `#1E1B2E` |
| Fonts | Segoe UI |
| Dark mode | Implemented via Power BI Bookmarks toggle on all 3 pages |

---

## ✅ Judgment Criteria Coverage

| Category | Weight | Implementation |
|---|---|---|
| Core Business Requirements | 40% | All 8 KPIs implemented, risk segmentation via DAX, visit gap calculated, current vs prior diagnostic comparison, threshold-based chronic conditions, disclaimer on all pages |
| Data Modelling & Performance | 20% | Star schema, single fact table, DateTable marked, all relationships single-direction, measures use VAR for optimization |
| Visual Accuracy & Interactivity | 20% | Drill-through Page 2 → Page 3, conditional formatting on risk, correct aggregations, no double-counting, tooltips active |
| UX, Design & Readability | 10% | KPI → Trends → Analysis → Detail flow, violet healthcare palette, consistent typography, dark/light mode toggle |
| Add-on Insights & Innovation | 10% | Smart Narrative visual on Page 3, scatter correlation analysis, visit frequency segmentation, symptom burden scoring, LinkedIn engagement |

---

## 📋 Dataset Summary

| Table | Rows | Notes |
|---|---|---|
| Patient_Symptoms | 10,000 | 3,678 unique patients, Jan 2024–Feb 2026 |
| Blood_Test_Results | 10,000 | 1:1 with visits via Visit_ID |
| Fever_Type_Profile | 8 | One row per fever type |
| Medication | 26 | Multiple medications per fever type |

**Key distributions:**
- Fever Types: Viral (1,290) · Malaria (1,278) · Influenza (1,271) · Chikungunya (1,247) · Dengue (1,241) · Bacterial (1,237) · COVID (1,222) · Typhoid (1,214)
- Risk: Medium 35.2% · Low 33.5% · High 31.2%
- Avg Diabetes: 162.5 mg/dL | Avg Temperature: 39.0°C | Avg Systolic BP: 114.6 mmHg
- Admission Rate: 65.8% | Avg visit gap: ~177 days

---

### License

MIT License

---

Created by Gyanankur Baruah
@Gyanankur23

Linkedln:- https://www.linkedin.com/in/gyanankur-baruah-bafc%E2%84%A2-sfc%E2%84%A2-kec%E2%84%A2-cpefpc%E2%84%A2-797205338/

---

*Submitted for the Databuzz Ltd Power BI Contest — March 2026*  
*Built with Power BI Desktop | DAX | Power Query M | Custom JSON Theme*
