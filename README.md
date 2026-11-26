# HR Analytics Dashboard

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-Data_Analysis_Expressions-0078D4?style=for-the-badge)](https://learn.microsoft.com/en-us/dax/)
[![Power Query](https://img.shields.io/badge/Power_Query-M_Language-2CA5E0?style=for-the-badge)](https://learn.microsoft.com/en-us/power-query/)

An interactive **Human Resources Analytics Dashboard** developed in **Power BI Desktop** to present descriptive observations of employee turnover, demographic distributions, compensation tiers, and workforce characteristics.

---

## 1. Project Overview

This project provides an exploratory business intelligence dashboard built to examine historical HR records. It allows users to review aggregate workforce figures, inspect demographic and departmental breakdowns, and observe attrition counts across various recorded attributes.

The report consolidates demographic information, salary tiers, departmental headcounts, job roles, and employee-reported satisfaction ratings into a single analytical canvas.

---

## 2. Dashboard Preview

![HR Analytics Dashboard](assets/HR%20ANALYTICS%20DASHBOARD.png)

---

## 3. Key Performance Indicators (KPIs)

The top KPI ribbon displays high-level workforce metrics calculated within the report:

| KPI Card | Displayed Value | Verified Calculation / Definition | Metric Description |
| :--- | :---: | :--- | :--- |
| **Total Employees** | `1.417K` | `Count of EmpID` (1,417 rows in model; 1,403 distinct employee IDs) | Total employee records loaded into the reporting model. |
| **Active Employees** | `1.186K` | `Count of EmployeeCount` with visual filter `Attrition = "No"` (1,186 rows) | Count of active employee records currently retained in the dataset. |
| **Attrition Count** | `231` | `Sum of AttritionCount` (`Attrition = "Yes"`) | Total count of recorded departures in the dataset. |
| **Attrition Rate %** | `16.3%` | `[Attrition Rate %] = DIVIDE(SUM(AttritionCount), SUM(EmployeeCount), 0)` | Overall departure percentage across loaded records (231 / 1,417). |
| **Average Age** | `36.94` | `Average of Age` | Arithmetic mean of recorded employee ages. |
| **Avg Experience** | `7.04` | `Average of YearsatCompany` (7.04 years) | Arithmetic mean of recorded tenure at the company (evaluates `YearsatCompany`). |

---

## 4. Dashboard Features & Visual Layout

The report page includes interactive slicers and six analytical visuals:

### Interactive Slicers
- **Age Group Slicer (Top Ribbon):** Filters data across five categorical age brackets: `18-25`, `26-35`, `36-45`, `46-55`, and `55+`.
- **Department Slicer (Left Panel):** Filters data by organizational unit: `Administration`, `Finance`, `Human Resources`, `IT`, `Marketing`, `Operations`, and `Sales`.

### Visual Analytics
1. **Attrition by Department (Donut Chart):**
   - *Plotted Metric:* `Sum of AttritionCount` categorized by `Department`.
   - *Observed Breakdown:* Administration (84 departures, 36.4%), Sales (52 departures, 22.5%), Operations (43 departures, 18.6%), IT (30 departures, 13.0%), Marketing (10 departures, 4.3%), Human Resources (9 departures, 3.9%), and Finance (3 departures, 1.3%).
   - *Design Focus:* Illustrates the distribution of total company departures across departments.

2. **Attrition by Salary Slab (Clustered Bar Chart):**
   - *Plotted Metric:* Headcount by `SalarySlab` grouped by `Attrition` (`No` vs. `Yes`).
   - *Observed Values:* `0-3 LPA` (299 No, 61 Yes), `3-6 LPA` (293 No, 49 Yes), `6-10 LPA` (399 No, 73 Yes), and `10+ LPA` (195 No, 48 Yes).
   - *Design Focus:* Displays the distribution of active and departed employees across defined compensation brackets.

3. **Attrition by Job Role & Job Satisfaction (Matrix Table):**
   - *Plotted Metric:* Cross-tabulation of `Sum of AttritionCount` across nine `JobRole` categories and four numerical `JobSatisfaction` levels (1 to 4).
   - *Formatting:* Conditional linear gradient color scale applied to cell backgrounds.
   - *Design Focus:* Displays departure counts across job roles alongside recorded survey satisfaction ratings.

4. **Attrition Rate by Age Group (Clustered Bar Chart):**
   - *Plotted Metric:* DAX Measure `[Attrition Rate by Age Group]` plotted by `AgeGroup`.
   - *Observed Values:* `18-25` (37.6%), `26-35` (18.9%), `55+` (18.2%), `46-55` (12.2%), and `36-45` (9.2%).
   - *Design Focus:* Displays the calculated attrition percentage within each age category.

5. **Attrition Trend by Experience (Area Chart):**
   - *Plotted Metric:* `Sum of AttritionCount` plotted across `TotalExperience(Years)`.
   - *Observed Values:* Departures span from 0 to 40 years of total experience, with the single highest count observed at 1 year of experience (40 departures).
   - *Design Focus:* Plots departure counts across discrete values of total professional experience.

6. **Department-Wise Employee Count (Funnel Chart):**
   - *Plotted Metric:* `Count of EmpID` categorized by `Department`.
   - *Observed Values:* Operations (512), Administration (406), Sales (207), IT (166), Marketing (64), Human Resources (39), Finance (23).
   - *Design Focus:* Shows comparative employee headcounts by department.

---

## 5. Dataset Information

The dashboard connects to a flat CSV file:

- **Filename:** `HR_Analytics-4.csv`
- **Initial Raw Records:** 1,480 rows, 37 columns.
- **Data Transformations Applied:**
  - **Null Value Filtering:** 61 rows were removed because they contained blank/null values in the `YearsWithCurrManager` field.
  - **Duplicate Removal:** 2 exact duplicate rows (identical across all 37 columns) were removed via `Table.Distinct`.
  - **Loaded Model Records:** **1,417 rows** in the final semantic model.
  - **Distinct Identifier Count:** 1,403 unique `EmpID` values exist in the loaded model. 14 `EmpID` identifiers appear twice where minor non-key field variations existed across rows.

---

## 6. Data Preparation (Power Query ETL)

The data transformation steps recorded in the table's M partition definition executed in the following order:

```powerquery
// Excerpt from table definition: HR_Analytics-4 (2).tmdl
let
    // 1. Ingest raw CSV (path must be configured to the local repository file location)
    Source = Csv.Document(File.Contents("<Path-To-Repository>\HR_Analytics-4.csv"), [Delimiter=",", Columns=37, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    
    // 2. Promote first row to column headers
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    
    // 3. Transform column data types across all 37 attributes
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {...}),
    
    // 4. Sort ascending by YearsWithCurrManager (places null/empty values at the top)
    #"Sorted Rows" = Table.Sort(#"Changed Type", {{"YearsWithCurrManager", Order.Ascending}}),
    
    // 5. Remove the first 61 rows (removes null YearsWithCurrManager records)
    #"Removed Top Rows" = Table.Skip(#"Sorted Rows", 61),
    
    // 6. Deduplicate exact matching rows across all 37 columns
    #"Removed Duplicates" = Table.Distinct(#"Removed Top Rows"),
    
    // 7. Replace text in BusinessTravel ("Travel_Rarely" -> "TravelRarely")
    #"Replaced Value" = Table.ReplaceValue(#"Removed Duplicates", "Travel_Rarely", "TravelRarely", Replacer.ReplaceText, {"BusinessTravel"}),
    
    // 8. Filter rows step
    #"Filtered Rows" = Table.SelectRows(#"Replaced Value", each true),
    
    // 9. Re-verify column data types
    #"Changed Type1" = Table.TransformColumnTypes(#"Filtered Rows", {...}),
    
    // 10. Add conditional indicator column for attrition (1 if Yes, 0 if No)
    #"Added Conditional Column" = Table.AddColumn(#"Changed Type1", "AttntonCount", each if [Attrition] = "Yes" then 1 else 0),
    
    // 11. Rename conditional column to standard naming
    #"Renamed Columns" = Table.RenameColumns(#"Added Conditional Column", {{"AttntonCount", "AttritionCount"}})
in
    #"Renamed Columns"
```

*Note: Column type lists in steps 3 and 9 are abbreviated above for clarity. Full definitions are accessible in the semantic model files.*

---

## 7. DAX Measures

The semantic model contains two explicit DAX measures:

### 1. Overall Attrition Rate %
```dax
measure 'Attrition Rate %' = 
DIVIDE(
    SUM('HR_Analytics-4 (2)'[AttritionCount]),
    SUM('HR_Analytics-4 (2)'[EmployeeCount]),
    0
)
```
*Calculates the overall departure percentage across filtered rows, formatted as `0.0%`.*

### 2. Attrition Rate by Age Group
```dax
measure 'Attrition Rate by Age Group' = 
DIVIDE(
    CALCULATE(
        COUNTROWS('HR_Analytics-4 (2)'),
        'HR_Analytics-4 (2)'[Attrition] = "Yes"
    ),
    COUNTROWS('HR_Analytics-4 (2)'),
    0
)
```
*Evaluates the within-context attrition percentage, formatted as `0.0%`.*

---

## 8. Tools & Technologies

- **Business Intelligence:** Microsoft Power BI Desktop
- **Data Modeling:** Power BI Project (`.pbip`), Tabular Model Definition Language (TMDL)
- **ETL:** Power Query (M Language)
- **Calculations:** DAX (Data Analysis Expressions)
- **Data Storage:** Flat CSV (`UTF-8`)

---

## 9. Project Structure

```text
HR-Analytics-PowerBI-Dashboard/
│
├── .gitignore                                      # Excludes Power BI local cache and user settings
├── README.md                                       # Project documentation
├── HR_Analytics-4.csv                              # Source dataset
│
├── HR ANALYTICS DASHBOARD.pbip                     # Power BI Project root file
├── HR ANALYTICS DASHBOARD.pbix                     # Compiled Power BI report binary
│
├── HR ANALYTICS DASHBOARD.Report/                  # Report visuals and canvas definition (PBIR)
│   ├── definition.pbir                             # Pointer linking report to semantic model
│   └── definition/
│       ├── report.json                             # Theme, canvas, and report-level properties
│       └── pages/
│           ├── pages.json                          # Page metadata
│           └── a168e05dc08419019b3c/               # Page 1 folder
│               ├── page.json                       # Canvas layout dimensions (1280x720)
│               └── visuals/                        # Visual container schemas for all 21 canvas items
│
├── HR ANALYTICS DASHBOARD.SemanticModel/           # Semantic model definitions (TMDL)
│   ├── definition.pbism                            # Model settings
│   ├── diagramLayout.json                          # Diagram view coordinates
│   └── definition/
│       ├── model.tmdl                              # Model culture and global settings
│       └── tables/
│           └── HR_Analytics-4 (2).tmdl             # Schema, M query, column properties, and DAX measures
│
├── assets/                                         # Media assets
│   ├── HR ANALYTICS DASHBOARD.png                  # Dashboard screenshot preview
│   ├── HR ANALYTICS DASHBOARD.pdf                  # Exported PDF version of the dashboard
│   └── HR ANALYTICS DASHBOARD.mp4                  # Recorded video demonstration
│
└── Icons/                                          # Visual icon images used in KPI cards
    ├── icons8-age-64.png
    ├── icons8-checked-user-male-64.png
    ├── icons8-employees-64.png
    ├── icons8-experience-64.png
    ├── icons8-minus-64.png
    └── icons8-percentage-64.png
```

---

## 10. How to Run the Project

### Prerequisites
- [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/) (version supporting Power BI Project `.pbip` files).

### Setup Instructions
1. **Clone the Repository:**
   ```bash
   git clone https://github.com/HAITHM-ALAKHALI/HR-Analytics-PowerBI-Dashboard.git
   cd HR-Analytics-PowerBI-Dashboard
   ```

2. **Open the Project:**
   - Double-click `HR ANALYTICS DASHBOARD.pbip` (or `HR ANALYTICS DASHBOARD.pbix`) to open the report in Power BI Desktop.

3. **Configure the Data Source Path:**
   - Because file paths in Power Query are absolute on initial import, update the data source location to match your local clone:
     1. In Power BI Desktop, click **Home** > **Transform Data** > **Data Source Settings** (or edit the **Source** step in the Power Query Editor).
     2. Update the file path to point to your local copy of `HR_Analytics-4.csv`:
        ```powerquery
        = Csv.Document(File.Contents("<Path-To-Your-Local-Repository>\HR_Analytics-4.csv"), [Delimiter=",", Columns=37, Encoding=65001, QuoteStyle=QuoteStyle.None])
        ```
     3. Click **Close & Apply** to refresh the dataset.

4. **Explore the Dashboard:**
   - Interact with the **Age Group** buttons and **Department** panel to view filtered values across all visuals.

---

## 11. Key Descriptive Observations

The dashboard calculations yield the following verified observations from the loaded data:

1. **Age Cohort Distribution:**
   - Within the `18-25` age category, the observed attrition rate is **37.6%** (44 departures out of 117 employees).
   - The lowest within-bracket rate occurs in the `36-45` cohort at **9.2%** (41 departures out of 446 employees).
2. **Departmental Distribution:**
   - The highest departmental attrition rates in the dataset occur in **Sales** at **25.1%** (52 departures / 207 employees), **Human Resources** at **23.1%** (9 departures / 39 employees), and **Administration** at **20.7%** (84 departures / 406 employees).
   - **Operations** accounts for the largest headcount in the dataset (512 employees) and records an attrition rate of **8.4%** (43 departures).
3. **Job Role Departure Counts:**
   - **Laboratory Technicians** (59 departures) and **Sales Executives** (56 departures) account for the highest individual departure headcounts, totaling **115 of the 231 recorded departures** (49.8%).
4. **Experience Distribution:**
   - The area chart distribution shows the highest concentration of departure counts among employees with **1 year of total experience** (40 departures), tapering down as total experience increases.

---

## 12. Project Limitations & Analytical Considerations

- **Data Reduction During Cleaning:** Removing 61 records with blank `YearsWithCurrManager` values excluded 8 departure records and 53 active records from the dashboard population.
- **Identifier Granularity:** Deduplication was performed across all 37 attributes; 14 `EmpID` identifiers appear twice due to differences in non-key fields, yielding 1,417 rows for 1,403 distinct employee IDs.
- **Metric Scope:**
  - The KPI card labeled **"Avg Experience"** aggregates the `YearsatCompany` column (mean: 7.04 years of company tenure) rather than total professional experience (`TotalExperience(Years)`, mean: 11.30 years).
  - The **"Attrition by Department"** Donut Chart shows the proportion of total departures by department rather than department-specific turnover rates.
- **Data Model Architecture:** The report imports a single denormalized flat table rather than a multi-table dimensional Star Schema.

---

## 13. Author

**Haithm Abdulwali Al-Akhali**  
Power BI & Data Analytics Enthusiast  
GitHub: [@HAITHM-ALAKHALI](https://github.com/HAITHM-ALAKHALI)
