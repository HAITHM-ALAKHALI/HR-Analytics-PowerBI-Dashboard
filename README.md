# HR Analytics Dashboard

[![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-Data_Analysis_Expressions-0078D4?style=for-the-badge)](https://learn.microsoft.com/en-us/dax/)
[![Power Query](https://img.shields.io/badge/Power_Query-M_Language-2CA5E0?style=for-the-badge)](https://learn.microsoft.com/en-us/power-query/)

An interactive **Human Resources Analytics Dashboard** developed in **Power BI Desktop** to analyze employee turnover, workforce demographics, salary distributions, and organizational retention drivers.

---

## 1. Project Overview

Employee attrition poses significant operational and financial challenges for modern organizations. This project provides HR leadership and department managers with an exploratory BI interface to evaluate key workforce metrics, identify flight-risk segments, and investigate the factors influencing employee departures.

The dashboard integrates demographic attributes, compensation tiers, departmental distributions, job roles, and satisfaction ratings into a centralized analytical view.

---

## 2. Dashboard Preview

![HR Analytics Dashboard](assets/HR%20ANALYTICS%20DASHBOARD.png)

---

## 3. Key Performance Indicators (KPIs)

The top KPI ribbon provides an executive-level summary of baseline workforce metrics:

| KPI Card | Displayed Value | Verified Calculation / Definition | Analytical Purpose |
| :--- | :---: | :--- | :--- |
| **Total Employees** | `1.417K` | `Count of EmpID` (1,417 records loaded; 1,403 distinct employee IDs) | Overall workforce headcount under analysis in the report model. |
| **Active Employees** | `1.186K` | `Count of EmployeeCount` with filter `Attrition = "No"` (1,186 employees) | Current active workforce retained within the organization. |
| **Attrition Count** | `231` | `Sum of AttritionCount` (`Attrition = "Yes"`) | Cumulative count of employees who have separated from the company. |
| **Attrition Rate %** | `16.3%` | `[Attrition Rate %] = DIVIDE(SUM(AttritionCount), SUM(EmployeeCount), 0)` | Overall organization-wide turnover percentage (231 / 1,417). |
| **Average Age** | `36.94` | `Average of Age` | Mean age of the workforce across all departments. |
| **Avg Experience** | `7.04` | `Average of YearsatCompany` (7.04 years at company) | Mean company tenure across employees (note: evaluates company tenure). |

---

## 4. Dashboard Features & Visual Layout

The report incorporates six core analytical visuals and two interactive slicers to facilitate multi-dimensional cross-filtering:

### Interactive Slicers
- **Age Group Slicer (Top Ribbon):** Allows quick segment filtering across defined cohorts: `18-25`, `26-35`, `36-45`, `46-55`, and `55+`.
- **Department Slicer (Left Panel):** Enables dedicated departmental drill-downs across `Administration`, `Finance`, `Human Resources`, `IT`, `Marketing`, `Operations`, and `Sales`.

### Visual Analytics
1. **Attrition by Department (Donut Chart):**
   - *Metric:* `Sum of AttritionCount` by `Department`.
   - *Breakdown:* Administration (84 leavers, 36.4%), Sales (52 leavers, 22.5%), Operations (43 leavers, 18.6%), IT (30 leavers, 13.0%), Marketing (10 leavers, 4.3%), Human Resources (9 leavers, 3.9%), and Finance (3 leavers, 1.3%).
   - *Purpose:* Illustrates the departmental share of total company departures.

2. **Attrition by Salary Slab (Clustered Bar Chart):**
   - *Metric:* Employee counts by `SalarySlab` categorized by `Attrition` (`No` vs. `Yes`).
   - *Categories:* `0-3 LPA` (61 leavers), `3-6 LPA` (49 leavers), `6-10 LPA` (73 leavers), and `10+ LPA` (48 leavers).
   - *Purpose:* Highlights turnover volume across compensation bands.

3. **Attrition by Job Role & Job Satisfaction (Matrix Table):**
   - *Metric:* Cross-tabulation of `Sum of AttritionCount` across 9 `JobRole` categories and 4 `JobSatisfaction` levels (1 = Low to 4 = Very High).
   - *Formatting:* Conditional linear gradient heat-map applied to identify concentrated turnover pockets.
   - *Purpose:* Pinpoints specific roles where dissatisfaction correlates with employee exit volume.

4. **Attrition Rate by Age Group (Clustered Bar Chart):**
   - *Metric:* DAX Measure `[Attrition Rate by Age Group]` plotted by `AgeGroup`.
   - *Results:* `18-25` (37.6%), `26-35` (18.9%), `55+` (18.2%), `46-55` (12.2%), and `36-45` (9.2%).
   - *Purpose:* Evaluates cohort-specific turnover risk, demonstrating elevated turnover among younger professionals.

5. **Attrition Trend by Experience (Area Chart):**
   - *Metric:* `Sum of AttritionCount` plotted against `TotalExperience(Years)`.
   - *Purpose:* Visualizes departures across professional career duration (0 to 40 years), highlighting attrition spikes during early career years (0–2 years of experience).

6. **Department-Wise Employee Count (Funnel Chart):**
   - *Metric:* `Count of EmpID` by `Department`.
   - *Headcount:* Operations (512), Administration (406), Sales (207), IT (166), Marketing (64), Human Resources (39), Finance (23).
   - *Purpose:* Displays comparative departmental sizing across the organization.

---

## 5. Dataset Information

The dashboard is powered by the local dataset `HR_Analytics-4.csv`:

- **Filename:** `HR_Analytics-4.csv`
- **Initial Raw Records:** 1,480 rows, 37 columns.
- **Data Ingestion & Cleaning:**
  - **Missing Values Removal:** 61 rows were excluded during data transformation due to null/blank entries in the `YearsWithCurrManager` field.
  - **Duplicate Removal:** 2 exact duplicate rows across all 37 attributes were removed via `Table.Distinct`.
  - **Final Model Records:** **1,417 rows** loaded into the semantic model.
  - *Data Integrity Note:* While exact duplicate records across all 37 columns were eliminated, 14 `EmpID` identifiers appear twice where minor non-key attribute variations existed (representing 1,403 unique employee IDs across 1,417 loaded rows).

---

## 6. Data Preparation (Power Query ETL)

Data transformation was executed within Power Query using the M formula language. The transformations were applied in the following verified sequence:

```powerquery
let
    // 1. Ingest raw CSV data
    Source = Csv.Document(File.Contents("D:\My Works\Projects\Power BI\HR-Analytics-PowerBI-Dashboard\HR_Analytics-4.csv"), [Delimiter=",", Columns=37, Encoding=65001, QuoteStyle=QuoteStyle.None]),
    
    // 2. Promote first row to column headers
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    
    // 3. Cast initial data types across all 37 attributes
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {...}),
    
    // 4. Sort ascending on YearsWithCurrManager (groups nulls at top)
    #"Sorted Rows" = Table.Sort(#"Changed Type", {{"YearsWithCurrManager", Order.Ascending}}),
    
    // 5. Remove the top 61 rows containing null YearsWithCurrManager values
    #"Removed Top Rows" = Table.Skip(#"Sorted Rows", 61),
    
    // 6. Deduplicate exact matching rows across all columns
    #"Removed Duplicates" = Table.Distinct(#"Removed Top Rows"),
    
    // 7. Standardize textual values in BusinessTravel ("Travel_Rarely" -> "TravelRarely")
    #"Replaced Value" = Table.ReplaceValue(#"Removed Duplicates", "Travel_Rarely", "TravelRarely", Replacer.ReplaceText, {"BusinessTravel"}),
    
    // 8. Row selection step
    #"Filtered Rows" = Table.SelectRows(#"Replaced Value", each true),
    
    // 9. Re-verify explicit column data types
    #"Changed Type1" = Table.TransformColumnTypes(#"Filtered Rows", {...}),
    
    // 10. Add conditional indicator column for attrition events (1 for Yes, 0 for No)
    #"Added Conditional Column" = Table.AddColumn(#"Changed Type1", "AttntonCount", each if [Attrition] = "Yes" then 1 else 0),
    
    // 11. Rename conditional column to standard naming
    #"Renamed Columns" = Table.RenameColumns(#"Added Conditional Column", {{"AttntonCount", "AttritionCount"}})
in
    #"Renamed Columns"
```

---

## 7. DAX Measures

The semantic model contains the following verified DAX expressions:

### 1. Overall Attrition Rate %
```dax
measure 'Attrition Rate %' = 
DIVIDE(
    SUM('HR_Analytics-4 (2)'[AttritionCount]),
    SUM('HR_Analytics-4 (2)'[EmployeeCount]),
    0
)
```
*Calculates overall turnover percentage formatted as `0.0%`.*

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
*Evaluates within-bracket attrition percentage formatted as `0.0%` when evaluated against the `AgeGroup` visual axis.*

---

## 8. Tools & Technologies

- **Business Intelligence Platform:** Microsoft Power BI Desktop
- **Data Modeling & Storage:** Tabular Model Definition Language (TMDL), Power BI Project (`.pbip`) format
- **ETL Architecture:** Power Query (M Language)
- **Analytical Calculations:** DAX (Data Analysis Expressions)
- **Data Source:** CSV Flat File (`UTF-8`)

---

## 9. Project Structure

```text
HR-Analytics-PowerBI-Dashboard/
│
├── .gitignore                                      # Git ignore rules for Power BI cache and local settings
├── README.md                                       # Project documentation
├── HR_Analytics-4.csv                              # Source HR dataset
│
├── HR ANALYTICS DASHBOARD.pbip                     # Power BI Project definition file
├── HR ANALYTICS DASHBOARD.pbix                     # Compiled standalone Power BI report file
│
├── HR ANALYTICS DASHBOARD.Report/                  # Report layout and visual definitions (PBIR format)
│   ├── definition.pbir                             # Binding pointer to semantic model
│   └── definition/
│       ├── report.json                             # Theme, global filters, and canvas settings
│       └── pages/
│           ├── pages.json                          # Page hierarchy and active page mapping
│           └── a168e05dc08419019b3c/               # "Page 1" visual configurations
│               ├── page.json                       # Page layout specifications (1280x720)
│               └── visuals/                        # Visual container JSON schemas for all 21 items
│
├── HR ANALYTICS DASHBOARD.SemanticModel/           # Semantic model definitions (TMDL format)
│   ├── definition.pbism                            # Model metadata settings
│   ├── diagramLayout.json                          # Model relationship layout
│   └── definition/
│       ├── model.tmdl                              # Global model configuration
│       └── tables/
│           └── HR_Analytics-4 (2).tmdl             # Schema, M query, column types, and DAX measures
│
├── assets/                                         # Project demonstration media
│   ├── HR ANALYTICS DASHBOARD.png                  # High-resolution dashboard screenshot
│   ├── HR ANALYTICS DASHBOARD.pdf                  # Exported static PDF report
│   └── HR ANALYTICS DASHBOARD.mp4                  # Dynamic video walkthrough of dashboard interactions
│
└── Icons/                                          # Custom visual icon assets
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
- [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/) (version supporting Power BI Project `.pbip` / Fabric report formats).

### Steps
1. **Clone the Repository:**
   ```bash
   git clone https://github.com/HAITHM-ALAKHALI/HR-Analytics-PowerBI-Dashboard.git
   cd HR-Analytics-PowerBI-Dashboard
   ```

2. **Open the Report:**
   - Double-click `HR ANALYTICS DASHBOARD.pbip` (or `HR ANALYTICS DASHBOARD.pbix`) to launch Power BI Desktop.

3. **Configure the Data Source Path (if prompted):**
   - In Power BI Desktop, navigate to **Home** > **Transform Data** > **Transform Data**.
   - In Power Query Editor, select the table query `HR_Analytics-4 (2)` and select the **Source** step in the **Applied Steps** panel.
   - Update the file path to point to your local path of `HR_Analytics-4.csv` within the cloned directory:
     ```powerquery
     = Csv.Document(File.Contents("<Your-Local-Path>\HR-Analytics-PowerBI-Dashboard\HR_Analytics-4.csv"), [Delimiter=",", Columns=37, Encoding=65001, QuoteStyle=QuoteStyle.None])
     ```
   - Click **Close & Apply**.

4. **Interact with the Dashboard:**
   - Use the **Age Group** buttons and **Department** panel to dynamically slice the visual data.

---

## 11. Key Insights

Based on verified data calculations within the model:

1. **Young Professionals Experience Highest Attrition:**
   - Employees aged **18–25** display an attrition rate of **37.6%**, more than double the company-wide average (16.3%). In contrast, mid-career professionals aged **36–45** maintain the highest stability with an attrition rate of **9.2%**.
2. **Departmental Retention Discrepancies:**
   - **Sales** exhibits the highest departmental turnover rate at **25.1%** (52 leavers out of 207 employees), closely followed by **Human Resources** at **23.1%** (9 leavers out of 39 employees) and **Administration** at **20.7%** (84 leavers out of 406 employees).
   - **Operations**, despite being the largest department (512 employees), maintains the lowest attrition rate at **8.4%** (43 leavers).
3. **High-Impact Roles:**
   - **Laboratory Technicians** (59 leavers) and **Sales Executives** (56 leavers) represent the highest cumulative departure volumes across the organization, accounting for **49.8%** of all attrition events.
4. **Early Tenure Vulnerability:**
   - The attrition distribution indicates a heavy concentration of exits within the first 0–2 years of professional experience, indicating that early-career onboarding and career development are critical retention levers.

---

## 12. Project Limitations & Considerations

- **Data Truncation during Cleaning:** Removing 61 rows with blank `YearsWithCurrManager` entries also dropped 8 historical attrition records and 53 active employee records.
- **Identifier Granularity:** Full-row deduplication eliminated exact duplicate records, but 14 `EmpID`s appear twice due to non-key field differences, resulting in 1,417 rows for 1,403 distinct employee IDs.
- **Metric Definitions:**
  - The KPI card titled **"Avg Experience"** aggregates the `YearsatCompany` column (mean: 7.04 years of company tenure) rather than total professional experience (`TotalExperience(Years)`, mean: 11.30 years).
  - The **"Attrition by Department"** Donut Chart illustrates raw departure volume per department rather than department-specific turnover rates.
- **Model Architecture:** The current implementation uses a single denormalized flat table rather than a multi-table dimensional Star Schema.

---

## 13. Author

**Haithm Abdulwali Al-Akhali**  
Power BI & Data Analytics Enthusiast  
GitHub: [@HAITHM-ALAKHALI](https://github.com/HAITHM-ALAKHALI)
