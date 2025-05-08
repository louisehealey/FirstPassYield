# First Pass Yield Report

The **First Pass Yield (FPY) Report** offers a detailed view of manufacturing efficiency by quantifying the percentage of units that successfully pass quality inspection or testing on the first attempt, without requiring rework or reprocessing. This report helps identify areas of improvement, monitor quality trends over time, and support decision-making to enhance overall production performance.

![FPY Report](https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FirstPassYieldReport.png)

## 🧮 The Data Model

# **Data Structure:** 
 This table follows a **Star Schema** design, a central fact table that connects to multiple dimension tables through key relationships.

**Fact Tables (Transactional Data)**
These tables store measurable events and operational data.

- **JOBS_CLOSED** – Records completed production jobs.  
  - **Key Field:** `part_number`
- **FAIL_LOG** – Tracks inspection failures.  
  - **Key Field:** `part_number`
  - 
**Dimension Tables (Descriptive Attributes)**
These tables provide contextual information for analysis.

- **ITEM_MASTER** – Stores part metadata (commodity type, description, cost).  
  - **Key Field:** `part_number`
- **STOCK_STATUS** – Contains inventory details (on-hand quantity, stock location).  
  - **Key Field:** `part_number`
- **Calendar** – Provides date hierarchy for time-based analysis.  
  - **Key Field:** `inspection_date`


![FPY Report](https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FPY_Data_Model.png)


## 🧮 Calculating the First Pass Yield- By Date (Measure)
**(Total Units-Total Unit Failed)/Total Units = First Pass Yield**

This measure tracks FPY on a Month-over-Month basis by aligning **inspection dates** with **calendar dates** using the function **TREATAS**. Since there's no direct relationship between the `FAIL_LOG` and `Calendar` tables, `TREATAS` enables proper data mapping while preserving the **star schema** structure of the model.
``` 
RevenuePerformanceMeasure =
FPYbyDate =
  VAR TotalJC =
      CALCULATE(
          SUM(JOBS_CLOSED[quantity]),
          TREATAS(VALUES('Calendar'[Date]), JOBS_CLOSED[inspection_date]) )
  VAR TotalFAIL =
      CALCULATE(
          SUM('FAIL_LOG'[quantity]),
          TREATAS(VALUES('Calendar'[Date]), 'FAIL_LOG'[inspection_date]))
  VAR TotalPASS = TotalJC - TotalFAIL
RETURN
IF(TotalJC = 0, BLANK(), DIVIDE(TotalPASS, TotalJC))
 ``` 
<p align="center">
  <img src="https://raw.githubusercontent.com/louisehealey/FirstPassYield/main/FirstPassYield.gif">
</p>


---

## 🧮 Calculating the First Pass Yield- By Part (Measure)

This measure follows the same logic as **FPYbyDate** but excludes the **TREATAS** function since both **JOBS_CLOSED** and **FAIL_LOG** are directly related to the **PARTMASTER** table. This ensures accurate FPY calculations while leveraging existing relationships in the data model
 ``` 
FPYbyPart =
VAR TotalJC = CALCULATE(SUM('JOBS_CLOSED'[quantity]))
VAR TotalFAIL=CALCULATE(SUM('FAIL_LOG'[quantity]))
VAR TotalPASS= TotalJC-TotalFAIL
RETURN
IF(
    TotalJC=0,BLANK(), TotalPASS/TotalJC)
 ``` 
REPLACE PHOTO
---


