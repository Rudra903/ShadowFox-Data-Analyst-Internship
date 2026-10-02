# Power BI Build Guide: HR Attrition Executive Dashboard

**Intern:** Rudra Kumar | **SF-PID:** SF-PID-2026-W6251

> **Data note:** `HR_Analytics_Sample_Raw.csv` is a sample dataset I created with the same columns as the IBM HR Analytics dataset. It is not real company data. You can also load the real IBM file from Kaggle. The steps and DAX stay the same, but the numbers will differ.

Power BI Desktop is free but runs on **Windows only**. On a Mac, use **Tableau Public** (free) and follow the same page plan and KPI list.

## Step 1: Load the data
1. Open Power BI Desktop. Click **Get data**, then **Text/CSV**.
2. Select `HR_Analytics_Sample_Raw.csv`. Click **Transform Data**.

## Step 2: Clean and transform (Power Query)
1. Check column types. Numbers must be whole numbers, text must be text.
2. Remove the constant columns `EmployeeCount`, `Over18` and `StandardHours`. They add nothing.
3. Check for nulls and duplicates. Use **Column quality** on the View tab. This sample has none. Note that in your video.
4. Rename the query to `HR`.
5. Click **Close & Apply**.

## Step 3: Add calculated columns (DAX)
In **Table view**, choose **New column** for each:
```
AttritionFlag = IF(HR[Attrition] = "Yes", 1, 0)

AgeBand =
SWITCH(TRUE(),
    HR[Age] < 30, "Under 30",
    HR[Age] < 40, "30-39",
    HR[Age] < 50, "40-49",
    "50+")

IncomeBand =
SWITCH(TRUE(),
    HR[MonthlyIncome] <= 3000, "Under 3K",
    HR[MonthlyIncome] <= 6000, "3K-6K",
    HR[MonthlyIncome] <= 10000, "6K-10K",
    "10K+")

TenureBand =
SWITCH(TRUE(),
    HR[YearsAtCompany] <= 2, "0-2 yrs",
    HR[YearsAtCompany] <= 5, "3-5 yrs",
    HR[YearsAtCompany] <= 10, "6-10 yrs",
    "10+ yrs")
```
To sort bands in the right order, add number columns (for example `AgeBandOrder` = 1 to 4). Then select the band column and use **Sort by column**.

## Step 4: Add measures (DAX)
Create a table to hold them: **Enter data**, name it `Measures`. Then use **New measure**:
```
Headcount = COUNTROWS(HR)
Attrition Count = CALCULATE(COUNTROWS(HR), HR[Attrition] = "Yes")
Active Employees = [Headcount] - [Attrition Count]
Attrition Rate = DIVIDE([Attrition Count], [Headcount])
Avg Monthly Income = AVERAGE(HR[MonthlyIncome])
Avg Tenure (Years) = AVERAGE(HR[YearsAtCompany])
Avg Job Satisfaction = AVERAGE(HR[JobSatisfaction])
Overtime Attrition Rate = CALCULATE([Attrition Rate], HR[OverTime] = "Yes")
No-Overtime Attrition Rate = CALCULATE([Attrition Rate], HR[OverTime] = "No")
Overall Attrition Rate = CALCULATE([Attrition Rate], ALL(HR))
Gap vs Overall = [Attrition Rate] - [Overall Attrition Rate]
Avg Income of Leavers = CALCULATE([Avg Monthly Income], HR[Attrition] = "Yes")
Avg Income of Stayers = CALCULATE([Avg Monthly Income], HR[Attrition] = "No")
```
Set formats: rates to percentage with 1 decimal, income to currency with 0 decimals.

## Step 5: Build the pages
Use one theme. Navy `#1F3864` for headers, a light blue `#DDEBF7` for cards, and one accent colour (orange `#ED7D31`) for attrition. Put page titles at the top left.

### Page 1: Executive Overview
- **KPI cards (top row):** Headcount, Attrition Count, Attrition Rate, Avg Monthly Income, Avg Tenure, Avg Job Satisfaction.
- **Slicers (left side):** Department, Gender, JobRole, AgeBand, OverTime. Use the **Sync slicers** pane so they apply to all pages.
- **Visuals:**
  - Clustered bar: `Department` and `Attrition Rate`.
  - Column chart: `AgeBand` and `Attrition Rate`.
  - Donut: `Attrition` and `Headcount`.
  - Bar chart: `JobRole` and `Headcount`, with data labels.
- **Drill-down:** Create a hierarchy `Department > JobRole`. Use it in the bar chart so users can drill down.

### Page 2: Attrition Drivers
- Column chart: `OverTime` and `Attrition Rate`.
- Column chart: `BusinessTravel` and `Attrition Rate`.
- Column chart: `IncomeBand` and `Attrition Rate`.
- Column chart: `TenureBand` and `Attrition Rate`.
- Matrix: rows `Department`, columns `JobSatisfaction`, values `Attrition Rate`. Add **Conditional formatting** (background colour scale).
- Scatter: x `Age`, y `MonthlyIncome`, legend `Attrition`.
- Optional: **Decomposition tree** with analyse = `Attrition Count`, explain by `OverTime`, `Department`, `IncomeBand`.

### Page 3: Employee Details (drill-through)
- Table: `EmployeeNumber`, `JobRole`, `Department`, `Age`, `MonthlyIncome`, `YearsAtCompany`, `OverTime`, `Attrition`.
- Drag `JobRole` into **Drill through** fields. Then on Pages 1 and 2, right-click a bar and choose **Drill through**.

### Optional extras
- A **Bookmark** button that resets all filters.
- A **Tooltip page** that shows headcount and attrition rate on hover.
- Alt text on each visual.

## Step 6: Save and export
1. Save as `HR_Attrition_Dashboard.pbix`.
2. **File > Export > PDF** and take screenshots of each page.
3. Put the `.pbix`, the PDF and the screenshots in the `advanced/` folder of your GitHub repo.

## Step 7: Check your numbers
If you loaded my sample CSV, your cards and charts should show these values:

| Check | Expected value |
|---|---|
| Headcount | 1,470 |
| Attrition Count | 238 |
| Attrition Rate | 16.2% |
| Avg Monthly Income | USD 7,071 |
| Avg Tenure | 6.71 years |
| Avg Job Satisfaction | 2.71 |
| Overtime attrition rate | 26.6% |
| No-overtime attrition rate | 12.2% |
| Sales / R&D / HR attrition | 20.4% / 14.9% / 7.0% |
| Under 30 / 50+ attrition | 27.4% / 10.0% |
| Under 3K / 10K+ income band | 25.3% / 4.2% |
| Sales Representative attrition | 32.2% |

If a value does not match, check the measure formula and the slicers (clear all slicers first).
